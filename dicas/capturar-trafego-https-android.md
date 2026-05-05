# Capturar tráfego HTTPS de um app Android

> **Objetivo:** ver as requisições HTTP/HTTPS que um app Android faz, com payload visível.
> **Plataforma:** Android
> **Tempo estimado:** 15-30 min (na primeira vez, por causa do cert)
> **Pré-requisitos:** ADB autorizado; um proxy de inspeção no PC ([Charles](../ferramentas/rede-proxy/charles.md) / Burp / mitmproxy / Fiddler) OU o [Multi Reverse Proxy](../ferramentas/rede-proxy/multi-reverse-proxy.md) deste projeto.

## Quando usar este fluxo

- Você quer **debugar** uma chamada de API específica do app.
- Você quer **redirecionar** o app de Dev pra um backend de teste sem mexer no código.
- Você suspeita de uso indevido de dados (analytics oculto, exfiltração).

Existem **dois caminhos**, e a escolha muda tudo:

| Caminho                            | O que intercepta                                | Use quando                                                          |
| ---------------------------------- | ----------------------------------------------- | ------------------------------------------------------------------- |
| **A) Proxy global via ADB**        | Todo tráfego HTTP/HTTPS do device que respeita proxy do sistema | Você quer ver tudo que o app faz, ou não tem como mudar a URL no app. |
| **B) `adb reverse` + proxy local** | Só o que o app aponta pra `localhost:<porta>`   | Você controla o app de Dev e pode apontar pra `localhost`.          |

## Ferramentas envolvidas

- [adb](../ferramentas/dispositivo-adb/adb.md)
- [adb-proxy-global](../ferramentas/dispositivo-adb/adb-proxy-global.md) (caminho A)
- [Multi Reverse Proxy](../ferramentas/rede-proxy/multi-reverse-proxy.md) (caminho B)
- Burp / Charles / mitmproxy / Fiddler (caminho A para inspeção real de HTTPS)

## Caminho A — Proxy global via ADB

### 1. Suba o proxy de inspeção no PC

Burp/Charles/mitmproxy/Fiddler escutando em **`0.0.0.0:8080`** (não só `127.0.0.1`).

### 2. Descubra o IP do PC alcançável pelo Android

No PC:

```powershell
ipconfig
```

Pegue o IP da interface que o Android enxerga (Wi-Fi compartilhada, tethering USB, etc.).

Confirme que o Android alcança:

```bash
adb shell ping -c 2 <IP_DO_PC>
```

### 3. Configure o proxy global

```bash
adb shell settings put global http_proxy <IP_DO_PC>:8080
adb shell settings get global http_proxy
```

Saída do `get` deve ser exatamente `<IP_DO_PC>:8080`.

### 4. Instale o certificado do proxy no Android

Sem isso, HTTPS aparece só como `CONNECT` opaco.

- Burp: `http://burp` no navegador do Android → baixar `cacert.der`.
- Charles: SSL Proxying → Save Charles Root Certificate → instalar no device.
- mitmproxy: `http://mitm.it` no navegador do Android → baixar Android cert.

Instale como **certificate authority** em **Configurações → Segurança → Credenciais**.
Em Android 7+, apps só confiam em CAs de usuário se o app permitir explicitamente — apps com **SSL pinning** ou Network Security Config restrito **não vão respeitar seu cert** mesmo instalado.

### 5. Desative quando terminar

```bash
adb shell settings put global http_proxy :0
```

## Caminho B — `adb reverse` + proxy local

Não inspeciona HTTPS de qualquer app. **Aponta** uma chamada `http://localhost:<porta>` do app pra um proxy seu no PC, que repassa pro backend real (HTTP ou HTTPS).

### 1. Suba o Multi Reverse Proxy

Veja [Multi Reverse Proxy](../ferramentas/rede-proxy/multi-reverse-proxy.md). Exemplo:

```bash
node proxy.js
```

Saída:

```
Proxy ouvindo em http://localhost:52714 -> https://api-de-teste.exemplo.com.br
```

### 2. Configure o `adb reverse`

```powershell
adb reverse --remove-all
adb reverse tcp:8092 tcp:52714
adb reverse --list
```

Esperado:

```
UsbFfs tcp:8092 tcp:52714
```

### 3. App aponta pra localhost

No código do app (modo Dev):

```js
if (TYPE_SYSTEM == 'Dev') {
  urlApi = 'http://localhost:8092';
} else {
  urlApi = 'https://api-prod.exemplo.com.br';
}
```

### 4. Confirme

Abra o app, dispare uma request, veja log do `proxy.js` no PC.

## Validação

- **Caminho A:** request do app aparece na timeline do Burp/Charles/mitmproxy com payload visível.
- **Caminho B:** terminal do `proxy.js` mostra request entrando e response saindo.

## Troubleshooting

**App não respeita o proxy global**
SSL pinning ou Network Security Config bloqueando. Próximos passos: [Objection](../ferramentas/seguranca-pentest/objection.md) (`android sslpinning disable`), [Frida](../ferramentas/seguranca-pentest/frida.md) com hook custom, patch no APK (apktool + recompile), ou trocar pra build debuggable.

**Cert instalado mas Chrome ainda dá ERR_CERT_AUTHORITY_INVALID**
Chrome tem store próprio em algumas versões — instale como CA em **Credenciais → CA do usuário** (não em "credenciais Wi-Fi/VPN").

**`adb reverse --list` vazio depois de funcionar**
USB caiu / `adbd` reiniciou. Refaça `adb reverse tcp:8092 tcp:52714`. Cabo USB ruim é a causa nº 1.

**Backend retorna 502 / connection reset**
No caminho B: o `proxy.js` não conseguiu falar com o backend. Verifique conectividade do PC.

**Caminho A não pega `localhost` do app**
`localhost` no Android = o próprio device. Caminho A não ajuda. Use caminho B.

## Ver também

- [Multi Reverse Proxy](../ferramentas/rede-proxy/multi-reverse-proxy.md)
- [adb-proxy-global](../ferramentas/dispositivo-adb/adb-proxy-global.md)
- [Fluxo: validar evento Google Analytics](validar-evento-google-analytics.md)
