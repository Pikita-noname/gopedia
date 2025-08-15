---
title: "4.1 📡 Работа с JSON и API"
description: "Работа с JSON в Go: encoding/json, Marshal/Unmarshal, API интеграция, HTTP запросы с JSON данными."
keywords: ["json golang", "marshal unmarshal go", "api golang", "http json go"]
date: 2025-08-14
weight: 41
draft: false
---

**Работа с JSON в Go** — один из самых востребованных навыков в современной разработке. JSON (JavaScript Object Notation) стал универсальным языком обмена данными между приложениями, API и сервисами. В Go есть мощная встроенная поддержка JSON, которая делает работу с этим форматом простой и эффективной.

---
## 🎯 **Зачем Go разработчику JSON?**

В современных Go проектах JSON используется повсеместно:

**REST API разработка**: Практически все API используют JSON для передачи данных  
**Микросервисная архитектура**: JSON — стандарт межсервисного взаимодействия  
**Конфигурационные файлы**: Многие приложения хранят настройки в JSON  
**Database интеграция**: NoSQL базы как MongoDB работают с JSON документами  
**Frontend интеграция**: Все современные фронтенды понимают JSON  
**Внешние API**: GitHub, Stripe, Slack — все используют JSON в своих API  

Гo особенно популярен для создания API именно благодаря отличной поддержке JSON. Многие компании выбирают Go для backend разработки из-за простоты работы с JSON.

---
## 📚 **Пакет encoding/json: полный набор инструментов**

### **Ключевые функции пакета**
**Сериализация (Marshal)** — преобразование Go структур в JSON  
**Десериализация (Unmarshal)** — парсинг JSON в Go структуры  
**Streaming API** — работа с большими JSON файлами  
**Custom marshaling** — настройка процесса сериализации  
**JSON теги** — контроль над форматом вывода  

### **Философия JSON в Go**
В Go подход к JSON прагматичный и типобезопасный. В отличие от динамических языков, где JSON мапится в универсальные объекты, Go требует четкого определения структуры данных. Это дает:
- Compile-time проверки
- Лучшую производительность
- Понятность кода
- Автодополнение в IDE

---

## 🔄 **Основы сериализации и десериализации**

### **Marshal: от Go к JSON**

Маршалинг — это процесс преобразования Go значений в JSON формат. Это фундаментальная операция для создания API ответов.

**Простейший пример:**

```go
type User struct {
    ID       int    `json:"id"`
    Name     string `json:"name"`
    Email    string `json:"email"`
    IsActive bool   `json:"is_active"`
}

func main() {
    user := User{ID: 123, Name: "Иван", Email: "ivan@example.com", IsActive: true}
    
    jsonData, err := json.Marshal(user)
    if err != nil {
        log.Fatal("Ошибка сериализации:", err)
    }
    
    fmt.Printf("JSON: %s\n", jsonData)
    // Выход: {"id":123,"name":"Иван","email":"ivan@example.com","is_active":true}
}
```

**Что происходит при маршалинге:**
- Go берет каждое поле структуры
- Применяет правила преобразования типов (int→number, string→string, bool→boolean)
- Использует JSON теги для названий полей
- Создает валидный JSON документ

**Форматированный вывод для отладки:**
`json.MarshalIndent()` создает читаемый JSON с отступами — отлично для логов и отладки.

### **Unmarshal: от JSON к Go**

Анмаршалинг — обратный процесс парсинга JSON в Go структуры. Это основа для обработки API запросов.

```go
func parseProductJSON() {
    jsonData := `{"id": 1, "name": "Ноутбук", "price": 45000.50, "in_stock": true}`
    
    var product Product
    err := json.Unmarshal([]byte(jsonData), &product)
    if err != nil {
        log.Fatal("Ошибка парсинга:", err)
    }
    
    fmt.Printf("ID: %d, Название: %s, Цена: %.2f\n", 
        product.ID, product.Name, product.Price)
}
```

**Ключевые моменты анмаршалинга:**
- JSON парсится строго по структуре — лишние поля игнорируются
- Отсутствующие поля получают zero values
- Типы должны совпадать (string в JSON не парсится в int в Go)
- Указатель на структуру обязателен (`&product`)

**Типичные ошибки парсинга:**
- Неверный JSON синтаксис
- Несовпадение типов данных  
- Отсутствие указателя при анмаршалинге
- Попытка парсинга в nil указатель

---

## 🏷️ **JSON теги: контроль над сериализацией**

JSON теги — мощный механизм управления тем, как Go структуры преобразуются в JSON и обратно.

### **Основные опции JSON тегов**

**Переименование полей**: `json:"custom_name"` меняет имя поля в JSON  
**Пропуск полей**: `json:"-"` полностью исключает поле из JSON  
**Условное включение**: `json:"field,omitempty"` исключает пустые значения  
**String опция**: `json:"id,string"` принудительно сериализует как строку  

### **Практический пример управления полями**

```go
type APIUser struct {
    ID       int    `json:"user_id"`           // Переименовано
    Username string `json:"username"`          // Стандартное имя
    Password string `json:"-"`                 // Исключено из JSON
    Email    string `json:"email,omitempty"`   // Исключается если пустое
    IsAdmin  bool   `json:"is_admin,string"`   // Сериализуется как "true"/"false"
}
```

**Когда использовать omitempty:**
- Опциональные поля API
- Поля с zero values, которые не должны попадать в JSON
- Оптимизация размера JSON
- Совместимость с внешними API

### **Работа с приватными полями**

Иногда нужно контролировать сериализацию приватных полей или добавлять вычисляемые поля. Для этого используются кастомные методы `MarshalJSON` и `UnmarshalJSON`.

---

## 🌐 **HTTP + JSON: создание API клиентов**

Комбинация HTTP и JSON — основа современного API взаимодействия. Go предоставляет все необходимые инструменты для создания надежных API клиентов.

### **Анатомия HTTP + JSON запроса**

**Этапы GET запроса:**
1. Формирование URL с параметрами
2. Отправка HTTP запроса
3. Проверка статус кода ответа
4. Чтение тела ответа
5. Парсинг JSON в Go структуры
6. Обработка ошибок на каждом этапе

### **Базовый GET запрос с JSON**

```go
type Post struct {
    UserID int    `json:"userId"`
    ID     int    `json:"id"`
    Title  string `json:"title"`
    Body   string `json:"body"`
}

func fetchPost(postID int) (*Post, error) {
    url := fmt.Sprintf("https://jsonplaceholder.typicode.com/posts/%d", postID)
    
    resp, err := http.Get(url)
    if err != nil {
        return nil, fmt.Errorf("ошибка запроса: %w", err)
    }
    defer resp.Body.Close()
    
    if resp.StatusCode != http.StatusOK {
        return nil, fmt.Errorf("неожиданный статус: %d", resp.StatusCode)
    }
    
    var post Post
    if err := json.NewDecoder(resp.Body).Decode(&post); err != nil {
        return nil, fmt.Errorf("ошибка парсинга JSON: %w", err)
    }
    
    return &post, nil
}
```

**Важные детали реализации:**
- **defer resp.Body.Close()** — обязательно закрываем тело ответа
- **Проверка статус кода** — не все ответы содержат валидные данные
- **json.NewDecoder** — более эффективен для прямого чтения из io.Reader
- **Wrapped errors** — используем `fmt.Errorf` с `%w` для лучшего error handling

### **POST запросы с JSON payload**

POST запросы требуют отправки JSON данных в теле запроса:

```go
type CreatePostRequest struct {
    Title  string `json:"title"`
    Body   string `json:"body"`
    UserID int    `json:"userId"`
}

func createPost(title, body string, userID int) error {
    requestData := CreatePostRequest{
        Title: title, Body: body, UserID: userID,
    }
    
    jsonData, err := json.Marshal(requestData)
    if err != nil {
        return fmt.Errorf("ошибка сериализации: %w", err)
    }
    
    resp, err := http.Post(
        "https://jsonplaceholder.typicode.com/posts",
        "application/json",
        bytes.NewBuffer(jsonData),
    )
    if err != nil {
        return fmt.Errorf("ошибка запроса: %w", err)
    }
    defer resp.Body.Close()
    
    fmt.Printf("Пост создан, статус: %d\n", resp.StatusCode)
    return nil
}
```

**Особенности POST запросов с JSON:**
- **Content-Type: application/json** — обязательный заголовок
- **Тело запроса** — JSON данные в виде bytes.Buffer
- **Структуры request/response** — четкое разделение входных и выходных данных
- **Валидация данных** — проверка обязательных полей перед отправкой

---

## 🔧 **Продвинутые техники работы с JSON**

### **Динамический JSON: работа с неизвестной структурой**

В реальных проектах часто приходится работать с JSON неизвестной или изменяющейся структуры. Go предоставляет несколько подходов для таких ситуаций.

**Основные сценарии динамического JSON:**
- API от третьих сторон с нестабильной схемой
- Конфигурационные файлы с опциональными полями
- Webhook payloads от различных сервисов
- JSON из баз данных с flexible схемой

### **Подход 1: map[string]interface{}**

```go
func parseDynamicJSON(jsonData string) {
    var data map[string]interface{}
    
    err := json.Unmarshal([]byte(jsonData), &data)
    if err != nil {
        log.Fatal("Ошибка парсинга:", err)
    }
    
    // Type assertions для безопасного доступа
    if name, ok := data["name"].(string); ok {
        fmt.Printf("Имя: %s\n", name)
    }
    
    if age, ok := data["age"].(float64); ok { // JSON числа всегда float64!
        fmt.Printf("Возраст: %.0f\n", age)
    }
}
```

**Важные моменты type assertions:**
- Все JSON числа в Go становятся `float64`
- JSON массивы становятся `[]interface{}`
- JSON объекты становятся `map[string]interface{}`
- Всегда проверяйте успешность assertion через второе возвращаемое значение

### **Подход 2: Частично определенные структуры**

Когда часть JSON схемы известна, можно использовать комбинированный подход:

### **Кастомная сериализация: полный контроль над JSON**

Иногда стандартная сериализация Go не подходит. Например, нужно особое форматирование дат, вычисляемые поля, или совместимость с legacy API.

**Когда нужна кастомная сериализация:**
- Специальные форматы дат/времени
- Вычисляемые поля в JSON
- Сложная логика валидации
- Совместимость со старыми API
- Оптимизация размера JSON

### **Пример: кастомный формат даты**

```go
type CustomDate time.Time

// Сериализуем дату в формате YYYY-MM-DD
func (cd CustomDate) MarshalJSON() ([]byte, error) {
    formatted := time.Time(cd).Format("2006-01-02")
    return json.Marshal(formatted)
}

// Парсим дату из формата YYYY-MM-DD
func (cd *CustomDate) UnmarshalJSON(data []byte) error {
    var dateStr string
    if err := json.Unmarshal(data, &dateStr); err != nil {
        return err
    }
    
    parsedTime, err := time.Parse("2006-01-02", dateStr)
    if err != nil {
        return err
    }
    
    *cd = CustomDate(parsedTime)
    return nil
}
```

**Ключевые принципы кастомных методов:**
- `MarshalJSON() ([]byte, error)` — для сериализации
- `UnmarshalJSON([]byte) error` — для десериализации
- Метод должен быть на типе, а не на указателе (кроме UnmarshalJSON)
- Всегда обрабатывайте ошибки корректно

---

## 🛠️ **Практический пример: API клиент для GitHub**

Давайте создадим реальный API клиент, демонстрирующий все изученные концепции. Этот пример показывает профессиональный подход к созданию HTTP + JSON клиентов.

### **Архитектура API клиента**

**Ключевые принципы проектирования:**
- Инкапсуляция HTTP логики в отдельном типе
- Четкие структуры для всех API ответов
- Centralized error handling
- Configurability (timeouts, base URL, etc.)
- Rate limiting awareness

```go
type GitHubClient struct {
    baseURL string
    client  *http.Client
    token   string // Для аутентификации
}

type Repository struct {
    ID          int    `json:"id"`
    Name        string `json:"name"`
    FullName    string `json:"full_name"`
    Description string `json:"description"`
    Language    string `json:"language"`
    Stars       int    `json:"stargazers_count"`
    Forks       int    `json:"forks_count"`
    Private     bool   `json:"private"`
}

func NewGitHubClient(token string) *GitHubClient {
    return &GitHubClient{
        baseURL: "https://api.github.com",
        client: &http.Client{
            Timeout: 30 * time.Second,
        },
        token: token,
    }
}

func (g *GitHubClient) GetRepository(owner, repo string) (*Repository, error) {
    url := fmt.Sprintf("%s/repos/%s/%s", g.baseURL, owner, repo)
    
    req, err := http.NewRequest("GET", url, nil)
    if err != nil {
        return nil, fmt.Errorf("создание запроса: %w", err)
    }
    
    // Добавляем заголовки
    req.Header.Set("Accept", "application/vnd.github.v3+json")
    if g.token != "" {
        req.Header.Set("Authorization", "token "+g.token)
    }
    
    resp, err := g.client.Do(req)
    if err != nil {
        return nil, fmt.Errorf("выполнение запроса: %w", err)
    }
    defer resp.Body.Close()
    
    if resp.StatusCode == http.StatusNotFound {
        return nil, fmt.Errorf("репозиторий %s/%s не найден", owner, repo)
    }
    
    var repository Repository
    if err := json.NewDecoder(resp.Body).Decode(&repository); err != nil {
        return nil, fmt.Errorf("парсинг JSON: %w", err)
    }
    
    return &repository, nil
}
```

**Что делает этот код профессиональным:**
- Настраиваемый HTTP клиент с таймаутами
- Правильные HTTP заголовки для GitHub API
- Аутентификация через токен
- Специфичные ошибки для разных статус кодов
- Прямое декодирование из response body

---

## 💡 **Лучшие практики JSON + HTTP в Go**

### **1. Безопасность и валидация**

**Всегда валидируйте входные данные:**
```go
type User struct {
    Email string `json:"email"`
    Age   int    `json:"age"`
}

func (u *User) Validate() error {
    if !strings.Contains(u.Email, "@") {
        return errors.New("неверный формат email")
    }
    if u.Age < 0 || u.Age > 120 {
        return errors.New("неверный возраст")
    }
    return nil
}
```

**Принципы безопасности:**
- Никогда не доверяйте внешним данным
- Валидируйте все поля перед обработкой
- Используйте белые списки, а не черные
- Ограничивайте размеры входных данных

### **2. Производительность и ресурсы**

**Контролируйте таймауты:**
```go
client := &http.Client{
    Timeout: 10 * time.Second,
}
```

**Используйте контексты для отмены:**
```go
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

req, _ := http.NewRequestWithContext(ctx, "GET", url, nil)
```

### **3. Обработка ошибок**

**Различайте типы ошибок:**
- Сетевые ошибки (таймауты, DNS)
- HTTP ошибки (4xx, 5xx статусы)
- Ошибки парсинга JSON
- Бизнес-логические ошибки

**Создавайте информативные сообщения об ошибках:**
```go
if resp.StatusCode >= 400 {
    return fmt.Errorf("API error %d: %s", resp.StatusCode, resp.Status)
}
```

### **4. Структурная организация**

- Выделяйте HTTP клиенты в отдельные пакеты
- Группируйте связанные структуры данных
- Используйте интерфейсы для mock-тестирования
- Инкапсулируйте authentication логику

---

## 🚀 **Что изучать дальше**

После освоения основ JSON и HTTP API в Go:

1. **HTTP middleware** — промежуточное ПО для обработки запросов
2. **GraphQL клиенты** — альтернатива REST API
3. **WebSocket** — real-time коммуникации
4. **gRPC** — высокопроизводительное RPC
5. **OpenAPI/Swagger** — документирование API
6. **Circuit breakers** — устойчивость к сбоям внешних сервисов
7. **Rate limiting** — ограничение частоты запросов

---

## 🔍 **Проверь себя**

1. В чем разница между `json.Marshal()` и `json.NewEncoder()`?
2. Когда использовать `omitempty` в JSON тегах?
3. Как правильно обработать HTTP ошибку 404?
4. Почему JSON числа в Go всегда становятся `float64`?
5. Когда нужны кастомные `MarshalJSON` методы?

---

## 📌 **Главное из главы**

- **encoding/json — мощный инструмент** встроенный в Go стандартную библиотеку
- **Marshal/Unmarshal — основа** для преобразования между Go и JSON
- **JSON теги дают полный контроль** над форматом сериализации
- **HTTP + JSON = современные API** — комбинация для веб-разработки
- **Type safety важнее гибкости** — Go подход к обработке JSON
- **Всегда обрабатывайте ошибки** — сеть и парсинг могут давать сбои
- **Валидация критична** — не доверяйте внешним данным

JSON и HTTP API — фундамент современной разработки на Go. Освоив эти навыки, вы сможете создавать надежные веб-сервисы и интегрироваться с любыми внешними API!

---