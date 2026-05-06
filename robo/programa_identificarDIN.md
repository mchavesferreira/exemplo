Sim. O HTML de `http://192.168.3.12/MD/IOSTATE.DG` é ótimo para isso, porque os estados aparecem em texto simples dentro da tag `<PRE>`, por exemplo:

```text
DIN[  81] OFF
DIN[ 101]  ON
DIN[ 120] OFF
```

No arquivo que você enviou, as entradas digitais aparecem exatamente nessa faixa **DIN[81] até DIN[120]**, com algumas já em `ON`, como `DIN[101]`, `DIN[102]`, `DIN[103]` e `DIN[108]`, e as demais em `OFF`. 

A ideia do programa será:

```text
1. Ler a página IOSTATE.DG repetidamente;
2. Extrair somente DIN[81] até DIN[120];
3. Guardar o estado anterior;
4. Comparar com o estado atual;
5. Mostrar na tela qual entrada mudou de OFF para ON ou de ON para OFF;
6. Gerar um log opcional em arquivo CSV.
```

---

# Programa Python para detectar mudança em DIN[81] até DIN[120]

Salve como:

```text
monitor_fanuc_din.py
```

Código completo:

```python
import re
import csv
import time
import argparse
from datetime import datetime

import requests


# Endereço da página de I/O do controlador FANUC
URL_PADRAO = "http://192.168.3.12/MD/IOSTATE.DG"

# Faixa de entradas digitais que serão monitoradas
DIN_INICIAL = 81
DIN_FINAL = 120


def ler_html_fanuc(url):
    """
    Lê a página HTML do webserver FANUC.

    O controlador antigo pode usar ISO-8859-1.
    Por isso, forçamos a interpretação do texto como ISO-8859-1
    se o requests não identificar corretamente.
    """

    # Parâmetro para evitar cache do navegador/proxy
    url_sem_cache = f"{url}?_={int(time.time() * 1000)}"

    resposta = requests.get(url_sem_cache, timeout=5)

    # Gera erro se HTTP for diferente de 200
    resposta.raise_for_status()

    # O HTML enviado pelo robô indica charset=iso-8859-1
    resposta.encoding = "iso-8859-1"

    return resposta.text


def extrair_din(html, din_inicial=81, din_final=120):
    """
    Extrai os estados DIN do HTML.

    Procura linhas no formato:
    DIN[  81] OFF
    DIN[ 101]  ON

    Retorna um dicionário:
    {
        81: "OFF",
        82: "OFF",
        101: "ON",
        ...
    }
    """

    estados = {}

    # Regex flexível para lidar com espaços variáveis:
    # DIN[  81] OFF
    # DIN[ 101]  ON
    padrao = re.compile(r"DIN\[\s*(\d+)\s*\]\s*(ON|OFF)", re.IGNORECASE)

    encontrados = padrao.findall(html)

    for numero_str, estado in encontrados:
        numero = int(numero_str)
        estado = estado.upper()

        if din_inicial <= numero <= din_final:
            estados[numero] = estado

    return estados


def imprimir_estado_inicial(estados):
    """
    Mostra o estado inicial das entradas monitoradas.
    """

    print("\nEstado inicial das entradas DIN monitoradas:")
    print("-" * 50)

    for numero in range(DIN_INICIAL, DIN_FINAL + 1):
        estado = estados.get(numero, "NAO_ENCONTRADO")
        print(f"DIN[{numero:3d}] = {estado}")

    print("-" * 50)
    print("Agora altere manualmente um sensor/entrada na planta.")
    print("O programa indicará automaticamente qual DIN mudou.\n")


def salvar_log_csv(nome_arquivo, linha):
    """
    Salva uma linha no arquivo CSV.

    linha deve ser uma lista, por exemplo:
    [data_hora, din, estado_anterior, estado_atual]
    """

    with open(nome_arquivo, mode="a", newline="", encoding="utf-8") as arquivo:
        escritor = csv.writer(arquivo, delimiter=";")
        escritor.writerow(linha)


def criar_cabecalho_log(nome_arquivo):
    """
    Cria o cabeçalho do arquivo CSV.
    """

    with open(nome_arquivo, mode="w", newline="", encoding="utf-8") as arquivo:
        escritor = csv.writer(arquivo, delimiter=";")
        escritor.writerow(["data_hora", "entrada", "estado_anterior", "estado_atual"])


def detectar_alteracoes(estados_anteriores, estados_atuais):
    """
    Compara dois dicionários de estados DIN.

    Retorna uma lista de alterações:
    [
        (81, "OFF", "ON"),
        (108, "ON", "OFF")
    ]
    """

    alteracoes = []

    for numero in range(DIN_INICIAL, DIN_FINAL + 1):
        anterior = estados_anteriores.get(numero)
        atual = estados_atuais.get(numero)

        # Se algum valor não foi encontrado, ignora essa entrada
        if anterior is None or atual is None:
            continue

        if anterior != atual:
            alteracoes.append((numero, anterior, atual))

    return alteracoes


def monitorar(url, intervalo, arquivo_log):
    """
    Loop principal de monitoramento.
    """

    print("Monitor de entradas digitais FANUC")
    print(f"URL: {url}")
    print(f"Faixa monitorada: DIN[{DIN_INICIAL}] até DIN[{DIN_FINAL}]")
    print(f"Intervalo de leitura: {intervalo} s")

    if arquivo_log:
        criar_cabecalho_log(arquivo_log)
        print(f"Log CSV: {arquivo_log}")

    print("\nLendo estado inicial...")

    html = ler_html_fanuc(url)
    estados_anteriores = extrair_din(html, DIN_INICIAL, DIN_FINAL)

    if not estados_anteriores:
        print("ERRO: Nenhuma entrada DIN foi encontrada no HTML.")
        print("Verifique se a página está acessível e se o formato é o esperado.")
        return

    imprimir_estado_inicial(estados_anteriores)

    while True:
        try:
            html = ler_html_fanuc(url)
            estados_atuais = extrair_din(html, DIN_INICIAL, DIN_FINAL)

            alteracoes = detectar_alteracoes(estados_anteriores, estados_atuais)

            if alteracoes:
                data_hora = datetime.now().strftime("%Y-%m-%d %H:%M:%S")

                for numero, anterior, atual in alteracoes:
                    mensagem = (
                        f"[{data_hora}] ALTERAÇÃO DETECTADA: "
                        f"DIN[{numero:3d}] {anterior} -> {atual}"
                    )

                    print(mensagem)

                    if arquivo_log:
                        salvar_log_csv(
                            arquivo_log,
                            [data_hora, f"DIN[{numero}]", anterior, atual]
                        )

                # Atualiza referência após detectar mudança
                estados_anteriores = estados_atuais

            time.sleep(intervalo)

        except requests.exceptions.RequestException as erro:
            print(f"Erro de comunicação com o controlador: {erro}")
            time.sleep(intervalo)

        except KeyboardInterrupt:
            print("\nMonitoramento encerrado pelo usuário.")
            break


def main():
    parser = argparse.ArgumentParser(
        description="Monitora entradas DIN[81] até DIN[120] de um controlador FANUC via webserver."
    )

    parser.add_argument(
        "--url",
        default=URL_PADRAO,
        help=f"URL da página IOSTATE.DG. Padrão: {URL_PADRAO}"
    )

    parser.add_argument(
        "--intervalo",
        type=float,
        default=0.5,
        help="Intervalo entre leituras em segundos. Padrão: 0.5"
    )

    parser.add_argument(
        "--log",
        default="alteracoes_din_fanuc.csv",
        help="Arquivo CSV de log. Use --log '' para desativar."
    )

    args = parser.parse_args()

    arquivo_log = args.log if args.log.strip() else None

    monitorar(args.url, args.intervalo, arquivo_log)


if __name__ == "__main__":
    main()
```

---

# Como instalar a dependência

No terminal:

```bash
pip install requests
```

---

# Como executar

Uso padrão:

```bash
python monitor_fanuc_din.py
```

Usando intervalo de leitura de 1 segundo:

```bash
python monitor_fanuc_din.py --intervalo 1
```

Sem gerar arquivo CSV:

```bash
python monitor_fanuc_din.py --log ""
```

Especificando outro IP:

```bash
python monitor_fanuc_din.py --url http://192.168.3.12/MD/IOSTATE.DG
```

---

# Exemplo esperado na tela

Ao iniciar, ele deve mostrar algo parecido com:

```text
Estado inicial das entradas DIN monitoradas:
--------------------------------------------------
DIN[ 81] = OFF
DIN[ 82] = OFF
DIN[ 83] = OFF
...
DIN[101] = ON
DIN[102] = ON
DIN[103] = ON
...
DIN[120] = OFF
--------------------------------------------------
Agora altere manualmente um sensor/entrada na planta.
O programa indicará automaticamente qual DIN mudou.
```

Quando você acionar um sensor manualmente:

```text
[2026-05-06 18:04:21] ALTERAÇÃO DETECTADA: DIN[105] OFF -> ON
```

Quando soltar o sensor:

```text
[2026-05-06 18:04:27] ALTERAÇÃO DETECTADA: DIN[105] ON -> OFF
```

---

# Arquivo CSV gerado

O programa também gera:

```text
alteracoes_din_fanuc.csv
```

Com conteúdo semelhante:

```csv
data_hora;entrada;estado_anterior;estado_atual
2026-05-06 18:04:21;DIN[105];OFF;ON
2026-05-06 18:04:27;DIN[105];ON;OFF
```

---

# Observação importante sobre atualização da página

Como o webserver do FANUC é antigo, pode haver cache. Por isso, no código usei esta linha:

```python
url_sem_cache = f"{url}?_={int(time.time() * 1000)}"
```

Ela força cada leitura a usar uma URL ligeiramente diferente:

```text
/MD/IOSTATE.DG?_=1715020000000
/MD/IOSTATE.DG?_=1715020000500
/MD/IOSTATE.DG?_=1715020001000
```

Assim, reduzimos a chance de o Python ler uma versão antiga da página.

---

# Versão mais simples, sem argumentos

Caso prefira uma versão bem direta para aula ou teste rápido:

```python
import re
import time
import requests
from datetime import datetime

URL = "http://192.168.3.12/MD/IOSTATE.DG"

DIN_INICIAL = 81
DIN_FINAL = 120


def ler_estados():
    url_sem_cache = f"{URL}?_={int(time.time() * 1000)}"

    resposta = requests.get(url_sem_cache, timeout=5)
    resposta.raise_for_status()
    resposta.encoding = "iso-8859-1"

    html = resposta.text

    padrao = re.compile(r"DIN\[\s*(\d+)\s*\]\s*(ON|OFF)", re.IGNORECASE)

    estados = {}

    for numero_str, estado in padrao.findall(html):
        numero = int(numero_str)

        if DIN_INICIAL <= numero <= DIN_FINAL:
            estados[numero] = estado.upper()

    return estados


print("Lendo estado inicial do controlador FANUC...")

estado_anterior = ler_estados()

print("Estado inicial:")
for numero in range(DIN_INICIAL, DIN_FINAL + 1):
    print(f"DIN[{numero:3d}] = {estado_anterior.get(numero, 'NAO_ENCONTRADO')}")

print("\nAltere manualmente um sensor. O programa mostrará qual DIN mudou.\n")

while True:
    try:
        estado_atual = ler_estados()

        for numero in range(DIN_INICIAL, DIN_FINAL + 1):
            anterior = estado_anterior.get(numero)
            atual = estado_atual.get(numero)

            if anterior is not None and atual is not None and anterior != atual:
                agora = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
                print(f"[{agora}] DIN[{numero:3d}] mudou de {anterior} para {atual}")

        estado_anterior = estado_atual

        time.sleep(0.5)

    except KeyboardInterrupt:
        print("\nEncerrado pelo usuário.")
        break

    except Exception as erro:
        print(f"Erro: {erro}")
        time.sleep(1)
```

Para sua aplicação de identificação física na planta, eu usaria a primeira versão, porque ela já registra tudo em CSV.
