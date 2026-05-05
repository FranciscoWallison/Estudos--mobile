# Performance

Ferramentas pra **medir e diagnosticar performance** de apps mobile: CPU, memória, FPS, rede, startup, render. Diferente de Engenharia Reversa (que olha código) e de Analytics (que olha eventos de negócio).

## Ferramentas

| Ferramenta | Quando usar | Link |
| ---------- | ----------- | ---- |
| Perfetto | Trace detalhado de sistema + app (jank, startup, scheduling). Substitui Systrace. | [perfetto.md](perfetto.md) |

## Roadmap (a adicionar)

- **Android Studio Profiler** — CPU / Memory / Network / Energy direto na IDE.
- **Layout Inspector** — inspecionar hierarquia de Views/Compose em tempo real.
- **Flipper** (Meta) — inspector de rede + DB + layout pra apps RN/nativos.
- **GPU Profiling** (Android Studio + ferramentas Snapdragon).
- **Macrobenchmark / Microbenchmark** — Jetpack libs pra medir startup e UI jank em CI.
- **simpleperf** — perfilador de baixo nível com sampling em CPU.

## Quando usar qual

- **Vazamento de memória / OOM:** Android Studio Profiler (Memory) — a documentar.
- **Jank / animação travando:** [Perfetto](perfetto.md) + GPU Profiling.
- **Startup lento:** [Perfetto](perfetto.md) (`startup-trace`) + Macrobenchmark.
- **Hierarquia de View pesada:** Layout Inspector — a documentar.
