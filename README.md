# DebugViewSwift

In-app инструменты отладки для iOS-приложений на UIKit: перехват сети, производительность, инспектор лейаута и 3D-иерархия вью, хранилища, консоль, краш-репорты, подмена геолокации и часового пояса, тестовые пуши.

Это бинарная сборка (XCFramework). Текущая версия — **1.5.0**.

## Содержание

- [Возможности](#возможности)
- [Требования](#требования)
- [Установка](#установка)
- [Быстрый старт](#быстрый-старт)
- [Как открыть панель](#как-открыть-панель)
- [Плавающая кнопка](#плавающая-кнопка)
- [Настройка](#настройка)
- [API](#api)
- [Документация в Xcode](#документация-в-xcode)
- [Только для debug-сборок](#только-для-debug-сборок)
- [Обновление](#обновление)
- [Решение проблем](#решение-проблем)

## Возможности

### Network
Три плитки со счётчиками: **Requests**, **Events**, **Breakpoints**.

- Перехват всех HTTP(S)-запросов `URLSession`: `URLSession.shared` и сессии, созданные из `.default` / `.ephemeral` конфигураций. Ответы доходят до приложения без изменений.
- Список запросов: статус, метод, хост, путь, время, длительность, размер ответа.
- Поиск по URL или методу и фильтр по статусу: **All / Success / Failed / Redirect / Pending** с количеством запросов.
- Запросы без ответа сервера показывают код ошибки так же, как HTTP-статус: `-1009 · Offline`, `-1001 · Timed out`, `-1003 · Host not found`, `-999 · Cancelled` и т.д.
- Детали запроса: заголовки и тело запроса и ответа (JSON форматируется), ошибки.
- Копирование запроса как cURL, шаринг полного отчёта.
- **Events** — события аналитики с параметрами, поиском и JSON. `Analytics.logEvent` Firebase перехватывается автоматически (без зависимости от Firebase); переключатель «Send to Firebase» позволяет смотреть события, не отправляя их. События без Firebase (например, которые в dev-сборке пишутся в консоль) — одной строкой `DebugViewSwift.logAnalyticsEvent(…)`.
- **Брейкпоинты на ответы**: запрос, подходящий под правило, ставится на паузу, панель открывается с редактором ответа. Можно поменять статус, тело и заголовки, вернуть вместо ответа ошибку (offline, timeout…) или отправить как есть. Если запрос упал без ответа, можно подсунуть свой ответ. Правило — часть URL (`*` — любые символы) и, по желанию, метод; добавляется кнопкой ⏸ в деталях запроса, на плитке **Breakpoints** или из кода. Правила сохраняются между запусками.

### Tools
Плитки по две в ряд, сгруппированы по категориям: Monitoring, Interface, Storage, App, Simulation. Каждая плитка открывает свой экран, в подзаголовке — текущее состояние.

#### Interface
- **3D view hierarchy** — экран раскладывается на слои, как Debug View Hierarchy в Xcode:
  - вращение одним пальцем, зум щипком, сдвиг двумя пальцами;
  - расстояние между слоями, каркасы пустых контейнеров, подписи с именами классов;
  - дерево вью с отступами по вложенности, выделение синхронизировано со сценой;
  - тап по слою — класс, фрейм, уровень, супервью.
- **Layout inspector** — тап или ведение пальцем по экрану выбирает вью:
  - синяя рамка — выбранная вью, фиолетовый пунктир — супервью, серый — соседние вью;
  - стрелки с расстояниями: фиолетовые — до краёв супервью, оранжевые — до ближайших соседей;
  - карточка с классом, фреймом и отступами; кнопка «вверх» выбирает супервью.
- Сетка 8pt, цветные рамки вокруг всех вью, отображение касаний, замедление анимаций ×10, принудительная светлая / тёмная тема.

#### Storage
- UserDefaults, Keychain, Cookies — просмотр, поиск, удаление.
- Браузер файлов песочницы: просмотр текстовых, JSON и plist-файлов, шаринг, удаление.

#### Monitoring, App, Simulation
- **Performance** — CPU, память и FPS в реальном времени с графиками за минуту; компактный виджет поверх приложения.
- **Console** — вывод `print`, `NSLog` и stderr.
- **Memory leaks** — счётчик живых контроллеров приложения и своих объектов, поиск утечек. Контроллер, который ушёл с экрана, но через 3 с всё ещё в памяти и не прикреплён ни к одному окну, помечается фиолетовым «!» как в Xcode, поверх приложения показывается «Possible leak: …». Сводка (контроллеры, объекты, утечки, память), список по типам с количеством экземпляров, детали с адресами и подсказками, где искать retain cycle; опциональная плашка-счётчик поверх приложения.
- **App & Device** — версия, модель, система, экран, локаль; UDID на симуляторе и Vendor ID (IDFV); свои поля (например, окружение или ID пользователя).
- **Crash reports** — неперехваченные исключения и фатальные сигналы, сохраняются между запусками.
- **Custom actions** — свои действия: «Сбросить онбординг», «Разлогинить» и т.п.
- **Location** — подмена геолокации для `CLLocationManager` (делегат и свойство `location`): пресеты городов или свои координаты. `CLLocationUpdate` / `CLMonitor` (iOS 17+) не подменяются.
- **Time zone** — подмена часового пояса для `TimeZone.current`, `Calendar.current` и новых `DateFormatter`.
- **Push notifications** — композер как в Firebase console: заголовок, текст, картинка, звук, бейдж, задержка, data-поля, сохранённые шаблоны. Обычный пуш приходит как локальное уведомление в `UNUserNotificationCenterDelegate` приложения, silent — в `application(_:didReceiveRemoteNotification:fetchCompletionHandler:)`.
  - **Edit as JSON** — вставить готовый payload APNs (`aps`, `fcm_options`, свои ключи) или FCM (`notification` / `data`, `message` из HTTP v1): поля заполнятся из него, вложенные значения своих ключей уйдут как JSON.
  - Картинка — по ссылке или **Choose from Photos** (без доступа к галерее, через системный пикер).

Выбранные локация и часовой пояс сохраняются между запусками.

### Scenarios
Вкладка появляется, если приложение зарегистрировало хотя бы одну группу сценариев. Сценарий — состояние приложения, в которое можно перейти в один тап. Например, тестовые аккаунты (премиум, базовый, вход через почту или Apple ID), чтобы не проходить логин руками. Библиотека не знает, что делает сценарий: это код приложения. Текущий сценарий отмечается галочкой, во время выполнения на плитке крутится спиннер, ошибка показывается в панели, после успеха панель закрывается. Аргумент запуска `-DebugViewSwiftScenario <id>` применяет сценарий сразу при старте.

## Требования

- iOS 16+
- Xcode 26.4.1 или новее — бинарник собран этой версией, более старый Xcode не сможет его импортировать.

## Установка

### Через Xcode

1. **File → Add Package Dependencies…**
2. Вставьте URL: `https://github.com/N1kita-ni/DebugViewSwift-Binary.git`
3. **Dependency Rule:** *Up to Next Major Version* от `1.5.0`.
4. Добавьте продукт **DebugViewSwift** в таргет приложения.

### Через Package.swift

```swift
dependencies: [
    .package(url: "https://github.com/N1kita-ni/DebugViewSwift-Binary.git", from: "1.5.0")
],
targets: [
    .target(
        name: "MyApp",
        dependencies: [.product(name: "DebugViewSwift", package: "DebugViewSwift-Binary")]
    )
]
```

> Имя пакета берётся из URL (`DebugViewSwift-Binary`), поэтому нужна форма `.product(name:package:)` — короткая строка `"DebugViewSwift"` не резолвится.

## Быстрый старт

```swift
import DebugViewSwift
import UIKit

@main
final class AppDelegate: UIResponder, UIApplicationDelegate {
    func application(
        _ application: UIApplication,
        didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
    ) -> Bool {
        #if DEBUG
        DebugViewSwift.setup()
        #endif
        return true
    }
}
```

Вызывайте `setup()` как можно раньше — тогда запросы и вывод консоли перехватываются с самого запуска. Повторные вызовы игнорируются.

## Как открыть панель

- **Тап по плавающей кнопке** (жучок или счётчик запросов).
- **Шейк устройства.** В симуляторе: меню **Device → Shake** или **⌃⌘Z**; из терминала:
  ```bash
  xcrun simctl notify_post booted com.apple.UIKit.SimulatorShake
  ```
- **Из кода:** `DebugViewSwift.presentPanel()` — например, по долгому нажатию на номер версии в настройках приложения.

## Плавающая кнопка

- Кнопку можно перетаскивать, она прилипает к краям экрана.
- **Скрыть:** перетащите кнопку на ✕ внизу экрана или выключите **Tools → Interface → Floating button**.
- **Вернуть:** шейк → **Tools → Interface → Floating button**, или `DebugViewSwift.showFloatingButton()`.
- Выбор запоминается между запусками.
- **Во время записи экрана** кнопка прячется автоматически и возвращается после неё.

## Настройка

```swift
var configuration = DebugViewSwift.Configuration()
configuration.showsFloatingButton = true
configuration.enablesShakeGesture = true
configuration.enablesNetworkLogging = true
configuration.enablesConsoleCapture = true
configuration.enablesLocationSimulation = true
configuration.enablesCrashReporting = false        // если в сборке есть Crashlytics / Sentry
configuration.ignoredHosts = ["analytics.example.com"]
configuration.maxNetworkRecords = 300

DebugViewSwift.setup(configuration)
```

| Параметр | По умолчанию | Описание |
|---|---|---|
| `showsFloatingButton` | `true` | Видна ли кнопка по умолчанию — пока пользователь сам не скрыл / не включил её |
| `enablesShakeGesture` | `true` | Открывать / закрывать панель шейком |
| `enablesNetworkLogging` | `true` | Перехватывать запросы `URLSession` |
| `enablesConsoleCapture` | `true` | Дублировать stdout / stderr во встроенную консоль (консоль Xcode продолжает работать) |
| `enablesCrashReporting` | `true` | Записывать краши. **Выключите, если в сборке есть другой крэш-репортер** — они конфликтуют за обработчики сигналов |
| `enablesLocationSimulation` | `true` | Разрешить подмену геолокации в Tools → Location (свизлинг `CLLocationManager`; пока локация не выбрана, вызовы проходят без изменений) |
| `ignoredHosts` | `[]` | Хосты, запросы к которым не логируются (точное совпадение) |
| `maxNetworkRecords` | `500` | Сколько последних запросов хранить |

## API

### Панель

```swift
DebugViewSwift.presentPanel()     // открыть
DebugViewSwift.dismissPanel()     // закрыть
DebugViewSwift.togglePanel()      // переключить
DebugViewSwift.isSetUp            // true после setup()
```

### Плавающая кнопка

```swift
DebugViewSwift.hideFloatingButton()       // скрыть (запоминается)
DebugViewSwift.showFloatingButton()       // показать (запоминается)
DebugViewSwift.isFloatingButtonHidden     // скрыта ли пользователем или через API
```

Временное скрытие во время записи экрана не меняет `isFloatingButtonHidden`.

### Свои действия — Tools → Custom actions

```swift
DebugViewSwift.addAction(title: "Reset onboarding") {
    UserDefaults.standard.removeObject(forKey: "onboardingShown")
}

DebugViewSwift.addAction(title: "Logout") {
    SessionManager.shared.logout()
}
```

### Свои поля — Tools → App & Device → Custom

```swift
DebugViewSwift.setInfo("Staging", forKey: "Environment")
DebugViewSwift.setInfo(user.id, forKey: "User ID")
DebugViewSwift.setInfo(nil, forKey: "User ID")       // удалить поле
```

Повторный вызов с тем же ключом заменяет значение; поля идут в порядке первого добавления.

### События аналитики — Network → Events

Firebase Analytics перехватывается сам (`configuration.capturesFirebaseAnalytics`, по умолчанию `true`). Чтобы события не уходили в Firebase во время отладки, выключите **Send to Firebase** на экране **Network → Events**.

Другие события — одна строка там, где приложение их отправляет или печатает:

```swift
// Тип событий приложения
extension AnalyticsEvent: DebugAnalyticsEvent {
    var debugName: String { name }
    var debugParameters: [String: Any] { parameters }
}

func log(_ event: AnalyticsEvent) {
    #if DEBUG
    print(event)                                  // вместо отправки в dev-сборке
    DebugViewSwift.logAnalyticsEvent(event)
    #else
    Analytics.logEvent(event.name, parameters: event.parameters)
    #endif
}

// Или без протокола, с подписью источника
DebugViewSwift.logAnalyticsEvent("purchase", parameters: ["value": 9.99], source: "Amplitude")
```

`logAnalyticsEvent` можно вызывать с любого потока.

### Утечки памяти — Tools → Memory leaks

Контроллеры приложения считаются автоматически. Свои объекты (view model, координаторы, сервисы) — через `trackLifetime`: если живых экземпляров больше `maxCount`, тип помечается как утечка.

```swift
final class ProfileViewModel {
    init() {
        #if DEBUG
        DebugViewSwift.trackLifetime(of: self, maxCount: 1)
        #endif
    }
}
```

- Утечка контроллера — эвристика (как MLeaksFinder): закрыт, через 3 с жив и не связан ни с одним окном. Контроллер, который приложение специально держит вне экрана (кэш), тоже попадёт в список. Кто именно держит объект, показывает Xcode → Debug Memory Graph.
- **Check now** проверяет все живые контроллеры сразу, не дожидаясь закрытия.
- Выключить: `configuration.enablesLeakDetection = false` (свизлятся `viewDidLoad`, `viewDidAppear(_:)`, `viewDidDisappear(_:)` у `UIViewController`).

### Брейкпоинты на ответы — Network → Breakpoints

```swift
#if DEBUG
DebugViewSwift.addNetworkBreakpoint("api.example.com/profile")
DebugViewSwift.addNetworkBreakpoint("api.example.com/orders/*", method: "POST")
#endif
```

Когда ответ на подходящий запрос получен, приложение его пока не видит: панель открывается на редакторе.

- **Continue** — отправить ответ (с правками или как есть). Поля: статус, тело (JSON форматируется; `Content-Length` пересчитывается) и заголовки (`Name: value` построчно). **Reset** откатывает правки.
- **{ }** — вставить готовый JSON вместо тела (вставка из буфера, форматирование, проверка); `Content-Type` становится `application/json`, пустой статус — `200`.
- **Fail** — вместо ответа вернуть ошибку: Offline (-1009), Timed out (-1001), Host not found (-1003), Connection lost (-1005).
- Если запрос упал без ответа, Continue пробрасывает ошибку, а заполненный статус превращает её в ответ.
- Несколько ответов на паузе обрабатываются по очереди; когда очередь пуста, панель закрывается (если её открыл брейкпоинт). Запрос на паузе виден в списке как `PAUSED`, изменённый — с пометкой «Modified at breakpoint».
- Таймаут приложения продолжает идти (по умолчанию 60 с): если он сработает, приложение получит ошибку таймаута, а редактор перейдёт к следующему ответу.
- Правила включаются, выключаются, редактируются и удаляются в **Network → Breakpoints** (кнопка ⏸ в списке запросов).

### Сценарии — Scenarios

Плитки во вкладке **Scenarios**, например, чтобы входить в тестовые аккаунты без экрана логина:

```swift
#if DEBUG
DebugViewSwift.addScenarioGroup("Accounts", current: { AuthService.shared.currentUser?.email }, scenarios: [
    DebugScenario(id: "premium@test.com", title: "Premium", subtitle: "Email · subscription", systemImage: "crown") {
        try await AuthService.shared.signIn(email: "premium@test.com", password: "…")
        AppRouter.shared.showMain()
    },
    DebugScenario(id: "basic@test.com", title: "Basic", subtitle: "Email") {
        try await AuthService.shared.signIn(email: "basic@test.com", password: "…")
        AppRouter.shared.showMain()
    },
    DebugScenario(id: "guest", title: "Signed out", systemImage: "person.crop.circle.badge.xmark") {
        AuthService.shared.signOut()
    }
])
#endif
```

- `apply` — код приложения (`async throws`, вызывается на main). Пока он выполняется, у плитки спиннер, остальные плитки недоступны. Если `apply` выбросит ошибку, она покажется в панели; при успехе панель закроется.
- Пока выполняется сценарий, key-окном снова становится окно приложения, так что код вида «заменить `rootViewController` у key window» работает как обычно.
- `current` возвращает `id` активного сценария (его плитка отмечается галочкой). Он запрашивается при каждом открытии вкладки, поэтому источником правды остаётся приложение.
- Группа с тем же названием заменяется. Групп может быть несколько (аккаунты, окружение, фича-флаги).
- Запуск с аргументом `-DebugViewSwiftScenario <id>` применяет сценарий, как только зарегистрирована его группа: `xcrun simctl launch booted <bundle id> -DebugViewSwiftScenario premium@test.com`. Если сценарию нужен UI приложения, регистрируйте группу после его создания.
- Sign in with Apple не автоматизировать (системный шит, настоящий Apple ID): для таких аккаунтов нужен debug-эндпоинт бэкенда, выдающий сессию тестового пользователя.
- Держите регистрацию с паролями под `#if DEBUG`, чтобы они не попали в релизную сборку.

## Документация в Xcode

У всего публичного API есть документация — она открывается прямо в Xcode, без браузера:

- **⌥-клик** (Option + клик) по `DebugViewSwift`, `setup`, `Configuration` или любому методу — всплывающее окно Quick Help.
- **Quick Help inspector:** поставьте курсор на символ и откройте правую панель **⌥⌘3** (View → Inspectors → Quick Help) — документация обновляется при перемещении курсора.
- **Автодополнение:** краткое описание показывается внизу списка подсказок.

## Только для debug-сборок

Оборачивайте вызовы в `#if DEBUG`, чтобы инструменты не включались в релизе:

```swift
#if DEBUG
DebugViewSwift.setup()
#endif
```

SwiftPM не умеет подключать пакет только для одной конфигурации, поэтому сам фреймворк попадает и в Release-сборку, но без `setup()` ничего не делает: ни перехвата сети, ни подмены методов, ни окна поверх приложения.

## Обновление

**File → Packages → Update to Latest Package Versions.** Список изменений — на странице [Releases](https://github.com/N1kita-ni/DebugViewSwift-Binary/releases).

## Решение проблем

**`Revision … does not match previously recorded value`**
SwiftPM запомнил другую ревизию тега. Удалите запомненные данные пакета и обновите пакеты:
```bash
rm ~/Library/org.swift.swiftpm/security/fingerprints/debugviewswift-binary-*.json
```
**File → Packages → Reset Package Caches** этот файл не очищает.

**`Failed to build module 'DebugViewSwift'` / `module compiled with Swift … cannot be imported`**
Xcode старше версии, которой собран бинарник. Обновите Xcode до 26.4.1 или новее.

**Шейк не открывает панель**
- Проверьте, что `enablesShakeGesture` не выключен.
- Если контроллер приложения переопределяет `motionEnded(_:with:)`, он должен вызывать `super`.
- В симуляторе используйте **Device → Shake (⌃⌘Z)**.

**Какие-то запросы не видны во вкладке Network**
Перехватываются запросы через `URLSession`. Не попадают: `WKWebView`, сокеты, сессии, у которых `protocolClasses` задаётся вручную после создания конфигурации, а также хосты из `ignoredHosts`.

**Крашей нет в Tools → Crash reports**
- Отчёт появляется после **следующего** запуска приложения.
- Если приложение запущено из Xcode, краш останавливается в отладчике и отчёт может не записаться — проверяйте, запуская приложение с иконки.
- Не включайте вместе с другим крэш-репортером (`enablesCrashReporting = false`).

**Тестовый пуш не показывается, пока приложение открыто**
Баннер в foreground решает `userNotificationCenter(_:willPresent:withCompletionHandler:)` вашего делегата — если он возвращает пустые опции, поставьте задержку 5–10 с и сверните приложение. Если уведомления запрещены, разрешите их в Настройках.

**Подменённая геолокация не приходит**
Подменяются `CLLocationManager` (делегат и `location`). `CLLocationUpdate.liveUpdates()` и `CLMonitor` (iOS 17+) не подменяются. Проверьте, что не выключен `enablesLocationSimulation`.
