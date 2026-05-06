# Estudos mobile

Base de conhecimento em PT-BR sobre **ferramentas e fluxos** pra estudar e trabalhar com mobile — análise de apps, debug, engenharia reversa, performance e analytics.

Cada **ferramenta** tem seu próprio MD descrevendo o que é, como instalar e como usar.
Cada **fluxo** em [dicas/](dicas/) é um passo-a-passo end-to-end pra uma tarefa concreta, combinando ferramentas.

> **Procurando algo específico?** Veja o [SUMARIO.md](SUMARIO.md) — índice plano A-Z + "pelo problema que você está resolvendo".

---

## Categorias de ferramentas

| Categoria | O que entra aqui | Link |
| --------- | ---------------- | ---- |
| Engenharia Reversa | Decompilar APK/DEX, ler bytecode, navegar por classes Java/Kotlin. | [ferramentas/engenharia-reversa/](ferramentas/engenharia-reversa/README.md) |
| Dispositivo / ADB | Controlar device físico ou emulador pelo PC: ADB, emulator, proxy global. | [ferramentas/dispositivo-adb/](ferramentas/dispositivo-adb/README.md) |
| Rede / Proxy | Redirecionar ou inspecionar tráfego de rede dos apps. | [ferramentas/rede-proxy/](ferramentas/rede-proxy/README.md) |
| Analytics / Debug | Inspecionar eventos de tracking (GA, Firebase, GTM) em tempo real. | [ferramentas/analytics-debug/](ferramentas/analytics-debug/README.md) |
| In-App Review | Popup de avaliação nativo dentro do app — guias por stack (nativo, RN, Flutter, Ionic). | [ferramentas/in-app-review/](ferramentas/in-app-review/README.md) |
| Performance | Medir CPU, memória, FPS, startup, jank. *(em construção)* | [ferramentas/performance/](ferramentas/performance/README.md) |
| Segurança / Pentest | Frida, Objection, MobSF, drozer, apktool. *(em construção)* | [ferramentas/seguranca-pentest/](ferramentas/seguranca-pentest/README.md) |
| iOS | Xcode tools, libimobiledevice, Proxyman. *(em construção)* | [ferramentas/ios/](ferramentas/ios/README.md) |

---

## Fluxos prontos

Tarefas concretas resolvidas passo-a-passo:

| Fluxo | O que entrega |
| ----- | ------------- |
| [Extrair APK de app instalado](dicas/extrair-apk-de-app-instalado.md) | `.apk` na sua máquina pra decompilar. |
| [Capturar tráfego HTTPS no Android](dicas/capturar-trafego-https-android.md) | Requests do app visíveis em proxy. |
| [Validar evento Google Analytics](dicas/validar-evento-google-analytics.md) | Confirmação que evento sai com nome/params certos. |

Índice completo em [dicas/](dicas/README.md).

---

## Como contribuir / adicionar conteúdo

- **Nova ferramenta:** copie [modelos/ferramenta-template.md](modelos/ferramenta-template.md) pra `ferramentas/<categoria>/<nome>.md`, preencha, adicione linha no README da categoria.
- **Novo fluxo:** copie [modelos/fluxo-template.md](modelos/fluxo-template.md) pra `dicas/<verbo-tarefa>.md`, preencha, adicione linha em [dicas/README.md](dicas/README.md).
- **Nova categoria:** crie a pasta + README, adicione linha aqui no índice de categorias.

Padrões (nomenclatura, cabeçalho, links): [CONVENCOES.md](CONVENCOES.md).

---

## Estrutura

```
.
├── README.md                  # você está aqui
├── CONVENCOES.md              # como manter consistente
├── ferramentas/
│   ├── engenharia-reversa/
│   ├── dispositivo-adb/
│   ├── rede-proxy/
│   ├── analytics-debug/
│   ├── in-app-review/
│   ├── performance/
│   ├── seguranca-pentest/
│   └── ios/                   # placeholder
├── dicas/                     # fluxos end-to-end
└── modelos/                   # templates copiáveis
```
