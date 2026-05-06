# In-App Review — iOS nativo (StoreKit)

> **Categoria:** In-App Review
> **Plataforma:** iOS 10.3+ (recursos novos a partir do iOS 14 / 16 / 18)
> **Quando usar:** popup de avaliação Apple sem sair do app, em projeto iOS nativo Swift / SwiftUI / UIKit.
> **Alternativas:** abrir App Store via deep link (`itms-apps://...?action=write-review`) — leva o usuário pra fora do app.

## O que faz

`SKStoreReviewController` (UIKit) e `requestReview()` / `RequestReviewAction` (SwiftUI) mostram um card sobreposto. Quem mostra é o sistema; você só pede.

## Qual API usar (depende da versão)

| Caso                                                  | API recomendada                                                                  |
| ----------------------------------------------------- | -------------------------------------------------------------------------------- |
| App SwiftUI, iOS 16+                                  | `@Environment(\.requestReview)` (`RequestReviewAction`)                          |
| App UIKit, iOS 14+                                    | `SKStoreReviewController.requestReview(in: windowScene)` ([doc](https://developer.apple.com/documentation/storekit/appstore/requestreview(in:)-1q8qs)) |
| App UIKit, iOS 10.3-13                                | `SKStoreReviewController.requestReview()` (deprecated em iOS 14)                 |
| Quer abrir página completa de avaliação na App Store  | `SKOverlay` ou `itms-apps://` com `?action=write-review` (não é in-app)          |

## Como usar

### SwiftUI (iOS 16+)

```swift
import StoreKit
import SwiftUI

struct CompraConcluidaView: View {
    @Environment(\.requestReview) private var requestReview
    @State private var compras = 0

    var body: some View {
        Button("Comprar") {
            compras += 1
            if compras == 3 {
                requestReview()  // Apple decide se mostra
            }
        }
    }
}
```

### UIKit / iOS 14+

```swift
import StoreKit
import UIKit

func pedirAvaliacao(em viewController: UIViewController) {
    guard let scene = viewController.view.window?.windowScene else { return }
    SKStoreReviewController.requestReview(in: scene)
}
```

### UIKit / iOS 10.3-13 (legado)

```swift
import StoreKit

if #available(iOS 10.3, *) {
    SKStoreReviewController.requestReview()
}
```

### Híbrido (suportar várias versões)

```swift
import StoreKit
import UIKit

func pedirAvaliacao(em viewController: UIViewController) {
    if #available(iOS 14.0, *) {
        guard let scene = viewController.view.window?.windowScene else { return }
        SKStoreReviewController.requestReview(in: scene)
    } else if #available(iOS 10.3, *) {
        SKStoreReviewController.requestReview()
    } else {
        // fallback: abrir App Store
        if let url = URL(string: "itms-apps://itunes.apple.com/app/idSEU_APP_ID?action=write-review") {
            UIApplication.shared.open(url)
        }
    }
}
```

## Como testar

- **Build de Dev (Xcode)**: o popup **sempre aparece**, sem rate limit. Não confunda com prod.
- **TestFlight**: comportamento parecido com prod (rate-limited), mas a review **não chega na App Store**.
- **App Store**: até **3 prompts/ano por usuário** controlados pela Apple.

## Dicas e pegadinhas

- A partir do **iOS 14**, omitir o `windowScene` (chamando `requestReview()` sem `in:`) é deprecated. Use a versão com cena.
- Em apps **multi-cena** (iPad com Stage Manager, split view), passe a `windowScene` da cena correta — senão o popup pode aparecer na cena errada ou não aparecer.
- A API **não tem callback**: você não sabe se o usuário avaliou, nem se o popup foi exibido.
- **Não condicione recompensa** ("avalie pra ganhar X") — viola a [App Store Review Guideline 1.1.7](https://developer.apple.com/app-store/review/guidelines/).
- Não chame em **onboarding inicial** ou **logo após erro** — é o caminho rápido pra rejeição na review.
- Configurações do iOS tem **Configurações → App Store → Avaliações no App** que o usuário pode desligar globalmente; nesse caso a chamada vira no-op silenciosa.

## Ver também

- [Android nativo](android-nativo.md) — equivalente Google.
- [react-native](react-native.md) / [flutter](flutter.md) — wrappers cross-platform.
- Doc oficial: <https://developer.apple.com/documentation/storekit/appstore/requestreview(in:)-1q8qs>
- SwiftUI `RequestReviewAction`: <https://developer.apple.com/documentation/storekit/requestreviewaction>
