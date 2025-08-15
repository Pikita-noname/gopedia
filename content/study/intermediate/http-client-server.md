---
title: "4.2 🌐 HTTP клиент и сервер"
description: "HTTP клиент и сервер в Go: net/http пакет, создание REST API, middleware, обработка запросов."
keywords: ["http сервер golang", "rest api go", "net/http golang", "web сервер go"]
date: 2025-08-14
weight: 42
draft: false
---

**HTTP клиент и сервер в Go** — фундаментальные навыки для создания современных веб-приложений и API. Go предоставляет исключительно мощный и элегантный пакет `net/http`, который делает создание производительных HTTP сервисов простым и интуитивным. Этот пакет используется в production-системах крупнейших технологических компаний мира.

---
## 🎯 **Зачем Go разработчику HTTP?**

В современной экосистеме Go HTTP используется повсеместно:

**Backend разработка**: REST API, GraphQL серверы, микросервисы  
**Cloud-native applications**: Kubernetes operators, service mesh  
**DevOps инструменты**: CI/CD системы, мониторинг, метрики  
**Integration tasks**: Webhooks, внешние API, data pipelines  
**Real-time applications**: WebSocket серверы, streaming APIs  
**Enterprise системы**: B2B интеграции, внутренние сервисы  

Go стал языком выбора для HTTP сервисов благодаря встроенной поддержке конкурентности, низким накладным расходам и простоте развертывания в виде одного бинарника.

---
## 📚 **Пакет net/http: полный арсенал**

### **Ключевые возможности пакета**
**HTTP сервер** — встроенный production-ready веб-сервер  
**HTTP клиент** — мощный клиент с поддержкой connection pooling  
**Middleware support** — легко создавать промежуточное ПО  
**TLS/HTTPS** — встроенная поддержка безопасных соединений  
**HTTP/2** — автоматическая поддержка современного протокола  
**Context integration** — полная поддержка контекстов для cancellation  

### **Философия HTTP в Go**
В Go HTTP сервер - это не отдельная сущность, а часть вашего приложения. Нет необходимости в сложных серверах приложений как Apache или Nginx для запуска Go приложений. Скомпилированный бинарник сам является веб-сервером, готовым к production использованию.

---

## 🖥️ HTTP Сервер

### Простейший HTTP сервер

```go
package main

import (
    "fmt"
    "log"
    "net/http"
)

func homeHandler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Привет, мир! Это мой первый HTTP сервер на Go\n")
    fmt.Fprintf(w, "Метод запроса: %s\n", r.Method)
    fmt.Fprintf(w, "URL: %s\n", r.URL.Path)
}

func aboutHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/html; charset=utf-8")
    fmt.Fprintf(w, "<h1>О проекте</h1>")
    fmt.Fprintf(w, "<p>Это учебный HTTP сервер на Go</p>")
}

func main() {
    // Регистрируем обработчики маршрутов
    http.HandleFunc("/", homeHandler)
    http.HandleFunc("/about", aboutHandler)
    
    fmt.Println("Сервер запущен на http://localhost:8080")
    
    // Запускаем сервер
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

### JSON API сервер

```go
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "strconv"
    "strings"
    "time"
)

// Структуры данных
type User struct {
    ID        int       `json:"id"`
    Name      string    `json:"name"`
    Email     string    `json:"email"`
    CreatedAt time.Time `json:"created_at"`
}

type APIResponse struct {
    Success bool        `json:"success"`
    Data    interface{} `json:"data,omitempty"`
    Error   string      `json:"error,omitempty"`
}

// Простое "хранилище" пользователей в памяти
var users = map[int]*User{
    1: {ID: 1, Name: "Иван Иванов", Email: "ivan@example.com", CreatedAt: time.Now()},
    2: {ID: 2, Name: "Петр Петров", Email: "petr@example.com", CreatedAt: time.Now()},
}
var nextUserID = 3

// Вспомогательные функции
func sendJSON(w http.ResponseWriter, status int, data interface{}) {
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(status)
    json.NewEncoder(w).Encode(data)
}

func sendError(w http.ResponseWriter, status int, message string) {
    sendJSON(w, status, APIResponse{
        Success: false,
        Error:   message,
    })
}

func sendSuccess(w http.ResponseWriter, data interface{}) {
    sendJSON(w, http.StatusOK, APIResponse{
        Success: true,
        Data:    data,
    })
}

// Обработчики API

// GET /api/users - получить всех пользователей
func getUsersHandler(w http.ResponseWriter, r *http.Request) {
    if r.Method != "GET" {
        sendError(w, http.StatusMethodNotAllowed, "Only GET allowed")
        return
    }
    
    userList := make([]*User, 0, len(users))
    for _, user := range users {
        userList = append(userList, user)
    }
    
    sendSuccess(w, userList)
}

// GET /api/users/{id} - получить пользователя по ID
func getUserHandler(w http.ResponseWriter, r *http.Request) {
    if r.Method != "GET" {
        sendError(w, http.StatusMethodNotAllowed, "Only GET allowed")
        return
    }
    
    // Извлекаем ID из URL
    idStr := strings.TrimPrefix(r.URL.Path, "/api/users/")
    id, err := strconv.Atoi(idStr)
    if err != nil {
        sendError(w, http.StatusBadRequest, "Invalid user ID")
        return
    }
    
    user, exists := users[id]
    if !exists {
        sendError(w, http.StatusNotFound, "User not found")
        return
    }
    
    sendSuccess(w, user)
}

// POST /api/users - создать нового пользователя
func createUserHandler(w http.ResponseWriter, r *http.Request) {
    if r.Method != "POST" {
        sendError(w, http.StatusMethodNotAllowed, "Only POST allowed")
        return
    }
    
    var user User
    
    // Парсим JSON из тела запроса
    if err := json.NewDecoder(r.Body).Decode(&user); err != nil {
        sendError(w, http.StatusBadRequest, "Invalid JSON")
        return
    }
    
    // Валидация
    if user.Name == "" || user.Email == "" {
        sendError(w, http.StatusBadRequest, "Name and Email are required")
        return
    }
    
    // Создаем пользователя
    user.ID = nextUserID
    user.CreatedAt = time.Now()
    users[nextUserID] = &user
    nextUserID++
    
    sendSuccess(w, user)
}

// DELETE /api/users/{id} - удалить пользователя
func deleteUserHandler(w http.ResponseWriter, r *http.Request) {
    if r.Method != "DELETE" {
        sendError(w, http.StatusMethodNotAllowed, "Only DELETE allowed")
        return
    }
    
    idStr := strings.TrimPrefix(r.URL.Path, "/api/users/")
    id, err := strconv.Atoi(idStr)
    if err != nil {
        sendError(w, http.StatusBadRequest, "Invalid user ID")
        return
    }
    
    if _, exists := users[id]; !exists {
        sendError(w, http.StatusNotFound, "User not found")
        return
    }
    
    delete(users, id)
    sendSuccess(w, map[string]string{"message": "User deleted"})
}

// Маршрутизатор API
func apiRouter(w http.ResponseWriter, r *http.Request) {
    switch {
    case r.URL.Path == "/api/users":
        if r.Method == "GET" {
            getUsersHandler(w, r)
        } else if r.Method == "POST" {
            createUserHandler(w, r)
        } else {
            sendError(w, http.StatusMethodNotAllowed, "Method not allowed")
        }
    case strings.HasPrefix(r.URL.Path, "/api/users/"):
        if r.Method == "GET" {
            getUserHandler(w, r)
        } else if r.Method == "DELETE" {
            deleteUserHandler(w, r)
        } else {
            sendError(w, http.StatusMethodNotAllowed, "Method not allowed")
        }
    default:
        sendError(w, http.StatusNotFound, "Endpoint not found")
    }
}

func main() {
    http.HandleFunc("/api/", apiRouter)
    
    fmt.Println("API сервер запущен на http://localhost:8080")
    fmt.Println("Доступные эндпоинты:")
    fmt.Println("  GET    /api/users      - список пользователей")
    fmt.Println("  POST   /api/users      - создать пользователя")
    fmt.Println("  GET    /api/users/{id} - получить пользователя")
    fmt.Println("  DELETE /api/users/{id} - удалить пользователя")
    
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

---

## 📡 HTTP Клиент

### Простые HTTP запросы

```go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "io"
    "log"
    "net/http"
    "time"
)

// Структура для создания пользователя
type CreateUserRequest struct {
    Name  string `json:"name"`
    Email string `json:"email"`
}

type User struct {
    ID        int       `json:"id"`
    Name      string    `json:"name"`
    Email     string    `json:"email"`
    CreatedAt time.Time `json:"created_at"`
}

type APIResponse struct {
    Success bool        `json:"success"`
    Data    interface{} `json:"data,omitempty"`
    Error   string      `json:"error,omitempty"`
}

// HTTP клиент с кастомными настройками
func createHTTPClient() *http.Client {
    return &http.Client{
        Timeout: 30 * time.Second,
        Transport: &http.Transport{
            MaxIdleConns:        10,
            IdleConnTimeout:     60 * time.Second,
            DisableCompression:  false,
        },
    }
}

// GET запрос
func getUsers(client *http.Client, baseURL string) error {
    resp, err := client.Get(baseURL + "/api/users")
    if err != nil {
        return fmt.Errorf("ошибка GET запроса: %w", err)
    }
    defer resp.Body.Close()
    
    if resp.StatusCode != http.StatusOK {
        return fmt.Errorf("неожиданный статус: %d", resp.StatusCode)
    }
    
    body, err := io.ReadAll(resp.Body)
    if err != nil {
        return fmt.Errorf("ошибка чтения ответа: %w", err)
    }
    
    var apiResp APIResponse
    if err := json.Unmarshal(body, &apiResp); err != nil {
        return fmt.Errorf("ошибка парсинга JSON: %w", err)
    }
    
    fmt.Printf("✅ GET /api/users успешно\n")
    fmt.Printf("Response: %+v\n\n", apiResp)
    return nil
}

// POST запрос
func createUser(client *http.Client, baseURL string, name, email string) (*User, error) {
    requestData := CreateUserRequest{
        Name:  name,
        Email: email,
    }
    
    jsonData, err := json.Marshal(requestData)
    if err != nil {
        return nil, fmt.Errorf("ошибка сериализации: %w", err)
    }
    
    resp, err := client.Post(
        baseURL+"/api/users",
        "application/json",
        bytes.NewBuffer(jsonData),
    )
    if err != nil {
        return nil, fmt.Errorf("ошибка POST запроса: %w", err)
    }
    defer resp.Body.Close()
    
    body, err := io.ReadAll(resp.Body)
    if err != nil {
        return nil, fmt.Errorf("ошибка чтения ответа: %w", err)
    }
    
    var apiResp APIResponse
    if err := json.Unmarshal(body, &apiResp); err != nil {
        return nil, fmt.Errorf("ошибка парсинга JSON: %w", err)
    }
    
    if !apiResp.Success {
        return nil, fmt.Errorf("API ошибка: %s", apiResp.Error)
    }
    
    // Преобразуем interface{} в User
    userData, err := json.Marshal(apiResp.Data)
    if err != nil {
        return nil, fmt.Errorf("ошибка преобразования данных: %w", err)
    }
    
    var user User
    if err := json.Unmarshal(userData, &user); err != nil {
        return nil, fmt.Errorf("ошибка парсинга пользователя: %w", err)
    }
    
    fmt.Printf("✅ Пользователь создан: %+v\n\n", user)
    return &user, nil
}

// DELETE запрос
func deleteUser(client *http.Client, baseURL string, userID int) error {
    url := fmt.Sprintf("%s/api/users/%d", baseURL, userID)
    
    req, err := http.NewRequest("DELETE", url, nil)
    if err != nil {
        return fmt.Errorf("ошибка создания запроса: %w", err)
    }
    
    resp, err := client.Do(req)
    if err != nil {
        return fmt.Errorf("ошибка DELETE запроса: %w", err)
    }
    defer resp.Body.Close()
    
    if resp.StatusCode == http.StatusNotFound {
        return fmt.Errorf("пользователь не найден")
    }
    
    if resp.StatusCode != http.StatusOK {
        return fmt.Errorf("неожиданный статус: %d", resp.StatusCode)
    }
    
    fmt.Printf("✅ Пользователь %d удален\n\n", userID)
    return nil
}

func main() {
    baseURL := "http://localhost:8080"
    client := createHTTPClient()
    
    fmt.Println("🚀 Тестируем HTTP клиент")
    fmt.Println("Убедитесь, что сервер запущен на http://localhost:8080")
    fmt.Println()
    
    // 1. Получаем список пользователей
    fmt.Println("1. Получаем всех пользователей:")
    if err := getUsers(client, baseURL); err != nil {
        log.Printf("Ошибка получения пользователей: %v", err)
    }
    
    // 2. Создаем нового пользователя
    fmt.Println("2. Создаем нового пользователя:")
    user, err := createUser(client, baseURL, "Анна Сидорова", "anna@example.com")
    if err != nil {
        log.Printf("Ошибка создания пользователя: %v", err)
        return
    }
    
    // 3. Снова получаем список пользователей
    fmt.Println("3. Получаем обновленный список пользователей:")
    if err := getUsers(client, baseURL); err != nil {
        log.Printf("Ошибка получения пользователей: %v", err)
    }
    
    // 4. Удаляем созданного пользователя
    fmt.Println("4. Удаляем созданного пользователя:")
    if err := deleteUser(client, baseURL, user.ID); err != nil {
        log.Printf("Ошибка удаления пользователя: %v", err)
    }
    
    // 5. Проверяем, что пользователь удален
    fmt.Println("5. Финальный список пользователей:")
    if err := getUsers(client, baseURL); err != nil {
        log.Printf("Ошибка получения пользователей: %v", err)
    }
    
    fmt.Println("✨ Тестирование завершено!")
}
```

---

## ⚡ Middleware и продвинутые возможности

### HTTP сервер с middleware

```go
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "time"
)

// Middleware для логирования запросов
func loggingMiddleware(next http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        start := time.Now()
        
        // Вызываем следующий обработчик
        next(w, r)
        
        // Логируем после выполнения
        duration := time.Since(start)
        log.Printf("%s %s %v", r.Method, r.URL.Path, duration)
    }
}

// Middleware для CORS
func corsMiddleware(next http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        w.Header().Set("Access-Control-Allow-Origin", "*")
        w.Header().Set("Access-Control-Allow-Methods", "GET, POST, PUT, DELETE, OPTIONS")
        w.Header().Set("Access-Control-Allow-Headers", "Content-Type, Authorization")
        
        // Обрабатываем preflight запросы
        if r.Method == "OPTIONS" {
            w.WriteHeader(http.StatusOK)
            return
        }
        
        next(w, r)
    }
}

// Middleware для аутентификации (упрощенный)
func authMiddleware(next http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        token := r.Header.Get("Authorization")
        
        if token == "" {
            http.Error(w, "Authorization header required", http.StatusUnauthorized)
            return
        }
        
        if token != "Bearer secret123" {
            http.Error(w, "Invalid token", http.StatusUnauthorized)
            return
        }
        
        next(w, r)
    }
}

// Middleware для проверки Content-Type
func jsonMiddleware(next http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        if r.Method == "POST" || r.Method == "PUT" {
            contentType := r.Header.Get("Content-Type")
            if contentType != "application/json" {
                http.Error(w, "Content-Type must be application/json", http.StatusBadRequest)
                return
            }
        }
        
        next(w, r)
    }
}

// Композитная функция для применения множественных middleware
func applyMiddleware(handler http.HandlerFunc, middlewares ...func(http.HandlerFunc) http.HandlerFunc) http.HandlerFunc {
    for i := len(middlewares) - 1; i >= 0; i-- {
        handler = middlewares[i](handler)
    }
    return handler
}

// Обработчики
func publicHandler(w http.ResponseWriter, r *http.Request) {
    response := map[string]string{
        "message": "Это публичный эндпоинт",
        "time":    time.Now().Format(time.RFC3339),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

func protectedHandler(w http.ResponseWriter, r *http.Request) {
    response := map[string]string{
        "message": "Это защищенный эндпоинт",
        "user":    "authenticated_user",
        "time":    time.Now().Format(time.RFC3339),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

func dataHandler(w http.ResponseWriter, r *http.Request) {
    var data map[string]interface{}
    
    if err := json.NewDecoder(r.Body).Decode(&data); err != nil {
        http.Error(w, "Invalid JSON", http.StatusBadRequest)
        return
    }
    
    response := map[string]interface{}{
        "message":      "Данные получены",
        "received_data": data,
        "processed_at": time.Now().Format(time.RFC3339),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

func main() {
    // Публичный эндпоинт с базовыми middleware
    http.HandleFunc("/api/public", 
        applyMiddleware(publicHandler, loggingMiddleware, corsMiddleware))
    
    // Защищенный эндпоинт
    http.HandleFunc("/api/protected", 
        applyMiddleware(protectedHandler, loggingMiddleware, corsMiddleware, authMiddleware))
    
    // Эндпоинт для работы с JSON данными
    http.HandleFunc("/api/data", 
        applyMiddleware(dataHandler, loggingMiddleware, corsMiddleware, jsonMiddleware, authMiddleware))
    
    fmt.Println("🚀 Сервер с middleware запущен на http://localhost:8080")
    fmt.Println()
    fmt.Println("Доступные эндпоинты:")
    fmt.Println("  GET  /api/public    - публичный эндпоинт")
    fmt.Println("  GET  /api/protected - защищенный эндпоинт (требует Authorization: Bearer secret123)")
    fmt.Println("  POST /api/data      - обработка JSON данных (требует аутентификацию)")
    fmt.Println()
    fmt.Println("Примеры запросов:")
    fmt.Println("  curl http://localhost:8080/api/public")
    fmt.Println("  curl -H \"Authorization: Bearer secret123\" http://localhost:8080/api/protected")
    fmt.Println("  curl -X POST -H \"Authorization: Bearer secret123\" -H \"Content-Type: application/json\" -d '{\"name\":\"test\"}' http://localhost:8080/api/data")
    
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

---

## 🔧 Практический пример: Веб-сервис для заметок

```go
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "strconv"
    "strings"
    "time"
)

// Модели данных
type Note struct {
    ID          int       `json:"id"`
    Title       string    `json:"title"`
    Content     string    `json:"content"`
    Tags        []string  `json:"tags"`
    CreatedAt   time.Time `json:"created_at"`
    UpdatedAt   time.Time `json:"updated_at"`
}

type CreateNoteRequest struct {
    Title   string   `json:"title"`
    Content string   `json:"content"`
    Tags    []string `json:"tags"`
}

type UpdateNoteRequest struct {
    Title   *string  `json:"title,omitempty"`
    Content *string  `json:"content,omitempty"`
    Tags    []string `json:"tags,omitempty"`
}

// Хранилище заметок
type NotesStorage struct {
    notes  map[int]*Note
    nextID int
}

func NewNotesStorage() *NotesStorage {
    return &NotesStorage{
        notes:  make(map[int]*Note),
        nextID: 1,
    }
}

func (ns *NotesStorage) Create(req CreateNoteRequest) *Note {
    note := &Note{
        ID:        ns.nextID,
        Title:     req.Title,
        Content:   req.Content,
        Tags:      req.Tags,
        CreatedAt: time.Now(),
        UpdatedAt: time.Now(),
    }
    
    ns.notes[ns.nextID] = note
    ns.nextID++
    
    return note
}

func (ns *NotesStorage) GetAll() []*Note {
    notes := make([]*Note, 0, len(ns.notes))
    for _, note := range ns.notes {
        notes = append(notes, note)
    }
    return notes
}

func (ns *NotesStorage) GetByID(id int) (*Note, bool) {
    note, exists := ns.notes[id]
    return note, exists
}

func (ns *NotesStorage) Update(id int, req UpdateNoteRequest) (*Note, bool) {
    note, exists := ns.notes[id]
    if !exists {
        return nil, false
    }
    
    if req.Title != nil {
        note.Title = *req.Title
    }
    if req.Content != nil {
        note.Content = *req.Content
    }
    if req.Tags != nil {
        note.Tags = req.Tags
    }
    
    note.UpdatedAt = time.Now()
    return note, true
}

func (ns *NotesStorage) Delete(id int) bool {
    if _, exists := ns.notes[id]; !exists {
        return false
    }
    delete(ns.notes, id)
    return true
}

func (ns *NotesStorage) Search(query string) []*Note {
    var results []*Note
    query = strings.ToLower(query)
    
    for _, note := range ns.notes {
        if strings.Contains(strings.ToLower(note.Title), query) ||
           strings.Contains(strings.ToLower(note.Content), query) {
            results = append(results, note)
        }
        
        // Поиск по тегам
        for _, tag := range note.Tags {
            if strings.Contains(strings.ToLower(tag), query) {
                results = append(results, note)
                break
            }
        }
    }
    
    return results
}

// HTTP обработчики
type NotesHandler struct {
    storage *NotesStorage
}

func NewNotesHandler() *NotesHandler {
    return &NotesHandler{
        storage: NewNotesStorage(),
    }
}

func (nh *NotesHandler) sendJSON(w http.ResponseWriter, status int, data interface{}) {
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(status)
    json.NewEncoder(w).Encode(data)
}

func (nh *NotesHandler) sendError(w http.ResponseWriter, status int, message string) {
    nh.sendJSON(w, status, map[string]string{"error": message})
}

// GET /notes - получить все заметки или поиск
func (nh *NotesHandler) getNotes(w http.ResponseWriter, r *http.Request) {
    query := r.URL.Query().Get("search")
    
    var notes []*Note
    if query != "" {
        notes = nh.storage.Search(query)
    } else {
        notes = nh.storage.GetAll()
    }
    
    nh.sendJSON(w, http.StatusOK, map[string]interface{}{
        "notes": notes,
        "count": len(notes),
    })
}

// POST /notes - создать заметку
func (nh *NotesHandler) createNote(w http.ResponseWriter, r *http.Request) {
    var req CreateNoteRequest
    
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        nh.sendError(w, http.StatusBadRequest, "Invalid JSON")
        return
    }
    
    if req.Title == "" {
        nh.sendError(w, http.StatusBadRequest, "Title is required")
        return
    }
    
    note := nh.storage.Create(req)
    nh.sendJSON(w, http.StatusCreated, note)
}

// GET /notes/{id} - получить заметку по ID
func (nh *NotesHandler) getNote(w http.ResponseWriter, r *http.Request, id int) {
    note, exists := nh.storage.GetByID(id)
    if !exists {
        nh.sendError(w, http.StatusNotFound, "Note not found")
        return
    }
    
    nh.sendJSON(w, http.StatusOK, note)
}

// PUT /notes/{id} - обновить заметку
func (nh *NotesHandler) updateNote(w http.ResponseWriter, r *http.Request, id int) {
    var req UpdateNoteRequest
    
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        nh.sendError(w, http.StatusBadRequest, "Invalid JSON")
        return
    }
    
    note, exists := nh.storage.Update(id, req)
    if !exists {
        nh.sendError(w, http.StatusNotFound, "Note not found")
        return
    }
    
    nh.sendJSON(w, http.StatusOK, note)
}

// DELETE /notes/{id} - удалить заметку
func (nh *NotesHandler) deleteNote(w http.ResponseWriter, r *http.Request, id int) {
    if !nh.storage.Delete(id) {
        nh.sendError(w, http.StatusNotFound, "Note not found")
        return
    }
    
    nh.sendJSON(w, http.StatusOK, map[string]string{
        "message": "Note deleted successfully",
    })
}

// Главный роутер
func (nh *NotesHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    // CORS
    w.Header().Set("Access-Control-Allow-Origin", "*")
    w.Header().Set("Access-Control-Allow-Methods", "GET, POST, PUT, DELETE, OPTIONS")
    w.Header().Set("Access-Control-Allow-Headers", "Content-Type")
    
    if r.Method == "OPTIONS" {
        w.WriteHeader(http.StatusOK)
        return
    }
    
    // Логирование
    log.Printf("%s %s", r.Method, r.URL.Path)
    
    path := strings.TrimPrefix(r.URL.Path, "/notes")
    
    if path == "" || path == "/" {
        // /notes
        switch r.Method {
        case "GET":
            nh.getNotes(w, r)
        case "POST":
            nh.createNote(w, r)
        default:
            nh.sendError(w, http.StatusMethodNotAllowed, "Method not allowed")
        }
        return
    }
    
    // /notes/{id}
    if strings.HasPrefix(path, "/") {
        idStr := strings.TrimPrefix(path, "/")
        id, err := strconv.Atoi(idStr)
        if err != nil {
            nh.sendError(w, http.StatusBadRequest, "Invalid note ID")
            return
        }
        
        switch r.Method {
        case "GET":
            nh.getNote(w, r, id)
        case "PUT":
            nh.updateNote(w, r, id)
        case "DELETE":
            nh.deleteNote(w, r, id)
        default:
            nh.sendError(w, http.StatusMethodNotAllowed, "Method not allowed")
        }
        return
    }
    
    nh.sendError(w, http.StatusNotFound, "Endpoint not found")
}

func main() {
    handler := NewNotesHandler()
    
    // Создаем несколько тестовых заметок
    handler.storage.Create(CreateNoteRequest{
        Title:   "Изучить Go",
        Content: "Пройти учебник по Go и сделать несколько проектов",
        Tags:    []string{"programming", "golang", "learning"},
    })
    
    handler.storage.Create(CreateNoteRequest{
        Title:   "Купить продукты",
        Content: "Молоко, хлеб, яйца, сыр",
        Tags:    []string{"shopping", "food"},
    })
    
    http.Handle("/notes", handler)
    http.Handle("/notes/", handler)
    
    fmt.Println("🚀 Notes API запущен на http://localhost:8080")
    fmt.Println()
    fmt.Println("Доступные эндпоинты:")
    fmt.Println("  GET    /notes           - получить все заметки")
    fmt.Println("  GET    /notes?search=query - поиск заметок")
    fmt.Println("  POST   /notes           - создать заметку")
    fmt.Println("  GET    /notes/{id}      - получить заметку")
    fmt.Println("  PUT    /notes/{id}      - обновить заметку")
    fmt.Println("  DELETE /notes/{id}      - удалить заметку")
    fmt.Println()
    fmt.Println("Примеры запросов:")
    fmt.Println("  curl http://localhost:8080/notes")
    fmt.Println("  curl http://localhost:8080/notes/1")
    fmt.Println("  curl http://localhost:8080/notes?search=go")
    
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

---

## 🧠 Лучшие практики

### 1. Используй контексты для отмены запросов
```go
func makeRequestWithTimeout(url string) error {
    ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
    defer cancel()
    
    req, err := http.NewRequestWithContext(ctx, "GET", url, nil)
    if err != nil {
        return err
    }
    
    resp, err := http.DefaultClient.Do(req)
    if err != nil {
        return err
    }
    defer resp.Body.Close()
    
    return nil
}
```

### 2. Настраивай HTTP клиент
```go
func createOptimizedClient() *http.Client {
    return &http.Client{
        Timeout: 30 * time.Second,
        Transport: &http.Transport{
            MaxIdleConns:        100,
            MaxIdleConnsPerHost: 10,
            IdleConnTimeout:     90 * time.Second,
        },
    }
}
```

### 3. Валидируй входные данные
```go
func validateCreateUserRequest(req CreateUserRequest) error {
    if req.Name == "" {
        return errors.New("name is required")
    }
    if !strings.Contains(req.Email, "@") {
        return errors.New("invalid email format")
    }
    return nil
}
```

---

## 🧠 Проверь себя

- Как создать простейший HTTP сервер?
- Как отправить POST запрос с JSON данными?
- Что такое middleware и зачем оно нужно?
- Как правильно обрабатывать ошибки HTTP запросов?
- Как настроить таймауты для HTTP клиента?

---

## 📌 Главное из главы

- **http.ListenAndServe()** запускает HTTP сервер
- **http.HandleFunc()** регистрирует обработчики маршрутов
- **http.Get()**, **http.Post()** для простых HTTP запросов
- **http.Client** для настройки клиента (таймауты, пулы соединений)
- **Middleware** позволяет добавлять сквозную функциональность
- **JSON** кодирование/декодирование для API
- **Контексты** для отмены длительных операций

---