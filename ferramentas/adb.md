
## O que é (na prática) ADB

**ADB = Android Debug Bridge**. É a “ponte” oficial pra você controlar um Android (emulador ou aparelho) pelo PC via **USB** ou **rede**. Ele faz isso com 3 peças:

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