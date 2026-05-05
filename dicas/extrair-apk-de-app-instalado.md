# Extrair APK de app instalado (Android)

> **Objetivo:** terminar com um `.apk` (ou conjunto `base.apk + splits`) na sua máquina, pronto pra abrir num decompiler.
> **Plataforma:** Android
> **Tempo estimado:** 5 min
> **Pré-requisitos:** ADB instalado e device/emulador autorizado.

## Quando usar este fluxo

- Você quer auditar um app que **já está instalado** num device/emulador (seu, de teste, ou de análise).
- Você precisa do APK pra rodar [JADX](../ferramentas/engenharia-reversa/jadx.md), [dex2jar](../ferramentas/engenharia-reversa/dex2jar.md) ou apktool.
- Não tem o `.apk` salvo em outro lugar.

> **Não usar** pra apps que você não tem direito de auditar. Engenharia reversa é pra estudo, malware analysis ou auditoria legítima.

## Ferramentas envolvidas

- [adb](../ferramentas/dispositivo-adb/adb.md) — listar pacotes, achar caminho, puxar arquivos.
- [JADX](../ferramentas/engenharia-reversa/jadx.md) — decompilar (recomendado).
- [dex2jar](../ferramentas/engenharia-reversa/dex2jar.md) + [JD-GUI](../ferramentas/engenharia-reversa/jd-gui.md) — caminho alternativo.

## Passo a passo

### 1. Confirmar que o device está conectado

```bash
adb devices -l
```

Deve mostrar pelo menos um device em estado `device`.

### 2. Achar o nome do pacote

Se você sabe parte do nome:

```bash
adb shell pm list packages | findstr <fragmento>
```

Sem filtro:

```bash
adb shell pm list packages
```

Vai listar `package:com.exemplo.app`. Anote.

### 3. Listar os caminhos do(s) APK(s)

```bash
adb shell pm path com.exemplo.app
```

Saída típica em apps modernos (App Bundle / split APKs):

```
package:/data/app/~~XYZ==/com.exemplo.app-ABC==/base.apk
package:/data/app/~~XYZ==/com.exemplo.app-ABC==/split_PlayAssetPack.apk
package:/data/app/~~XYZ==/com.exemplo.app-ABC==/split_config.arm64_v8a.apk
package:/data/app/~~XYZ==/com.exemplo.app-ABC==/split_config.pt.apk
```

### 4. Puxar todos os APKs em um comando (PowerShell)

```powershell
$serial = "<SERIAL_DO_DEVICE>"   # ex: 4AD01LH0H, ou "emulator-5554"
$pkg    = "com.exemplo.app"

# Pega todos os caminhos e remove o prefixo "package:"
$paths = (adb -s $serial shell pm path $pkg) -replace '^package:',''

# Cria pasta destino
New-Item -ItemType Directory -Force -Path ".\$pkg" | Out-Null

# Baixa cada APK
foreach ($p in $paths) {
  adb -s $serial pull $p ".\$pkg\"
}
```

No final, a pasta `.\com.exemplo.app\` tem `base.apk` + todos os splits.

### 5. Decompilar

**Caminho recomendado — JADX:**

```bash
jadx.bat -d output base.apk split_PlayAssetPack.apk split_config.arm64_v8a.apk
```

JADX entende vários APKs no mesmo job e produz pasta `output/sources/`.

**Caminho alternativo — dex2jar + JD-GUI:**

```bash
d2j-dex2jar.bat base.apk
```

Abre `base-dex2jar.jar` no [JD-GUI](../ferramentas/engenharia-reversa/jd-gui.md).

## Validação

- A pasta `output/` (JADX) ou `base-dex2jar.jar` (dex2jar) existe e tem tamanho > 0.
- No JADX-GUI, a árvore esquerda mostra pacotes (`com.exemplo.app.*`) navegáveis.
- `AndroidManifest.xml` reconstruído está visível e legível.

## Troubleshooting

**`pm path` retorna vazio**
Pacote errado, ou app desinstalado. Confirme com `pm list packages | findstr <fragmento>`.

**`adb pull` falha com "Permission denied"**
APKs em `/data/app/...` normalmente são lidos sem root. Se cair em `/data/data/<pacote>/...`, aí precisa de root ou `run-as <pacote>` (só funciona se o app for **debuggable**).

**Reinstalar em outro device (mesma arquitetura)**

```powershell
adb -s <NOVO_SERIAL> install-multiple -r base.apk split_*.apk
```

**App é Unity / Flutter / React Native**
Você consegue o APK e os assets, mas **não** recupera o "projeto editável" original. Inspeção é possível, edição vai ter limites.

## Ver também

- [Fluxo: capturar tráfego HTTPS](capturar-trafego-https-android.md) — quando o objetivo é olhar a rede do app, não o código.
- [adb](../ferramentas/dispositivo-adb/adb.md)
- [JADX](../ferramentas/engenharia-reversa/jadx.md)
