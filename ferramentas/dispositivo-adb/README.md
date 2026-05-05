# Dispositivo / ADB

Tudo que controla um device Android (físico ou emulador) pelo PC: comandos do `adb`, configuração de proxy global do device, gerenciamento de emuladores.

## Ferramentas

| Ferramenta | Quando usar | Link |
| ---------- | ----------- | ---- |
| ADB | Comando central. Instalar, desinstalar, logcat, pull, shell, port forward. | [adb.md](adb.md) |
| ADB — proxy global | Forçar todo o tráfego HTTP/HTTPS do device a passar por um proxy do PC. | [adb-proxy-global.md](adb-proxy-global.md) |
| Emulator (CLI) | Subir AVDs sem abrir o Android Studio inteiro. | [emulator.md](emulator.md) |

## Ordem sugerida de aprendizado

1. **ADB** — leia primeiro. É a base de tudo aqui.
2. **emulator** — só se você for trabalhar com AVDs.
3. **adb-proxy-global** — quando precisar interceptar tráfego.

## Roadmap (a adicionar)

- scrcpy — espelhar a tela do device no PC.
- frida-server via ADB — pra instrumentação dinâmica.
- run-as / shell tricks — coletar dados de apps debuggable sem root.

## Fluxos relacionados

- [Extrair APK de app instalado](../../dicas/extrair-apk-de-app-instalado.md)
- [Capturar tráfego HTTPS no Android](../../dicas/capturar-trafego-https-android.md)
- [Validar evento Google Analytics](../../dicas/validar-evento-google-analytics.md)
