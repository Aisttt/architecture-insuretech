# Задание 5. Проектирование GraphQL API

## Анализ существующего REST API

### Проблемы текущего подхода
- **Over-fetching**: клиенты получают все 500 атрибутов, даже если нужны только некоторые
- **Under-fetching (N+1)**: для полной информации нужно 3 отдельных запроса
- **Высокий RPS**: множественные запросы увеличивают нагрузку на сервис
- **Отсутствие гибкости**: фиксированная структура ответов

### Существующие эндпоинты
1. `GET /clients/{id}` - базовая информация о клиенте
2. `GET /clients/{id}/documents` - документы клиента
3. `GET /clients/{id}/relatives` - родственники клиента

## GraphQL решение

### Ключевые типы
- **Client** - основная сущность с полями id, name, age + связанные documents и relatives
- **Document** - документы клиента с типизацией и статусами
- **Relative** - родственники с типами связей и возможностью быть клиентами
- **ContactInfo** и **Address** - дополнительная контактная информация

### Enum типы
- **DocumentType** - типы документов (паспорт, права, и т.д.)
- **DocumentStatus** - статусы документов (активный, истек, отозван)
- **RelationType** - типы родственных связей

### Входные типы для фильтрации
- **ClientSearchInput** - поиск клиентов по различным критериям
- **DocumentFilterInput** - фильтрация документов
- **RelativeFilterInput** - фильтрация родственников

## Основные запросы (Queries)

### Базовые запросы
```graphql
client(id: ID!): Client                    # Получить клиента
clients(search: ClientSearchInput): [Client!]!  # Поиск клиентов
clientDocuments(clientId: ID!, filter: DocumentFilterInput): [Document!]!
clientRelatives(clientId: ID!, filter: RelativeFilterInput): [Relative!]!
```

### Оптимизированный запрос
```graphql
clientFullInfo(id: ID!): Client  # Полная информация одним запросом
```

## Преимущества GraphQL решения

### 1. Решение проблемы Over-fetching
```graphql
# Запросить только имя и возраст
query {
  client(id: "123") {
    name
    age
  }
}
```

### 2. Решение проблемы Under-fetching
```graphql
# Получить всё одним запросом вместо 3
query {
  client(id: "123") {
    name
    age
    documents { type, number }
    relatives { relationType, name }
  }
}
```

### 3. Гибкая фильтрация
```graphql
# Только активные паспорта
query {
  clientDocuments(
    clientId: "123",
    filter: { type: PASSPORT, status: ACTIVE }
  ) {
    number
    expiryDate
  }
}
```

### 4. Вложенные запросы
```graphql
# Родственники, которые тоже клиенты
query {
  clientRelatives(clientId: "123") {
    name
    isClient
    clientRecord {
      documents { type }
    }
  }
}
```

## Мутации (опциональное расширение)
- `createClient` - создание клиента
- `addDocument` - добавление документа
- `addRelative` - добавление родственника

## Подписки (real-time обновления)
- `clientUpdated` - изменения клиента
- `documentAdded` - новые документы

## Технические преимущества

### Производительность
- Уменьшение RPS с 3 запросов до 1
- Снижение объема передаваемых данных
- Возможность batch loading и кэширования

### Разработка
- Строгая типизация схемы
- Автоматическая валидация
- Self-documenting API через introspection
- Лучший developer experience

### Масштабируемость
- Легкое добавление новых полей без breaking changes
- Версионирование через deprecation
- Федерация схем для микросервисной архитектуры

## Файлы решения
- `client_info_schema.graphql` - полная GraphQL схема
- `graphql_examples.md` - примеры запросов и сравнение с REST
- `client_info_analysis.md` - детальный анализ проблем и решений 