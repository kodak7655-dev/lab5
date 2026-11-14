# Последовательность: добавление товара в заказ

```mermaid
sequenceDiagram
    autonumber
    actor Client
    participant Controller
    participant CommandBus
    participant AddProductHandler
    participant Order
    participant WriteRepository
    participant EventBus
    participant OrderProjector
    participant OrderReadRepository

    Client->>Controller: POST /orders/{id}/items {productId, quantity}
    Controller->>CommandBus: dispatch(AddProductCommand)
    CommandBus->>AddProductHandler: handle(command)
    AddProductHandler->>WriteRepository: find(orderId)

    alt заказ не найден или отменён
        WriteRepository-->>AddProductHandler: null
        AddProductHandler-->>Controller: OrderCannotBeModified
        Controller-->>Client: 409 Conflict
    else заказ можно менять
        WriteRepository-->>AddProductHandler: order
        AddProductHandler->>Order: addItem(product, quantity)
        Order->>Order: recalculateTotal()
        AddProductHandler->>WriteRepository: save(order)
        WriteRepository->>EventBus: publish(OrderItemAdded)
        EventBus->>OrderProjector: onOrderItemAdded(event)
        OrderProjector->>OrderReadRepository: update(OrderView)
        AddProductHandler-->>Controller: ok
        Controller-->>Client: 204 No Content
    end
```

Перед изменением обработчик проверяет, что заказ существует и не отменён, а
инварианты количества и цены проверяет сам агрегат в методе `addItem`. После
сохранения публикуется `OrderItemAdded`, и проектор пересчитывает `OrderView`.
