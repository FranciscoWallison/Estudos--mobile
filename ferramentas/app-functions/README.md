# App Functions (expor o app pra IA / agents)

Categoria sobre **deixar seu app ser chamado por uma IA** — assistentes (Gemini, Google Assistant, Siri, Apple Intelligence) e agents conseguem descobrir funções que seu app expõe e invocar com argumentos tipados.

Exemplo prático: o usuário fala "manda 30 reais pro João" pro Gemini; o Gemini sabe que seu app de pagamentos tem uma função `enviarPix(destinatario, valor)` e chama com `destinatario="João"`, `valor=30`. Sua função executa e retorna resultado. Tudo sem abrir a UI do seu app.

## Os dois mundos

| Plataforma | Nome oficial | Status (2026) |
| ---------- | ------------ | ------------- |
| Android    | **AppFunctions** (Jetpack `androidx.appfunctions:*`) | Em rollout, parte do ecossistema Gemini / AICore |
| iOS / iPadOS / macOS | **App Intents** (`AppIntent` protocol, StoreKit-free) | Estável desde iOS 16, conectado ao Apple Intelligence desde iOS 18 |

> **Não é a mesma API.** Você implementa uma vez por plataforma. O conceito é o mesmo: declarar funções tipadas + parâmetros + retorno, e o sistema operacional cuida da descoberta e da chamada pela IA.

## O que muda no seu app

Você passa de "app que o usuário abre e clica" pra "app que o usuário **fala**" — e a IA decide quando abrir UI, quando só rodar em background, e como compor com funções de outros apps.

Implicações de arquitetura:

- **Funções precisam ser puras / sem efeito colateral oculto** — a IA pode chamar com `dry-run` ou em loop.
- **Tipos importam** — `String`, `Int`, `enum`, data class. Não dá pra passar `Bundle` solto.
- **Permissões e auth** ficam no fluxo da IA: a função pode pedir autorização do usuário antes de executar.
- **Funções podem retornar `View`/`AppEntity`** que a IA renderiza inline na resposta dela — não só `String`.

## Arquivos por stack

| Stack | Link |
| ----- | ---- |
| Android nativo (Kotlin + KSP) | [android-nativo.md](android-nativo.md) |
| iOS nativo (Swift) | [ios-nativo.md](ios-nativo.md) |
| React Native | [react-native.md](react-native.md) |
| Flutter | [flutter.md](flutter.md) |
| Cordova / Ionic / Capacitor | [cordova-ionic.md](cordova-ionic.md) |

## Lógica comum (independente de stack)

```text
1. Declarar a função no app, com:
   - nome estável (vira identificador pra IA),
   - parâmetros tipados (com descrição em linguagem natural),
   - tipo de retorno (string simples, entity, ou view).
2. Sistema operacional indexa as funções no install/upgrade.
3. IA recebe pergunta do usuário em linguagem natural.
4. IA escolhe função e gera argumentos tipados a partir do texto.
5. Sistema chama sua função no contexto do seu app (background, normalmente).
6. Sua função executa lógica de negócio (pode chamar API, banco, etc.).
7. Retorno volta pra IA, que apresenta pro usuário.
```

## Pegadinhas comuns a todas as plataformas

- **Descrições importam.** Texto do parâmetro `"Destinatário do PIX"` é usado pela IA pra inferir o argumento — descrição ruim = IA mapeia errado.
- **IDs estáveis.** Renomear função quebra atalhos salvos pelo usuário, conexões com automações.
- **Background-friendly.** Se sua função precisa de UI obrigatória (input do usuário, captcha), declare isso — a IA decide se vale abrir o app ou pedir desambiguação.
- **Testar sem agent é difícil.** Em ambos os mundos a melhor validação é via assistente real (Gemini / Siri); em dev você tem inspetores próprios (ver arquivos da stack).
- **Privacidade.** Apenas exponha funções que você quer que assistentes (incluindo de terceiros, no Android) consigam disparar.

## Quando vale investir

- Apps com **ações claras e parametrizáveis** (pagamentos, mensagem, criar nota, agendar, controlar dispositivo).
- Apps que querem aparecer em **resultados do assistente** sem o usuário abrir o app.
- Apps de **produtividade** (notas, tasks, calendário) ganham muito.
- Apps **só de conteúdo passivo** (catálogo, leitor) ganham menos.

## Documentação oficial

- Android AppFunctions: <https://developer.android.com/reference/androidx/appfunctions/package-summary>
- Apple App Intents: <https://developer.apple.com/documentation/appintents>
- Apple Intelligence + App Intents: <https://developer.apple.com/documentation/appintents/making-your-app-s-actions-available-to-siri>

## Ver também

- [adb](../dispositivo-adb/adb.md) — pra inspecionar funções registradas (Android: `adb shell dumpsys appfunctions`).
- [In-App Review](../in-app-review/README.md) — outra categoria de "feature de plataforma" exposta por SDK do SO.
