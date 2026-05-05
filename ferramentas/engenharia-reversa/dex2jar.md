### O que é o **dex2jar**

* Android apps (APK) têm bytecode em **.dex** (ex: `classes.dex`), não em `.class`.
* **dex2jar** pega um **.dex** e converte pra um **.jar** com **bytecode Java (.class)**.
* Aí você abre esse `.jar` num decompiler Java (CFR, Fernflower, JD-GUI, etc.) pra ver algo “parecido” com código Java.

### O que é **d2j-dex2jar**

* É basicamente **o mesmo conversor**, só que com o **nome “novo”** dentro do pacote **dex-tools (d2j)**.
* Em versões mais recentes, o comando principal virou `d2j-dex2jar` (script `.sh` no Linux/macOS e `.bat` no Windows).
* Em muita instalação antiga ou em alguns sistemas, ainda existe `dex2jar` como **alias/symlink** ou script antigo.

**Resumindo:**
✅ `d2j-dex2jar` = “dex2jar moderno” (mesma ideia, tooling atualizado)
✅ `dex2jar` = nome antigo / atalho / dependendo do pacote que você instalou

### Entrada e saída

* **Entrada:** `classes.dex` (ou até um `.apk`, dependendo do wrapper/versão que extrai o dex)
* **Saída:** `algo.jar` (normalmente `*-dex2jar.jar`)

### Pra que isso serve (na prática)

* Auditoria e engenharia reversa de apps (com permissão).
* Entender fluxo, chamadas, classes, APIs usadas.
* Análise de malware Android (também comum).

### Limitações que você vai ver sempre

* O `.jar` **não “recria o código fonte original”**. Ele cria bytecode e você decompila isso → sai um código aproximado.
* **Obfuscação** (ProGuard/R8), **Kotlin**, **reflection**, **dynamic loading**, etc. deixam o resultado bem mais feio.
* **Multi-dex** (`classes2.dex`, `classes3.dex`) pode exigir converter mais de um dex (dependendo do seu fluxo).
* Apps com DEX “quebrado”/malformado (comum em malware) podem fazer a conversão falhar.

### Alternativas que muita gente usa hoje

* **jadx**: abre o APK direto e já decompila pra Java/Kotlin de uma vez (bem prático).
* **apktool**: foca em recursos + smali (bom quando decompilação “alto nível” fica ruim).



Para extrair o APK de um app rodando em um emulador e analisar com dex2jar, você tem algumas opções:

## Extração do APK

**1. Usando ADB (Android Debug Bridge)**

Se o app já está instalado no emulador:

```bash
# Liste os pacotes instalados
adb shell pm list packages

# Encontre o caminho do APK
adb shell pm path com.exemplo.app

package:/data/app/com.exemplo.app-Nl2qVTwZE1pc_Tx-jsTeIA==/base.apk
package:/data/app/com.exemplo.app-Nl2qVTwZE1pc_Tx-jsTeIA==/split_PlayAssetPack.apk
package:/data/app/com.exemplo.app-Nl2qVTwZE1pc_Tx-jsTeIA==/split_PlayAssetPack.config.other_countries.apk
package:/data/app/com.exemplo.app-Nl2qVTwZE1pc_Tx-jsTeIA==/split_config.arm64_v8a.apk
```

**App Bundle** (multiple APKs). Isso é comum em apps modernos da Play Store. Veja como proceder:

## Análise da estrutura

- **base.apk** - APK principal com código e recursos básicos
- **split_PlayAssetPack.apk** - Assets adicionais do jogo/app
- **split_config.arm64_v8a.apk** - Bibliotecas nativas para arquitetura ARM64
- **split_*.config.other_countries.apk** - Recursos específicos de região

## Extração completa

```bash
# Extraia todos os APKs
adb pull /data/app/com.exemplo.app-Nl2qVTwZE1pc_Tx-jsTeIA==/base.apk
adb pull /data/app/com.exemplo.app-Nl2qVTwZE1pc_Tx-jsTeIA==/split_PlayAssetPack.apk
adb pull /data/app/com.exemplo.app-Nl2qVTwZE1pc_Tx-jsTeIA==/split_config.arm64_v8a.apk
adb pull /data/app/com.exemplo.app-Nl2qVTwZE1pc_Tx-jsTeIA==/split_PlayAssetPack.config.other_countries.apk

# Ou extraia tudo de uma vez
adb pull /data/app/com.exemplo.app-Nl2qVTwZE1pc_Tx-jsTeIA==/ ./app_extraido/
```

## Análise com dex2jar

Para analisar o código, foque no **base.apk**:

```bash
# Converta o base.apk
d2j-dex2jar.sh base.apk

# Analise com JD-GUI
jd-gui base-dex2jar.jar
```

## Reconstruir APK único (opcional)

Se precisar de um APK único para outras ferramentas:

```bash
# Use APKEditor ou SAI (Split APKs Installer)
# Ou combine manualmente com apktool:

apktool d base.apk -o base_decoded
apktool d split_config.arm64_v8a.apk -o split_decoded

# Mescle os recursos e reconstrua
```

## Recomendação para análise profunda

Use **jadx** que lida melhor com múltiplos APKs:

```bash
jadx base.apk split_PlayAssetPack.apk -d output_analise/
```

O **base.apk** contém todo o código Java/Kotlin principal. Os splits contêm principalmente recursos e bibliotecas nativas (.so files). Para engenharia reversa, o base.apk é seu alvo primário. Precisa analisar as bibliotecas nativas também?



**2. Diretamente do sistema de arquivos**

Se você tem acesso root no emulador:

```bash
adb shell
su
cd /data/app/
ls
# Copie o APK desejado
```

**3. Usando ferramentas de terceiros**

- **APK Extractor** (app para Android)
- **Apktool** para desempacotar
- **jadx** ou **jadx-gui** (alternativa moderna ao dex2jar)

## Análise com dex2jar

Após extrair o APK:

```bash
# Converta APK para JAR
d2j-dex2jar.sh app.apk

# O resultado será app-dex2jar.jar
# Depois use JD-GUI ou outra ferramenta para analisar
jd-gui app-dex2jar.jar
```

## Dica profissional

Considere usar **jadx** ao invés de dex2jar - é mais moderno e eficiente:

```bash
jadx app.apk -d output_dir
# ou use jadx-gui para interface gráfica
```



