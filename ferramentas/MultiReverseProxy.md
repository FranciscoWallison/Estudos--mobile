# Multi Reverse Proxy (Node.js)

Um **proxy reverso HTTP** simples em Node.js que escuta em **portas locais** e encaminha as requisições para **APIs alvo** (HTTP ou HTTPS), preservando **rota** e **query string**.

Exemplo do projeto:
- `http://localhost:52714/*` -> `https://api-teste-teste.com.br/*`
- `http://localhost:58889/*` -> `https://api-teste3-teste.com.br/*`

---

## O que essa ferramenta faz

- Sobe um servidor HTTP local (`http.createServer`)
- Para cada requisição recebida:
  - monta o `path` final juntando:
    - `targetUrl.pathname` (base do alvo)
    - `req.url` (rota + query da requisição local)
  - replica **método** (GET/POST/PUT/DELETE etc.)
  - repassa **headers** (ajusta `host` pro host do backend)
  - encaminha o **body** (stream) pro backend
  - devolve **status code** e **headers** do backend pro cliente
- Se o backend falhar, responde `502 Bad Gateway` em JSON

---

## Quando isso é útil

- Desenvolvimento local chamando APIs remotas
- Ter vários backends acessíveis por portas diferentes
- Testar integrações sem alterar o código do cliente
- Centralizar endpoints de teste/QA

> Observação: isso **não** é um proxy “com autenticação”, “cache” ou “inspeção TLS”. É um encaminhador direto.

---

## Requisitos

- Node.js 16+ (recomendado 18+)

---

## Como rodar

1) Salve o arquivo como `proxy.js`

2) Rode:
```bash
node proxy.js
````

Saída esperada:

* `Proxy ouvindo em http://localhost:52714 -> https://api-teste-teste.com.br`
* `Proxy ouvindo em http://localhost:58889 -> https://api-teste3-teste.com.br`

---

## Como usar

### No navegador / cliente HTTP

* `http://localhost:52714/minha/rota?x=1`

  * vai para `https://api-teste-teste.com.br/minha/rota?x=1`

* `http://localhost:58889/v2/users`

  * vai para `https://api-teste3-teste.com.br/v2/users`

### Exemplo com curl

```bash
curl -i http://localhost:52714/health
curl -i http://localhost:58889/status
```

---

## Configuração (mapeamento porta -> alvo)

No final do `proxy.js` você define os proxys:

```js
createProxy(52714, 'https://api-teste-teste.com.br');
createProxy(58889, 'https://api-teste3-teste.com.br');
```

Para adicionar mais:

```js
createProxy(60000, 'https://minha-api.com.br');
```

---

## Detalhes importantes do comportamento

### Preserva rota e query

* Se você chamar: `http://localhost:52714/a/b?c=1`
* O proxy envia para: `https://api-teste-teste.com.br/a/b?c=1`

### Ajuste de Host

O `host` do header é forçado para o host do backend:

```js
host: targetUrl.hostname
```

Isso evita backend rejeitando request por `Host` errado.

### Erros

Se o backend cair/recusar conexão, você recebe:

* Status: `502`
* Body:

```json
{ "error": "Bad Gateway", "detail": "mensagem do erro" }
```

---

## Limitações (do jeito que está)

* **Sem suporte explícito a WebSocket**
* Não tem timeout configurado (depende do padrão do Node)
* Não tem logs detalhados por request (só loga erro)
* HTTPS é usado apenas **do proxy para o alvo**; local continua HTTP
* Não faz reescrita avançada de headers (além do `host`)

---

## Segurança / Boas práticas

* Evite expor isso na internet (ideal é usar só local / rede controlada)
* Se for usar em time/ambiente compartilhado, considere:

  * allowlist de hosts
  * autenticação
  * rate limit
  * logs e métricas
  * timeouts e limites de tamanho de body

---

## Troubleshooting rápido

**502 Bad Gateway**

* API alvo fora do ar
* DNS/host errado
* bloqueio de rede/firewall
* endpoint exige TLS/cert específico (casos raros)

**Backend responde diferente do esperado**

* confira se o `path` está sendo montado como você quer
* se o alvo tem um `pathname` diferente de `/`, ele entra como base

---

## Licença

Defina a sua (ex.: MIT).

```

Se você quiser, eu também adapto o README com:
- nome oficial do projeto
- comandos `npm init` + `npm start`
- config via `.env` (PORTA=ALVO)
- logs por request (método, url, tempo, status)
- timeout / keep-alive / limite de payload
```

# Profeto JS

```js
// proxy.js
const http = require('http');
const https = require('https');
const { URL } = require('url');

function createProxy(localPort, target) {
  const targetUrl = new URL(target);
  const isHttps = targetUrl.protocol === 'https:';
  const client = isHttps ? https : http;
  const defaultPort = isHttps ? 443 : 80;

  const server = http.createServer((req, res) => {
    // Monta o path final (preserva rota e query)
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
        host: targetUrl.hostname, // Host correto pro backend
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

    // Encaminha o body da requisição original
    req.pipe(proxyReq, { end: true });
  });

  server.listen(localPort, () => {
    console.log(`Proxy ouvindo em http://localhost:${localPort} -> ${target}`);
  });
}

// 8080 -> api-teste-hml
createProxy(52714, 'https://api-teste-teste.teste1.com.br');

// 8081 -> api-teste-hml
createProxy(58889, 'https://api-teste-teste.teste.com.br');


```
