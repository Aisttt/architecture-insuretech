# Примеры GraphQL запросов

## 1. Базовые запросы (эквивалент REST API)

### Получить информацию о клиенте (GET /clients/{id})
```graphql
query GetClient($id: ID!) {
  client(id: $id) {
    id
    name
    age
  }
}
```

### Получить документы клиента (GET /clients/{id}/documents)
```graphql
query GetClientDocuments($clientId: ID!) {
  clientDocuments(clientId: $clientId) {
    id
    type
    number
    issueDate
    expiryDate
  }
}
```

### Получить родственников клиента (GET /clients/{id}/relatives)
```graphql
query GetClientRelatives($clientId: ID!) {
  clientRelatives(clientId: $clientId) {
    id
    relationType
    name
    age
  }
}
```

## 2. Оптимизированные запросы (решение N+1 проблемы)

### Получить клиента со всеми связанными данными одним запросом
```graphql
query GetClientFullInfo($id: ID!) {
  client(id: $id) {
    id
    name
    age
    documents {
      id
      type
      number
      issueDate
      expiryDate
      status
    }
    relatives {
      id
      relationType
      name
      age
      contactInfo {
        phone
        email
      }
    }
  }
}
```

### Выборочная загрузка (только нужные поля)
```graphql
query GetClientBasicWithActiveDocuments($id: ID!) {
  client(id: $id) {
    name
    age
    documents(filter: { activeOnly: true }) {
      type
      number
      expiryDate
    }
  }
}
```

## 3. Расширенные сценарии

### Поиск клиентов с фильтрацией
```graphql
query SearchClients($search: ClientSearchInput!) {
  clients(search: $search) {
    id
    name
    age
    documents {
      type
      status
    }
  }
}

# Переменные:
{
  "search": {
    "name": "Иван",
    "ageFrom": 25,
    "ageTo": 45,
    "documentType": "PASSPORT",
    "limit": 10
  }
}
```

### Получить только родственников-супругов с контактами
```graphql
query GetSpousesWithContacts($clientId: ID!) {
  clientRelatives(
    clientId: $clientId, 
    filter: { relationType: SPOUSE }
  ) {
    name
    age
    contactInfo {
      phone
      email
      address {
        city
        street
      }
    }
    isClient
    clientRecord {
      id
      name
    }
  }
}
```

### Получить документы определенного типа
```graphql
query GetPassports($clientId: ID!) {
  clientDocuments(
    clientId: $clientId,
    filter: { 
      type: PASSPORT,
      status: ACTIVE 
    }
  ) {
    id
    number
    issueDate
    expiryDate
    issuer
  }
}
```

## 4. Мутации (изменение данных)

### Создать нового клиента
```graphql
mutation CreateClient($name: String!, $age: Int!) {
  createClient(name: $name, age: $age) {
    id
    name
    age
    createdAt
  }
}
```

### Добавить документ клиенту
```graphql
mutation AddDocument($clientId: ID!, $type: DocumentType!, $number: String!, $issueDate: String!) {
  addDocument(
    clientId: $clientId,
    type: $type,
    number: $number,
    issueDate: $issueDate
  ) {
    id
    type
    number
    status
    client {
      name
    }
  }
}
```

### Добавить родственника
```graphql
mutation AddRelative($clientId: ID!, $relationType: RelationType!, $name: String!, $age: Int!) {
  addRelative(
    clientId: $clientId,
    relationType: $relationType,
    name: $name,
    age: $age
  ) {
    id
    relationType
    name
    client {
      name
    }
  }
}
```

## 5. Подписки (real-time)

### Подписка на изменения клиента
```graphql
subscription ClientUpdates($id: ID!) {
  clientUpdated(id: $id) {
    id
    name
    age
    updatedAt
  }
}
```

### Подписка на новые документы
```graphql
subscription NewDocuments($clientId: ID!) {
  documentAdded(clientId: $clientId) {
    id
    type
    number
    issueDate
    client {
      name
    }
  }
}
```

## 6. Сравнение с REST API

### REST: 3 запроса для полной информации
```
GET /clients/123           # Базовая информация
GET /clients/123/documents # Документы  
GET /clients/123/relatives # Родственники
```

### GraphQL: 1 запрос для той же информации
```graphql
query GetClientComplete($id: ID!) {
  client(id: $id) {
    id
    name
    age
    documents {
      id
      type
      number
      issueDate
      expiryDate
    }
    relatives {
      id
      relationType
      name
      age
    }
  }
}
```

## Преимущества GraphQL решения

1. **Уменьшение количества запросов** - с 3 до 1
2. **Гибкость выборки** - только нужные поля
3. **Типизация** - строгие типы и валидация
4. **Фильтрация** - встроенные возможности фильтрации
5. **Introspection** - самодокументирующийся API
6. **Real-time** - подписки для живых обновлений 