# Relatório

### Questão 1

### Questão 2

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

`iperf -s`: `-s` coloca o iperf em modo servidor.

`local 10.67.103.29 port 5001 connected with 10.67.103.12`: Computador 2 (10.67.103.12) iniciou uma conexão com o Computador 2 (10.67.103.29).

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


### Questão 3

### Questão 4
