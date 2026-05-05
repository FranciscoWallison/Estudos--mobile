# Android Emulator (CLI)

> **Categoria:** Dispositivo / ADB
> **Plataforma:** Android (rodando no PC)
> **Quando usar:** subir um AVD pelo terminal sem abrir o Android Studio inteiro.
> **Alternativas:** Genymotion, MEmu/LDPlayer/BlueStacks (alguns expõem ADB, outros não).

## O que faz

CLI oficial do emulador Android (`emulator.exe`) que sobe AVDs criados no Android Studio. Útil pra automação, scripts e CI.

## Instalação (Windows)

Vem com o **Android SDK**. O executável fica em:

```
C:\Users\<seu_usuário>\AppData\Local\Android\Sdk\emulator\emulator.exe
```

## Como usar

Listar AVDs disponíveis:

```bash
emulator -list-avds
```

Iniciar um AVD (substitua `<NOME>`):

```bash
emulator -avd <NOME>
```

Iniciar com renderização por software (útil em VM ou sem GPU):

```bash
emulator -avd <NOME> -gpu swiftshader_indirect
```

## Se `emulator` não for reconhecido

Adicione o diretório do `emulator.exe` ao **PATH** do Windows:

1. Encontre o SDK do Android (ex.: `C:\Users\<seu_usuário>\AppData\Local\Android\Sdk`).
2. Copie o caminho da pasta `Sdk\emulator` (ex.: `...\Android\Sdk\emulator`).
3. **Painel de Controle → Sistema → Configurações avançadas do sistema → Variáveis de Ambiente**.
4. Em **Path**, **Editar → Novo** e cole o caminho da pasta `emulator`.
5. (Opcional) Adicione também `...\Android\Sdk\platform-tools` para usar `adb`.
6. Feche e reabra o terminal.

Teste:

```bash
emulator -version
```

## Dicas e pegadinhas

- Imagens **`google_apis_playstore`** não permitem `adb root`. Pra root no emulador, use **`google_apis`** (sem Play Store) ou **AOSP**.
- Se o emulador não aparece em `adb devices`, tente `adb connect 127.0.0.1:5555`.
- `-gpu swiftshader_indirect` resolve travas em VMs e PCs sem GPU dedicada — em troca, fica mais lento.

## Ver também

- [adb](adb.md) — controlar o emulador depois que sobe.
