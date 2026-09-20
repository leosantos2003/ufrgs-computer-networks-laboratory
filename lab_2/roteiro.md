# Roteiro didático - Wireshark, atraso e HTTP

Este roteiro acompanha o documento `_LabAplic1_Wireshark, Atraso, HTTP.pdf`. Ele foi escrito para quem já instalou o Wireshark, mas ainda não o utilizou. Faça as capturas e anote os valores observados: tempos, endereços e números de pacotes dependem da sua rede e não devem ser copiados de outro computador.

## Antes de começar

1. **Não use uma janela anônima/privada.** Este é um requisito do laboratório.
2. Limpe o cache do navegador antes da primeira consulta: no Chrome/Edge, abra o menu de três pontos, procure por **Privacidade e segurança** e escolha **Limpar dados de navegação**. Marque ao menos os dados/imagens em cache. Feche as abas que possam estar acessando os sites do laboratório.
3. Abra o Wireshark. Na tela inicial há uma lista de interfaces de rede. Use a que mostra atividade no pequeno gráfico:
   - normalmente **Wi-Fi** para rede sem fio;
   - normalmente **Ethernet** para cabo.
4. Não aplique um *capture filter* nesta prática. Capture o tráfego normalmente e use os **filtros de exibição** depois. Assim você não corre o risco de descartar um pacote importante.

> **HTTP versus HTTPS:** o laboratório precisa de HTTP visível. Digite exatamente a URL com `http://`, não `https://`. Se o navegador trocar automaticamente para HTTPS, desative temporariamente a opção “Sempre usar conexões seguras”/“Always use secure connections” nas configurações de segurança, limpe o cache e tente novamente. Se o servidor redirecionar ou a política da rede impedir HTTP, avise o professor e guarde a captura: HTTPS não permite ler o GET e os cabeçalhos sem uma configuração adicional, que não faz parte desta atividade.

### Vocabulário essencial

- **Captura:** registrar os pacotes que passam pela interface escolhida.
- **Filtro de exibição:** texto na barra logo acima da lista de pacotes; apenas esconde/mostra pacotes, sem apagar nada.
- **Frame/pacote:** uma linha da lista de pacotes. O número da linha é o número do frame.
- **Fluxo TCP:** a conversa entre dois pares `IP:porta`. Uma página pode gerar vários fluxos.
- **Tempo relativo:** coluna `Time`, por padrão, medida desde o início da captura.

## 1. Conhecer e preparar o Wireshark

### 1.1 As três áreas da janela

Depois de iniciar uma captura, a janela principal é dividida em três painéis:

1. **Lista de pacotes** (parte superior): uma linha por frame. As colunas mais comuns são `No.`, `Time`, `Source`, `Destination`, `Protocol`, `Length` e `Info`.
2. **Detalhes do pacote** (parte central): ao clicar em uma linha, este painel mostra uma árvore expansível. Clique na seta à esquerda de cada protocolo para ver seus campos.
3. **Bytes do pacote** (parte inferior): mostra o conteúdo bruto em hexadecimal e, quando possível, em ASCII. Ao selecionar um campo no painel central, seus bytes correspondentes ficam destacados aqui.

Clique em um pacote qualquer e experimente expandir `Ethernet II`, `Internet Protocol Version 4`, `Transmission Control Protocol` e `Hypertext Transfer Protocol`. Não altere nada nessa exploração.

### 1.2 Criar um perfil pessoal

Perfis guardam preferências, colunas e regras de cores. Assim suas mudanças não afetam o perfil padrão.

1. No canto inferior direito, localize **Profile** (ou o nome do perfil atual).
2. Clique nele com o botão direito e escolha **New**/**Novo**.
3. Dê um nome que comece pelo seu nome ou identificador, por exemplo `Leonardo-lab-redes`.
4. Selecione esse perfil e continue todo o laboratório nele.
5. Para levá-lo a outra máquina, abra **Manage Profiles**/**Gerenciar perfis** e use a opção de exportação do seu perfil.

Os nomes de menus podem variar um pouco entre versões e idiomas; procure os equivalentes de *Profile*, *Preferences* e *Coloring Rules*.

### 1.3 Ajustar o layout

Abra **Edit > Preferences > Appearance > Layout** (ou **Editar > Preferências > Aparência > Layout**). Escolha uma disposição que deixe a lista em cima, os detalhes no meio e os bytes embaixo. A recomendação para iniciantes é manter essa ordem, porque ela segue o caminho “selecionar, interpretar, conferir os bytes”. Clique em **OK** para salvar no seu perfil.

### 1.4 Criar regras de cor

As cores facilitam encontrar pacotes, mas não mudam a captura.

#### Pacotes enviados pela sua máquina

1. Descubra o IP local em uma linha que você saiba que foi enviada pelo seu computador; o valor na coluna **Source** será seu IP. Alternativamente, em um pacote HTTP, o IP/porta de origem que usa uma porta alta (por exemplo, `52341`) costuma ser o cliente.
2. Abra **View > Coloring Rules** (**Exibir > Regras de coloração**).
3. Clique em **+** para adicionar uma regra.
4. Dê o nome `Saida-meu-IP`.
5. Em **Filter**, escreva `ip.src == SEU_IP`, trocando `SEU_IP` pelo valor encontrado, por exemplo `ip.src == 192.168.1.23`.
6. Escolha uma cor de fundo clara e uma cor de texto legível, confirme e observe os pacotes do seu computador mudarem de cor.

#### Pacotes SYN

Crie outra regra do mesmo modo, com nome `TCP-SYN` e filtro:

```
tcp.flags.syn == 1
```

Esse filtro atende ao enunciado e colore tanto o SYN inicial quanto um possível SYN/ACK. Se quiser destacar **somente** o primeiro SYN de cada conexão, use a versão mais específica abaixo:

```
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

### 1.5 Adicionar a coluna Delta

O Delta é o tempo desde o pacote exibido imediatamente antes. Ele muda quando você muda o filtro de exibição, pois “exibido” significa visível no momento.

Há duas formas de adicioná-lo:

- Em **Edit > Preferences > Appearance > Columns**, adicione uma coluna cujo tipo seja **Delta time displayed**; ou
- selecione um pacote, procure no painel de detalhes o campo **Time delta from previous displayed frame**, clique nele com o botão direito e escolha **Apply as Column**/**Add as Column**.

O campo interno correspondente é `frame.time_delta_displayed`.

## 2. Primeira captura: `INTRO-wireshark-file1.html`

URL da atividade:

```
http://gaia.cs.umass.edu/wireshark-labs/INTRO-wireshark-file1.html
```

### 2.1 Fazer e salvar a captura

1. Volte à tela inicial do Wireshark ou escolha **Capture > Options**.
2. Marque a interface ativa e inicie a captura com o ícone da barbatana azul ou com um duplo clique na interface.
3. Abra uma aba normal do navegador e cole a URL acima. Espere a página terminar de carregar.
4. Volte ao Wireshark e pare a captura pelo quadrado vermelho.
5. Salve-a imediatamente em **File > Save As**, por exemplo como `intro-http.pcapng`. O formato padrão `.pcapng` é adequado.

Na barra de filtro, digite `http` e pressione Enter. Se aparecerem muitos resultados, use um filtro mais preciso:

```
http.host == "gaia.cs.umass.edu"
```

Para ver somente a requisição desse arquivo:

```
http.request && http.request.uri contains "INTRO-wireshark-file1.html"
```

Para voltar a enxergar tudo, limpe a barra de filtro com o `x` à direita ou apague o texto e pressione Enter.

### 2.2 Como responder às questões

#### a) Protocolos e modelo de camadas da Internet

Selecione o GET HTTP e olhe a árvore de detalhes. Em uma captura Ethernet/IPv4 típica, relacione as entradas assim:

| Camada do modelo Internet | O que aparece no pacote |
| --- | --- |
| Aplicação | HTTP: a linha `GET ... HTTP/1.1` e os cabeçalhos, como `Host` |
| Transporte | TCP: portas de origem/destino, números de sequência e confirmações |
| Internet | IPv4 (ou IPv6, se for o caso): endereços IP de origem e destino |
| Enlace | Ethernet II: endereços MAC de origem e destino |

O frame também tem metadados de captura, e a camada física não aparece como um cabeçalho separado no Wireshark. Escreva os protocolos que **a sua** árvore mostrar; se houver IPv6 em vez de IPv4, registre IPv6.

#### b) Formação do pacote

Use o mesmo GET e expanda cada nível. Sua explicação pode seguir este modelo, substituindo pelos valores observados:

> O frame nº ___ contém uma trama Ethernet com MAC de origem ___ e destino ___. Dentro dela há um pacote IPv4/IPv6 de ___ para ___. O segmento TCP usa a porta de origem ___ e a porta de destino 80. A carga útil TCP é uma requisição HTTP `GET` para o recurso ___.

Não confunda o IP do servidor com o MAC do servidor: numa rede doméstica, o MAC de destino provavelmente é o do roteador, que encaminha o pacote até o servidor.

#### c) RTT aproximado da conexão

1. Localize o início da conexão com o filtro:

   ```
   tcp.flags.syn == 1 && tcp.flags.ack == 0
   ```

2. Ache o SYN cuja origem seja sua máquina e cujo destino seja o servidor da página. Anote o `Time`, os IPs e as portas TCP.
3. Localize o SYN/ACK correspondente: os IPs e portas estarão invertidos e as flags serão `SYN, ACK`. Um filtro auxiliar é:

   ```
   tcp.flags.syn == 1 && tcp.flags.ack == 1
   ```

4. Subtraia o tempo do SYN do tempo do SYN/ACK. Essa diferença é uma boa estimativa do RTT daquela conexão TCP. O Wireshark também pode mostrar um campo de análise chamado `tcp.analysis.initial_rtt` em alguns pacotes; use-o como conferência, se aparecer.

Registre também os números dos dois frames para que o cálculo seja verificável.

#### d) Atraso entre o GET e o `200 OK` correspondente

1. Encontre o GET do arquivo com o filtro de URI mostrado acima.
2. Clique nele com o botão direito e escolha **Follow > TCP Stream**/**Seguir > Fluxo TCP**. O Wireshark aplicará algo semelhante a `tcp.stream eq N`; anote o valor `N` e feche a janela do fluxo.
3. Mantenha o filtro do fluxo e acrescente `&& http`, ficando, por exemplo:

   ```
   tcp.stream eq 3 && http
   ```

4. No mesmo fluxo, localize a primeira resposta `HTTP/1.1 200 OK` após o GET. Confirme que ela responde ao recurso solicitado observando a ordem no fluxo e o `Host`/URI do GET.
5. Subtraia o `Time` do GET do `Time` da resposta. Esse é o atraso pedido. A coluna Delta pode ajudar, mas prefira a subtração direta dos tempos, pois pode haver outros pacotes entre os dois.

#### e) Tamanho do arquivo baixado

Selecione o `200 OK`, expanda **Hypertext Transfer Protocol** e procure o cabeçalho **Content-Length**. O valor, em bytes, é o tamanho da entidade/arquivo HTTP baixada. Não use a coluna `Length` do frame: ela inclui cabeçalhos e pode representar apenas uma parte do arquivo.

Se não houver `Content-Length` e aparecer `Transfer-Encoding: chunked`, use **File > Export Objects > HTTP**, selecione o objeto correspondente e verifique o tamanho na janela de exportação. Registre o método usado.

### 2.3 Registro sugerido para a primeira captura

```text
Interface usada: ______________________________
GET: frame ___, tempo ___ s, fluxo TCP ___
200 OK: frame ___, tempo ___ s
RTT: SYN frame ___ (___ s) -> SYN/ACK frame ___ (___ s) = ___ s
Atraso GET -> 200 OK: ___ s
Content-Length/tamanho do arquivo: ___ bytes
```

## 3. Segunda captura: `HTTP-wireshark-file3.html`

URL da atividade:

```
http://gaia.cs.umass.edu/wireshark-labs/HTTP-wireshark-file3.html
```

Faça uma nova captura para manter as respostas organizadas. Antes da primeira carga deste arquivo, limpe novamente o cache. Inicie a captura, abra a URL, espere terminar, pare e salve como `file3-primeira-carga.pcapng`.

O filtro mais útil é:

```
http
```

Para isolar uma conversa, escolha um pacote dela, use **Follow > TCP Stream**, anote o número do fluxo, feche a janela e acrescente `&& http` ao filtro `tcp.stream eq N`.

### 3.1 Perguntas sobre a primeira carga

1. **Código de status:** selecione a resposta ao GET e leia a primeira linha do HTTP, por exemplo `HTTP/1.1 200 OK`. O código também pode ser filtrado por `http.response.code`.
2. **Última modificação:** no mesmo pacote, expanda HTTP e copie o valor do cabeçalho `Last-Modified` (data, hora e fuso, normalmente GMT).
3. **Tamanho do cabeçalho HTTP da resposta:** o cabeçalho começa no primeiro caractere da linha de status (`HTTP/...`) e termina no segundo `CRLF` após o último cabeçalho, **incluindo** esse `CRLF` final. Não inclua o corpo do arquivo, nem os cabeçalhos TCP/IP/Ethernet.
   - Para a resposta pequena esperada nesta atividade, ela normalmente cabe em um único segmento TCP. Expanda **Transmission Control Protocol** e anote o valor de **TCP payload length** (ou `tcp.len`). Depois anote `Content-Length` no HTTP. Calcule: **tamanho do cabeçalho HTTP = `tcp.len - Content-Length`**. Exemplo: carga TCP de 410 bytes e `Content-Length: 128` significam cabeçalho HTTP de 282 bytes.
   - Confira visualmente no painel **Packet Bytes**: os bytes do cabeçalho terminam na linha vazia imediatamente antes do HTML/corpo.
   - Se a resposta vier segmentada ou em *chunked encoding*, não some simplesmente os tamanhos dos frames. Use **Follow > TCP Stream**, visualize em **Raw** e conte somente o bloco de cabeçalhos até a linha em branco (cada quebra HTTP `CRLF` ocupa 2 bytes). A reassemblagem exibida pelo Wireshark reúne o cabeçalho corretamente.
4. **Tamanho do arquivo:** leia `Content-Length` no `200 OK`, como na seção 2.2(e). Em resposta *chunked*, use **File > Export Objects > HTTP**.

Anote o frame da resposta para cada item. Isso mostra de onde cada resposta foi obtida.

### 3.2 Segunda carga e cache (item 3e)

Sem apagar o cache, ainda com a captura em andamento, volte ao navegador e pressione **F5**. Espere a atualização e pare/salve a captura, por exemplo `file3-f5.pcapng`.

Filtre novamente por `http` e compare a primeira carga com a atualização:

- Se houver um novo GET seguido de `200 OK`, o servidor enviou o conteúdo de novo.
- Se houver um GET com cabeçalhos condicionais, como `If-Modified-Since`, e a resposta for `304 Not Modified`, o navegador perguntou se a versão armazenada ainda vale. O servidor informou que ela não mudou, então o navegador reutiliza o corpo que já estava no cache.
- Se não houver novo GET, o navegador considerou a entrada do cache ainda válida e usou o arquivo localmente. Anote essa evidência; não é erro da captura.

O ponto principal é explicar **se** houve consulta ao servidor e **por que** o navegador baixou ou reutilizou o conteúdo. O navegador pode estimar a validade do cache usando a data `Last-Modified`, como menciona o enunciado.

## 4. Animação de estimativa de atraso HTTP

Abra:

```
https://media.pearsoncmg.com/aw/ecs_kurose_compnetwork_7/cw/content/interactiveanimations/http-delay-estimation/index.html
```

Se o navegador bloquear a animação por conteúdo desatualizado, registre o erro e peça ao professor uma alternativa; não é necessário tentar contornar proteções do navegador.

### 4.1 HTTP não persistente (HTTP/1.0)

1. Escolha **non-persistent connections**.
2. Defina **4 objects**.
3. Defina o atraso de transmissão de cada objeto para **1 RTT**.
4. Execute a animação e registre a linha do tempo/tempo total mostrado.

Comentário esperado: uma conexão TCP não persistente usa uma nova conexão para cada objeto. Em série, para cada objeto há tipicamente 1 RTT para estabelecer TCP, 1 RTT para enviar GET/começar a receber a resposta, mais o tempo de transmissão. Com quatro objetos, repetir o estabelecimento torna o total maior. HTTP/1.0 também podia abrir conexões paralelas (por exemplo, seis); isso reduz o tempo percebido ao baixar objetos simultaneamente, mas cria mais conexões e concorrência na rede/servidor.

### 4.2 HTTP persistente com pipeline (HTTP/1.1)

1. Escolha **persistent connections with pipeline**.
2. Mantenha 4 objetos e 1 RTT de transmissão por objeto.
3. Execute novamente e compare com o cenário anterior.

Comentário esperado: uma única conexão TCP é estabelecida e reutilizada. Com *pipeline*, várias requisições podem ser colocadas na conexão sem aguardar individualmente cada resposta, eliminando a repetição do *handshake*. Porém as respostas permanecem ordenadas: se um objeto anterior atrasar ou se perder, os seguintes podem ficar retidos. Esse é o **Head-of-Line Blocking** do pipeline HTTP/1.1.

### 4.3 Debate: RTTs e perdas

Use esta síntese para organizar a resposta, sempre conferindo o tempo que a animação realmente apresentou:

| Solução | Conexões TCP | RTTs de estabelecimento | Efeito de uma perda |
| --- | --- | --- | --- |
| Não persistente, em série | Uma por objeto | Repetido para cada objeto | Atraso fica associado à conexão/objeto perdido; objetos posteriores em série já aguardavam mesmo |
| Não persistente, em paralelo | Várias simultâneas | Um por conexão, mas sobrepostos no tempo | Um objeto pode atrasar sem necessariamente parar os outros; há mais competição por recursos |
| Persistente com pipeline | Uma reutilizada | Pago uma vez | A resposta perdida/atrasada na frente pode bloquear as respostas seguintes em ordem |
| HTTP/2 multiplexado | Uma TCP reutilizada, vários fluxos HTTP | Pago uma vez | Evita o bloqueio na camada HTTP entre objetos, mas uma perda TCP ainda pode atrasar dados de todos os fluxos da conexão |

Não escreva apenas “precisa de X RTTs” sem dizer a hipótese. O número depende de se os objetos são buscados em série ou em paralelo, se a conexão TCP já existe, do tempo de transmissão e de haver perda. Para a configuração pedida, explique como a animação chegou ao total observado e compare os dois casos.

## Filtros rápidos para consulta

Cole estes filtros na barra de exibição e pressione Enter:

```text
http                                      # todos os pacotes HTTP reconhecidos
http.request                              # requisições HTTP
http.response                             # respostas HTTP
http.response.code == 200                 # respostas 200 OK
http.response.code == 304                 # respostas 304 Not Modified
http.host == "gaia.cs.umass.edu"          # host do laboratório
http.request.uri contains "file3.html"    # requisição do segundo arquivo
tcp.stream eq N                           # substitua N pelo número do fluxo
tcp.flags.syn == 1 && tcp.flags.ack == 0  # SYN inicial de uma conexão
tcp.flags.syn == 1 && tcp.flags.ack == 1  # SYN/ACK
```

Se um filtro ficar vermelho, ele contém erro de sintaxe ou não é aceito na sua versão. Apague-o, comece por `http` e vá acrescentando partes; o Wireshark oferece sugestões enquanto você digita.

## Checklist de entrega

- [ ] Perfil pessoal criado e selecionado.
- [ ] Layout, duas regras de cor e coluna Delta configurados.
- [ ] Captura da primeira URL salva, com frames do GET, 200 OK, RTT, atraso e tamanho identificados.
- [ ] Captura da segunda URL salva, com status, `Last-Modified`, tamanho de cabeçalho, tamanho de arquivo e teste de F5 explicados.
- [ ] Dois cenários da animação executados e comparados.
- [ ] Em cada resposta numérica, foram anotados os números dos frames ou o filtro/fluxo que permite reencontrá-la.
