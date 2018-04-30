> Autor: André Teixeira, nº 48047

# 2 - Sistemas de Comunicação Direta

## Sincronização

Regra geral, o recetor fica *bloqueado* até ser possível receber dados.

Comunicação *assíncrona*:

* O emissor só fica *bloqueado* até o seu pedido de envio ser tomado em consideração.

Comunicação *síncrona*:
* o emissor fica *bloqueado* até:
    * O recetor “receber” os dados – *comunicação síncrona unidireccional*.
    * Receber a resposta do recetor – *comunicação pedido / resposta* ou cliente / servidor.

---

## Persistência

Comunicação *volátil*:

* As mensagens apenas são encaminhadas se o recetor existir e estiver a executar, caso contrário são destruídas.

Comunicação *persistente*:
* As mensagens são guardadas pelo sistema de comunicação até serem consumidas pelos destinatários, que podem não estar a executar.
* Mensagens são guardadas num receptáculo independente do recetor – mailbox, canal, porta persistente, etc.

---

## Fiabilidade

Comunicação *fiável*: 
* O sistema garante a entrega das mensagens em caso de falha temporária.

Comunicação *não-fiável (best-effort)*: 
* Em caso de falha, as mensagens podem perder-se.

---

# 3 - Invoção Remota
 
## RPCs - Automatização (Proxy/Stub Compilers)

* É costume designar-se de *stub do cliente* às funções do cliente que efectuam a comunicação com o servidor para executar o método no servidor.
* Do lado do servidor, o *stub* ou *skeleton* do servidor corresponde ao código de comunicação para esperar as invocações e executá-las, devolvendo o.

Em alguns sistemas e ambientes usam-se ferramentas (compiladores) para gerar os stubs:
* *wsimport* para WebServices SOAP
* Java RMI, Servidor JAX-RS(Jersey)
* .NET Remoting

---

## Java RMI

As interfaces são definidas na própria linguagem Java, usando código standard.

* A interface deve estender java.rmi.Remote
* Todos os métodos devem lançar a excepção java.rmi.RemoteException

---

## WSDL – IDL para Web Services

Definição da interface em XML:

* WSDL permite definir a interface do serviço, indicando quais as mensagens trocadas na interacção.
* WSDL permite também definir a forma de representação dos dados e a forma de aceder ao serviço

---

## Passagem de Parâmetros - *Valor vs. Referência*

Passagem por valor para tipos básicos, arrays, estruturas e objectos não remotos:

* Apontadores/referências para arrays, objectos, etc. são seguidas
* Estado dos objectos é copiado (ex: Java RMI)

Passagem por referência para objectos remotos:

* Quando o tipo de um parâmetro é um objecto remoto, uma referência para o objecto é transferida

> A passagem por referência não faz sentido quando estão envolvidas máquinas distintas

*Explicação*:
* Em Java, quando a invocação é local, os objetos são passados por referência. Efeitos aplicados sobre os parâmetros (não primitivos) permanecem após o retorno dos métodos.
* Quando a invocação é remota (Java RMI), os objectos não remotos são passados por valor (cópia). Efeitos aplicados nas cópias dos parâmetros (objectos não remotos) não são visíveis na máquina que fez a invocação.
* Quando a invocação é remota (Java RMI), os objetos remotos são passados por referência. Efeitos aplicados sobre os parâmetros que sejam objectos remotos permanecem nesses objectos após o retorno.


---

## Codificação de Dados

* *Marshalling* – processo de codificar do *formato interno* para o *formato rede*
* *Unmarshalling* – processo de descodificar do *formato rede* para o *formato interno*

### Aproximações à Codificação de Dados

* *Formato intermédio independente (network standard representation)*
    * Emissor converte da sua representação nativa para a representação da rede
    * O receptor converte da representação da rede para a sua representação
* *Formato do emissor (Receiver makes it right)*
    * Emissor envia usando a sua representação interna, indicando qual ela é
    * Receptor, ao receber, faz a conversão para a sua representação
* *Formato do receptor (Sender makes it right)*

---

## [JSON - JavaScript Object Notation](https://json.org/)

JSON é uma forma de codificação de dados que permite descrever estruturas de dados complexas em formato de texto que suporta

* *Tipos primitivos*
    * Number
    * String
    * Boolean
* *Tipos complexos*
    * Array
    * Object (dicionário chave/valor)    

Como a codificação JSON não preserva os tipos dos dados, a *descodificação* de estruturas complexas *exige saber "navegar" no resultado*. Alternativamente, em Java, existem bibliotecas que *dada a classe do objeto* esperado retornam uma instância quando a *descodificação tem sucesso*

---

##  [ProtoBuf](https://developers.google.com/protocol-buffers/)

É um formato de codificação de dados desenvolvido pela Google.
Difere dos dois anteriores, na medida que os dados passam na rede em *formato binário*.
Tem como objetivo, *reduzir a dimensão* da codificação e *acelerar* o seu processamento.

---

## Impacto do Protocolo na Semântica

### Protocolo TCP / HTTP

O cliente usa uma conexão TCP (ou HTTP) para contactar o servidor, enquanto não receber resposta não avança com outro pedido.

* O cliente fez um pedido e recebeu a resposta:
    * O cliente tem a certeza que o procedimento executou uma e uma só vez
* O cliente fez o pedido e não recebeu resposta nenhuma até um certo timeout.
    * Não se sabe se a operação foi executada ou não.
* O cliente fez o pedido e recebeu uma notificação de que a conexão foi quebrada (por falha na comunicação ou por crash no servidor)
    * Não se sabe se a operação foi executada ou não.

---
### Protocolo UDP

O cliente usa o protocolo UDP para contactar o servidor, enquanto não receber resposta a um pedido não avança com outro pedido.

* O cliente enviou uma só vez o pedido e recebeu a resposta
    * O cliente tem a certeza que o procedimento executou uma e uma só vez (mas apenas se assumirmos que não existe duplicação de pacotes)
* O cliente fez o pedido e não recebeu nenhuma resposta até um certo timeout.
    * Não se sabe se a operação foi executada ou não

---


## Semânticas da invocação remota

* Zero ou uma vez / talvez / *maybe*
    * Variante: Zero ou mais vezes
* Pelo menos uma vez / *at least once*
    * [Operações idempotentes](https://en.wikipedia.org/wiki/Idempotence#Computer_science_meaning)
* No máximo uma vez / *at most once*
* Exatamente uma vez / *exactly once*

*Zero ou mais vezes (talvez)*

* *Não dá quaisquer garantias* 
    * A operação poderá ter sido executada uma vez ou nem ter sido executada.
    * Pouco utilizada. Pode ser usada quando *não são esperados resultados* , por exemplo para implementar notificações e *heart beats não críticos*.


*Pelo menos uma vez (uma ou mais vezes)*
* Se o cliente recebeu a resposta, a operação foi executado uma ou mais vezes.
* Se o cliente não recebeu a resposta, a operação foi executada zero ou mais vezes, obrigando a repetir o pedido.

*No máximo uma vez*
* Se cliente recebeu a resposta, o procedimento foi executado uma e uma só vez.
* Se o cliente não recebeu a resposta, o procedimento foi executado zero ou uma vez.


*Exatamente uma vez*
* Garante-se que a operação é executada exatamente uma vez (nem mais, nem menos), assumindo que as falhas não são permanentes.

---

## Operações Idempotentes

Uma operação é *idempotente* se a sua execução repetida não altera o efeito produzido, deixando o servidor no mesmo estado, ou num estado aplicacionalmente aceitável como equivalente e produzindo o mesmo resultado.

---

### Soluções geralmente adoptadas
* Java RMI
    * **no máximo uma vez** sobre TCP
* .NET Remoting
   *  **no máximo uma vez** sobre TCP, HTTP, pipes
* Web Services
    * **no máximo uma vez** sobre HTTP (SMTP, etc.)
    * WS-Reliability: suporte para “pelo menos uma vez” e “exatamente uma vez”
* REST
    * **no máximo uma vez** sobre HTTP

---

## Relação Threads/Invocações

As relações entre threads e invocações mais comuns são:

* *Thread-per-Request* - cada invocação é executada por um thread
* *Thread-per-Connection* - as invocações de uma conexão são executadas todas pelo mesmo thread
* *Thread-per-Object* - as invocações para um objeto são executadas todas pelo mesmo thread
---

# 4 - Web services e modelos alternativos de interacção cliente/servidor na internet 

## Compenentes Básicos

*SOAP* - protocolo de invocação remota. Fornece um mecanismo que pode ser usado para trocar mensagens entre clientes e servidores da web.
* Oneway
* Pedido-Resposta
* Notificação
* Notificação-Resposta

*WSDL* - linguagem de espeficicação de serviços.
* É um documento xml com todos as carateristicas dos metodos *SOAP*,permite definir interface, representação dos dados, etc.   

*UDDI* - serviço de registo.
* É um Web Service que usa um mecanismo de desoberta de serviços da web, fornece um mecanismo que pode ser usado pra localizar um serviço web/aceder info sobre esse serviço


*TL;DR* - *UDDI* é um Web Service em si próprio, é possível aceder ao *UDDI* usando *SOAP* e a sua interface escuta usando *WSDL*.
Serviços registados em *UDDI* não são obrigatórios ter interface *SOAP* nem descrição *WSDL*.

---

# 5 - Sistemas de comunicação indireta

## Publish/Subscrive : Arquiteturas

* *Centralizadas*
    * Existem servidores, denominados *broker(s)*.
        * Tanto os editores como os assinantes operam como clientes do broker.
    * Permite implementar funcionalidades adicionais com relativa facilidade
        * e.g. - persistência, filtragem baseada no conteúdo, replicação particionamento.
* *Descentralizadas* (P2P)
    * Os editores e os subscritores organizam-se entre si para encaminhar os eventos.
    
---

## Caracterização de Multicast: *Fiabilidade*

*Multicast não fiável (unreliable multicast)* – em caso de falha, não existem garantias sobre a entrega das mensagem aos
vários elementos do grupo.
* e.g. - a mensagem pode ser não entregue a nenhum ou a alguns dos
membros do grupo.
* e.g. - IP mutlicast


*Multicast fiável (reliable multicast)* – uma mensagem enviada para um grupo ou é entregue por todos os membros correctos (que não falham) ou por nenhum

---

## Caracterização de Multicast: *Ordem*

*Sem ordem* – as mensagens podem ser entregues por
diferentes ordens em diferentes processos

*FIFO (First In First Out)* – as mensagens de um processo são entregues
pela ordem de emissão em todos os processos

*Ordem causal* – se *M1* pode causar *M2*, *M1* deve ser entregue
sempre antes de *M2*

*Ordem total* – as mensagens *M1*, *M2* são entregues por ordem
total, se forem entregues pela mesma ordem em todos os
processos

* Implementação pode ser conseguida à custa de um sequenciador.
    * solução centralizada – as mensagens são enviadas ao sequenciador
    e este impõe a ordem (transformando a ordem total em ordem
    FIFO).
    *  solução descentralizada – o sequenciador apenas emite um “stream”
    adicional que estabelece a ordem de entrega a observar em todos os
    receptores.

*Ordem total causal* – ordem total & ordem causal

---

# 6 - Arquiteturas e Modelos de Sistemas Distribuídos

## Arquitetura em Camadas: Modelo Three-Tier

| Nível | Descrição |
|---|---|
| *Apresentação* | Interface utilizador, serve para traduzir tarefas e obter os resultados (input e output do user) |
| *Lógica (Aplicação)* | Coordena os processos comando da aplicação. Faz decisões logicas e avalia, faz calculos e transporta data de processos entre as 2 camadas (*Apresentação*) e (*Armazenamento*) |
| *Armazenamento (Dados)* | Informação aqui é armazenada para uma base da dados ou file system, depois a informação é passada para a parte (*Lógica*) para ser processada |
    
---

## Modelo P2P (Peer to Peer)

*Todos os processos têm funcionalidades semelhantes:*
* Durante a sua operação podem assumir o papel de clientes e servidores do mesmo serviço em diferentes momentos

*Positivo:*
* Não existe ponto único de falha
Melhor potencial de escalabilidade
Baixo custo de operação.

*Negativo:*
* Interação mais complexa (do que num sistema cliente/servidor) leva a
implementações mais complexas
Operações de pesquisa são complexas
Maior número de computadores envolvido

---

## Variantes do Modelo C/S (Cliente/Servidor): *Servidor*

### *Cliente/servidor particionado:*
* Existem vários servidores com a mesma interface, cada um capaz de
 responder a uma parte dos pedidos

    *Positivo:*
    * Permite distribuir a carga, melhorando o desempenho (potencialmente)
    Não existe um ponto de falha único.

    *Negativo:*
    * Falha de um servidor impede acesso aos dados presentes nesse servidor
    difícil de aplicar em alguns modelos de dados.

### *Cliente/servidor replicado:*
* Existem vários servidores idênticos (e.g. capazes de responder aos
mesmo pedidos)

    *Positivo:*
    * Redundância - não existe um ponto de falha único
    Permite distribuir a carga, melhorando o desempenho (potencialmente)

    *Negativo:*
    * Coordenação - manter estado do servidor coerente em todas as réplicas
    Recuperar da falha parcial de um servidor

### *Cliente/servidor geo-replicado:*
* Servidor replicado, com réplicas distribuídas geograficamente

    *Positivo:*
    * Proximidade aos clientes melhora a qualidade de serviço (latência)
    Redundância acrescida – as falhas das réplicas são (ainda) mais
    independentes

    *Negativo:*
    * Coordenação mais dispendiosa – maior separação física das réplicas
    traduz-se na utilização de canais com latência significativa

---

## Variantes do Modelo C/S (Cliente/Servidor): *Cliente*


### *Cliente leve (Thin Client)/Servidor:*
* O cliente apenas inclui uma interface (gráfica) para executar operações
no servidor (ex.: browser)

    *Positivo*
    * Cliente pode ser muito simples

    *Negativo*
    * Maior peso no servidor
    * Impacto na interatividade (latência)

### *Cliente completo (estendido)/servidor:*

O cliente executa localmente algumas operações que seriam
executadas pelo servidor

Usado na Web em vários níveis: 
* Browsers fazem cache de páginas, imagens, etc;
* HTML 5.0 permite às aplicações Web guardar dados
localmente 
    * e.g. suporte offline no Google Docs, Gmail

*Positivo:*
* Permite funcionar offline, quando não é possível contactar o servidor
(recorrendo a caching)
* Permite diminuir a carga do servidor e melhorar o desempenho e a
interatividade

*Negativo:*
* Implementação do cliente mais complexa
* Necessário tratar da coerência dos dados entre o cliente e o servidor

---

## Variantes do Modelo P2P (Peer to Peer)

### Sistema P2P não estruturado

As ligações entre os membros são formadas de forma *não-determinista*.
* e.g. quando se junta à rede, um membro escolhe para vizinhos um pequeno
conjunto de contactos (os contactos podem variar durante a execução do
sistema e de execução para execução).

*Positivo:*
* Simplicidade de construir, robusto ao dinamismo da rede (churn – entrada e
saída de nós participantes).

*Negativo:*
* Produz topologias que podem ser difíceis de explorar de forma eficiente
eg., dificuldade em indexar informação / pesquisa pesada (frequentemente
por inundação).


### Sistemas P2P estruturados

Os membros do sistema comunicam de acordo com uma organização definida de forma determinista com base num *endereço lógico*.

*Positivo:*
* Boa latência/escalabilidade - Conhecem-se topologias com encaminhamento/pesquisa com custo `O(log n)`, com `n` nós no sistema, por exemplo

*Negativo:*
* Maior complexidade - É necessário gastar recursos para manter a topologia correta face às entradas e saídas dos nós (churn)


Permitem implementar tabelas de dispersão distribuídas, e.g., DHTs (Distributed Hash Tables):

* `lookup(chave) -> IP`
* `get(IP, chave) -> valor`
* `put(IP, chave, valor)`

---

## DHT (Distributed Hash Table) - Características
> *identificador = hash(info)*

Aspetos importantes

* Pesquisa? – pesquisa por identificador deve ser eficiente e suportada por
todos os nós.

* Distribuição da informação? – deve ser uniforme por todos os nós.

* Replicação da informação? – a entrada e saída contínua de nós obriga a
manter a informação replicada nos nós certos e em número adequado.

---

## BitTorrent - Versão Original (Simplificado)

### Arquitetura BitTorrent:
* **C/S**
    * *Tracker* - servidor centralizado HTTP que serve o ficheiro `.torrent` e a lista dos peers que estão a descarregar o ficheiro

    * *Peers:* como clientes, acedem ao tracker para obter a lista de outros peers a descarregar o ficheiro

* **P2P**
    * *Peers* - comunicam entre si para trocar blocos do ficheiro
        * Toma-lá dá-cá (tit-for-tat) - Peer só envia bloco em troca de outro bloco
        * Peer envia alguns blocos a outros peers (sem contrapartida - aleatoriamente)
        * Peer descarrega blocos aleatoriamente

---

## Edge-Server

*Sistemas edge-server*
* Existem servidores colocados nos ISP para responder a pedidos e.g., content-distribution networks (CDNs)
    * Propriedades - Menor latência, filtragem, distribuição de carga, etc.

*Nota:*
  *   *Origin server* - Servidor origem de ondem vem a info.
   *  *Akamai*- Servidores distribuidos por vários continentes/países que comunição com o *origin server* e os *End users*, funciona como um proxy em que contem uma cache.
  *   *End users*: clientes que acedem a *akamai*.


*Akamai edge servers só contactam o servidor origem quando*
* O objeto não se encontra na cache
* Se o TTL tiver expirado
* Se objeto se encontrar no-store

---

## Modelo de Interação
* **Ativa** - Processo solicita execução de operação noutro processo
* **Reativa** - Evento no sistema desencadeia ação num processo
* **Indireta** - Processos comunicam através de um espaço partilhado

---

## Algumas Definições
### **Fault tolerance** - Tolerância a Falhas
* Propriedade de um sistema distribuído que lhe permite *recuperar* da existência de falhas *sem introduzir comportamentos incorretos*. Um sistema deste tipo pode mascarar as falhas e continuar *a operar*, ou *parar* e voltar a operar mais tarde, de forma coerente, após reparação da falha

---

### **Availability** – Disponibilidade

* Mede a fracção de tempo em que um serviço está a operar correctamente, isto é, de acordo com a sua especificação.

---

### **Reliability** - Fiabilidade

* Mede o tempo desde um instante inicial até à primeira falha, e.g., o tempo que um sistema funciona correctamente sem falhas.

---

## Tipos de Falhas dos Componentes

* *Falha por Omissão* - dá-se quando um processo ou um canal de
comunicação falha a execução de uma acção que devia executar 

* *Falha Temporal* - dá-se quando um evento que se devia produzir num
determinado período de tempo ocorreu mais tarde – normalmente em
sistemas de tempo real

* *Falha Arbitrária ou Bizantina* - dá-se quando se produziu algo não
previsto.
    * Exemplo: chegou uma mensagem corrompida, um atacante produziu uma mensagem não esperada. Para lidar com estas falhas é necessário garantir que elas não levam a que outros componentes passem a estados incorrectos

---

## Tipos de Erros/Falha: DURAÇÃO

* *Permanentes*: mantêm-se enquanto não forem reparadas (ex:
computador avaria)
Mais fáceis de detectar.
Mais difíceis de reparar.

* *Temporárias*: ocorrem durante um intervalo de tempo limitado,
geralmente por influência externa
Mais difíceis de reproduzir, detectar.
Mais fáceis de reparar.
