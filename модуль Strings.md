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

Получение номера телефона из массива чисел

```go
func CreatePhoneNumber(numbers [10]uint) string {
  var test string = strings.Trim(strings.Replace(fmt.Sprint(numbers), " ", "", -1), "[]")
  return fmt.Sprintf("(%s) %s-%s", test[0:3], test[3:6], test[6:10])  
}
```

Количество букв в строке

```go
import s "strings"

func GetCount(str string) (count int) {
  // Enter solution here
  return s.Count(str, "a") + s.Count(str, "e")
}
```


[[Основы GO]]