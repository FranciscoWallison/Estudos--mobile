# Charles Proxy

> **Categoria:** Rede / Proxy
> **Plataforma:** PC (Windows / macOS / Linux), inspeciona Android e iOS
> **Quando usar:** ver, modificar e replay requests HTTP/HTTPS de um app, com interface gráfica fácil. Map remote/local, breakpoints, throttling.
> **Alternativas:** mitmproxy (terminal, scriptável), Burp Suite (foco segurança), Proxyman (Mac), Fiddler.

## O que faz

Proxy de inspeção HTTP/HTTPS clássico, com GUI. Você configura o device pra usar o Charles como proxy, instala o cert, e passa a ver toda chamada.

- Inspeciona request/response (headers, body, cookies, JSON).
- **Map Local** — substitui resposta de uma URL por arquivo local.
- **Map Remote** — redireciona uma URL pra outro host (tipo o [Multi Reverse Proxy](multi-reverse-proxy.md), com GUI).
- **Rewrite** — modifica headers/body em runtime.
- **Breakpoints** — pausa request, edita, libera.
- **Throttle** — simular 3G/Edge/offline.
- **Repeat** — re-disparar request manualmente.

## Instalação (Windows)

Baixe em <https://www.charlesproxy.com/download/>. Instalador `.msi`. Versão paga (trial 30 dias com janelas de aviso a cada 30 minutos).

## Como usar

### 1. Configurar Charles

Por padrão escuta em `127.0.0.1:8888`. Pra device externo (Android/iOS):

- **Proxy → Proxy Settings → Proxies** → confirma porta 8888.
- **Proxy → Access Control Settings** → adiciona rede do device (ou `0.0.0.0/0` em rede isolada).
- **Proxy → SSL Proxying → Enable SSL Proxying** + adiciona `*` (ou hosts específicos) na lista.

### 2. Instalar cert no Android

- No Charles: **Help → SSL Proxying → Install Charles Root Certificate on a Mobile Device or Remote Browser**.
- Ele mostra: configurar proxy `<IP_PC>:8888` no device + abrir `chls.pro/ssl` no navegador → baixar cert → instalar como **CA do usuário** em **Configurações → Segurança → Credenciais**.

### 3. Apontar o Android pro Charles

Caminho A — proxy global via ADB (não precisa de Wi-Fi):

```bash
adb shell settings put global http_proxy <IP_DO_PC>:8888
```

Caminho B — proxy do Wi-Fi: **Configurações → Wi-Fi → rede → Avançado → Proxy → Manual**.

### 4. Pronto

Abre o app. Requests aparecem na timeline do Charles em árvore (host > path > request).

### Map Remote (substitui Multi Reverse Proxy com GUI)

**Tools → Map Remote → Add**. From: `https://api.exemplo.com/*` → To: `https://api-teste.exemplo.com/*`.

### Repeat

Botão direito numa request → **Repeat**. Pra fuzz: **Repeat Advanced** (define quantas vezes, intervalo).

## Dicas e pegadinhas

- Em **Android 7+**, apps que não optam por confiar em "User CA" no Network Security Config **não vão respeitar** o cert Charles. Workaround: usar build debuggable ou patch no APK.
- **SSL pinning** ignora o cert do Charles mesmo com tudo configurado — combine com [Objection](../seguranca-pentest/objection.md) (`android sslpinning disable`).
- A versão trial mostra um diálogo bloqueante a cada 30min — ok pra debug pontual, ruim pra sessão longa.
- **Map Remote** preserva path/query, igual ao [Multi Reverse Proxy](multi-reverse-proxy.md), só que com checkbox.
- Pra debug de payload binário (gRPC, protobuf), Charles ajuda menos — considere mitmproxy com plugin.

## Ver também

- [Multi Reverse Proxy](multi-reverse-proxy.md) — alternativa code-only sem GUI.
- [adb-proxy-global](../dispositivo-adb/adb-proxy-global.md) — apontar Android pro Charles via USB.
- [Objection](../seguranca-pentest/objection.md) — bypass de SSL pinning antes da inspeção.
- [Fluxo: capturar tráfego HTTPS no Android](../../dicas/capturar-trafego-https-android.md)
- Doc oficial: <https://www.charlesproxy.com/documentation/>
