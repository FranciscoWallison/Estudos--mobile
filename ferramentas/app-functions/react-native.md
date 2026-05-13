# App Functions — React Native

> **Categoria:** App Functions
> **Plataforma:** Android (AppFunctions) + iOS (App Intents)
> **Quando usar:** expor funções de um app React Native pro Gemini / Siri / Apple Intelligence.
> **Alternativas:** continuar só com Deep Links + Shortcuts antigos; ainda não há lib pronta cobrindo tudo.

## Realidade atual (2026)

Não existe **um único package** que cubra AppFunctions (Android) + App Intents (iOS) de forma simétrica em React Native. Isso porque:

- **AppFunctions usa KSP** — só Kotlin. Não dá pra anotar JS.
- **App Intents usa `AppIntent` Swift** com macros — só Swift.

Resultado: você implementa **na ponta nativa**, e cria uma bridge pro JS que delega a lógica de negócio (ou chama JS de volta via headless task).

Algumas libs comunitárias começaram a aparecer (verificar `react-native-app-intents`, `react-native-app-functions`), mas estão em estágio inicial. Pra cenário sério hoje, vale fazer module próprio.

## Arquitetura recomendada

```
React Native
└── App.tsx (JS) ─────────────┐
                              │ (não chamado direto pela IA)
Android (Kotlin)              │
└── @AppFunction criarNota    │ ─ chama serviço/repositório nativo ─ devolve resultado pra IA
                              │
iOS (Swift)                   │
└── AppIntent CriarNotaIntent │ ─ chama serviço/repositório nativo ─ devolve resultado pra IA
```

A lógica de negócio fica em camadas que **tanto o app RN quanto a função do agent** acessam. Pode ser:

- API HTTP / GraphQL própria — função do agent chama o backend igual o app chama.
- Banco local compartilhado (Room/CoreData) — função do agent acessa direto.
- **Headless JS Task** (Android) ou **App Extension** (iOS) — função do agent dispara um task JS quando precisa de lógica que vive no bundle RN.

## Passo a passo (Android)

### 1. Crie um módulo Kotlin no projeto Android RN

`android/app/src/main/java/com/seuapp/appfunctions/MinhasFuncoes.kt`:

```kotlin
package com.seuapp.appfunctions

import androidx.appfunctions.AppFunction
import androidx.appfunctions.AppFunctionContext
import androidx.appfunctions.AppFunctionSerializable

@AppFunctionSerializable
data class NovaNota(val titulo: String, val conteudo: String)

class MinhasFuncoes {

    @AppFunction
    suspend fun criarNota(
        context: AppFunctionContext,
        nota: NovaNota
    ): String {
        // Caminho A — chama backend direto (recomendado)
        val id = ApiClient.criarNota(nota.titulo, nota.conteudo)
        return "Nota $id criada"

        // Caminho B — dispara headless JS task pra reusar lógica RN:
        // HeadlessJsTaskService.start(context, "criarNotaTask", nota)
    }
}
```

### 2. Configure KSP + dependências

Detalhes em [android-nativo.md](android-nativo.md#instalação).

### 3. (Opcional) Bridge pro JS chamar a mesma função em flow normal

```kotlin
@ReactModule(name = "Notas")
class NotasModule(context: ReactApplicationContext) : ReactContextBaseJavaModule(context) {
    override fun getName() = "Notas"

    @ReactMethod
    fun criarNota(titulo: String, conteudo: String, promise: Promise) {
        // mesma lógica que o AppFunction chama
        runBlocking { promise.resolve(ApiClient.criarNota(titulo, conteudo)) }
    }
}
```

## Passo a passo (iOS)

### 1. Crie o Intent em Swift

`ios/AppIntents/CriarNotaIntent.swift`:

```swift
import AppIntents

struct CriarNotaIntent: AppIntent {
    static let title: LocalizedStringResource = "Criar nota"

    @Parameter(title: "Título") var titulo: String
    @Parameter(title: "Conteúdo") var conteudo: String

    func perform() async throws -> some IntentResult & ProvidesDialog {
        // Caminho A — chama backend
        let id = try await ApiClient.shared.criarNota(titulo: titulo, conteudo: conteudo)
        return .result(dialog: "Nota \(id) criada")

        // Caminho B — chama React Native Background Task
        // RNBackgroundTask.dispatch("criarNotaTask", [...])
    }
}
```

### 2. AppShortcutsProvider

```swift
struct MeuAppShortcuts: AppShortcutsProvider {
    static var appShortcuts: [AppShortcut] {
        AppShortcut(
            intent: CriarNotaIntent(),
            phrases: ["Criar nota no \(.applicationName)"],
            shortTitle: "Criar nota",
            systemImageName: "note.text.badge.plus"
        )
    }
}
```

Detalhes em [ios-nativo.md](ios-nativo.md).

## Reusando lógica RN via Headless Task / Background Task

Se a lógica que você quer expor pro agent **só existe no bundle JS**, dá pra:

- **Android:** `HeadlessJsTaskService` — sua função `@AppFunction` inicia um service que carrega o bundle RN e executa um task JS registrado.
- **iOS:** mais limitado. App Intents podem rodar em background, mas iniciar o JS engine só pra responder é caro. Prefira mover a lógica pra Swift ou pro backend.

> Regra prática: **se a função do agent precisa rodar com app fechado**, **não dependa do bundle RN**. Reescreva a lógica em Kotlin/Swift ou bata num backend.

## Como testar

- Android: `adb shell dumpsys appfunctions` mostra catálogo do seu pacote.
- iOS: abra Apple Atalhos, busque pelo nome do seu app.
- Validação real: Gemini / Siri responder à phrase configurada.

## Dicas e pegadinhas

- **Não tente expor JS function direto.** O sistema operacional indexa metadata em build-time (KSP no Android, macros Swift no iOS). JS não participa disso.
- **Mantenha a lógica reutilizável** entre o app RN e a função do agent — duplicar inevitavelmente sai de sincronia.
- **Use API/backend como fonte da verdade** sempre que possível. É o caminho menos doloroso.
- **Versões nativas crescem.** Avalie se faz sentido manter uma codebase RN se metade do app moderno é nativo. Pode valer migrar trechos.

## Ver também

- [Android nativo](android-nativo.md) — base do lado Android.
- [iOS nativo](ios-nativo.md) — base do lado iOS.
- [flutter](flutter.md) / [cordova-ionic](cordova-ionic.md) — mesma estratégia em outras stacks.
