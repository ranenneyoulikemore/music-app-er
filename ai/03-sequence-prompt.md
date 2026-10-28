## Перший промпт 
Проаналізуй spec.md та згенеруй 3 окремі Sequence-діаграми у синтаксисі Mermaid для сценаріїв SC-01, SC-02 та SC-05. # Специфікація взаємодії: UC-02 "Відтворювати TRACK"

## 1. Мета
Описати внутрішню взаємодію компонентів системи, сервісів та баз даних під час відтворення аудіотреків (UC-02). Документ деталізує обробку успішного запуску потоку, обробку таймаутів від аудіосховища та механіку автовідтворення черги з пропуском заблокованого контенту.

## 2. Учасники (Participants) та зв'язок із моделлю даних
- **User** — авторизований слухач (відповідає сутності `USER`).
- **AppClient** — UI-клієнт (мобільний або веб-застосунок).
- **AudioService** — бекенд-сервіс, який керує сесією відтворення.
- **BlockService** — сервіс перевірки чорних списків (працює зі таблицями блокувань `blocks` для `ARTIST` та `TRACK`).
- **TrackRepository** — доступ до даних сутностей `TRACK`, `ALBUM`, `ARTIST`.
- **AudioStorageProvider** — зовнішнє сховище / CDN для отримання аудіофайлів.

## 3. Сценарії та послідовність дій
1. **SC-01 (Успішне відтворення треку):** `User` тисне `pressPlay(trackId)` -> `AppClient` викликає `AudioService.playTrack()` -> `AudioService` запитує `BlockService` -> завантажує метадані з `TrackRepository` -> запитує потік в `AudioStorageProvider` -> повертає `200 OK (AudioStreamDTO)`.
2. **SC-02 (Джерело недоступне / Таймаут 5 с):** `AudioService` звертається до `AudioStorageProvider`. Якщо відповіді немає протягом 5000 мс (5 с), сервіс перериває очікування й повертає `504 Gateway Timeout` без запуску плеєра.
3. **SC-05 (Автовідтворення з пропуском):** `AudioService` у циклі `loop` перебирає треки плейліста. Якщо `BlockService` повертає `isBlocked = true`, трек пропускається, і сервіс одразу переходить до наступного.

## 4. Критерії прийняття (Acceptance Criteria)
- **AC-1:** Для кожного сценарію (SC-01, SC-02, SC-05) побудовано окрему Sequence-діаграму.
- **AC-2:** Усі виклики методів мають чіткі сигнатури та типи повернення (`methodName(param: Type): ReturnType`).
- **AC-3:** Activation bars повністю огортають блоки розгалужень (`alt`/`loop`) і не обриваються всередині умовних гілок.
- **AC-4:** Учасники діаграм строго відповідають сутностям ER-моделі та сценаріям Gherkin.

## Відповідь АІ

sequenceDiagram
    autonumber
    actor User
    participant AppClient
    participant AudioService
    participant BlockService
    participant TrackRepository
    participant AudioStorageProvider

    User->>AppClient: pressPlay(trackId: String)
    activate AppClient
    AppClient->>AudioService: playTrack(userId: String, trackId: String): AudioStreamDTO
    activate AudioService
    AudioService->>BlockService: checkBlockStatus(userId: String, trackId: String): BlockStatusDTO
    activate BlockService
    BlockService-->>AudioService: BlockStatusDTO(isBlocked = false)
    deactivate BlockService
    AudioService->>TrackRepository: getTrackMetadata(trackId: String): TrackDTO
    activate TrackRepository
    TrackRepository-->>AudioService: TrackDTO
    deactivate TrackRepository
    AudioService->>AudioStorageProvider: requestStream(trackId: String): AudioStream
    activate AudioStorageProvider
    AudioStorageProvider-->>AudioService: AudioStream
    deactivate AudioStorageProvider
    AudioService-->>AppClient: 200 OK (AudioStreamDTO)
    deactivate AudioService
    AppClient-->>User: renderPlayer(stream: AudioStreamDTO)
    deactivate AppClient

    sequenceDiagram
    autonumber
    actor User
    participant AppClient
    participant AudioService
    participant BlockService
    participant TrackRepository
    participant AudioStorageProvider

    User->>AppClient: pressPlay(trackId: String)
    activate AppClient
    AppClient->>AudioService: playTrack(userId: String, trackId: String): AudioStreamDTO
    activate AudioService
    AudioService->>BlockService: checkBlockStatus(userId: String, trackId: String): BlockStatusDTO
    activate BlockService
    BlockService-->>AudioService: BlockStatusDTO(isBlocked = false)
    deactivate BlockService
    AudioService->>TrackRepository: getTrackMetadata(trackId: String): TrackDTO
    activate TrackRepository
    TrackRepository-->>AudioService: TrackDTO
    deactivate TrackRepository
    AudioService->>AudioStorageProvider: requestStream(trackId: String): AudioStream
    activate AudioStorageProvider

    alt Таймаут очікування джерела (>= 5000 мс)
        Note over AudioService,AudioStorageProvider: Немає відповіді протягом 5000 мс
        AudioService->>AudioService: handleTimeout(): Void
        AudioService-->>AppClient: 504 Gateway Timeout
        AppClient-->>User: showError(message: String)
    end

    deactivate AudioStorageProvider
    deactivate AudioService
    deactivate AppClient

    sequenceDiagram
    autonumber
    actor User
    participant AppClient
    participant AudioService
    participant BlockService
    participant TrackRepository
    participant AudioStorageProvider

    User->>AppClient: startAutoplay(playlistId: String)
    activate AppClient
    AppClient->>AudioService: playNextInQueue(userId: String, playlistId: String): AudioStreamDTO
    activate AudioService

    loop Перебір треків плейліста
        AudioService->>BlockService: checkBlockStatus(userId: String, trackId: String): BlockStatusDTO
        activate BlockService
        BlockService-->>AudioService: BlockStatusDTO(isBlocked: Boolean)
        deactivate BlockService

        alt isBlocked == true
            AudioService->>AudioService: skipTrack(trackId: String): Void
        else isBlocked == false
            AudioService->>TrackRepository: getTrackMetadata(trackId: String): TrackDTO
            activate TrackRepository
            TrackRepository-->>AudioService: TrackDTO
            deactivate TrackRepository
            AudioService->>AudioStorageProvider: requestStream(trackId: String): AudioStream
            activate AudioStorageProvider
            AudioStorageProvider-->>AudioService: AudioStream
            deactivate AudioStorageProvider
            AudioService-->>AppClient: 200 OK (AudioStreamDTO)
            AppClient-->>User: renderPlayer(stream: AudioStreamDTO)
        end
    end

    deactivate AudioService
    deactivate AppClient

## Другий промпт 
Проаналізуй spec.md та виправ згенеровані Sequence-діаграми для SC-01, SC-02 та SC-05 відповідно до нових критеріїв прийняття # Специфікація взаємодії: UC-02 "Відтворювати TRACK"

## 1. Мета
Описати внутрішню взаємодію компонентів системи, сервісів та баз даних під час відтворення аудіотреків (UC-02). Документ деталізує обробку успішного запуску потоку, обробку таймаутів від аудіосховища та механіку автовідтворення черги з пропуском заблокованого контенту.

## 2. Учасники (Participants) та зв'язок із моделлю даних
- **User** — авторизований слухач (відповідає сутності `USER`).
- **AppClient** — UI-клієнт (мобільний або веб-застосунок).
- **AudioService** — бекенд-сервіс, який керує сесією відтворення.
- **BlockService** — сервіс перевірки чорних списків (працює зі таблицями блокувань `blocks` для `ARTIST` та `TRACK`).
- **TrackRepository** — доступ до даних сутностей `TRACK`, `ALBUM`, `ARTIST`.
- **AudioStorageProvider** — зовнішнє сховище / CDN для отримання аудіофайлів.

## 3. Сценарії та послідовність дій
1. **SC-01 (Успішне відтворення треку):** `User` тисне `pressPlay(trackId)` -> `AppClient` викликає `AudioService.playTrack()` -> `AudioService` запитує `BlockService` -> завантажує метадані з `TrackRepository` -> запитує потік в `AudioStorageProvider` -> повертає `200 OK (AudioStreamDTO)`.
2. **SC-02 (Джерело недоступне / Таймаут 5 с):** `AudioService` звертається до `AudioStorageProvider`. Якщо відповіді немає протягом 5000 мс (5 с), сервіс перериває очікування й повертає `504 Gateway Timeout` без запуску плеєра.
3. **SC-05 (Автовідтворення з пропуском):** `AudioService` у циклі `loop` перебирає треки плейліста. Якщо `BlockService` повертає `isBlocked = true`, трек пропускається, і сервіс одразу переходить до наступного.

## 4. Критерії прийняття (Acceptance Criteria)
- **AC-1:** Для кожного сценарію (SC-01, SC-02, SC-05) побудовано окрему Sequence-діаграму.
- **AC-2:** Усі виклики методів мають чіткі сигнатури та типи повернення (`methodName(param: Type): ReturnType`).
- **AC-3:** Activation bars повністю огортають блоки розгалужень (`alt`/`loop`) і не обриваються всередині умовних гілок.
- **AC-4:** Учасники діаграм строго відповідають сутностям ER-моделі та сценаріям Gherkin.
- **AC-2:** Кожна діаграма містить щонайменше один складний фрагмент: SC-01 (`alt`), SC-02 (`opt`), SC-05 (`loop` + `alt`).

## Друга відповідь АІ

sequenceDiagram
    autonumber
    actor User
    participant AppClient
    participant AudioService
    participant BlockService
    participant TrackRepository
    participant AudioStorageProvider

    User->>AppClient: pressPlay(trackId: String): Void
    activate AppClient
    AppClient->>AudioService: playTrack(userId: String, trackId: String): AudioStreamDTO
    activate AudioService
    AudioService->>BlockService: checkBlockStatus(userId: String, trackId: String): BlockStatusDTO
    activate BlockService
    BlockService-->>AudioService: BlockStatusDTO(isBlocked = false)
    deactivate BlockService

    alt isBlocked == false
        AudioService->>TrackRepository: getTrackMetadata(trackId: String): TrackDTO
        activate TrackRepository
        TrackRepository-->>AudioService: TrackDTO
        deactivate TrackRepository
        AudioService->>AudioStorageProvider: requestStream(trackId: String): AudioStream
        activate AudioStorageProvider
        AudioStorageProvider-->>AudioService: AudioStream
        deactivate AudioStorageProvider
        AudioService-->>AppClient: 200 OK (AudioStreamDTO)
        AppClient-->>User: renderPlayer(stream: AudioStreamDTO): Void
    else isBlocked == true
        AudioService-->>AppClient: 403 Forbidden (TrackBlockedException)
        AppClient-->>User: showError(message: String): Void
    end

    deactivate AudioService
    deactivate AppClient

    sequenceDiagram
    autonumber
    actor User
    participant AppClient
    participant AudioService
    participant BlockService
    participant TrackRepository
    participant AudioStorageProvider

    User->>AppClient: pressPlay(trackId: String): Void
    activate AppClient
    AppClient->>AudioService: playTrack(userId: String, trackId: String): AudioStreamDTO
    activate AudioService
    AudioService->>BlockService: checkBlockStatus(userId: String, trackId: String): BlockStatusDTO
    activate BlockService
    BlockService-->>AudioService: BlockStatusDTO(isBlocked = false)
    deactivate BlockService
    AudioService->>TrackRepository: getTrackMetadata(trackId: String): TrackDTO
    activate TrackRepository
    TrackRepository-->>AudioService: TrackDTO
    deactivate TrackRepository
    AudioService->>AudioStorageProvider: requestStream(trackId: String): AudioStream
    activate AudioStorageProvider

    opt Таймаут відповіді >= 5000 мс
        Note over AudioStorageProvider,AudioService: Час очікування перевищено (5000 мс)
        AudioService-->>AppClient: 504 Gateway Timeout
        AppClient-->>User: showError(message: String): Void
    end

    deactivate AudioStorageProvider
    deactivate AudioService
    deactivate AppClient

    sequenceDiagram
    autonumber
    actor User
    participant AppClient
    participant AudioService
    participant BlockService
    participant TrackRepository
    participant AudioStorageProvider

    User->>AppClient: startAutoplay(playlistId: String): Void
    activate AppClient
    AppClient->>AudioService: playNextInQueue(userId: String, playlistId: String): AudioStreamDTO
    activate AudioService

    loop Для кожного треку в плейлісті
        AudioService->>BlockService: checkBlockStatus(userId: String, trackId: String): BlockStatusDTO
        activate BlockService
        BlockService-->>AudioService: BlockStatusDTO(isBlocked: Boolean)
        deactivate BlockService

        alt isBlocked == true
            AudioService->>AudioService: skipTrack(trackId: String): Void
        else isBlocked == false
            AudioService->>TrackRepository: getTrackMetadata(trackId: String): TrackDTO
            activate TrackRepository
            TrackRepository-->>AudioService: TrackDTO
            deactivate TrackRepository
            AudioService->>AudioStorageProvider: requestStream(trackId: String): AudioStream
            activate AudioStorageProvider
            AudioStorageProvider-->>AudioService: AudioStream
            deactivate AudioStorageProvider
            AudioService-->>AppClient: 200 OK (AudioStreamDTO)
            AppClient-->>User: renderPlayer(stream: AudioStreamDTO): Void
        end
    end

    deactivate AudioService
    deactivate AppClient