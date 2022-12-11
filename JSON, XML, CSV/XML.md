За работу с XML в Go отвечает пакет `encoding/xml`. Кодирование и декодирование XML работают аналогично JSON, так что ограничусь парой примеров.

#### Кодирование в XML

Человек с адресом и друзьями:

```go
type Address struct {
    Country string `xml:"country,attr"`  // (1)
    City    string `xml:"city,attr"`     // (2)
}

type Person struct {
    XMLName   xml.Name  `xml:"person"`          // (3)
    Name      string    `xml:"name"`
    IsAwesome bool      `xml:"awesome,attr,omitempty"`  // (4)
    Residence *Address  `xml:"residence"`
    Friends   []*Person `xml:"friends>person"`  // (5)
}
```

Теги задают правила преобразования значений в XML. Поскольку это более сложный формат, чем JSON, то и возможностей больше:

-   `Address.Country` и `Address.City` будут кодироваться как атрибуты, а не как элементы ➊ ➋
-   Значения типа `Person` будут записаны внутри элемента `person` ➌
-   `Person.IsAwesome` будет кодироваться как атрибут. Причем значение `false` вовсе не будет записано, только `true` ➍
-   Массив друзей будет записан внутри вложенного элемента `friends` ➎

Кодированием занимаются уже знакомые нам функции `Marshal()` / `MarshalIndent()`:

```go
paris := Address{"France", "Paris"}
emma := Person{Name: "Emma"}
grace := Person{Name: "Grace"}
alice := Person{
    Name:      "Alice",
    IsAwesome: true,
    Residence: &paris,
    Friends:   []*Person{&emma, &grace},
}

b, err := xml.MarshalIndent(alice, "", "    ")
if err != nil {
    panic(err)
}
fmt.Println(string(b))
```

Результат: 

```xml
<person awesome="true">
    <name>Alice</name>
    <residence country="France" city="Paris"></residence>
    <friends>
        <person>
            <name>Emma</name>
            <friends></friends>
        </person>
        <person>
            <name>Grace</name>
            <friends></friends>
        </person>
    </friends>
</person>
```

[песочница](https://go.dev/play/p/mxEXcdMEIAZ)

#### Декодирование из XML

Декодированием, как и с JSON, занимается функция `Unmarshal()`:

```go
var alice Person
err := xml.Unmarshal([]byte(src), &alice)
if err != nil {
    panic(err)
}

fmt.Printf(alice.Name)
if alice.IsAwesome {
    fmt.Printf(" ✓ awesome")
}
fmt.Println()
fmt.Printf("from %s, %s\n", alice.Residence.City, alice.Residence.Country)
fmt.Println("friends:")
for _, person := range alice.Friends {
    fmt.Printf("- %s\n", person.Name)
}
```

Результат:

```no-highlight
Alice ✓ awesome
from Paris, France
friends:
- Emma
- Grace
```

[песочница](https://go.dev/play/p/Ke2BQtY35wZ)
[[JSON, XML, CSV]] 