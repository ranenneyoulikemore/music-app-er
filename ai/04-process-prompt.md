## Перший промпт 
Побудуй Activity-діаграму у форматі Mermaid Flowchart на основі спек# UC-02 "Відтворення треку"

## 1. Мета
Описати єдиний флоу запуску аудіотреку, який об'єднує відразу три сценарії: нормальне відтворення (SC-01), відмову через блокування (SC-05) та помилку таймауту від CDN (SC-02).

## 2. Формат та учасники
- **Формат:** Activity-діаграма. Вибрала цей варіант, бо GitHub нормально рендерить Mermaid-схеми в маркдауні, і зручно розбити процес на пули та намалювати паралельні завдання.
- **Пули (Swimlanes):** `User`, `AppClient`, `AudioService`, `AudioStorageProvider`.

## 3. Кроки
1. `User` натискає кнопку відтворення.
2. `AudioService` спочатку перевіряє, чи не кинув користувач цей трек у чорний список. Якщо заблоковано — процес переривається (віддаємо 403).
3. Якщо все нормально, процес роздвоюється:
   - перша гілка йде в БД за метаданими;
   - друга гілка паралельно відправляє запит до `AudioStorageProvider` за самим аудіопотоком.
4. Чекаємо на стрім. Якщо `AudioStorageProvider` не відповідає довше ніж 5 секунд — обриваємо процес по таймауту (помилка 504).
5. Якщо сховище відповіло вчасно, гілки зливаються, клієнт (`AppClient`) отримує дані й починає програвати музику.

## 4. Критерії прийняття 
- **AC-1:** На схемі є явні початковий та кінцевий вузли (Start/End).
- **AC-2:** Є блок розгалуження (if/else), де ми перевіряємо статус `isBlocked`.
- **AC-3:** Є паралельне виконання (через Fork/Join) для одночасного стягування метаданих і аудіофайлу.
- **AC-4:** На схемі показано надсилання повідомлення до іншої системи (`AudioStorageProvider`) і чітко прописана реакція на нього (або успішне отримання потоку, або помилка по таймауту).

## Відповідь АІ

flowchart TD
    subgraph User ["User (Слухач)"]
        Start((● Початок))
        PressPlay["Натискає кнопку 'Play'"]
        ViewError["Бачить повідомлення про помилку"]
        ListenAudio["Слухає аудіотрек"]
        End((◉ Кінець))
    end

    subgraph AppClient ["AppClient (UI-клієнт)"]
        SendPlayReq["Викликає playTrack(userId, trackId)"]
        ShowError403["Відображає помилку: Трек заблоковано"]
        ShowError504["Відображає помилку: Сховище недоступне"]
        RenderPlayer["Ініціалізує та запускає плеєр"]
    end

    subgraph AudioService ["AudioService (Бекенд-сервіс)"]
        CheckBlock{"isBlocked == true?"}
        Return403["Повертає 403 Forbidden"]

        Fork[=== Розгалуження / Fork ===]

        FetchMetadata["Запит метаданих з TrackRepository"]
        ReqStream["Запит аудіопотоку"]
        
        CheckTimeout{"Відповідь отримана\nпротягом 5 с?"}
        Return504["Повертає 504 Gateway Timeout"]

        Join[=== Злиття / Join ===]
        
        PrepareResponse["Формування 200 OK (AudioStreamDTO)"]
    end

    subgraph AudioStorageProvider ["AudioStorageProvider (CDN / Сховище)"]
        ProcessStreamReq["Обробка запиту та передача AudioStream"]
    end

    %% Основна послідовність дій
    Start --> PressPlay
    PressPlay --> SendPlayReq
    SendPlayReq --> CheckBlock

    %% Гілка блокування (SC-05)
    CheckBlock -- Так --> Return403
    Return403 --> ShowError403
    ShowError403 --> ViewError
    ViewError --> End

    %% Паралельне виконання (AC-3)
    CheckBlock -- Ні --> Fork
    Fork --> FetchMetadata
    Fork --> ReqStream

    %% Гілка метаданих
    FetchMetadata --> Join

    %% Гілка аудіопотоку та завантаження з CDN (AC-4, SC-02)
    ReqStream --> ProcessStreamReq
    ProcessStreamReq --> CheckTimeout

    CheckTimeout -- Ні (Таймаут >= 5с) --> Return504
    Return504 --> ShowError504
    ShowError504 --> ViewError

    %% Успішне злиття та відтворення (SC-01)
    CheckTimeout -- Так (< 5с) --> Join
    Join --> PrepareResponse
    PrepareResponse --> RenderPlayer
    RenderPlayer --> ListenAudio
    ListenAudio --> End