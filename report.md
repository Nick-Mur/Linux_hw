# Отчёт по лабораторной работе: Управление процессами

## Введение

В современных операционных системах управление процессами играет ключевую роль в обеспечении эффективной работы системы. Понимание основ управления процессами позволяет пользователям и системным администраторам оптимизировать использование ресурсов, контролировать выполнение задач и обеспечивать стабильность системы. Цель данной лабораторной работы — изучить основные инструменты и команды для управления процессами в Linux, научиться запускать, приостанавливать, возобновлять процессы, изменять их приоритеты, а также взаимодействовать со службами системы.

## Цель работы

- Освоить команды для просмотра и управления запущенными процессами в Linux.
- Научиться запускать процессы в фоновом режиме и управлять их выполнением.
- Изучить методы приостановки, возобновления и завершения процессов.
- Понять принцип изменения приоритетов процессов с помощью `nice` и `renice`.

## Оборудование и программное обеспечение

- Операционная система Linux (например, Ubuntu, Fedora и т.д.)
- Терминал с доступом к стандартным утилитам: `bash`, `pgrep`, `kill`, `renice`, `ps`, `sleep`, `date`.
- Редактор текста для написания скриптов (например, `nano`, `vim`, `gedit`).

## Теоретическая часть

### Управление процессами в Linux

Процесс в Linux — это выполняемая программа, имеющая собственный идентификатор (PID). Управление процессами включает в себя запуск, приостановку, возобновление, изменение приоритетов и завершение процессов. Основные команды для управления процессами:

- `ps` — отображение списка текущих процессов.
- `pgrep` — поиск процессов по имени.
- `kill` — отправка сигналов процессам (для приостановки, возобновления, завершения).
- `nice` и `renice` — управление приоритетом процессов.
- `bg` и `fg` — управление фоновыми и передними процессами.
- `jobs` — просмотр текущих задач в оболочке.

### Приоритеты процессов

Каждый процесс имеет приоритет, определяемый значением `nice`. Значение `nice` может варьироваться от -20 (высокий приоритет) до +19 (низкий приоритет). Использование `renice` позволяет изменять приоритет уже запущенных процессов.

## Практическая часть

### Задача

Написать и выполнить скрипт `process_control.sh`, который выполняет следующие действия:

1. Запускает три разных процесса:
   - **Процесс 1:** Бесконечно записывает дату и время в файл `output1.log` каждые 3 секунды.
   - **Процесс 2:** Бесконечно выводит в терминал случайные числа каждые 5 секунд.
   - **Процесс 3:** Бесконечно записывает строку `"Hello from process 3"` в файл `output2.log` каждые 2 секунды.
   
2. После запуска процессов:
   - Находит их PID.
   - Приостанавливает второй процесс, возобновляет его в фоне через 10 секунд.
   - Изменяет приоритет третьего процесса на более низкий (`nice` +10).
   - Через 30 секунд завершает все три процесса.

### Реализация

#### 1. Создание вспомогательных скриптов

Для удобства создания и управления процессами, были созданы три отдельных скрипта:

- **process1.sh**
  ```bash
  #!/bin/bash
  while true
  do
      date >> output1.log
      sleep 3
  done
  ```

- **process2.sh**
  ```bash
  #!/bin/bash
  while true
  do
      echo $RANDOM
      sleep 5
  done
  ```

- **process3.sh**
  ```bash
  #!/bin/bash
  while true
  do
      echo "Hello from process 3" >> output2.log
      sleep 2
  done
  ```

После создания скриптов им были присвоены права на исполнение:
```bash
chmod +x process1.sh process2.sh process3.sh
```

#### 2. Создание основного скрипта `process_control.sh`

```bash
#!/bin/bash

# Очистка лог-файлов перед запуском
> output1.log
> output2.log

# 1. Запуск трех процессов в фоновом режиме
./process1.sh &
PID1=$!
echo "Запущен процесс 1 с PID: $PID1"

./process2.sh &
PID2=$!
echo "Запущен процесс 2 с PID: $PID2"

./process3.sh &
PID3=$!
echo "Запущен процесс 3 с PID: $PID3"

# 2. Приостановка второго процесса
echo "Приостанавливаем процесс 2 (PID: $PID2)"
kill -STOP $PID2

# Ожидание 10 секунд
echo "Ждем 10 секунд..."
sleep 10

# Возобновление второго процесса в фоне
echo "Возобновляем процесс 2 (PID: $PID2) в фоне"
kill -CONT $PID2

# 3. Изменение приоритета третьего процесса
echo "Изменяем приоритет процесса 3 (PID: $PID3) на +10"
renice +10 -p $PID3

# 4. Ожидание 30 секунд перед завершением процессов
echo "Ждем 30 секунд..."
sleep 30

# Завершение всех трех процессов
echo "Завершаем процессы..."
kill $PID1 $PID2 $PID3

echo "Все процессы успешно завершены."
```

После создания скрипта `process_control.sh` ему были присвоены права на исполнение:
```bash
chmod +x process_control.sh
```

#### 3. Пояснения к работе скрипта

- **Запуск процессов:** Скрипт запускает три вспомогательных процесса в фоновом режиме с помощью символа `&`. После каждого запуска сохраняется PID процесса с помощью переменной `$!`.

- **Приостановка и возобновление процесса:** Второй процесс приостанавливается с помощью команды `kill -STOP PID`, затем через 10 секунд возобновляется с помощью `kill -CONT PID`.

- **Изменение приоритета:** Приоритет третьего процесса изменяется на более низкий (увеличивается значение `nice`) с помощью команды `renice +10 -p PID`.

- **Завершение процессов:** Через 30 секунд все три процесса завершаются командой `kill PID1 PID2 PID3`.

### Запуск и проверка работы скрипта

1. **Запуск скрипта:**
   ```bash
   ./process_control.sh
   ```

2. **Наблюдение за выполнением:**
   - **Процесс 1:** Проверка содержимого `output1.log` должна показать запись даты и времени каждые 3 секунды.
   - **Процесс 2:** В терминале выводятся случайные числа каждые 5 секунд. После приостановки этот вывод прекращается на 10 секунд, затем возобновляется.
   - **Процесс 3:** Проверка содержимого `output2.log` должна показать запись строки `"Hello from process 3"` каждые 2 секунды с пониженным приоритетом.

3. **Завершение процессов:** После 30 секунд работы все процессы автоматически завершаются, и вывод терминала подтверждает это.

### Пример вывода терминала

```
Запущен процесс 1 с PID: 12345
Запущен процесс 2 с PID: 12346
Запущен процесс 3 с PID: 12347
Приостанавливаем процесс 2 (PID: 12346)
Ждем 10 секунд...
Возобновляем процесс 2 (PID: 12346) в фоне
Изменяем приоритет процесса 3 (PID: 12347) на +10
Ждем 30 секунд...
Завершаем процессы...
Все процессы успешно завершены.
```

## Результаты

В результате выполнения лабораторной работы был разработан и протестирован скрипт `process_control.sh`, который успешно выполняет поставленные задачи:

- Запускает три процесса с различными задачами.
- Определяет PID запущенных процессов.
- Приостанавливает и возобновляет второй процесс.
- Изменяет приоритет третьего процесса.
- Завершает все процессы через заданное время.

Лог-файлы `output1.log` и `output2.log` содержат соответствующие записи, подтверждающие корректное выполнение процессов. Вывод случайных чисел в терминале подтверждает работу второго процесса, который был приостановлен и возобновлён.

## Выводы

В ходе лабораторной работы были успешно освоены основные команды и методы управления процессами в Linux. Было продемонстрировано, как запускать процессы в фоновом режиме, определять их PID, приостанавливать и возобновлять выполнение, изменять приоритеты и завершать процессы. Полученные знания позволяют эффективно управлять ресурсами системы, оптимизировать выполнение задач и обеспечивать стабильность работы операционной системы.

## Приложения

### Содержимое скрипта `process_control.sh`

```bash
#!/bin/bash

# Очистка лог-файлов перед запуском
> output1.log
> output2.log

# 1. Запуск трех процессов в фоновом режиме
./process1.sh &
PID1=$!
echo "Запущен процесс 1 с PID: $PID1"

./process2.sh &
PID2=$!
echo "Запущен процесс 2 с PID: $PID2"

./process3.sh &
PID3=$!
echo "Запущен процесс 3 с PID: $PID3"

# 2. Приостановка второго процесса
echo "Приостанавливаем процесс 2 (PID: $PID2)"
kill -STOP $PID2

# Ожидание 10 секунд
echo "Ждем 10 секунд..."
sleep 10

# Возобновление второго процесса в фоне
echo "Возобновляем процесс 2 (PID: $PID2) в фоне"
kill -CONT $PID2

# 3. Изменение приоритета третьего процесса
echo "Изменяем приоритет процесса 3 (PID: $PID3) на +10"
renice +10 -p $PID3

# 4. Ожидание 30 секунд перед завершением процессов
echo "Ждем 30 секунд..."
sleep 30

# Завершение всех трех процессов
echo "Завершаем процессы..."
kill $PID1 $PID2 $PID3

echo "Все процессы успешно завершены."
```

### Содержимое вспомогательных скриптов

- **process1.sh**
  ```bash
  #!/bin/bash
  while true
  do
      date >> output1.log
      sleep 3
  done
  ```

- **process2.sh**
  ```bash
  #!/bin/bash
  while true
  do
      echo $RANDOM
      sleep 5
  done
  ```

- **process3.sh**
  ```bash
  #!/bin/bash
  while true
  do
      echo "Hello from process 3" >> output2.log
      sleep 2
  done
  ```

## Рекомендации

Для успешного выполнения лабораторной работы рекомендуется:

- Тщательно проверять синтаксис скриптов перед их запуском.
- Убедиться в наличии необходимых прав на исполнение скриптов.
- Проверять содержимое лог-файлов и вывод в терминал для подтверждения корректной работы процессов.
- Использовать команды `ps`, `pgrep`, `kill`, `renice` для мониторинга и управления процессами.

## Литература

- `man ps`
- `man kill`
- `man nice` и `man renice`
- [Linux Processes Explained (англ.)](https://tldp.org/LDP/intro-linux/html/sect_05_01.html)
- [Статья о приоритетах процессов (рус.)](https://habr.com/ru/post/537040/)
- Google, StackOverflow
