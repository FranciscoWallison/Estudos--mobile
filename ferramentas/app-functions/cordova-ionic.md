# App Functions — Cordova / Ionic / Capacitor

> **Categoria:** App Functions
> **Plataforma:** Android (AppFunctions) + iOS (App Intents)
> **Quando usar:** expor funções de um app híbrido (Cordova ou Capacitor) pro Gemini / Siri / Apple Intelligence.
> **Alternativas:** Deep Links + URL schemes; abrir o app via Atalho legado.

## Realidade atual (2026)

App Functions (Android) e App Intents (iOS) são **features fortemente nativas** — KSP/Kotlin no Android, Swift macros no iOS. Em apps híbridos isso significa:

- **Você precisa escrever código nativo de qualquer jeito.**
- **A lógica de negócio que vive na WebView (JS/TS/Angular)** não roda em background quando o agent chama — a WebView só existe quando o app está aberto.
- **Capacitor** tem mecanismos melhores que **Cordova clássico** pra interop nativo, então é o caminho recomendado.

Não existe plugin oficial cross-platform pronto cobrindo as duas APIs nesse ecossistema. Você cria um **plugin local** (Capacitor) ou um **plugin Cordova custom**.

## Arquitetura recomendada (Capacitor)

```
Ionic WebView (Angular/React/Vue)
└── (não chamado direto pela IA)

Android (Kotlin, em android/app/src/main/...)
└── @AppFunction criarNota
    └─→ chama API HTTP / Capacitor Bridge / serviço nativo
         (lógica reusada, vive em backend ou camada nativa)

iOS (Swift, em ios/App/App/...)
└── AppIntent CriarNota
    └─→ chama API HTTP / serviço nativo
```

> Tudo que a função do agent precisa ser capaz de fazer **sem a WebView ativa** deve viver em backend ou camada nativa.

## Passo a passo — Capacitor (Android)

### 1. Adicione o arquivo Kotlin no projeto Android

`android/app/src/main/java/com/exemplo/app/appfunctions/MinhasFuncoes.kt`:

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
        // Caminho recomendado: chamar API/backend
        val id = ApiClient.criarNota(nota.titulo, nota.conteudo)
        return "Nota $id criada"
    }
}
```

### 2. Configure KSP em `android/app/build.gradle`

```gradle
plugins {
    id 'com.google.devtools.ksp'
}

dependencies {
    implementation 'androidx.appfunctions:appfunctions-runtime:<versão>'
    ksp 'androidx.appfunctions:appfunctions-compiler:<versão>'
}
```

Detalhes em [android-nativo.md](android-nativo.md#instalação).

### 3. Manifest

`android/app/src/main/AndroidManifest.xml` ganha o `<meta-data>` apontando pro `app_functions.xml`. Igual em [android-nativo.md](android-nativo.md#3-declare-no-androidmanifestxml).

## Passo a passo — Capacitor (iOS)

### 1. Adicione o intent em Swift no Xcode

`ios/App/App/AppIntents/CriarNotaIntent.swift`:

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

Inclua os arquivos no target principal do Xcode.

## Cordova clássico

Em projetos Cordova puros (sem Capacitor):

- Você cria um plugin Cordova com **platform code** Kotlin e Swift.
- A parte Kotlin tem a classe com `@AppFunction`.
- A parte Swift tem o `AppIntent`.
- O plugin **não expõe nada pro JS pelos canais clássicos** — a função do agent não passa pela WebView.
- O JS continua reusando os métodos do plugin via `cordova.exec(...)` pra quando o usuário abre o app normalmente.

Esse caminho é viável mas trabalhoso. Em 2026, **migrar pra Capacitor** facilita muito.

## Reusando lógica da WebView (limites)

Diferente de Flutter (FlutterEngine background) ou RN (HeadlessJsTaskService), Capacitor/Cordova **não tem mecanismo prático** pra subir uma WebView headless só pra responder a uma chamada do agent.

Opções:

- **Abrir o app:** declare `OpenIntent` no iOS / use intent que abre o app no Android. Perde-se o background, mas reusa-se a lógica JS.
- **Mover lógica pro nativo:** reescrever em Kotlin/Swift o que precisa ser callable por agent.
- **Backend como ponte:** o intent nativo chama API; a API faz o que o JS faria.

## Como testar

- Android: `adb shell dumpsys appfunctions`.
- iOS: Apple Atalhos.
- Real: Gemini / Siri respondendo phrase.

## Dicas e pegadinhas

- **Capacitor sync** (`npx cap sync`) **não propaga código nativo novo automaticamente** — você adiciona arquivos no `android/` ou `ios/` direto. Eles ficam fora do fluxo de re-geração do Capacitor.
- Cuidado pra não **perder os arquivos** quando alguém roda `npx cap copy` de forma errada — guarde Kotlin/Swift custom dentro de pastas próprias (`appfunctions/`, `AppIntents/`) que o Capacitor não toca.
- A maioria das libs Capacitor de "intents" antigas era pra **App Actions / Shortcuts** geração antiga, não pra AppFunctions/App Intents modernos. Verifique o que cada uma cobre antes de adotar.
- **Idioma da WebView ≠ idioma dos intents.** O intent é localizado pelo idioma do device, não pelo i18n da sua app Ionic.

## Ver também

- [Android nativo](android-nativo.md) — base do lado Android.
- [iOS nativo](ios-nativo.md) — base do lado iOS.
- [react-native](react-native.md) / [flutter](flutter.md) — outras stacks com a mesma fronteira nativa/JS.
- [In-App Review — Cordova/Ionic](../in-app-review/cordova-ionic.md) — exemplo de outra feature de plataforma exposta via plugin Capacitor.
