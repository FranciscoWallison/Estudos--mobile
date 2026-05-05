# Perfetto

> **Categoria:** Performance
> **Plataforma:** Android (também Linux, Chrome)
> **Quando usar:** capturar trace detalhado do sistema + app (CPU scheduling, frames, binder, locks, I/O, memória) pra investigar **jank**, **startup lento** ou **consumo anômalo**.
> **Alternativas:** Android Studio Profiler (mais simples, menos detalhe), simpleperf (sampling de CPU low-level), systrace (legado, substituído pelo Perfetto).

## O que faz

Sucessor oficial do Systrace. Captura **tracing de sistema** unificado (kernel + Android framework + app) e dá um viewer web (`ui.perfetto.dev`) pra explorar timeline com zoom infinito, queries SQL sobre o trace, e flame graphs.

- Trace de **threads, scheduling, frames, vsync**.
- **Atrace tags** do framework: `view`, `am`, `wm`, `hal`, `gfx`, `input`, `binder_driver`...
- Eventos custom do app via `Trace.beginSection()` / `Trace.endSection()`.
- **Memory tracing** (heap profiler, com símbolos).
- **CPU profiling** com `perf`/simpleperf integrado.

## Instalação (Windows)

Não precisa instalar nada no PC pra **capturar** — usa ADB.
Pra **visualizar**, abra no navegador: <https://ui.perfetto.dev>.

Pra capturas avançadas via CLI no PC:

```powershell
# Baixe o binário "trace_processor" e "perfetto" da release oficial
# (https://github.com/google/perfetto/releases)
```

## Como usar

### Caminho 1 — Captura via UI Perfetto

1. Abra <https://ui.perfetto.dev>.
2. **Record new trace** → escolha **Android device (via USB)**.
3. Selecione probes: **Scheduling, Frames, Atrace, App-defined**.
4. Em **Atrace tags**, marque pelo menos: `am`, `gfx`, `view`, `wm`.
5. Em **Atrace apps**, adicione `com.exemplo.app` pra ver `Trace.beginSection` do seu app.
6. **Start recording** (10-30s pra startup; mais pra jank prolongado).
7. Reproduza o cenário no app.
8. **Stop**. Trace abre direto na UI.

### Caminho 2 — Captura via ADB direto

```bash
adb shell perfetto -o /data/misc/perfetto-traces/trace.perfetto-trace -t 10s \
  -c - --txt <<EOF
buffers: { size_kb: 65536 fill_policy: DISCARD }
data_sources: { config { name: "linux.process_stats" } }
data_sources: { config { name: "linux.ftrace" ftrace_config { ftrace_events: "sched/sched_switch" atrace_categories: "view" atrace_categories: "am" atrace_apps: "com.exemplo.app" } } }
duration_ms: 10000
EOF

adb pull /data/misc/perfetto-traces/trace.perfetto-trace .
```

Abre o `.perfetto-trace` em <https://ui.perfetto.dev>.

### Caminho 3 — Trace de startup automatizado

```bash
adb shell am start -W -n com.exemplo.app/.MainActivity
```

(O `-W` retorna `TotalTime` do startup. Pra detalhe, capture trace começando antes do `am start`.)

### Marcar seções do app

```kotlin
import android.os.Trace

Trace.beginSection("MinhaTela.carregarDados")
try {
    carregarDados()
} finally {
    Trace.endSection()
}
```

A seção aparece como bloco nomeado na timeline da thread.

## Dicas e pegadinhas

- **Trace grande (>50MB) trava a UI.** Reduza duração ou desabilite probes pesados (`linux.ftrace` com muitos eventos).
- Em **release builds**, o R8 pode remover chamadas `Trace.beginSection` se não estiverem em allowlist do ProGuard.
- Pra investigar **jank**, ative **Frames** + **Atrace `gfx, view`** + use a aba **Slices → "expected vs actual"** na UI.
- A UI tem **SQL** (botão `Query`) pra rodar consultas tipo "qual thread mais consumiu CPU entre 5s e 7s". É o jeito mais rápido de achar culpado em traces grandes.
- `traceconv` (CLI) converte trace pra JSON pra usar em ferramentas terceiras.

## Ver também

- [adb](../dispositivo-adb/adb.md) — base pra captura.
- Doc oficial: <https://perfetto.dev/docs/>
- UI: <https://ui.perfetto.dev>
