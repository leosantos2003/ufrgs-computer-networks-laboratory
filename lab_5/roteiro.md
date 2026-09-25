# Roteiro didático - TCP, UDP e controle de congestionamento

Este roteiro acompanha o arquivo `_LabTransp1_TCP.pdf`. Faça as atividades na ordem proposta: as primeiras consolidam leitura de pacotes no Wireshark, e as últimas usam os mesmos conceitos para entender os protocolos confiáveis.

> Os números de IP, portas efêmeras, sequência, janela e tempos observados podem mudar entre capturas. No relatório, use os valores da **sua** captura e indique o número do pacote que serviu de evidência.

## 0. Preparação

1. Abra o Wireshark e identifique a interface que está realmente conectada à rede (`Wi-Fi` ou `Ethernet`). Os contadores/mini gráficos devem aumentar enquanto você navega.
2. Familiarize-se com os três painéis: lista de pacotes (em cima), detalhes decodificados (centro) e bytes brutos (embaixo). Os campos pedidos pelo laboratório ficam no painel central, expandindo as setas de cada protocolo.
3. Use a barra superior para **filtros de exibição**. Ela não apaga nada da captura; apenas mostra os pacotes de interesse. Exemplos que serão úteis:

   ```wireshark
   dns
   udp
   tcp.flags.syn == 1
   tcp.stream eq N
   ```

4. Se preferir enxergar a pilha de protocolos como um desenho, no Wireshark use **Editar > Preferências > Layout** e configure o painel 3 como **Packet Diagram**. Isso é opcional, mas ajuda bastante na questão 1.

5. Crie uma pasta ou um documento de anotações. Para cada questão, registre: filtro usado, número do pacote/etapa da animação, campos observados, cálculo e conclusão. Capturas de tela devem mostrar os campos que justificam a resposta.

## 1. UDP e a formação de um pacote DNS

### 1.1 Calcule antes de capturar

O cabeçalho UDP tem sempre **8 bytes**. Portanto, se a mensagem DNS (os dados entregues ao UDP) possui 40 bytes:

```text
UDP Length = cabeçalho UDP + dados DNS
UDP Length = 8 + 40 = 48 bytes
```

Logo, a resposta esperada para o campo **Length** do UDP é `48`. Não confunda esse campo com a coluna `Length` da lista do Wireshark: a coluna normalmente mostra o tamanho do quadro inteiro.

Os quatro campos do cabeçalho UDP são:

| Campo | Tamanho | Como interpretar |
|---|---:|---|
| Source Port | 16 bits | Porta do processo que enviou o datagrama; numa consulta DNS, em geral é uma porta efêmera do cliente. |
| Destination Port | 16 bits | Porta do serviço destinatário; DNS convencional costuma usar 53. |
| Length | 16 bits | Tamanho em bytes de **UDP inteiro**: cabeçalho de 8 bytes + dados. |
| Checksum | 16 bits | Verificação de integridade sobre UDP, pseudo-cabeçalho IP e dados. |

### 1.2 Gere e localize uma consulta DNS

1. Inicie a captura no Wireshark.
2. Faça uma consulta DNS. Em Linux, execute `nslookup www.ufrgs.br.` em um terminal. Em Windows, execute o mesmo comando no Prompt/PowerShell. O ponto final torna o nome absoluto.
3. Pare a captura e use o filtro `dns`.
4. Escolha uma **consulta** ou **resposta** que esteja usando UDP. No painel central, expanda `User Datagram Protocol` e confirme o valor de `Length`.
5. Para relacionar a captura com o enunciado, procure um pacote cujo bloco DNS tenha `40 bytes`/`DNS length: 40`, se ele existir. Caso a sua consulta tenha outro tamanho, não altere o cálculo teórico: explique que, para 40 bytes DNS, o valor seria 48; depois registre separadamente o valor realmente visto na captura.

Se não aparecer DNS em UDP, a rede pode usar DNS sobre HTTPS/TLS. Tente consultar o resolvedor local explicitamente, desative temporariamente o navegador durante o teste ou peça orientação ao professor; não conclua que UDP deixou de existir.

### 1.3 Desenhe a encapsulação

Use o pacote selecionado como modelo. Em uma rede Ethernet com IPv4 e cabeçalho IP sem opções, o desenho-base é:

```text
Quadro Ethernet
├── Cabeçalho Ethernet ........ 14 bytes
├── Pacote IPv4
│   ├── Cabeçalho IPv4 ........ 20 bytes (valor comum; confirme IHL)
│   └── Datagrama UDP
│       ├── Cabeçalho UDP ..... 8 bytes
│       └── Mensagem DNS ...... 40 bytes (hipótese da questão)
└── FCS/padding ............... dependem da interface/captura
```

Para desenhar a sua captura, expanda, nessa ordem, `Ethernet II`, `Internet Protocol Version 4` (ou IPv6), `User Datagram Protocol` e `Domain Name System`. Anote ao menos os endereços MAC, IPs, portas UDP e a pergunta/resposta DNS. Se houver uma tag VLAN, opções IP ou IPv6, desenhe-a: não force os tamanhos do exemplo acima a uma captura diferente.

## 2. Animação de controle de congestionamento TCP

Abra a animação indicada no enunciado:

<https://media.pearsoncmg.com/aw/ecs_kurose_compnetwork_7/cw/content/interactiveanimations/tcp-congestion/index.html>

Caso a página abra sobre fundo escuro/poluído, o lembrete do enunciado é usar `about:blank` para abrir uma tela branca antes de acessar a animação.

1. Leia a legenda e identifique as duas conexões/estações, o gargalo e o gráfico de taxa de transmissão.
2. Execute a animação até aparecer o comportamento semelhante ao da figura do enunciado. Se houver reinício, faça uma rodada limpa e tire uma captura de tela quando as curvas já estiverem estáveis.
3. Anote o que ocorre logo após uma perda: a janela de congestionamento/taxa diminui. Nos intervalos sem perda, ela cresce novamente.
4. Responda à pergunta central relacionando o gráfico ao **AIMD** (*additive increase, multiplicative decrease*): conexões que compartilham o mesmo gargalo detectam congestionamento por perdas; cada uma reduz sua taxa de forma multiplicativa e, sem novas perdas, aumenta gradualmente. Esse mecanismo faz as taxas tenderem a uma divisão aproximadamente justa da capacidade do gargalo. Elas não precisam ficar exatamente iguais em todo instante.

Use no relatório termos como `cwnd` (janela de congestionamento), capacidade do enlace gargalo, perda e ACK. Não atribua a aproximação apenas ao RTT: o fator decisivo no experimento é o controle de congestionamento compartilhando o gargalo.

## 3. Slow start: determine os próximos pacotes

Observe com atenção a figura da questão 3. O transmissor recebeu ACKs para `a3`, `a4`, `a5` e `a6`, sem erros, e ainda está em **slow start**.

O procedimento para resolver é:

1. Na figura, encontre quais pacotes já foram enviados e quais foram confirmados antes de chegar o ACK de `a3`.
2. Conte a janela de congestionamento disponível imediatamente antes de cada ACK. Em slow start, para cada ACK novo recebido, o TCP aumenta `cwnd` em aproximadamente 1 MSS.
3. Após cada aumento, compare `cwnd` com a quantidade de dados que já está “em voo” (enviados, mas ainda sem ACK). A diferença é a quantidade que pode ser enviada agora.
4. Siga a linha do tempo do desenho: quando o ACK libera uma vaga, envie o próximo pacote numerado que ainda não apareceu. Quando vários ACKs chegam, não envie novamente `p` já transmitido; avance a numeração.
5. Ao final dos quatro ACKs, escreva a sequência completa de novos pacotes na notação solicitada, por exemplo `p7, p8, ...`, preservando a ordem de envio mostrada pelo seu raciocínio.

Uma tabela evita saltos de numeração:

| ACK recebido | `cwnd` depois do ACK | Dados ainda em voo | Novos pacotes liberados |
|---|---:|---:|---|
| `a3` |  |  |  |
| `a4` |  |  |  |
| `a5` |  |  |  |
| `a6` |  |  |  |

> Atenção: `slow start` não significa “enviar tudo de uma vez”. O crescimento é por ACK recebido; por RTT, se todos os segmentos forem confirmados, a janela tende a dobrar.

## 4. MSS e MTU no handshake TCP

### 4.1 Encontre os dois pacotes do estabelecimento

Abra uma captura TCP própria ou a captura de apoio fornecida pelo laboratório, se ela tiver sido indicada pelo professor. Aplique:

```wireshark
tcp.flags.syn == 1
```

O resultado traz o `SYN` inicial e o `SYN, ACK` de resposta. Para não misturar conexões, selecione um deles, clique com o botão direito e escolha **Conversation Filter > TCP** ou **Follow > TCP Stream**. Depois examine os dois pacotes desse mesmo fluxo.

### 4.2 Leia o MSS anunciado e determine o MSS efetivo

1. No `SYN` do cliente, expanda `Transmission Control Protocol > Options` e localize `Maximum segment size` / `MSS Value`.
2. Repita no `SYN, ACK` do servidor.
3. Cada valor anuncia o maior payload TCP que **aquele emissor** aceita receber. Assim, há dois limites direcionais: o MSS oferecido pelo cliente limita os dados enviados pelo servidor; o MSS oferecido pelo servidor limita os dados enviados pelo cliente.
4. Se o enunciado tratar a conexão em um único sentido, use o MSS anunciado pelo receptor daquele sentido. Se pedir um único valor sem especificar, apresente os dois e explique essa direção, em vez de escolher arbitrariamente.

### 4.3 Estime o MTU

Para IPv4, sem opções TCP nem IP, use:

```text
MTU estimado = MSS + cabeçalho TCP (20) + cabeçalho IPv4 (20)
```

Exemplo apenas de método: MSS 1460 leva a `1460 + 20 + 20 = 1500 bytes`, um MTU Ethernet comum. Confirme se o cabeçalho TCP possui opções: elas aparecem no handshake, mas normalmente não estão presentes nos segmentos de dados; quando estiverem, use o tamanho real exibido em `Header Length`. Para IPv6, o cabeçalho IP básico tem 40 bytes, e extensões devem ser acrescentadas quando existirem.

O MTU se refere ao pacote da camada de rede (IP + cabeçalho TCP + dados), não ao quadro Ethernet completo. Ethernet, preâmbulo e FCS não entram nessa conta.

### 4.4 Caso o MSS não seja anunciado

Leia a RFC 1122, seção 4.2.2.6 (`MSS Option`), disponível em <https://www.rfc-editor.org/rfc/rfc1122#section-4.2.2.6>. Para TCP sobre IPv4, a regra histórica indicada é assumir MSS padrão de **536 bytes** quando a opção MSS não está presente. No relatório, cite a seção e deixe claro que esse é o padrão de interoperabilidade descrito na RFC, não um valor obtido da sua captura.

## 5. Interpretar opções TCP, janela e tamanho de segmento

Nesta questão as duas imagens são a visão da máquina A. A figura 1a é o `SYN, ACK` vindo de B; a figura 1b é um segmento de dados posterior, também enviado por B. Leia os valores das figuras, não os substitua por valores de outra captura.

### 5.1 Item (a): buffer de recepção de B

No `SYN, ACK` (figura 1a), anote:

- o campo `Window size value`/`Window` anunciado por B;
- a opção `Window scale`, se ela estiver presente.

Calcule a janela de recepção anunciada por B com:

```text
Janela de recepção = Window size value × 2^(window scale)
```

Essa é a capacidade que B anuncia para receber dados de A naquele momento. A expressão “buffer de recepção” da questão normalmente se refere a esse espaço/janela anunciada; escreva a unidade em bytes. Não aplique `Window scale` ao campo da figura 1b sem antes verificar se o fator foi negociado no handshake: ele é negociado no SYN e vale durante a conexão.

### 5.2 Item (b): SACK

No `SYN, ACK`, procure a opção `SACK Permitted`. Ela significa que A e B negociaram a capacidade de usar **Selective Acknowledgment**. Assim:

- se `SACK Permitted` aparece no handshake, pode-se afirmar que a opção foi negociada/permitida;
- isso, isoladamente, não prova que já houve blocos SACK enviados em um ACK posterior;
- para provar uso efetivo, seria preciso encontrar uma opção `SACK` com blocos de borda em um ACK posterior.

Formule a resposta respeitando exatamente o que a figura mostra: “SACK foi negociado” e “SACK foi efetivamente usado” são afirmações diferentes.

### 5.3 Itens (c) e (d): dados TCP e unidade de transferência

Na figura 1b, localize o valor `TCP Segment Len` (ou `Len`). Esse é o tamanho da área de dados TCP: ele não inclui o cabeçalho TCP nem o IP.

Para estimar a unidade de transferência/pacote IP do lado direito, some:

```text
Tamanho IP estimado = TCP Segment Len + tamanho real do cabeçalho TCP + tamanho real do cabeçalho IP
```

Em IPv4 sem opções, os dois cabeçalhos são normalmente 20 bytes cada. Se o pacote for IPv6, use 40 bytes para o cabeçalho IPv6 básico. Confira os campos `Header Length` e `Internet Header Length`: as opções TCP/IP alteram a conta. Novamente, a resposta é tamanho de pacote de rede; não acrescente 14 bytes de Ethernet a menos que o professor tenha pedido explicitamente o quadro.

### 5.4 Item (e): de onde vem `TCP Segment Len`?

`TCP Segment Len` é uma informação **calculada pelo Wireshark**, não um campo transmitido no cabeçalho TCP. O analisador obtém o tamanho do payload subtraindo o tamanho efetivo do cabeçalho TCP do tamanho de dados TCP disponível no pacote IP. Explique isso e mostre, na figura, onde o Wireshark exibe o comprimento do cabeçalho e o `Len` calculado.

## 6. Simulação Go-Back-N (GBN)

Abra:

<https://media.pearsoncmg.com/aw/ecs_kurose_compnetwork_7/cw/content/interactiveanimations/go-back-n-protocol/index.html>

Antes de começar, identifique o controle de envio de pacote, a opção para perder pacote/ACK e o botão para avançar. Faça capturas de tela dos instantes de perda, ACK repetido e retransmissão.

### 6.1 Rodada (a): pacotes 0 a 4 sem erro

1. Reinicie a simulação.
2. Envie os pacotes `0`, `1`, `2`, `3` e `4` sem marcar perda de pacote nem perda de ACK.
3. Anote os ACKs. Em GBN, o ACK normalmente é **cumulativo**: um ACK para o próximo número esperado confirma todos os pacotes anteriores recebidos em ordem.
4. Confirme que a base da janela avança normalmente, sem timeout nem retransmissão.

### 6.2 Rodada (b): pacotes 5 a 9, com perda do pacote 6

1. Continue a numeração da rodada anterior ou reinicie e avance corretamente até 5, conforme a animação exigir.
2. Envie `5` normalmente e provoque a perda de `6`.
3. Envie `7`, `8` e `9` sem outros erros.
4. Observe que, em GBN, o receptor descarta pacotes fora de ordem. Ele repete o ACK cumulativo do último dado recebido em ordem (ou do próximo esperado).
5. Espere o timeout do pacote mais antigo ainda não confirmado. O emissor retransmite **6 e todos os pacotes posteriores pendentes**: esse é o “go back N”.
6. Registre a sequência de ACKs duplicados, o timeout e a lista de retransmissões.

### 6.3 Rodada (c): pacotes 11 a 15, com perda do ACK 13

1. Envie `11` a `15`, causando somente a perda de `ACK 13` conforme a numeração apresentada pela animação.
2. Observe que um ACK cumulativo posterior pode confirmar também os dados anteriores. Portanto, a perda de um único ACK não implica automaticamente retransmissão.
3. Anote se o ACK posterior chegou antes do timeout. Se chegou, a base avança; se não chegou, descreva o timeout e a retransmissão que a animação mostra.

## 7. Simulação Selective Repeat (SR)

Abra:

<https://media.pearsoncmg.com/aw/ecs_kurose_compnetwork_7/cw/content/interactiveanimations/selective-repeat-protocol/index.html>

Repita exatamente os três cenários de GBN, registrando os mesmos eventos. O objetivo é comparar os protocolos sob o mesmo erro, não obter uma animação “bonita”.

### 7.1 Rodada (a): pacotes 0 a 4 sem erro

Envie `0` a `4` sem erros e confira que cada pacote é reconhecido individualmente. A janela avança quando a base e os pacotes necessários já estão confirmados.

### 7.2 Rodada (b): pacotes 5 a 9, com perda do pacote 6

1. Envie `5`; provoque perda apenas de `6`; envie `7`, `8` e `9` normalmente.
2. Ao contrário de GBN, o receptor SR aceita e guarda pacotes que chegam fora de ordem, desde que estejam na janela de recepção.
3. O receptor envia ACKs individuais para `7`, `8` e `9`, mesmo enquanto espera `6`.
4. Espere o temporizador de `6`: o emissor retransmite **somente `6`**.
5. Quando `6` chega, o receptor pode entregar a sequência já armazenada à aplicação. Registre esse momento e compare-o com a retransmissão em bloco do GBN.

### 7.3 Rodada (c): pacotes 11 a 15, com perda do ACK 13

1. Envie os pacotes, provocando somente a perda do ACK solicitado.
2. Em SR, a confirmação é por pacote. Se o ACK de `13` se perder, o emissor pode ter de retransmitir `13` quando o temporizador individual dele expirar, ainda que os ACKs de outros pacotes tenham chegado.
3. O receptor reconhece uma cópia retransmitida e volta a responder com ACK de `13`; ele não deve entregar os mesmos dados duas vezes à aplicação.
4. Registre o ACK perdido, a expiração do temporizador de `13`, a retransmissão seletiva e o ACK repetido.

## 8. Comparação que deve aparecer nas suas conclusões

| Situação | Go-Back-N | Selective Repeat |
|---|---|---|
| Pacote 6 perdido | Descarta fora de ordem e, após timeout, retransmite 6 e pendentes posteriores. | Armazena fora de ordem e retransmite apenas 6. |
| ACK isolado perdido | ACK cumulativo posterior pode resolver a perda sem retransmissão. | O pacote cujo ACK se perdeu pode expirar e ser retransmitido individualmente. |
| ACKs | Cumulativos. | Individuais/seletivos. |
| Buffer no receptor | Em geral, não precisa guardar fora de ordem. | Precisa guardar pacotes fora de ordem dentro da janela. |

Não misture **SACK TCP** (uma extensão opcional do TCP real, questão 5) com o protocolo didático **Selective Repeat**. Ambos comunicam informação seletiva de recebimento, mas não são a mesma especificação nem têm o mesmo formato de pacote.

## Checklist de entrega

- [ ] Expliquei por que UDP `Length` é 48 bytes no caso de 40 bytes DNS e identifiquei os quatro campos UDP.
- [ ] Desenhei a encapsulação Ethernet/IP/UDP/DNS a partir de uma captura, com tamanhos e endereços coerentes.
- [ ] Registrei a evidência visual da convergência das taxas TCP e a expliquei usando AIMD/gargalo.
- [ ] Mostrei a evolução de janela e dos pacotes na questão de slow start.
- [ ] Extraí MSS dos dois lados do handshake, expliquei o sentido de cada anúncio e estimei MTU com os cabeçalhos corretos.
- [ ] Calculei janela anunciada usando `Window scale`, diferenciei SACK negociado de uso efetivo e justifiquei `TCP Segment Len` como cálculo do Wireshark.
- [ ] Executei GBN e SR nos três cenários, registrando perdas, ACKs, timeout e retransmissões.
- [ ] Comparei GBN e SR para as mesmas perdas, sem confundi-los com SACK TCP.
