Декодирование похоже на кодирование, только функция отличается.

Допустим, есть у нас исходный JSON:

```json
{
    "name": "Alice",
    "is_awesome": true,
    "residence": {
        "country": "France",
        "city": "Paris"
    },
    "friends": [
        { "name": "Emma" },
        { "name": "Grace" }
    ]
}
```

Проверим корректность с помощью `json.Valid()`:

```go
src := `{...}`
fmt.Println(json.Valid([]byte(src)))
// true
```

Создадим подходящие типы:

```go
type Address struct {
    Country string
    City    string
}

type Person struct {
    Name      string
    IsAwesome bool `json:"is_awesome"`
    Residence Address
    Friends   []*Person
}
```

И декодируем в них:

```go
src := `{...}`

var alice Person
err := json.Unmarshal([]byte(src), &alice)

fmt.Println(err, alice)
// <nil> {Alice true {France Paris} [0x1400010c0a0 0x1400010c0f0]}

fmt.Println(alice.Friends[0])
// &{Emma { } []}

fmt.Println(alice.Friends[1])
// &{Grace { } []}
```

`json.Unmarshal()` принимает срез байт с исходным JSON и ссылку на переменную, в которую поместит результат. Как видите, поля-значения `Name`, `IsAwesome` и `Residence` заполнились подходящими значениями, а срез `Friends` — указателями на объекты `Person`.

Хотя в исходном JSON названия полей в нижнем регистре, нам не пришлось прописывать теги — за исключением `IsAwesome`, которое отличается сильнее. `json.Unmarshal()` достаточно умна, чтобы сопоставить поля и так.

[песочница](https://go.dev/play/p/vOr9Q2ArdlC)

#### Определяемые типы

Чтобы задать собственную логику декодирования из JSON, тип должен реализовать интерфейс `json.Unmarshaler`:

```go
type Unmarshaler interface {
    UnmarshalJSON([]byte) error
}
```

Допустим, мы решили сделать тип `AncientNumber`, который записывает число как строку по заветам мудрых предков:

```go
var n AncientNumber

err := json.Unmarshal([]byte("1"), &n)
fmt.Println(err, n)
// <nil> one

err = json.Unmarshal([]byte("2"), &n)
fmt.Println(err, n)
// <nil> two

err = json.Unmarshal([]byte("42"), &n)
fmt.Println(err, n)
// <nil> many

err = json.Unmarshal([]byte("-1"), &n)
fmt.Println(err, n)
// <nil> impossible!
```

Логика декодирования может быть такой: 

```go
type AncientNumber string

func (an *AncientNumber) UnmarshalJSON(data []byte) error {
    // Go рекомендует игнорировать значения null
    if string(data) == "null" {
        return nil
    }
    // декодируем исходное число
    var n int
    err := json.Unmarshal(data, &n)
    if err != nil {
        return err
    }
    // преобразуем в значение типа AncientNumber
    switch {
    case n <= 0:
        *an = "impossible!"
    case n == 1:
        *an = "one"
    case n == 2:
        *an = "two"
    case n > 2:
        *an = "many"
    }
    return nil
}
```

[песочница](https://go.dev/play/p/BXP3yNEXHlU)
[[JSON, XML, CSV]]