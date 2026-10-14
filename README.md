# DebugViewSwift

In-app инструменты отладки для iOS-приложений на UIKit: перехват сети, производительность, инспектор лейаута и 3D-иерархия вью, хранилища, консоль, краш-репорты.

Это бинарная сборка (XCFramework). Версия: **1.0.0**.

## Требования

- iOS 14+
- Xcode 26.4+ (бинарник собран этой версией; более старый Xcode не сможет его импортировать)

## Установка

**Xcode:** File → Add Package Dependencies… → URL этого репозитория → добавьте продукт `DebugViewSwift` в таргет приложения.

**Package.swift:**

```swift
dependencies: [
    .package(url: "https://github.com/N1kita-ni/DebugViewSwift-Binary.git", from: "1.0.0")
]
```

## Подключение

```swift
import DebugViewSwift

#if DEBUG
DebugViewSwift.setup()
#endif
```

Настройки и полное API — `DebugViewSwift.Configuration`, `presentPanel()`, `dismissPanel()`, `togglePanel()`, `showFloatingButton()`, `hideFloatingButton()`, `addAction(title:handler:)`, `setInfo(_:forKey:)`. Документация доступна в Xcode через Quick Help.
