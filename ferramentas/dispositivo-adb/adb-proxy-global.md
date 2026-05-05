# ADB — proxy global do Android

> **Categoria:** Dispositivo / ADB
> **Plataforma:** Android
> **Quando usar:** forçar TODO o tráfego HTTP do Android a passar por um proxy do PC (Burp, Charles, Fiddler, mitmproxy) sem mexer em Wi-Fi.
> **Alternativas:** [`adb reverse`](../rede-proxy/multi-reverse-proxy.md) (alvo é só o app que aponta pra `localhost`); proxy via Wi-Fi (precisa estar na mesma rede).

## O que faz

Grava nas configurações globais do Android um proxy `host:porta`. Tudo que respeita proxy do sistema passa a sair pelo seu PC.

## Como usar

### Definir o proxy

```bash
adb shell settings put global http_proxy 192.168.1.10:8080
```

> Substitua `192.168.1.10:8080` pelo IP do seu PC + porta onde o proxy está escutando.

### Validar

```bash
adb shell settings get global http_proxy
```

Deve retornar `192.168.1.10:8080`.

### Desativar

```bash
adb shell settings put global http_proxy :0
```

(Alternativa: deletar `http_proxy` / `global_http_proxy_host` / `global_http_proxy_port`. `:0` normalmente resolve.)

## Dicas e pegadinhas

- **IP precisa ser alcançável pelo Android.** `127.0.0.1` no PC ≠ `127.0.0.1` no Android. Use o IP da interface que o Android enxerga (ex: tethering USB cria uma `192.168.42.x` ou link-local APIPA `169.254.x.x` no Windows).
- Faça **`ping <IP_PC>`** do Android pra confirmar alcance.
- O proxy do PC tem que escutar em **`0.0.0.0:porta`** (ou no IP específico que o Android usa). Se escutar só em `127.0.0.1`, não funciona.
- **Firewall do Windows** precisa liberar a porta de entrada na interface usada.
- Pra interceptar **HTTPS**, ainda falta instalar e confiar no **certificado do proxy** no Android. [Exemplo em vídeo](https://www.youtube.com/watch?v=ZUVUhkh2ELY).
- **Alguns apps com SSL pinning** ignoram esse proxy mesmo configurado — aí você precisa de Frida/Objection ou patch.

## Ver também

- [adb](adb.md) — base.
- [Multi Reverse Proxy](../rede-proxy/multi-reverse-proxy.md) — alternativa quando o app aponta pra `localhost`.
- [Fluxo: capturar tráfego HTTPS no Android](../../dicas/capturar-trafego-https-android.md)
