### Comandos para usar o emulador Android

* Listar os emuladores (AVDs) disponíveis:

```
emulator -list-avds
```

* Iniciar um emulador (substitua `<NOME>` pelo nome listado):

```
emulator -avd <NOME>
```

* Iniciar com renderização por software (útil em VM ou sem GPU):

```
emulator -avd <NOME> -gpu swiftshader_indirect
```

## Se `emulator` não for reconhecido

Adicione o diretório do `emulator.exe` ao **PATH** do Windows:

1. Encontre o SDK do Android (ex.: `C:\Users\<seu_usuário>\AppData\Local\Android\Sdk`).
2. Copie o caminho da pasta `Sdk\emulator` (ex.: `...\Android\Sdk\emulator`).
3. Abra **Painel de Controle → Sistema → Configurações avançadas do sistema → Variáveis de Ambiente**.
4. Em **Path**, clique em **Editar** → **Novo** e cole o caminho da pasta `emulator`.
5. (Opcional) Adicione também `...\Android\Sdk\platform-tools` para usar `adb`.
6. Feche e reabra o CMD.

Teste:

```
emulator -version
```
