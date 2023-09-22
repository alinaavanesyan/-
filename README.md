Все файлы, относящиеся к алогритму, находятся в папке *function*:
- *algorithm.ipynb* (файл с алгоримтмом)
- *uniq_cases.xlsx* (список с редкими словами)
- *предлоги.txt* (список предлогов русского языка)
- *даль_словарь.txt* (толковый словарь Даля) (файл слишком большой, поэтому не поместился в гитхабе, скачать его можно [здесь](https://drive.google.com/file/d/1IwG7tVARlRqu-_cXIx6wrLJmgNVCN5e5/view?usp=sharing))
- *Обучающий корпус.xlsx* (размеченные данные Обучающего корпуса)
- *main_source_list.txt* (пример токенов)
Для запуска скрипта Вам понадобятся все вышперечисленные файлы (кроме *main_source_list.txt* , если у Вас есть собственный список токенов, которые Вам нужно разметить)

# Цель проекта 

Целью проекта является разработка алгоритма для автоматизации разметки несловарных токенов НКРЯ и подготовка с его помощью размеченного датасета Основного корпуса данных.

Несловарные токены в контексте нашей задачи представляют собой слова с дефисами, которые:
- либо искажены (токены удвоением/пропуском букв)

Пример: *поко-о-ой-йю-у-у > покою*
- либо являются междометиями

Пример: *уррр-аа-аэ*

## Задачи:
**1) нормализация данных**

1.1. Привести слово к нормальному виду, убрав ненужные повторы

1.2. Определить лемму (начальную форму) исправленного слова

1.3. Приписать слову частеречный тэг.

**2) разработка классификатора “искажение-междометие”**
Необходимо отделить искаженное слово, которое, например, может являться существительным, глаголом, наречием и т.д., от междометия, в котором растягивание букв и постановка дефисов прагматически мотивированы.

**3) разметка токенов Основного корпуса НКРЯ с помощью написанной функции**
Разметка должна осуществлятья в соответствии со следующими правилами:
- для искаженных форм:
<img height='25' src='https://github.com/alinaavanesyan/Automating-the-annotation-of-corpus-data/blob/main/Искаженные_формы.png'>
-  для междометий/звукоподражаний
<img height='20' src='https://github.com/alinaavanesyan/Automating-the-annotation-of-corpus-data/blob/main/Междометие.png'>

**4) оценка возможностей применения спеллчекера как дополнительного инструмента нормализации данных.**

## Методы
Алгоритм нормализации данных происходит по следующей схеме:

I) Сперва проверяется наличие токеан в словарях, в составе которые:
- размеченные данные части Обучающего корпуса (Обучающий корпус.xlsx)
- собранные самостоятельно предлоги русского языка
- словарь Даля, из которого вытащены токены,
- а также таблица с уникальными случаями, включающая в себя частотные специфичные токены, нормализацию которые не удастся провести с помощью алгоритма (например, алгоритм не сможет понять, что “гасаа–а” – это на самом деле “господа”). 

Если токен в словаре, это значит, что он корректен → его нормализация не нужна. К такой группе относятся слова, где постановка дефиса грамматична: светло-зеленый, Эр-Рияд и т.д.

II) Если токена не оказалось в словаре:
сперва проводятся тесты, которые позволяют отсечь такой вариант, что дефис всё-таки необходим, но слова просто не оказалось в словаре (либо оно очень редкое, либо стоит не в начальной форме)

В таком случае по внешним признакам можно сразу определить и часть речи. С помощью многочисленных регулярных выражений осуществляется поиск следующих групп:
1) прилагательное  и сущ. с -то (полный-то, страшный-то / стол-то, квартира-то и т.д.)
2) имен собственных, начинающихся с заглавной буквы
3) наречия и местоимения:
с кое- (кое-где, кое-как, кое-что, кое-кто и др. + косвенные формы)
на -то, -либо, -нибудь (чей-то, кому-то, сколько-то и т.д.)
При этом попутно исправляются недочета вида “нить” (нибудь), “койни” (кое), “чиво” (что) и др.
4) наречия с приставкой по- и суффиксом и/ски/цки/ому/ему (по-царски, по-иному и т.д.)
5) стороны света (северо-восток, зюйд-вест)
6) существительные с приставками авто-, агро-, вело- и т.д., после которых требуется дефис
7) существительные типа пол-лимона, пол-апельсина и т.д.

III) Если токен не подходит ни под одну категорию, слово проверяется на “междометность”.
В тестах в основном учитывается соотношение кол-ва согласных к кол-ву гласных, а также их состав, например:
- по одному из тестов, если слово состоит из одной гласной и одной согласной буквы, при том, что != ‘не, но, он…’ (и другие немногочисленные слова), оно является междометием → нормализация не требуется, часть речи определена.
- если слово начинается с гласной, а остальные его части одинаковые, то это междометие: э-ге-геее, о-хо-хоооо
- если слово состоит только из гласных или только из согласных, то это междометие: ооо-оооо, еее-еее, фрр-рр, бр-ррр
- если слово начинается с Ы, после которой следует согласный, то это междометие (если после Ы следует гласный, это слово может быть искаженным, а не междометием: ыэээ-ра)

Также можно выделять тест, который выявляет слова, состоящие из двух повторяющихся частей. Мы можем гарантировать, что это междометие, если эти части двухбуквенных (т.к. таких слов не междометий ограниченное количество: папа, мама, дядя, няня, баба): ха-ха

IV) Если токен не определен как междометие, начинается процедура нормализации искаженных форм, в которых дефис не нужен, согласно нормам русского языка.

Слова делятся на слоги, где разделителем выступает дефис. Повторяющиеся слоги удаляются, после чего каждой слог сравнивается с последующим: их пересечение удаляется (предполагается, что это растягивание/удвоение букв, которого не требует словарь). Если слог одинарный, он крепится к следующему за ним.

Далее удаляется удвоение одинаковых букв на конце слова (если между ними не было дефиса, они были в одном слоге, соответственно, мы не смогли избавиться от этого удвоения на предыдущем этапе). Если окончание не входит в список допустимых окончаний русского языка (-юю, -ии, -ее, -яя), оно сокращается до одной буквы.
Также в алгоритмы есть словари и списки сочетаний, невозможных в русском языке. Эти сочетания заменяются на наиболее вероятные (например, ‘йа’ переходит в ‘я’, ‘ыы’ в ‘ы’ // удаляется ь между двумя гласными и т.д.).

V) После нормализации восстанавливается исходный регистр слова, а также определяется его начальная форма (с помощью библиотеки pymorphy2). Если начальная форма и теперь не находится в словаре, то это сигнализирует о необходимости исправить слово. Здесь полезным оказывается спеллчекер (библиотека SpellChecker), поскольку помогает исправить недочеты, с которыми не смог справиться алгоритм (например, удвоенное -н, непроизносимые согласные (‘серце’ → ‘сердце’).

VI) Функция алгоритма возвращает список с исправленным словом (не лемматизированным!) и его частью речи (если ее удалось определить).

Например, ‘ууура-ааа’ → [‘ууура-ааа’, междометие]
‘покойуу-юю’ → [‘покою’, ‘’]

Если слово не получило частеречный, функции lemma_func и part_of_speech проходятся с помощью pymorphy2 по результатам и определяют лемму и часть речи токена соответственно.

## Результаты
Результаты разметки были представлены в виде датафрейма:
<img height='400' src='https://github.com/alinaavanesyan/Automating-the-annotation-of-corpus-data/blob/main/Размеченные_данные.png'> *пример разметки*

Точность алгоритма, измеренная метрикой accuracy, составила 96%.

С помощью предложенного алгоритма удалось разметить [111830 токенов Основного корпуса](https://drive.google.com/file/d/1IwG7tVARlRqu-_cXIx6wrLJmgNVCN5e5/view?usp=sharing](https://docs.google.com/spreadsheets/d/1kb43EWXHl7nssNoB_e0yewwewK47z21q/edit?usp=sharing&ouid=112015372771126047321&rtpof=true&sd=true)https://docs.google.com/spreadsheets/d/1kb43EWXHl7nssNoB_e0yewwewK47z21q/edit?usp=sharing&ouid=112015372771126047321&rtpof=true&sd=true), в которых был найден дефис.
