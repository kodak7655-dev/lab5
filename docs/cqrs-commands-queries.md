# Команды, запросы и обработчики

## Правило CQRS

Команда описывает намерение изменить состояние и не возвращает данные для
отображения. Запрос только читает подготовленную проекцию и не меняет
состояние. Стороны связаны доменными событиями, а не общими моделями.

## Команды

| Команда | Обработчик | Входные данные | Назначение | События |
|---|---|---|---|---|
| `CreateOrderCommand` | `CreateOrderHandler` | `customerId` | создать заказ | `OrderCreated` |
| `AddProductCommand` | `AddProductHandler` | `orderId`, `productId`, `quantity` | добавить позицию | `OrderItemAdded` |
| `RemoveProductCommand` | `RemoveProductHandler` | `orderId`, `productId` | удалить позицию | `OrderItemRemoved` |
| `ChangeOrderStatusCommand` | `ChangeOrderStatusHandler` | `orderId`, `status` | сменить статус по правилам | `OrderStatusChanged` |
| `CancelOrderCommand` | `CancelOrderHandler` | `orderId`, `reason` | отменить заказ | `OrderCancelled` |

## Запросы

| Запрос | Обработчик | Входные данные | Что возвращает |
|---|---|---|---|
| `GetOrderQuery` | `GetOrderHandler` | `orderId` | `OrderView` |
| `GetOrdersQuery` | `GetOrdersHandler` | `page`, `size` | `OrderView[]` |
| `GetOrdersByCustomerQuery` | `GetOrdersByCustomerHandler` | `customerId` | `OrderView[]` |
| `GetOrdersByStatusQuery` | `GetOrdersByStatusHandler` | `status` | `OrderView[]` |
| `GetOrderStatisticsQuery` | `GetOrderStatisticsHandler` | `from`, `to` | `OrderStatistics` |

## Обработчики команд

- **CreateOrderHandler** — создаёт агрегат `Order`, сохраняет его и публикует
  `OrderCreated`.
- **AddProductHandler** — загружает заказ, проверяет, что он существует и не
  отменён, вызывает `addItem` и публикует `OrderItemAdded`.
- **RemoveProductHandler** — загружает заказ, удаляет позицию через агрегат и
  публикует `OrderItemRemoved`.
- **ChangeOrderStatusHandler** — загружает заказ и переводит его в новый статус
  только по допустимому переходу, публикует `OrderStatusChanged`.
- **CancelOrderHandler** — проверяет, что статус не `SHIPPED` и не `COMPLETED`,
  отменяет заказ и публикует `OrderCancelled`.

## Обработчики запросов

- **GetOrderHandler** — читает одну запись `OrderView` по идентификатору.
- **GetOrdersHandler** — возвращает страницу списка `OrderView`.
- **GetOrdersByCustomerHandler** — выбирает `OrderView` по `customerId`.
- **GetOrdersByStatusHandler** — выбирает `OrderView` по статусу.
- **GetOrderStatisticsHandler** — считает количество и суммы заказов за период.

## Разделение ответственности

Command handler не возвращает данные для отображения: результат команды —
идентификатор и признак успеха. Query handler не вызывает write-репозиторий и
не публикует события: он работает только с read-моделью `OrderView`.
