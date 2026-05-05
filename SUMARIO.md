# Sumário

Índice plano e direto pra encontrar qualquer tema rápido. Ordem alfabética dentro de cada bloco.

> Pra navegação por categoria com descrição, veja o [README.md](README.md).
> Pra padrões de contribuição, veja [CONVENCOES.md](CONVENCOES.md).

---

## Por palavra-chave (A-Z)

- **ADB** → [adb.md](ferramentas/dispositivo-adb/adb.md)
- **Analytics (debug GA/Firebase)** → [google-analytics-debugger.md](ferramentas/analytics-debug/google-analytics-debugger.md)
- **APK — extrair de app instalado** → [extrair-apk-de-app-instalado.md](dicas/extrair-apk-de-app-instalado.md)
- **APK — decompilar** → [jadx.md](ferramentas/engenharia-reversa/jadx.md)
- **Charles Proxy** → [charles.md](ferramentas/rede-proxy/charles.md)
- **Cert (instalar no Android)** → [capturar-trafego-https-android.md](dicas/capturar-trafego-https-android.md#4-instale-o-certificado-do-proxy-no-android)
- **dex2jar** → [dex2jar.md](ferramentas/engenharia-reversa/dex2jar.md)
- **Eventos GA — validar** → [validar-evento-google-analytics.md](dicas/validar-evento-google-analytics.md)
- **Emulador Android (CLI)** → [emulator.md](ferramentas/dispositivo-adb/emulator.md)
- **Firebase Analytics — debug** → [google-analytics-debugger.md](ferramentas/analytics-debug/google-analytics-debugger.md#android--ga4--firebase-analytics)
- **Frida** → [frida.md](ferramentas/seguranca-pentest/frida.md)
- **HTTPS — capturar tráfego** → [capturar-trafego-https-android.md](dicas/capturar-trafego-https-android.md)
- **JADX** → [jadx.md](ferramentas/engenharia-reversa/jadx.md)
- **Jank / Performance — investigar** → [perfetto.md](ferramentas/performance/perfetto.md)
- **JD-GUI** → [jd-gui.md](ferramentas/engenharia-reversa/jd-gui.md)
- **Logcat** → [adb.md](ferramentas/dispositivo-adb/adb.md#coletar-logs-e-estado)
- **Multi Reverse Proxy** → [multi-reverse-proxy.md](ferramentas/rede-proxy/multi-reverse-proxy.md)
- **Objection** → [objection.md](ferramentas/seguranca-pentest/objection.md)
- **Perfetto / Trace** → [perfetto.md](ferramentas/performance/perfetto.md)
- **Proxy global no Android** → [adb-proxy-global.md](ferramentas/dispositivo-adb/adb-proxy-global.md)
- **Reverse (adb reverse)** → [multi-reverse-proxy.md](ferramentas/rede-proxy/multi-reverse-proxy.md) + [capturar-trafego-https-android.md](dicas/capturar-trafego-https-android.md#caminho-b--adb-reverse--proxy-local)
- **Root no emulador** → [emulator.md](ferramentas/dispositivo-adb/emulator.md#dicas-e-pegadinhas)
- **SSL pinning — bypass** → [objection.md](ferramentas/seguranca-pentest/objection.md#bypass-de-ssl-pinning)
- **Startup lento** → [perfetto.md](ferramentas/performance/perfetto.md#caminho-3--trace-de-startup-automatizado)

---

## Por categoria

### Engenharia Reversa
[Categoria →](ferramentas/engenharia-reversa/README.md)
- [JADX](ferramentas/engenharia-reversa/jadx.md)
- [JD-GUI](ferramentas/engenharia-reversa/jd-gui.md)
- [dex2jar](ferramentas/engenharia-reversa/dex2jar.md)

### Dispositivo / ADB
[Categoria →](ferramentas/dispositivo-adb/README.md)
- [ADB](ferramentas/dispositivo-adb/adb.md)
- [ADB — proxy global](ferramentas/dispositivo-adb/adb-proxy-global.md)
- [Emulator (CLI)](ferramentas/dispositivo-adb/emulator.md)

### Rede / Proxy
[Categoria →](ferramentas/rede-proxy/README.md)
- [Multi Reverse Proxy (Node)](ferramentas/rede-proxy/multi-reverse-proxy.md)
- [Charles Proxy](ferramentas/rede-proxy/charles.md)

### Analytics / Debug
[Categoria →](ferramentas/analytics-debug/README.md)
- [Google Analytics Debugger](ferramentas/analytics-debug/google-analytics-debugger.md)

### Performance
[Categoria →](ferramentas/performance/README.md)
- [Perfetto](ferramentas/performance/perfetto.md)

### Segurança / Pentest
[Categoria →](ferramentas/seguranca-pentest/README.md)
- [Frida](ferramentas/seguranca-pentest/frida.md)
- [Objection](ferramentas/seguranca-pentest/objection.md)

### iOS
[Categoria →](ferramentas/ios/README.md) *(em construção)*

---

## Fluxos (dicas/)

[Índice →](dicas/README.md)
- [Extrair APK de app instalado](dicas/extrair-apk-de-app-instalado.md)
- [Capturar tráfego HTTPS no Android](dicas/capturar-trafego-https-android.md)
- [Validar evento Google Analytics](dicas/validar-evento-google-analytics.md)

---

## Pelo problema que você está resolvendo

| Cenário | Por onde começar |
| ------- | ---------------- |
| "Quero entender o que esse app faz" | [Extrair APK](dicas/extrair-apk-de-app-instalado.md) → [JADX](ferramentas/engenharia-reversa/jadx.md) |
| "Quero ver as APIs que o app chama" | [Capturar tráfego HTTPS](dicas/capturar-trafego-https-android.md) → [Charles](ferramentas/rede-proxy/charles.md) |
| "Meu app de Dev tem que apontar pra outro backend" | [Multi Reverse Proxy](ferramentas/rede-proxy/multi-reverse-proxy.md) |
| "Esse evento de analytics está saindo certo?" | [Validar evento GA](dicas/validar-evento-google-analytics.md) |
| "App tem SSL pinning, proxy não pega" | [Objection](ferramentas/seguranca-pentest/objection.md) |
| "App está com jank / startup lento" | [Perfetto](ferramentas/performance/perfetto.md) |
| "Quero hookar uma função específica do app" | [Frida](ferramentas/seguranca-pentest/frida.md) |
| "Não sei nem rodar `adb devices`" | [ADB](ferramentas/dispositivo-adb/adb.md) |

---

## Modelos / Templates

- [Template de ferramenta](modelos/ferramenta-template.md)
- [Template de fluxo](modelos/fluxo-template.md)
