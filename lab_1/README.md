# Relatório

## Notas importantes

### 1. Rede

Uma rede é um conjunto de dispositivos capazes de trocar dados através de algum meio de comunicação.

Computador A ---> Enlace ---> Computador B

Dispositivo ---> Interface ---> Enlace ---> Interface ---> Dispositivo

PC A ---> Switch ---> Roteador ---> Switch ---> PC B

### 2. Nó

Um nó é qualquer ponto ou dispositivo participante da rede.

Ex: computador, servidor, roteador, switch, impressora, access point, sensor.

### 3. Enlace

Um enlace, ou link, é a conexão que permite a transmissão de dados entre dois pontos da rede.

O enlace pode utilizar diferentes tecnologias ou meios físicos: cabo Ethernet, fibra óptica, Wi-Fi, rádio.

Se um enlace possui 2 Mbit/s significa que a **capacidade máxima de transmissão** do enlace é de 2 Mbits/s.

### 4. Interface de rede

O computador se conecta ao cabo por uma interface de rede. A interface pode ser Ethernet, Wi-Fi, fibra, interface virtual. Em Linux, é comum encontrar nomes como eth0, enp3s0, wlan0, lo. Cade interface pode ter configurações próprias, incluindo endereço IP.

```bash
Computador
┌─────────────────────────┐
│                         │
│       Sistema           │
│                         │
│    Interface Ethernet ──┼──── cabo
│                         │
└─────────────────────────┘
```

### 5. Placa de rede - NIC

A NIC (Network Interface Card) é o hardware responsável pela interface de rede.

```bash
CPU
 │
Sistema Operacional
 │
Driver
 │
NIC / placa de rede
 │
cabo Ethernet
```

Se uma placa de rede suporta apenas 100 Mbits/s, não adianta conectá-la a um switch de 1 Gbit/s esperando obter 1 Gbit/s. O caminho é limitado pela menor capacidade relevante. Portanto, nesse caso, a placa de rede seria o **gargalo**, ou bottleneck, ou seja, o recurso que limita a taxa do caminho. 

### 6. Topologia

Topologia é a forma como os nós e enlaces estão organizados.

A topologia da questão 1 é:

```bash
0 ──┐
    ├── 2 ─── 3
1 ──┘
```

Tanto o tráfego 0 -> 3 quanto 1 -> 3 precisa atravessar 2 -> 3. Logo, esse enlance é compatilhado pelos dois caminhos. Esse é o princípio do problema de **disputa de capacidade**.

### 7. Comutador

Um comutador Ethernet, ou Switch, conecta dispositivos dentro de uma rede local.

```bash
              ┌─────────┐
PC A ─────────│         │
PC B ─────────│ Switch  │──────── Servidor
PC C ─────────│         │
              └─────────┘
```

Cada cabo normalmente corresponde a um enlace. O switch recebe quadros Ethernet e decide para qual porta encaminhá-los.

PC A ---> porta 1

PC B ---> porta 2

PC C ---> porta 3

Se A envia um quadro destinado a B, o switch aprende onde cada dispositivo está e pode fazer `porta 1 ---> porta 2` em vez de simplesmente enviar para todo mundo.

### 8. Endereço MAC

Para realizar uma comutação Ethernet, o switch utiliza principalmente endereços MAC. Um endereço MAC se parece com `00:1A:2B:3C:4D;5E`. Cada interface Ethernet possui um endereço MAC.

MAC ---> usado na comunicação Ethernet local

IP ---> usado para comunicação lógica entre redes

O switch mantém uma espécie de tabela:

```bash
Endereço MAC       Porta
AA:AA:AA:AA:AA     1
BB:BB:BB:BB:BB     2
CC:CC:CC:CC:CC     3
```

### 9. Quadro Ethernet

Quando os dados estão trafegando em uma LAN Ethernet, eles são transportados em quadros, ou frames.

```bash
┌───────────────────────────────┐
│ Endereço MAC destino          │
│ Endereço MAC origem           │
│ Informações Ethernet          │
│ Dados                         │
│ Verificação de erros          │
└───────────────────────────────┘
```

Dentro da parte "Dados", normalmente existe um pacote IP. Essa ideia é chamada de encapsulamento.

```bash
Quadro Ethernet
┌──────────────────────────────┐
│ Cabeçalho Ethernet           │
│                              │
│   Pacote IP                  │
│   ┌──────────────────────┐   │
│   │ Cabeçalho IP         │   │
│   │                      │   │
│   │ Segmento TCP         │   │
│   │ ┌──────────────────┐ │   │
│   │ │ TCP + dados      │ │   │
│   │ └──────────────────┘ │   │
│   └──────────────────────┘   │
└──────────────────────────────┘
```

### 10. Pacote, segmento e quadro

| Camada    | Unidade típica   |
| --------- | ---------------- |
| Aplicação | dados            |
| TCP       | segmento         |
| IP        | pacote/datagrama |
| Ethernet  | quadro           |
| Física    | bits             |

Por exemplo, o `iperf` produz dados:

```bash
iperf
 ↓
dados
 ↓
TCP
 ↓
segmento TCP
 ↓
IP
 ↓
pacote IP
 ↓
Ethernet
 ↓
quadro Ethernet
 ↓
bits no cabo
```

### 11. Roteador

Um roteadro possui uma função diferente do switch; ele conecta redes IP diferentes e decide por ondde encaminhar pacotes.

```bash
Rede A                     Rede B
192.168.1.x                10.0.0.x

PC ── Switch ── Roteador ── Switch ── Servidor
```

Switch ---> liga dispositivos dentro de uma LAN ---> trabalha principalmente com MAC/Ethernet


Roteador ---> liga diferentes redes IP ---> trabalha principalmente com endereços IP

O roteador precisa decidir: **"Para onde envio este pacote para que ele se aproxime do destino?"** Para isso usa uma tabela de roteamento.

```bash
Rede destino        Próximo salto
10.0.0.0/24         interface 1
192.168.1.0/24      interface 2
0.0.0.0/0           roteador X
```

Cada passagem por um roteador pode ser chamada de um **salto**, ou hop.

### 12. LAN

Uma LAN (Local Area Network) é uma rede local.

```bash
              LAN
┌────────────────────────────────┐
│                                │
│ PC1 ─┐                         │
│      ├── Switch ─── Servidor   │
│ PC2 ─┘                         │
│                                │
└────────────────────────────────┘
```

### 13. WAN

Uma WAN (Wide Area Network) conecta redes a distâncias maiores.

```bash
LAN Porto Alegre
      │
   roteador
      │
      │
     WAN
      │
      │
   roteador
      │
LAN Rio de Janeiro
```

### 14. Rede de acesso e backbone

Uma rede de acesso conecta o usuário à infraestrutura de rede.

```bash
Seu PC
  │
switch
  │
roteador local
```

Um backbone é uma infraestrutura de alta capacidade que interliga diferentes partes da rede.

```bash
cidade A ═════ cidade B ═════ cidade C
          backbone
```

### 15. Fila

Imagine um switch ou roteador recebendo pacotes mais rapidamente do que consegue transmiti-los.

```bash
entrada:
3 Mbit/s
   ↓
┌──────────┐
│ fila     │
└──────────┘
   ↓
saída:
2 Mbit/s
```

Os pacotes precisam esperar, e essa espera ocorre em uma fila.

```bash
Pacotes chegando
↓ ↓ ↓ ↓ ↓ ↓

[ P1 ][ P2 ][ P3 ][ P4 ] → enlace de saída
         fila
```

A memória onde os pacotes ficam aguardando é chamada de buffer. O buffer possui tamanho limitado.

### 16. Drop e perda de pacotes

Se a fila estiver cheia quando um novo pacote chega, ele pode ser descartado:

```bash
fila cheia

[P1][P2][P3][P4]
               ↑
               cheia

novo pacote P5
      ↓
      X
   descartado
```

Isso é **packet loss**, a perda de pacote.

Uma fila **DropTail** descarta pacotes no final da fila.

```bash
pacote novo
    ↓

[P1][P2][P3][  ]
             ↓
           entra
```

```bash
[P1][P2][P3][P4]

pacote P5
    ↓
    X
```

Quando a soma dos fluxos ultrapassa a capacidade do enlace, a fila e os descartes começam.

---------------------

## Questão 1

---------------------

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

---------------------

## Questão 3

---------------------

## Questão 4


---------------------