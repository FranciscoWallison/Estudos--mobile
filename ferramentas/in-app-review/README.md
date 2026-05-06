# In-App Review

Categoria sobre **popup de avaliação dentro do app** — aquele card pequeno que aparece com "Curtindo o app? Avalia aí" e o usuário deixa estrelas/comentário **sem sair pro Play Store / App Store**.

Tanto Google quanto Apple têm API oficial. Em ambos, **o sistema decide** se mostra ou não — seu app só **pede**. Você não controla quando o popup vai aparecer de fato, e isso é proposital pra evitar abuso.

## Quando usar

- Após momentos positivos: terminou uma tarefa, ganhou conquista, fechou compra com sucesso.
- **Não** logo na primeira sessão.
- **Não** em telas com fricção (erro, login, payment falhando).
- **Não** condicionado a recompensa ("avalia pra ganhar X") — viola política das duas lojas.

## Quotas e comportamento (essencial pra não quebrar a cabeça)

| Plataforma | Limite                                       | Em build de Dev                                        |
| ---------- | -------------------------------------------- | ------------------------------------------------------ |
| iOS        | Até **3 prompts/ano por usuário** (Apple controla) | Sempre exibe (sem rate limit) — não confunda com prod |
| Android    | Quota interna não documentada (Google controla) | **Nunca** exibe em debug build instalado por ADB. Use Internal App Sharing pra validar |

> **Implicação prática:** em prod, mesmo chamando `requestReview()` toda vez, o usuário **não vai** ver toda vez. Não tente controlar isso no client — confie na API.

## Arquivos por plataforma

| Stack | Link |
| ----- | ---- |
| Android nativo (Java/Kotlin) | [android-nativo.md](android-nativo.md) |
| iOS nativo (Swift / SwiftUI / UIKit) | [ios-nativo.md](ios-nativo.md) |
| React Native | [react-native.md](react-native.md) |
| Flutter | [flutter.md](flutter.md) |
| Cordova / Ionic / Capacitor | [cordova-ionic.md](cordova-ionic.md) |

## Lógica comum (independente de stack)

```text
1. Em algum gatilho positivo (ex: usuário completou ação N vezes):
   - Verificar se elegível (sua heurística: usuário ativo há ≥ X dias, sem
     prompts recentes salvos por você, etc).
2. Pedir o ReviewManager / requestReview().
3. Receber o ReviewInfo (Android) ou só "feito" (iOS).
4. Lançar o flow (Android) ou já é exibido (iOS).
5. Salvar timestamp local pra não chamar com frequência absurda.
6. Não saber se o usuário avaliou ou não — a API NÃO te conta.
```

## Pegadinhas comuns a todas as plataformas

- **A API não retorna se o usuário avaliou.** Não trate o callback como sucesso de avaliação — ele só diz "fluxo terminou".
- **Não tem como forçar pra testar visualmente em prod** sem instalar pela loja.
- Em **TestFlight (iOS)**, o popup tem comportamento parecido com prod, mas **não envia review real** pra App Store.
- O **idioma do popup** segue o do device, não o do app.
- Em **tablets / iPads**, lembra de passar a `windowScene` correta no iOS — em multi-cena pode dar crash silencioso.

## Documentação oficial

- Apple: <https://developer.apple.com/documentation/storekit/appstore/requestreview(in:)-1q8qs>
- Google: <https://developer.android.com/guide/playcore/in-app-review?hl=pt-br>

## Ver também

- [adb](../dispositivo-adb/adb.md) — pra instalar via Internal App Sharing e testar Android.
