# AppFunctions — Android nativo (Kotlin + KSP)

> **Categoria:** App Functions
> **Plataforma:** Android (Jetpack AppFunctions, em rollout — checar versão atual)
> **Quando usar:** expor funções do seu app pro Gemini / Google Assistant / outros agents poderem chamar com argumentos tipados, sem o usuário abrir o app.
> **Alternativas:** App Actions (geração antiga, baseada em `shortcuts.xml` + BIIs), App Shortcuts estáticos. AppFunctions é a evolução com tipagem e descoberta pela IA.

## O que faz

Você anota funções Kotlin com `@AppFunction`. O **KSP** (Kotlin Symbol Processing) gera código de registro. No install, o Android indexa essas funções num catálogo do sistema. O Gemini (ou outro agent autorizado) descobre, gera argumentos a partir da linguagem natural do usuário e chama sua função em background. O retorno vai pra IA, que apresenta ao usuário.

## Pré-requisitos

- `compileSdk` na versão que suporta AppFunctions (Android 15+/16+ — confira a doc oficial, a API ainda está em evolução).
- KSP configurado no projeto.
- Kotlin 1.9+.

## Instalação

`build.gradle.kts` (Module):

```kotlin
plugins {
    id("com.google.devtools.ksp")
}

dependencies {
    implementation("androidx.appfunctions:appfunctions-runtime:<versão-atual>")
    implementation("androidx.appfunctions:appfunctions-agent:<versão-atual>")
    ksp("androidx.appfunctions:appfunctions-compiler:<versão-atual>")
}
```

> Veja a versão estável atual em <https://developer.android.com/jetpack/androidx/releases/appfunctions>. A API está em fase de rollout; nomes de artefatos podem mudar.

## Como usar

### 1. Declare uma data class serializável

```kotlin
import androidx.appfunctions.AppFunctionSerializable

@AppFunctionSerializable
data class NovaNota(
    val titulo: String,
    val conteudo: String,
    val etiquetas: List<String> = emptyList()
)
```

### 2. Crie uma classe com a função e anote

```kotlin
import androidx.appfunctions.AppFunction
import androidx.appfunctions.AppFunctionContext

class MinhasFuncoesDeApp {

    @AppFunction
    suspend fun criarNota(
        context: AppFunctionContext,
        nota: NovaNota
    ): String {
        // sua lógica de negócio aqui (DB, API, etc)
        val id = repositorio.salvar(nota)
        return "Nota criada com id $id"
    }

    @AppFunction
    suspend fun enviarMensagem(
        context: AppFunctionContext,
        destinatario: String,
        texto: String
    ): Boolean {
        return mensageiro.enviar(destinatario, texto)
    }
}
```

### 3. Declare no `AndroidManifest.xml`

```xml
<application ...>
    <meta-data
        android:name="android.app.appfunctions"
        android:resource="@xml/app_functions" />
</application>
```

`res/xml/app_functions.xml` (gerado parcialmente pelo KSP — confira o arquivo final no `build/`):

```xml
<app-functions xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- KSP gera entries baseadas nas funções anotadas -->
</app-functions>
```

### 4. Compile

O KSP gera as classes de bridge. No build, o catálogo de funções é embedado e o sistema indexa no install.

## Como testar

### Inspecionar funções registradas

```bash
adb shell dumpsys appfunctions
```

Mostra catálogo de funções por pacote.

### Chamar manualmente via shell (dev)

```bash
adb shell cmd appfunctions invoke \
  --package com.exemplo.app \
  --function criarNota \
  --args '{"nota":{"titulo":"Lembrete","conteudo":"Pagar conta"}}'
```

> Comando exato pode variar entre versões — confira `adb shell cmd appfunctions help`.

### Via Gemini (validação real)

Em device com Gemini ativo, peça em linguagem natural: *"crie uma nota no [seu app] com título 'Lembrete' e conteúdo 'Pagar conta'"*. Se o catálogo foi indexado, Gemini propõe a chamada.

## Dicas e pegadinhas

- **Funções são `suspend`** — rodam em coroutine. Faça I/O sem bloquear.
- O **`AppFunctionContext`** é injetado pelo framework. Não invente um, não chame de fora — só do agent.
- **Descrições matam ou salvam o feature.** Use KDoc nos parâmetros — o KSP extrai pra metadata da função, e a IA usa pra mapear linguagem natural pro argumento certo.
- Funções precisam ser **idempotentes quando possível**. A IA pode tentar de novo em caso de timeout.
- **R8/ProGuard:** anote as classes geradas como `@Keep` ou ajuste regras — sem isso a release build perde o catálogo.
- Pra retornar **UI rica** (card que aparece na resposta do Gemini), use `AppFunctionView` (composable). Pra retornar dados estruturados, `AppFunctionEntity`.
- **Permissões sensíveis** (locação, contatos): declare no manifest E anote a função com a permissão necessária — a IA pede ao usuário antes de chamar.

## Schemas pré-definidos

Pra funções comuns (mensagem, nota, item de checklist, contato), use schemas do AndroidX que já vêm com nomes e parâmetros padronizados. Isso facilita pra IA mapear entre apps diferentes (ex: "criar nota" funciona em qualquer app que implemente o schema `CreateNote`).

```kotlin
import androidx.appfunctions.schema.notes.CreateNoteAppFunction

class MinhasFuncoes : CreateNoteAppFunction {
    override suspend fun createNote(
        context: AppFunctionContext,
        parameters: CreateNoteAppFunction.Parameters
    ): CreateNoteAppFunction.Response {
        // implementação
    }
}
```

## Ver também

- [iOS nativo](ios-nativo.md) — equivalente Apple (App Intents).
- [react-native](react-native.md) / [flutter](flutter.md) — como expor a partir de stack cross-platform.
- [adb](../dispositivo-adb/adb.md) — inspeção via `dumpsys appfunctions`.
- Doc oficial: <https://developer.android.com/reference/androidx/appfunctions/package-summary>
- Schemas: <https://developer.android.com/reference/androidx/appfunctions/schema/package-summary>
