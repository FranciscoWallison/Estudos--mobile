# Google Analytics Debugger

> **Categoria:** Analytics / Debug
> **Plataforma:** Web (extensão Chrome) + Android (via ADB / setprop)
> **Quando usar:** validar em tempo real os hits/eventos que estão sendo enviados ao Google Analytics, sem esperar relatório.
> **Alternativas:** Firebase DebugView (GA4 web), Charles/mitmproxy (inspeção via proxy do request HTTP do hit).

## O que faz

Permite ver no console / logcat **exatamente o que está sendo enviado pro Google Analytics**: nome do evento, parâmetros, screen, sessão, user properties. Existem duas frentes complementares:

- **Extensão Chrome (Google Analytics Debugger):** liga modo verbose nas chamadas GA do navegador e imprime no DevTools.
- **Modo verbose no Android via ADB:** o SDK do GA passa a logar cada hit no `logcat`, com `setprop log.tag.GAv4 VERBOSE` (Universal) ou `setprop log.tag.FA VERBOSE` (GA4/Firebase).

## Instalação

### Extensão Chrome (web)

1. Chrome Web Store → procure "Google Analytics Debugger".
2. Adicionar ao Chrome.
3. Clicar no ícone da extensão alterna ON/OFF.
4. Abrir DevTools (F12) → aba **Console**. Os hits aparecem com prefixo do GA.

### Mobile (Android)

Não tem instalação. É só [ADB](../dispositivo-adb/adb.md) + `setprop`.

## Como usar

### Web (Chrome)

1. Liga a extensão.
2. Recarrega a página.
3. Console mostra coisas tipo:
   ```
   GA4 Tag manager event: page_view
   parameters: { page_location: "https://...", page_title: "..." }
   ```
4. Se o site usa **GTM**, dá pra usar também **Tag Assistant Companion** + **Google Tag Assistant** pra debug mais estruturado.

### Android — GA Universal (SDK `com.google.android.gms:play-services-analytics`)

```bash
adb shell setprop log.tag.GAv4 VERBOSE
adb shell setprop log.tag.GAv4-SVC VERBOSE

# (em outro terminal, ou na mesma)
adb logcat -v time -s GAv4 GAv4-SVC
```

Force-stop e reabre o app pra garantir que o SDK leia a flag:

```bash
adb shell am force-stop com.exemplo.app
```

Cada hit aparece na saída com nome, parâmetros e timestamp.

### Android — GA4 / Firebase Analytics

```bash
adb shell setprop log.tag.FA VERBOSE
adb shell setprop log.tag.FA-SVC VERBOSE
adb shell setprop debug.firebase.analytics.app com.exemplo.app

adb logcat -v time -s FA FA-SVC
```

Bônus: enquanto `debug.firebase.analytics.app` está setado pro pacote, os eventos também aparecem no **Firebase Console → Analytics → DebugView** (em ~30s).

### Desligando

```bash
adb shell setprop log.tag.GAv4 ""
adb shell setprop log.tag.GAv4-SVC ""
adb shell setprop log.tag.FA ""
adb shell setprop log.tag.FA-SVC ""
adb shell setprop debug.firebase.analytics.app .none.
```

(Ou reinicia o device — `setprop` não persiste no reboot.)

## Dicas e pegadinhas

- **App precisa ser debuggable** pra logs do SDK saírem em build de release. Use Dev/debug pra QA.
- `setprop` é runtime, mas muitos SDKs só leem a flag **na inicialização do app**. Sempre `am force-stop` + reabrir após setar.
- **Universal Analytics foi descontinuado em 2023** — se o app foi lançado depois, provavelmente é GA4/Firebase. Em apps antigos, pode coexistir.
- O **Firebase DebugView** só funciona com GA4, não com Universal.
- Nem todo evento "verbose" no logcat foi de fato enviado — alguns SDKs fazem batching. Confirme cruzando com [captura de tráfego](../../dicas/capturar-trafego-https-android.md) (procure requests pra `www.google-analytics.com` / `app-measurement.com`).
- VPN ou ad-blocker no device pode comer hits silenciosamente. Se o logcat mostra evento mas DebugView não, suspeite primeiro disso.
- Ad-blocker no Chrome quebra a extensão também — desative na página em teste.

## Ver também

- [adb](../dispositivo-adb/adb.md) — base.
- [Fluxo: validar evento Google Analytics](../../dicas/validar-evento-google-analytics.md) — passo-a-passo prático.
- [Fluxo: capturar tráfego HTTPS](../../dicas/capturar-trafego-https-android.md) — pra cruzar logcat × request real saindo do device.
