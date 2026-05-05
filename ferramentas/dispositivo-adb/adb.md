# ADB (Android Debug Bridge)

> **Categoria:** Dispositivo / ADB
> **Plataforma:** Android
> **Quando usar:** controlar um dispositivo (físico ou emulador) pelo PC — instalar/desinstalar app, ver logs, puxar arquivos, abrir shell, port forward.
> **Alternativas:** nenhuma direta — ADB é a ponte oficial. Para inspeção visual, complementa com Android Studio.

## O que faz

**ADB = Android Debug Bridge**. Ponte oficial pra controlar um Android (emulador ou aparelho) pelo PC via **USB** ou **rede**. Funciona em três peças:

- **adb client** (comando que você roda no PC)
- **adb server** (processo que escuta no PC)
- **adbd** (daemon no Android)

## Instalação (Windows)

Vem no **Android SDK Platform Tools**. Baixe direto do site do Android Developers ou via Android Studio. Adicione `...\Android\Sdk\platform-tools` ao PATH.

Teste:

```bash
adb version
```

## Como usar

### Listar dispositivos conectados

```bash
adb devices -l
```

Estados possíveis:

- `device` — conectado e autorizado
- `unauthorized` — falta aceitar "Confiar neste computador?" no Android
- `offline` — conexão ruim/travada

Detalhes de um device específico:

```bash
adb -s <ID> shell getprop ro.product.model
adb -s <ID> shell getprop ro.build.version.release
```

### Reiniciar o servidor (quando "some" um device)

```bash
adb kill-server
adb start-server
adb devices -l
```

### Instalar / abrir / limpar app

```bash
adb install app.apk
adb shell am start -n pacote/.MainActivity
adb shell pm clear pacote
adb uninstall pacote
```

### Coletar logs e estado

```bash
adb logcat
adb shell dumpsys package pacote
adb shell dumpsys activity
adb bugreport
```

Pega:

- crashes e erros
- chamadas suspeitas (rede, permissões, serviços, receivers)
- comportamento em background

### Inspecionar arquivos

```bash
adb pull /sdcard/Download/arquivo .
adb shell ls -la
adb shell cat /proc/net/tcp
```

Acessar `/data/data/<pacote>` exige **root** ou `run-as` (se o app for debuggable).

### Shell remoto

```bash
adb shell
```

Daí usa `pm`, `am`, `ps`, `top`, `getprop`, `ip`, etc.

### Port forwarding (instrumentação, Frida, debug)

```bash
adb forward tcp:27042 tcp:27042
adb reverse tcp:8080 tcp:8080
```

`forward` = PC → device. `reverse` = device → PC.

### Acesso root no emulador

Em emuladores **AOSP/google_apis** (não em `google_apis_playstore`):

```bash
adb root
adb shell
id
ls /data
```

## Dicas e pegadinhas

- **Cabo USB ruim** é a causa nº 1 de "ADB cai sozinho". Vale trocar antes de culpar driver/Windows.
- Em **aparelho pessoal**, ADB ligado é risco. Em laboratório: device isolado, sem conta pessoal, **revogar autorizações** quando terminar (Opções do desenvolvedor).
- Prefira **USB** em vez de ADB via rede. Rede só em ambiente isolado.
- Se múltiplos devices conectados, sempre use `-s <serial>` pra evitar comando ir pro device errado.

## Ver também

- [adb-proxy-global](adb-proxy-global.md) — configurar proxy global do Android via ADB.
- [emulator](emulator.md) — controlar emuladores pela CLI.
- [Fluxo: extrair APK de app instalado](../../dicas/extrair-apk-de-app-instalado.md) — `pm path` + `adb pull` em um só lugar.
- [Fluxo: capturar tráfego HTTPS no Android](../../dicas/capturar-trafego-https-android.md)
- [Fluxo: validar evento Google Analytics](../../dicas/validar-evento-google-analytics.md)
