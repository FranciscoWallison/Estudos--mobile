# In-App Review — Cordova / Ionic / Capacitor

> **Categoria:** In-App Review
> **Plataforma:** Android (5.0+) + iOS (10.3+)
> **Quando usar:** popup de avaliação nativo em apps híbridos (Cordova clássico, Ionic com Cordova, Ionic com Capacitor).
> **Alternativas:** abrir Play/App Store via `InAppBrowser` ou `window.open` com URL da loja.

## Cenário 1 — Capacitor (Ionic moderno, recomendado)

`@capacitor-community/in-app-review` é o padrão atual.

### Instalação

```bash
npm install @capacitor-community/in-app-review
npx cap sync
```

### Uso (TypeScript / Angular / React / Vue)

```ts
import { InAppReview } from '@capacitor-community/in-app-review';

async function pedirAvaliacao() {
  try {
    await InAppReview.requestReview();
    salvarTimestampUltimoPrompt();
  } catch (err) {
    console.warn('InAppReview falhou', err);
  }
}
```

### Em Angular (Ionic + Capacitor)

```ts
import { Component } from '@angular/core';
import { InAppReview } from '@capacitor-community/in-app-review';

@Component({ selector: 'app-home', template: '<ion-button (click)="rate()">Avaliar</ion-button>' })
export class HomePage {
  async rate() {
    await InAppReview.requestReview();
  }
}
```

## Cenário 2 — Cordova clássico

Plugin: **`cordova-plugin-app-rating`** (popup nativo) ou **`cordova-launch-review`** (que originalmente abria a Store, mas a partir da v3 também faz in-app review).

### Instalação

```bash
cordova plugin add cordova-launch-review
# ou
ionic cordova plugin add cordova-launch-review
npm install @awesome-cordova-plugins/launch-review @awesome-cordova-plugins/core
```

### Uso (Ionic / Angular com Awesome Cordova Plugins)

```ts
import { LaunchReview } from '@awesome-cordova-plugins/launch-review/ngx';

constructor(private launchReview: LaunchReview) {}

async pedirAvaliacao() {
  if (this.launchReview.isRatingSupported()) {
    try {
      await this.launchReview.rating();
      // popup nativo (Android Play Core / iOS StoreKit)
    } catch (e) {
      console.warn(e);
    }
  } else {
    // fallback: abrir Store
    await this.launchReview.launch('seu-app-id-ios', 'com.exemplo.app');
  }
}
```

### Uso direto (Cordova vanilla, sem Ionic Native)

```js
document.addEventListener('deviceready', () => {
  if (cordova.plugins.LaunchReview && cordova.plugins.LaunchReview.isRatingSupported()) {
    cordova.plugins.LaunchReview.rating(
      () => console.log('flow ok'),
      (err) => console.warn(err),
    );
  }
});
```

## Como testar

Mesmas regras das outras stacks:

- **Android**: debug build via ADB **NÃO mostra**. Use Internal App Sharing.
- **iOS**: build via Xcode mostra toda vez. TestFlight rate-limited.

Confirmar chamada no Android:

```bash
adb logcat | findstr "PlayCore"
```

## Dicas e pegadinhas

- **Capacitor é o caminho atual** pra Ionic. Cordova clássico está em manutenção, e várias libs estão sem release há anos.
- `isRatingSupported()` (cordova-launch-review) checa Android 5+/iOS 10.3+ e Play Store presente — sempre cheque antes.
- Em **Capacitor**, depois de instalar plugin **sempre** rode `npx cap sync` pra propagar pro Android/iOS native.
- App híbrido roda dentro de WebView, mas o popup é **nativo** — quem mostra é o sistema, não a WebView.
- Em Ionic com Cordova, se você abrir o popup e a WebView estiver em segundo plano (ex: app pausado), nada aparece.

## Plugins atuais (referência rápida)

| Stack             | Plugin                                    | Status                           |
| ----------------- | ----------------------------------------- | -------------------------------- |
| Capacitor         | `@capacitor-community/in-app-review`      | Ativo, recomendado.              |
| Cordova / Ionic-Cordova | `cordova-launch-review`             | Ativo; cobre rating in-app + abrir store. |
| Cordova clássico  | `cordova-plugin-app-rating`               | Funciona, manutenção fria.       |

## Ver também

- [Android nativo](android-nativo.md) — o que esses plugins chamam por baixo.
- [iOS nativo](ios-nativo.md) — idem.
- [react-native](react-native.md) / [flutter](flutter.md) — outras stacks.
- Plugin Capacitor: <https://github.com/capacitor-community/in-app-review>
- Plugin Cordova: <https://github.com/dpa99c/cordova-launch-review>
