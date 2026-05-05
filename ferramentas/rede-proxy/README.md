# Rede / Proxy

Ferramentas pra **redirecionar** ou **inspecionar** o tráfego de rede de apps. Difere de "Dispositivo / ADB" por focar no proxy/servidor em si, não no device.

## Ferramentas

| Ferramenta | Quando usar | Link |
| ---------- | ----------- | ---- |
| Multi Reverse Proxy (Node) | App de Dev aponta pra `localhost:porta` e cai num backend HTTPS de teste, via cabo USB. | [multi-reverse-proxy.md](multi-reverse-proxy.md) |
| Charles Proxy | Inspeção HTTPS com GUI: timeline, breakpoints, map remote/local, repeat, throttle. | [charles.md](charles.md) |

## Roadmap (a adicionar)

- **mitmproxy** — proxy de inspeção em terminal, scriptável em Python.
- **Burp Suite (Community)** — proxy de inspeção HTTPS focado em segurança.
- **Fiddler Classic / Everywhere** — alternativa Microsoft.
- **Proxyman** — Mac (e plano Windows novo).

## Quando usar qual

- **Reescrever uma URL pra outro backend, sem ler o tráfego:** [Multi Reverse Proxy](multi-reverse-proxy.md) (code-only, sem GUI).
- **Ler payload HTTPS, modificar requests/responses, rastreio passo-a-passo:** [Charles](charles.md) / mitmproxy / Burp.
- **Bypass de SSL pinning:** apenas com [Objection](../seguranca-pentest/objection.md) ou [Frida](../seguranca-pentest/frida.md).

## Fluxos relacionados

- [Capturar tráfego HTTPS no Android](../../dicas/capturar-trafego-https-android.md)
