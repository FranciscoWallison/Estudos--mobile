# JD-GUI

> **Categoria:** Engenharia Reversa
> **Plataforma:** Android/Java (analisado no PC)
> **Quando usar:** abrir um `.jar` (gerado por dex2jar) e navegar visualmente pelas classes Java reconstruídas.
> **Alternativas:** [JADX](jadx.md) (mais moderno, abre APK direto).

## O que faz

Decompilador com interface gráfica que abre **`.class`** e pacotes (**`.jar/.war/.ear/.zip`**) e mostra uma versão reconstruída do **código Java** pra você navegar.

> Não abre `.dex` diretamente. Pra Android, o fluxo clássico é: `dex2jar gera .jar → JD-GUI abre`.

## Instalação (Windows)

Baixe o **zip do Windows** nas *Releases* do projeto JD-GUI no GitHub. Extraia e abra **`jd-gui.exe`**.

Se você só tiver o `.jar` da ferramenta:

```bat
java -jar jd-gui.jar
```

> Se der erro de Java, instale **Java 8+**.

Atalho via Chocolatey:

```bat
choco install javadecompiler-gui
```

## Como usar

1. **File → Open File…** e selecione o **`.jar`** (ou `.class`, `.war`, `.ear`, `.zip`).
   Ou só **arrasta e solta** o arquivo na janela.
2. Árvore da esquerda mostra **pacotes/classes**; clicou numa classe, ele mostra o "Java" no painel principal.
3. Pra exportar tudo: **File → Save All Sources** (gera um ZIP com as fontes decompiladas).

## Dicas e pegadinhas

- O que aparece **não é o fonte original** — é uma reconstrução a partir do bytecode. Em app ofuscado ou Kotlin, vai ficar bem mais feio.
- Pra Android, considere ir direto no [JADX](jadx.md): ele abre APK sem o passo `dex2jar`.
- Útil quando o JADX trava ou produz código pior que o JD-GUI naquele caso específico (acontece).

## Ver também

- [dex2jar](dex2jar.md) — gera o `.jar` que o JD-GUI consome.
- [JADX](jadx.md) — alternativa moderna que pula o JD-GUI no fluxo Android.
- [Fluxo: extrair APK de app instalado](../../dicas/extrair-apk-de-app-instalado.md)
