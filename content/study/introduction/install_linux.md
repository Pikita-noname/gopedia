---
title: "🐧 Установка Go на Linux"
description: "Пошаговое руководство по установке Go/Golang на Linux Ubuntu, CentOS, Debian. Настройка GOPATH, GOROOT, проверка установки."
keywords: ["установка go linux", "golang ubuntu", "go centos", "настройка gopath"]
date: 2024-01-01T12:03:00+03:00
weight: 12
draft: false
hidden: true
---

Установка Go на Linux зависит от дистрибутива. Мы рассмотрим самые популярные варианты. Выбери свой дистрибутив и следуй инструкциям.

> **Важно**: Инструкции предполагают, что ты используешь терминал. Если ты не знаком с командами Linux, не переживай — мы объясним всё пошагово.

---

### 🔹 Установка Go на Ubuntu и Debian

Ubuntu и Debian используют пакетный менеджер `apt`, но мы рекомендуем устанавливать Go напрямую с официального сайта, чтобы получить последнюю версию.

1. **Скачай Go**:
   - Перейди на официальный сайт Go: [https://go.dev/dl/](https://go.dev/dl/).
   - Найди последнюю версию (например, `go1.22.3.linux-amd64.tar.gz` для 64-битных систем).
   - Скачай архив в терминале:
     ```bash
     wget https://go.dev/dl/go1.22.3.linux-amd64.tar.gz
     ```

2. **Распакуй архив**:
   - Распакуй скачанный файл в `/usr/local`:
     ```bash
     sudo tar -C /usr/local -xzf go1.22.3.linux-amd64.tar.gz
     ```

3. **Настрой переменные окружения**:
   - Открой файл `~/.bashrc` или `~/.zshrc` (в зависимости от используемой оболочки) в редакторе, например:
     ```bash
     nano ~/.bashrc
     ```
   - Добавь следующие строки в конец файла:
     ```bash
     export PATH=$PATH:/usr/local/go/bin
     export GOPATH=$HOME/go
     export PATH=$PATH:$GOPATH/bin
     ```
   - Сохрани файл и обнови конфигурацию:
     ```bash
     source ~/.bashrc
     ```

4. **Проверь установку**:
   - Введи в терминале:
     ```bash
     go version
     ```
   - Ты должен увидеть что-то вроде:
     ```
     go version go1.22.3 linux/amd64
     ```

---

### 🔹 Установка Go на Fedora

Fedora использует пакетный менеджер `dnf`, и Go доступен в официальных репозиториях, но мы установим последнюю версию с сайта Go.

1. **Скачай Go**:
   - Перейди на [https://go.dev/dl/](https://go.dev/dl/) и выбери последнюю версию (например, `go1.22.3.linux-amd64.tar.gz`).
   - Скачай архив:
     ```bash
     wget https://go.dev/dl/go1.22.3.linux-amd64.tar.gz
     ```

2. **Распакуй архив**:
   - Распакуй в `/usr/local`:
     ```bash
     sudo tar -C /usr/local -xzf go1.22.3.linux-amd64.tar.gz
     ```

3. **Настрой переменные окружения**:
   - Открой файл `~/.bashrc` или `~/.zshrc`:
     ```bash
     nano ~/.bashrc
     ```
   - Добавь:
     ```bash
     export PATH=$PATH:/usr/local/go/bin
     export GOPATH=$HOME/go
     export PATH=$PATH:$GOPATH/bin
     ```
   - Примени изменения:
     ```bash
     source ~/.bashrc
     ```

4. **Проверь установку**:
   - Выполни:
     ```bash
     go version
     ```
   - Ожидаемый вывод:
     ```
     go version go1.22.3 linux/amd64
     ```

---

### 🔹 Установка Go на Arch Linux

Arch Linux предоставляет Go через пакетный менеджер `pacman`, но мы также можем установить последнюю версию вручную.

1. **Установка через `pacman`** (рекомендуемый способ):
   - Обнови систему и установи Go:
     ```bash
     sudo pacman -Syu
     sudo pacman -S go
     ```

2. **Альтернатива: установка вручную**:
   - Скачай Go с [https://go.dev/dl/](https://go.dev/dl/) (например, `go1.22.3.linux-amd64.tar.gz`):
     ```bash
     wget https://go.dev/dl/go1.22.3.linux-amd64.tar.gz
     ```
   - Распакуй:
     ```bash
     sudo tar -C /usr/local -xzf go1.22.3.linux-amd64.tar.gz
     ```
   - Настрой переменные окружения в `~/.bashrc` или `~/.zshrc`:
     ```bash
     export PATH=$PATH:/usr/local/go/bin
     export GOPATH=$HOME/go
     export PATH=$PATH:$GOPATH/bin
     ```
   - Примени:
     ```bash
     source ~/.bashrc
     ```

3. **Проверь установку**:
   - Выполни:
     ```bash
     go version
     ```
   - Ожидаемый вывод:
     ```
     go version go1.22.3 linux/amd64
     ```

---

### 🔹 Установка Go на openSUSE

openSUSE использует менеджер пакетов `zypper`, но для последней версии мы установим Go вручную.

1. **Скачай Go**:
   - Перейди на [https://go.dev/dl/](https://go.dev/dl/) и выбери `go1.22.3.linux-amd64.tar.gz`:
     ```bash
     wget https://go.dev/dl/go1.22.3.linux-amd64.tar.gz
     ```

2. **Распакуй архив**:
   - Распакуй в `/usr/local`:
     ```bash
     sudo tar -C /usr/local -xzf go1.22.3.linux-amd64.tar.gz
     ```

3. **Настрой переменные окружения**:
   - Открой `~/.bashrc` или `~/.zshrc`:
     ```bash
     nano ~/.bashrc
     ```
   - Добавь:
     ```bash
     export PATH=$PATH:/usr/local/go/bin
     export GOPATH=$HOME/go
     export PATH=$PATH:$GOPATH/bin
     ```
   - Примени:
     ```bash
     source ~/.bashrc
     ```

4. **Проверь установку**:
   - Выполни:
     ```bash
     go version
     ```
   - Ожидаемый вывод:
     ```
     go version go1.22.3 linux/amd64
     ```

---

## ⚙️ Настройка окружения

После установки Go убедись, что переменные окружения настроены правильно:

- `PATH` включает `/usr/local/go/bin` для доступа к командам Go.
- `GOPATH` указывает на директорию `$HOME/go` (обычно `~/go`), где хранятся исходные коды, зависимости и бинарные файлы.

Если ты используешь другой путь для `GOPATH`, убедись, что он существует:
```bash
mkdir -p $GOPATH/src $GOPATH/bin $GOPATH/pkg
```

---

## 🧪 Проверка установки

**Проверь версию Go**:
   ```bash
   go version
   ```
   Если всё установлено правильно, ты увидишь версию, например:
   ```
   go version go1.22.3 linux/amd64
   ```