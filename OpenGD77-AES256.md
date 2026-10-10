# OpenGD77-AES256

## Коротко

OpenGD77-AES256 — дуже важливий проект, бо в ньому явно видно одну з найбільш практичних ідей серед кастомних прошивок: модифікація DMR firmware під локальні ключі, крипто-алгоритми і розширений key handling.

GitHub:
https://github.com/dondch/OpenGD77-AES256

## Чому саме цей проект цікавий

У ньому явно видно, що кастомна прошивка не просто “прибирає обмеження”. Вона:
- додає support для AES-256
- розширює key handling в DMR-стеку
- модифікує логіку шифрування і work with keystream
- має реальні сервіси, які були відсутні в vendor code

## Як виконується шифрування в OpenGD77-AES256

Типова DMR crypto pipeline в проектах такого типу виглядає так:

1. визначення `alg_id` і `key_id`
2. побудова IV / seed / MI-expansion
3. генерація keystream з AES / RC4 / DES / TEA-style функцій
4. xor з payload або блоками даних
5. перевірка CRC / frame validity / decryption success

У AES256-напрямку це зазвичай не “магічна заміна всієї системи”, а модифікація конкретних місць:
- key schedule generation
- key-selection logic
- IV expansion / LFSR seed
- output keystream generation
- application of xor to voice/data payload

## Як це відрізняється від класичної прошивки

Класична OEM прошивка:
- vendor-замкнута реалізація AES/ARC4/TEA
- key storage і key alignment ідентичні vendor policy
- UI і CPS не розкривають доступ до ключів

OpenGD77-AES256:
- краще видно, де в DMR коду йде key flow
- ключі можуть бути частиною розширеної local config
- логіка шифрування стає локально керована інженером
- розкриває реальний принцип: crypto is not hidden art, it is keyed stream processing

## Чим він корисний для Decode research

Це один із найкращих прикладів для розробника, який хоче зрозуміти:
- де в DMR стеку шифрування реально впроваджується
- як key id, slot і payload interact з keystream
- як AES-256 pipeline на практиці інтерфейсується з радіо firmware
- чому кастомні прошивки часто є більш зрозумілими, ніж OEM

## Важливий висновок

OpenGD77-AES256 — один із найбільш цінних репозиторіїв для тих, хто хоче вивчити “практичний crypto layer” DMR firmware. Він показує, що кастомний firmware часто модифікує не весь радіо, а конкретну крипто-частину і key flow.

