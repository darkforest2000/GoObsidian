Первая часть goEnv.go выглядит так:  

```go
package main 

import (  
"fmt"  
"runtime"  
) 
``` 
Как вы вскоре увидите, в пакет runtime входят функции и свойства, которые  предоставят нужную информацию. Во второй части кода goEnv.go содержится реализация функции main():  
```go
func main() {  
	fmt.Print("You are using ", runtime.Compiler, " ")  
	fmt.Println("on a", runtime.GOARCH, "machine")  
	fmt.Println("Using Go version", runtime.Version())  
	fmt.Println("Number of CPUs:", runtime.NumCPU())  
	fmt.Println("Number of Goroutines:", runtime.NumGoroutine())  
}
```

Выполнение goEnv.go на компьютере с macOS Mojave с версией Go 1.11.4 приведет к следующим результатам: 

``` bash
$ go run goEnv.go  
You are using gc on a amd64 machine  
Using Go version go1.11.4  
Number of CPUs: 8  
Number of Goroutines: 1
```

На компьютере с Debian Linux и версией Go 1.3.3 эта же программа генерирует  
следующие выходные данные:  

```bash
$ go run goEnv.go  
You are using gc on a amd64 machine  
Using Go version go1.3.3  
Number of CPUs: 1  
Number of Goroutines: 4
```

Однако, чтобы получить по-настоящему полезную информацию о среде Go, мы воспользуемся следующей программой с именем requiredVersion.go. Эта программа сообщает, какую версию Go вы используете — 1.8 или выше:

```go
package main  

import (  
	"fmt"  
	"runtime"  
	"strconv"  
	"strings"  
)  

func main() {  
	myVersion := runtime.Version()  
	major := strings.Split(myVersion, ".")[0][2]  
	minor := strings.Split(myVersion, ".")[1]  
	m1, _ := strconv.Atoi(string(major))  
	m2, _ := strconv.Atoi(minor)  
	if m1 == 1 && m2 < 8 {  
		fmt.Println("Need Go version 1.8 or higher!")  
		return  
	}  
	fmt.Println("You are using Go version 1.8 or higher!")  
}
```

Стандартный Go-пакет strings здесь используется для разбиения строки с версией Go, которую возвращает функция runtime.Version(). Из этой строки мы извлекаем две первые части, а функция strconv.Atoi() используется для преобразования строки в целое число.

Выполнение requiredVersion.go на компьютере с macOS Mojave дает следующий результат:

```bash
$ go run requiredVersion.go  
You are using Go version 1.8 or higher!
```

Если же запустить requiredVersion.go на компьютере с Debian Linux, который  
мы уже использовали ранее в этом разделе, то программа выдаст следующий результат:  
```bash
$ go run requiredVersion.go  
Need Go version 1.8 or higher!
```

[[Внутри Go]]