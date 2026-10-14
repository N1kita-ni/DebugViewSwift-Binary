# DebugViewSwift

In-app инструменты отладки для iOS-приложений на UIKit: перехват сети, производительность, инспектор лейаута и 3D-иерархия вью, хранилища, консоль, краш-репорты.

Это бинарная сборка (XCFramework). Текущая версия — **1.2.2**.

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
- Перехват всех HTTP(S)-запросов `URLSession`: `URLSession.shared` и сессии, созданные из `.default` / `.ephemeral` конфигураций. Ответы доходят до приложения без изменений.
- Список запросов: статус, метод, хост, путь, время, длительность, размер ответа.
- Поиск по URL или методу и фильтр по статусу: **All / Success / Failed / Redirect / Pending** с количеством запросов.
- Запросы без ответа сервера показывают код ошибки так же, как HTTP-статус: `-1009 · Offline`, `-1001 · Timed out`, `-1003 · Host not found`, `-999 · Cancelled` и т.д.
- Детали запроса: заголовки и тело запроса и ответа (JSON форматируется), ошибки.
- Копирование запроса как cURL, шаринг полного отчёта.

### Performance
- CPU, память и FPS в реальном времени с графиками за последнюю минуту.
- Компактный виджет с метриками поверх приложения.

### Interface
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

### Resources
- UserDefaults, Keychain, Cookies — просмотр, поиск, удаление.
- Браузер файлов песочницы: просмотр текстовых, JSON и plist-файлов, шаринг, удаление.

### App
- Информация о приложении и устройстве: версия, модель, система, экран, локаль; UDID на симуляторе и Vendor ID (IDFV).
- Свои поля (например, окружение или ID пользователя).
- Консоль: вывод `print`, `NSLog` и stderr.
- Краш-репорты: неперехваченные исключения и фатальные сигналы, сохраняются между запусками.
- Свои действия: «Сбросить онбординг», «Разлогинить» и т.п.

## Требования

- iOS 14+
- Xcode 26.4.1 или новее — бинарник собран этой версией, более старый Xcode не сможет его импортировать.

## Установка

### Через Xcode

1. **File → Add Package Dependencies…**
2. Вставьте URL: `https://github.com/N1kita-ni/DebugViewSwift-Binary.git`
3. **Dependency Rule:** *Up to Next Major Version* от `1.2.2`.
4. Добавьте продукт **DebugViewSwift** в таргет приложения.

### Через Package.swift

```swift
dependencies: [
    .package(url: "https://github.com/N1kita-ni/DebugViewSwift-Binary.git", from: "1.2.2")
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
- **Скрыть:** перетащите кнопку на ✕ внизу экрана или выключите **Interface → Floating button**.
- **Вернуть:** шейк → **Interface → Floating button**, или `DebugViewSwift.showFloatingButton()`.
- Выбор запоминается между запусками.
- **Во время записи экрана** кнопка прячется автоматически и возвращается после неё.

## Настройка

```swift
var configuration = DebugViewSwift.Configuration()
configuration.showsFloatingButton = true
configuration.enablesShakeGesture = true
configuration.enablesNetworkLogging = true
configuration.enablesConsoleCapture = true
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

### Свои действия — App → Custom actions

```swift
DebugViewSwift.addAction(title: "Reset onboarding") {
    UserDefaults.standard.removeObject(forKey: "onboardingShown")
}

DebugViewSwift.addAction(title: "Logout") {
    SessionManager.shared.logout()
}
```

### Свои поля — App → App & Device → Custom

```swift
DebugViewSwift.setInfo("Staging", forKey: "Environment")
DebugViewSwift.setInfo(user.id, forKey: "User ID")
DebugViewSwift.setInfo(nil, forKey: "User ID")       // удалить поле
```

Повторный вызов с тем же ключом заменяет значение; поля идут в порядке первого добавления.

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

**Крашей нет в App → Crash reports**
- Отчёт появляется после **следующего** запуска приложения.
- Если приложение запущено из Xcode, краш останавливается в отладчике и отчёт может не записаться — проверяйте, запуская приложение с иконки.
- Не включайте вместе с другим крэш-репортером (`enablesCrashReporting = false`).
