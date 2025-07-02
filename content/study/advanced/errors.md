---

title: "4.3 ❗ Ошибки и исключения"
date: 2025-07-02
weight: 43
draft: false
------------

## 🚨 Ошибки в Go — часть обычного потока

В Go **нет исключений**, как в других языках. Вместо этого Go использует **значения ошибок** — это простой, но мощный способ обработки проблем.

---

## 📦 Тип `error`

Тип `error` — это встроенный интерфейс:

```go
type error interface {
    Error() string
}
```

Функции возвращают ошибку как **второе значение**:

```go
value, err := someFunc()
if err != nil {
    fmt.Println("Произошла ошибка:", err)
    return
}
```

---

## 🛠 Создание ошибок

Создать простую ошибку можно с помощью `errors.New`:

```go
import "errors"

err := errors.New("что-то пошло не так")
```

Или с форматированием:

```go
import "fmt"

err := fmt.Errorf("ошибка: %v", cause)
```

---

## 📏 Пример с возвратом ошибки

```go
func divide(a, b int) (int, error) {
    if b == 0 {
        return 0, fmt.Errorf("деление на ноль")
    }
    return a / b, nil
}

result, err := divide(10, 0)
if err != nil {
    fmt.Println("Ошибка:", err)
} else {
    fmt.Println("Результат:", result)
}
```

---

## 🔧 Свои типы ошибок

Ты можешь определить собственные типы ошибок:

```go
type MyError struct {
    Code int
    Msg  string
}

func (e MyError) Error() string {
    return fmt.Sprintf("код %d: %s", e.Code, e.Msg)
}
```

Использование:

```go
return MyError{Code: 404, Msg: "Не найдено"}
```

---

## ⛔ Panic и recover (продвинуто)

Иногда ошибки критичны, и программа должна завершиться. Тогда используют `panic`:

```go
panic("фатальная ошибка")
```

`panic` можно перехватить с помощью `recover`, но это используется редко — например, в middleware:

```go
func safeExecute() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("Восстановление после паники:", r)
        }
    }()

    panic("что-то пошло не так")
}
```

---

## 📌 Главное из главы

* Ошибки — обычная часть работы функций в Go.
* Проверяй `err != nil` после каждой операции.
* Используй `fmt.Errorf` и `errors.New` для создания ошибок.
* Определяй свои типы ошибок для сложных случаев.
* `panic/recover` — только для экстренных ситуаций.

---

В следующем разделе ты узнаешь, что такое **параллелизм и каналы**, а так же про **паттерны конкурентности**.
