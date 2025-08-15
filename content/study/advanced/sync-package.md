---
title: "5.6 🔄 Синхронизация (sync пакет)"
description: "Синхронизация в Go: sync пакет, Mutex, RWMutex, WaitGroup, Once. Безопасная работа с разделяемыми данными."
keywords: ["sync golang", "mutex go", "синхронизация golang", "waitgroup go"]
date: 2025-08-14
weight: 56
draft: true
---

**Пакет sync** предоставляет примитивы синхронизации для безопасной работы с разделяемыми данными в многопоточных приложениях. Хотя в Go предпочтительнее использовать каналы, sync пакет незаменим для некоторых задач: защиты критических секций, одноразовой инициализации и группировки горутин.

> 💬 **Когда использовать sync?**  
> Каналы отлично подходят для передачи данных, но когда нужно защитить общее состояние (например, счетчик) или дождаться завершения группы горутин — sync пакет идеален!

---

## 🔒 Mutex - взаимное исключение

### Базовое использование Mutex

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

// Небезопасный счетчик
type UnsafeCounter struct {
    value int
}

func (c *UnsafeCounter) Increment() {
    c.value++
}

func (c *UnsafeCounter) Value() int {
    return c.value
}

// Безопасный счетчик с Mutex
type SafeCounter struct {
    mu    sync.Mutex
    value int
}

func (c *SafeCounter) Increment() {
    c.mu.Lock()         // Захватываем блокировку
    defer c.mu.Unlock() // Освобождаем блокировку при выходе
    c.value++
}

func (c *SafeCounter) Value() int {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.value
}

func testCounter(name string, counter interface{}) {
    var wg sync.WaitGroup
    start := time.Now()
    
    // Запускаем 1000 горутин, каждая увеличивает счетчик 1000 раз
    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            for j := 0; j < 1000; j++ {
                switch c := counter.(type) {
                case *UnsafeCounter:
                    c.Increment()
                case *SafeCounter:
                    c.Increment()
                }
            }
        }()
    }
    
    wg.Wait()
    duration := time.Since(start)
    
    var finalValue int
    switch c := counter.(type) {
    case *UnsafeCounter:
        finalValue = c.Value()
    case *SafeCounter:
        finalValue = c.Value()
    }
    
    fmt.Printf("%s:\n", name)
    fmt.Printf("  Ожидаемое значение: 1,000,000\n")
    fmt.Printf("  Реальное значение:  %,d\n", finalValue)
    fmt.Printf("  Время выполнения:   %v\n", duration)
    
    if finalValue == 1000000 {
        fmt.Printf("  ✅ Результат корректен\n\n")
    } else {
        fmt.Printf("  ❌ Race condition! Данные повреждены\n\n")
    }
}

func main() {
    fmt.Println("🔄 Сравнение безопасного и небезопасного доступа к данным")
    fmt.Println()
    
    // Тестируем небезопасный счетчик
    unsafeCounter := &UnsafeCounter{}
    testCounter("Небезопасный счетчик", unsafeCounter)
    
    // Тестируем безопасный счетчик
    safeCounter := &SafeCounter{}
    testCounter("Безопасный счетчик (Mutex)", safeCounter)
}
```

---

## 📖 RWMutex - читатели-писатели

```go
package main

import (
    "fmt"
    "math/rand"
    "sync"
    "time"
)

// Структура данных с поддержкой множественных читателей
type DataStore struct {
    mu   sync.RWMutex
    data map[string]string
}

func NewDataStore() *DataStore {
    return &DataStore{
        data: make(map[string]string),
    }
}

// Чтение (может выполняться параллельно)
func (ds *DataStore) Get(key string) (string, bool) {
    ds.mu.RLock()         // Блокировка только для чтения
    defer ds.mu.RUnlock()
    
    // Симулируем медленное чтение
    time.Sleep(10 * time.Millisecond)
    
    value, exists := ds.data[key]
    return value, exists
}

// Запись (эксклюзивная блокировка)
func (ds *DataStore) Set(key, value string) {
    ds.mu.Lock()         // Полная блокировка
    defer ds.mu.Unlock()
    
    // Симулируем медленную запись
    time.Sleep(50 * time.Millisecond)
    
    ds.data[key] = value
    fmt.Printf("📝 Записано: %s = %s\n", key, value)
}

// Получение всех ключей (чтение)
func (ds *DataStore) Keys() []string {
    ds.mu.RLock()
    defer ds.mu.RUnlock()
    
    keys := make([]string, 0, len(ds.data))
    for key := range ds.data {
        keys = append(keys, key)
    }
    
    return keys
}

// Безопасное обновление (если ключ существует)
func (ds *DataStore) Update(key string, updater func(string) string) bool {
    ds.mu.Lock()
    defer ds.mu.Unlock()
    
    if oldValue, exists := ds.data[key]; exists {
        newValue := updater(oldValue)
        ds.data[key] = newValue
        fmt.Printf("🔄 Обновлено: %s = %s -> %s\n", key, oldValue, newValue)
        return true
    }
    
    return false
}

func main() {
    store := NewDataStore()
    var wg sync.WaitGroup
    
    fmt.Println("🔄 Тест RWMutex с читателями и писателями")
    fmt.Println()
    
    // Предварительно добавляем данные
    initialData := map[string]string{
        "user1": "Alice",
        "user2": "Bob",
        "user3": "Charlie",
    }
    
    for key, value := range initialData {
        store.Set(key, value)
    }
    
    fmt.Println("✅ Начальные данные добавлены\n")
    
    start := time.Now()
    
    // Запускаем много читателей (они могут работать параллельно)
    for i := 0; i < 20; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            
            for j := 0; j < 5; j++ {
                keys := []string{"user1", "user2", "user3", "nonexistent"}
                key := keys[rand.Intn(len(keys))]
                
                value, exists := store.Get(key)
                if exists {
                    fmt.Printf("👀 Читатель %d: %s = %s\n", id, key, value)
                } else {
                    fmt.Printf("👀 Читатель %d: %s не найден\n", id, key)
                }
            }
        }(i)
    }
    
    // Запускаем несколько писателей (они будут ждать друг друга)
    for i := 0; i < 3; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            
            for j := 0; j < 2; j++ {
                key := fmt.Sprintf("writer%d_key%d", id, j)
                value := fmt.Sprintf("value_from_writer_%d", id)
                store.Set(key, value)
            }
        }(i)
    }
    
    // Запускаем процессы обновления
    for i := 0; i < 3; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            
            keys := []string{"user1", "user2", "user3"}
            key := keys[rand.Intn(len(keys))]
            
            store.Update(key, func(oldValue string) string {
                return fmt.Sprintf("%s_updated_by_%d", oldValue, id)
            })
        }(i)
    }
    
    wg.Wait()
    
    duration := time.Since(start)
    fmt.Printf("\n⏱️ Все операции завершены за %v\n", duration)
    
    // Показываем финальное состояние
    fmt.Println("\n📊 Финальное состояние:")
    keys := store.Keys()
    for _, key := range keys {
        value, _ := store.Get(key)
        fmt.Printf("  %s = %s\n", key, value)
    }
}
```

---

## 👥 WaitGroup - ожидание горутин

```go
package main

import (
    "fmt"
    "math/rand"
    "sync"
    "time"
)

// Задача для выполнения
type Task struct {
    ID   int
    Name string
    Duration time.Duration
}

// Worker выполняет задачи
func worker(id int, tasks <-chan Task, wg *sync.WaitGroup, results chan<- string) {
    defer wg.Done()
    
    fmt.Printf("🔧 Worker %d запущен\n", id)
    
    for task := range tasks {
        fmt.Printf("📋 Worker %d начинает задачу %d (%s)\n", id, task.ID, task.Name)
        
        // Симулируем выполнение задачи
        time.Sleep(task.Duration)
        
        result := fmt.Sprintf("Задача %d (%s) выполнена worker %d", task.ID, task.Name, id)
        results <- result
        
        fmt.Printf("✅ Worker %d завершил задачу %d\n", id, task.ID)
    }
    
    fmt.Printf("🏁 Worker %d завершен\n", id)
}

// Функция для создания случайных задач
func generateTasks(count int) []Task {
    tasks := make([]Task, count)
    taskNames := []string{
        "Обработка данных",
        "Генерация отчета",
        "Отправка email",
        "Резервное копирование",
        "Обновление кэша",
        "Валидация входных данных",
    }
    
    for i := 0; i < count; i++ {
        tasks[i] = Task{
            ID:       i + 1,
            Name:     taskNames[rand.Intn(len(taskNames))],
            Duration: time.Duration(rand.Intn(3)+1) * time.Second,
        }
    }
    
    return tasks
}

func main() {
    fmt.Println("👥 Пример использования WaitGroup")
    fmt.Println()
    
    const numWorkers = 3
    const numTasks = 10
    
    // Создаем каналы
    tasksChan := make(chan Task, numTasks)
    resultsChan := make(chan string, numTasks)
    
    // WaitGroup для ожидания завершения всех workers
    var workersWG sync.WaitGroup
    
    // Запускаем workers
    fmt.Printf("🚀 Запускаем %d workers\n\n", numWorkers)
    
    for i := 1; i <= numWorkers; i++ {
        workersWG.Add(1)
        go worker(i, tasksChan, &workersWG, resultsChan)
    }
    
    // Отправляем задачи
    tasks := generateTasks(numTasks)
    fmt.Printf("📤 Отправляем %d задач\n\n", numTasks)
    
    go func() {
        for _, task := range tasks {
            tasksChan <- task
        }
        close(tasksChan) // Закрываем канал, чтобы workers завершились
    }()
    
    // Собираем результаты в отдельной горутине
    var resultsWG sync.WaitGroup
    var results []string
    var resultsMutex sync.Mutex
    
    resultsWG.Add(1)
    go func() {
        defer resultsWG.Done()
        for i := 0; i < numTasks; i++ {
            result := <-resultsChan
            
            resultsMutex.Lock()
            results = append(results, result)
            resultsMutex.Unlock()
        }
    }()
    
    // Ждем завершения всех workers
    workersWG.Wait()
    fmt.Println("\n🏁 Все workers завершены")
    
    // Ждем получения всех результатов
    resultsWG.Wait()
    close(resultsChan)
    
    // Выводим результаты
    fmt.Println("\n📊 Результаты выполнения:")
    for i, result := range results {
        fmt.Printf("  %d. %s\n", i+1, result)
    }
    
    fmt.Println("\n✨ Все задачи выполнены!")
}
```

---

## 🔄 Once - одноразовая инициализация

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

// Глобальная переменная для инициализации
var (
    config     *AppConfig
    configOnce sync.Once
)

type AppConfig struct {
    DatabaseURL string
    APIKey      string
    Debug       bool
    LoadedAt    time.Time
}

// Инициализация конфигурации (вызовется только один раз)
func initConfig() {
    fmt.Println("🔧 Загружаем конфигурацию...")
    
    // Симулируем долгую инициализацию
    time.Sleep(2 * time.Second)
    
    config = &AppConfig{
        DatabaseURL: "postgresql://localhost:5432/mydb",
        APIKey:      "secret-api-key",
        Debug:       true,
        LoadedAt:    time.Now(),
    }
    
    fmt.Println("✅ Конфигурация загружена!")
}

// GetConfig возвращает конфигурацию (ленивая инициализация)
func GetConfig() *AppConfig {
    configOnce.Do(initConfig) // initConfig вызовется только один раз
    return config
}

// Сервис, который использует конфигурацию
func databaseService(id int, wg *sync.WaitGroup) {
    defer wg.Done()
    
    fmt.Printf("🗄️  Database Service %d: получаем конфигурацию...\n", id)
    
    cfg := GetConfig()
    
    fmt.Printf("🗄️  Database Service %d: подключение к %s\n", id, cfg.DatabaseURL)
    fmt.Printf("🗄️  Database Service %d: конфигурация загружена в %s\n", id, cfg.LoadedAt.Format("15:04:05"))
}

func apiService(id int, wg *sync.WaitGroup) {
    defer wg.Done()
    
    fmt.Printf("🌐 API Service %d: получаем конфигурацию...\n", id)
    
    cfg := GetConfig()
    
    fmt.Printf("🌐 API Service %d: используем API ключ %s\n", id, cfg.APIKey)
    fmt.Printf("🌐 API Service %d: режим отладки: %t\n", id, cfg.Debug)
}

// Singleton паттерн с Once
type Logger struct {
    level string
    file  string
}

var (
    logger     *Logger
    loggerOnce sync.Once
)

func GetLogger() *Logger {
    loggerOnce.Do(func() {
        fmt.Println("📝 Инициализируем логгер...")
        time.Sleep(1 * time.Second) // Симулируем инициализацию
        
        logger = &Logger{
            level: "INFO",
            file:  "/var/log/app.log",
        }
        fmt.Println("📝 Логгер инициализирован!")
    })
    return logger
}

func (l *Logger) Log(message string) {
    fmt.Printf("[%s] %s\n", l.level, message)
}

func serviceWithLogging(name string, wg *sync.WaitGroup) {
    defer wg.Done()
    
    log := GetLogger()
    log.Log(fmt.Sprintf("Сервис %s запущен", name))
    
    time.Sleep(500 * time.Millisecond)
    
    log.Log(fmt.Sprintf("Сервис %s выполняет работу", name))
    
    time.Sleep(500 * time.Millisecond)
    
    log.Log(fmt.Sprintf("Сервис %s завершен", name))
}

func main() {
    fmt.Println("🔄 Примеры использования sync.Once")
    fmt.Println()
    
    // Тест 1: Ленивая инициализация конфигурации
    fmt.Println("=== Тест 1: Инициализация конфигурации ===")
    
    var wg1 sync.WaitGroup
    
    // Запускаем несколько сервисов одновременно
    for i := 1; i <= 3; i++ {
        wg1.Add(1)
        go databaseService(i, &wg1)
    }
    
    for i := 1; i <= 2; i++ {
        wg1.Add(1)
        go apiService(i, &wg1)
    }
    
    wg1.Wait()
    fmt.Println()
    
    // Тест 2: Singleton паттерн с логгером
    fmt.Println("=== Тест 2: Singleton логгер ===")
    
    var wg2 sync.WaitGroup
    services := []string{"UserService", "OrderService", "PaymentService"}
    
    for _, serviceName := range services {
        wg2.Add(1)
        go serviceWithLogging(serviceName, &wg2)
    }
    
    wg2.Wait()
    
    fmt.Println("\n✨ Все тесты завершены!")
    
    // Проверяем, что объекты создались только один раз
    fmt.Println("\n🔍 Проверка результатов:")
    fmt.Printf("Config pointer: %p\n", config)
    fmt.Printf("Logger pointer: %p\n", logger)
    
    // Еще один вызов - объекты не пересоздаются
    cfg2 := GetConfig()
    log2 := GetLogger()
    fmt.Printf("Config pointer (2nd call): %p\n", cfg2)
    fmt.Printf("Logger pointer (2nd call): %p\n", log2)
    
    if config == cfg2 && logger == log2 {
        fmt.Println("✅ Объекты не пересоздавались - sync.Once работает корректно!")
    }
}
```

---

## 🎯 Практический пример: Thread-safe кэш

```go
package main

import (
    "fmt"
    "math/rand"
    "sync"
    "time"
)

// Элемент кэша
type CacheItem struct {
    Value     interface{}
    ExpiresAt time.Time
}

func (ci *CacheItem) IsExpired() bool {
    return time.Now().After(ci.ExpiresAt)
}

// Thread-safe кэш с TTL
type Cache struct {
    mu    sync.RWMutex
    items map[string]*CacheItem
    
    // Статистика
    hits   int64
    misses int64
    
    // Для автоочистки
    stopCleanup chan bool
    cleanupOnce sync.Once
}

func NewCache() *Cache {
    cache := &Cache{
        items:       make(map[string]*CacheItem),
        stopCleanup: make(chan bool),
    }
    
    // Запускаем автоочистку
    cache.startCleanup()
    
    return cache
}

// Запуск автоочистки (только один раз)
func (c *Cache) startCleanup() {
    c.cleanupOnce.Do(func() {
        go c.cleanupLoop()
    })
}

// Цикл автоочистки истекших элементов
func (c *Cache) cleanupLoop() {
    ticker := time.NewTicker(5 * time.Second)
    defer ticker.Stop()
    
    for {
        select {
        case <-ticker.C:
            c.removeExpired()
        case <-c.stopCleanup:
            return
        }
    }
}

// Удаление истекших элементов
func (c *Cache) removeExpired() {
    c.mu.Lock()
    defer c.mu.Unlock()
    
    expired := make([]string, 0)
    
    for key, item := range c.items {
        if item.IsExpired() {
            expired = append(expired, key)
        }
    }
    
    for _, key := range expired {
        delete(c.items, key)
    }
    
    if len(expired) > 0 {
        fmt.Printf("🗑️  Удалено %d истекших элементов\n", len(expired))
    }
}

// Установка значения
func (c *Cache) Set(key string, value interface{}, ttl time.Duration) {
    c.mu.Lock()
    defer c.mu.Unlock()
    
    c.items[key] = &CacheItem{
        Value:     value,
        ExpiresAt: time.Now().Add(ttl),
    }
    
    fmt.Printf("💾 Добавлено в кэш: %s\n", key)
}

// Получение значения
func (c *Cache) Get(key string) (interface{}, bool) {
    c.mu.RLock()
    defer c.mu.RUnlock()
    
    item, exists := c.items[key]
    if !exists || item.IsExpired() {
        c.misses++
        return nil, false
    }
    
    c.hits++
    return item.Value, true
}

// Удаление элемента
func (c *Cache) Delete(key string) {
    c.mu.Lock()
    defer c.mu.Unlock()
    
    delete(c.items, key)
    fmt.Printf("🗑️  Удалено из кэша: %s\n", key)
}

// Получение размера кэша
func (c *Cache) Size() int {
    c.mu.RLock()
    defer c.mu.RUnlock()
    
    return len(c.items)
}

// Статистика кэша
func (c *Cache) Stats() (hits, misses int64, hitRatio float64) {
    c.mu.RLock()
    defer c.mu.RUnlock()
    
    hits = c.hits
    misses = c.misses
    total := hits + misses
    
    if total > 0 {
        hitRatio = float64(hits) / float64(total) * 100
    }
    
    return
}

// Закрытие кэша
func (c *Cache) Close() {
    close(c.stopCleanup)
}

// Симулируем работу с базой данных
func fetchFromDatabase(key string) (string, time.Duration) {
    // Симулируем задержку БД
    delay := time.Duration(rand.Intn(1000)) * time.Millisecond
    time.Sleep(delay)
    
    return fmt.Sprintf("data_for_%s", key), delay
}

// Сервис, который использует кэш
func dataService(cache *Cache, serviceID int, wg *sync.WaitGroup) {
    defer wg.Done()
    
    keys := []string{"user:123", "user:456", "user:789", "product:1", "product:2"}
    
    for i := 0; i < 10; i++ {
        key := keys[rand.Intn(len(keys))]
        
        // Сначала проверяем кэш
        if value, found := cache.Get(key); found {
            fmt.Printf("🟢 Service %d: CACHE HIT для %s = %v\n", serviceID, key, value)
        } else {
            fmt.Printf("🔴 Service %d: CACHE MISS для %s\n", serviceID, key)
            
            // Получаем из "базы данных"
            data, dbDelay := fetchFromDatabase(key)
            
            // Сохраняем в кэш на 10 секунд
            cache.Set(key, data, 10*time.Second)
            
            fmt.Printf("🟡 Service %d: загружено из БД за %v: %s = %s\n", 
                serviceID, dbDelay, key, data)
        }
        
        time.Sleep(500 * time.Millisecond)
    }
}

func main() {
    fmt.Println("🎯 Thread-safe кэш с TTL")
    fmt.Println()
    
    cache := NewCache()
    defer cache.Close()
    
    var wg sync.WaitGroup
    
    // Запускаем несколько сервисов, которые конкурентно используют кэш
    for i := 1; i <= 3; i++ {
        wg.Add(1)
        go dataService(cache, i, &wg)
    }
    
    // Мониторим статистику в отдельной горутине
    wg.Add(1)
    go func() {
        defer wg.Done()
        
        ticker := time.NewTicker(3 * time.Second)
        defer ticker.Stop()
        
        for i := 0; i < 5; i++ {
            <-ticker.C
            
            hits, misses, hitRatio := cache.Stats()
            size := cache.Size()
            
            fmt.Printf("\n📊 Статистика кэша:\n")
            fmt.Printf("   Размер: %d элементов\n", size)
            fmt.Printf("   Попадания: %d\n", hits)
            fmt.Printf("   Промахи: %d\n", misses)
            fmt.Printf("   Коэффициент попаданий: %.2f%%\n\n", hitRatio)
        }
    }()
    
    wg.Wait()
    
    // Финальная статистика
    hits, misses, hitRatio := cache.Stats()
    fmt.Printf("🏁 Финальная статистика:\n")
    fmt.Printf("   Попадания: %d\n", hits)
    fmt.Printf("   Промахи: %d\n", misses)
    fmt.Printf("   Коэффициент попаданий: %.2f%%\n", hitRatio)
}
```

---

## 🛡️ Лучшие практики sync

### 1. Всегда используй defer с Unlock
```go
func (c *Counter) Increment() {
    c.mu.Lock()
    defer c.mu.Unlock() // ✅ Гарантированно разблокируется
    c.value++
}
```

### 2. Используй RWMutex для данных с частым чтением
```go
type Cache struct {
    mu   sync.RWMutex // ✅ Позволяет множественное чтение
    data map[string]interface{}
}
```

### 3. Избегай блокировок внутри блокировок
```go
// ❌ Опасно - может вызвать deadlock
func (c *Cache) BadMethod() {
    c.mu.Lock()
    defer c.mu.Unlock()
    
    c.AnotherMethod() // Если AnotherMethod тоже захватывает блокировку
}
```

### 4. Всегда закрывай ресурсы
```go
func (c *Cache) Close() {
    close(c.stopCleanup) // ✅ Завершаем горутину очистки
}
```

---

## 🧠 Проверь себя

- В чем разница между Mutex и RWMutex?
- Зачем использовать WaitGroup?
- Когда применять sync.Once?
- Как избежать deadlock при работе с мьютексами?
- Почему важно использовать defer с Unlock()?

---

## 📌 Главное из главы

- **sync.Mutex** для взаимного исключения при записи
- **sync.RWMutex** для множественных читателей и эксклюзивной записи
- **sync.WaitGroup** для ожидания завершения группы горутин
- **sync.Once** для одноразовой инициализации
- **defer unlock** всегда используй для гарантированного освобождения
- **RWMutex лучше Mutex** когда чтение происходит чаще записи
- **Предпочитай каналы**, но sync незаменим для защиты состояния

---