# Музичний сервіс

Цей проєкт моделює базу даних для музичного сервісу. Користувачам надається можливість прослуховувати музику, шукати її за артистами чи альбомами, додавати треки у "Вибране" та створювати власні плейлісти. Окрім того, користувач може блокувати рекомендацію конкретного виконавця або приховувати можливість відтворення його окремих треків.

## ER-діаграма

erDiagram
    USER {
        UUID id PK
        string email
        string nickname
    }
    
    ARTIST {
        UUID id PK
        string name
    }
    
    ALBUM {
        UUID id PK
        string title
        int release_year
    }
    
    TRACK {
        UUID id PK
        string title
        int duration_seconds
    }
    
    PLAYLIST {
        UUID id PK
        UUID user_id FK
        string title
        timestamp created_at
    }

    %% Зв'язки користувача
    USER ||--o{ PLAYLIST : creates
    USER }o--o{ TRACK : "favorites"
    USER }o--o{ ARTIST : "blocks"
    USER }o--o{ TRACK : "blocks"

    %% Зв'язки виконавця
    ARTIST }o--o{ ALBUM : "releases"
    ARTIST }o--o{ TRACK : "authors"

    %% Зв'язки альбому
    ALBUM }o--o{ TRACK : "contains"

    %% Зв'язки плейліста
    PLAYLIST }o--o{ TRACK : "contains"