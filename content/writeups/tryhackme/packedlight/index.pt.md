+++
title = "Packed Light (Blue)"
date = "2026-08-08"
author = "berzrk"
description = "Análise de PCAP de um keylogger Python exfiltrando teclas via C2"
+++

Recebemos o arquivo `traffic.pcapng`.

Olhando os pacotes `http`, por sorte o primeiro pacote já é suspeito.

![Pacote HTTP](http-packet.png)

- Domínio suspeito
- Arquivo suspeito `updates.py`

E confirmamos que é malicioso depois de olhar a _response_ dessa requisição,
que retorna um **script Python malicioso**

![Keylogger](script.png)

O script é um **keylogger**.

1. Armazena a tecla pressionada
2. Criptografa a tecla usando XOR com a chave secreta de `getkey()`
3. Codifica em Base64
4. Envia os dados codificados como cookie `hotel_sess_data` para o servidor C2

E olhando as requisições HTTP enviadas ao servidor C2, conseguimos ver
quais teclas foram enviadas pelo keylogger: `ip.dst == <MALICIOUS IP> and http`
e adicionar o `Cookie` como coluna.

![Teclas](keyspressed.png)

Para extrair todos os valores do cookie, podemos usar o `tshark`:

```bash
tshark -r traffic.pcapng -Y 'http.cookie contains "hotel_sess_state"' -T fields \
-e http.cookie | cut -d '=' -f2- > keys.txt

# keys.txt vai ficar assim:
# HA==
# AA==
# BQ==
# ...
```

Só falta descriptografar esses valores. Basta reverter o processo
do script do keylogger: `decrypt(decode(key))`.

> A descriptografia com XOR usa a mesma função da criptografia, desde que a chave seja a mesma.

Fiz isso de duas formas: com um script Python e o [CyberChef](https://gchq.github.io/CyberChef/)

# Python Script

```python
from base64 import b64decode

key = b"H0t3lSt@ff0NlyK3epS3cr3t!"

# Mesma função do script malicioso
def xor(data: bytes, key: bytes) -> bytes:
    return bytes(b ^ key[i % len(key)] for i, b in enumerate(data))

with open("keys.txt", "r") as f:
    chars = f.readlines()
    for char in chars:
        decoded = b64decode(char.rstrip())
        plain = xor(decoded, key)
        print(plain.decode(), end="")
```

# CyberChef

![Solução no CyberChef](solution2.png)
