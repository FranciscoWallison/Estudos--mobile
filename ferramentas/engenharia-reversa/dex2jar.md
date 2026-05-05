# dex2jar (d2j-dex2jar)

> **Categoria:** Engenharia Reversa
> **Plataforma:** Android (executado no PC)
> **Quando usar:** converter `.dex` → `.jar` quando você precisa do bytecode Java pra abrir num decompiler de `.class` (JD-GUI, CFR, Fernflower).
> **Alternativas:** [JADX](jadx.md) — abre APK direto e já decompila pra Java/Kotlin.

## O que faz

Apps Android (APK) têm bytecode em **`.dex`** (ex: `classes.dex`), não em `.class`. O **dex2jar** converte um `.dex` para um `.jar` com bytecode Java (`.class`). Daí você abre esse `.jar` num decompiler Java.

## `dex2jar` vs `d2j-dex2jar`

- **`d2j-dex2jar`** = nome moderno, dentro do pacote **dex-tools (d2j)**. Script `.bat` no Windows.
- **`dex2jar`** = nome antigo / alias / atalho dependendo da instalação.

Mesma coisa na prática.

### Entrada e saída

- **Entrada:** `classes.dex` (ou `.apk`, dependendo do wrapper).
- **Saída:** `algo.jar` (normalmente `*-dex2jar.jar`).

## Instalação (Windows)

1. Baixe o pacote **dex-tools** nas Releases do projeto `pxb1988/dex2jar` no GitHub.
2. Extraia em qualquer pasta (ex: `C:\tools\dex-tools`).
3. Adicione essa pasta ao **PATH** ou rode os scripts `.bat` por caminho completo.

## Como usar

```bat
d2j-dex2jar.bat base.apk
```

Resultado: `base-dex2jar.jar`. Abra no [JD-GUI](jd-gui.md).

## Dicas e pegadinhas

- O `.jar` gerado **não recria o código fonte original**. É só bytecode Java; quem reconstrói o código é o decompiler que você usar depois.
- **Obfuscação** (ProGuard/R8), **Kotlin**, **reflection** e **dynamic loading** deixam o resultado bem mais feio.
- **Multi-dex** (`classes2.dex`, `classes3.dex`): pode exigir converter mais de um `.dex`.
- Apps com DEX malformado (comum em malware) podem fazer a conversão **falhar silenciosamente** — confira se o `.jar` saiu com tamanho razoável.
- Hoje em dia, o [JADX](jadx.md) costuma ser mais prático: abre o APK direto, pula o passo do `.jar`.

## Ver também

- [JADX](jadx.md) — alternativa moderna ao fluxo `dex2jar + JD-GUI`.
- [JD-GUI](jd-gui.md) — visualizador para o `.jar` gerado.
- apktool *(a documentar)* — foca em recursos + smali (bom quando decompilação alto nível fica ruim).
- [Fluxo: extrair APK de app instalado](../../dicas/extrair-apk-de-app-instalado.md) — o "como pegar o APK" mora lá agora.
