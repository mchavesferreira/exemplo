
# Projeto voltametrico


## Correções circuito
-mux para o ganho ADG444BRZ

https://github.com/mchavesferreira/exemplo/blob/main/voltametrico/ganhoADG444BRZ.md


-versao dip Isolador DIP nme0505

-Vamos considerar que o voltametrico funcione a bateria, isolado e via  wifi.  utilize este módulo tp4052, mas estou em duvida com o nme0505 que precisa de 5V


## Entender as diferenças entre potenciostato/galvanostato

<img src=imagens/galvanostatopotenciostato.png>
https://drive.google.com/file/d/1yTjzVBx_RDu1pM47FJziYduYy4tpQ3qN/view?usp=sharing



## Artigos exemplos objetivos

- Ter um módulo de baixo custo 
- Comunicar via wifi 
- Ter isolação galvânica 



https://drive.google.com/file/d/113iuUMNPAcvJdJro1Io1QJc1cyO33_ir/view?usp=drive_link  

Explica as partes do potenciostato/galvanostato explicando suas diferenças
https://drive.google.com/file/d/1gMekI8pySnw4SjDgDWFvgZCK0BO0uaw2/view?usp=sharing

### pode ser controlado via smartphone


Faz um panorama em portugues sobre estado da arte de potenciostato de baixo custo
https://drive.google.com/file/d/1n0MxDC6zaWN0GzV9-p9nlvbsQcckggf-/view?usp=sharing

## partes do potenciostato como referencia

https://drive.google.com/file/d/1ZwtPqTY9BzZw_8TZtRkg7ToJRidK9tyj/view?usp=sharing


## referencias principais com autoriza

https://www.teses.usp.br/teses/disponiveis/3/3140/tde-18122009-125440/publico/TESEzaira.pdf   pagina 48 esquema eletrico do potenciostato


## Circuito Ibanis/Zaira/Marcos

<img src=referencias.png>
LM4040 2.5V e 4.096V com ganho buffer

<img src=ibanismarcos.png>
Circuito potenciostato. IC6-LT é um conversor DA, este sinal é somado a referencia 2.5V obtendo a linha )Senal) é aplicado a entrada (+) do amplificador TIA (op177). O ganho deste amplificador vem do circuito com CD4052. A saida do TIA é aplicado ao somador com 2.5V.  
Após o sinal é aplicado ao conversor DA.
### Dúvida
Montar o circuito para verificar se é necessário outro inversor  (IN-fil out-fil)



<img src=ganhocd4052.png>
circuito de ganho

## Circuito de uma monografia do mesmo grupo

<img src=potenciostato_cardoso.png>

