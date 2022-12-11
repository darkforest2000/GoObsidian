JSON, XML, CSVКодирование — это преобразование «родных» структур данных языка в строку (набор байт). В Go для кодирования используют термин «маршалинг» (marshal), а для декодирования — «анмаршалинг» (unmarshaling). Я буду использовать «кодирование» и «декодирование», чтобы не ломать язык.

За кодирование и декодирование JSON в Go отвечает пакет `encoding/json`.

Допустим, есть тип, который описывает человека:

```go
type Person struct {
    Name      string
    Age       int
    Weight    float64
    IsAwesome bool
    secret    string
}
```

Преобразуем объект типа `Person` в JSON:

```go
alice := Person{
    Name:      "Alice",
    Age:       25,
    Weight:    55.5,
    IsAwesome: true,
    secret:    "42",
}

b, err := json.Marshal(alice)
if err != nil {
    panic(err)
}
fmt.Println(string(b))
// {"Name":"Alice","Age":25,"Weight":55.5,"IsAwesome":true}
```

`json.Marshal()` принимает значение произвольного типа `any`, кодирует его в JSON, и возвращает срез байт.

Структура кодируется в JSON-объект. При этом типы полей Go преобразует автоматически:

-   поля типа `string` превращаются в строку;
-   `int` и `float64` — в число;
-   `bool` — в true/false.

Go игнорирует приватные поля структуры — те, что написаны со строчной буквы. Поэтому `secret` на выходе отсутствует.

Чтобы получить красивый JSON с отступами, используют `json.MarshalIndent()`:

```go
b, err := json.MarshalIndent(alice, "", "  ")
if err != nil {
    panic(err)
}
fmt.Println(string(b))
```

```json
{
  "Name": "Alice",
  "Age": 25,
  "Weight": 55.5,
  "IsAwesome": true
}
```

На практике, конечно, в структурах встречаются не только строки, числа и логические значения. Сейчас посмотрим на более сложные случаи.

[песочница](https://go.dev/play/p/gVxI8gP2NU8)

[[JSON, XML, CSV]]