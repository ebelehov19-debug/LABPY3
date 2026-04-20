# Лабораторная работа №2. Модель задачи: дескрипторы и @property

## Цель работы
-  Освоить управление доступом к атрибутам и защиту инвариантов доменной
модели

## Постановка задачи
Необходимо расширить модель задачи (Task) из первой лабораторной работы, добавив корректную инкапсуляцию и валидацию состояния. Базовая структура задачи превращается из простого dataclass в полноценную доменную модель с проверкой целостности данных.

**Task** - доменная модель, содержащая:
-  `id` - уникальный идентификатор задачи (строка, 1-64 символа)
- `payload` - произвольные данные задачи (любой тип)
- `description` - описание задачи (строка, 1-1024 символа)
- `priority` - приоритет задачи (low, medium, high, critical)
- `status` - статус задачи (pending, in_progress, completed, failed, blocked)
- `created_at` - время создания задачи (только для чтения)
- `updated_at`- время последнего обновления (только для чтения)
- `completed_at` -  время завершения задачи (только для чтения)
- `is_ready_to_execute` - вычисляемое свойство готовности к выполнению
- `is_overdue` - просрочка дедлайна 

## Технические требования
- использование пользовательских дескрипторов для валидации атрибутов;
- использование `@property` для вычисляемых или защищённых свойств;
- предотвращение некорректных состояний объекта;
- генерация специализированных исключений при нарушении инвариантов;

## Чему я научился
- Созданию пользовательских data-дескрипторов с методами `__get__`, `__set__`, `__set_name__`
- Валидации данных на уровне атрибутов класса
- Использованию @property и @setter для управления доступом к полям
- Созданию специализированной иерархии исключений
- Защите инвариантов доменной модели
- Тестированию дескрипторов и вычисляемых свойств

## Дескрипторы

### StringField - валидация строк 
- Проверяет что значение является строкой
- Проверяет корректность длины строки 
- Используется для полей `id` `payload`

### PriorityField — валидация приоритета
- Допустимые значения: `low`, `medium`, `high`, `critical`
- Не зависит от регистра приоретета 
- Используется для priority

### StatusField — валидация статуса с проверкой переходов
- Допустимые значения: `pending`, `in_progress`, `completed`, `failed`, `blocked`
- Проверка допустимых переходов между статусами
- Защита от изменения конечных состояний `completed`, `failed`

### Правила переходов статусов
- Из pending и in_progress можно перейти в любой статус
- Из blocked можно перейти только в pending или failed
- Из completed и failed нельзя изменить статус 

### Вычисляемые свойства (@property)
- is_ready_to_execute
- is_overdue

## Расширенные форматы источников задач

### Стандартный поток ввода (stdin)
- Базовый формат: `id:payload`
- Расширенный формат с описанием: `id:payload:description`
- Полный формат с приоритетом: `id:payload:description:priority`

### Чтение из файла (JSON)
- Бфзовый формат: {"id": "1", "payload": "Купить билеты на концерт GSPD"}
- Расширенный формат с описанием: {"id": "1", "payload": "Купить билеты на концерт GSPD", "description": "Срочно купить 2 билета"}
- Полный формат с приоритетом: {"id": "1", "payload": "Купить билеты на концерт GSPD", "description": "Срочно купить 2 билета", "priority": "high"}

## Исключения 
- TaskError (базовое)
- TaskValidationError (ошибка валидации, наследует ValueError)
- InvalidTaskStatusTransitionError (ошибка перехода статуса)

## Инструкции по запуску

1. **Клонирование репозитория**
```git clone https://github.com/ebelehov19-debug/LabPY2_2```
2. **Создание вертуального окружения**
```python -m venv .venv```
```source .venv/bin/activate```
3. **Установка зависимостей**
```pip install typer```
4. **Просмотр команд**
```python -m src --help```
5. **Просмотр плагинов**
```python -m src plugins```

## Примеры использования программы
1. **Чтение из JSONL(JOSN) файла**
```python -m src read --jsonl source/tasks.jsonl```
2. **Чтение из stdin**
- Базовый формат:
```echo "1: Buy milk" | python -m src read --stdin```
- Расширенный формат:
```echo "1: Buy milk: 2litra" | python -m src read --stdin```
- Полный формат: 
```echo "1: Buy milk: 2litra: high" | python -m src read --stdin```

- Многострочный ввод в stdin  
``` 
echo "1:Buy milk
2:Work matan
3:work work" | python -m src read --stdin
``` 
3. **Фильтрация задач**
```python -m src read --jsonl source/tasks.jsonl --contains "work"```

4. Интерактивная проверка модели Task
``` 
python -c "
from src.contracts.task import Task
from datetime import datetime

task = Task(id='1', payload='Тест', description='Проверка', priority='high')
print(f'Создана задача: {task}')
print(f'Готова к выполнению: {task.is_ready_to_execute}')
print(f'Создана: {task.created_at}')

task.status = 'in_progress'
print(f'Новый статус: {task.status}')
print(f'Обновлена: {task.updated_at}')

try:
    task.status = 'completed'
    task.status = 'in_progress'
except Exception as e:
    print(f'Ошибка: {e}')
"
```
## Запуск тестов
1. **Запуск всех тестов**
```python -m pytest tests/ -v```
