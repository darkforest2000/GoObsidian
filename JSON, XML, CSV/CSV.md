CSV — более примитивный формат, чем JSON или тем более XML. Единственные значения, которые Go умеет кодировать и декодировать в CSV — это срезы строк. Преобразованием более сложных типов в строки приходится заниматься самостоятельно.

Разберемся на примере.

#### Кодирование в CSV

Человек, который умеет превратить себя в срез строк:

```go
type Person struct {
    Name      string
    Age       int
    IsAwesome bool
}

func (p Person) Slice() []string {
    return []string{p.Name, strconv.Itoa(p.Age), strconv.FormatBool(p.IsAwesome)}
}
```

Создадим список людей и закодируем их в CSV с помощью `csv.Writer`:

```go
people := []Person{
    {"Alice", 25, true},
    {"Emma", 23, false},
    {"Grace", 27, false},
}

w := csv.NewWriter(os.Stdout)
for _, person := range people {
    err := w.Write(person.Slice())
    if err != nil {
        panic(err)
    }
}
w.Flush()    // (1)

if err := w.Error(); err != nil {  // (2)
    panic(err)
}
```

`csv.Writer` похож на знакомый нам по предыдущему уроку `bufio.Writer`. Разве что по какой-то загадочной причине метод `Flush()` не возвращает ошибку ➊. Ее приходится проверять через отдельный метод `Error()` ➋

Результат:

```no-highlight
Alice,25,true
Emma,23,false
Grace,27,false
```

[песочница](https://go.dev/play/p/9bftjzDCezr)

#### Декодирование из CSV

Тут придется написать конструктор, который создает человека из среза строк:

```go
func MakePerson(record []string) (Person, error) {
    if len(record) != 3 {
        return Person{}, fmt.Errorf("invalid person slice: %v", record)
    }
    name := record[0]
    age, err := strconv.Atoi(record[1])
    if err != nil {
        return Person{}, fmt.Errorf("invalid Age: %v", record[1])
    }
    isAwesome, err := strconv.ParseBool(record[2])
    if err != nil {
        return Person{}, fmt.Errorf("invalid IsAwesome: %v", record[2])
    }
    return Person{name, age, isAwesome}, nil
}
```

Затем прочитать людей из CSV с помощью `csv.Reader`:

```go
src := `...`
r := csv.NewReader(strings.NewReader(src))

var people []Person
for {
    record, err := r.Read()
    if err == io.EOF {
        break
    }
    if err != nil {
        panic(err)
    }
    person, err := MakePerson(record)
    if err != nil {
        panic(err)
    }
    people = append(people, person)
}

fmt.Println(people)
```

Результат:

```http
[{Alice 25 true} {Emma 23 false} {Grace 27 false}]
```

CSV-разделитель настраивается с помощью свойства `Comma` у `csv.Reader` и `csv.Writer`.

[песочница](https://go.dev/play/p/7pi2oZ73SQq)
[[JSON, XML, CSV]]