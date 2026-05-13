# App Functions — Flutter

> **Categoria:** App Functions
> **Plataforma:** Android (AppFunctions) + iOS (App Intents)
> **Quando usar:** expor funções de um app Flutter pro Gemini / Siri / Apple Intelligence.
> **Alternativas:** Deep Links + flutter_intent_helper antigos; ainda não há plugin pronto cobrindo tudo de forma uniforme.

## Realidade atual (2026)

Mesma situação do React Native: **nenhum plugin oficial cross-platform** pra App Functions ainda.

- **AppFunctions usa KSP** (Kotlin) — não toca Dart.
- **App Intents usa Swift macros** — não toca Dart.

Você implementa **na ponta nativa** (Kotlin pro Android, Swift pro iOS) e, se precisar, comunica com Dart via `MethodChannel` quando o app está em primeiro plano. Em background (caso comum quando IA chama), use backend ou camada compartilhada nativa.

## Arquitetura recomendada

```
Flutter (Dart)
└── main.dart ─────────────┐  (não chamado direto pela IA)
                           │
Android (Kotlin)           │
└── @AppFunction criarNota │ ─ acessa backend / Room / serviço ─ retorna pra IA
                           │
iOS (Swift)                │
└── AppIntent CriarNota   │ ─ acessa backend / CoreData / serviço ─ retorna pra IA
```

A lógica de negócio deve viver em **camada compartilhada**:

- Backend HTTP / GraphQL próprio.
- Camada nativa (Kotlin + Swift) que tanto o Flutter chama via MethodChannel quanto o intent/função chama direto.
- **FlutterEngine background** (Android) ou **App Extension** (iOS) — pra reusar Dart, ver "Reusando Dart" abaixo.

## Passo a passo (Android)

### 1. Crie um módulo Kotlin no `android/app/src/main/kotlin/`

```kotlin
package com.exemplo.app.appfunctions

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
        // Recomendado: chamar backend / camada nativa
        val id = ApiClient.criarNota(nota.titulo, nota.conteudo)
        return "Nota $id criada"
    }
}
```

### 2. Adicione KSP + dependências

Detalhes em [android-nativo.md](android-nativo.md#instalação). Edite o `android/app/build.gradle.kts` com `id("com.google.devtools.ksp")` e `ksp("androidx.appfunctions:appfunctions-compiler:...")`.

### 3. Manifest

Adicione meta-data `android.app.appfunctions` em `android/app/src/main/AndroidManifest.xml`. Igual ao [android-nativo.md](android-nativo.md#3-declare-no-androidmanifestxml).

## Passo a passo (iOS)

### 1. Adicione o Intent em Swift dentro do projeto iOS

`ios/Runner/AppIntents/CriarNotaIntent.swift`:

```swift
import AppIntents

struct CriarNotaIntent: AppIntent {
    static let title: LocalizedStringResource = "Criar nota"

    @Parameter(title: "Título") var titulo: String
    @Parameter(title: "Conteúdo") var conteudo: String

    func perform() async throws -> some IntentResult & ProvidesDialog {
        let id = try await ApiClient.shared.criarNota(titulo: titulo, conteudo: conteudo)
        return .result(dialog: "Nota \(id) criada")
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

Confira que o arquivo está incluído no target principal do `Runner.xcodeproj`.

## Reusando lógica Dart (avançado)

Se a regra de negócio só existe em Dart, dá pra subir uma **FlutterEngine background** quando a função do agent é chamada:

### Android — FlutterEngine background

```kotlin
@AppFunction
suspend fun criarNota(context: AppFunctionContext, nota: NovaNota): String {
    val engine = FlutterEngine(context.applicationContext)
    engine.dartExecutor.executeDartEntrypoint(
        DartExecutor.DartEntrypoint("lib/main.dart", "criarNotaEntrypoint")
    )
    val channel = MethodChannel(engine.dartExecutor.binaryMessenger, "appfunctions/criarNota")
    val result = channel.invokeMethodAsync("criarNota", mapOf("titulo" to nota.titulo, "conteudo" to nota.conteudo))
    return result.toString()
}
```

E no Dart:

```dart
@pragma('vm:entry-point')
void criarNotaEntrypoint() {
  WidgetsFlutterBinding.ensureInitialized();
  const channel = MethodChannel('appfunctions/criarNota');
  channel.setMethodCallHandler((call) async {
    if (call.method == 'criarNota') {
      return await NotasRepository.criar(call.arguments['titulo'], call.arguments['conteudo']);
    }
  });
}
```

### iOS — limitado

`AppIntent` rodando em background no iOS **não** sobe FlutterEngine facilmente. Custo de inicialização é alto. Pra iOS, prefira:

- mover lógica pra Swift, ou
- bater num backend, ou
- aceitar `OpensAppWhenRun = true` (abre o app pra processar, perdendo a vantagem do background).

> **Regra prática:** se a função do agent precisa rodar com app fechado, **não dependa de Dart**. Use camada nativa ou backend.

## Como testar

- Android: `adb shell dumpsys appfunctions`.
- iOS: abrir Apple Atalhos, buscar pelo nome do app.
- Real: Gemini / Siri respondendo phrase configurada.

## Dicas e pegadinhas

- **Hot reload do Flutter não afeta o catálogo** — mudanças nas funções nativas exigem rebuild full e reinstall.
- **Dart é seu app, não sua interface pra IA.** A API contratada com o agent vive no nativo.
- **Plugins de terceiros** podem surgir nessa área (`flutter_app_intents`, `flutter_appfunctions`) — antes de adotar, confira que o repo é ativo (commit nos últimos 3-6 meses) e cobre ambas as plataformas.
- Pra **mesmas funções aparecerem nos atalhos da loja**, mantenha title/phrases estáveis entre versões.
- **Localização** do título e descrição dos intents impacta diretamente como o assistente entende o usuário.

## Ver também

- [Android nativo](android-nativo.md) — base do lado Android.
- [iOS nativo](ios-nativo.md) — base do lado iOS.
- [react-native](react-native.md) / [cordova-ionic](cordova-ionic.md) — estratégia equivalente em outras stacks.
