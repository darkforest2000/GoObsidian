```go
func main() {  
	var f *os.File  
	f = os.Stdin  
	defer f.Close()  
	scanner := bufio.NewScanner(f)  
	for scanner.Scan() {  
		fmt.Println(">", scanner.Text())  
	}
}
```

Во-первых, здесь вызывается функция bufio.NewScanner(), которой в качестве параметра передается стандартный поток ввода (os.Stdin). Этот вызов возвращает переменную bufio.Scanner, которая используется функцией Scan() для построчного чтения из os.Stdin. Каждая прочитанная строка выводится на экран, после чего считывается следующая строка. Обратите внимание, что каждая строка, которую выводит программа, начинается с символа >.  
Запуск stdIN.go даст нам следующий вывод

```bash
$ go run stdIN.go  
This is number 21  
> This is number 21  
This is Mihalis  
> This is Mihalis  
Hello Go!  
> Hello Go!  
Press Control + D on a new line to end this program!  
> Press Control + D on a new line to end this program!
```