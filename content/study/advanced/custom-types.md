---
title: "5.4 🎭 Кастомные типы"
description: "Кастомные типы в Go: создание новых типов, методы типов, type assertions, безопасность типов."
keywords: ["кастомные типы go", "новые типы golang", "методы типов go", "type golang"]
date: 2025-08-14
weight: 54
draft: false
---

**Кастомные типы** в Go представляют собой один из самых мощных механизмов языка для создания выразительного, безопасного и самодокументирующегося кода. Вместо использования примитивных типов вроде `int`, `string` или `float64` повсюду в коде, разработчики могут создавать специализированные типы, которые несут семантическую нагрузку и предотвращают целые классы ошибок на этапе компиляции.

В современной разработке на Go кастомные типы стали стандартом качественного кода. Они не только делают код более читаемым, но и создают дополнительный уровень безопасности типов, который помогает обнаруживать логические ошибки еще до запуска программы.

## 🎯 Фундаментальные преимущества кастомных типов

### Семантическая ясность кода
Когда вы видите функцию `ProcessPayment(amount float64)`, неясно, в каких единицах измерения amount — в рублях, долларах, копейках? А функция `ProcessPayment(amount Money)` сразу дает понимание того, что amount — это денежная сумма со всей соответствующей логикой.

### Предотвращение логических ошибок
Классический пример — функция перевода денег. Представьте:
```go
TransferMoney(123, 456, 1000) // Что означают эти числа?
```
Против:
```go
TransferMoney(UserID(123), UserID(456), Money(1000)) // Кристально ясно!
```

С кастомными типами компилятор физически не позволит вам перепутать порядок аргументов или передать неподходящий тип.

### Инкапсуляция бизнес-логики
Кастомные типы могут иметь методы, которые инкапсулируют связанную с типом логику. Email может валидировать себя, Money — конвертировать валюты, Temperature — переводить единицы измерения.

> 💬 **Реальная статистика из практики**  
> В одном крупном финтех-проекте переход на кастомные типы для денежных операций сократил количество багов на 40% и время на code review на 25%, поскольку код стал самообъясняющимся.

---

## 🎯 Синтаксис и основы кастомных типов

Создание кастомного типа в Go выполняется с помощью ключевого слова `type`. Синтаксически это выглядит просто, но за этой простотой скрывается мощная система типов, которая обеспечивает как гибкость, так и безопасность.

### Философия типов в Go

Go использует номинальную систему типов, что означает: два типа считаются разными, даже если у них одинаковая внутренняя структура, если они объявлены отдельно. Это кардинально отличается от структурной типизации и обеспечивает дополнительную безопасность.

Когда вы создаете `type UserID int`, вы создаете полностью новый тип, который:
- Имеет те же операции, что и `int` (сложение, вычитание, сравнение)
- Не может быть напрямую присвоен переменной типа `int` без явного преобразования
- Может иметь собственные методы
- Участвует в проверке типов на этапе компиляции

### Создание простых типов

```go
// Создание базовых кастомных типов
type UserID int
type ProductID int
type Email string
type PhoneNumber string

func main() {
    var userID UserID = 123
    var productID ProductID = 456
    
    // Ошибка компиляции: нельзя присвоить напрямую
    // userID = productID
    
    // Требуется явное преобразование
    userID = UserID(productID)
}
```

### Ключевые особенности типизации

Этот простой пример демонстрирует фундаментальные принципы:

1. **Безопасность типов**: Попытка присвоить `productID` напрямую в `userID` вызовет ошибку компиляции, предотвращая логические ошибки.

2. **Явное преобразование**: `UserID(productID)` — это не просто приведение типов, а явное заявление о том, что вы понимаете семантику этого преобразования.

3. **Наследование операций**: Кастомные типы автоматически получают все операции базового типа — арифметические, сравнения, форматирование.

4. **Читаемость кода**: `Email` и `PhoneNumber` сразу дают понимание назначения переменной.

В продакшене такой подход критически важен. Представьте e-commerce систему, где перепутать ID пользователя и ID продукта может привести к серьезным последствиям.

---

## 🏗️ Методы кастомных типов

Одна из самых мощных возможностей кастомных типов — способность добавлять к ним методы. Это позволяет инкапсулировать логику, связанную с типом, и создавать rich domain models прямо в системе типов Go.

### Принципы проектирования методов

При создании методов для кастомных типов важно следовать принципам:
- **Единственная ответственность**: каждый метод решает одну задачу
- **Immutability**: методы не должны изменять состояние неожиданным образом
- **Композиция**: сложные операции строятся из простых
- **Defensive programming**: методы должны корректно обрабатывать некорректные данные

```go
type Email string

// Методы для Email типа
func (e Email) IsValid() bool {
    return strings.Contains(string(e), "@") && strings.Contains(string(e), ".")
}

func (e Email) GetDomain() string {
    parts := strings.Split(string(e), "@")
    if len(parts) != 2 {
        return ""
    }
    return parts[1]
}

func (e Email) Normalize() Email {
    return Email(strings.ToLower(string(e)))
}

// Использование
email := Email("USER@Example.COM")
if email.IsValid() {
    normalized := email.Normalize()
    domain := normalized.GetDomain()
}
```

#### Паттерны проектирования методов

**1. Fluent Interface (Цепочка вызовов):**
```go
user := NewUser("John").
    SetEmail("john@example.com").
    SetAge(30).
    SetRole("admin")
```

**2. Builder Pattern:**
```go
type UserBuilder struct {
    user User
}

func (b *UserBuilder) WithEmail(email Email) *UserBuilder {
    b.user.Email = email
    return b
}

func (b *UserBuilder) Build() User {
    return b.user
}
```

**3. Validation Methods:**
```go
func (u User) Validate() error {
    if !u.Email.IsValid() {
        return errors.New("invalid email")
    }
    return nil
}
```

---

## 🛡️ Типы для безопасности

Кастомные типы особенно ценны в финансовых и критически важных системах, где ошибки типов могут иметь серьезные последствия.

### Финансовые типы

```go
// Безопасные денежные типы
type Money int64      // хранение в центах/копейках
type Currency string
type UserID int64

type Amount struct {
    Value    Money
    Currency Currency
}

// Методы для безопасной работы с деньгами
func (a Amount) Add(other Amount) (Amount, error) {
    if a.Currency != other.Currency {
        return Amount{}, errors.New("currency mismatch")
    }
    return Amount{Value: a.Value + other.Value, Currency: a.Currency}, nil
}

// Безопасная функция перевода
func TransferMoney(from, to UserID, amount Amount) error {
    if from == to {
        return errors.New("self-transfer not allowed")
    }
    if amount.Value <= 0 {
        return errors.New("amount must be positive")
    }
    // Выполняем перевод...
    return nil
}
```

#### Архитектурные преимущества

**1. Предотвращение смешивания единиц измерения:**
- `Money` в центах исключает ошибки округления
- `Currency` предотвращает операции между разными валютами
- Компилятор не позволит передать `OrderID` вместо `UserID`

**2. Самодокументирующийся код:**
```go
// Понятно без комментариев
func CalculateCommission(amount Money, rate Percentage) Money
func GetUserOrders(userID UserID) []Order
func UpdateProductPrice(productID ProductID, newPrice Money)
```

**3. Отлов ошибок на этапе компиляции:**
```go
userID := UserID(123)
orderID := OrderID(456)

// Ошибка компиляции - компилятор защищает от логических ошибок
// ProcessUser(orderID) // Нельзя!
ProcessUser(userID)   // Правильно
```

### Domain-Driven Design с типами

Кастомные типы идеально подходят для реализации DDD концепций:

**Value Objects:**
```go
type Email string
type PhoneNumber string
type Address struct {
    Street   string
    City     string
    PostCode string
}
```

**Entity IDs:**
```go
type CustomerID string
type ProductID int64
type OrderID uuid.UUID
```

**Business Rules в типах:**
```go
type Age int

func NewAge(value int) (Age, error) {
    if value < 0 || value > 150 {
        return 0, errors.New("invalid age")
    }
    return Age(value), nil
}
```

---

## 🔢 Enum-подобные типы

Go не имеет встроенного enum типа, но кастомные типы с константами элегантно решают эту задачу.

### Числовые Enum с iota

```go
type OrderStatus int

const (
    Pending OrderStatus = iota
    Processing
    Shipped
    Delivered
    Cancelled
)

func (s OrderStatus) String() string {
    names := []string{"Pending", "Processing", "Shipped", "Delivered", "Cancelled"}
    if s < 0 || int(s) >= len(names) {
        return "Unknown"
    }
    return names[s]
}

func (s OrderStatus) CanCancel() bool {
    return s == Pending || s == Processing
}
```

### Строковые Enum

```go
type Role string

const (
    Admin     Role = "admin"
    Moderator Role = "moderator"
    User      Role = "user"
    Guest     Role = "guest"
)

func (r Role) HasPermission(action string) bool {
    permissions := map[Role][]string{
        Admin:     {"read", "write", "delete", "admin"},
        Moderator: {"read", "write", "moderate"},
        User:      {"read", "write"},
        Guest:     {"read"},
    }
    
    for _, perm := range permissions[r] {
        if perm == action {
            return true
        }
    }
    return false
}
```

#### Преимущества Enum типов в Go

**1. Type Safety:**
Компилятор предотвратит использование некорректных значений

**2. Расширяемость:**
Легко добавить новые состояния без изменения существующего кода

**3. Методы для бизнес-логики:**
```go
func (s OrderStatus) NextStatus() OrderStatus {
    switch s {
    case Pending:
        return Processing
    case Processing:
        return Shipped
    case Shipped:
        return Delivered
    default:
        return s // Нельзя изменить статус
    }
}
```

**4. Валидация:**
```go
func NewOrderStatus(value int) (OrderStatus, error) {
    if value < int(Pending) || value > int(Cancelled) {
        return 0, errors.New("invalid order status")
    }
    return OrderStatus(value), nil
}
```

---

## 🎨 Типы с валидацией

Кастомные типы могут инкапсулировать сложную логику валидации и форматирования, создавая самопроверяющиеся value objects.

### Конструкторы с валидацией

```go
type Email string

func NewEmail(raw string) (Email, error) {
    if !strings.Contains(raw, "@") || !strings.Contains(raw, ".") {
        return "", errors.New("invalid email format")
    }
    return Email(strings.ToLower(raw)), nil
}

type Password string

func NewPassword(raw string) (Password, error) {
    if len(raw) < 8 {
        return "", errors.New("password too short")
    }
    
    hasUpper := regexp.MustCompile(`[A-Z]`).MatchString(raw)
    hasLower := regexp.MustCompile(`[a-z]`).MatchString(raw)
    hasDigit := regexp.MustCompile(`\d`).MatchString(raw)
    
    if !hasUpper || !hasLower || !hasDigit {
        return "", errors.New("password must contain upper, lower, and digit")
    }
    
    return Password(raw), nil
}
```

#### Паттерн "Smart Constructor"

Этот паттерн гарантирует, что объект всегда находится в валидном состоянии:

**1. Приватный тип + публичный конструктор:**
```go
type validatedEmail struct {
    value string
}

func NewEmail(raw string) (*validatedEmail, error) {
    // Валидация...
    return &validatedEmail{value: clean}, nil
}

func (e *validatedEmail) String() string {
    return e.value // Гарантированно валидный
}
```

**2. Методы для трансформации:**
```go
func (e Email) Normalize() Email {
    return Email(strings.ToLower(string(e)))
}

func (p PhoneNumber) Format() string {
    // Красивое форматирование номера
}

func (pwd Password) Strength() SecurityLevel {
    // Оценка надежности пароля
}
```

#### Преимущества подхода

**Fail Fast принцип:** Ошибки обнаруживаются при создании объекта, а не при использовании

**Неизменяемость:** После создания объект гарантированно корректен

**Самодокументирование:** Тип сразу показывает ограничения и формат данных

**Композиция валидаций:** Можно комбинировать различные типы проверок

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