# Roteiro didático - Laboratório DNS

Este roteiro segue o enunciado `_LabAplic3_DNS.pdf`. A ideia é que você não apenas obtenha as respostas, mas consiga associar cada observação a uma parte do protocolo DNS. Guarde capturas de tela do `nslookup`/`dig` e dos pacotes relevantes no Wireshark: elas servem como evidência das suas conclusões.

> Os endereços IP, TTLs e respostas podem mudar com o tempo. Quando a atividade pedir uma resposta observada, registre o valor que aparecer na sua própria consulta, incluindo data e hora.

## 0. Preparação

1. Abra o Wireshark e selecione a interface que realmente está conectada à rede (Wi-Fi ou Ethernet). Gere um pouco de tráfego antes de começar; se os contadores subirem, a interface está correta.
2. Inicie a captura. Como filtro de **captura**, você pode usar `udp port 53`; como filtro de **exibição**, use `dns`. O filtro de captura limita o que é gravado, enquanto o de exibição apenas esconde/mostra pacotes já capturados.
3. Abra um terminal (Linux) ou Prompt de Comando/PowerShell (Windows). Os comandos abaixo são alternativas: use os da sua plataforma.
4. Caso uma resposta venha do cache e você precise repetir o experimento, limpe o cache. O enunciado sugere:

   ```bash
   # Linux (quando esse serviço existir)
   sudo systemctl restart nslcd.service

   # Windows, Prompt de Comando como administrador
   ipconfig /flushdns
   ```

   Em distribuições Linux atuais que usam `systemd-resolved`, o equivalente mais comum é `sudo resolvectl flush-caches`. Não execute comandos de limpeza se você não tiver permissão administrativa; primeiro tente a consulta normalmente.

## 1. Aquecimento: o que o DNS resolve

Antes de executar comandos, revise estes papéis:

- O DNS transforma nomes de domínio em dados de recursos, especialmente endereços IP. O registro `A` contém IPv4 e o `AAAA`, IPv6.
- O computador normalmente pergunta ao seu resolvedor local (roteador, provedor, universidade ou servidor corporativo), que pode responder do cache ou consultar outros servidores.
- A hierarquia usual é raiz (`.`) -> TLD (por exemplo, `.br`) -> servidor autoritativo da zona (`ufrgs.br`).
- Uma consulta direta/reversa não é a mesma coisa: a direta parte do nome; a reversa parte do IP e consulta uma zona `in-addr.arpa` (IPv4), em busca de um registro `PTR`.

Use essa revisão para explicar, com suas palavras, o objetivo do DNS no relatório.

## 2. Animação: DNS recursivo versus iterativo

Abra a animação indicada no enunciado:

<https://media.pearsoncmg.com/aw/ecs_kurose_compnetwork_7/cw/content/interactiveanimations/recursive-iterative-queries-in-dns/index.html>

Faça quatro rodadas. Em cada uma, escolha primeiro uma das situações de cache e depois teste **todas** as configurações oferecidas pela animação para os demais servidores:

| Rodada | Estado inicial a escolher |
|---|---|
| A | `Root name server has the Destination IP cached` |
| B | `Root name server has the Authoritative name server cached` |
| C | `Root name server has an Intermediate name server cached` |
| D | `Local name server has the Destination IP cached` |

Para cada execução, anote: quem recebeu a consulta, se retornou a resposta final ou uma referência, quantas mensagens foram trocadas e em qual ponto o cache eliminou consultas. Uma tabela curta como esta ajuda:

| Rodada | Modo do local | Modo do root | Caminho percorrido | Quem devolve o IP ao cliente | Efeito do cache |
|---|---|---|---|---|---|
| A |  |  |  |  |  |

Interprete corretamente os termos da simulação:

- A máquina solicitante sempre pede uma consulta **recursiva** ao servidor local, como ocorre normalmente na prática.
- Uma consulta **recursiva** pede: “obtenha a resposta final para mim”.
- Uma consulta **iterativa** recebe uma referência ao próximo servidor e prossegue com ele por conta própria.
- Na simulação, as opções recursivo/iterativo definem como **aquele servidor** consulta o próximo nível, não alteram a natureza do pedido inicial do host.
- O comportamento mais parecido com o real é servidor local e root usando consultas iterativas entre si, pois isso distribui melhor o trabalho no núcleo da Internet.

Quando chegar à parte de Wireshark, relacione isso às flags DNS: `RD` (Recursion Desired) mostra que o cliente desejou recursão; `RA` (Recursion Available), presente em uma resposta, informa que o servidor oferece esse serviço.

## 3. `gaia.cs.umass.edu`, comparação e WHOIS

Execute uma resolução dos dois nomes. Exemplos:

```bash
# Linux
dig +short gaia.cs.umass.edu A
dig +short www-net.cs.umass.edu A

# Windows
nslookup gaia.cs.umass.edu
nslookup www-net.cs.umass.edu
```

Também é válido usar `ping gaia.cs.umass.edu`; observe o IP que ele mostra, mas interrompa após uma ou duas respostas para não gerar tráfego desnecessário.

Compare os resultados:

- Se ambos retornarem o mesmo IP, os dois nomes apontam para o mesmo destino naquele momento. Isso pode representar aliases, hospedagem virtual ou um mesmo servidor/serviço; somente a igualdade de IP não prova que sejam exatamente a mesma máquina física.
- Se retornarem IPs diferentes, registre os dois e lembre que DNS pode usar balanceamento de carga e respostas variáveis.

Para a parte de WHOIS, use um serviço como <https://www.whois.com/whois/>. Pesquise `umass.edu` para contextualizar `gaia.cs.umass.edu` (subdomínios normalmente não têm um registro de registro independente) e `ufrgs.br`. Anote apenas informações relevantes, como registrador, contatos administrativos/técnicos quando públicos, datas e servidores de nomes. Não exponha dados pessoais que o serviço eventualmente apresente sem necessidade.

## 4. Consulta normal, captura e leitura do DNS

### 4.1 Gere a consulta

Com a captura já em andamento, execute:

```bash
nslookup www.ufrgs.br.
```

O ponto final torna o nome absoluto (FQDN) e impede que um sufixo de busca local seja acrescentado. Pare a captura e aplique `dns`. Em alguns sistemas haverá uma consulta reversa e consultas `A` e `AAAA`; para as respostas seguintes, escolha um par de consulta/resposta `A` ou `AAAA` e mantenha essa escolha consistente.

Se não aparecer DNS em texto claro, confirme o servidor configurado: redes modernas podem empregar DNS sobre HTTPS/TLS, que não aparece como pacote DNS UDP comum. O exercício espera a consulta DNS convencional ao resolvedor local.

### 4.2 Configure a resolução de nomes no Wireshark

Vá em **Editar > Preferências > Name Resolution** e marque as quatro opções solicitadas. O significado é:

| Opção | Efeito |
|---|---|
| `Resolve MAC addresses` | Troca endereços MAC por nomes/identificadores conhecidos quando possível. |
| `Resolve transport names` | Troca números de porta por nomes de serviços, como `53` por `domain`. |
| `Resolve network (IP) addresses` | Tenta mostrar nomes para endereços IP. Pode disparar resolução de nomes fora da captura. |
| `Use captured DNS packet data for name resolution` | Usa as respostas DNS presentes no próprio arquivo para dar nomes aos IPs vistos. |

Compare a lista de pacotes antes e depois: colunas e detalhes podem passar de valores numéricos para nomes legíveis. Em seguida, clique com o botão direito sobre o seu IP e use **Edit Resolved Name** para criar, por exemplo, `Cliente` ou `EU`. Isso só rotula a visualização local do Wireshark; não altera o DNS real.

### 4.3 Responda observando o par de pacotes

Abra a consulta e a resposta correspondentes. Use o mesmo `Transaction ID` e as portas UDP efêmeras para confirmar que formam um par.

- **c) Transporte:** na captura usual, DNS usa `UDP`. O DNS também pode usar `TCP`, por exemplo em transferências de zona, respostas truncadas ou outros casos específicos; responda com base no pacote que você capturou.
- **d) Como o computador conhece o DNS local:** veja a configuração recebida por DHCP ou definida manualmente. Use `resolvectl status` no Linux ou `ipconfig /all` no Windows e registre o(s) servidor(es) DNS mostrado(s).
- **e) Porta padrão:** `53` (UDP na consulta típica; TCP também usa a porta 53 quando aplicável).
- **f) Transaction ID:** identifica uma transação DNS e permite ao cliente associar a resposta certa à consulta certa, mesmo com consultas simultâneas.
- **g) Flags:** `Response`/`QR` vale 0 na consulta e 1 na resposta. `Recursion Desired`/`RD` vale 1 quando o cliente pede que o resolvedor faça a resolução recursiva; a resposta normalmente ecoa essa intenção. Não confunda `RD` com `RA`, que informa que recursão está disponível no servidor.
- **h) Campo `Queries`:** a consulta IPv4 pede tipo `A` (tipo numérico 1); a IPv6 pede `AAAA` (tipo 28). O nome consultado pode ser igual.
- **i) Campo `Answers`:** a resposta `A` contém endereço IPv4; a `AAAA`, endereço IPv6. Registre os endereços e TTL exibidos na sua captura.

## 5. Explique a captura `_LabAplic3_dns_apoio.pcapng`

O enunciado chama o arquivo de `dns_apoio.pcap`, mas a cópia da pasta é `_LabAplic3_dns_apoio.pcapng`. Abra-a no Wireshark e aplique o filtro `dns`. Em vez de explicar cada linha isoladamente, agrupe consulta e resposta pelo `Transaction ID`: há sete transações, totalizando catorze linhas DNS.

| ID | Consulta | Resultado da resposta | Interpretação |
|---:|---|---|---|
| 1 | `PTR 9.11.54.143.in-addr.arpa` | `PTR dhcp.inf.ufrgs.br` | Consulta reversa do IP `143.54.11.9`. |
| 2 | `A www.ufrgs.br.inf.ufrgs.br` | `NXDOMAIN`, SOA de `inf.ufrgs.br` | O sufixo de busca local foi anexado e criou um nome inexistente. |
| 3 | `AAAA www.ufrgs.br.inf.ufrgs.br` | `NXDOMAIN`, SOA de `inf.ufrgs.br` | Mesmo problema da transação 2, agora para IPv6. |
| 4 | `A www.ufrgs.br.ufrgs.br` | `NXDOMAIN`, SOA de `ufrgs.br` | Outra tentativa com sufixo anexado, também inexistente. |
| 5 | `AAAA www.ufrgs.br.ufrgs.br` | `NXDOMAIN`, SOA de `ufrgs.br` | Versão IPv6 da transação 4. |
| 6 | `A www.ufrgs.br` | Resposta com `A 143.54.2.20` | Resolução IPv4 bem-sucedida na captura fornecida. |
| 7 | `AAAA www.ufrgs.br` | Resposta com `AAAA 2804:1f20:0:1::20` | Resolução IPv6 bem-sucedida na captura fornecida. |

Para cada linha, confirme no painel de detalhes: IP/porta de origem e destino, ID, flag `Response`, `Queries`, `Answers`, `Authority` e código de resposta. Nas respostas `NXDOMAIN`, a seção `Authority` traz um `SOA`: ela indica a zona autoritativa e ajuda o resolvedor a fazer cache negativo. Os checksums UDP marcados como incorretos apenas nos pacotes enviados pelo cliente são compatíveis com checksum offloading da placa de rede durante a captura; não conclua automaticamente que houve erro na rede.

Essa captura também demonstra por que o ponto final em `www.ufrgs.br.` é importante: ele evita tentativas como `www.ufrgs.br.inf.ufrgs.br`.

## 6. Obtenha resposta autoritativa para `www.inf.ufrgs.br`

Primeiro descubra um servidor de nomes da zona e depois consulte **diretamente** esse servidor. No Linux, o caminho mais fácil é:

```bash
dig inf.ufrgs.br. NS +short
dig @ns1.inf.ufrgs.br. www.inf.ufrgs.br. A
```

No resultado do segundo comando, confira a flag `aa` (Authoritative Answer). Para enfatizar que não quer recursão, você pode testar também:

```bash
dig +norecurse @ns1.inf.ufrgs.br. www.inf.ufrgs.br. A
```

No Windows, uma sequência equivalente no `nslookup` é:

```text
nslookup
set type=ns
inf.ufrgs.br.
server ns1.inf.ufrgs.br.
set type=a
www.inf.ufrgs.br.
exit
```

Outra forma concisa é `nslookup www.inf.ufrgs.br. ns1.inf.ufrgs.br.`. Anote os comandos que funcionaram, o IP devolvido e a evidência de autoridade (`aa` no `dig` ou a captura da resposta direta no Wireshark). Não basta consultar o resolvedor padrão: ele pode devolver uma resposta cacheada e não autoritativa.

## 7. Interprete `dhcp.inf.ufrgs.br` e `ns1.inf.ufrgs.br` com precisão

A captura/figura permite afirmar o que os **registros DNS** dizem, sem inferir funções apenas pelo nome:

- `dhcp.inf.ufrgs.br` aparece como dado de um registro `PTR`: é o nome associado ao IP `143.54.11.9` pela consulta reversa `9.11.54.143.in-addr.arpa`. O rótulo `dhcp` sugere uma função, mas o que se sabe com certeza a partir do DNS é essa associação nome-IP reversa.
- `ns1.inf.ufrgs.br` aparece no registro `SOA` da zona `inf.ufrgs.br` como o servidor de nomes primário (`MNAME`) da zona. O SOA também traz o responsável técnico em formato de nome DNS e os temporizadores/serial da zona.

No Wireshark, expanda `Domain Name System` > `Authoritative nameservers`/`SOA` ou `Answers`, conforme o pacote, para apontar exatamente de qual campo veio cada afirmação.

## 8. Faça uma consulta reversa para `143.54.11.34`

Com a captura ativa, execute:

```bash
nslookup 143.54.11.34
```

Depois filtre por `dns` e localize o par `PTR`. Para IPv4, o DNS inverte os octetos e consulta a zona `in-addr.arpa`; portanto, o nome consultado deve ser:

```text
34.11.54.143.in-addr.arpa
```

No pacote de resposta, copie exatamente o valor do `PTR` (esse é o nome obtido) e o TTL. DNS reverso serve para mapear IPs em nomes, algo útil em logs, diagnóstico, auditoria, controles de e-mail e apresentação mais legível de conexões. Ele não garante, sozinho, identidade ou confiabilidade de uma máquina: é um dado DNS administrado pela zona reversa.

## Checklist de entrega

- [ ] Explicação do objetivo do DNS, recursão e iteração.
- [ ] Tabela/anotações das quatro rodadas da animação.
- [ ] Comparação dos IPs de `gaia.cs.umass.edu` e `www-net.cs.umass.edu`, com interpretação cuidadosa.
- [ ] Dados relevantes de WHOIS para `umass.edu` e `ufrgs.br`.
- [ ] Captura do `nslookup www.ufrgs.br.` com um par consulta/resposta marcado.
- [ ] Respostas c-i justificadas por campos do pacote, não só por teoria.
- [ ] Explicação das sete transações da captura de apoio.
- [ ] Comandos e prova de uma resposta autoritativa para `www.inf.ufrgs.br`.
- [ ] Consulta reversa de `143.54.11.34`, com QNAME e resposta PTR anotados.
