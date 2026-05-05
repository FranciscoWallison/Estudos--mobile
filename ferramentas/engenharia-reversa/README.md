# Engenharia Reversa

Ferramentas pra abrir um APK/DEX e entender o que ele faz: decompilar bytecode, decodificar resources, navegar por classes.

## Ferramentas

| Ferramenta | Quando usar | Link |
| ---------- | ----------- | ---- |
| JADX | Padrão de hoje. Abre APK direto e gera Java/Kotlin. | [jadx.md](jadx.md) |
| dex2jar (d2j-dex2jar) | Converter `.dex` em `.jar` quando JADX falha ou pra usar outro decompiler. | [dex2jar.md](dex2jar.md) |
| JD-GUI | Visualizar `.jar` (gerado por dex2jar) com GUI. | [jd-gui.md](jd-gui.md) |

## Ordem sugerida de aprendizado

1. **JADX** — basta saber isso pra 90% dos casos.
2. **dex2jar + JD-GUI** — entenda como caminho de fallback ou pra projetos antigos.
3. (Próximos) apktool, smali — pra recompilar APK modificado.

## Roadmap (a adicionar)

- apktool — resources + smali, recompilar APK.
- CFR / Fernflower — decompilers Java alternativos pra cruzar saídas.
- bytecode-viewer — interface multi-decompiler.

## Fluxos relacionados

- [Extrair APK de app instalado](../../dicas/extrair-apk-de-app-instalado.md)
