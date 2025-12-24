**JADX** é um **decompilador “DEX → Java”** que já abre **APK/DEX** direto e tenta reconstruir o código Java, além de decodificar recursos do app. Ele vem em duas formas: **CLI (`jadx`)** e **GUI (`jadx-gui`)**. ([GitHub][1])

## O que ele faz

* Decompila bytecode Dalvik/ART pra **código Java** a partir de **APK, DEX, AAR, AAB, ZIP** etc. ([GitHub][1])
* Decodifica **AndroidManifest.xml** e recursos de **resources.arsc** e tem **deobfuscator**. ([GitHub][1])
* Na GUI você tem **busca**, **pular pra declaração**, **achar usos** e mais. ([GitHub][1])
* Nem sempre sai 100% perfeito (vai ter erro/trecho feio em vários apps). ([GitHub][1])

## Como usar no Windows (jeito mais fácil: GUI)

1. Baixe no GitHub Releases o pacote **`jadx-gui-*-with-jre-win.zip`** (já vem com Java embutido). ([GitHub][2])

   * Se baixar o “sem JRE”, aí você precisa ter **Java 11+ 64-bit** instalado. ([GitHub][1])

2. Extraia o zip.

3. Entre na pasta `bin` e rode **`jadx-gui.bat`** (no Windows dá pra abrir com duplo clique). ([GitHub][1])

4. No JADX-GUI, abra o **APK/DEX** e navegue pelo código. Use:

* busca geral,
* “go to declaration”,
* “find usage”. ([GitHub][1])

## Como usar no Windows (CLI, pra exportar tudo)

Depois de extrair, rode pelo `.bat` também. Sintaxe geral: ([GitHub][1])

```bat
jadx.bat -d out pasta\app.apk
```

Opções úteis:

* `-d, --output-dir` (pasta de saída)
* `-ds` (só fontes)
* `-dr` (só recursos)
* `-r` (não decodificar recursos)
* `-s` (não gerar fontes) ([GitHub][1])

E dá pra exportar como projeto Gradle:

* `-e, --export-gradle` ([GitHub][1])
