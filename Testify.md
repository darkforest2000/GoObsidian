 [testify](https://github.com/stretchr/testify) — это швейцарский нож в мире тестирования Go-кода. Поэтому поговорим про часто используемые пакеты из этого репозитория отдельно.
 
-   [assert](https://pkg.go.dev/github.com/stretchr/testify/assert){target="_blank"} — пакет, в котором собрано множество удобных функций с говорящими названиями: вроде `assert.Equal` или `assert.Nil`. Предназначен для использования внутри обычных Go-тестов вида `func TestXxx(t *testing.T)` (про необычные поговорим, когда доберёмся до [suite](https://pkg.go.dev/github.com/stretchr/testify/suite)).
-   [require](https://pkg.go.dev/github.com/stretchr/testify/require) — то же самое, что [assert](https://pkg.go.dev/github.com/stretchr/testify/assert), но в случае, если проверки из этого пакета падают, выполнение теста останавливается. То есть внутри используется `Fatal` вместо `Error`.
-   [suite](https://pkg.go.dev/github.com/stretchr/testify/suite) — пакет, который вводит концепцию тест-сьюта (**test suite**). Если вы работали с тестами на Java или Python, то, скорее всего, она вам знакома.

**Сьют** — это объект, содержащий сами тесты в виде методов, а также некоторый набор переменных, доступный всем тестам. Кроме того, сьют даёт возможность определить код, который будет вызван перед и/или после исполнения теста. Эта функция полезна, например, для очистки базы данных между тестами (в случае интеграционных тестов).

Типичный пример работы со [suite](https://pkg.go.dev/github.com/stretchr/testify/suite) из официальной документации выглядит так:

```go
import (
    "testing"
    "github.com/stretchr/testify/suite"
)

// ExampleTestSuite — это тестовый сьют, который создан путём эмбеддинга suite.Suite.
type ExampleTestSuite struct {
    suite.Suite
    VariableThatShouldStartAtFive int
}

// SetupTest заполняет переменную VariableThatShouldStartAtFive перед началом теста.
func (suite *ExampleTestSuite) SetupTest() {
    suite.VariableThatShouldStartAtFive = 5
}

func (suite *ExampleTestSuite) TestExample() { // все тесты должны начинаться со слова Test
    suite.Equal(5, suite.VariableThatShouldStartAtFive)
}

func TestExampleTestSuite(t *testing.T) {
    // чтобы go test смог запустить сьют, нужно создать обычную тестовую функцию
    // и вызвать в ней suite.Run
    suite.Run(t, new(ExampleTestSuite))
} 
```

## Паттерны тестирования

Есть несколько рекомендаций, как стоит и не стоит писать тесты. Они не специфичны для Go, поэтому их можно применить для тестирования на любом языке.

### Каждый тест должен тестировать что-то одно

Например, если вы тестируете функцию `func Divide(a, b int) (int, error)`, не стоит писать такой код:

```go
func TestDivision(t *testing.T) {
    result, err := Divide(0, 1)
    require.NoError(t, err)
    assert.Equal(t, 0, result)

    result, err = Divide(4, 2)
    require.NoError(t, err)
    assert.Equal(t, 2, result)

    _, err = Divide(1, 0)
    require.Error(err)
} 
```

Он будет падать при любой проблеме в тестируемой функции. Лучше разбить его на три теста (или три подтеста внутри этого теста), каждый из которых тестирует свой специфический сценарий:

```go
func TestDivision(t *testing.T) {
    t.Run("ZeroNumerator", func(t *testing.T) {
        result, err := Divide(0, 1)
        require.NoError(t, err)
        assert.Equal(t, 0, result)
    })

    t.Run("BothNonZero", func(t *testing.T) {
        result, err = Divide(4, 2)
        require.NoError(t, err)
        assert.Equal(t, 2, result)
    })

    t.Run("ZeroDenominator", func(t *testing.T) {
        _, err = Divide(1, 0)
        require.Error(err)
    })
} 
```

### Тесты не должны зависеть друг от друга

Если один тест опирается на глобальное состояние, устанавливаемое другими тестами, — это проблема. Тестовая архитектура такого типа приводит к неприятным ошибкам, когда локально тесты проходят, а на CI/CD иногда падают, потому что время от времени порядок выполнения тестов меняется.

Эта же рекомендация относится к тестам внутри одного сьюта. При написании каждого теста нужно исходить из того, что ему на вход передаётся состояние после вызова подготовительного метода `SetupTest`.

### Результат работы теста — это не лог

При написании теста стоит считать, что при успешном прохождении теста никто на его логи смотреть не будет. Все контракты, которые фиксирует тест, должны быть прописаны в виде проверок. В этом случае можно безопасно гонять тесты на CI/CD без участия человека.

## Table-driven-тесты

В примерах выше мы видели, что тесты содержат много повторяющегося кода, и это неспроста. Существует паттерн тестирования, который называется table-driven, или, говоря по-русски, [[Табличные тесты]]. Он реализуется не только в Go, но и в других языках программирования.

Суть паттерна в отделении тестовых данных от выполнения самих тестов. Для коротких тестов паттерн может показаться избыточным, однако он позволяет эффективно организовать и расширять тесты, если тестовых наборов требуется несколько.

В Go есть пакет генерации шаблонов тестов именно в стиле table-driven, поэтому для рассмотрения примера будем использовать генерацию кода.

---

Если вы используете GoLand или VS Code с плагинами, то инструменты генерации уже встроены в IDE. Если нет, то потребуется немного дополнительных действий.

Установите пакет `github.com/cweill/gotests`:

```go
$ go get -u github.com/cweill/gotests/... 
```

---

Вернёмся к нашей функции `Add`. Сгенерируем для неё заготовку теста:

```go
gotests -only Add . 
```

В файл `math_test.go` добавился следующий код. Мы добавили к нему комментарии, чтобы было понятнее, что он делает.

```go
func TestAdd(t *testing.T) {
    // args — описывает аргументы тестируемой функции
    type args struct {
        a int
        b int
    }
    // описывает структуру тестовых данных и сами тесты
    tests := []struct {
        name    string // название теста
        args    args // аргументы 
        want    int // ожидаемое значение
        wantErr bool // должна ли функция вернуть ошибку 
    }{
        // TODO: Add test cases.
        // сюда добавляем тестовые случаи (testcases)
    }
    // вызываем тестируемую функцию для каждого тестового случая  
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := Add(tt.args.a, tt.args.b)
            if (err != nil) != tt.wantErr {
                t.Errorf("Add() error = %v, wantErr %v", err, tt.wantErr)
                return
            }
            if got != tt.want {
                t.Errorf("Add() = %v, want %v", got, tt.want)
            }
        })
    }
}
 
```

Добавим тестовые случаи в место, указанное в `//TODO`:

```go
        {
            name: "Test Positive",
            args: args{
                a: 1,
                b: 2,
            },
            want:    3,
            wantErr: false,
        },
        {
            name: "Test Negative 1",
            args: args{
                a: -1,
                b: 2,
            },
            want:    0,
            wantErr: true,
        },
        {
            name: "Test Negative 2",
            args: args{
                a: 1,
                b: -2,
            },
            want:    0,
            wantErr: true,
        },

        {
            name: "Test Negative all",
            args: args{
                a: -1,
                b: -2,
            },
            want:    0,
            wantErr: true,
        }, 
```

И запустим тесты.

Удобство такого подхода в том, что не требуется добавлять вызов и проверки, чтобы добавить ещё несколько случаев. Достаточно описать аргументы и желаемое поведение.