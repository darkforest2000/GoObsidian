## Как превратить массив в строку в Golang?

```go
package main

import (
	"fmt"
	"strings"
)

const selectBase = "SELECT * FROM user WHERE %s "

var refStringSlice = []string{
	" FIRST_NAME = 'Jack' ",
	" INSURANCE_NO = 333444555 ",
	" EFFECTIVE_FROM = SYSDATE "}

func main() {

	sentence := strings.Join(refStringSlice, "AND")
	fmt.Printf(selectBase+"\n", sentence)

}
```

## Получение номера телефона из массива чисел

```go
func CreatePhoneNumber(numbers [10]uint) string {
  var test string = strings.Trim(strings.Replace(fmt.Sprint(numbers), " ", "", -1), "[]")
  return fmt.Sprintf("(%s) %s-%s", test[0:3], test[3:6], test[6:10])  
}
```

## Количество букв - текста в строке

```go
import s "strings"

func GetCount(str string) (count int) {
  // Enter solution here
  return s.Count(str, "a") + s.Count(str, "e")
}
```

## Перевод строк в верхний и нижний регистр

```go
ss := "Sammy Shark"
fmt.Println(strings.ToUpper(ss))  
fmt.Println(strings.ToLower(ss))
```


## Функция        Использование

`strings.HasPrefix`      Поиск строки с начала
`strings.HasSuffix`      Поиск строки с конца
`strings.Contains`        Поиск в любом месте строки
`strings.Count`             Счетчик количества появлений строки в тексте


## Разделение строки по пробелам:

```go
balloon := "Sammy has a balloon."
s := strings.Split(balloon, " ")
fmt.Println(s)
//[Sammy has a balloon]
```

Поскольку мы использовали `strings.Println`, очень трудно сказать, какой мы получили результат, посмотрев на вывод. Чтобы убедиться, что мы получили набор строк, используйте функцию `fmt.Printf` с оператором `%q`, чтобы вывести строки:

```go
fmt.Printf("%q", s)
//["Sammy" "has" "a" "balloon."]
```

Еще одной полезной функцией, помимо `strings.Split`, является функция `strings.Fields`. Разница между этими функциями состоит в том, что `strings.Fields` будет игнорировать все пробелы и разделять строку по реальным `полям` в строке:

```go
data := "  username password     email  date"
fields := strings.Fields(data)
fmt.Printf("%q", fields)
//["username" "password" "email" "date"]
```

## Замена слов-текста в строке

```go
//Sammy has a balloon.
fmt.Println(strings.ReplaceAll(balloon, "has", "had"))
//Sammy had a balloon.
```


## func [Map](https://cs.opensource.google/go/go/+/go1.19.3:src/strings/strings.go;l=465) 

Map returns a copy of the string s with all its characters modified according to the mapping function. If mapping returns a negative value, the character is dropped from the string with no replacement.

```go
func main() {
	rot13 := func(r rune) rune {
		switch {
		case r >= 'A' && r <= 'Z':
			return 'A' + (r-'A'+13)%26
		case r >= 'a' && r <= 'z':
			return 'a' + (r-'a'+13)%26
		}
		return r
	}
	fmt.Println(strings.Map(rot13, "'Twas brillig and the slithy gopher..."))
}

// 'Gjnf oevyyvt naq gur fyvgul tbcure...
```


## func [Repeat](https://cs.opensource.google/go/go/+/go1.19.3:src/strings/strings.go;l=528) 

```go
func Repeat(s string, count int) string
```
Repeat returns a new string consisting of count copies of the string s.
It panics if count is negative or if the result of (len(s) * count) overflows.

```go
fmt.Println("ba" + strings.Repeat("na", 2))
```

## func [SplitAfter](https://cs.opensource.google/go/go/+/go1.19.3:src/strings/strings.go;l=320) 

```go
func SplitAfter(s, sep string) []string
```

SplitAfter slices s into all substrings after each instance of sep and returns a slice of those substrings.

If s does not contain sep and sep is not empty, SplitAfter returns a slice of length 1 whose only element is s.

If sep is empty, SplitAfter splits after each UTF-8 sequence. If both s and sep are empty, SplitAfter returns an empty slice.

It is equivalent to SplitAfterN with a count of -1.

```go
fmt.Printf("%q\n", strings.SplitAfter("a,b,c", ","))
//["a," "b," "c"]
```




#### Поиск по строке

```go
s := "go is awesome"

fmt.Println(strings.Contains(s, "go"))
// true

fmt.Println(strings.HasPrefix(s, "go"))
// true

fmt.Println(strings.HasSuffix(s, "some"))
// true

fmt.Println(strings.Index(s, "is"))
// 3

fmt.Println(strings.LastIndex("go no go", "go"))
// 6

fmt.Println(strings.Count("go no go", "go"))
// 2
```

#### Замена

```go
s := "go go go"

fmt.Println(strings.ReplaceAll(s, "go", "no"))
// no no no

fmt.Println(strings.Replace(s, "go", "no", 2))
// no no go
```

`ReplaceAll()` заменяет все вхождения искомой подстроки, а `Replace()` — только указанное количество.

#### Разделить / соединить

```go
fmt.Println(strings.Split("go, is, awesome", ", "))
// [go is awesome]

fmt.Println(strings.Fields("go is awesome"))
// [go is awesome]

before, after, found := strings.Cut("go is awesome", " is ")
fmt.Println(before, after, found)
// go awesome true

fmt.Println(strings.Join([]string{"go", "is", "awesome"}, " "))
// go is awesome
```

`Split()` бьет по указанному разделителю, а `Fields()` — только по последовательностям пробельных символов.

`Cut()` делит строку на две части — до и после указанной подстроки.

#### Обрезка

```go
s := "--=go is awesome=--"

fmt.Println(strings.Trim(s, "-="))
// go is awesome

fmt.Println(strings.TrimLeft(s, "-="))
// go is awesome=--

fmt.Println(strings.TrimRight(s, "-="))
// --=go is awesome

fmt.Println(strings.TrimSpace(" go is awesome \n"))
// go is awesome
```

`Trim()`, `TrimLeft()` и `TrimRight()` вырезают не указанную подстроку, а любые входящие в нее символы.

#### Преобразование регистра

```go
fmt.Println(strings.ToUpper("go is awesome"))
// GO IS AWESOME

fmt.Println(strings.ToLower("GO IS AWESOME"))
// go is awesome
```

#### Повтор

```go
fmt.Println(strings.Repeat("go", 3))
// gogogo
```

 [песочница](https://go.dev/play/p/5uSqWt55B8D)


[[Основы GO]] [[Преобразование типа]] [[Юникод]] [[Строитель строк]] [[Регулярные выражения]] [[Шаблонизатор]]