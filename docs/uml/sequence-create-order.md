# Последовательность: создание заказа

```mermaid
sequenceDiagram
    autonumber
    actor Client
    participant Controller
    participant CommandBus
    participant CreateOrderHandler
    participant WriteRepository
    participant EventBus
    participant OrderProjector
    participant OrderReadRepository

    Client->>Controller: POST /orders {customerId}
    Controller->>CommandBus: dispatch(CreateOrderCommand)
    CommandBus->>CreateOrderHandler: handle(command)
    CreateOrderHandler->>CreateOrderHandler: Order.create(customerId)
    CreateOrderHandler->>WriteRepository: save(order)
    WriteRepository->>EventBus: publish(OrderCreated)
    EventBus->>OrderProjector: onOrderCreated(event)
    OrderProjector->>OrderReadRepository: save(OrderView)
    CreateOrderHandler-->>CommandBus: orderId
    CommandBus-->>Controller: orderId
    Controller-->>Client: 201 Created {orderId}
```

Команда проходит путь «контроллер → шина → обработчик → репозиторий», после
сохранения публикуется `OrderCreated`. Проектор по событию создаёт строку
`OrderView`, поэтому чтение сразу видит новый заказ. Клиенту возвращается
только идентификатор, а не данные заказа.
