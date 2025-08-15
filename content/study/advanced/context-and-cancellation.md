---
title: "5.5 🚫 Context и отмена операций"
description: "Context в Go: управление жизненным циклом операций, отмена запросов, таймауты, передача данных."
keywords: ["context golang", "отмена операций go", "таймауты golang", "context package go"]
date: 2025-08-14
weight: 55
draft: true
---

**Context** — это один из самых важных пакетов в Go для управления жизненным циклом операций. Он позволяет отменять длительные операции, передавать данные между функциями и устанавливать таймауты. Context особенно важен в веб-серверах, API и любых приложениях, работающих с внешними ресурсами.

> 💬 **Зачем нужен Context?**  
> Представь, что пользователь закрыл браузер, а сервер всё еще обрабатывает его запрос к базе данных. Context позволяет сразу отменить эту операцию и освободить ресурсы!

---

## 🎯 Основы Context

### Создание и использование Context

```go
package main

import (
    "context"
    "fmt"
    "time"
)

// Простая функция, которая может быть отменена
func doWork(ctx context.Context, name string) error {
    for i := 0; i < 5; i++ {
        select {
        case <-ctx.Done():
            // Контекст отменен
            return ctx.Err()
        default:
            fmt.Printf("%s: выполняю работу шаг %d\n", name, i+1)
            time.Sleep(1 * time.Second)
        }
    }
    fmt.Printf("%s: работа завершена!\n", name)
    return nil
}

func main() {
    // 1. Background context - корневой контекст
    ctx := context.Background()
    
    fmt.Println("1. Выполняем работу без отмены:")
    doWork(ctx, "worker1")
    
    fmt.Println("\n2. Выполняем работу с отменой:")
    // Создаем контекст с возможностью отмены
    ctx, cancel := context.WithCancel(context.Background())
    
    go func() {
        time.Sleep(2 * time.Second)
        fmt.Println("Отменяем работу!")
        cancel()
    }()
    
    err := doWork(ctx, "worker2")
    if err != nil {
        fmt.Printf("Работа прервана: %v\n", err)
    }
}
```

---

## ⏰ Context с таймаутами

### WithTimeout и WithDeadline

```go
package main

import (
    "context"
    "fmt"
    "time"
)

// Симулируем долгую операцию
func longOperation(ctx context.Context, duration time.Duration) error {
    select {
    case <-time.After(duration):
        fmt.Println("✅ Операция завершена успешно")
        return nil
    case <-ctx.Done():
        fmt.Println("❌ Операция прервана:", ctx.Err())
        return ctx.Err()
    }
}

func main() {
    fmt.Println("🕐 Context с таймаутом")
    
    // 1. WithTimeout - автоматическая отмена через указанное время
    fmt.Println("\n1. Таймаут 3 секунды, операция займет 2 секунды:")
    ctx1, cancel1 := context.WithTimeout(context.Background(), 3*time.Second)
    defer cancel1()
    
    longOperation(ctx1, 2*time.Second)
    
    // 2. Таймаут меньше времени операции
    fmt.Println("\n2. Таймаут 1 секунда, операция займет 2 секунды:")
    ctx2, cancel2 := context.WithTimeout(context.Background(), 1*time.Second)
    defer cancel2()
    
    longOperation(ctx2, 2*time.Second)
    
    // 3. WithDeadline - отмена в конкретное время
    fmt.Println("\n3. Deadline через 2 секунды:")
    deadline := time.Now().Add(2 * time.Second)
    ctx3, cancel3 := context.WithDeadline(context.Background(), deadline)
    defer cancel3()
    
    fmt.Printf("Deadline: %s\n", deadline.Format("15:04:05"))
    longOperation(ctx3, 3*time.Second)
}
```

---

## 📦 Context с данными

### WithValue для передачи данных

```go
package main

import (
    "context"
    "fmt"
)

// Типы ключей для Context
type contextKey string

const (
    UserIDKey    contextKey = "user_id"
    SessionKey   contextKey = "session"
    RequestIDKey contextKey = "request_id"
)

// Функции-помощники для работы с контекстом
func GetUserID(ctx context.Context) (int, bool) {
    userID, ok := ctx.Value(UserIDKey).(int)
    return userID, ok
}

func SetUserID(ctx context.Context, userID int) context.Context {
    return context.WithValue(ctx, UserIDKey, userID)
}

func GetRequestID(ctx context.Context) (string, bool) {
    requestID, ok := ctx.Value(RequestIDKey).(string)
    return requestID, ok
}

// Симулируем функции приложения
func authenticate(ctx context.Context, token string) context.Context {
    // Симулируем аутентификацию
    userID := 123 // получили из токена
    sessionID := "session_xyz"
    
    ctx = SetUserID(ctx, userID)
    ctx = context.WithValue(ctx, SessionKey, sessionID)
    
    fmt.Printf("🔐 Пользователь %d аутентифицирован\n", userID)
    return ctx
}

func processRequest(ctx context.Context) {
    requestID, _ := GetRequestID(ctx)
    userID, hasUser := GetUserID(ctx)
    session, _ := ctx.Value(SessionKey).(string)
    
    fmt.Printf("📨 Обрабатываем запрос %s\n", requestID)
    
    if hasUser {
        fmt.Printf("👤 Пользователь: %d, Сессия: %s\n", userID, session)
    } else {
        fmt.Println("🚫 Неавторизованный запрос")
    }
    
    // Вызываем другие функции с тем же контекстом
    fetchUserData(ctx)
    logActivity(ctx)
}

func fetchUserData(ctx context.Context) {
    userID, hasUser := GetUserID(ctx)
    requestID, _ := GetRequestID(ctx)
    
    fmt.Printf("💾 [%s] Загружаем данные пользователя", requestID)
    
    if hasUser {
        fmt.Printf(" %d\n", userID)
        // Здесь был бы реальный запрос к базе данных
    } else {
        fmt.Println(" - пользователь не найден")
    }
}

func logActivity(ctx context.Context) {
    userID, hasUser := GetUserID(ctx)
    requestID, _ := GetRequestID(ctx)
    
    fmt.Printf("📊 [%s] Логируем активность", requestID)
    
    if hasUser {
        fmt.Printf(" пользователя %d\n", userID)
    } else {
        fmt.Println(" анонимного пользователя")
    }
}

func main() {
    // Создаем корневой контекст
    ctx := context.Background()
    
    // Добавляем ID запроса
    ctx = context.WithValue(ctx, RequestIDKey, "req_12345")
    
    fmt.Println("=== Обработка авторизованного запроса ===")
    authCtx := authenticate(ctx, "valid_token")
    processRequest(authCtx)
    
    fmt.Println("\n=== Обработка неавторизованного запроса ===")
    processRequest(ctx)
}
```

---

## 🌐 Context в HTTP серверах

```go
package main

import (
    "context"
    "encoding/json"
    "fmt"
    "log"
    "math/rand"
    "net/http"
    "time"
)

type contextKey string

const (
    RequestIDKey contextKey = "request_id"
    UserIDKey    contextKey = "user_id"
)

// Middleware для добавления Request ID
func requestIDMiddleware(next http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        requestID := fmt.Sprintf("req_%d", rand.Int63())
        ctx := context.WithValue(r.Context(), RequestIDKey, requestID)
        
        // Добавляем в заголовок ответа
        w.Header().Set("X-Request-ID", requestID)
        
        // Передаем модифицированный контекст
        next(w, r.WithContext(ctx))
    }
}

// Middleware для логирования
func loggingMiddleware(next http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        start := time.Now()
        requestID := r.Context().Value(RequestIDKey)
        
        log.Printf("[%v] %s %s - started", requestID, r.Method, r.URL.Path)
        
        next(w, r)
        
        duration := time.Since(start)
        log.Printf("[%v] %s %s - completed in %v", requestID, r.Method, r.URL.Path, duration)
    }
}

// Middleware для аутентификации
func authMiddleware(next http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        token := r.Header.Get("Authorization")
        
        if token == "" {
            http.Error(w, "Authorization required", http.StatusUnauthorized)
            return
        }
        
        // Симулируем проверку токена
        var userID int
        if token == "Bearer valid_token" {
            userID = 123
        } else {
            http.Error(w, "Invalid token", http.StatusUnauthorized)
            return
        }
        
        // Добавляем пользователя в контекст
        ctx := context.WithValue(r.Context(), UserIDKey, userID)
        next(w, r.WithContext(ctx))
    }
}

// Симулируем долгую операцию с базой данных
func queryDatabase(ctx context.Context, query string) (map[string]interface{}, error) {
    requestID := ctx.Value(RequestIDKey)
    log.Printf("[%v] Выполняем запрос к БД: %s", requestID, query)
    
    select {
    case <-time.After(2 * time.Second):
        return map[string]interface{}{
            "result": "data from database",
            "query":  query,
        }, nil
    case <-ctx.Done():
        log.Printf("[%v] Запрос к БД отменен: %v", requestID, ctx.Err())
        return nil, ctx.Err()
    }
}

// Обработчики
func homeHandler(w http.ResponseWriter, r *http.Request) {
    requestID := r.Context().Value(RequestIDKey)
    
    response := map[string]interface{}{
        "message":    "Hello, World!",
        "request_id": requestID,
        "timestamp":  time.Now().Format(time.RFC3339),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

func dataHandler(w http.ResponseWriter, r *http.Request) {
    ctx := r.Context()
    requestID := ctx.Value(RequestIDKey)
    userID := ctx.Value(UserIDKey)
    
    // Создаем контекст с таймаутом для запроса к БД
    dbCtx, cancel := context.WithTimeout(ctx, 3*time.Second)
    defer cancel()
    
    data, err := queryDatabase(dbCtx, "SELECT * FROM users")
    if err != nil {
        if err == context.DeadlineExceeded {
            log.Printf("[%v] Таймаут запроса к БД", requestID)
            http.Error(w, "Database timeout", http.StatusRequestTimeout)
            return
        }
        log.Printf("[%v] Ошибка БД: %v", requestID, err)
        http.Error(w, "Database error", http.StatusInternalServerError)
        return
    }
    
    response := map[string]interface{}{
        "request_id": requestID,
        "user_id":    userID,
        "data":       data,
        "timestamp":  time.Now().Format(time.RFC3339),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

func slowHandler(w http.ResponseWriter, r *http.Request) {
    ctx := r.Context()
    requestID := ctx.Value(RequestIDKey)
    
    // Симулируем очень долгую операцию
    select {
    case <-time.After(10 * time.Second):
        response := map[string]interface{}{
            "message":    "Slow operation completed",
            "request_id": requestID,
        }
        w.Header().Set("Content-Type", "application/json")
        json.NewEncoder(w).Encode(response)
        
    case <-ctx.Done():
        log.Printf("[%v] Клиент отменил запрос: %v", requestID, ctx.Err())
        // Клиент отключился, но мы уже не можем отправить ответ
        return
    }
}

func main() {
    // Публичные эндпоинты
    http.HandleFunc("/", 
        requestIDMiddleware(loggingMiddleware(homeHandler)))
    
    http.HandleFunc("/slow", 
        requestIDMiddleware(loggingMiddleware(slowHandler)))
    
    // Защищенные эндпоинты
    http.HandleFunc("/data", 
        requestIDMiddleware(loggingMiddleware(authMiddleware(dataHandler))))
    
    fmt.Println("🚀 HTTP сервер с Context запущен на http://localhost:8080")
    fmt.Println()
    fmt.Println("Эндпоинты:")
    fmt.Println("  GET  /           - публичный")
    fmt.Println("  GET  /slow       - долгая операция (10 сек)")
    fmt.Println("  GET  /data       - требует авторизации")
    fmt.Println()
    fmt.Println("Примеры запросов:")
    fmt.Println("  curl http://localhost:8080/")
    fmt.Println("  curl -H 'Authorization: Bearer valid_token' http://localhost:8080/data")
    fmt.Println("  curl http://localhost:8080/slow  # отмените через Ctrl+C")
    
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

---

## 🔄 Context в горутинах

```go
package main

import (
    "context"
    "fmt"
    "sync"
    "time"
)

// Worker - рабочий, который обрабатывает задачи
type Worker struct {
    id   int
    name string
}

func (w *Worker) processTask(ctx context.Context, taskID int) error {
    select {
    case <-time.After(time.Duration(taskID) * time.Second):
        fmt.Printf("🟢 Worker %d (%s): завершил задачу %d\n", w.id, w.name, taskID)
        return nil
    case <-ctx.Done():
        fmt.Printf("🔴 Worker %d (%s): задача %d отменена - %v\n", w.id, w.name, taskID, ctx.Err())
        return ctx.Err()
    }
}

func (w *Worker) run(ctx context.Context, tasks <-chan int, wg *sync.WaitGroup) {
    defer wg.Done()
    
    fmt.Printf("✨ Worker %d (%s) запущен\n", w.id, w.name)
    
    for {
        select {
        case task, ok := <-tasks:
            if !ok {
                fmt.Printf("📤 Worker %d (%s): канал задач закрыт\n", w.id, w.name)
                return
            }
            
            fmt.Printf("📥 Worker %d (%s): получил задачу %d\n", w.id, w.name, task)
            w.processTask(ctx, task)
            
        case <-ctx.Done():
            fmt.Printf("🛑 Worker %d (%s): остановлен - %v\n", w.id, w.name, ctx.Err())
            return
        }
    }
}

func main() {
    // Создаем контекст с отменой
    ctx, cancel := context.WithCancel(context.Background())
    
    // Канал для задач
    tasks := make(chan int, 10)
    
    // WaitGroup для ожидания завершения горутин
    var wg sync.WaitGroup
    
    // Запускаем несколько воркеров
    workers := []Worker{
        {id: 1, name: "Alice"},
        {id: 2, name: "Bob"},
        {id: 3, name: "Charlie"},
    }
    
    for _, worker := range workers {
        wg.Add(1)
        go worker.run(ctx, tasks, &wg)
    }
    
    // Отправляем задачи
    go func() {
        for i := 1; i <= 10; i++ {
            tasks <- i
            time.Sleep(500 * time.Millisecond)
        }
        close(tasks)
    }()
    
    // Отменяем через 8 секунд
    go func() {
        time.Sleep(8 * time.Second)
        fmt.Println("\n🚨 Отменяем все операции!")
        cancel()
    }()
    
    // Ждем завершения всех воркеров
    wg.Wait()
    fmt.Println("\n✅ Все воркеры завершены")
}
```

---

## 🎯 Практический пример: HTTP клиент с Context

```go
package main

import (
    "context"
    "encoding/json"
    "fmt"
    "io"
    "net/http"
    "time"
)

// APIClient с поддержкой Context
type APIClient struct {
    baseURL    string
    httpClient *http.Client
}

func NewAPIClient(baseURL string) *APIClient {
    return &APIClient{
        baseURL: baseURL,
        httpClient: &http.Client{
            Timeout: 30 * time.Second,
        },
    }
}

// Выполняем HTTP запрос с контекстом
func (c *APIClient) makeRequest(ctx context.Context, method, path string) (map[string]interface{}, error) {
    url := c.baseURL + path
    
    // Создаем запрос с контекстом
    req, err := http.NewRequestWithContext(ctx, method, url, nil)
    if err != nil {
        return nil, fmt.Errorf("создание запроса: %w", err)
    }
    
    // Выполняем запрос
    resp, err := c.httpClient.Do(req)
    if err != nil {
        return nil, fmt.Errorf("выполнение запроса: %w", err)
    }
    defer resp.Body.Close()
    
    if resp.StatusCode != http.StatusOK {
        return nil, fmt.Errorf("HTTP error: %d", resp.StatusCode)
    }
    
    // Читаем ответ
    body, err := io.ReadAll(resp.Body)
    if err != nil {
        return nil, fmt.Errorf("чтение ответа: %w", err)
    }
    
    var result map[string]interface{}
    if err := json.Unmarshal(body, &result); err != nil {
        return nil, fmt.Errorf("парсинг JSON: %w", err)
    }
    
    return result, nil
}

// Получаем данные пользователя
func (c *APIClient) GetUser(ctx context.Context, userID int) (map[string]interface{}, error) {
    path := fmt.Sprintf("/users/%d", userID)
    return c.makeRequest(ctx, "GET", path)
}

// Получаем список постов
func (c *APIClient) GetPosts(ctx context.Context) ([]map[string]interface{}, error) {
    result, err := c.makeRequest(ctx, "GET", "/posts")
    if err != nil {
        return nil, err
    }
    
    // Преобразуем в массив
    if posts, ok := result["posts"].([]interface{}); ok {
        var typedPosts []map[string]interface{}
        for _, post := range posts {
            if typedPost, ok := post.(map[string]interface{}); ok {
                typedPosts = append(typedPosts, typedPost)
            }
        }
        return typedPosts, nil
    }
    
    return nil, fmt.Errorf("неожиданный формат ответа")
}

func main() {
    client := NewAPIClient("https://jsonplaceholder.typicode.com")
    
    fmt.Println("🔄 Тестируем API клиент с Context")
    
    // 1. Успешный запрос
    fmt.Println("\n1. Получаем пользователя (успешно):")
    ctx1 := context.Background()
    
    user, err := client.GetUser(ctx1, 1)
    if err != nil {
        fmt.Printf("❌ Ошибка: %v\n", err)
    } else {
        fmt.Printf("✅ Пользователь: %s (%s)\n", user["name"], user["email"])
    }
    
    // 2. Запрос с коротким таймаутом
    fmt.Println("\n2. Запрос с таймаутом 100мс (должен провалиться):")
    ctx2, cancel2 := context.WithTimeout(context.Background(), 100*time.Millisecond)
    defer cancel2()
    
    _, err = client.GetUser(ctx2, 1)
    if err != nil {
        fmt.Printf("❌ Ошибка (ожидаемо): %v\n", err)
    }
    
    // 3. Отмена запроса вручную
    fmt.Println("\n3. Отмена запроса через 1 секунду:")
    ctx3, cancel3 := context.WithCancel(context.Background())
    
    // Отменяем через 1 секунду
    go func() {
        time.Sleep(1 * time.Second)
        fmt.Println("🚨 Отменяем запрос!")
        cancel3()
    }()
    
    start := time.Now()
    _, err = client.GetUser(ctx3, 1)
    duration := time.Since(start)
    
    if err != nil {
        fmt.Printf("❌ Запрос отменен через %v: %v\n", duration, err)
    }
    
    // 4. Множественные запросы с общим контекстом
    fmt.Println("\n4. Множественные запросы с общим таймаутом:")
    ctx4, cancel4 := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel4()
    
    userIDs := []int{1, 2, 3}
    for _, id := range userIDs {
        user, err := client.GetUser(ctx4, id)
        if err != nil {
            fmt.Printf("❌ Пользователь %d: %v\n", id, err)
            break
        } else {
            fmt.Printf("✅ Пользователь %d: %s\n", id, user["name"])
        }
        
        // Проверяем, не отменен ли контекст
        if ctx4.Err() != nil {
            fmt.Printf("🛑 Контекст отменен: %v\n", ctx4.Err())
            break
        }
    }
    
    fmt.Println("\n✨ Тестирование завершено!")
}
```

---

## 🛡️ Лучшие практики Context

### 1. Всегда передавай Context первым параметром
```go
// ✅ Правильно
func ProcessUser(ctx context.Context, userID int) error

// ❌ Неправильно
func ProcessUser(userID int, ctx context.Context) error
```

### 2. Не храни Context в структурах
```go
// ❌ Неправильно
type Service struct {
    ctx context.Context
}

// ✅ Правильно
type Service struct {
    // другие поля
}

func (s *Service) DoWork(ctx context.Context) error
```

### 3. Используй типизированные ключи
```go
// ✅ Правильно
type contextKey string
const UserIDKey contextKey = "user_id"

// ❌ Неправильно
ctx = context.WithValue(ctx, "user_id", 123)
```

### 4. Не игнорируй отмену
```go
func longOperation(ctx context.Context) error {
    for i := 0; i < 1000; i++ {
        // ✅ Проверяем отмену
        select {
        case <-ctx.Done():
            return ctx.Err()
        default:
        }
        
        // выполняем работу
    }
    return nil
}
```

---

## 🧠 Проверь себя

- Зачем нужен Context в Go?
- Как создать Context с таймаутом?
- Как правильно передавать данные через Context?
- Почему Context должен быть первым параметром функции?
- Как обрабатывать отмену операций в горутинах?

---

## 📌 Главное из главы

- **Context** управляет жизненным циклом операций и передачей данных
- **context.WithCancel()** для ручной отмены
- **context.WithTimeout()** для автоматической отмены по времени  
- **context.WithValue()** для передачи данных между функциями
- **ctx.Done()** сигналит об отмене операции
- **Всегда проверяй** ctx.Done() в длительных операциях
- **Context первый параметр** в функциях по конвенции

---