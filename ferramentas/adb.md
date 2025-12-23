
## O que é (na prática) ADB

**ADB = Android Debug Bridge**. É a “ponte” oficial pra você controlar um Android (emulador ou aparelho) pelo PC via **USB** ou **rede**. Ele faz isso com 3 peças:


* **Celular/tablet** com *Depuração USB* ligada
* **Emuladores Android** (Android Studio Emulator, Genymotion, alguns casos de MEmu/LDPlayer/BlueStacks se eles expõem ADB)

O comando direto pra ver o que está conectado é:

```bash
adb devices -l
```

Vai listar algo tipo:

* `device` = conectado e autorizado
* `unauthorized` = o aparelho não autorizou (precisa aceitar a janela de “Confiar neste computador?” no Android)
* `offline` = conexão ruim / travada

### Ver também emuladores

Se o emulador estiver rodando e tiver ADB ativo, ele aparece como:

* `emulator-5554` (comum no Android Studio)

Se você quer ver **mais detalhes** de cada um:

```bash
adb devices -l
adb -s <ID> shell getprop ro.product.model
adb -s <ID> shell getprop ro.build.version.release
```

### Se não aparece nada

1. Confere se o ADB está ok e reinicia o servidor:

```bash
adb kill-server
adb start-server
adb devices -l
```

2. Para **celular**: liga *Depuração USB* e aceita a autorização na tela.

3. Para **emulador**: alguns não expõem ADB por padrão. Se ele tiver ADB via TCP, dá pra conectar assim (exemplo):

```bash
adb connect 127.0.0.1:5555
adb devices -l
```

* **adb client** (o comando que você roda no PC)
* **adb server** (processo que fica escutando no PC)
* **adbd** (daemon no Android)

## Por que ele é importante (especialmente pra analisar app malicioso)

Sem ADB, você fica preso na UI do aparelho. Com ADB, você ganha **automação, visibilidade e controle**:

### 1) Instalar/rodar/forçar comportamento do app

* instalar/desinstalar sem tocar no celular
* abrir Activities/Intents
* limpar dados e “resetar” o app rápido

Comandos úteis:

```bash
adb install app.apk
adb shell am start -n pacote/.MainActivity
adb shell pm clear pacote
adb uninstall pacote
```

### 2) Coletar evidências rápido (logs e estado do sistema)

O básico de análise dinâmica é ver o que o app faz enquanto roda:

```bash
adb logcat
adb shell dumpsys package pacote
adb shell dumpsys activity
adb bugreport
```

Isso ajuda a pegar:

* crashes e erros
* chamadas suspeitas (rede, permissões, serviços, receivers)
* comportamento em background

### 3) Inspecionar arquivos e artefatos

Puxar coisas do device pro PC (configs, bancos, cache etc. — quando permitido):

```bash
adb pull /sdcard/Download/arquivo .
adb shell ls -la
adb shell cat /proc/net/tcp
```

Obs: acessar `/data/data/<pacote>` normalmente exige **root** ou `run-as` (se o app for debuggable).

### 4) Shell remoto pra triagem “ao vivo”

Você ganha um terminal no Android:

```bash
adb shell
```

Daí usa `pm`, `am`, `ps`, `top`, `getprop`, `ip`, etc. Pra malware isso é ouro pra checar serviços, processos, props, rede, permissões.

### 5) Encaminhamento de portas e controle externo

Port-forward é muito usado pra instrumentação (Frida, serviços locais, debug):

```bash
adb forward tcp:27042 tcp:27042
adb reverse tcp:8080 tcp:8080
```

## O lado “segurança” do ADB (não vacila aqui)

* **ADB ligado em aparelho pessoal é risco.** Se alguém tiver acesso físico + PC autorizado, dá pra fazer estrago.
* Em laboratório, mantém:

  * device/emulador isolado, sem conta pessoal
  * **revogar autorizações** quando terminar (Opções do desenvolvedor)
  * preferir USB; ADB via rede só quando você precisar e em rede isolada

Se você me disser **emulador ou aparelho físico** e **qual objetivo (logcat, captura de tráfego, Frida/MobSF)** eu te passo um fluxo de comandos “padrão” pra triagem rápida de APK malicioso.


Lista arquivos do projeto
```
adb -s 4AD01LH0H shell pm path com.seu.app
```

## comando para bixar apk pelo usb
```
$serial="4AD01LH0H"
$pkg="com.seu.app"

# pega todos os caminhos e remove o prefixo "package:"
$paths = (adb -s $serial shell pm path $pkg) -replace '^package:',''

# cria pasta
New-Item -ItemType Directory -Force -Path ".\$pkg" | Out-Null

# baixa tudo
foreach ($p in $paths) {
  adb -s $serial pull $p ".\$pkg\"
}

```

No final você vai ter uma pasta `.\com.seu.app\` com `base.apk` + todos os splits.

## Reinstalar em outro aparelho (mesma arquitetura) ou depois de mexer em arquivos sem “modificar código”

Dentro da pasta onde estão os APKs:

```powershell
adb -s 4AD01LH0H install-multiple -r base.apk split_*.apk
```

> Se for instalar em **outro** device, troca o `-s` pelo serial dele.

## Sobre “modificar o projeto”

* Do celular você **não baixa o “projeto”** (Android Studio/Unity). Só dá pra baixar **APK(s)**.
* Se isso for **Unity (parece que é)**: mesmo extraindo o APK, você **não recupera o projeto Unity**. Dá pra inspecionar recursos, libs, manifest, etc., mas não volta pro “projeto editável” como se tivesse os arquivos originais.

## O que dá pra fazer com segurança (análise)

* **Android Studio → Build → Analyze APK…** e abrir `base.apk` (e olhar splits também).
* Ver `AndroidManifest.xml`, permissões, libs nativas, assets, etc.

