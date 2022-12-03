не ну это просто БАЗА
```go
package main

import "fmt"

func main() {
    fmt.Println("hello everyone")
}
```

Чтобы запустить программу, [установите Go](https://go.dev/dl/), сохраните код в `hello.go` и запустите из консоли с помощью `go run`. Если пока неохота возиться с установкой — используйте [песочницу](https://go.dev/play/).

```bash
$ go run hello.go
hello everyone
```

Чтобы скомпилировать исходный код в бинарник, воспользуемся `go build`:

```bash
$ go build hello.go
$ ls
hello	hello.go
```

Запустим скомпилированный бинарник:

```bash
$ ./hello
hello everyone
```

### [Значения]

В Go есть целые и дробные числа, логические значения и строки. Вот несколько примеров.

Целые и дробные числа:

```go
fmt.Println("1+1 =", 1+1)
// 1+1 = 2

fmt.Println("5.0/2.0 =", 5.0/2.0)
// 5.0/2.0 = 2.5
```

Логические значения и операторы:

```go
fmt.Println(true && false)
// false

fmt.Println(true || false)
// true

fmt.Println(!true)
// false
```

Строки можно складывать через `+`:

```go
fmt.Println("go" + "lang")
```

Что-то типо f строк
```go
fmt.Printf("%s = %v min", s, d.Minutes())
```

Объявление переменных
```go
var s string
```

Считывание пользовательского ввода
```go
fmt.Scan(&s) // s - то, куда запишется ввод

_, err := fmt.Scan(&s) // можно сделать так и тогда в err попадут все ошибки
fmt.Println(err)

var text, width string
fmt.Scanf("%s %d", &text, &width)
```

Заметь, есть отличие Println и Printf

### [Переменные]

Go статически типизирован. Переменные явно объявляются, типы заранее известны компилятору.

`var` объявляет переменную, `=` инициализирует конкретным значением:

```go
var b bool = true
fmt.Println(b)
// true

var s string = "hello"
fmt.Println(s)
// hello

var i int = 42
fmt.Println(i)
// 42

var f float64 = 12.34
fmt.Println(f)
// 12.34
```

Можно объявить сразу несколько:

```go
var one, two int = 1, 2
fmt.Println(one, two)
// 1 2
```

Если инициализировать переменную при объявлении, тип можно и не указывать. Go сам догадается:

```go
var sunny = true
fmt.Printf("%v is %T\n", sunny, sunny) // \n - просто символ новой строки, без него тоже все работает
// true is bool
```

`fmt.Printf()` подставляет переменные в строку по шаблону. `%v` — формат по умолчанию, `%T` — тип переменной.

Если не инициализировать переменную при объявлении, она получит _нулевое значение_ (zero value). У каждого типа оно свое: int — `0`, string — `""`, bool — `false`.

```go
var num int
var str string
var ok bool

fmt.Printf("%#v %#v %#v\n", num, str, ok) // # - не обязательно в 1 и 3 случае
// 0 "" false
```

Оператор `:=` объявляет и инициализирует переменную. `var` и `тип` можно не указывать:

```go
food := "apple"  // var food string = "apple"
fmt.Println(food)
// apple
```

### [Константы]

Go поддерживает _константы_ для символов, строк, логических и числовых значений.

`const` объявляет константу:

```go
const s string = "constant"
fmt.Println(s)
// constant

const n = 500000000
fmt.Println(n)
// 500000000

const ch = 'a' // это числовой код латинского символа 'a'
fmt.Println(ch)
// 97
```

Если указать выражение — Go рассчитает и сохранит результат. В выражении можно использовать объявленные ранее константы:

```go
const n = 500000000
const d = 3e20 / n
fmt.Println(d)
// 6e+11
```

Тип числовой константы определяется из контекста. Например, когда есть явное приведение типа:

```go
fmt.Println(int64(d))
// 600000000000
```

Или функция, которая требует значение определенного типа (`math.Sin()` ожидает `float64`):

```go
fmt.Println(math.Sin(n))
// -0.28470407323754404
```

Так же как в питоне, в гоу есть библиотека math
Лучше пользоваться функциями оттуда, наджнее.

#### func [Hypot]

func Hypot(p, q [float64]) [float64]

Hypot returns **Sqrt(p^2 + q^2)**, taking care to avoid unnecessary overflow and underflow.

### Циклы

`for` — единственная конструкция для циклов в Go. Вот несколько примеров.

С одним условием, аналог `while` в питоне:

```go
i := 1
for i <= 3 {
    fmt.Println(i)
    i = i + 1
}
// 1
// 2
// 3
```

Классический `for` из трех частей (инициализация, условие, шаг):

```go
for j := 7; j <= 9; j++ {
    fmt.Println(j)
}
// 7
// 8
// 9
```

Бесконечный цикл, выполняется до `break` для выхода из цикла или `return` для выхода из функции:

```go
for {
    fmt.Println("loop")
    break
}
// loop
```

`continue` переходит к следующей итерации цикла:

```go
for n := 0; n <= 5; n++ {
    if n%2 == 0 {
        continue
    }
    fmt.Println(n)
}
// 1
// 3
// 5
```

Задачу с повторением строк возможно решить с помощью функции Repeat из модуля ```
strings :

```go
fmt.Println(strings.Repeat(source,times))
```


### Отличия Sprintf, Printf, Fprint, Scan, Print, Println

[Print] - Выведет переданное значение в терминал, не добавляет после себя новую строку.
```go
fmt.Print("Print")
fmt.Print("Print")
// PrintPrint
```

[Println] - Выведет переданное значение в терминал, добавляет после себя новую строку.
```go
fmt.Println("Print")
fmt.Println("Print")
// Print
// Print
```

[Sprintf] - Вернет строку новую строку. Удобно для создания своих строковых представлений для данных других типов.  

```go
var age = 20
NewAge := fmt.Sprintf("New %v", age)     // "New 20"
```

[Fprint] - Используется для отправки данных в "ридер" - в примере происходит отправка в терминал значения переменной age. Print("123") - это обвертка над Fprint(os.Stdout, "123")  

```go
var age = 20
err := fmt.Fprintf(os.Stdout, age)      // "20"
```

[Printf] - Выводит в терминал форматированную строку. Правила форматирования [тут](https://metanit.com/go/tutorial/8.5.php)  

```go
var age = 20
fmt.Printf("New %v\n", age)    
// "New 20". Не забывайте ставить символ '\n' - перехода на новую строку.
```

[Scan] - Сохраняет введеное строковое значение в переменную  
```go
var name string
Scan(&name)
```
  
При вводе "My name is ThiefPytin"  
Сохранит в переменную name только часть строки подходящей под формат (ThiefPytin)  
```go
var name string
fmt.Scanf("My name is %s", &name)
fmt.Println(name)

// My name is Egor
// Выведет Egor
```

Мы можем пропускать стартовое значение в цикле for
``` go
for ;i < times; i++{
    result = fmt.Sprintf("%s%s", result, source)
}
```

![[Pasted image 20221029153311.png]]

### [If/else]

Конструкция `if-else` в Go ведет себя без особых сюрпризов.

Вокруг условия не нужны круглые скобки, но фигурные скобки для веток обязательны:

```go
if 7%2 == 0 {
    fmt.Println("7 is even")
} else {
    fmt.Println("7 is odd")
}
// 7 is odd
```

Можно использовать `if` без `else`:

```go
if 8%4 == 0 {
    fmt.Println("8 is divisible by 4")
}
// 8 is divisible by 4
```

Единственный нюанс: перед условием может идти выражение. Объявленные в нем переменные доступны во всех ветках:

```go
if num := 9; num < 0 {
    fmt.Println(num, "is negative")
} else if num < 10 {
    fmt.Println(num, "has 1 digit")
} else {
    fmt.Println(num, "has multiple digits")
}
// 9 has 1 digit
```

### [Switch]

`switch` описывает условие с множеством веток.

В отличие от многих других языков, не нужно указывать `break`. Go выполняет только подходящую ветку и не проваливается в следующую:

```go
i := 2
fmt.Print("Write ", i, " as ")
switch i {
case 1:
    fmt.Println("one")
case 2:
    fmt.Println("two")
case 3:
    fmt.Println("three")
}
// Write 2 as two
```

Чтобы провалиться в следующую ветку, есть специальное ключевое слово `fallthrough`. Его можно использовать только в качестве последней инструкции в блоке:

```go
i := 2
fmt.Print("Write ", i, " as ")
switch i {
case 1:
    fmt.Println("one")
case 2:
    fmt.Println("two")
    fallthrough
case 3:
    fmt.Println("Bye-bye")
}
// Write 2 as two
// Bye-bye
```

В одной ветке можно указать несколько выражений. Ветка `default` сработает, если остальные не подошли:

```go
// одной строкой инициализировали day
// и сразу сделали switch по нему
switch day := time.Now().Weekday(); day {
case time.Saturday, time.Sunday:
    fmt.Println(day, "is a weekend")
default:
    fmt.Println(day, "is a weekday")
}
// Tuesday is a weekday
```

Выражения в ветках не обязательно должны быть константами. `switch` может работать как `if`:

```go
t := time.Now()
switch {
case t.Hour() < 12:
    fmt.Println("It's before noon")
default:
    fmt.Println("It's after noon")
}
// It's before noon
```

Несколько условий для одного case 
```go
var lang string
switch code {
case "en":
    lang = "English"
case "fr":
    lang = "French"
case "ru", "rus":
    lang = "Russian"
default:
    lang = "Unknown"
}
```

Создание словаря
```go
ldict := map[interface{}]interface{} {"e": "English","r": "Russian","f": "French"}
```

### goto
**goto**- передает управление в утверждение с соответствующей меткой в той же функции.

Оператор `goto` в Go позволяет выполнить переключение контекста выполнения внутри одной фукнции к указанном оператору.

Учтите, что использование `goto` не поощряется,  так как приводит к запутанности кода.

Утверждение "goto" вне блока не может перейти к метке внутри этого блока. Например, этот пример:

```go
if n%2 == 1 {
  goto L1
}
for n > 0 {
  f()
  n--
L1:
  f()
  n--
}
```

ошибочен, потому что метка L1 находится внутри блока утверждения "for", а goto нет.

```go
package main

import "fmt"

func main() {

   /* local variable definition */
   var a int = 10

   LOOP: fmt.Printf("GOTO here\n")

   /* do loop execution */
   for a < 20 {
      if a == 15 {
         /* skip the iteration */
         a = a + 1
         goto LOOP
      }
      fmt.Printf("value of a: %d\n", a)
      a++
   }
}
```

![[Pasted image 20221029173503.png]]

![[Pasted image 20221029173535.png]]

### Обход коллекции

`range` обходит элементы коллекций. Посмотрим, как использовать его на срезах, картах и строках.

Просуммировать элементы среза (или массива):

```go
nums := []int{2, 3, 4}
sum := 0
for _, num := range nums {
    sum += num
}
fmt.Println("sum:", sum)
// sum: 9
```

`range` на массивах и срезах возвращает индекс и значение для каждого элемента. В примере выше мы не использовали индекс, поэтому заглушили его пустым идентификатором `_`. Но иногда индекс может и пригодиться:

```go
nums := []int{2, 3, 4}
for idx, num := range nums {
    if num == 3 {
        fmt.Println("index:", idx)
    }
}
// index: 1
```

`range` на карте итерирует по записям:

```go
m := map[string]string{"a": "apple", "b": "banana"}
for key, val := range m {
    fmt.Printf("%s -> %s\n", key, val)
}
// a -> apple
// b -> banana
```

Или только по ключам:

```go
m := map[string]string{"a": "apple", "b": "banana"}
for key := range m {
    fmt.Println("key:", key)
}
// key: a
// key: b
```

`range` на строках итерирует по unicode-символам (рунам). Первое значение — порядковый номер байта, с которого начинается руна (руна может занимать несколько байт). Второе значение — числовой код самой руны:

```go
for idx, char := range "ого" {
    fmt.Println(idx, char, string(char))
}
// 0 1086 о
// 2 1075 г
// 4 1086 о
```

## [[Базовые типы]]

![[1.2.types_new_1653911677.png]]

Для начала вспомним [операторы сравнения] (переменных одного типа):

-   `>` — больше;
-   `<` — меньше;
-   `>=` — больше или равно;
-   `<=` — меньше или равно;
-   `==` — равно;
-   `!=` — не равно.

А также [логические операторы]:

-   `&&` — логическое И;
-   `||` — логическое ИЛИ;
-   `!` — логическое НЕ.

Основное условие `switch` может быть не задано явно:

```go
var a int

switch {
case a == 100:
    fmt.Println("EQ 100")
    fallthrough
case a > 0: // выполниться этот вне зависимости от условия
    fmt.Println("GT 0 AND NEQ 100")
case a < 0:
    fmt.Println("LT 0 AND NEQ 100")
} 
```

С циклом `for range` связан ещё один важный момент — операнд `range` копируется во временную переменную, которая уже используется для обхода.

Это также может замедлить выполнение программы для массивов, поэтому следует использовать взятие указателя, чтобы не простаивать:

```go
for _, temp := range &weekTemp {
   temp = 0 
}
```
# Fprint

Cемейство принтов для записи в файл c io.Writer

```go
fmt.Fprintln(), fmt.Fprint(), fmt.Fprintf()
```

