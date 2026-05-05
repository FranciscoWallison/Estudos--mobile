# Validar evento Google Analytics em app Android

> **Objetivo:** confirmar que um evento de analytics (GA Universal ou GA4/Firebase) está sendo disparado com nome e parâmetros corretos quando o usuário interage com o app.
> **Plataforma:** Android
> **Tempo estimado:** 5-10 min
> **Pré-requisitos:** ADB autorizado; app instalado (idealmente build de Dev/debuggable).

## Quando usar este fluxo

- QA de tracking: confirmar se botão X dispara evento `click_y` com parâmetro `z`.
- Suspeita que evento não chega: ver se sai do cliente, se sai com payload certo.
- Auditoria pós-deploy: validar lote novo de eventos antes de subir pra produção.
- Comparar dois ambientes: Dev dispara X, Prod dispara Y, é esperado?

## Ferramentas envolvidas

- [adb](../ferramentas/dispositivo-adb/adb.md)
- [Google Analytics Debugger](../ferramentas/analytics-debug/google-analytics-debugger.md) (a parte mobile, via setprop)
- Firebase DebugView (placeholder — adicionar quando entrar no projeto)

## Identifique qual GA o app usa

Antes de qualquer coisa: GA Universal e GA4 logam coisas diferentes.

| App usa…                          | Tag de logcat                       | Validação visual extra        |
| --------------------------------- | ----------------------------------- | ----------------------------- |
| **GA Universal** (Google Analytics SDK) | `GAv4`, `GAv4-SVC`            | —                             |
| **GA4 / Firebase Analytics**       | `FA`, `FA-SVC`                      | Firebase DebugView (web)      |

> Em dúvida, ative os dois — quem não tem dado, não polui.

## Passo a passo

### 1. Conecte e confirme o device

```bash
adb devices -l
```

### 2. Ative log verbose

**Para GA Universal:**

```bash
adb shell setprop log.tag.GAv4 VERBOSE
adb shell setprop log.tag.GAv4-SVC VERBOSE
```

**Para GA4 / Firebase:**

```bash
adb shell setprop log.tag.FA VERBOSE
adb shell setprop log.tag.FA-SVC VERBOSE
adb shell setprop debug.firebase.analytics.app com.exemplo.app
```

### 3. Limpe o logcat e comece a observar

```bash
adb logcat -c
adb logcat -v time -s GAv4 GAv4-SVC FA FA-SVC
```

(Ajuste as tags pro caso do seu app.)

### 4. Reproduza a ação no app

Abra o app, navegue até o ponto, clique no botão que deveria disparar o evento.

### 5. Leia o payload no logcat

Cada hit aparece com nome do evento e parâmetros. Em GA4 o formato é mais legível (`name=screen_view, params=Bundle[...]`).

Confirme:

- nome do evento bate com a especificação,
- parâmetros estão presentes,
- ordem de disparo (ex: `screen_view` antes de `click`).

### 6. (Opcional) Validação visual em GA4

Mantenha `debug.firebase.analytics.app` ativo e abra o **Firebase Console → Analytics → DebugView**. Eventos aparecem em ~30s.

### 7. Desative quando terminar

```bash
adb shell setprop log.tag.GAv4 ""
adb shell setprop log.tag.GAv4-SVC ""
adb shell setprop log.tag.FA ""
adb shell setprop log.tag.FA-SVC ""
adb shell setprop debug.firebase.analytics.app .none.
```

(Ou simplesmente reinicie o device — `setprop` não persiste no reboot.)

## Validação

- Logcat mostra a tag GA com o evento esperado pouco depois da ação.
- (GA4) DebugView mostra o evento em até 1 minuto.
- Parâmetros críticos não estão `null`/vazios.

## Troubleshooting

**Logcat fica vazio mesmo com setprop**

- App não é debuggable. Em release builds com R8/ProGuard agressivo, alguns logs são removidos. Use build de Dev/debug.
- Dispositivo precisa ter sido reiniciado depois de mudar o pacote? Não — `setprop` é runtime — mas o app precisa **detectar** a flag, o que muitos SDKs só fazem ao iniciar. Force-stop e reabra: `adb shell am force-stop com.exemplo.app`.

**Eventos disparam mas não chegam no Firebase DebugView**

- `debug.firebase.analytics.app` aponta pro pacote certo? Confira: `adb shell getprop debug.firebase.analytics.app`.
- Build com `FirebaseAnalytics.setAnalyticsCollectionEnabled(false)` em algum lugar.
- Ad blocker / VPN bloqueando saída pra `app-measurement.com`.

**Vários eventos iguais por click**

- Listener registrado várias vezes. Bug do app, não do tracking.

**Quer ver TUDO do app, não só GA**

```bash
adb logcat -v time | findstr com.exemplo.app
```

## Ver também

- [Google Analytics Debugger](../ferramentas/analytics-debug/google-analytics-debugger.md)
- [adb](../ferramentas/dispositivo-adb/adb.md)
- [Fluxo: capturar tráfego HTTPS](capturar-trafego-https-android.md) — pra ver o request HTTP/S de saída do hit.
