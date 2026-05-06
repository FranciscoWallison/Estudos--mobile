# In-App Review — React Native

> **Categoria:** In-App Review
> **Plataforma:** Android (5.0+) + iOS (10.3+)
> **Quando usar:** popup de avaliação nativo dentro de um app React Native, usando uma única API JS.
> **Alternativas:** chamar nativo direto via NativeModule próprio; abrir Play/App Store via `Linking.openURL`.

## O que faz

A lib **`react-native-in-app-review`** é uma bridge fina sobre as APIs nativas — Play Core In-App Review (Android) e StoreKit (iOS). Mesmo comportamento das nativas: **sistema decide**, sem callback de "avaliou ou não".

## Pré-requisitos

- React Native 0.60+ (autolinking).
- Android `minSdkVersion >= 21` e Play Store no device.
- iOS 10.3+ no Podfile (`platform :ios, '10.3'` ou superior).

## Instalação

```bash
yarn add react-native-in-app-review
# ou: npm install react-native-in-app-review

# iOS
cd ios && pod install
```

Em **React Native 0.60+** o autolinking cuida do registro nativo. Em versões mais antigas, link manual via `react-native link`.

## Como usar

```ts
import InAppReview from 'react-native-in-app-review';

async function pedirAvaliacao() {
  if (!InAppReview.isAvailable()) {
    // Android < 5.0, iOS < 10.3, ou device sem Play Store
    return;
  }

  try {
    const flowFinishedSuccessfully = await InAppReview.RequestInAppReview();
    // flowFinishedSuccessfully === true só significa que o flow terminou.
    // NÃO significa que o usuário avaliou.
    salvarTimestampUltimoPrompt();
  } catch (e) {
    // falha silenciosa — não mostre erro pro usuário
    console.warn('InAppReview falhou', e);
  }
}
```

### Boa prática — gatilho com heurística simples

```ts
import AsyncStorage from '@react-native-async-storage/async-storage';
import InAppReview from 'react-native-in-app-review';

const ULTIMA_KEY = '@review/ultimoPrompt';
const DIAS_MIN = 90;

async function tentarPedirAvaliacao() {
  if (!InAppReview.isAvailable()) return;

  const ultimaIso = await AsyncStorage.getItem(ULTIMA_KEY);
  if (ultimaIso) {
    const dias = (Date.now() - new Date(ultimaIso).getTime()) / 86400000;
    if (dias < DIAS_MIN) return;
  }

  try {
    await InAppReview.RequestInAppReview();
    await AsyncStorage.setItem(ULTIMA_KEY, new Date().toISOString());
  } catch {}
}
```

## Como testar

- **Android**: em debug build instalado por Metro/USB **NÃO aparece**. Use Internal App Sharing (ver [android-nativo](android-nativo.md#como-testar-sem-o-popup-nunca-aparecer)).
- **iOS**: em build via Xcode aparece toda vez (sem rate limit). Em TestFlight, rate-limited igual prod.

Pra confirmar que a chamada saiu no Android:

```bash
adb logcat | findstr "PlayCore"
```

## Dicas e pegadinhas

- `InAppReview.isAvailable()` retorna **`false`** em emuladores AOSP (sem Play Store), iOS antigo, e Android < 5.0 — sempre cheque antes.
- O retorno `true` da Promise **não significa que o usuário avaliou**. É só "o flow terminou sem erro".
- Em **apps Expo** (managed), use `expo-store-review` em vez dessa lib.
- Apps com **vários jeitos de entrar** (deep link, push) devem garantir que o popup só aparece em cenas onde a Activity/Scene está em primeiro plano — chamar com app em background = no-op silencioso.

## Alternativas equivalentes

- **`expo-store-review`** — pra apps Expo managed. API: `StoreReview.requestReview()`.
- **`react-native-store-review`** — antiga, ainda mantida; só iOS no escopo original.

## Ver também

- [Android nativo](android-nativo.md) — o que essa lib chama por baixo no Android.
- [iOS nativo](ios-nativo.md) — equivalente no iOS.
- [flutter](flutter.md) / [cordova-ionic](cordova-ionic.md) — outras stacks cross-platform.
- Lib: <https://github.com/MinaSamir11/react-native-in-app-review>
- Expo: <https://docs.expo.dev/versions/latest/sdk/storereview/>
