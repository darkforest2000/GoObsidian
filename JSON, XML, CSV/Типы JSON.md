Добавим человеку дату рождения:

```go
type Person struct {
    Name      string
    BirthDate time.Time
}
```

Проверим, как она закодируется:

```go
date, _ := time.Parse("2006-01-02", "2000-05-25")
alice := Person{
    Name:      "Alice",
    BirthDate: date,
}

b, err := json.Marshal(alice)
fmt.Println(err, string(b))
// <nil> {"Name":"Alice","BirthDate":"2000-05-25T00:00:00Z"}
```

Вообще говоря, для типа `time.Time` нет соответствия в JSON. Несмотря на это, он вполне прилично закодировался — в строку по стандарту RFC 3339 (ISO 8601). Дело в том, что `time.Time` «знает», как преобразовать время в строку. Для этого он реализует интерфейс `Marshaler`:

```go
// интерфейс
type Marshaler interface {
    MarshalJSON() ([]byte, error)
}

// реализация в time.Time
func (t Time) MarshalJSON() ([]byte, error) {
    // ...
    b := make([]byte, 0, len(RFC3339Nano)+2)
    b = append(b, '"')
    b = t.AppendFormat(b, RFC3339Nano)
    b = append(b, '"')
    return b, nil
}
```

`json.Marshal()` для каждого встреченного значения проверяет — реализует ли оно интерфейс `Marshaler`? Если да, преобразует с помощью `MarshalJSON()`. Так что если захотите какое-то хитрое преобразование в самописном типе — достаточно реализовать этот метод.

### Карты и срезы

Срез автоматически превращается в JSON-массив:

```go
nums := []int{1, 3, 5}
b, err := json.Marshal(nums)
fmt.Println(err, string(b))
// <nil> [1,3,5]
```

Карта — в JSON-объект:

```go
m := map[string]int{
    "one":   1,
    "three": 3,
    "five":  5,
}
b, err := json.Marshal(m)
fmt.Println(err, string(b))
// <nil> {"five":5,"one":1,"three":3}
```

### Неподдерживаемые типы

Некоторые типы — например, функции и каналы — вовсе не получится закодировать:

```go
ch := make(chan int)
_, err := json.Marshal(ch)
fmt.Println(err)
// json: unsupported type: chan int

fn := func() int { return 42 }
_, err := json.Marshal(fn)
fmt.Println(err)
// json: unsupported type: func() int
```

[песочница](https://go.dev/play/p/kUwi5CmoIc7)

[[Динамическая информация о типе JSON]]
[[JSON, XML, CSV]]