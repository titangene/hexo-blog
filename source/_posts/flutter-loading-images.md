---
title: Flutter 載入圖片
date: 2018-12-31 21:10:00
author: Titangene
tags:
  - Flutter
  - Dart
  - Native App
categories:
  - Native App
cover_image: /images/cover/flutter.jpg
---

![](../images/cover/flutter.jpg)

要如何在 Flutter 內載入圖片？這篇做個小記錄。

<!-- more -->

## 指定資源
Flutter 是在根目錄內的 `pubspec.yaml` 檔案來設定應用程式所需的資源，設定的資源沒有順序關係。

若要指定某些資源，資源的路徑是相對於 `pubspec.yaml` 檔案的相對路徑：

```yaml
flutter:
  assets:
    - assets/images/avatar.jpg
    - assets/images/background.jpg
```

也可以指定某個目錄，代表可以存取到這個目錄下的所有資源，但記得要在目錄的最後加上 `/` 這個符號：

```yaml
flutter:
  assets:
    - assets/
```

若要存取子目錄內的資源，請記得要另外為子目錄設定，例如：

```yaml
flutter:
  assets:
    - assets/
    - assets/images/
```

## 載入圖片
可使用 `Image.asset()` 來載入圖片，裡面的參數就是圖片的路徑：

```dart
Widget build(BuildContext context) {
  // ...
  return Image.asset('assets/images/background.jpg');
  // ...
}
```

或是在 `Image()` 內的 `image` 參數使用 `AssetImage()`：

```dart
Widget build(BuildContext context) {
  // ...
  return Image(
    image: AssetImage('assets/images/background.jpg'),
  );
  // ...
}
```

最後顯示的結果會一樣：

![](../images/flutter-loading-images/flutter_loading-images.png)

## 參考連結
- [Adding assets and images - Flutter](https://flutter.io/docs/development/ui/assets-and-images)