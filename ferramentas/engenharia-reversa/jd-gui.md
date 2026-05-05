**JD-GUI** é um **decompilador com interface gráfica**: ele abre arquivos **`.class`** e pacotes tipo **`.jar/.war/.ear/.zip`** e mostra uma versão “reconstruída” do **código Java** pra você navegar.

## Como usar no Windows (passo a passo)

### 1) Baixar

Você pega o **zip do Windows** nas *Releases* do projeto (ele já vem pronto pra rodar). 
Também dá pra baixar pelo site do projeto.

**Atalho:** se você usa Chocolatey:

```bat
choco install javadecompiler-gui
```

([Chocolatey Software][4])

### 2) Rodar

* Extraia o zip.
* Abra **`jd-gui.exe`** (quando vem no pacote do Windows).
* Se você só tiver o `.jar`, roda assim:

```bat
java -jar jd-gui.jar
```

> Se der erro de Java, normalmente é porque precisa de **Java 8+** instalado. ([GitHub][5])

### 3) Abrir o arquivo pra decompilar

No JD-GUI:

* **File → Open File…** e selecione o **`.jar`** (ou `.class`, `.war`, `.ear`, `.zip` etc.). 
* Ou só **arrasta e solta** o arquivo na janela. 

A árvore da esquerda mostra **pacotes/classes**; clicou numa classe, ele mostra o “Java” no painel principal.

### 4) Exportar tudo pra código fonte

Pra salvar tudo que ele decompilou:

* **File → Save All Sources**
  Ele exporta as fontes decompiladas (geralmente num **ZIP**). ([Stack Overflow][6])

## Onde ele entra no seu fluxo do Android

O JD-GUI **não abre `.dex`** direito “puro” — o uso comum é:
**dex2jar / d2j-dex2jar → gera `.jar` → abre no JD-GUI**. ([GitHub][1])

## Limite importante

O que aparece no JD-GUI **não é o fonte original**, é uma reconstrução a partir do bytecode. Em app ofuscado, Kotlin, etc., vai ficar bem mais feio.