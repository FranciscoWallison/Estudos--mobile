# App Intents — iOS nativo (Swift)

> **Categoria:** App Functions
> **Plataforma:** iOS 16+ (App Intents) / iOS 18+ (integração com Apple Intelligence)
> **Quando usar:** expor ações do seu app pro Siri, Atalhos, Spotlight e Apple Intelligence chamarem com argumentos tipados.
> **Alternativas:** SiriKit (geração antiga, intents enumerados pela Apple). App Intents é a substituição moderna, livre.

## O que faz

Você implementa o protocolo `AppIntent` em Swift. Cada intent é uma "função do app" com parâmetros tipados, título, descrição. O sistema indexa, e:

- **Siri** chama por voz.
- **App Atalhos** monta automações.
- **Spotlight** sugere com base no contexto.
- **Apple Intelligence** (iOS 18+) descobre e invoca quando faz sentido pra pergunta do usuário.

## Pré-requisitos

- Xcode 14+ (App Intents). Pra Apple Intelligence integration, Xcode 16+.
- iOS 16+ no `Deployment Target` (ou 18+ pra usar Intelligence features).

## Como usar

### 1. Defina um Intent simples

```swift
import AppIntents

struct CriarNotaIntent: AppIntent {
    static let title: LocalizedStringResource = "Criar nota"
    static let description: IntentDescription = "Cria uma nova nota no MeuApp."

    @Parameter(title: "Título")
    var titulo: String

    @Parameter(title: "Conteúdo")
    var conteudo: String

    func perform() async throws -> some IntentResult & ProvidesDialog {
        let id = try await Repositorio.shared.salvarNota(titulo: titulo, conteudo: conteudo)
        return .result(dialog: "Nota '\(titulo)' criada (id \(id)).")
    }
}
```

### 2. Exponha um AppShortcut (atalho global)

```swift
struct MeuAppShortcuts: AppShortcutsProvider {
    static var appShortcuts: [AppShortcut] {
        AppShortcut(
            intent: CriarNotaIntent(),
            phrases: [
                "Criar nota no \(.applicationName)",
                "Nova anotação no \(.applicationName)"
            ],
            shortTitle: "Criar nota",
            systemImageName: "note.text.badge.plus"
        )
    }
}
```

`\(.applicationName)` é obrigatório pelo menos uma vez em cada phrase.

### 3. Intent que retorna uma entidade

```swift
struct Nota: AppEntity {
    var id: UUID
    var titulo: String
    var conteudo: String

    static var typeDisplayRepresentation: TypeDisplayRepresentation = "Nota"
    static var defaultQuery = NotaQuery()

    var displayRepresentation: DisplayRepresentation {
        DisplayRepresentation(title: "\(titulo)", subtitle: "\(conteudo)")
    }
}

struct NotaQuery: EntityQuery {
    func entities(for identifiers: [Nota.ID]) async throws -> [Nota] {
        try await Repositorio.shared.notas(ids: identifiers)
    }
}

struct BuscarNotaIntent: AppIntent {
    static let title: LocalizedStringResource = "Buscar nota"

    @Parameter(title: "Termo")
    var termo: String

    func perform() async throws -> some IntentResult & ReturnsValue<[Nota]> {
        let resultados = try await Repositorio.shared.buscar(termo)
        return .result(value: resultados)
    }
}
```

### 4. (iOS 18+) Integração com Apple Intelligence

Adicione `OpenIntent`, `OpensAppWhenRun` e use `IntentParameterSummary` rico pra que a Intelligence consiga compor:

```swift
struct AbrirNotaIntent: AppIntent, OpenIntent {
    static let title: LocalizedStringResource = "Abrir nota"
    static let openAppWhenRun: Bool = true

    @Parameter(title: "Nota") var target: Nota

    func perform() async throws -> some IntentResult {
        NavigationCoordinator.shared.abrir(nota: target)
        return .result()
    }
}
```

## Como testar

- **Apple Atalhos (Shortcuts app):** seus intents aparecem automaticamente na busca.
- **Siri:** "Ei Siri, criar nota no MeuApp título 'Lembrete' conteúdo 'Pagar conta'".
- **Spotlight:** depois de algumas execuções, aparece como sugestão contextual.
- **Apple Intelligence (iOS 18+):** ative no Settings e teste via Siri / pedido livre.
- **Em Xcode:** Run → executa o intent direto sem assistente, útil pra ciclo rápido.

## Dicas e pegadinhas

- **`perform()` é `async throws`** — faça I/O sem bloquear thread principal.
- **Phrases do `AppShortcut` precisam de `\(.applicationName)`** pelo menos uma vez. Sem isso o build falha.
- **Localize as strings** (`LocalizedStringResource`) — a IA usa o título e a descrição como features. Em outros idiomas, intents sem tradução perdem precisão.
- **`AppEntity` deve ter `id` estável** — Apple Intelligence guarda referências; trocar id quebra atalhos do usuário.
- Pra **abrir o app**, use `OpensAppWhenRun = true`. Senão, intent roda em **background** (extension-like) e seu app **não é trazido pra frente**.
- **Não exponha** intent que requer auth pesado sem fluxo claro — `RequestConfirmationIntent` ajuda a pedir confirmação do usuário antes de executar.
- Pra Apple Intelligence reconhecer **parâmetros em linguagem natural**, declare `IntentParameterSummary` com placeholders ricos.

## Estrutura mínima recomendada

```
MeuApp/
├── Intents/
│   ├── CriarNotaIntent.swift
│   ├── BuscarNotaIntent.swift
│   └── MeuAppShortcuts.swift   # AppShortcutsProvider
├── Entities/
│   └── Nota.swift              # AppEntity + Query
└── ...
```

## Ver também

- [Android nativo](android-nativo.md) — equivalente Google (AppFunctions).
- [react-native](react-native.md) / [flutter](flutter.md) — wrappers cross-platform.
- Doc oficial App Intents: <https://developer.apple.com/documentation/appintents>
- Apple Intelligence + App Intents: <https://developer.apple.com/documentation/appintents/making-your-app-s-actions-available-to-siri>
- WWDC Sessions sobre App Intents: <https://developer.apple.com/videos/play/wwdc2024/10134/>
