## 1. Початковий промпт

Згенеруй ER-діаграму у синтаксисі Mermaid (erDiagram) на основі цієї специфікації:
## Сутності, атрибути та зв'язки

-User (Користувач): має 'id', 'email', 'nickname'.
    -Створює багато Playlist (1:N).
    -Може додавати багато Track у "Вибране" (N:M).
    -Може заблокувати багато Artist (N:M).
    -Може заблокувати багато Track (N:M).

-Artist (Виконавець): має 'id', 'name'.
    -Випускає багато Album (N:M).
    -Є автором багатьох пісень (N:M).

-Album (Альбом): має 'id', 'title', 'release_year'.
    -Може належати багатьом Artist (N:M).
    -Містить багато Track (N:M).
    
-Track (Трек):  має 'id', 'title', 'duration_seconds'.
    -Може належати багатьом Album (N:M).
    -Може мати кількох авторів Artist (N:M).

-Playlist (Плейліст): має 'id', 'title', 'created_at'.
    -Належить одному User (N:1).
    -Містить багато Track (M:N).

## Критерії прийняття 
1.Усi ідентифікатори ('id') мають тип 'UUID'.
2.Модель відповідає 3-й нормальній формі (3-NF).
3.Зв'язки N:M відображаються прямо між сутностями, без додаткових сполучних таблиць.
4.Результат має бути у форматі Mermaid ('erDiagram').

## 2. Перша відповідь AI

erDiagram
    USER {
        UUID id
        string email
        string nickname
    }
    
    ARTIST {
        UUID id
        string name
    }
    
    ALBUM {
        UUID id
        string title
        int release_year
    }
    
    TRACK {
        UUID id
        string title
        int duration_seconds
    }
    
    PLAYLIST {
        UUID id
        string title
        datetime created_at
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

## Другий промпт

spec оновлено, тепер там є нові критерії 5.Для зв'язків 1:N сутність на боці «багато» повинна явно містити поле зовнішнього ключа з позначкою 'FK'.
6.Поля з часом повинні мати тип 'timestamp', а не 'datetime'. згенеруй ще раз  ER-діаграму у синтаксисі Mermaid. врахуй усі нові критерії

## Сутності, атрибути та зв'язки

-User (Користувач): має 'id', 'email', 'nickname'.
    -Створює багато Playlist (1:N).
    -Може додавати багато Track у "Вибране" (N:M).
    -Може заблокувати багато Artist (N:M).
    -Може заблокувати багато Track (N:M).

-Artist (Виконавець): має 'id', 'name'.
    -Випускає багато Album (N:M).
    -Є автором багатьох пісень (N:M).

-Album (Альбом): має 'id', 'title', 'release_year'.
    -Може належати багатьом Artist (N:M).
    -Містить багато Track (N:M).
    
-Track (Трек):  має 'id', 'title', 'duration_seconds'.
    -Може належати багатьом Album (N:M).
    -Може мати кількох авторів Artist (N:M).

-Playlist (Плейліст): має 'id', 'title', 'created_at'.
    -Належить одному User (N:1).
    -Містить багато Track (M:N).

## Критерії прийняття 
1.Усi ідентифікатори ('id') мають тип 'UUID'.
2.Модель відповідає 3-й нормальній формі (3-NF).
3.Зв'язки N:M відображаються прямо між сутностями, без додаткових сполучних таблиць.
4.Результат має бути у форматі Mermaid ('erDiagram').
5.Для зв'язків 1:N сутність на боці «багато» повинна явно містити поле зовнішнього ключа з позначкою 'FK'.
6.Поля з часом повинні мати тип 'timestamp', а не 'datetime'.

## 2. Друга відповідь AI

erDiagram
    USER {
        UUID id
        string email
        string nickname
    }
    
    ARTIST {
        UUID id
        string name
    }
    
    ALBUM {
        UUID id
        string title
        int release_year
    }
    
    TRACK {
        UUID id
        string title
        int duration_seconds
    }
    
    PLAYLIST {
        UUID id
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

## Третій промпт
оновлено перший пункт критеріїв: 1.Усi ідентифікатори ('id') мають тип 'UUID' та обов'язково позначені як 'PK'. згенеруй діаграму ще раз з урахуванням цього

## 2. Третя відповідь AI

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