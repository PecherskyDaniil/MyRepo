# Настройка Filebeat
Вначале создайте файл elvis.py в любой папке со следующим содержанием:
```
#!/usr/bin/python

import logging
import ecs_logging
import time
from random import randint

#logger = logging.getLogger(__name__)
logger = logging.getLogger("app")
logger.setLevel(logging.DEBUG)
handler = logging.FileHandler('elvis.json')
handler.setFormatter(ecs_logging.StdlibFormatter())
logger.addHandler(handler)

print("Generating log entries...")

messages = [
    "Elvis has left the building.",#
    "Elvis has left the oven on.",
    "Elvis has two left feet.",
    "Elvis was left out in the cold.",
    "Elvis was left holding the baby.",
    "Elvis left the cake out in the rain.",
    "Elvis came out of left field.",
    "Elvis exited stage left.",
    "Elvis took a left turn.",
    "Elvis left no stone unturned.",
    "Elvis picked up where he left off.",
    "Elvis's train has left the station."
    ]

while True:
    random1 = randint(0,15)
    random2 = randint(1,10)
    if random1 > 11:
        random1 = 0
    if(random1<=4):
        logger.info(messages[random1], extra={"http.request.body.content": messages[random1]})
    elif(random1>=5 and random1<=8):
        logger.warning(messages[random1], extra={"http.request.body.content": messages[random1]})
    elif(random1>=9 and random1<=10):
        logger.error(messages[random1], extra={"http.request.body.content": messages[random1]})
    else:
        logger.critical(messages[random1], extra={"http.request.body.content": messages[random1]})
    time.sleep(random2)
```
Данный файл нужен нам для создания логов. Вы можете создать или использовать свой, но учтите в таком случае вам нужно поменять некоторые конфиг filebeat.
1. Установим filebeat 
```
wget "https://cloud.iszf.irk.ru/index.php/s/AyjAk8tIB2P24H8/download?path=%2F&files=filebeat-8.13.2-amd64.deb" -O filebeat-8.13.2-amd64.deb

wget "https://cloud.iszf.irk.ru/index.php/s/AyjAk8tIB2P24H8/download?path=%2F&files=filebeat-8.13.2-amd64.deb.sha512" -O filebeat-8.13.2-amd64.deb.sha512

shasum -a 512 -c filebeat-8.13.2-amd64.deb.sha512
# should see:
# filebeat-8.13.2-amd64.deb: OK

sudo dpkg -i filebeat-8.13.2-amd64.deb
```
2. Получим ssl сертификат для отправки логов в elastic. Для этого пропишите следующую команду
   ```
    openssl s_client -showcerts -connect localhost:9200 < /dev/null | sed -n '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p'
   ```
После этого должна вывестись тарабарщина, с огражденная словами BEGIN CERTIFICTE и END CERTIFICATE. Если вывелась одна тарабарщина, то берите её, если 2, то берите вторую (нижнюю).
   ```
   -----BEGIN CERTIFICATE-----
MIIFqDCCA5CgAwIBAgITVCTcmkpU+47aVyuv/I6LfruFEjANBgkqhkiG9w0BAQsF
ADA8MTowOAYDVQQDEzFFbGFzdGljc2VhcmNoIHNlY3VyaXR5IGF1dG8tY29uZmln
dXJhdGlvbiBIVFRQIENBMB4XDTI0MDUxNDA3MjUwMloXDTI2MDUxNDA3MjUwMlow
HzEdMBsGA1UEAxMUc3Ryb25ndmlydHVhbG1hY2hpbmUwggIiMA0GCSqGSIb3DQEB
AQUAA4ICDwAwggIKAoICAQCiM3yHmafn7FVCeQ8NeysaUIUq77cdQ4IUj7ECRojT
Jo5w9ofc00vaF76FaLdJ/H37awcCTrHqJfu59vBtd7rlT6s3h33JZYXbXvHYwrL6
vvM4fyjBWpgN1/iHk6hEbIeSkCw7Et1o1ecrT5Lopy451XPSqwYNKHq9lD8q6SI7
1tLTM+UuZWflQUtP9PMtKHdtb/ZyTUE0v4dIiFsDhzAaHMeH+NnhOMfK2MtN/4LQ
Bx36Q7XfwZfX3LldrwedNzT4GuY72R3P0ChZx0kgWS+FxytJMCvpH+Yxh0RiLDRj
ueJCgbTFWqrILsH2sxNIkEBETszW8WNaeMTkahmneJMRMomC7Ul3NucnftOcEywU
4nLoQR9wL5HXjubRNWO2T9LERQwpUxNbb+QUXwwzORE9pOwxn94rXgtmfmUhF1Ew
dOhc2fa96kopQzIhbtN9v8TSrpGBR+O/ZE6VLIY7i4WAJAvmWCVXr7+fNYgPMBog
UuBX+P8GDbxiY33bM5rl4Id+wAhYAIZc00siNOxX/P84BpUlRy+TEYJECRSitWqm
2RrOLSQCsKqEF+FBUA2xyRUSLSq6YDp7t+P+jolhcPhprlkrwKGt5Y34Wif0R17a
nMEbfxHW8sGOWmxwQCfO4B1BoDbRMF7XyzL19RjLCI1u2d3BYUnAawQXR4a1fnGk
AwIDAQABo4G/MIG8MB0GA1UdDgQWBBTBcnDCfysPMp5tPQTp16JBlQ1ugjAfBgNV
HSMEGDAWgBS4HfPR5Tq0ynGG9/FkiJtAzj6uCTBaBgNVHREEUzBRhxD+gAAAAAAA
ANINHv/+5RpBhxAAAAAAAAAAAAAAAAAAAAABhwR/AAABghRzdHJvbmd2aXJ0dWFs
bWFjaGluZYIJbG9jYWxob3N0hwQKgAAUMAkGA1UdEwQCMAAwEwYDVR0lBAwwCgYI
KwYBBQUHAwEwDQYJKoZIhvcNAQELBQADggIBAApr966GwKZsGsIr2ggjl4al7fdM
Vbm1HE0tlBh5NyrheUwKCxu9LDM7GEpAC4GtffZFO1jWRgXR3FcTC0vyUADQjxQN
2G/UK5Ydmij0qxE2pNZr484Nn4b+18lNGUNoXIHlUjn4ks0mwFo2Zjk2N0rUi/jc
rLUY5FH/G2/E8I2DOb3cZHIjrciYd4a2DjaC5wAAE1P2joZyj6z/x0/4SZJB4v3m
4m+VJoLqysVQICt45YprvOx6pXdXIG0UREMWIGlapHGEYriqQjk8y6fAP0tnjkp7
VP+gisTM/QXKf6VQBdmQ6M2sMW5c5jFKnRfUrR41wLpyGzXlptW1RYnt9TA+X3iP
eCZGzds+HVPka7jXK+5QJFKAct8ThFwCioln4YxPQ6ogxAI8NDY4fiE8LFa3leIB
iMoAR73GbdsmmjAeaV0dsuYUe7QZvr+spK3J+iDprm9W77FxJ0nzr1OdnVkECHwy
iAOYeLspGG4F03HS5G/Atm4fRhmaqTFVuSwlla33BLHcJumy65wv3aQMPCCWzyNO
nXPTYjwmyDtCeUxj5qsKzQqsZNvLKYcXDV1tS4sHIvEiHMri6NQnILLksSkQaE8q
YnyiuiZX3WqOzhKRxHiTZGVsALUGiTNHbPUyIo7qoIkFD0yhXx6JmEA8z8jMPGpA
+gtWrZdkX2Pvl+j2
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
MIIFWTCCA0GgAwIBAgIURagdoQ6gawpz3PBOKH7EINxb6kIwDQYJKoZIhvcNAQEL
BQAwPDE6MDgGA1UEAxMxRWxhc3RpY3NlYXJjaCBzZWN1cml0eSBhdXRvLWNvbmZp
Z3VyYXRpb24gSFRUUCBDQTAeFw0yNDA1MTQwNzI1MDFaFw0yNzA1MTQwNzI1MDFa
MDwxOjA4BgNVBAMTMUVsYXN0aWNzZWFyY2ggc2VjdXJpdHkgYXV0by1jb25maWd1
cmF0aW9uIEhUVFAgQ0EwggIiMA0GCSqGSIb3DQEBAQUAA4ICDwAwggIKAoICAQDN
vCPlXiM1L1f5eRQM/dACDBE1zpgSkdHvfNGUxHpTaiCX9n/azjZJa0lORLp/hrTI
1WuMezyG98bGSkQXw+Yd/1jmCmfAdsbUcyMN5yWs++9GGBm/adqLD3k+IzgmUBBe
6QlQVGXIAPmUmXoq2nXGUVg9vATQ4bsXTfo0h7o/LS9VYPKdHpHKEoNGbrhziGEX
oGUqIBLlr8mjYvNj6qH32dZrEUuCCAHFuPIsEyJ5Ihh/HsT49W+s7rCzxRAispuI
5LcgTC40ku3raYeou46OjJsnx2nuoZskwavXszT73nHy4PT/WFm70J0o5vmfJv0z
bYbKP+Bqaz1WWpuaLfAgKQIrrg2kM8Cnd79n8Vbc9QIfesRLgg4AMU8hEebkbrVa
HG81/1L2IlWxuXiMOuGGjHJ5ppi9YybIfGqG72lt8S/nXeUFefmjCJdPHcSHkbv8
+PRbdaZmK5BBbkPHIoka45WoXnUdSzTyp/BQ/hntXIEQTrW65pWNtoTlFqY+gyj+
2OVgOc0znXACIRW0dbzGiEI5qZflUNHQEXSgFYX8Gvc/PRraiGHe4qyVXm/QZ/DD
RXVqI0iG6J4pQCLUGodSI4tzCrX3GVXSk7mE4ckM6ogYpeNZoyy4LLt2tuehwKHM
lhC0FCxwxwCpN3yz5btAMg65FChouIn8KqabnXxggwIDAQABo1MwUTAdBgNVHQ4E
FgQUuB3z0eU6tMpxhvfxZIibQM4+rgkwHwYDVR0jBBgwFoAUuB3z0eU6tMpxhvfx
ZIibQM4+rgkwDwYDVR0TAQH/BAUwAwEB/zANBgkqhkiG9w0BAQsFAAOCAgEAjfny
8xsrZqz4MAwRk5kdj7gWSPy2GjBersvO4aObsJpW3ICbYOlsgY0PWyTb7zPxi7qh
/BkfPIta7Bo+sRaZMzf/K3VWOAzMP8rexMhfMtYvSvW38U5LSP/nGJIjEhGKU9aS
+fza83ClIwr03eeIEYZeZt2gcVPjQNaFjnss9zM9U4IdIBr/0Sx04t3bG8JvBnxn
Jee0qG1tEOZEDGwI1BEbgu/Z2CDzvZDUcw1oIrGvybYWiDAmI+XD1QV32c3Y1qmy
rerOBsQKPslQkBMBVixiRpFvR4cc3+WuBZXrN+cIsaY38IwMCSbtgisNJn3ys/wl
37khYhmpvFZEsUlGvpxroukiPlASNLcsoJoU4uSzIfehRcYKVfaRIE5hl+zUDSUE
9bgNBYOHktB+FEMJOYkrHYCg3z2HXT6hLQMOd+H6PlyGKbLoe1kbjj4oEEaqskDv
gxkGWK86ZfuGx9wwjHkxw97Bl+J7+lXDAqsd3zQ/lvMoy9Edvuf8TXkIwKm1y+Zb
E/22ZSD/L70EadvyvpVpoPekmo7drqDzqhVbHLeGLCXQUfr3QDlkfuW/ffMwMmUY
09rn4B9EKLJ7Lk5Iu//ru34Ez83T4k8xDGvYM76Wd8L913PKKJPHc8Fbo/uXB0TA
eVBmmdtA7qrXfcxpA4bZx+2/UJ2Lwv0se+vRuJs=
-----END CERTIFICATE-----
   ```
4. Запишите сертификат(тарабарщина вместе с BEGIN CERTIFICATE и END CERTIFICATE) в отдельный файл и назовите его cafilebeat.pem

5. В папке где вы создали файл запишите следующую команду:
```
openssl x509 -in cafilebeat.pem -sha256 -fingerprint | sed 's/://g'
```
Скопируйте строку из символов идущую в первой строке после фразы "sha256 Fingerprint=" и запишите где нибудь, он нам понадобится.

6. Откройте файл ```/etc/filebeat/filebeat.yml```, найдите и запишите следующие праметры. УКАЖИТЕ СВОИ paths,password и ca_trusted_fingerprint. Убедитесь что все эти строки есть в файле и расскоментированы.
```
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /home/daniil/*.json #Напишите путь к папке с elvis.py, но "/*.json" в конце оставьте
  json.keys_under_root: true
  json.overwrite_keys: true
  json.add_error_key: true
  json.expand_keys: true

filebeat.config.modules:

  path: ${path.config}/modules.d/*.yml

  reload.enabled: false

setup.template.settings:
  index.number_of_shards: 1

setup.kibana:

output.elasticsearch:

  hosts: ["localhost:9200"]

  preset: balanced

  protocol: "https"

  username: "elastic"
  password: "your password" #Добавьте свой пароль
  ssl:
    enabled: true
    ca_trusted_fingerprint: "your_finger_print" #Подставьте свой fingerprint, который мы получили на прошлом шаге и записали

processors:
  - add_host_metadata:
      when.not.contains.tags: forwarded
  - add_cloud_metadata: ~
  - add_docker_metadata: ~
  - add_kubernetes_metadata: ~
```
7. Запустите сетап filebeat с помощью команды ```sudo filebeat setup -e```. При верном запуске в конце огромного вывода у вас должна быть строка
```
Loaded Ingest pipelines
```
8. Теперь запустите filebeat с помощью команды ```sudo filebeat -e``` и паралелльно в другом окне терминала запустите файл elvis.py

9. Filebeat отправляет логи на сервер, осталось лишь зайти в elactic и настроить Discover на поток данных filebeat.
![image](https://github.com/PecherskyDaniil/MyRepo/assets/78026424/c413e8f7-51d5-4d53-a8ca-bc9117de524b)
![image](https://github.com/PecherskyDaniil/MyRepo/assets/78026424/b4f2905a-3cdc-4814-b5e1-a966846547fb)


