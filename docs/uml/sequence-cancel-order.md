# Последовательность: отмена заказа

```mermaid
sequenceDiagram
    autonumber
    actor Client
    participant Controller
    participant CommandBus
    participant CancelOrderHandler
    participant Order
    participant WriteRepository
    participant EventBus
    participant OrderProjector
    participant OrderReadRepository

    Client->>Controller: POST /orders/{id}/cancel {reason}
    Controller->>CommandBus: dispatch(CancelOrderCommand)
    CommandBus->>CancelOrderHandler: handle(command)
    CancelOrderHandler->>WriteRepository: find(orderId)

    alt статус SHIPPED или COMPLETED
        WriteRepository-->>CancelOrderHandler: order
        CancelOrderHandler-->>Controller: OrderCannotBeCancelled
        Controller-->>Client: 409 Conflict
    else статус NEW, CONFIRMED или PAID
        WriteRepository-->>CancelOrderHandler: order
        CancelOrderHandler->>Order: cancel(reason)
        CancelOrderHandler->>WriteRepository: save(order)
        WriteRepository->>EventBus: publish(OrderCancelled)
        EventBus->>OrderProjector: onOrderCancelled(event)
        OrderProjector->>OrderReadRepository: update(OrderView)
        CancelOrderHandler-->>Controller: ok
        Controller-->>Client: 204 No Content
    end
```

Отмена запрещена для отправленных и выполненных заказов — это проверяет и
обработчик, и сам агрегат. После сохранения публикуется `OrderCancelled`, а
проектор переводит `OrderView` в статус `CANCELLED`.
