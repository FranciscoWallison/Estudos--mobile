# Analytics / Debug

Ferramentas pra **inspecionar eventos de analytics** disparados por um app (ou site) e validar tracking sem esperar relatório.

## Ferramentas

| Ferramenta | Quando usar | Link |
| ---------- | ----------- | ---- |
| Google Analytics Debugger | Validar hits GA Universal e GA4/Firebase em tempo real (Chrome + Android via setprop). | [google-analytics-debugger.md](google-analytics-debugger.md) |

## Roadmap (a adicionar)

- **Firebase DebugView** — visualização web dos eventos GA4 disparados pelo app (já citado no GA Debugger, vale ter doc própria).
- **GTM Preview / Tag Assistant Companion** — debug de tags via Google Tag Manager.
- **Charles map remote** — manipular response de hit pra simular ambiente.
- **Bugsnag / Sentry mobile** — quando "validar evento" vira "validar crash".

## Quando usar qual

- **GA Universal / GA4 no app Android:** GA Debugger via ADB.
- **GA4 mais visual com console web:** GA Debugger + Firebase DebugView.
- **GTM (web):** GTM Preview Mode + Tag Assistant.

## Fluxos relacionados

- [Validar evento Google Analytics](../../dicas/validar-evento-google-analytics.md)
