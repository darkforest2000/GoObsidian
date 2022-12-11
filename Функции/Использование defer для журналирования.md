Рассмотрим способ применения defer, связанный с журналированием. Цель этого метода — упростить журналирование информации о выполнении функции, сделать ее более заметной. Go-программа, в которой показано использование ключевого слова defer для журналирования, называется logDefer.go. Исследуя ее, мы раз-  
делим эту программу на три части

Первая часть logDefer.go выглядит так:  

```go
package main  

import (  
	"fmt"  
	"log"  
	"os"  
)  

var LOGFILE = "/tmp/mGo.log"  

func one(aLog *log.Logger) {  
	aLog.Println("-- FUNCTION one ------")  
	defer aLog.Println("-- FUNCTION one ------")  
	for i := 0; i < 10; i++ {  
		aLog.Println(i)  
	}  
}
```

В функции one() ключевое слово defer используется для того, чтобы гарантировать, что второй вызов aLog.Println() будет выполнен непосредственно перед завершением работы функции one(). Поэтому все журнальные сообщения от функции one() будут размещаться между открывающим и закрывающим вызовами aLog.Println(). Так намного легче обнаружить журнальные сообщения данной функции в журнальных файлах. Вторая часть logDefer.go выглядит так:  

```go
func two(aLog *log.Logger) {  
	aLog.Println("---- FUNCTION two")  
	defer aLog.Println("FUNCTION two ------")  
	for i := 10; i > 0; i-- {  
		aLog.Println(i)  
	}  
}  
```

Для удобства группировки журнальных сообщений функции two() также применяется функция defer. Однако в функции two() используются сообщения немного отличающиеся от тех, что выводятся функцией one(). Пользователь сам выбирает формат сообщений.  

Последняя часть logDefer.go содержит следующий Go-код:

```go
func main() {  
	f, err := os.OpenFile(LOGFILE, os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0644)  
	if err != nil {  
		fmt.Println(err)  
		return  
	}  
	defer f.Close()  
	iLog := log.New(f, "logDefer ", log.LstdFlags)  
	iLog.Println("Hello there!")  
	iLog.Println("Another log entry!")  
	one(iLog)  
	two(iLog)
}
```

При выполнении logDefer.go ничего не будет выводиться на экран. Однако просмотр содержимого /tmp/mGo.log — журнального файла, который используется программой, — покажет, насколько удобно в данном случае применять defer:

```bash
$ cat /tmp/mGo.log  
logDefer 2019/01/19 21:15:11 Hello there!  
logDefer 2019/01/19 21:15:11 Another log entry!  
logDefer 2019/01/19 21:15:11 -- FUNCTION one ------  
logDefer 2019/01/19 21:15:11 0  
logDefer 2019/01/19 21:15:11 1  
logDefer 2019/01/19 21:15:11 2  
logDefer 2019/01/19 21:15:11 3  
logDefer 2019/01/19 21:15:11 4  
logDefer 2019/01/19 21:15:11 5  
logDefer 2019/01/19 21:15:11 6  
logDefer 2019/01/19 21:15:11 7  
logDefer 2019/01/19 21:15:11 8  
logDefer 2019/01/19 21:15:11 9  
logDefer 2019/01/19 21:15:11 -- FUNCTION one ------  
logDefer 2019/01/19 21:15:11 ---- FUNCTION two  
logDefer 2019/01/19 21:15:11 10  
logDefer 2019/01/19 21:15:11 9  
logDefer 2019/01/19 21:15:11 8  
logDefer 2019/01/19 21:15:11 7  
logDefer 2019/01/19 21:15:11 6  
logDefer 2019/01/19 21:15:11 5  
logDefer 2019/01/19 21:15:11 4  
logDefer 2019/01/19 21:15:11 3  
logDefer 2019/01/19 21:15:11 2  
logDefer 2019/01/19 21:15:11 1  
logDefer 2019/01/19 21:15:11 FUNCTION two ------
```

[[Функции]] [[Оператор отложенного вызова]]