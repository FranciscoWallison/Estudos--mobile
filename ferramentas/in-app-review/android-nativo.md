# In-App Review — Android nativo (Play Core)

> **Categoria:** In-App Review
> **Plataforma:** Android (API 21+ / Play Store instalado)
> **Quando usar:** popup de avaliação Google sem sair do app, em projeto Android nativo Kotlin ou Java.
> **Alternativas:** abrir Play Store via `Intent` (`market://details?id=...`) — leva o usuário pra fora do app, fluxo antigo.

## O que faz

A API **Play Core In-App Review** mostra um card sobreposto no seu app onde o usuário deixa estrelas e comentário. Tudo dentro da sua Activity. Quem mostra o popup é o **Play Store**, não você.

## Pré-requisitos

- Device com **Google Play Store** instalado e usuário logado.
- App publicado no Play Store (ou em **Internal App Sharing** pra teste).
- `minSdk >= 21`.

## Instalação

`build.gradle` (Module):

```gradle
dependencies {
    implementation 'com.google.android.play:review:2.0.1'
    // versão Kotlin (extensions com coroutine support)
    implementation 'com.google.android.play:review-ktx:2.0.1'
}
```

## Como usar

### Kotlin (com extensions)

```kotlin
import com.google.android.play.core.review.ReviewManagerFactory

class MainActivity : AppCompatActivity() {

    private val reviewManager by lazy { ReviewManagerFactory.create(this) }

    private fun pedirAvaliacao() {
        val request = reviewManager.requestReviewFlow()
        request.addOnCompleteListener { task ->
            if (task.isSuccessful) {
                val reviewInfo = task.result
                val flow = reviewManager.launchReviewFlow(this, reviewInfo)
                flow.addOnCompleteListener {
                    // Acabou o fluxo. Você NÃO sabe se o usuário avaliou.
                    salvarTimestampUltimoPrompt()
                }
            } else {
                // Falha silenciosa — pode ser: sem Play Store, quota interna, etc.
                // Não mostre erro pro usuário.
            }
        }
    }
}
```

### Java

```java
ReviewManager manager = ReviewManagerFactory.create(this);
Task<ReviewInfo> request = manager.requestReviewFlow();
request.addOnCompleteListener(task -> {
    if (task.isSuccessful()) {
        ReviewInfo reviewInfo = task.getResult();
        Task<Void> flow = manager.launchReviewFlow(this, reviewInfo);
        flow.addOnCompleteListener(t -> {
            // fluxo terminou
        });
    }
});
```

### Coroutine (KTX)

```kotlin
import com.google.android.play.core.ktx.requestReview
import com.google.android.play.core.ktx.launchReview

lifecycleScope.launch {
    try {
        val reviewInfo = reviewManager.requestReview()
        reviewManager.launchReview(this@MainActivity, reviewInfo)
    } catch (e: Exception) {
        // log, sem mostrar pro usuário
    }
}
```

## Como testar (sem o popup nunca aparecer)

Em **debug build instalado via Android Studio / ADB**, o card **NÃO aparece**. Isso é por design.

Pra validar visualmente:

1. **Internal App Sharing** — Play Console → Configuração → Compartilhamento interno do app → upload do APK/AAB → instalar pelo link.
2. **Faixa de teste interna** — Play Console → Testes → Teste interno → upload + adicionar testers → instalar pelo Play Store.

Em ambos, a API funciona como em prod (sujeita a quota).

Pra confirmar que **a chamada está sendo feita corretamente** sem o card visual, use [logcat](../dispositivo-adb/adb.md):

```bash
adb logcat | findstr "PlayCore"
```

Mensagens `Requested in-app review` / `Launching review flow` confirmam que o lado app fez a parte dele.

## Dicas e pegadinhas

- **`launchReviewFlow` precisa rodar na Main thread** e com a Activity em primeiro plano. Em Fragment, use `requireActivity()`.
- O `ReviewInfo` é **descartável** — depois de `launchReviewFlow`, ele expira; não dá pra reusar.
- Em devices **sem Play Store** (Huawei AppGallery, emuladores AOSP, ROM aberta), `requestReviewFlow` falha. Trate o `addOnFailureListener` em silêncio.
- A **quota é por usuário, por app, por janela de tempo** — Google não publica os números. Não vai aparecer toda vez.
- **Não** prefixe o popup com texto seu ("Avalia aí?") + botão pra chamar a API. Apple/Google querem o popup limpo, sem coerção.

## Ver também

- [iOS nativo](ios-nativo.md) — equivalente Apple.
- [react-native](react-native.md) / [flutter](flutter.md) — wrappers cross-platform sobre essa API.
- [adb](../dispositivo-adb/adb.md) — pra instalar via Internal App Sharing.
- Doc oficial: <https://developer.android.com/guide/playcore/in-app-review?hl=pt-br>
