
# Documentação — Proxy local via USB (ADB Reverse) para testar API

## Objetivo

Rodar um **proxy no PC** que encaminha requisições HTTP para a API HTTPS de teste, e fazer o **app no Android** acessar esse proxy usando **USB**, sem depender de Wi-Fi.

Fluxo final:

```
APP (Android) -> http://localhost:8092 -> ADB reverse (USB) -> PC:52714 -> Proxy Node -> https://api-osvendas-teste.odontosystem.com.br
```

---

## Requisitos

* Android com **Depuração USB** ativada
* `adb` instalado (Android Platform Tools)
* Node.js instalado no PC
* Cabo USB decente (isso afeta MUITO a estabilidade)

---

## Portas usadas (exemplo do seu caso)

* **No app (Android):** `localhost:8092`
* **No PC (Proxy Node):** `localhost:52714`
* **Target (API real):** `https://api-osvendas-teste.odontosystem.com.br`

Você fez o mapeamento:

* `tcp:8092` (no Android) → `tcp:52714` (no PC)

---

## 1) Rodar o proxy no PC

### `proxy.js` (ideia)

Ele cria um server HTTP local que encaminha tudo para a API HTTPS.

Você roda assim:

```bash
node proxy.js
```

E deve aparecer algo como:

```
Proxy ouvindo em http://localhost:52714 -> https://api-osvendas-teste.odontosystem.com.br
```

> Dica: com ADB reverse, **não precisa** abrir em `0.0.0.0`, `localhost` já resolve porque o tráfego “entra” localmente no PC.

---

## 2) Configurar o ADB Reverse (USB)

No PowerShell:

```powershell
adb devices
adb reverse --remove-all
adb reverse tcp:8092 tcp:52714
adb reverse --list
```

Saída esperada no `--list`:

```
UsbFfs tcp:8092 tcp:52714
```

### O que isso faz?

* Quando o app chama `http://localhost:8092` **no celular**,
* o ADB “intercepta” e manda para `http://127.0.0.1:52714` **no PC** (seu proxy).

---

## 3) Config do app (Dev)

Você precisa apontar o **Dev** pro endpoint que existe “no celular”, ou seja **`localhost:8092`** (porque é a porta que você expôs via reverse).

Exemplo:

```js
if (TYPE_SYSTEM == 'Dev') {
  urlApi = 'http://localhost:8092';
} else {
  urlApi = 'https://promotorapi.odontosystem.com.br';
}
```

Se o app concatena assim:
`api_url + app_path + '/' + app_vs`

então uma chamada vira:
`http://localhost:8092/api/v1/...`

---

## Checklist rápido (pra saber se tá tudo certo)

1. Proxy rodando?

* Você vê o log “Proxy ouvindo …:52714 …”
* Quando o app faz request, você vê logs/requests chegando.

2. Reverse ativo?

```powershell
adb reverse --list
```

Tem que aparecer `tcp:8092 tcp:52714`.

3. App no Dev aponta pra:

* `http://localhost:8092` (não é 52714, porque 52714 é no PC)

---

## Problemas comuns (e como identificar)

### A) “Cai direto” / para de funcionar do nada

Normalmente é **USB/ADB reiniciando**.

Sinais:

* `adb reverse --list` fica vazio
* `adb devices` some ou muda pra “unauthorized”

Solução rápida:

```powershell
adb reverse --remove-all
adb reverse tcp:8092 tcp:52714
```

### B) `ECONNRESET` / `socket hang up`

Rede USB instável OU seu proxy não está lidando bem com conexões abortadas.

* Melhora com cabo/porta melhores
* (Opcional) usar a versão “casca grossa” do proxy com keep-alive/timeouts

### C) App não pega “localhost”

Em Android, sem `adb reverse`, `localhost` aponta pro próprio celular.
Com reverse ligado, funciona.

---

## Script PowerShell pronto (setup 1 comando)

Crie um `setup-proxy.ps1`:

```powershell
adb devices
adb reverse --remove-all
adb reverse tcp:8092 tcp:52714
adb reverse --list
```

---

## Observações importantes

* Isso é **Android** (ADB). Em **iOS** não funciona assim (aí seria outro esquema).
* Se você usar outra porta no app, tem que ajustar o reverse:

  * app = 9000 → proxy = 52714:

    ```powershell
    adb reverse tcp:9000 tcp:52714
    ```

---






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

