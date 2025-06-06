---
title: "2.4 🛠 Функции"
date: 2025-05-30
weight: 14
draft: false
---

После изучения переменных и управляющих структур пора познакомиться с **функциями** — ключевым элементом любой программы на Go. Функции позволяют **упаковывать код** в многократно используемые блоки, делая программы более организованными и читаемыми.

> 💬 **Что такое функция?**  
> Функция — это именованный блок кода, который выполняет определённую задачу. Например, ты можешь создать функцию для вычисления суммы двух чисел или для вывода приветствия.

---

## 🔧 Объявление функции

Функции в Go объявляются с помощью ключевого слова `func`. Вот базовый синтаксис:

```go
package main

import "fmt"

func имяФункции(параметры) возвращаемыйТип {
    // код функции
}
```

Пример:

```go
package main

import "fmt"

func sayHello() {
    fmt.Println("Привет, Go!")
}

func main() {
    sayHello() // Вызов функции
}
```

> 💡 Функция `main` — это точка входа в программу. Она вызывается автоматически при запуске.

---

## 📥 Параметры функции

Функции могут принимать **параметры** — данные, с которыми они работают. Параметры указываются в скобках с их типами.

```go
package main

import "fmt"

func add(a int, b int) {
    fmt.Println("Сумма:", a+b)
}

func main() {
    add(3, 5) // Сумма: 8
}
```

Если параметры одного типа, можно указать тип только для последнего:

```go
func add(a, b int) {
    fmt.Println("Сумма:", a+b)
}
```

---

## 📤 Возвращаемые значения

Функции могут **возвращать значения** с помощью ключевого слова `return`. Тип возвращаемого значения указывается после параметров.

```go
package main

import "fmt"

func multiply(x, y int) int {
    return x * y
}

func main() {
    result := multiply(4, 5)
    fmt.Println("Результат:", result) // Результат: 20
}
```

---

## 🔄 Множественный возврат

Go позволяет возвращать **несколько значений** из функции, что очень удобно, например, для обработки ошибок.

```go
package main

import "fmt"

func divide(a, b int) (int, bool) {
    if b == 0 {
        return 0, false // Возвращаем 0 и false, если деление невозможно
    }
    return a / b, true
}

func main() {
    result, success := divide(10, 2)
    if success {
        fmt.Println("Результат деления:", result) // Результат деления: 5
    } else {
        fmt.Println("Деление на ноль!")
    }
}
```

> ⚠️ При получении нескольких значений нужно указать переменные для всех возвращаемых значений или использовать `_` для игнорирования ненужных.

---

## 📋 Именованные возвращаемые значения

Вы можете назвать возвращаемые значения, чтобы сделать код яснее. Они автоматически инициализируются нулями для своего типа.

```go
package main

import "fmt"

func subtract(a, b int) (result int) {
    result = a - b
    return // Автоматически возвращает result
}

func main() {
    fmt.Println(subtract(7, 3)) // 4
}
```

---

## 🛠 Практическое использование функций

Функции помогают:
- **Упрощать код**: Разбивайте сложную логику на маленькие функции.
- **Повторно использовать код**: Например, функцию `sayHello` можно вызывать многократно.
- **Улучшать читаемость**: Имена функций вроде `add` или `divide` говорят сами за себя.

Пример комбинированной программы:

```go
package main

import "fmt"

func greet(name string) string {
    return "Привет, " + name + "!"
}

func sumAndDiff(a, b int) (sum, diff int) {
    sum = a + b
    diff = a - b
    return
}

func main() {
    fmt.Println(greet("Алиса")) // Привет, Алиса!
    
    s, d := sumAndDiff(10, 4)
    fmt.Println("Сумма:", s) // Сумма: 14
    fmt.Println("Разность:", d) // Разность: 6
}
```

---

## 📚 Полезные советы

- **Имена функций**: Используйте понятные имена, описывающие действие (например, `calculateArea`, а не `ca`).
- **Короткие функции**: Старайтесь, чтобы функции были небольшими и выполняли одну задачу.
- **Ошибки**: Используйте множественный возврат для обработки ошибок, как в примере с `divide`.
- **Документация**: Добавляйте комментарии перед функцией, чтобы объяснить её назначение.

> 🧠 Функции — это как инструменты в ящике: правильно подобранная функция делает код проще и эффективнее.

---

## 🧪 Пример программы

```go
package main

import "fmt"

func max(a, b int) int {
    if a > b {
        return a
    }
    return b
}

func isEven(num int) (result bool) {
    result = num%2 == 0
    return
}

func main() {
    x, y := 15, 8
    fmt.Println("Максимум:", max(x, y)) // Максимум: 15
    
    if isEven(x) {
        fmt.Println(x, "— чётное")
    } else {
        fmt.Println(x, "— нечётное") // 15 — нечётное
    }
    
    result, ok := divide(10, 0)
    if ok {
        fmt.Println("Деление:", result)
    } else {
        fmt.Println("Ошибка: деление на ноль")
    }
}

func divide(a, b int) (int, bool) {
    if b == 0 {
        return 0, false
    }
    return a / b, true
}
```

---

## 🔍 Вопросы для самопроверки

1. Как объявить функцию в Go?
2. Что делает ключевое слово `return`?
3. Как вернуть несколько значений из функции?
4. Что такое именованные возвращаемые значения?
5. Как вызвать функцию `greet` с параметром `"Боб"`?
6. Почему полезно использовать функции в программах?

---

## 📌 Главное из главы

- Функции в Go объявляются с помощью `func` и могут принимать параметры и возвращать значения.
- Go поддерживает **множественный возврат**, что удобно для обработки ошибок.
- Именованные возвращаемые значения упрощают код.
- Функции делают код **модульным**, **читаемым** и **повторно используемым**.
- Используйте понятные имена и держите функции короткими для лучшей читаемости.

---

## 🛠 Упражнение

Напишите функцию `square`, которая принимает число `n` и возвращает его квадрат. Вызовите её в `main` и выведите результат для `n = 6`.