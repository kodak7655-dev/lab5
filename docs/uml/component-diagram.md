# Диаграмма компонентов

```mermaid
flowchart LR
    Client[Client] --> API[API Controller]
    API -->|command| CB[CommandBus]
    API -->|query| QB[QueryBus]
    CB --> CH[CommandHandlers]
    QB --> QH[QueryHandlers]
    CH --> DM[DomainModel]
    CH --> WR[WriteRepository]
    CH --> EB[EventBus]
    EB --> PR[OrderProjector]
    PR --> RR[OrderReadRepository]
    QH --> RR
```

Схема показывает два независимых маршрута: команда идёт через `CommandBus` в
домен и write-репозиторий, запрос — через `QueryBus` сразу в
`OrderReadRepository`. Стороны связаны только шиной событий и проектором,
поэтому чтение не загружает агрегат, а запись не формирует DTO.
