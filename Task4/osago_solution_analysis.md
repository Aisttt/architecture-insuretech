# Проектирование системы продажи ОСАГО

## Анализ требований

### Функциональные требования
- Клиент заполняет заявку с информацией об автомобиле
- Система запрашивает предложения у всех страховых компаний
- Предложения отображаются в real-time по мере поступления
- Максимальное время ожидания: 60 секунд
- API страховых компаний: создать заявку + получить предложение

### Нефункциональные требования
- Пиковая нагрузка: 2500 одновременных пользователей
- Real-time отображение результатов
- Отказоустойчивость при недоступности страховых компаний
- Несколько экземпляров сервисов (масштабирование)

## Архитектурные решения

### 1. osago-aggregator сервис

**Функции:**
- Отправка заявок в страховые компании
- Опрос решений по заявкам
- Передача результатов в core-app

**Хранилище данных:** ДА, необходимо
- Временное хранение заявок и статусов
- Кэширование результатов
- Отслеживание состояния запросов к страховым компаниям

**API для core-app:**
```
POST /osago/quotes - создать запрос на котировки
GET /osago/quotes/{requestId} - получить статус и результаты
WebSocket /osago/quotes/{requestId}/stream - real-time обновления
```

### 2. Интеграция между сервисами

**core-app ↔ osago-aggregator:**
- REST API для инициации запросов
- WebSocket для real-time обновлений
- Event-driven через Kafka для асинхронной обработки

**Веб-приложение ↔ core-app:**
- WebSocket для real-time отображения предложений
- REST API для создания заявок

### 3. Паттерны отказоустойчивости

**Rate Limiting:**
- На osago-aggregator (защита от перегрузки)
- На API страховых компаний

**Circuit Breaker:**
- Между osago-aggregator и страховыми компаниями
- Между core-app и osago-aggregator

**Retry:**
- При временных сбоях запросов к страховым компаниям
- Exponential backoff

**Timeout:**
- 60 секунд для получения предложений
- 5 секунд для отдельных HTTP запросов

### 4. Масштабирование

**Множественные экземпляры:**
- Load balancer перед osago-aggregator
- Sticky sessions для WebSocket соединений
- Распределенное кэширование (Redis)
- Координация через Kafka events

## Технические детали

### База данных osago-aggregator
```sql
CREATE TABLE osago_requests (
    id UUID PRIMARY KEY,
    customer_id VARCHAR(100),
    vehicle_data JSONB,
    status VARCHAR(50),
    created_at TIMESTAMP,
    expires_at TIMESTAMP
);

CREATE TABLE osago_quotes (
    id UUID PRIMARY KEY,
    request_id UUID REFERENCES osago_requests(id),
    company_id VARCHAR(100),
    quote_data JSONB,
    status VARCHAR(50),
    received_at TIMESTAMP
);
```

### WebSocket API
```javascript
// Клиентская подписка
const ws = new WebSocket('/osago/quotes/{requestId}/stream');
ws.onmessage = (event) => {
    const quote = JSON.parse(event.data);
    displayNewQuote(quote);
};
```

### Event Flow
1. Клиент создает заявку → core-app
2. core-app → osago-aggregator (REST)
3. osago-aggregator → страховые компании (async)
4. Получение предложений → WebSocket уведомления
5. Timeout через 60 секунд → закрытие сессии 