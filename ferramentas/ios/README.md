# iOS

> Categoria placeholder — sem ferramentas documentadas ainda.

Ferramentas focadas em **iOS** (e iPadOS). O equivalente ao que existe em Android, com nomes e fluxos próprios da Apple.

## Roadmap (a adicionar)

- **Xcode + Console.app** — equivalente ao logcat (logs do device).
- **ios-deploy / `devicectl`** — instalar/abrir app no device pela CLI.
- **libimobiledevice (`idevice_*`)** — equivalente ao ADB em parte: `idevicesyslog`, `idevicepair`, `idevicedebug`.
- **Charles / Proxyman** — proxy de inspeção (Proxyman é praticamente Charles refeito).
- **Frida (iOS)** — funciona em device com jailbreak. Mais limitada que em Android.
- **class-dump / Hopper / Ghidra** — engenharia reversa de Mach-O.
- **objection (iOS)** — bypass de jailbreak detection / SSL pinning.

## Diferenças marcantes vs Android

- **APK** ↔ **`.ipa`** (zip com `.app/` dentro). Não existe `dex` — é Mach-O / Swift / Objective-C.
- Sem **ADB**. O análogo é uma combinação de `devicectl` (Apple, Xcode 15+) + `libimobiledevice` (open source).
- **Proxy global** existe em **Configurações → Wi-Fi → Configurar Proxy**. Não há `setprop` global.
- **SSL pinning bypass** sem jailbreak é muito mais limitado.
- Análise estática/decomp de IPA é mais difícil que de APK — código costuma vir mais "puro" Mach-O.

Conforme as ferramentas forem adicionadas, esta seção vira tabela como nas outras categorias.
