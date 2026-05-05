
```bash
adb shell settings put global http_proxy 111.254.11.11:8080
```

Isso grava o proxy global como `host:porta` (com `:` separando). ([Android Enthusiasts Stack Exchange][1])

### Como validar se pegou

```bash
adb shell settings get global http_proxy
```

Deve retornar `111.254.11.11:8080`. ([The Poet Engineer][2])

### Como desativar

O jeito mais comum é:

```bash
adb shell settings put global http_proxy :0
```


(Alternativa: deletar as chaves `http_proxy` / `global_http_proxy_host` / `global_http_proxy_port`, mas `:0` normalmente resolve.) ([Stack Overflow][4])

### Sobre o seu IP (111.254.11.11)

* Esse `111.254.11.11` é **link-local/APIPA** do Windows. Funciona **se o Android alcança esse host** — e seu `ping` do Android pro PC confirma que alcança.
* O que mais costuma dar errado aqui não é o ADB, é o **proxy no PC**:

  1. o Burp/Charles/Fiddler tem que estar **escutando** em `0.0.0.0:8080` ou especificamente em `111.254.11.11:8080` (não só `127.0.0.1`) ([book.jorianwoltjer.com][5])
  2. **firewall do Windows** liberando entrada na porta 8080 nessa interface.



Depois de fazer a pote e validar temos que instalar o certificado. [exemplo de video](https://www.youtube.com/watch?v=ZUVUhkh2ELY)
