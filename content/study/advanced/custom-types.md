---
title: "5.4 🎭 Кастомные типы"
description: "Кастомные типы в Go: создание новых типов, методы типов, type assertions, безопасность типов."
keywords: ["кастомные типы go", "новые типы golang", "методы типов go", "type golang"]
date: 2025-08-14
weight: 54
draft: true
---

**Кастомные типы** в Go позволяют создавать собственные типы данных на основе существующих. Это мощный инструмент для создания более выразительного, безопасного и читаемого кода. Вместо использования простых `int` или `string`, ты можешь создать `UserID` или `Email` — и компилятор поможет избежать ошибок.

> 💬 **Зачем нужны кастомные типы?**  
> Представь функцию `TransferMoney(from, to, amount)`. Что если передать аргументы в неправильном порядке? С кастомными типами `UserID` и `Money` такая ошибка станет невозможной!

---

## 🎯 Основы кастомных типов

### Создание простого типа

```go
package main

import "fmt"

// Создаем новый тип на основе int
type UserID int
type ProductID int

// Создаем тип на основе string
type Email string
type PhoneNumber string

func main() {
    var userID UserID = 123
    var productID ProductID = 456
    var email Email = "user@example.com"
    
    fmt.Printf("User ID: %d\n", userID)
    fmt.Printf("Product ID: %d\n", productID)
    fmt.Printf("Email: %s\n", email)
    
    // Нельзя напрямую присвоить разные типы!
    // userID = productID // Ошибка компиляции
    
    // Нужно явное преобразование
    userID = UserID(productID)
    fmt.Printf("Converted: %d\n", userID)
}
```

---

## 🏗️ Методы для кастомных типов

```go
package main

import (
    "fmt"
    "strings"
)

type Email string

// Метод для валидации email
func (e Email) IsValid() bool {
    return strings.Contains(string(e), "@") && strings.Contains(string(e), ".")
}

// Метод для получения домена
func (e Email) GetDomain() string {
    parts := strings.Split(string(e), "@")
    if len(parts) != 2 {
        return ""
    }
    return parts[1]
}

// Метод для приведения к нижнему регистру
func (e Email) Normalize() Email {
    return Email(strings.ToLower(string(e)))
}

type Temperature float64

// Константы для единиц измерения
const (
    Celsius Temperature = iota
    Fahrenheit
    Kelvin
)

// Метод для конвертации в Fahrenheit
func (t Temperature) ToFahrenheit() Temperature {
    return t*9/5 + 32
}

// Метод для конвертации в Celsius
func (t Temperature) ToCelsius() Temperature {
    return (t - 32) * 5 / 9
}

// Метод для красивого вывода
func (t Temperature) String() string {
    return fmt.Sprintf("%.1f°", float64(t))
}

func main() {
    // Работа с Email
    email := Email("USER@Example.COM")
    fmt.Printf("Email: %s\n", email)
    fmt.Printf("Is valid: %t\n", email.IsValid())
    fmt.Printf("Domain: %s\n", email.GetDomain())
    fmt.Printf("Normalized: %s\n", email.Normalize())
    
    // Работа с Temperature
    celsius := Temperature(25)
    fahrenheit := celsius.ToFahrenheit()
    
    fmt.Printf("Celsius: %s\n", celsius)
    fmt.Printf("Fahrenheit: %s\n", fahrenheit)
}
```

---

## 🛡️ Типы для безопасности

```go
package main

import (
    "errors"
    "fmt"
    "time"
)

// Типы для денежных операций
type Money int64      // в копейках
type Currency string

const (
    USD Currency = "USD"
    EUR Currency = "EUR"
    RUB Currency = "RUB"
)

type Amount struct {
    Value    Money
    Currency Currency
}

// Методы для Money
func (m Money) Dollars() float64 {
    return float64(m) / 100
}

func (m Money) String() string {
    return fmt.Sprintf("$%.2f", m.Dollars())
}

// Методы for Amount
func (a Amount) String() string {
    return fmt.Sprintf("%.2f %s", float64(a.Value)/100, a.Currency)
}

func (a Amount) Add(other Amount) (Amount, error) {
    if a.Currency != other.Currency {
        return Amount{}, errors.New("cannot add different currencies")
    }
    return Amount{
        Value:    a.Value + other.Value,
        Currency: a.Currency,
    }, nil
}

// Типы для идентификаторов
type UserID int64
type OrderID int64
type ProductID int64

// Структура заказа
type Order struct {
    ID       OrderID
    UserID   UserID
    Products []ProductID
    Total    Amount
    Created  time.Time
}

// Безопасная функция перевода денег
func TransferMoney(from, to UserID, amount Amount) error {
    if from == to {
        return errors.New("cannot transfer to the same user")
    }
    if amount.Value <= 0 {
        return errors.New("amount must be positive")
    }
    
    fmt.Printf("Transferring %s from user %d to user %d\n", amount, from, to)
    return nil
}

func main() {
    // Создаем суммы
    price1 := Amount{Value: Money(1250), Currency: USD} // $12.50
    price2 := Amount{Value: Money(750), Currency: USD}  // $7.50
    
    fmt.Printf("Price 1: %s\n", price1)
    fmt.Printf("Price 2: %s\n", price2)
    
    // Складываем суммы
    total, err := price1.Add(price2)
    if err != nil {
        fmt.Printf("Error: %v\n", err)
    } else {
        fmt.Printf("Total: %s\n", total)
    }
    
    // Безопасный перевод
    userA := UserID(100)
    userB := UserID(200)
    
    err = TransferMoney(userA, userB, total)
    if err != nil {
        fmt.Printf("Transfer error: %v\n", err)
    }
    
    // Эта ошибка будет поймана компилятором:
    // err = TransferMoney(OrderID(123), userB, total) // Ошибка!
}
```

---

## 🔢 Enum-подобные типы

```go
package main

import "fmt"

// Status для заказа
type OrderStatus int

const (
    Pending OrderStatus = iota
    Processing
    Shipped
    Delivered
    Cancelled
)

// Метод для красивого вывода статуса
func (s OrderStatus) String() string {
    switch s {
    case Pending:
        return "Pending"
    case Processing:
        return "Processing"
    case Shipped:
        return "Shipped"
    case Delivered:
        return "Delivered"
    case Cancelled:
        return "Cancelled"
    default:
        return "Unknown"
    }
}

// Метод для проверки, можно ли отменить заказ
func (s OrderStatus) CanCancel() bool {
    return s == Pending || s == Processing
}

// Приоритет задачи
type Priority int

const (
    Low Priority = iota
    Medium
    High
    Critical
)

func (p Priority) String() string {
    names := []string{"Low", "Medium", "High", "Critical"}
    if p < 0 || int(p) >= len(names) {
        return "Unknown"
    }
    return names[p]
}

// Уровень доступа пользователя
type Role string

const (
    Admin     Role = "admin"
    Moderator Role = "moderator"
    User      Role = "user"
    Guest     Role = "guest"
)

func (r Role) CanEdit() bool {
    return r == Admin || r == Moderator
}

func (r Role) CanDelete() bool {
    return r == Admin
}

func main() {
    // Работа с OrderStatus
    status := Processing
    fmt.Printf("Order status: %s\n", status)
    fmt.Printf("Can cancel: %t\n", status.CanCancel())
    
    status = Delivered
    fmt.Printf("Order status: %s\n", status)
    fmt.Printf("Can cancel: %t\n", status.CanCancel())
    
    // Работа с Priority
    task := Priority(High)
    fmt.Printf("Task priority: %s\n", task)
    
    // Работа с Role
    user := Role(Moderator)
    fmt.Printf("User role: %s\n", user)
    fmt.Printf("Can edit: %t\n", user.CanEdit())
    fmt.Printf("Can delete: %t\n", user.CanDelete())
}
```

---

## 🎨 Сложные кастомные типы

```go
package main

import (
    "fmt"
    "regexp"
    "strings"
)

// URL тип с валидацией
type URL string

func NewURL(raw string) (URL, error) {
    if !strings.HasPrefix(raw, "http://") && !strings.HasPrefix(raw, "https://") {
        return "", fmt.Errorf("URL must start with http:// or https://")
    }
    return URL(raw), nil
}

func (u URL) String() string {
    return string(u)
}

func (u URL) IsSecure() bool {
    return strings.HasPrefix(string(u), "https://")
}

func (u URL) GetDomain() string {
    s := string(u)
    s = strings.TrimPrefix(s, "https://")
    s = strings.TrimPrefix(s, "http://")
    
    if idx := strings.Index(s, "/"); idx != -1 {
        s = s[:idx]
    }
    return s
}

// PhoneNumber с форматированием
type PhoneNumber string

func NewPhoneNumber(raw string) (PhoneNumber, error) {
    // Убираем все кроме цифр
    re := regexp.MustCompile(`\D`)
    clean := re.ReplaceAllString(raw, "")
    
    if len(clean) != 10 && len(clean) != 11 {
        return "", fmt.Errorf("invalid phone number length")
    }
    
    return PhoneNumber(clean), nil
}

func (p PhoneNumber) Format() string {
    s := string(p)
    if len(s) == 11 {
        return fmt.Sprintf("+%s (%s) %s-%s-%s", 
            s[0:1], s[1:4], s[4:7], s[7:9], s[9:11])
    }
    return fmt.Sprintf("(%s) %s-%s-%s", s[0:3], s[3:6], s[6:8], s[8:10])
}

// Password с требованиями безопасности
type Password string

func NewPassword(raw string) (Password, error) {
    if len(raw) < 8 {
        return "", fmt.Errorf("password must be at least 8 characters")
    }
    
    hasUpper := regexp.MustCompile(`[A-Z]`).MatchString(raw)
    hasLower := regexp.MustCompile(`[a-z]`).MatchString(raw)
    hasDigit := regexp.MustCompile(`\d`).MatchString(raw)
    
    if !hasUpper || !hasLower || !hasDigit {
        return "", fmt.Errorf("password must contain upper case, lower case, and digit")
    }
    
    return Password(raw), nil
}

func (p Password) Strength() string {
    s := string(p)
    score := 0
    
    if len(s) >= 12 {
        score++
    }
    if regexp.MustCompile(`[!@#$%^&*]`).MatchString(s) {
        score++
    }
    if len(s) >= 16 {
        score++
    }
    
    switch score {
    case 0, 1:
        return "Weak"
    case 2:
        return "Medium"
    default:
        return "Strong"
    }
}

func main() {
    // Работа с URL
    url, err := NewURL("https://example.com/page")
    if err != nil {
        fmt.Printf("URL error: %v\n", err)
    } else {
        fmt.Printf("URL: %s\n", url)
        fmt.Printf("Secure: %t\n", url.IsSecure())
        fmt.Printf("Domain: %s\n", url.GetDomain())
    }
    
    // Работа с телефоном
    phone, err := NewPhoneNumber("8-800-555-35-35")
    if err != nil {
        fmt.Printf("Phone error: %v\n", err)
    } else {
        fmt.Printf("Phone: %s\n", phone.Format())
    }
    
    // Работа с паролем
    password, err := NewPassword("MySecret123")
    if err != nil {
        fmt.Printf("Password error: %v\n", err)
    } else {
        fmt.Printf("Password strength: %s\n", password.Strength())
    }
}
```

---

## 🔄 Реализация интерфейсов

```go
package main

import (
    "fmt"
    "strconv"
)

// Интерфейс для объектов, которые можно сравнивать
type Comparable interface {
    Compare(other Comparable) int
}

// Version тип для версий ПО
type Version struct {
    Major, Minor, Patch int
}

func (v Version) String() string {
    return fmt.Sprintf("%d.%d.%d", v.Major, v.Minor, v.Patch)
}

func (v Version) Compare(other Comparable) int {
    otherVersion, ok := other.(Version)
    if !ok {
        return -1
    }
    
    if v.Major != otherVersion.Major {
        return v.Major - otherVersion.Major
    }
    if v.Minor != otherVersion.Minor {
        return v.Minor - otherVersion.Minor
    }
    return v.Patch - otherVersion.Patch
}

// Score тип для игрового счета
type Score int

func (s Score) String() string {
    return strconv.Itoa(int(s))
}

func (s Score) Compare(other Comparable) int {
    otherScore, ok := other.(Score)
    if !ok {
        return -1
    }
    return int(s - otherScore)
}

func CompareItems(a, b Comparable) string {
    result := a.Compare(b)
    switch {
    case result > 0:
        return fmt.Sprintf("%v > %v", a, b)
    case result < 0:
        return fmt.Sprintf("%v < %v", a, b)
    default:
        return fmt.Sprintf("%v = %v", a, b)
    }
}

func main() {
    v1 := Version{1, 2, 3}
    v2 := Version{1, 2, 4}
    v3 := Version{1, 2, 3}
    
    fmt.Println(CompareItems(v1, v2))
    fmt.Println(CompareItems(v1, v3))
    
    score1 := Score(100)
    score2 := Score(95)
    
    fmt.Println(CompareItems(score1, score2))
}
```

---

## 🧠 Практический пример

Система управления задачами:

```go
package main

import (
    "fmt"
    "time"
)

type TaskID int
type UserID int

type Priority int
const (
    Low Priority = iota
    Medium
    High
    Critical
)

type Status int
const (
    Todo Status = iota
    InProgress
    Done
    Archived
)

type Task struct {
    ID          TaskID
    Title       string
    Description string
    AssignedTo  UserID
    Priority    Priority
    Status      Status
    Created     time.Time
    Updated     time.Time
    DueDate     *time.Time
}

// Методы для Priority
func (p Priority) String() string {
    names := []string{"Low", "Medium", "High", "Critical"}
    return names[p]
}

func (p Priority) Color() string {
    colors := []string{"🟢", "🟡", "🟠", "🔴"}
    return colors[p]
}

// Методы для Status
func (s Status) String() string {
    names := []string{"Todo", "In Progress", "Done", "Archived"}
    return names[s]
}

func (s Status) Icon() string {
    icons := []string{"⭕", "🔄", "✅", "📦"}
    return icons[s]
}

// Методы для Task
func (t *Task) SetPriority(p Priority) {
    t.Priority = p
    t.Updated = time.Now()
}

func (t *Task) SetStatus(s Status) {
    t.Status = s
    t.Updated = time.Now()
}

func (t *Task) IsOverdue() bool {
    if t.DueDate == nil {
        return false
    }
    return time.Now().After(*t.DueDate) && t.Status != Done
}

func (t *Task) String() string {
    overdue := ""
    if t.IsOverdue() {
        overdue = " ⚠️ OVERDUE"
    }
    
    due := "No due date"
    if t.DueDate != nil {
        due = t.DueDate.Format("2006-01-02")
    }
    
    return fmt.Sprintf("[%d] %s %s\n  %s %s | Due: %s%s\n  Assigned to: User %d",
        t.ID, t.Status.Icon(), t.Title,
        t.Priority.Color(), t.Priority,
        due, overdue, t.AssignedTo)
}

func main() {
    dueDate := time.Now().AddDate(0, 0, -1) // Вчера (просрочено)
    
    task := Task{
        ID:          TaskID(1),
        Title:       "Implement user authentication",
        Description: "Add JWT authentication to the API",
        AssignedTo:  UserID(123),
        Priority:    High,
        Status:      InProgress,
        Created:     time.Now().AddDate(0, 0, -7),
        Updated:     time.Now(),
        DueDate:     &dueDate,
    }
    
    fmt.Printf("Task Details:\n%s\n\n", &task)
    
    // Изменяем приоритет
    task.SetPriority(Critical)
    fmt.Printf("After priority change:\n%s\n\n", &task)
    
    // Завершаем задачу
    task.SetStatus(Done)
    fmt.Printf("After completion:\n%s\n", &task)
}
```

---

## 🧠 Проверь себя

- Как создать кастомный тип на основе существующего?
- Можно ли напрямую присвоить значения разных кастомных типов?
- Как добавить методы к кастомному типу?
- В чем преимущество использования кастомных типов вместо встроенных?
- Как создать enum-подобный тип в Go?

---

## 📌 Главное из главы

- **Кастомные типы** создаются с помощью `type NewType ExistingType`
- **Методы** можно добавлять к любым кастомным типам
- **Безопасность типов** предотвращает ошибки на этапе компиляции
- **Enum-подобные типы** создаются с помощью `const` и `iota`
- **Валидация** может быть встроена в конструкторы типов
- **Интерфейсы** работают с кастомными типами так же, как с встроенными

---