# OpenDMR

## Коротко

OpenDMR — проєкт, який ідеєно ставить собі за мету зробити open-source DMR firmware для TYT MD-380 на базі власної розробки, а не просто “патчі до vendor firmware”. Це дуже важливо: він демонструє збільшення видимості, коли ідея ядра зсунута з vendor-обмежень до відкритої модульної логіки.

GitHub:
https://github.com/KG5RKI/OpenDMR

## Що тут важливо

OpenDMR виявляється цікавішим саме як архітектурний підхід, а не як готовий DMR-суперінструмент:
- він формує реальну open-source альтернативу OEM-фірмварі
- описує як можна писати DMR firmware “з нуля” або з нуля з мінімальною залежністю від vendor bindings
- показує, що "крипто" в DMR радіо — це не особливий високий магічний блок, а частина системи: key schedule, slot state, algorithm selection, IV generation, stream xor

## Як у ньому відбувається шифрування

У відкритих DMR stack проектах, як правило, схема така:

1. Сигнал потрапляє в DMR stack
2. Визначається `alg id`, `key id`, `slot`, `MI`, `data`
3. Формуються IV / keystream / LFSR seed
4. Проводиться xor або OFB-подібна операція 
5. Після цього payload стає валідним для подальшої обробки

Тобто в кастомних збірках шірування часто реалізується не “у скритому vendor code”, а більш прозоро в самому voice/data processing pipeline.

## Що змінено від класики

Класична OEM прошивка:
- cryptographic state hidden in deep radio driver / media stack
- key import and selection tightly coupled with vendor configuration
- UI often not editable

OpenDMR:
- зміщення у бік відкритого data flow
- логіка key selection і crypto pipeline більш структурована
- easy to instrument for research and custom key handling

## Важливість для дослідження

OpenDMR — хороший проєкт для розуміння того, як думати про DMR радіо як про програмну систему, а не як про “закритий пристрій”. Це дуже цінно для розробки власного декодера, бо в цьому проекті crypto вже не виглядає як “secret layer”, а як інтегрована частина радіо-стека.

## Рекомендація

Цей проект важливий як архітектурний лінк між old vendor firmware і modern custom open firmware. Його варто розглядати не як “готовий aes patch”, а як демонстрацію того, як DMR крипто-логіка ставиться в відкритий code flow.

