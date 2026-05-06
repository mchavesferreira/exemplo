Pelo **SUMMARY.DG** do webserver, o controlador FANUC está bem mais completo do que parecia pela primeira página. Há vários recursos úteis para integração, inclusive alguns que mudam bastante a análise anterior.

## 1. Identificação do controlador

O relatório mostra:

```text id="e0wyek"
Robot No: YS36352
Software: LR Tool
Versão: V7.50114
Data interna do relatório: 06/05/2026 16:55
Modelo/personality: LR Mate 200iC V7.50P/22
Servo Code: V20.00
DCS: V2.1.8
```

Também aparece:

```text id="x30qnm"
Controller ID: YS36352
Default Personality: LR Mate 200iC V7.50P/22
```

Ou seja, embora você tenha citado “200 iC”, o relatório indica **LR Mate 200iC** com software **V7.50P/22**. 

---

# 2. Recursos ativos importantes encontrados

A lista de configuração mostra vários recursos relevantes. Os principais para comunicação, supervisão e possível integração com ROS são:

## 2.1 Webserver e rede

Recursos encontrados:

```text id="juphbx"
TCP/IP Interface
FTP Interface
Telnet Interface
Web Server
Web Svr Enhancements
Host Communications
HTTP Proxy Svr
RDM Robot Discovery
```

Isso confirma que o controlador possui uma camada de comunicação Ethernet relativamente completa. O relatório também mostra:

```text id="6g1vxn"
Hostname: ROBOT
Endereço Ethernet: 192.168.3.12
Subnet mask: 255.255.255.0
Gateway/Router: 192.168.3.1
FTP Server state: 3
```

O fato de aparecer **Web Svr Enhancements R626** é relevante porque o webserver não está apenas no modo mínimo; ele possui melhorias carregadas. 

---

## 2.2 CGTP/iPendant

O relatório mostra explicitamente:

```text id="oql70t"
iPendant CGTP
iPendant Grid Display
iPendant Setup
iPendant HMI Setup
IntelligentTP PC I/F
```

Isso é muito importante.

Na homepage anterior aparecia:

```text id="xw4cci"
Navigate iPendant (CGTP): Unavailable
```

Mas no SUMMARY aparece que o recurso **iPendant CGTP** está listado entre as configurações carregadas. 

Minha interpretação é:

> O recurso/arquivo CGTP existe no sistema, mas a função “Navigate iPendant” pode estar indisponível por configuração, senha, bloqueio HTTP, falta de sessão válida, limitação de licença IPCC, modo operacional, navegador incompatível ou dependência de controle antigo no PC.

Portanto, não dá para concluir que “não existe CGTP”. Ele **aparece carregado** no resumo. O que não está disponível é o acesso navegável pela homepage naquele momento.

---

## 2.3 KAREL

Este é o ponto mais importante para ROS-Industrial.

O relatório mostra:

```text id="dvm8u6"
KAREL Cmd. Language J650
KAREL Run-Time Env J539
```

Isso significa que o controlador tem:

```text id="t3ij3k"
KAREL Command Language
KAREL Runtime Environment
```

Ou seja, ele aparentemente pode executar programas KAREL `.PC`. Isso é essencial para muitas integrações FANUC, inclusive o driver ROS-Industrial clássico. 

---

## 2.4 Socket Messaging

Outro ponto decisivo:

```text id="eokdln"
Socket Messaging R636
```

Esse recurso também aparece carregado. 

Isso muda bastante a conclusão anterior: o seu controlador aparentemente possui os dois recursos principais que costumam ser exigidos pelo **ROS-Industrial fanuc_driver**:

```text id="wqgx4t"
KAREL
Socket Messaging
```

Em outras palavras:

> Há boa chance de integração com ROS 1 usando ROS-Industrial sem comprar software FANUC adicional, desde que seja possível carregar os programas KAREL/TP necessários no controlador e configurar a rede corretamente.

---

## 2.5 SNPX / HMI Device

O relatório também mostra:

```text id="ftnpvw"
SNPX basic
HMI Device (SNPX) R553
```

Isso é muito útil para supervisão industrial.

SNPX/HMI Device normalmente permite que sistemas externos, IHMs ou supervisórios leiam/escrevam dados do robô, como registradores, posições, sinais e variáveis dependendo da configuração.

Para integração didática, isso pode permitir:

```text id="dnbyuz"
- leitura de registradores;
- troca de comandos simples;
- integração com supervisório;
- comunicação com CLP/IHM;
- monitoramento de status;
- eventual integração com Node-RED, Python ou ROS como supervisor.
```

---

## 2.6 I/O, UOP e Cell I/O

O controlador também possui:

```text id="g9dgiz"
Cell I/O
Analog I/O
Ext. DIO Config
External DI BWD
I/O Interconnect 2
```

E o relatório mostra sinais UOP ativos/configurados:

```text id="ciuzg3"
UI[1] *IMSTP
UI[2] *Hold
UI[3] *SFSPD
UI[4] Cycle stop
UI[5] Fault reset
UI[6] Start
UI[7] Home
UI[8] Enable
UI[9] a UI[16] RSR/PNS/STYLE
UI[17] PNS strobe
UI[18] Prod start
```

Saídas UOP:

```text id="ioxwm7"
UO[1] Cmd enabled
UO[2] System ready
UO[3] Prg running
UO[4] Prg paused
UO[5] Motion held
UO[6] Fault
UO[7] At perch
UO[8] TP enabled
UO[9] Batt alarm
UO[10] Busy
UO[11] a UO[18] ACK/SNO
UO[19] SNACK
```

Isso confirma que existe uma boa estrutura para integração por **I/O/UOP**, caso você queira uma alternativa mais simples e segura ao ROS controlando trajetória. 

---

# 3. Estado atual dos sinais de comando

O relatório mostra:

```text id="juauka"
UI[1] ON  *IMSTP
UI[2] ON  *Hold
UI[3] ON  *SFSPD
UI[8] ON  Enable
UO[1] ON  Cmd enabled
UO[2] ON  System ready
UO[6] OFF Fault
UO[8] OFF TP enabled
```

Interpretação:

| Sinal              | Estado | Interpretação prática                        |
| ------------------ | -----: | -------------------------------------------- |
| UI[1] *IMSTP       |     ON | cadeia de parada imediata está OK            |
| UI[2] *Hold        |     ON | não está em hold externo                     |
| UI[3] *SFSPD       |     ON | velocidade segura/entrada associada OK       |
| UI[8] Enable       |     ON | habilitação externa presente                 |
| UO[1] Cmd enabled  |     ON | comandos externos habilitados                |
| UO[2] System ready |     ON | sistema pronto                               |
| UO[6] Fault        |    OFF | sem falha ativa                              |
| UO[8] TP enabled   |    OFF | teach pendant não está habilitado no momento |

Isso indica que o sistema está em condição relativamente boa para comunicação/comando externo, pelo menos do ponto de vista dos sinais UOP. 

---

# 4. Posição atual e calibração

O relatório mostra posição articular:

```text id="rxg3ju"
J1: 33.27
J2: -14.78
J3: -3.52
J4: 2.07
J5: 6.64
J6: -14.86
```

Mas também mostra:

```text id="ojlucb"
CURRENT USER FRAME POSITION: NOT CALIBRATED
CURRENT WORLD POSITION: NOT CALIBRATED
```

Isso merece atenção.

A posição de juntas está disponível, mas a posição cartesiana em frame/world aparece como **NOT CALIBRATED**. Para ROS, MoveIt, células de visão ou integração espacial, isso pode ser um problema se a calibração/masterização/frame não estiver correta. 

Antes de fazer integração de trajetória, eu verificaria:

```text id="ozoow8"
- mastering do robô;
- zero position mastering;
- validade do Tool Frame;
- validade do User Frame;
- calibração mecânica;
- alarmes de mastering;
- DCS e espaço de trabalho.
```

---

# 5. Recursos que favorecem integração com ROS

Com base no SUMMARY, os recursos mais favoráveis são:

```text id="vl7r9k"
KAREL Cmd. Language J650
KAREL Run-Time Env J539
Socket Messaging R636
TCP/IP Interface
Host Communications
FTP Interface
Web Server
Web Svr Enhancements
SNPX basic
HMI Device (SNPX)
Cell I/O
UOP configurado
Position Registers
Program Status
Multi-Tasking
Background Editing
```

Conclusão direta:

> Este controlador parece possuir os recursos necessários para tentar uma integração ROS-Industrial clássica via KAREL + Socket Messaging.

Isso é uma ótima notícia.

---

# 6. Caminho recomendado para ROS 1

Para esse controlador **R-30iA / V7.50**, eu começaria por **ROS 1**, não por ROS 2.

Arquitetura sugerida:

```text id="xjw0ek"
PC Ubuntu com ROS Noetic ou Melodic
        |
        | Ethernet TCP/IP
        |
Controlador FANUC R-30iA
        |
        | KAREL + Socket Messaging
        |
Robô LR Mate 200iC
```

O driver ROS-Industrial para FANUC normalmente usa programas no controlador para comunicação com o PC. Como seu SUMMARY mostra **KAREL** e **Socket Messaging**, o caminho parece tecnicamente viável.

O pacote típico do ROS-Industrial usa programas como:

```text id="cexakb"
ros_relay
ros_state
simple_message
motion_streaming
```

A ideia geral é:

```text id="ee1mpi"
ROS / MoveIt gera comandos
Driver FANUC envia via TCP/IP
Programa KAREL no controlador recebe
Robô executa movimentos
Controlador devolve estado das juntas ao ROS
```

---

# 7. Caminho recomendado para ROS 2

ROS 2 também é possível em conceito, mas eu seria mais cauteloso.

O seu controlador tem:

```text id="k3r502"
HMI Device (SNPX)
Socket Messaging
KAREL
PC Interface
```

Mas não vi explicitamente no SUMMARY algo como:

```text id="97iu2a"
Remote Motion Interface
RMI
```

O driver ROS 2 oficial da FANUC costuma estar associado a opções mais modernas, como **Remote Motion Interface** e **HMI Device**. Como **HMI Device** aparece, mas **RMI** não aparece claramente, eu não assumiria compatibilidade direta com o driver ROS 2 oficial sem testar.

Minha recomendação:

```text id="v881ns"
1. Começar com ROS 1 + ROS-Industrial FANUC.
2. Depois, se necessário, criar uma ponte ROS 1 <-> ROS 2.
3. Ou usar ROS 2 apenas como supervisor via I/O, SNPX ou socket próprio.
```

---

# 8. Possibilidade de integração sem software proprietário novo

Com os recursos ativos encontrados, há três caminhos possíveis.

## Caminho A — ROS-Industrial via KAREL + Socket

Provavelmente o melhor.

Vantagens:

```text id="um1855"
- usa recursos já existentes no controlador;
- permite integração real com ROS;
- pode expor estado de juntas;
- pode aceitar comandos de movimento;
- compatível com abordagem acadêmica e pesquisa.
```

Riscos/cuidados:

```text id="sfwsrh"
- exige carregar arquivos KAREL/TP no robô;
- exige backup completo antes;
- exige atenção a segurança;
- pode depender de versão do driver;
- pode exigir ajuste de modelo URDF do LR Mate 200iC;
- DCS e frames precisam estar corretos.
```

## Caminho B — ROS supervisor via UOP/I/O

Mais simples e seguro.

```text id="dyaqmz"
ROS detecta/decide
ROS aciona CLP ou módulo I/O
UOP seleciona programa
Robô executa programa TP já ensinado
Robô devolve status por UO
```

Vantagens:

```text id="wga5az"
- não precisa comandar trajetória pelo ROS;
- mais seguro;
- mais fácil para ensino;
- usa sinais UOP já configurados;
- ideal para células didáticas.
```

Limitação:

```text id="yajgfo"
- ROS não controla diretamente as juntas;
- apenas seleciona e dispara programas.
```

## Caminho C — integração por SNPX/HMI Device

Útil para supervisão e troca de dados.

```text id="l36mjo"
ROS/Python/Node-RED <-> SNPX/HMI Device <-> registradores do robô
```

Vantagens:

```text id="w78zjv"
- bom para leitura/escrita de dados;
- pode integrar com supervisório;
- útil para dashboards;
- pode combinar com programas TP.
```

Limitação:

```text id="nzohau"
- não é necessariamente controle de trajetória em tempo real.
```

---

# 9. Sobre o ECHO/CGTP com base no SUMMARY

Agora temos uma informação nova: o SUMMARY lista **iPendant CGTP** como recurso carregado. 

Então, para o ECHO/CGTP, eu revisaria estas hipóteses:

## 9.1 Testar URLs diretas

```text id="nk5kj4"
http://192.168.3.12/frh/cgtp/echo.htm
http://192.168.3.12/frh/cgtp/cgtp.htm
http://192.168.3.12/frh/cgtp/banner.htm
http://192.168.3.12/frh/cgtp/echosvr.stm
```

## 9.2 Testar com Edge em modo Internet Explorer

Como o controlador é de 2012 e usa Browser Plugins/iPendant antigo, o teste com navegador moderno pode falhar mesmo com tudo instalado.

O SUMMARY mostra:

```text id="lu0llb"
Browser Plugins: V7.7063
TP Core Firmware: V7.7063
Teach Pendant: 7D0F/01O
```

Isso reforça que há componentes específicos de pendant/browser envolvidos. 

## 9.3 Verificar bloqueio de HTTP/iPendant

No teach pendant, procure configurações de:

```text id="gi3djo"
HTTP Authentication
iPendant Access
Remote iPendant
Web Server
CGTP
```

O recurso pode estar instalado, mas bloqueado por senha/permissão.

---

# 10. Diagnóstico final

Com base no arquivo SUMMARY, eu diria:

## Recursos favoráveis já presentes

```text id="s4wd6l"
KAREL: presente
KAREL Runtime: presente
Socket Messaging: presente
TCP/IP: presente
FTP: presente
Telnet: presente
Web Server: presente
Web Server Enhancements: presente
Host Communications: presente
HMI Device/SNPX: presente
Cell I/O/UOP: presente
iPendant CGTP: listado
DCS: presente
```

## Melhor conclusão

> Este controlador FANUC possui recursos suficientes para uma tentativa séria de integração com **ROS 1 / ROS-Industrial**, sem compra imediata de software FANUC adicional, porque o SUMMARY indica **KAREL** e **Socket Messaging** já carregados.

## Cuidado principal

Antes de controlar movimento via ROS, é necessário resolver/verificar:

```text id="hyrfo4"
- backup completo do controlador;
- mastering/calibração;
- User Frame e Tool Frame;
- estado de segurança;
- DCS;
- modo remoto/local;
- UOP;
- permissões de execução de programas KAREL;
- compatibilidade do driver FANUC ROS-Industrial com V7.50;
- modelo correto do robô no ROS/URDF.
```

Para fins acadêmicos, eu começaria com a integração em três níveis:

```text id="osokji"
Nível 1: leitura de estado via web/SNPX/I/O.
Nível 2: supervisão por UOP, Start, Hold, Reset e seleção de programa.
Nível 3: ROS-Industrial via KAREL + Socket Messaging.
```
