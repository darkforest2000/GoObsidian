Go позволяет создавать код WebAssembly с помощью инструмента go. Прежде чем показать этот процесс, приведу дополнительную информацию о WebAssembly.

# Краткое введение в WebAssembly

[WebAssembly]  [Wasm] — это машинная модель и исполняемый формат для виртуальной машины, разработанный для повышения скорости и сокращения размера  файла. Это означает, что двоичный файл WebAssembly можно использовать на любой платформе без каких-либо изменений.  

WebAssembly поставляется в двух форматах: простом текстовом и двоичном.  Файлы WebAssembly в простом текстовом формате имеют расширение .wat, а двоичные файлы — .wasm. Обратите внимание, что для загрузки и использования двоичного файла WebAssembly необходимо использовать JavaScript API.  

Кроме Go, код WebAssembly можно генерировать из других языков программирования, которые поддерживают статическую типизацию, таких как Rust, C и C++.

# Почему WebAssembly так важен

WebAssembly важен по следующим причинам:  

	 код WebAssembly работает со скоростью, очень близкой к «родной» скорости машины, то есть быстро;  
	 код WebAssembly можно генерировать из многих языков программирования, в том числе, возможно, и тех, которые вы уже знаете;  
	 большинство современных браузеров изначально поддерживают WebAssembly, не требуя подключения плагина или какого-либо другого программного обеспечения;  
	 код WebAssembly работает намного быстрее, чем код JavaScript.

# Go и WebAssembly

Для Go WebAssembly — это просто еще одна архитектура. Поэтому для создания кода WebAssembly допускается использов ать возможности кросс-компиляции Go.  

Подробнее о возможностях кросс-компиляции Go вы прочитаете в главе 11. А пока обратите внимание на значения переменных среды GOOS и GOARCH, которые используются при компиляции Go-кода в WebAssembly, — именно здесь происходит вся магия.


## Пример

В этом разделе будет показано, как скомпилировать Go-программу в код WebAssembly. Go-код программы toWasm.go выглядит так:  

```go
package main  

import (  
	"fmt"  
)  

func main() {  
	fmt.Println("Creating WebAssembly code from Go!")  
}
```

Здесь важно отметить, что в данном коде нет признаков WebAssembly,  
и toWasm.go можно скомпилировать и выполнить как отдельную программу.  Другими словами, у данной программы нет внешних зависимостей, связанных с WebAssembly.

Последний шаг, который нужно будет сделать для создания кода WebAssembly, — выполнить следующую команду:  

```bash
$ GOOS=js GOARCH=wasm go build -o main.wasm toWasm.go  
$ ls -l  
total 4760  
-rwxr-xr-x 1 mtsouk staff 2430633 Jan 19 21:00 main.wasm  
-rw-r--r--@ 1 mtsouk staff 100 Jan 19 20:53 toWasm.go  
$ file main.wasm  
main.wasm: , created: Thu Oct 25 20:41:08 2007, modified: Fri May 28 13:51:43 2032
```

Как видим, значения GOOS и GOARCH, указанные в первой команде, дают инструкцию Go сгенерировать код WebAssembly. Если не указать правильные значения  GOOS и GOARCH, то при компиляции не будет создан код WebAssembly или же возникнет сбой.

# Использование сгенерированного  кода WebAssembly

Пока мы создали только двоичный файл WebAssembly. Но это еще не все. Нужно сделать еще кое-что, прежде чем мы сможем использовать этот двоичный файл WebAssembly и увидеть его результаты в окне браузера.

	Если вы используете браузер Google Chrome, то можно применить флаг,  который позволяет включить Liftoff — компилятор для WebAssembly, что  теоретически улучшает время выполнения кода WebAssembly. Попробуйте — это не больно! Чтобы активизировать этот флаг, откройте страницу  
	chrome://flags/#enable-webassembly-baseline.

Прежде всего нужно скопировать main.wasm в каталог веб-сервера, а затем выполнить следующую команду:

```bash
$ cp "$(go env GOROOT)/misc/wasm/wasm_exec.js" .
```

Эта команда скопирует файл wasm_exec.js из установочного каталога Go в текущий каталог. Поместите этот файл в тот же каталог веб-сервера, что и файл main.wasm.

Мы не будем приводить здесь код JavaScript, который содержится в wasm_exec.js.  
Представим HTML-код файла index.html:

```html
<HTML>  
<head>  
	<meta charset="utf-8">  
	<title>Go and WebAssembly</title>  
</head>  
<body>  
<script src="wasm_exec.js"></script>  
<script>  
if (!WebAssembly.instantiateStreaming) { // polyfill  
WebAssembly.instantiateStreaming = async (resp, importObject) => {  
const source = await (await resp).arrayBuffer();  
return await WebAssembly.instantiate(source, importObject);  
};  
}  
const go = new Go();  
let mod, inst;  
WebAssembly.instantiateStreaming(fetch("main.wasm"),  
go.importObject).then((result) => {
mod = result.module;  
inst = result.instance;  
document.getElementById("runButton").disabled = false;  
}).catch((err) => {  
console.error(err);  
});  
async function run() {  
console.clear();  
await go.run(inst);  
inst = await WebAssembly.instantiate(mod, go.importObject);  
}  
</script>  
<button onClick="run();" id="runButton" disabled>Run</button>  
</body>  
</HTML>
```

Обратите внимание, что кнопка Run, созданная этим HTML-кодом, не будет активизирована, пока не загрузится код WebAssembly.  
На рис. 2.2 показан результат работы кода WebAssembly, представленный в консоли JavaScript браузера Google Chrome. В других браузерах результаты будут аналогичными.

![[Pasted image 20221210222015.png]]

Однако я считаю, что есть гораздо более простой и удобный способ тестирования приложений WebAssembly — с помощью Node.js. При использовании Node.js  отпадает необходимость в веб-сервере, потому что Node.js — это среда выполнения  JavaScript, построенная на движке Chrome V8 JavaScript.

Если на вашем локальном компьютере установлен Node.js, то вы можете выполнить следующую команду:  

```bash
$ export PATH="$PATH:$(go env GOROOT)/misc/wasm"  
$ GOOS=js GOARCH=wasm go run .  
Creating WebAssembly code from Go!  
```

Вторая команда показывает, правильно ли был выполнен код WebAssembly,  и генерирует желаемое сообщение. Обратите внимание, что первая команда  не является обязательной, она лишь меняет текущее значение переменной среды  PATH на тот каталог, в котором хранятся файлы Go, связанные с WebAssembly.