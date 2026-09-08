# Relatório

(Necessita nova revisão aprofundada.)

## Questão 1

Topologia: `0 -> 2 -> 3` e `1 -> 2 -> 3`. Todos os enlaces têm capacidade de 2 Mbit/s. O gargalo comum aos três fluxos é a saída do nó 2 (`2 -> 3`). Para os gráficos, considere que TCP divide de forma justa a capacidade que resta depois do fluxo UDP; portanto, quando os três fluxos coexistem, o UDP usa 1,2 Mbit/s e cada TCP usa `(2 - 1,2)/2 = 0,4` Mbit/s.

### a) Sniffer na saída do nó 2

| Intervalo | TCP 0 -> 3 | TCP 1 -> 3 | UDP 0 -> 3 | Total no enlace 2 -> 3 |
| --- | ---: | ---: | ---: | ---: |
| 0–1 s | 0 | 0 | 0 | 0 Mbit/s |
| 1–2 s | 2,0 | 0 | 0 | 2,0 Mbit/s |
| 2–3 s | 1,0 | 1,0 | 0 | 2,0 Mbit/s |
| 3–5 s | 0,4 | 0,4 | 1,2 | 2,0 Mbit/s |
| 5–7 s | 0 | 2,0 | 0 | 2,0 Mbit/s |

Assim, o gráfico tem um patamar de 2 Mbit/s entre 1 s e 7 s, mas a composição dele muda nos instantes 2 s, 3 s e 5 s.

### b) Sniffer na saída do nó 0

Neste ponto não aparece o TCP iniciado no nó 1.

| Intervalo | TCP 0 -> 3 | UDP 0 -> 3 | Total no enlace 0 -> 2 |
| --- | ---: | ---: | ---: |
| 0–1 s | 0 | 0 | 0 Mbit/s |
| 1–3 s | 2,0 | 0 | 2,0 Mbit/s |
| 3–5 s | 0,4 | 1,2 | 1,6 Mbit/s |
| 5–7 s | 0 | 0 | 0 Mbit/s |

Entre 3 s e 5 s sobra capacidade em `0 -> 2` (0,4 Mbit/s), mas não no enlace compartilhado `2 -> 3`. O tráfego do nó 1 consome essa capacidade no gargalo.

### Segundo fluxo UDP (1 -> 3, de 4 s a 5 s)

De 4 s a 5 s, a demanda passaria a ser `1,2 + 1,2 + 0,4 + 0,4 = 3,2` Mbit/s em `2 -> 3`, maior que os 2 Mbit/s disponíveis. A fila *Droptail* encheria e descartaria os pacotes que chegassem quando ela estivesse cheia. Logo, a soma efetivamente transmitida não passaria de 2 Mbit/s e haveria perdas. Como *Droptail* não garante divisão por fluxo, não é possível fixar uma taxa exata para cada fluxo apenas com o enunciado: ela depende da ordem de chegada dos pacotes. Na prática, os dois UDPs não reduzem a oferta por
controle de congestionamento e os TCPs são os fluxos mais prejudicados.

## Questão 2

Computador 1: `10.67.103.29`

Computador 2: `10.67.103.12`

**Terminal 1 - Computador 1 como servidor**

```bash
aluno@s-67-103-29:~$ iperf -s
------------------------------------------------------------
Server listening on TCP port 5001
TCP window size: 128 KByte (default)
------------------------------------------------------------
[ 1] local 10.67.103.29 port 5001 connected with 10.67.103.12 port 45538 (icwnd/mss/irtt=14/1448/261)
[ ID] Interval Transfer Bandwidth
[ 1] 0.0000-10.0184 sec 1.10 GBytes 941 Mbits/sec
[ 2] local 10.67.103.29 port 5001 connected with 10.67.103.12 port 51904 (icwnd/mss/irtt=14/1448/151)
[ ID] Interval Transfer Bandwidth
[ 2] 0.0000-10.0192 sec 1.10 GBytes 941 Mbits/sec
[ 3] local 10.67.103.29 port 5001 connected with 10.67.103.12 port 40052 (icwnd/mss/irtt=14/1448/330)
[ ID] Interval Transfer Bandwidth
[ 3] 0.0000-10.0196 sec 1.10 GBytes 941 Mbits/sec
```

Em uma comunicação de rede, o **Servidor** fica esperando conexões e o **Cliente** inicia a conexão com o servidor.

`iperf -s`: `-s` coloca o iperf em modo servidor.

`local 10.67.103.29 port 5001 connected with 10.67.103.12`: Computador 2 (10.67.103.12) iniciou uma conexão com o Computador 2 (10.67.103.29).

Computador 1: 10.67.103.29 (cliente)  ---> TCP --->  Computador 2: 10.67.103.12 (servidor)                     

Os testes executados usam **TCP** (Transmission Control Protocol). É um protocolo de transporte orientado à conexão. Ele procura garantir:
- entrega de dados;
- entrega na ordem correta;
- retransmissão de segmentos perdidos;
- controle de fluxo;
- controle de congestionamento.

### TCP vs. UDP:

Testes:

| Teste | Transferência | Duração | Banda |
| --- | --- | --- | --- |
| 1 | 1,10 GB | ~10 s | 941 Mbit/s |
| 2 | 1,10 GB | ~10 s | 941 Mbit/s |
| 3 | 1,10 GB | ~10 s | 941 Mbit/s |

No sentido `Computador 2 -> Computador 1` a capacidade TCP ficou em aproximadamente 941 Mbit/s ≈ 0,94 Gbit/s.

**Terminal 2 - Computador 1 como cliente**

```bash
aluno@s-67-103-29:~$ iperf -c 10.67.103.12 -i 1
------------------------------------------------------------
Client connecting to 10.67.103.12, TCP port 5001
TCP window size: 16.0 KByte (default)
------------------------------------------------------------
[ 1] local 10.67.103.29 port 34910 connected with 10.67.103.12 port 5001 (icwnd/mss/irtt=14/1448/247)
[ ID] Interval Transfer Bandwidth
[ 1] 0.0000-1.0000 sec 114 MBytes 958 Mbits/sec
[ 1] 1.0000-2.0000 sec 112 MBytes 936 Mbits/sec
[ 1] 2.0000-3.0000 sec 113 MBytes 945 Mbits/sec
[ 1] 3.0000-4.0000 sec 112 MBytes 937 Mbits/sec
[ 1] 4.0000-5.0000 sec 113 MBytes 946 Mbits/sec
[ 1] 5.0000-6.0000 sec 112 MBytes 941 Mbits/sec
[ 1] 6.0000-7.0000 sec 112 MBytes 941 Mbits/sec
[ 1] 7.0000-8.0000 sec 112 MBytes 944 Mbits/sec
[ 1] 8.0000-9.0000 sec 112 MBytes 944 Mbits/sec
[ 1] 9.0000-10.0000 sec 112 MBytes 940 Mbits/sec
[ 1] 0.0000-10.0347 sec 1.10 GBytes 940 Mbits/sec
aluno@s-67-103-29:~$
```

`iperf -c 10.67.103.12 -i 1`: Computador 1 atua como cliente e envia dados para o Computador 2. A medição é a cada 1 segundo.

| Intervalo |      Banda |
| --------- | ---------: |
| 0–1 s     | 958 Mbit/s |
| 1–2 s     | 936 Mbit/s |
| 2–3 s     | 945 Mbit/s |
| 3–4 s     | 937 Mbit/s |
| 4–5 s     | 946 Mbit/s |
| 5–6 s     | 941 Mbit/s |
| 6–7 s     | 941 Mbit/s |
| 7–8 s     | 944 Mbit/s |
| 8–9 s     | 944 Mbit/s |
| 9–10 s    | 940 Mbit/s |

`Computador 1 -> Computador 2` ≈ ~940 Mbits/sec

Como a rede é de 1Gbit/s e os resultados foram ~940 Mbits/s, a rede provavelmente está operando em sua capacidade máxima.

## Questão 3

Como é necessário impor 1, 10 e 30 Mbit/s, o teste deve usar UDP, pois em TCP o `iperf` tenta ocupar a maior taxa disponível, e `-b` não produz os três patamares desejados. No servidor (`10.67.103.12`), o comando é:

```bash
iperf -s -u
```

No cliente (`10.67.103.29`), os três testes consecutivos, com uma amostra por segundo, podem ser executados assim:

```bash
iperf -c 10.67.103.12 -u -b 1M  -t 30 -i 1 > q3-1M.log
iperf -c 10.67.103.12 -u -b 10M -t 30 -i 1 > q3-10M.log
iperf -c 10.67.103.12 -u -b 30M -t 30 -i 1 > q3-30M.log
```

Concatenando os três logs, o eixo do tempo terá aproximadamente 90 s.

0–30 s em 1 Mbit/s,
30–60 s em 10 Mbit/s e
60–90 s em 30 Mbit/s.

Se for necessário um único arquivo de saída, pode-se usar:

```bash
for taxa in 1M 10M 30M; do
  iperf -c 10.67.103.12 -u -b "$taxa" -t 30 -i 1
done | tee q3-90s.log
```

O teste fornecido com `-u -b 10M` confirma a configuração: em todos os intervalos mostrados foram enviados 1,25 MBytes, ou aproximadamente 10,5 Mbit/s. O pequeno excesso sobre 10 Mbit/s é compatível com a granularidade de temporização e com a apresentação arredondada do `iperf`. Já `-b 10` significa 10 **bit/s**, não 10 Mbit/s; por isso o programa recusou o intervalo entre datagramas de 1176 s. É necessário usar o sufixo `M`.

O teste TCP de 90 s não é apropriado para construir esses três patamares, pois sua taxa não foi limitada: ele ficou próximo de 470 Mbit/s até cerca de 43 s e próximo de 940 Mbit/s depois disso. A mudança também sugere uma alteração de condição externa (por exemplo, outro fluxo concorrente), não os níveis de 1, 10 e 30 Mbit/s solicitados.


## Questão 4

Consulta feita ao [Panorama de Tráfego da Rede Ipê](https://redeipe.rnp.br/panorama)
em **07/09/2026, 22:00:47 (BRT)**.

Um **backbone** é a parte central, de alta capacidade, que interliga redes, cidades ou pontos de presença e transporta o tráfego agregado entre elas. A Rede Ipê é o backbone acadêmico nacional operado pela RNP.

![alt text](<Inspected image.png>)

![alt text](<Inspected image(1).png>)

![alt text](<Inspected image(2).png>)

**a) Duas rotas RS -> RJ.** As duas rotas a seguir existem no mapa:

1. `RS -> SP -> RJ`
2. `RS -> PR -> SP -> RJ`

Também seria possível usar `RS -> SC -> SP -> RJ`.

**b) Alternativa se SP -> RJ ficar indisponível.** Uma rota que não usa esse enlace é `RS -> SP -> MG -> RJ`. Ela aproveita os enlaces SP–MG e MG–RJ.

**c) PTT.** Um PTT (ou IX) é a infraestrutura neutra onde redes diferentes fazem *peering* e trocam tráfego diretamente. Ele diminui o número de saltos, latência e custo de trânsito, além de manter tráfego local na região. Três exemplos visíveis no panorama são PTT-RS (Porto Alegre), PTT-SP (São Paulo) e PTT-RJ (Rio de Janeiro).

**d) Conexões internacionais.** O mapa mostra, por exemplo, o ponto MIA (Miami, EUA), conectado ao RS por 100 Gb/s, e a conexão CE -> RedCLARA, de 100 Gb/s, que interliga a rede acadêmica brasileira à rede acadêmica latino-americana. O panorama também mostra conexões internacionais a partir de SP.

**e) Três PoPs.** PoP-RS (Porto Alegre), PoP-SP (São Paulo) e PoP-RJ (Rio de Janeiro). A RNP mantém 27 PoPs, um em cada unidade da federação.

**f) UFRGS.** A UFRGS abriga o PoP-RS no seu Centro de Processamento de Dados, em Porto Alegre. Portanto, ela é parte da infraestrutura de acesso da RNP no estado e se conecta ao restante da Rede Ipê pelo PoP-RS/Rede Tchê. Essa relação é documentada pelo [PoP-RS](https://pop-rs.rnp.br/).

**g) Link fora do ar.** No instante consultado, o enlace PB -> PB_JPA (200 Gb/s) aparecia tracejado/sem tráfego; seu gráfico diário indicava valor atual de 0 bit/s tanto na entrada quanto na saída. Assim, não havia tráfego circulando por ele naquele momento. Isso não permite, sozinho, afirmar a causa da indisponibilidade (manutenção, falha ou desligamento), apenas o efeito observado: tráfego nulo.

**h) Cores.** Elas representam a carga relativa do enlace: verde indica baixa utilização, amarelo indica utilização elevada/intermediária e vermelho indica utilização muito alta, próxima da capacidade. Preto/tracejado indica ausência de dados ou de tráfego no enlace. Logo, as cores não são, por si só, uma medida de capacidade: elas dependem da razão entre tráfego e capacidade.

**i) Enlace RS–PR.** A capacidade indicada é **200 Gb/s**. Na consulta, o gráfico do enlace PR–RS mostrava aproximadamente **1,60 Gb/s de entrada** e **1,83 Gb/s de saída** (médias no gráfico diário: 1,17 Gb/s e 1,07 Gb/s, respectivamente). É uma utilização muito inferior à capacidade total do enlace.
