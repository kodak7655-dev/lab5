# Последовательность: заказы клиента (запрос)

```mermaid
sequenceDiagram
    autonumber
    actor Client
    participant Controller
    participant QueryBus
    participant GetOrdersByCustomerHandler
    participant OrderReadRepository

    Client->>Controller: GET /customers/{id}/orders
    Controller->>QueryBus: ask(GetOrdersByCustomerQuery)
    QueryBus->>GetOrdersByCustomerHandler: handle(query)
    GetOrdersByCustomerHandler->>OrderReadRepository: findByCustomer(customerId)
    OrderReadRepository-->>GetOrdersByCustomerHandler: OrderView[]
    GetOrdersByCustomerHandler-->>QueryBus: OrderView[]
    QueryBus-->>Controller: OrderView[]
    Controller-->>Client: 200 OK [OrderView]

    Note over GetOrdersByCustomerHandler,OrderReadRepository: write-репозиторий и домен не используются
```

В этом сценарии нет ни домена, ни write-репозитория: обработчик запроса
обращается только к `OrderReadRepository` и возвращает готовые `OrderView`.
Это и есть смысл разделения — чтение не зависит от модели записи и её
инвариантов.
