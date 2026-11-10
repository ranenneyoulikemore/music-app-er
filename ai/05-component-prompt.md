### Перший промпт
Нижче spec.md. Побудуй UML Component Diagram у PlantUML за цією специфікацією.
Формат: package/rectangle "Music Streaming System" як межа; компоненти за шарами; interface через lollipop/socket (provided/required) з іменами з таблиці; зовнішні компоненти поза межею. Видай лише код PlantUML.

# Завдання 5. Компонентна модель музичного сервісу

## Намір
Описати архітектуру системи (UI, API, сервіси, Data Access, БД, зовнішні інтеграції) так, щоб вона трасувалась до ER (Завд. 1), вимог/UC (Завд. 2), Sequence UC-02 (Завд. 3) і Activity UC-02 (Завд. 4).

## Межа системи
Усередині "Music Streaming System": AppClient, ApiGateway, AuthService, AudioService, BlockService, PlaylistService, SearchService, CatalogImportService, TrackRepository, UserDataRepository, MusicDB, LogStore.
Ззовні: AudioStorageProvider (CDN, інфраструктура, не актор — ADR-0003), MetadataProvider (зовнішня система, сама ініціює імпорт — UC-14).

## Компоненти
| Шар | Компонент | Provided-інтерфейс | Required-інтерфейс | Трасування |
| :-- | :-- | :-- | :-- | :-- |
| UI | AppClient (+OfflineStore) | — | IApi | UC-01..13, REQ-10 |
| API | ApiGateway | IApi | IAuth, IAudio, IBlock, IPlaylist, ISearch | ADR-0006 |
| Service | AuthService | IAuth (validateToken, isPremium) | — | REQ-01, REQ-09 |
| Service | AudioService | IAudio (playTrack, playNextInQueue, downloadPlaylist) | IBlock, ITrackData, IUserData, IAudioStorage, IAuth | UC-02, UC-03, UC-12, REQ-01/02/09, SC-01/02/05 |
| Service | BlockService | IBlock (checkBlockStatus, blockArtist, blockTrack) | IUserData | UC-09..11, REQ-03/04 |
| Service | PlaylistService | IPlaylist (createPlaylist, addTrack, getPlaylistPage, addFavorite) | IUserData | UC-04..08, REQ-05/06/07 |
| Service | SearchService | ISearch (search) | ITrackData | UC-01, REQ-08 |
| Service | CatalogImportService | ICatalogImport (importCatalogUpdate) | ITrackData, ILog | UC-14, REQ-11, NFR-02 |
| Data Access | TrackRepository | ITrackData (getTrackMetadata, search, upsert) | SQL до MusicDB | ARTIST, ALBUM, TRACK |
| Data Access | UserDataRepository | IUserData (blocks, favorites, playlists) | SQL до MusicDB | USER, PLAYLIST, blocks, favorites |
| DB | MusicDB | SQL | — | ER Завд. 1 |
| Infra | LogStore | ILog (writeError) | — | NFR-02 |
| Зовнішній | AudioStorageProvider | IAudioStorage (requestStream) | — | SC-01/02 |
| Зовнішній | MetadataProvider | — | ICatalogImport | UC-14 |

## Критерії прийняття
- AC-1: Межа системи показана явно; зовнішніх компонентів рівно два: AudioStorageProvider, MetadataProvider.
- AC-2: Кожен компонент і залежність трасуються до UC/REQ/ER/Seq/Activity.
- AC-3: Кожна залежність між компонентами іде через іменований інтерфейс (lollipop/socket).
- AC-4: Імена AppClient, AudioService, BlockService, TrackRepository, AudioStorageProvider збігаються з Sequence (Завд. 3).
- AC-5: AudioService викликає IBlock перед ITrackData та IAudioStorage (ADR-0004, Activity Завд. 4).
- AC-6: AppClient взаємодіє з бекендом лише через ApiGateway (ADR-0006); AppClient не залежить від сервісів і репозиторіїв.
- AC-7: До MusicDB звертаються лише TrackRepository і UserDataRepository; сервіси до БД напряму не звертаються.
- AC-8: MetadataProvider залежить від ICatalogImport; система не викликає MetadataProvider (ADR-0003).
- AC-9: CatalogImportService залежить від ILog (NFR-02, ADR-0002).
- AC-10: Артефакт — PlantUML component diagram + один рендер.

## Припущення
- ApiGateway у Sequence (Завд. 3) прозорий: лише маршрутизація й перевірка токена.
- Статус Premium віддає AuthService (isPremium); у ER (Завд. 1) цього поля немає, ER не змінюється.

### Перша відповідь АІ
@startuml
left to right direction
skinparam componentStyle uml2

component "AudioStorageProvider" as AudioStorageProvider <<external>>
component "MetadataProvider" as MetadataProvider <<external>>

() "IAudioStorage" as IAudioStorage
AudioStorageProvider -up- IAudioStorage

() "ICatalogImport" as ICatalogImport
MetadataProvider -up- ICatalogImport

package "Music Streaming System" {
    
    component "AppClient" as AppClient
    component "ApiGateway" as ApiGateway
    
    package "Services Layer" {
        component "AuthService" as AuthService
        component "AudioService" as AudioService
        component "BlockService" as BlockService
        component "PlaylistService" as PlaylistService
        component "SearchService" as SearchService
        component "CatalogImportService" as CatalogImportService
    }
    
    package "Data Access Layer" {
        component "TrackRepository" as TrackRepository
        component "UserDataRepository" as UserDataRepository
    }
    
    database "MusicDB" as MusicDB
    component "LogStore" as LogStore
    
    () "IApi" as IApi
    () "IAuth" as IAuth
    () "IAudio" as IAudio
    () "IBlock" as IBlock
    () "IPlaylist" as IPlaylist
    () "ISearch" as ISearch
    () "ITrackData" as ITrackData
    () "IUserData" as IUserData
    () "SQL" as SQL
    () "ILog" as ILog
}

ApiGateway -up- IApi
AuthService -up- IAuth
AudioService -up- IAudio
BlockService -up- IBlock
PlaylistService -up- IPlaylist
SearchService -up- ISearch
TrackRepository -up- ITrackData
UserDataRepository -up- IUserData
MusicDB -up- SQL
LogStore -up- ILog

AppClient --( IApi
AppClient --( IBlock

ApiGateway --( IAuth
ApiGateway --( IAudio
ApiGateway --( IBlock
ApiGateway --( IPlaylist
ApiGateway --( ISearch

AudioService --( IBlock
AudioService --( IUserData
AudioService --( IAudioStorage
AudioService --( IAuth
AudioService --( SQL

BlockService --( IUserData

PlaylistService --( IUserData

SearchService --( ITrackData

CatalogImportService --( ICatalogImport
CatalogImportService --( ITrackData

TrackRepository --( SQL
UserDataRepository --( SQL

@enduml

### Другий промпт
перероби діаграму під спек 
# Завдання 5. Компонентна модель музичного сервісу

## Намір
Описати архітектуру системи (UI, API, сервіси, Data Access, БД, зовнішні інтеграції) так, щоб вона трасувалась до ER (Завд. 1), вимог/UC (Завд. 2), Sequence UC-02 (Завд. 3) і Activity UC-02 (Завд. 4).

## Межа системи
Усередині "Music Streaming System": AppClient, ApiGateway, AuthService, AudioService, BlockService, PlaylistService, SearchService, CatalogImportService, TrackRepository, UserDataRepository, MusicDB, LogStore.
Ззовні: AudioStorageProvider (CDN, інфраструктура, не актор — ADR-0003), MetadataProvider (зовнішня система, сама ініціює імпорт — UC-14).

## Компоненти
| Шар | Компонент | Provided-інтерфейс | Required-інтерфейс | Трасування |
| :-- | :-- | :-- | :-- | :-- |
| UI | AppClient (+OfflineStore) | — | IApi | UC-01..13, REQ-10 |
| API | ApiGateway | IApi | IAuth, IAudio, IBlock, IPlaylist, ISearch | ADR-0006 |
| Service | AuthService | IAuth (validateToken, isPremium) | — | REQ-01, REQ-09 |
| Service | AudioService | IAudio (playTrack, playNextInQueue, downloadPlaylist) | IBlock, ITrackData, IUserData, IAudioStorage, IAuth | UC-02, UC-03, UC-12, REQ-01/02/09, SC-01/02/05 |
| Service | BlockService | IBlock (checkBlockStatus, blockArtist, blockTrack) | IUserData | UC-09..11, REQ-03/04 |
| Service | PlaylistService | IPlaylist (createPlaylist, addTrack, getPlaylistPage, addFavorite) | IUserData | UC-04..08, REQ-05/06/07 |
| Service | SearchService | ISearch (search) | ITrackData | UC-01, REQ-08 |
| Service | CatalogImportService | ICatalogImport (importCatalogUpdate) | ITrackData, ILog | UC-14, REQ-11, NFR-02 |
| Data Access | TrackRepository | ITrackData (getTrackMetadata, search, upsert) | SQL до MusicDB | ARTIST, ALBUM, TRACK |
| Data Access | UserDataRepository | IUserData (blocks, favorites, playlists) | SQL до MusicDB | USER, PLAYLIST, blocks, favorites |
| DB | MusicDB | SQL | — | ER Завд. 1 |
| Infra | LogStore | ILog (writeError) | — | NFR-02 |
| Зовнішній | AudioStorageProvider | IAudioStorage (requestStream) | — | SC-01/02 |
| Зовнішній | MetadataProvider | — | ICatalogImport | UC-14 |

## Критерії прийняття
- AC-1: Межа системи показана явно; зовнішніх компонентів рівно два: AudioStorageProvider, MetadataProvider.
- AC-2: Кожен компонент і залежність трасуються до UC/REQ/ER/Seq/Activity.
- AC-3: Кожна залежність між компонентами іде через іменований інтерфейс (lollipop/socket).
- AC-4: Імена AppClient, AudioService, BlockService, TrackRepository, AudioStorageProvider збігаються з Sequence (Завд. 3).
- AC-5: AudioService викликає IBlock перед ITrackData та IAudioStorage (ADR-0004, Activity Завд. 4).
- AC-6: AppClient взаємодіє з бекендом лише через ApiGateway (ADR-0006).
  - AC-6а: AppClient не має залежностей від BlockService, AudioService, PlaylistService, SearchService, репозиторіїв; єдина залежність — IApi.
- AC-7: До MusicDB звертаються лише TrackRepository і UserDataRepository.
  - AC-7а: Сервіси залежать від БД лише транзитивно через ITrackData та IUserData; прямих залежностей сервіс -> SQL/MusicDB немає. AudioService requires ITrackData.
- AC-8: MetadataProvider залежить від ICatalogImport; система не викликає MetadataProvider (ADR-0003).
  - AC-8а: ICatalogImport надається CatalogImportService (lollipop); MetadataProvider — required-сторона (socket).
- AC-9: CatalogImportService залежить від ILog (NFR-02, ADR-0002).
- AC-10: Артефакт — PlantUML component diagram + один рендер.

## Припущення
- ApiGateway у Sequence (Завд. 3) прозорий: лише маршрутизація й перевірка токена.
- Статус Premium віддає AuthService (isPremium); у ER (Завд. 1) цього поля немає, ER не змінюється.