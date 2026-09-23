# Roteiro didático - tarefas 3 a 5 (HTTPS no Wireshark)

Este roteiro parte do princípio de que o Wireshark já está instalado e que você está começando a usá-lo. Ele cobre as tarefas 3, 4 e 5 do arquivo `_LabAplic2_HTTPS.pdf`.

## Antes de começar: como ler a tela do Wireshark

Ao abrir uma captura, a tela tem três partes principais:

1. **Lista de pacotes** (parte superior): uma linha por pacote. As colunas mais úteis são `No.`, `Time`, `Source`, `Destination`, `Protocol`, `Length` e `Info`.
2. **Detalhes do pacote** (parte central): ao selecionar uma linha, expanda as setas para examinar protocolos e campos. É aqui que ficam os campos de TLS.
3. **Bytes do pacote** (parte inferior): representação bruta em hexadecimal e texto. Normalmente você não precisará dela neste laboratório.

No alto há a barra **Display Filter**. Ela apenas esconde ou mostra pacotes; não altera o arquivo. Para aplicar um filtro, escreva-o ali e pressione `Enter`. Para voltar a ver tudo, apague o filtro e pressione `Enter`.

> Dica: o Wireshark diferencia filtros de exibição de filtros de captura. Neste roteiro use principalmente filtros de **exibição**.

## 3. Analisar a captura fornecida

### 3.0 Abrir a captura e se orientar

1. Abra o Wireshark.
2. Escolha **File > Open** e abra `lab_3/_LabAplic2_tls-handshake_greer.pcapng`.
3. Clique no pacote 4. No painel central, expanda **Transport Layer Security** e depois os itens internos. Quando precisar localizar algo, use `Ctrl+f`, selecione a opção de procurar nos detalhes do pacote e pesquise pelo nome do campo, por exemplo `supported_versions`.
4. Para retornar rapidamente a um pacote específico, use `Ctrl+g`, informe o número e confirme.

O enunciado chama o arquivo de `_Lab2_tls-handshake-greer.pcap`; nesta pasta, o equivalente disponibilizado é o arquivo `.pcapng` acima.

### 3.a Colorir os pacotes TLS de handshake

O objetivo é destacar os registros TLS cujo tipo de conteúdo é **Handshake (22)**.

1. No pacote 4, expanda **Transport Layer Security**.
2. Expanda a primeira entrada TLS até encontrar **Content Type: Handshake (22)**.
3. Clique nesse campo com o botão direito e escolha **Apply as Filter > Selected**. A barra de filtro deve passar a mostrar `tls.record.content_type == 22` (a forma sem espaços também é equivalente).
4. Com o filtro aplicado, escolha **View > Coloring Rules**.
5. Clique no botão **+** para criar uma regra. Dê um nome como `TLS Handshake`, cole o filtro `tls.record.content_type == 22`, escolha cores de fundo/texto legíveis e confirme com **OK**.
6. Apague o filtro da barra e pressione `Enter`. Os pacotes de handshake continuam coloridos, mas todos os demais voltam a aparecer.

Se a regra não surtir efeito, confira se ela está marcada/ativa e se não existe uma regra anterior que também corresponda ao pacote e tenha prioridade.

### 3.b e 3.c Descobrir as versões TLS

Em TLS 1.3, o campo `Version` visível pode carregar um valor legado para compatibilidade. Por isso, a resposta correta vem dos campos de **Supported Versions**, não apenas do primeiro campo chamado `Version`.

1. No pacote 4 (**Client Hello**), expanda **Transport Layer Security > Handshake Protocol: Client Hello**.
2. Anote o campo `Version`/`Legacy Version` e, principalmente, expanda **Extension: supported_versions**.
3. Em **Supported Version** anote a menor e a maior versão que o cliente oferece. Essas respondem à parte “mais baixa” e “mais alta”.
4. Ainda nesse pacote, observe se há um valor de versão legado. Ele é o “real”/valor efetivamente escrito no campo de versão do Client Hello, conforme a terminologia usada pelo enunciado.
5. Abra o pacote 6 (**Server Hello**) e expanda **Handshake Protocol: Server Hello > Extension: supported_versions**. A versão única escolhida pelo servidor é a resposta da tarefa 3.c e a versão efetivamente negociada.

Registre sempre o nome da versão tal como o Wireshark exibe (por exemplo, `TLS 1.3`), em vez de inferi-la pela cor ou pela coluna `Protocol`.

### 3.d Comparar os Session IDs

1. No **Client Hello** (pacote 4), encontre e copie/anote `Session ID` dentro de **Handshake Protocol: Client Hello**.
2. No **Server Hello** (pacote 6), encontre `Session ID` dentro de **Handshake Protocol: Server Hello**.
3. Compare os valores byte a byte. Responda se são iguais e registre ambos se o relatório pedir evidência.

### 3.e Contar cipher suites e descobrir a escolhida

1. No pacote 4, em **Handshake Protocol: Client Hello**, expanda **Cipher Suites**.
2. O próprio Wireshark normalmente mostra `Cipher Suites (N suites)`: `N` é a quantidade de algoritmos/suites oferecidos pelo cliente. Se não aparecer, conte as entradas listadas.
3. No pacote 6, expanda **Handshake Protocol: Server Hello** e localize `Cipher Suite`.
4. O valor único desse campo é a suite escolhida pelo servidor para a conexão. Anote o nome completo mostrado pelo Wireshark.

Em TLS 1.3, uma cipher suite descreve sobretudo a criptografia simétrica e a função de hash; a troca de chaves aparece em extensões como `key_share`. Para esta questão, use exatamente o campo `Cipher Suite` solicitado.

### 3.f Descobrir o nome do servidor (SNI) e filtrar por ele

1. No pacote 4, expanda **Handshake Protocol: Client Hello > Extension: server_name**.
2. Expanda **Server Name Indication extension** e localize **Server Name**. Esse é o host consultado.
3. Para procurar esse texto na captura, substitua `NOME_ENCONTRADO` pelo host descoberto e use um destes filtros:

   ```wireshark
   tcp contains "NOME_ENCONTRADO"
   ```

   ou

   ```wireshark
   frame contains "NOME_ENCONTRADO"
   ```

`tcp contains` restringe a busca ao conteúdo TCP; `frame contains` procura no quadro inteiro e costuma ser uma boa alternativa se o primeiro não retornar resultado.

### 3.g e 3.h Configurar a descriptografia com o key log

O arquivo fornecido contém segredos de sessão de uma captura específica. Ele permite ao Wireshark decifrar essa captura, mas não é uma chave genérica para todo HTTPS.

1. No Wireshark, abra **Edit > Preferences** (em macOS, **Wireshark > Settings/Preferences**).
2. Na árvore da esquerda, abra **Protocols** e selecione **TLS**.
3. Em **(Pre)-Master-Secret log filename**, clique em **Browse...** e selecione `lab_3/_LabAplic2_sslkeylog.log`.
4. Confirme com **OK**. Se a captura já estava aberta, use **Analyze > Reload** ou feche e abra o arquivo novamente.
5. Na barra de filtro, experimente `http`, `http2` e `tls`. Antes da configuração, os dados de aplicação aparecem como `Application Data`/TLS criptografado; depois, o Wireshark deve conseguir exibir HTTP ou HTTP/2 quando a captura contiver esse tráfego.

Se nada for decifrado, verifique: (a) se o caminho do arquivo está correto; (b) se você recarregou a captura; e (c) se o key log pertence àquela mesma captura.

### 3.i Examinar o pacote 57 e o HTML remontado

1. Vá ao pacote 57 com `Ctrl+g`.
2. No painel de detalhes, expanda as camadas HTTP/HTTP2 e procure uma seção de dados reconstituídos, como `Reassembled TCP Segments`, `Reassembled PDU` ou uma entrada equivalente de **Data**.
3. Clique nos itens e observe as abas/painéis inferiores. Procure a aba **Uncompressed entity body**.
4. Nela, o conteúdo HTML deve aparecer em texto aberto. Use essa visão para confirmar que a descriptografia e a remontagem funcionaram.

“Remontar” significa que o Wireshark juntou dados que chegaram divididos em vários segmentos TCP para reconstruir uma mensagem de aplicação inteira.

## 4. Entender a conexão recusada no pacote 7

1. Abra o pacote 7 e expanda **Transport Layer Security**.
2. Abra **Alert Message** e leia os campos `Level` e `Description`.
3. Use também o contexto: o pacote anterior é um **Client Hello** e, após o alerta fatal, o servidor encerra a conexão com `FIN, ACK`.

Neste exemplo, a evidência é um alerta fatal **Protocol Version**. A explicação provável é que o cliente ofereceu uma versão de TLS que o servidor não aceita (ou não houve uma versão TLS compatível entre os dois). No relatório, cite o campo do alerta e a sequência Client Hello -> Alert fatal -> encerramento TCP como justificativa.

## 5. Capturar e analisar um fluxo HTTPS próprio

### 5.0 Criar um arquivo de chaves de sessão

Feche completamente o navegador antes deste passo. O navegador precisa iniciar com a variável `SSLKEYLOGFILE` definida; defini-la depois não recupera chaves de conexões já abertas.

#### Linux

1. Abra um terminal.
2. Execute, **sem espaços em torno de `=`**:

   ```bash
   export SSLKEYLOGFILE="$HOME/sslkey.log"
   ```

3. Inicie o navegador a partir desse mesmo terminal. Exemplos:

   ```bash
   firefox &
   ```

   ou, conforme o navegador instalado:

   ```bash
   google-chrome &
   # ou chromium &
   ```

4. Depois de abrir um site, confirme que o arquivo foi criado com `ls -l "$HOME/sslkey.log"`. Não publique nem envie esse arquivo: ele pode permitir a leitura do tráfego capturado enquanto suas chaves estiverem nele.

#### Windows

1. No menu Iniciar, pesquise **Editar as variáveis de ambiente para sua conta**.
2. Escolha **Variáveis de Ambiente...** e, em **Variáveis de usuário**, clique em **Novo...**.
3. Crie `SSLKEYLOGFILE` como nome e, por exemplo, `C:\Users\SEU_USUARIO\Desktop\sslkey.log` como valor.
4. Feche todas as janelas do navegador e abra-o novamente. Se ele já estava em execução em segundo plano, encerre-o por completo antes de reiniciar.

### 5.1 Apontar o Wireshark para o novo key log

Em **Edit > Preferences > Protocols > TLS**, informe o mesmo arquivo criado acima em **(Pre)-Master-Secret log filename**. Confirme com **OK**.

### 5.2 Fazer uma captura limpa de `www.ufrgs.br`

1. No Wireshark, volte à tela inicial ou use **Capture > Options**.
2. Selecione a interface que está realmente conectada à Internet. Em geral, ela apresenta atividade em um pequeno gráfico: `Wi-Fi` para rede sem fio ou `Ethernet` para cabo.
3. Clique duas vezes nessa interface, ou selecione-a e clique no ícone de barbatana azul para iniciar a captura.
4. No navegador iniciado pelo terminal/variável, abra uma janela anônima/privada e acesse `https://www.ufrgs.br`.
5. Espere a página carregar e pare a captura pelo botão quadrado vermelho do Wireshark.

Uma janela privada reduz o reaproveitamento de conexões e cache, o que torna mais provável capturar um novo handshake. Se você não encontrar `Client Hello`, feche o navegador, apague apenas a captura atual, abra o navegador novamente com a variável já definida e repita.

### 5.3 Encontrar e isolar o fluxo correto

1. Na barra de filtro, tente:

   ```wireshark
   frame contains "ufrgs"
   ```

   Isso normalmente encontra o nome em uma extensão SNI do Client Hello ou em conteúdo HTTP já decifrado.
2. Clique em um pacote desse resultado e escolha **Follow > TCP Stream** com o botão direito. A janela mostra o fluxo e, ao fechá-la, o Wireshark oferece/aplica um filtro do tipo `tcp.stream eq N`.
3. Mantenha esse filtro para examinar somente a conversa cliente-servidor. Para remover o isolamento, limpe a barra de filtro.

Se `frame contains "ufrgs"` não encontrar nada, localize um pacote TCP para a porta 443 próximo ao horário do acesso, clique nele e use **Follow > TCP Stream**. Também vale tentar `tls.handshake.type == 1` para mostrar Client Hello.

### 5.4 Localizar as mensagens pedidas

Com o fluxo isolado, use estes filtros quando necessário:

| O que procurar | Filtro de exibição útil |
| --- | --- |
| Client Hello | `tls.handshake.type == 1` |
| Server Hello | `tls.handshake.type == 2` |
| Certificate | `tls.handshake.type == 11` |
| HTTP/HTTP2 decifrado | `http or http2` |

Para cada pacote, clique nele e confirme o tipo expandindo **Transport Layer Security**. Anote o número do pacote e o valor pedido; isso facilita montar a resposta depois.

**Atenção à versão do TLS:** se a conexão usar TLS 1.3, é normal não haver mensagens separadas `Server Key Exchange`, `Server Hello Done` e `Client Key Exchange`. Elas são características do handshake TLS 1.2 e anteriores. Em TLS 1.3, procure em vez disso mensagens como **Encrypted Extensions**, **Certificate**, **Certificate Verify** e **Finished**, e registre essa diferença no relatório. Além disso, o tráfego de aplicação pode aparecer como HTTP/2, não necessariamente como HTTP/1.1.

### 5.5 Responder às questões finais

1. **Versão TLS estabelecida (5.f):** abra o **Server Hello**, expanda `supported_versions` e anote a versão selecionada.
2. **Algoritmo escolhido pelo servidor (5.g):** no mesmo **Server Hello**, localize e anote `Cipher Suite`.
3. **Random e Session ID (5.h):** abra cada **Hello** e expanda os respectivos campos `Random` e `Session ID`. Compare/anote os valores, mesmo que a questão não peça uma conclusão sobre eles.

## Checklist de entrega

- [ ] Capturei a tela/registrei os campos que justificam cada resposta.
- [ ] Diferenciei versões oferecidas pelo cliente da versão selecionada pelo servidor.
- [ ] Contei as cipher suites no Client Hello e anotei a escolhida no Server Hello.
- [ ] Configurei o key log e confirmei que conteúdo HTTP/HTTP2 foi decifrado.
- [ ] No exercício próprio, informei se TLS 1.3 substituiu mensagens antigas do enunciado.
- [ ] Não anexei nem compartilhei meu `sslkey.log`.
