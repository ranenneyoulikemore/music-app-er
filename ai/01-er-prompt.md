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