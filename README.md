В нашем курсе мы больше будем работать с GCC пакетом, т.е. gcc компилятором и g++

## gcc и g++
Часто говорят что это компиляторы, т.к. по больше привыкли так называть, но на самом деле это прогшраммы, которые делают за кулисами следующие действия препроцессинг, компиляция, и линковка.
На самом деле эти две программы выполняют абсолютно одинаковые действия, но разница у них только в процессе линковики.

### Препроцессинг
Препроцессинг это самый первый этам когда мы собираем нашшу программу, на этом этапе происходит раскрытие всех макросов, и `include` деректив, т.е подготавливаеться исходный код(просто текстовый файл) для следующего этапа компиляции.

### Компиляция
Копилятор, он уже преобразует ваш исходный код, или текстовые данные, в язык программирования. Т.е. он на самом деле строит лексические, синтаксическое дерево и т.д., на основе исходного кода, и если что то есть, которое нарушает это построение дерав, т.е. то что не поддерживаеться компилятором или с его точки зрения есть ошибкой,то компилятор выдаст вам ошибку компиляции с небольшим описанием и возсожной рекомендацией по устранению ошибки. Дальше после построения и анализа синтаксически коретного вашего исходного кода, компиятор начинает преробразовывать этот код в машинные инструкции, или объектные файлы. Иногда компиляцию можно преобразовать в асемблерный язык. Например на Unix системах это ELF формат ( Executable and Linkable Format), т.е это некий двоичный файл (иногда называемый двоичным интерфейсом). В этом файле может содержаться таблицы символов, информация о функциях которые не определены в данном моделу трансляции. Но и это также полноценно исполняемый файл.

### Линковка
Задача линковщика очень проста, она связывает все созданные ранее компиятором объектные файлы в один исполняемый файл. Его задача, после компиляции найти определения функций, типов, котрые были вставленные компилятором при процессе компиляции

А теперь в чем же отличие gcc от g++
Компилятор gcc он может компилировать как файлы c/cpp, т.е. в зависимости от расширения, он их будет по разному синтаксически рассматривать. В то время как g++ будет любой файл c/cpp рассматривать как С++ файл. Также что важно g++ линкует стандартную библиотеку С++, в то время как gcc этого не делает. Но это и логично, т.к. gcc при компиляции может компилить как C программу, которой С++ библиотека не нужна.
Скомпилируем простую программу С++ двумя компиляторами
```cpp
#include <iostream>

int func(int a) {
  return a;
}
int main(int argc, char const *argv[]) {
  std::cout << func(0) << std::endl; 
  return 0;
}

```
Сначала скомпилируем ее спомощью gcc
`gcc main.cpp -o main`
Т.е. тут мы вызываем компилятор, передаем на вход ей исходный код(модуль трансляции), и линкеру говорим какое выходное имя исполняемого файла у нас будет
Но если мы запустим такую команду, то получим поток ошибок
```
/tmp/ccp1ruHh.o: In function `main':
main.cpp:(.text+0x28): undefined reference to `std::cout'
main.cpp:(.text+0x2d): undefined reference to `std::ostream::operator<<(int)'
main.cpp:(.text+0x32): undefined reference to `std::basic_ostream<char, std::char_traits<char> >& std::endl<char, std::char_traits<char> >(std::basic_ostream<char, std::char_traits<char> >&)'
main.cpp:(.text+0x3a): undefined reference to `std::ostream::operator<<(std::ostream& (*)(std::ostream&))'
/tmp/ccp1ruHh.o: In function `__static_initialization_and_destruction_0(int, int)':
main.cpp:(.text+0x68): undefined reference to `std::ios_base::Init::Init()'
main.cpp:(.text+0x77): undefined reference to `std::ios_base::Init::~Init()'
collect2: error: ld returned 1 exit status
```

Т.е. как видим линкер пытаеться найти наш объект потока, вызвать функции для вывода на экран числа. Проблема как было сказано в том что нам необходимо сказать компилятору где определены эти функции и объекты. Есть два места где они могут быть определены это в нашей программе и в библиотеке. В нашей программе нет определения наших функций, соответсвенно линкер будет искать это в библиотеке. Но линкеру надо указать что это за библиотека, т.к. на компъютере их может быть нереально много, поэтому мы должны указать в какой именно библиотеке. Но чтобы пойти дальше в двух словах что такое библиотека.
Библиотека это то же исполняемый компилируемый файл идея которого следующея. Например мы написали крутой функционал работы кучи, и этот функционал необходимо использовать в нескольких программах и вместо копирования функционала, для этого пишут библиотеки, которые представляют скоплириуемый совместно используемый код. Но есть два типа библиотек это статическая и динамическая.
Статическая это когда весь код библиотеки встраиваетиься или правильно сказать мапится(вставляеться) в нашу программу. Но есть много минусов этого, это то что код нашей программы становиться больше на размер этой библиотеки, второй если что то поменялось, то необходимо перекомпилировать нашу программу. Но работа с приложения становиться быстрей.
Динамическая. При линковке, линковщих, грубо говоря добавляет так называемые заглушки типа с именами библиотек, и библиотека если она не была загружена в память, то она загружаеться, и необходимое адрессное пространство библиотекимапится в память приложения(виртуальное адрессное пространство). Т.е. вывод что если при запуске приложения не будет найдено библиогтеку, то наша программа не запуститься. Другой вариант если необходимого имени функции нет в библиотеке, то наша программа таже не загрузиться. Приимуществом есть то что нам не надо перекомпилировать программу, если что то поменялось в динамической библиотеке, т.е. для этого просто надо обновить библиотеку и перезапустить нашу программу. Другой тип динамической загрузки это когда вместо нашей ос мы сами загружаем нужную нам библиотеку, когда нам это нужно.

Теперь когда мы разобрались что такое библиотеки, теперь понятно что нужно gcc указать библиотеку где находиться определения этих функций. В данном случае это libstdc++. Ок давайте соберем нашу программу, теперь с другой строкой сборки.
`gcc main.cpp -lstdc++ -o main`
Как видим спомощью `-lstdc++` мы указываем что мы думаем что функции вывода на печать находяться в этой библиотеке. При компиляции нам не нужно указывать, `libstdc++`, т.е. полное имя. Мы указываем опцию `-l` и в имени скипаем `lib`. И наша программа собралась без ошибок.

Ок, теперь запустим на сборку наше приложение, спомощью компилятора `g++`
`g++ main.cpp -o main`

Если запустить данную команду, то у нас все собереться без ошибок, т.е. нам не надо указывать дополнительных библиотек при зборке, т.к. g++ сделает это автоматически, он понимает где искать определения функций и т.д. Но это касаеться только стандартных библиотек, пользовательские необходимо указывать самостоятельно.

Все выше примеры зауска можно задавать без указания имени исполняемого файла, т.е. можно `-o main`  опускать вместо этого компилятор подставит имя по умолчанию. В Linux это a.out.

Т.е. при сборке программы у нас есть два вида ошибок, 1-й это ошибка компиляции, т.е. синтаксическая ошибка, 2-я это ошибка линковки, т.е. когда линковщик не может найти реализацию функции, объекта, и т.д..

Иногда можно услышать такие понятия как компилятор эсть бекэнд часть, а аргументы компилятора это фронэнд часть. Т.е. компилятору на вход подаеться модуль трансляции, и он его преобразет во ствое внутренее преставление, т.е. бекэнд

Следующие команды разбиваються на отдельные этапы.
1. Этап препроцессинга.
`g++ -E main.cpp`
Эта команда раскрывает все макросы и инклуд файлы

2. Этап компиляции.
`g++ -S main.cpp`
Получение ассемблерного кода
`g++ -c main.cpp`
Получение объектного файла `main.o`

3. Следующая команда соберет объектные файлы в испролняемый один файл
`g++  main.o -o main`

Немного слов о еще одном популярном компиялторе это clang, его цель немного другая, т.е. он транслирует исходные коды в байт код, который потом производит оптимизацию и и кодегенерацию. Т.е. clang есть просто частью пакета LLVM.

MinGW минимизированный GCC под Windows.