Sim, você colou certo. E os seus testes mostram **exatamente o problema**:

* **Android**: `wlan0 = 172.16.1.15/24`
* **PC**: não tem **nenhum** IP `172.16.1.x` no `ipconfig` (só `192.168.100.3`, `169.254.*`, `26.*` etc.)
* Por isso `ping 172.16.1.10` deu **Destination Host Unreachable**: esse IP **não é o seu PC** nessa rede.

Resultado: você configurou proxy pra um host que não existe/ não é alcançável.

##  Funciona mesmo com redes diferentes (ADB REVERSE) 

Esse é o melhor pra quando PC e celular não estão na mesma sub-rede (que é o seu caso agora).

### 1) No PC, deixe seu proxy escutando em `127.0.0.1:8080`

(geralmente já é o default)

### 2) Faça o “túnel” USB do Android → PC

```bash
adb reverse tcp:8080 tcp:8080
adb reverse --list
```

Tem que listar `tcp:8080`.

### 3) Aponte o proxy do Android pra localhost

```bash
adb shell settings put global http_proxy 127.0.0.1:8080
adb shell settings get global http_proxy
```

### 4) Valide

Abra no Android:

* `http://neverssl.com`

A requisição deve aparecer no seu proxy no PC.

### 5) Desfazer tudo

```bash
adb shell settings put global http_proxy :0
adb reverse --remove tcp:8080
```

---

## Sobre o “169.254.42.79”

`169.254.*` é IP automático (APIPA) de interface sem DHCP. Ele só serve se **o Android também tiver uma interface nessa ligação** (tipo USB/Ethernet/RNDIS ativa).
No seu `adb shell ip -4 addr show`, só aparece `wlan0`, então **o Android não está nessa rede 169.254 agora** — por isso não use 169.254 como “regra geral”, só quando a conexão entre PC↔celular for por aquela interface.
