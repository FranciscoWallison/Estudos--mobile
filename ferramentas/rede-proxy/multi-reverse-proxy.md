# Multi Reverse Proxy (Node.js) + ADB Reverse

> **Categoria:** Rede / Proxy
> **Plataforma:** Android (lado app) + PC (lado proxy)
> **Quando usar:** apontar o app de Dev pra `http://localhost:porta` e fazer essa porta cair num backend HTTPS de teste, sem precisar de Wi-Fi nem mexer em DNS.
> **Alternativas:** [proxy global via ADB](../dispositivo-adb/adb-proxy-global.md) (intercepta TODO tráfego do device); [Charles](charles.md)/mitmproxy (interceptam, mas focados em inspeção e não em rewrite simples para um host fixo).

## O que faz

Sobe um servidor HTTP local em Node que encaminha cada requisição recebida para um backend (HTTP ou HTTPS) preservando rota e query string. Combinado com `adb reverse`, o app no Android chama `localhost:8092` e a request chega no backend real através do PC, via cabo USB.

Fluxo:

```
APP (Android) -> http://localhost:8092 -> adb reverse (USB) -> PC:52714 -> Proxy Node -> https://api-de-teste.exemplo.com.br
```

## O que ele faz, detalhado

- Sobe um servidor HTTP local (`http.createServer`).
- Para cada request:
  - monta o `path` final juntando `targetUrl.pathname` (base do alvo) + `req.url` (rota + query da requisição local).
  - replica **método** (GET/POST/PUT/DELETE etc.).
  - repassa **headers** (ajusta `host` pro host do backend).
  - encaminha o **body** (stream) pro backend.
  - devolve **status** e **headers** do backend pro cliente.
- Se o backend falhar, responde `502 Bad Gateway` em JSON.

## Quando isso é útil

- Desenvolvimento local chamando APIs remotas.
- Vários backends acessíveis por portas diferentes (mesmo PC, mesmo proxy).
- Testar integrações sem alterar o código do cliente.
- Centralizar endpoints de teste/QA.

> Não é proxy "com autenticação", "cache" ou "inspeção TLS". É um encaminhador direto.

## Requisitos

- Node.js 16+ (recomendado 18+).
- Android com **Depuração USB** ativada.
- `adb` no PATH.
- Cabo USB decente (afeta MUITO a estabilidade).

## Como rodar

1. Salve o código abaixo como `proxy.js`.
2. Rode:

   ```bash
   node proxy.js
   ```

   Saída esperada:

   ```
   Proxy ouvindo em http://localhost:52714 -> https://api-teste-teste.com.br
   Proxy ouvindo em http://localhost:58889 -> https://api-teste3-teste.com.br
   ```

3. Configure o `adb reverse` (ver [fluxo: capturar tráfego HTTPS](../../dicas/capturar-trafego-https-android.md), seção "via reverse").

   Comando rápido:

   ```powershell
   adb reverse --remove-all
   adb reverse tcp:8092 tcp:52714
   adb reverse --list
   ```

## Configuração (porta → alvo)

No fim do `proxy.js`, declare cada par:

```js
createProxy(52714, 'https://api-teste-teste.com.br');
createProxy(58889, 'https://api-teste3-teste.com.br');
```

Adicionar mais:

```js
createProxy(60000, 'https://minha-api.com.br');
```

## Detalhes importantes

### Preserva rota e query

- App chama `http://localhost:52714/a/b?c=1`
- Proxy encaminha para `https://api-teste-teste.com.br/a/b?c=1`

### Ajuste de Host

O header `host` é forçado para o host do backend:

```js
host: targetUrl.hostname
```

Evita backend rejeitando request por `Host` errado.

### Erros

Backend caído / recusando conexão → resposta:

- Status: `502`
- Body: `{ "error": "Bad Gateway", "detail": "mensagem do erro" }`

## Limitações (do jeito que está)

- **Sem suporte explícito a WebSocket**.
- Não tem timeout configurado (depende do default do Node).
- Não tem logs detalhados por request (só loga erro).
- HTTPS é usado **do proxy para o alvo**; o lado local continua HTTP.
- Não faz reescrita avançada de headers além do `host`.

## Boas práticas / segurança

- Não exponha isso na internet — só uso local / rede controlada.
- Em time/ambiente compartilhado, considere allowlist de hosts, autenticação, rate limit, logs e timeouts.

## Troubleshooting rápido

**502 Bad Gateway**

- API alvo fora do ar.
- DNS/host errado.
- Bloqueio de rede/firewall.
- Endpoint exige TLS/cert específico (raros).

**Backend responde diferente do esperado**

- Confira como o `path` está sendo montado.
- Se o alvo tem `pathname` ≠ `/`, ele entra como base.

**`adb reverse --list` vazio depois de funcionar**

- USB caiu / `adbd` reiniciou. Refaça `adb reverse tcp:8092 tcp:52714`.

## Código de referência (`proxy.js`)

```js
const http = require('http');
const https = require('https');
const { URL } = require('url');

function createProxy(localPort, target) {
  const targetUrl = new URL(target);
  const isHttps = targetUrl.protocol === 'https:';
  const client = isHttps ? https : http;
  const defaultPort = isHttps ? 443 : 80;

  const server = http.createServer((req, res) => {
    const basePath = targetUrl.pathname === '/' ? '' : targetUrl.pathname;
    const finalPath = basePath + req.url;

    const options = {
      protocol: targetUrl.protocol,
      hostname: targetUrl.hostname,
      port: targetUrl.port || defaultPort,
      method: req.method,
      path: finalPath,
      headers: {
        ...req.headers,
        host: targetUrl.hostname,
      },
    };

    const proxyReq = client.request(options, (proxyRes) => {
      res.writeHead(proxyRes.statusCode || 500, proxyRes.headers);
      proxyRes.pipe(res, { end: true });
    });

    proxyReq.on('error', (err) => {
      console.error(`Erro no proxy da porta ${localPort}:`, err.message);
      if (!res.headersSent) {
        res.writeHead(502, { 'Content-Type': 'application/json' });
      }
      res.end(JSON.stringify({ error: 'Bad Gateway', detail: err.message }));
    });

    req.pipe(proxyReq, { end: true });
  });

  server.listen(localPort, () => {
    console.log(`Proxy ouvindo em http://localhost:${localPort} -> ${target}`);
  });
}

createProxy(52714, 'https://api-teste-teste.exemplo.com.br');
createProxy(58889, 'https://api-teste3-teste.exemplo.com.br');
```

## Ver também

- [adb](../dispositivo-adb/adb.md) — porta de entrada do `adb reverse`.
- [adb-proxy-global](../dispositivo-adb/adb-proxy-global.md) — alternativa quando você precisa interceptar TODO o tráfego do device, não só o app de Dev.
- [Fluxo: capturar tráfego HTTPS no Android](../../dicas/capturar-trafego-https-android.md) — passo-a-passo escolhendo entre os dois caminhos.
