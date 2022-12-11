```go
package main

import (
    "fmt"
    "log"
    "log/syslog"
    "os"
    "path/filepath"
)

func main() {
	/// 1
    programName := filepath.Base(os.Args[0])
    sysLog, err := syslog.New(syslog.LOG_INFO|syslog.LOG_LOCAL7, programName)
    /// 2
    if err != nil {
        log.Fatal(err)
    } else {
        log.SetOutput(sysLog)
    }
    log.Println("LOG_INFO + LOG_LOCAL7: Logging in Go!")
	/// 3
    sysLog, err = syslog.New(syslog.LOG_MAIL, "Some program!")
    if err != nil {
        log.Fatal(err)
    } else {
        log.SetOutput(sysLog)
    }

    log.Println("LOG_MAIL: Logging in Go!")
    fmt.Println("Will you see this?")
}
```

# 1
Первым параметром функции `syslog.New()` является приоритет, который представляет собой соединение средства журналирования и уровня журналирования. Так, приоритет `LOG_NOTICE | LOG_MAIL`, который выбран в качестве примера, будет отправлять сообщения уровня журналирования `NOTICE` в средство журналирования `MAIL`. 

В результате этот код устанавливает такой режим журналирования по умолчанию: средство журналирования `local7` и уровень журналирования `info`. Второй параметр функции `syslog.New()` — это имя процесса, который будет отображаться в журналах в качестве отправителя сообщения. Вообще, рекомендуется использовать настоящее имя исполняемого файла, чтобы впоследствии можно было легко найти нужную информацию в файлах журналов.

# 2
После вызова функции `syslog.New()` нужно проверить переменную типа error, которая возвращается этой функцией, чтобы убедиться, что все в порядке. Если это так, тогда значение переменной типа error равно `nil` и можно вызвать функцию `log.SetOutput()`, которая задает приемник вывода для записи в журнал по умолчанию, — в данном случае это созданный нами ранее регистратор журнала (`sysLog`). Затем можно использовать функцию `log.Println()` для отправки информации на сервер журнала.

# 3
Как видно из последней части кода, мы можем изменять конфигурацию ведения журнала в своих программах столько раз, сколько захотим, и при этом все равно можем использовать `fmt.Println()` для вывода данных на экран. 

При выполнении `logFiles.go` на экран компьютера с Debian Linux будут выведены следующие данные: 
```bash
$ go run logFiles.go 
Broadcast message from systemd-journald@mail (Tue 2017-10-17 20:06:08 EEST): 
logFiles[23688]: Some program![23688]: 2017/10/17 20:06:08 LOG_MAIL: 
Logging in Go! 
Message from syslogd@mail at Oct 17 20:06:08 ... 
Some program![23688]: 2017/10/17 20:06:08 LOG_MAIL: Logging in Go! Will you see this? 
```

Выполнение того же Go-кода на компьютере с macOS High Sierra приведет к следующим результатам: 

```bash
$ go run logFiles.go 
Will you see this?
```

Имейте в виду, что на большинстве UNIX-машин информация журналов хранится в нескольких файлах. Это также относится и к машине с Debian Linux, использованной в этом разделе. В результате logFiles.go отправляет выходные данные в несколько файлов журнала, в чем можно убедиться с помощью следующих команд оболочки: 
```bash
$ grep LOG_MAIL /var/log/mail.log 
Oct 17 20:06:08 mail Some program![23688]: 2017/10/17 20:06:08 LOG_MAIL: Logging in Go! 
$ grep LOG_LOCAL7 /var/log/cisco.log 
Oct 17 20:06:08 mail logFiles[23688]: 2017/10/17 20:06:08 LOG_INFO + LOG_LOCAL7: Logging in Go! 
$ grep LOG_ /var/log/syslog 
Oct 17 20:06:08 mail logFiles[23688]: 2017/10/17 20:06:08 LOG_INFO + LOG_LOCAL7: Logging in Go! 
Oct 17 20:06:08 mail Some program![23688]: 2017/10/17 20:06:08 LOG_MAIL: Logging in Go!
``` 
Как видно из результатов, сообщение оператора `log.Println("LOG_INFO + LOG_ LOCAL7: Logging in Go!")` записано в два файла: `/var/log/cisco.log` и `/var/log/ syslog`, тогда как сообщение оператора `log.Println("LOG_MAIL: Logging in Go!")` попало в` /var/log/syslog` и `/var/log/mail.log`. 

Из этого раздела важно запомнить следующее: если сервер журналирования на UNIX-машине не настроен на перехват всех средств журналирования, то некоторые из отправляемых на него записей журнала могут быть удалены без каких-либо предупреждений.

# Log.Fatal

```go
package main

import (
	"fmt"
	"log"
	"log/syslog"
)

func main() {
	sysLog, err := syslog.New(syslog.LOG_ALERT|syslog.LOG_MAIL, "Some program!")
	if err != nil {
		log.Fatal(err)
	} else {
		log.SetOutput(sysLog)
	}

	log.Fatal(sysLog)
	fmt.Println("Will you see this?")
}
```

```bash
$ go run logFatal.go 
exit status 1
```

Легко понять, что при использовании` log.Fatal()` Go-программа завершается в том месте, где была вызвана функция `log.Fatal()`. Именно поэтому вы не увидели вывод оператора `fmt.Println("Will you see this?")`. 

Однако в соответствии с параметрами вызова `syslog.New()` при этом в файл журнала `/var/log/mail.log`, связанный с почтой, добавлена запись: 

```bash
$ grep "Some program" /var/log/mail.log
Jan 10 21:29:34 iMac Some program![7123]: 2019/01/10 21:29:34 &{17 Some program! iMac.local {0 0} 0xc00000c220}
```

# log.Panic()

```go
package main

import (
	"fmt"
	"log"
	"log/syslog"
)

func main() {
	sysLog, err := syslog.New(syslog.LOG_ALERT|syslog.LOG_MAIL, "Some program!")
	if err != nil {
		log.Fatal(err)
	} else {
		log.SetOutput(sysLog)
	}

	log.Panic(sysLog)
	fmt.Println("Will you see this?")
}
```

```bash
$ go run logPanic.go 
panic: &{17 Some program! iMac.local {0 0} 0xc0000b21e0} 
goroutine 1 [running]: 
log.Panic(0xc00004ef68, 0x1, 0x1) 
     /usr/local/Cellar/go/1.11.4/libexec/src/log/log.go:326 +0xc0 main.main() 
     /Users/mtsouk/Desktop/mGo2nd/Mastering-Go-Second-Edition/ch01/logPanic.go:17 +0xd6 
exit status 2
```

Таким образом, `log.Panic()` выводит дополнительную низкоуровневую информацию, которая, по идее, должна помочь разрешить сложную ситуацию, возникшую в Go-коде. 

Как и `log.Fatal()`, функция `log.Panic()` добавит запись в соответствующий файл журнала и немедленно прекратит работу Go-программы.

# Запись в специальный журнальный файл 

Иногда нужно просто записать журнальные сообщения в выбранный вами файл. Это может понадобиться по многим причинам. Например, для записи данных отладки, которые бывают слишком большими. Или если вы не хотите записывать их в системные журнальные файлы, а собираетесь хранить данные в собственных журналах, отдельно от системных данных, а потом, допустим, передать их куда-то или сохранить в базе данных. Или же вы хотите хранить свои данные в другом формате. В этом подразделе вы узнаете, как организовать запись в специальный журнальный файл. 

Назовем нашу Go-утилиту customLog.go и будем записывать данные в журнальный файл `/tmp/mGo.log`
Первая часть кода:

```go
package main

import (
    "fmt"
    "log"
    "os"
)

var LOGFILE = "/tmp/mGo.log"
```

Путь к журнальному файлу в customLog.go задан жестко как значение глобальной переменной с именем `LOGFILE`. Для наших целей в примере журнальный файл находится в каталоге `/tmp`, что не является обычным местом для хранения данных, поскольку, как правило, каталог `/tmp `очищается после каждой перезагрузки системы. Однако на данном этапе это избавит нас от необходимости запускать customLog.go с полномочиями пользователя root и не потребует размещать лишние файлы в системных каталогах. 

Если позже вы решите использовать код customLog.go в реальном приложении, вам следует изменить путь на более правильный

Вторая часть customLog.go выглядит следующим образом:

```go
func main() {
    f, err := os.OpenFile(LOGFILE, os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0644)

    if err != nil {
        fmt.Println(err)
        return
    }
    defer f.Close()
```

Здесь мы создаем новый журнальный файл, используя функцию `os.OpenFile()` с необходимыми правами доступа к UNIX-файлам (`0644`). Последняя часть customLog.go выглядит следующим образом:

```go
	// LstdFlags = Ldate | Ltime
    // initial values for the standard logger
    iLog := log.New(f, "customLog ", log.LstdFlags)
    iLog.Println("Hello there!")
    iLog.Println("Another log entry!")
}
```

Если вы посмотрите на страницу документации пакета `log`, которую можно найти по адресу https://golang.org/pkg/log/, то увидите, что функция `SetFlags` позволяет устанавливать выходные флаги (варианты) для текущего средства журналирования. 

По умолчанию функция предлагает значения LstdFlags: Ldate и Ltime, то есть в каждой записи журнала, которая записывается в журнальный файл, будут указаны текущая дата и время. При выполнении customLog.go не будет генерироваться вывод на экран. 

Однако если дважды выполнить эту программу, получим следующее содержимое файла /tmp/mGo.log:

```bash
$ go run customLog.go 
$ cat /tmp/mGo.log 
customLog 2019/01/10 18:16:09 Hello there! 
CustomLog 2019/01/10 18:16:09 Another log entry! 
$ go run customLog.go 
$ cat /tmp/mGo.log 
customLog 2019/01/10 18:16:09 Hello there! 
CustomLog 2019/01/10 18:16:09 Another log entry! 
CustomLog 2019/01/10 18:16:17 Hello there! 
CustomLog 2019/01/10 18:16:17 Another log entry!
```


# Вывод номеров строк в записях журнала

В этом разделе вы узнаете, как указать в журнальном файле номер строки исходного файла, который выполнил инструкцию, сделавшую запись в журнал. Для этого рассмотрим Go-программу из файла customLogLineNumber.go, разделив ее на две части. Первая часть выглядит так:

```go
package main

import (
    "fmt"
    "log"
    "os"
)

var LOGFILE = "/tmp/mGo.log"

func main() {
    f, err := os.OpenFile(LOGFILE, os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0644)

    if err != nil {
        fmt.Println(err)
        return
    }
    defer f.Close()

    // LstdFlags = Ldate | Ltime
    // initial values for the standard logger
    iLog := log.New(f, "customLogLineNumber ", log.LstdFlags)

    // Print the line number of when and where something has happened
    iLog.SetFlags(log.LstdFlags | log.Lshortfile)
    iLog.Println("Hello there!")
    iLog.Println("Another log entry!")
}
```

Всю магию выполняет оператор `iLog.SetFlags(log.LstdFlags | log.Lshortfile)`, который, кроме `log.LstdFlags`, также активизирует флаг `log.Lshortfile`. Последний добавляет в строку записи журнала полное имя файла, а также номер строки Go-оператора, который создал эту запись. 

Выполнение customLogLineNumber.go не выводит данные на экран. Но после двух запусков файла customLogLineNumber.go содержимое файла журнала` /tmp/mGo.log` будет примерно таким: 
```bash
$ go run customLogLineNumber.go 
$ cat /tmp/mGo.log 
customLogLineNumber 2019/01/10 18:25:14 customLogLineNumber.go:26: Hello there! 
CustomLogLineNumber 2019/01/10 18:25:14 customLogLineNumber.go:27: Another log entry! 
CustomLogLineNumber 2019/01/10 18:25:23 customLogLineNumber.go:26: Hello there!
CustomLogLineNumber 2019/01/10 18:25:23 customLogLineNumber.go:27: Another log entry! 
CustomLogLineNumber 2019/01/10 18:25:23 customLogLineNumber.go:26: Hello there! 
CustomLogLineNumber 2019/01/10 18:25:23 customLogLineNumber.go:27: Another log entry!
```

Как видим, использование длинных имен для утилит командной строки затрудняет чтение журнальных файлов.

[[Основы GO]] 