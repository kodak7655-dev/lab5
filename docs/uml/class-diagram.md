# Диаграмма классов

```mermaid
classDiagram
    class Order {
        +String id
        +String customerId
        +OrderStatus status
        +DateTime createdAt
        +OrderItem[] items
        +Money total
        +addItem(Product product, int quantity) void
        +removeItem(String productId) void
        +changeStatus(OrderStatus status) void
        +cancel() void
        +recalculateTotal() void
    }
    class OrderItem {
        +String productId
        +int quantity
        +Money price
        +amount() Money
    }
    class Product {
        +String id
        +String name
        +Money price
    }
    class Customer {
        +String id
        +String name
        +String email
    }
    class OrderStatus {
        <<enumeration>>
        NEW
        CONFIRMED
        PAID
        SHIPPED
        COMPLETED
        CANCELLED
    }
    class CreateOrderCommand {
        +String customerId
    }
    class CreateOrderHandler {
        +handle(CreateOrderCommand command) String
    }
    class AddProductCommand {
        +String orderId
        +String productId
        +int quantity
    }
    class AddProductHandler {
        +handle(AddProductCommand command) String
    }
    class GetOrderQuery {
        +String orderId
    }
    class GetOrderHandler {
        +handle(GetOrderQuery query) OrderView
    }

    Order "1" *-- "1..*" OrderItem : contains
    OrderItem --> Product : references
    Order --> Customer : belongs to
    Order --> OrderStatus : has

    CreateOrderHandler ..> CreateOrderCommand : handles
    CreateOrderHandler ..> Order : creates
    AddProductHandler ..> AddProductCommand : handles
    AddProductHandler ..> Order : modifies
    GetOrderHandler ..> GetOrderQuery : handles

    note for GetOrderHandler "reads OrderView only, no domain"
```

Диаграмма фиксирует границу агрегата: `Order` владеет позициями (композиция),
а `Product` и `Customer` — внешние сущности, на которые заказ ссылается по
идентификатору. Обработчики команд зависят от команды и агрегата, обработчик
запроса — только от запроса и read-модели `OrderView`, поэтому домен в
чтении не участвует.
