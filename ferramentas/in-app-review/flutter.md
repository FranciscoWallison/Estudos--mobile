# In-App Review — Flutter

> **Categoria:** In-App Review
> **Plataforma:** Android (5.0+) + iOS (10.3+)
> **Quando usar:** popup de avaliação nativo dentro de um app Flutter, usando uma única API Dart.
> **Alternativas:** chamar via `MethodChannel` próprio; abrir Play/App Store via `url_launcher`.

## O que faz

O package **`in_app_review`** é uma bridge sobre Play Core In-App Review (Android) e StoreKit (iOS). Mesmo comportamento das nativas: **sistema decide**, sem callback de "avaliou ou não".

## Pré-requisitos

- Flutter SDK estável (≥ 2.x serve).
- Android `minSdkVersion >= 21` e Play Store no device.
- iOS 10.3+.

## Instalação

`pubspec.yaml`:

```yaml
dependencies:
  in_app_review: ^2.0.10
```

Depois:

```bash
flutter pub get
cd ios && pod install
```

## Como usar

```dart
import 'package:in_app_review/in_app_review.dart';

final InAppReview _inAppReview = InAppReview.instance;

Future<void> pedirAvaliacao() async {
  if (await _inAppReview.isAvailable()) {
    await _inAppReview.requestReview();
    // Não há retorno indicando se o usuário avaliou.
    salvarTimestampUltimoPrompt();
  }
}
```

### Abrir página da Store (fallback)

Quando você quer **forçar** a página completa (não o popup):

```dart
await _inAppReview.openStoreListing(
  appStoreId: '123456789', // obrigatório no iOS
);
```

### Boa prática — gatilho com heurística simples

```dart
import 'package:in_app_review/in_app_review.dart';
import 'package:shared_preferences/shared_preferences.dart';

const _kUltimaKey = 'review.ultimoPrompt';
const _kDiasMin = 90;

Future<void> tentarPedirAvaliacao() async {
  final inApp = InAppReview.instance;
  if (!await inApp.isAvailable()) return;

  final prefs = await SharedPreferences.getInstance();
  final ultima = prefs.getString(_kUltimaKey);
  if (ultima != null) {
    final dias = DateTime.now().difference(DateTime.parse(ultima)).inDays;
    if (dias < _kDiasMin) return;
  }

  await inApp.requestReview();
  await prefs.setString(_kUltimaKey, DateTime.now().toIso8601String());
}
```

## Como testar

- **Android**: em debug build instalado por `flutter run` **NÃO aparece**. Use Internal App Sharing — gere AAB com `flutter build appbundle` e suba no Play Console.
- **iOS**: build via Xcode/`flutter run` mostra sempre, sem rate limit.

Pra confirmar a chamada no Android:

```bash
adb logcat | findstr "PlayCore"
```

## Dicas e pegadinhas

- `isAvailable()` retorna **`false`** em devices sem Play Store, iOS < 10.3, Android < 5.0.
- O `await requestReview()` resolve quando o flow termina, mas **não diz se o usuário avaliou**.
- Em **iOS**, se você quiser usar `openStoreListing` é **obrigatório** passar o `appStoreId`. Sem ele, no iOS o método joga exceção.
- Plataformas além de Android/iOS (web, desktop) **não suportam** — `isAvailable()` retorna false e o `requestReview()` é no-op.
- App em **Flutter dev/profile mode** se comporta como release pra esse caso, mas o **install flow** é o que define se aparece (debug build instalado por ADB → não aparece).

## Ver também

- [Android nativo](android-nativo.md) — o que o package chama por baixo no Android.
- [iOS nativo](ios-nativo.md) — equivalente no iOS.
- [react-native](react-native.md) / [cordova-ionic](cordova-ionic.md) — outras stacks cross-platform.
- Package: <https://pub.dev/packages/in_app_review>
