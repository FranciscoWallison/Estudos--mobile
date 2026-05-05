# JADX

> **Categoria:** Engenharia Reversa
> **Plataforma:** Android (analisado no PC)
> **Quando usar:** abrir APK/DEX e ler o código Java reconstruído sem precisar de etapas extras.
> **Alternativas:** [dex2jar + JD-GUI](dex2jar.md) (fluxo antigo, em duas etapas).

## O que faz

Decompilador "DEX → Java" que abre **APK/DEX** direto e tenta reconstruir o código Java, além de decodificar recursos do app. Vem em duas formas: **CLI (`jadx`)** e **GUI (`jadx-gui`)**.

- Decompila bytecode Dalvik/ART pra **código Java** a partir de **APK, DEX, AAR, AAB, ZIP** etc.
- Decodifica **AndroidManifest.xml**, recursos de **resources.arsc** e tem **deobfuscator**.
- GUI tem **busca**, **pular pra declaração**, **achar usos**.
- Nem sempre sai 100% perfeito (vai ter erro/trecho feio em vários apps).

## Instalação (Windows — GUI, jeito mais fácil)

1. Baixe no GitHub Releases o pacote **`jadx-gui-*-with-jre-win.zip`** (já vem com Java embutido).
   - Se baixar o "sem JRE", precisa de **Java 11+ 64-bit** instalado.
2. Extraia o zip.
3. Entre na pasta `bin` e rode **`jadx-gui.bat`** (duplo clique funciona).

## Como usar

### GUI

No JADX-GUI, abra o **APK/DEX** e navegue pelo código. Use:

- busca geral
- "go to declaration"
- "find usage"

### CLI (exportar tudo)

```bat
jadx.bat -d out pasta\app.apk
```

Opções úteis:

- `-d, --output-dir` — pasta de saída
- `-ds` — só fontes
- `-dr` — só recursos
- `-r` — não decodificar recursos
- `-s` — não gerar fontes
- `-e, --export-gradle` — exporta como projeto Gradle

## Dicas e pegadinhas

- Em apps **muito ofuscados** (ProGuard/R8) ou **Kotlin pesado**, o resultado fica feio — não é falha do JADX, é o estado do bytecode.
- Pra **APK com splits** (App Bundle), passe `base.apk` + os splits relevantes na linha de comando. JADX entende múltiplos APKs num mesmo job.
- "Find usages" na GUI é o atalho mais útil pra rastrear chamadas a um método suspeito.

## Ver também

- [dex2jar](dex2jar.md) — quando o JADX falha em algum DEX, dá pra cair pro caminho `dex → jar → JD-GUI`.
- [JD-GUI](jd-gui.md) — visualizador de `.jar` (não abre `.dex` direto).
- [Fluxo: extrair APK de app instalado](../../dicas/extrair-apk-de-app-instalado.md)
