# Порівняння проектів: OEM vs кастомні прошивки

## 1. OEM firmware

Виробнича прошивка зазвичай має такі характеристики:
- закритий код і vendor SDK
- жорстка схема key storage
- crypto logic прихована в runtime pipeline
- доступ до ключів і шифрування часто обмежений CPS або vendor tool
- швидкі runtime-оптимізації, але мало прозорості

## 2. Кастомні прошивки

Кастомні прошивки зазвичай змінюють такі речі:
- menu і key UI
- key storage format
- flash offset / codeplug structure
- key selection and alg id flow
- product integration with AES, ARC4, TEA or open local store

## 3. Чим вони відрізняються в криптографії

### OEM
- vendor code handles:
  - key selection
  - key load
  - IV / key stream
  - encryption / decrypting logic
- bolt-on security layer is not transparent

### Кастомна прошивка
- дозволяє або розширити або відкрити:
  - key input UI
  - key table management
  - clearer logic for key id / alg id / slot
  - local cryptography support or external wrappers

## 4. Типові зміни в коді кастомних проектів

У типових кастомних DMR firmware знаходимо такі речі:
- нові масиви ключів у flash
- додаткові menu items для key name, enable/disable
- custom settings for `alg id` / `key id`
- зміна в DMR voice/data processing
- крипто-функції часто мають вигляд “key schedule → keystream → xor payload”, а не “vendor secret box”

## 5. Чим модифікації виглядають на практиці

У реальних проєктах з відкритим кодом:
- фабрична cryptographic pipeline не видаляється повністю
- вона розкладається на ясніші етапи
- key handling і storage стають більш доступними
- часто додається більше перевірок і управління параметрами mode/slot

## 6. Для декодера

Якщо ви хочете будувати власний декодер, важливо знати таке:
- кастомні прошивки часто “показують” місце, де в системі формується key stream
- вони зменшують vendor lock-in і допомагають зрозуміти, який майданчик коду потребує модифікації
- crypto не є магією: це pipeline, що складається з key table, IV, alg id, xor і CRC validation

## 7. Короткий висновок

Кастомні прошивки не “зламують” DMR шифрування краще за OEM просто так. Вони більше:
- розкривають структуру системи
- відкривають key management
- реорганізують crypto logic
- дають інженерний доступ до того, що OEM тримав закритим

Саме ця відкритість — головна цінність для дослідження і для побудови власного декодера.
