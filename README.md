# Алгоритм Лемпеля-Зива-Велча
###### ***Выполнил студент гр. № Б9121-09.03.03пикд(3)***
###### ***Кирилов Денис Юрьевич***
###### ***Дальневосточный федеральный университет, 2022г.***

# 1. Введение
Алгоритм Лемпеля—Зива—Велча (Lempel-Ziv-Welch, LZW) — это универсальный алгоритм сжатия данных без потерь, созданный Авраамом Лемпелем (англ. Abraham Lempel), Яаковом Зивом (англ. Jacob Ziv) и Терри Велчем (англ. Terry Welch). Оригинальная версия метода была создана Лемпелем и Зивом в 1978 году (LZ78) и доработана Уэлчем в 1984 году.


Если взглянуть почти на любой файл данных в компьютере, просматривая символ за символом, то можно заметить множество повторяющихся элементов. LZW — это метод сжатия данных, который воспользовался этим повторением.


Как и в любом адаптивном/динамическом методе сжатия, идея заключается в том, чтобы (1) начать с исходной модели, (2) читать данные по частям, (3) обновлять модель и кодировать данные по мере продвижения. LZW — алгоритм сжатия на основе "словаря", т. е. LZW кодирует данные, обращаясь к словарю. Таким образом, чтобы закодировать подстроку, в выходной файл нужно записать только одно кодовое число, соответствующее индексу этой подстроки в словаре.

### 1.1. Преимущества алгоритма LZW
* Алгоритм является однопроходным.
* Для декомпрессии не надо сохранять таблицу строк в файл для распаковки. Алгоритм построен таким образом, что мы в состоянии восстановить таблицу строк, пользуясь только потоком кодов.
* Высокая степень сжатия.
* Достаточно высокая скорость сжатия и распаковки.

### 1.2. Недостатки алгоритма LZW
* Алгоритм не проводит анализ входных данных.

# 2. Применение
На момент своего появления алгоритм LZW давал лучший коэффициент сжатия для большинства приложений, чем любой другой хорошо известный метод того времени. Он стал первым широко используемым на компьютерах методом сжатия данных.

Алгоритм (а точнее его модификация, LZC, см. ниже) был реализован в программе compress, которая стала более или менее стандартной утилитой Unix-систем приблизительно в 1986 году. Несколько других популярных утилит-архиваторов также используют этот метод или близкие к нему.

В 1987 году алгоритм стал частью стандарта на формат изображений GIF. Он также может (опционально) использоваться в формате TIFF. Помимо этого, алгоритм применяется в протоколе модемной связи v.42bis и стандарте PDF (хотя по умолчанию большая часть текстовых и визуальных данных в PDF сжимается с помощью алгоритма Deflate).

# 3. Сжатие
LZW-алгоритм начинает работу с исходного словаря символов и использует его в качестве "стандартного" набора. В начале словарь представляет собой односимвольные цепочки, которые могут встретиться в процессе работы алгоритма. Процесс кодирования состоит в том, что алгоритм считывает из входного потока символ и проверяет, есть ли этот символ в словаре. Встретив символ в словаре, алгоритм кодирует их в виде числа, которое представляет собой индекс в этом самом словаре. Всякий раз, встречая новую подстроку (например, "ab"), он добавляет ее в словарь; каждый раз, когда ему попадается подстрока, которая ранее уже встречалась, он просто считывает новый символ и выполняет его конкатенацию с текущей строкой, чтобы получить новую подстроку.

Говоря простым языком, LZW-алгоритм считывает очередной непрочитанный символ и пытается найти его в словаре. Если он в словаре есть, то алгоритм пытается его удлинить, путём добавления следующего, и ищет в словаре уже удлиненную цепочку. Как только алгоритм находит цепочку, которая в коротком варианте уже записана в словаре, а в удлинённом нет, короткий вариант кодируется индексом, который уже есть в словаре, а удлиненный вариант заносится в словарь.

### 3.1. Пример
Предположим, что наш поток, который мы хотим сжать, это ***«abdabccabcdab»***, при этом используется только начальный словарь:
|ИНДЕКС|ВХОД|
|:-----:|:-----:|
|1|a|
|2|b|
|3|c|
|4|d|


Этапы кодирования будут выполняться следующим образом:


|ИНДЕКС|ТЕКУЩАЯ СТРОКА|ВИДЕЛИ ЭТО РАНЬШЕ?|КОДИРОВАННЫЙ ВЫХОД|НОВАЯ ЗАПИСЬ В СЛОВАРЕ/ИНДЕКС|
|:-----:|:-----:|:-----:|:-----:|:-----:|
|*а*|*а*|да|ничего|никакой|
|a*b*|a*b*|нет|0|ab / 4|
|ab*d*|b*d*|нет|0, 1|bd / 5|
|abd*а*|d*а*|нет|0, 1, 3|da / 6|
|abda*b*|a*b*|да|без изменений|никакой|
|abdab*c*|ab*c*|нет|0, 1, 3, 4|abc / 7|
|abdabc*c*|c*c*|нет|0, 1, 3, 4, 2|cc / 8|
|abdabcc*a*|c*a*|нет|0, 1, 3, 4, 2, 2|ca / 9|
|abdabcca*b*|a*b*|да|без изменений|никакой|
|abdabccab*c*|ab*c*|да|без изменений|никакой|
|abdabccabc*d*|abc*d*|нет|0, 1, 3, 4, 2, 2, 7|abcd / 10|
|abdabccabcd*a*|d*a*|да|без изменений|никакой|
|abdabccabcda*b*|da*b*|нет|0, 1, 3, 4, 2, 2, 7, 6|dab / 11|
|abdabccabсdab|*b*|да|0, 1, 3, 4, 2, 2, 7, 6, 1|никакой|

После считывания последнего символа, ‘b’, должен быть выведен индекс последней подстроки, ‘b’.

### 3.2. Псевдокод алгоритма сжатия
```
string s;
char ch;
...
s = empty string; //пустая цепочка
while (there is still data to be read) //пока есть символы во входном потоке
{
 ch = read a character; //взять очередной символ
 if (dictionary contains s+ch) //цепочка s+ch уже есть в таблице
 {
s = s+ch; //удлиняем цепочку прочитанным символом
 }
 else //цепочки s+ch нет в таблице
 {
encode s to output file; //вывод кода, соответствующего цепочке s
add s+ch to dictionary; //добавляем новую цепочку s+ch в таблицу
s = ch; //готовимся к «обнаружению» следующей цепочки
 }
}
encode s to output file; //вывод кода для оставшейся цепочки s
```
Алгоритм кодирования каждый раз пытается найти в таблице наиболее длинную цепочку, соответствующую читаемой последовательности символов. Если это в какой-то момент не удается, то накопленная к этому времени цепочка заносится в таблицу. В какой-то момент может наступить переполнение таблицы. В этом случае кодировщик выводит в выходной поток специальный код очистки, и таблица цепочек инициализируется заново. Обычно в реальных алгоритмах применяется таблица с 4096 входами для цепочек, при этом число 256 является кодом очистки (CLC), а число 257 – кодом конца информации (EOI), эти строки таблицы не используются для цепочек.

Заметим, что в этом случае для каждого выходного кода достаточно 12 бит. Поэтому при выводе целесообразно упаковывать каждые два кода в три байта.

# 4. Распаковка
Формирование словаря независимо от того, находимся мы на стороне кодировщика или декодировщика. И кодировщик и декодировщик работают по одной и той же схеме, и поэтому словарь они заполняют как бы синхронно. Поэтому словари у них получаются идентичные и никакой дополнительной информации о частотных или каких-либо ещё характеристиках набора символов им передавать не нужно. В процессе работы словарь, идентичный тому, который был создан во время сжатия, восстанавливается. Программы кодирования и декодирования должны начинаться с одного и того же начального словаря.

Декодер LZW сначала считывает индекс (целое число), ищет этот индекс в словаре и выводит подстроку, связанную с этим индексом. Первый символ этой подстроки конкатенируется с текущей рабочей строкой. Эта новая конкатенация добавляется в словарь. Затем декодированная строка становится текущей рабочей строкой (текущий индекс, т. е. подстрока, запоминается), и процесс повторяется.

### 4.1. Пример
Этапы декодирования будут выполняться следующим образом
|ИНДЕКС|ПРЕОБРАЗОВАНИЕ СЛОВАРЯ|ДЕКОДИРОВАННЫЙ ВЫХОД|ТЕКЩАЯ СТРОКА|НОВАЯ ЗАПИСЬ В СЛОВАРЕ/ИНДЕКС|
|:-----:|:-----:|:-----:|:-----:|:-----:|
|*0*|0 = *а*|*а*|никакая|никакая|
|0, *1*|1 = *b*|a*b*|a|ab / 4|
|0, 1, *3*|3 = *d*|ab*d*|b|bd / 5|
|0, 1, 3, *4* |4 = *ab*|abd*ab*|d|da / 6|
|0, 1, 3, 4, *2*|2 = *c*|abdab*c*|ab|abc / 7|
|0, 1, 3, 4, 2, *2*|2 = *c*|abdabc*c* |c|cc / 8|
|0, 1, 3, 4, 2, 2, *7* |7 = *abc*|abdabcc*abc*|c|ca / 9|
|0, 1, 3, 4, 2, 2, 7, *6*|6 = *da*|abdabccabc*da*|abc|abcd / 10|
|0, 1, 3, 4, 2, 2, 7, 6, *1*|1 = *b*|abdabccabcda*b*|da|dab / 11|

### 4.2. Возможные проблемы при декодировании
Есть исключение, когда алгоритм не работает; это происходит, когда код вызывает индекс, который еще не был введен в словарь (или когда только что закодированная строка встречается на входе). Например, предположим, у нас есть строка «abababa» и начальный словарь, состоящий только из a и b с индексами 0 и 1 соответственно. Тогда процесс кодирования будет состоять из следующих этапов:

|ИНДЕКС|ТЕКУЩАЯ СТРОКА|ВИДЕЛИ ЭТО РАНЬШЕ?|КОДИРОВАННЫЙ ВЫХОД|НОВАЯ ЗАПИСЬ В СЛОВАРЕ/ИНДЕКС|
|:-----:|:-----:|:-----:|:-----:|:-----:|
|*а*|*а*|да|ничего|никакой|
|a*b*|a*b*|нет|0|ab / 2|
|ab*a*|b*a*|нет|0, 1|ba / 3|
|aba*b*|a*b*|да|без изменений|никакой|
|abab*a*|ab*a*|нет|0, 1, 2|aba / 4|
|ababa*b*|a*b*|да|без изменений|никакой|
|ababab*a*|ab*a*|да|0, 1, 2, 4|никакой|

Тогда при декодировании возникает следующая проблема
|ИНДЕКС|ПРЕОБРАЗОВАНИЕ СЛОВАРЯ|ДЕКОДИРОВАННЫЙ ВЫХОД|ТЕКЩАЯ СТРОКА|НОВАЯ ЗАПИСЬ В СЛОВАРЕ/ИНДЕКС|
|:-----:|:-----:|:-----:|:-----:|:-----:|
|*0*|0 = *а*|*а*|никакая|никакая|
|0, *1*|1 = *b*|a*b*|a|ab / 2|
|0, 1, *2*|2 = *ab*|ab*ab*|b|ba / 3|
|0, 1, 2, *4*|4 = ???|abab???|ab|???|

Таким образом, чтобы алгоритм справился с этим исключением, нужно взять подстроку, которую уже получили, ‘ab’, и конкатенировать ее первый символ с самим собой, ‘ab’ + ‘a’ = “aba” вместо того, чтобы следовать процедуре как обычно.

### 4.2. Псевдокод алгоритма распаковки
```
while ((k = nextcode)) != EOI) //пока не конец информации
{
 if (k == CLC) //k есть код очистки
 {
InitTable(); //заново инициализируем таблицу
k = nextcode; //читаем следующий код
if (k == EOI)
break;
OutString(k); //выводим цепочку для кода k
old = k; //запоминаем текущий код
 }
 else
 {
if (InTable(k)) //в таблице есть строка для кода k
{
OutString(k); //выводим цепочку для кода k
AddString(String(old)+Char(k)); //формируем и добавляем новую цепочку
old = k;
}
else // в таблице нет строки для кода k
{
s = String(old)+Char(old); //формируем цепочку
OutString(s); //выводим цепочку
AddString(s); //и добавляем её в таблицу
old = k;
}
 }
}
```

# 6. Тесты
**! Для запуска автоматического тестирования алгоритма измените значение переменной *auto_tester* с "false" на "true" в строке 76, а также укажите количество тестов и путь к ним в строке 250.**


Ниже приведены тесты, используемые для автоматического тестирования итоговой реализации.
## Тест 1
Проверка на пустоту файла.
## Тест 2
Проверка на отсутствие данных для кодирования.
## Тест 3
Проверка на отсутствие данных для декодирования.
## Тест 4
Проверка на незначащие символы (пробелы, переносы строк).
## Тест 5
Проверка на незначащие символы при декодировании (пробелы, переносы строк).
## Тест 6
Проверка на тип входных данных при декодировании.
## Тест 7
Проверка на существование записи в словаре при начальном заходе в алгоритм декодирования.
## Тест 8
Проверка на запрос encode/decode.
## Тест 9
Проверка на подряд идущие пробелы/переносы строк при декодировании.
## Тест 10-14
Проверка на корректность работы кодирования.
## Тест 15-19
Проверка на корректность работы декодирования

**Входные данные должны содержать строку со значением "encode" для кодирования и "decode" для декодирования, а также текст для кодирования/декодирования с новой строки.**

# Источники
* . Семенюк В. В. Экономное кодирование дискретной информации. –СПб.: СПбГИТМО (ТУ), 2001. – 115 с. - Режим доступа: http://compression.ru/download/articles/rev_univ/semenyuk_2001_econom_encoding.pdf
* Mark R. Nelson «LZW Data Compression» [Электронный ресурс]: электрон. cтатья/ Mark R. Nelson, 1989. – Режим доступа: https://marknelson.us/posts/1989/10/01/lzw-data-compression.html
* Алгоритм LZW [Электронный ресурс]: электрон. cтатья/ информационный ресурс ИТМО, 2022. – Режим доступа: http://neerc.ifmo.ru/mediawiki/index.php?title=Алгоритм_LZW&redirect=no
* Hans Wennborg «Shrink, Reduce, and Implode: The Legacy Zip Compression Methods» [Электронный ресурс]: электрон. cтатья/ Hans Wennborg, 2021. – Режим доступа: https://www.hanshq.net/zip2.html
* «Подстановочные или словарно-ориентированные алгоритмы сжатия информации. Методы Лемпела-Зива» [Электронный ресурс]/ ИНТУИТ, 2007. Режим доступа: https://intuit.ru/studies/courses/2256/140/lecture/3904
* Michael Dipperstein «Lempel-Ziv-Welch (LZW) Encoding Discussion and Implementation» [Электронный ресурс]: электрон. cтатья/ Michael Dipperstein, 2015. Режим доступа: http://michael.dipperstein.com/lzw/
* Ziv J., Lempel A. «Compression of Individual Sequences via Variable-Rate Coding. » / IEEE Trans. Inform. Theory, 1978 – 530-536 с
* Welch T. «A Technique for High-Performance Data Compression. »/ Computer, 1984 – 8-19 с.
* Сэломон Д. «Сжатие данных, изображений и звука.» / М.Техносфера. 2006.
* Пантелеев Е. Р., Алыкова А. Л. «Алгоритмы сжатия данных без потерь»: учебное пособие для вузов / Е. Р. Пантелеев, А. Л. Алыкова. — СанктПетербург: Лань, 2021. – 172 с
