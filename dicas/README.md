# Dicas e fluxos

Guias **end-to-end** para tarefas comuns no dia a dia mobile. Cada fluxo combina várias ferramentas e foca no resultado, não nas ferramentas em si.

> **Filosofia:** ferramenta descreve a ferramenta; fluxo descreve a tarefa. Se um conteúdo está duplicado, mantém aqui.

## Fluxos disponíveis

| Fluxo | O que entrega | Link |
| ----- | ------------- | ---- |
| Extrair APK de app instalado | `.apk` (e splits) na sua máquina, pronto pra decompilar. | [extrair-apk-de-app-instalado.md](extrair-apk-de-app-instalado.md) |
| Capturar tráfego HTTPS no Android | Requests HTTP/HTTPS do app visíveis em proxy de inspeção ou proxy local. | [capturar-trafego-https-android.md](capturar-trafego-https-android.md) |
| Validar evento Google Analytics | Confirmação em logcat (e DebugView, se GA4) de que o evento sai com nome e parâmetros corretos. | [validar-evento-google-analytics.md](validar-evento-google-analytics.md) |

## Roadmap (próximos fluxos)

- **Bypass de SSL pinning com Objection** — quando o caminho A do fluxo de tráfego HTTPS não resolve.
- **Patchear APK e reinstalar** — apktool + uber-apk-signer + adb install.
- **Instrumentar função com Frida** — hook básico de método Java.
- **Investigar startup lento** — Android Studio Profiler + Macrobenchmark.
- **Validar evento iOS** — equivalente do GA debugger no iOS (Xcode console + Charles).
- **Comparar comportamento Dev vs Prod** — usar Multi Reverse Proxy + analytics simultaneamente.

## Como adicionar um fluxo

1. Copie [modelos/fluxo-template.md](../modelos/fluxo-template.md) pra `dicas/<verbo-tarefa>.md`.
2. Preencha. Cada passo tem comando concreto + o que conferir.
3. Adicione linha aqui na tabela acima.
4. Em cada ferramenta envolvida, adicione o link na seção "Ver também".

Detalhes em [CONVENCOES.md](../CONVENCOES.md).
