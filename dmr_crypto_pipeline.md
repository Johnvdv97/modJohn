# DMR Crypto Pipeline: where it lives in custom firmware

Цей документ пояснює, де в DMR firmware зазвичай живе крипто-pipeline, як вона зв’язана з key storage і як це видно в кастомних прошивках типу OpenRTX / md380tools / OpenGD77-AES256.

## 1. Коротка відповідь

У DMR firmware крипто-pipeline зазвичай не лежить в одному “магічному файлі”. Вона розкидана по кількох частинах коду:

- key storage / key table
- key selection / key matching
- algorithm dispatch
- IV / seed / message-id logic
- keystream generation
- payload XOR / decrypt stage
- frame validation / CRC check / decode result

Тобто crypto — це не одне місце, а цілий pipeline в рантаймі і flash-структурах.

## 2. Типовий логічний ланцюжок

```c
if (frame_has_encryption_flag) {
    key = key_store.find_by_key_id(frame.key_id);
    alg = select_algorithm(frame.alg_id, key);

    if (!key.valid) {
        mark_state("needs key");
        return;
    }

    iv = build_iv(frame.mi, frame.slot, frame.cc, frame.ts, frame.frame_index);
    ks = generate_keystream(alg, key, iv, frame);

    payload = xor_payload(frame.payload, ks);

    if (validate_crc(payload)) {
        emit_decoded_voice_or_data(payload);
    } else {
        emit_invalid_or_encrypted_frame();
    }
}
```

Це узагальнена модель. Де саме вона розбита в реальні прошивки — залежить від проєкту, але логіка майже завжди однакова.

## 3. Де саме в коді це живе в реальних кастомних прошивках

### 3.1. Key storage layer

Це часто в файлах, які працюють з:

- key table
- codeplug memory
- flash layout
- radio settings
- user config

Шукати треба слова типу:

- `key_id`
- `key_data`
- `codeset`
- `key_table`
- `radio_key`
- `user_key`
- `encrypt_key`
- `alg_id`
- `crypto_mode`
- `enable_encrypt`

Це найчастіше — перший важливий блок.

### 3.2. Key selection / matching layer

Після того, як прийшов кадр або voice packet, потрібно вибрати правильний key. Для цього в коді є типові місця:

- `find_key_by_id()`
- `match_key()`
- `select_key()`
- `lookup_key()`
- `get_key_for_slot()`
- `key_for_timeslot()`

Тут береться:
- key id із кадру / фрейму
- slot / talkgroup / CC / TS / MI
- активний key set радіо
- algorithm mode

### 3.3. Algorithm dispatch layer

Далі код вибирає, який алгоритм застосувати:

- AES-128
- AES-256
- ARC4 / RC4
- TEA / TEA1 / TEA2
- DES / vendor custom stream

В коді пошукати треба:

- `alg_id`
- `switch (alg_id)`
- `if (alg == AES128)`
- `if (alg == AES256)`
- `if (alg == ARC4)`
- `if (alg == TEA)`
- `crypto_dispatch()`
- `select_algorithm()`

Тут і відбувається “вибір крипто-алгоритму”.

### 3.4. IV / seed / message-id generation

Після того, як вибрали ключ і алгоритм, треба побудувати seed для генерації keystream. У DMR важливі:

- frame index
- slot
- timeslot
- talkgroup / service ID
- message identifier (MI)
- color code / logical channel
- frame number / burst number

Пошукати треба:

- `iv_derive`
- `generate_iv`
- `derive_seed`
- `build_iv`
- `lfsr`
- `message_id`
- `mi_to_seed`

Це дуже важливий блок, бо саме тут інколи “виправляється” відмінність між OEM і кастомною прошивкою.

### 3.5. Keystream generation layer

Далі йде формування keystream. Це головний crypto-блок:

- key schedule expansion
- AES round function or RC4 output
- keystream buffer
- xor mask

Пошукати треба лексеми:

- `keystream`
- `xor_mask`
- `decrypt_block`
- `encrypt_block`
- `generate_stream`
- `stream_key`
- `aes_schedule`
- `rc4_ks`
- `tea_round`
- `lfsr_next`

### 3.6. XOR payload stage

Після генерації keystream — apply to the payload:

- voice payload
- data payload
- sync bytes / service frame data
- header bytes

Це часто робиться дуже явно:

- `for (...) payload[i] ^= keystream[i];`
- `xor_payload()`
- `decrypt_payload()`
- `apply_stream()`

### 3.7. Validation stage

Потім радіо перевіряє, чи це дійсний decrypt result:

- CRC
- frame length
- valid control / data field
- slot and timeslot matching
- valid voice / data header

Шукати:

- `crc_ok`
- `validate_frame`
- `frame_valid`
- `check_crc`
- `decrypt_status`

## 4. Яка структура у типових open firmware проєктах

У кастомних прошивках зазвичай є три видимі шари:

### 4.1. UI / config / settings layer

Тут живуть:
- key import
- key table
- enable/disable toggle
- slot binding
- algorithm flags
- key id editing

### 4.2. Runtime stack layer

Тут живе сама DMR state machine:
- decode voice
- decode data
- route to slot
- select algorithm
- do key lookup
- generate stream
- validate

### 4.3. Low-level crypto layer

Тут живе реальний crypto engine:
- AES-128 / AES-256 logic
- RC4 / ARC4 generator
- TEA or vendor-specific
- LFSR / keystream scheduler

## 5. Як це відрізняється від OEM прошивки

У OEM прошивці ці блоки часто стиснуті в vendor-бібліотеки, а UI і codeplug ідентичні “vendor-фіксованим” даним. Тому ви в основному бачите:

- vendor-specific names for APIs
- tight coupling between key table and UI
- crypto hidden in vendor functions
- tougher reverse engineering

У кастомних прошивках ці блоки часто розірвані і видимі, бо є відкриті knobs і key settings. Тому легко побачити:

- як key таблиця зберігається
- як algorithm id передається в crypto
- де seed і keystream будуються
- як payload XOR застосовується

## 6. Що саме шукати у коді

Якщо розбираєте кастомну прошивку, шукайте ці ключові компоненти:

- `key_id`
- `alg_id`
- `encrypt` / `decrypt`
- `dmr_voice`
- `dmr_data`
- `slot`
- `timeslot`
- `mi`
- `iv`
- `seed`
- `keystream`
- `xor`
- `crc`
- `aes`
- `arc4`
- `tea`
- `rc4`
- `lfsr`
- `crypto_state`

## 7. Практичний підхід для декодера

Якщо ви пишете власний декодер, вам треба відтворити саме такий набір:

1. parse key data from keys file or flash layout
2. map key id to active key
3. read alg id and slot selection from frame
4. build IV from frame metadata
5. generate keystream
6. XOR payload
7. check CRC and decode voice/data

Це і є “ціль” DMR crypto pipeline.

## 8. Короткий висновок

У кастомних прошивках DMR crypto pipeline зазвичай виглядає так:

- key storage → key selection → algorithm dispatch → IV generation → keystream generation → XOR payload → validation

І саме цей ланцюжок треба відтворити в декодері, якщо хочеш не просто “зчитувати сигнал”, а реально розуміти, як DMR шифрування працює і де замінюються ключі та алгоритми.

## 9. Що далі

Я можу далі додати в репозиторій окремий файл:
- `dmr_crypto_pipeline.md`

У ньому розпишу:
- точні кроки pipeline по секціях
- що пошукати в OpenRTX, md380tools, OpenGD77-AES256
- як цей pipeline відбивається в custom firmware vs OEM
- чим відрізняється AES-128, AES-256, ARC4 і TEA у контексті DMR

Якщо хочеш — я зроблю саме цей файл в `modJohn` наступним.