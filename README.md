# System Design do Zero

## Motivação

- Aprender a desenhar sistemas de larga escala
- Se preparar para entrevistas de system design
- Conhecer e expandir seus conhecimentos do componentes de system design, tendo em mente benefícios e trade-offs
- Se tornar um engenheiro de software melhor
- Disponibilizar conteúdo sobre assunto em português

> Este repositório também serve para processo de aprendizado do autor

> Ao longo desta leitura, indicarei outros materiais. Alguns estarão em inglês.

## Sumário

## Sumário

- [System Design do Zero](#system-design-do-zero)
  - [Motivação](#motivação)
  - [Sumário](#sumário)
  - [Sumário](#sumário-1)
  - [O que é system design?](#o-que-é-system-design)
  - [Latência e throughput (vazão)](#latência-e-throughput-vazão)
  - [Performance e escalabilidade](#performance-e-escalabilidade)
    - [Escalabilidade horizontal e vertical](#escalabilidade-horizontal-e-vertical)
  - [Teorema CAP](#teorema-cap)
    - [CP (Consistência e tolerância a partição)](#cp-consistência-e-tolerância-a-partição)
    - [AP (Disponibilidade e tolerância a partição)](#ap-disponibilidade-e-tolerância-a-partição)
    - [Referência](#referência)
  - [Banco de dados](#banco-de-dados)
    - [Relacional (SQL)](#relacional-sql)
      - [Conceitos Fundamentais](#conceitos-fundamentais)
        - [ACID](#acid)
        - [Normalização vs Desnormalização](#normalização-vs-desnormalização)
      - [Performance e Operações](#performance-e-operações)
        - [Índices](#índices)
        - [Connection pooling](#connection-pooling)
        - [Problema N+1](#problema-n1)
        - [Paginação: cursor vs offset](#paginação-cursor-vs-offset)
        - [Lock otimista e pessimista](#lock-otimista-e-pessimista)
      - [Replicação](#replicação)
      - [Sharding](#sharding)
      - [Federation](#federation)
    - [Não relacional (NoSQL)](#não-relacional-nosql)
      - [BASE](#base)
      - [Chave Valor (Key Value)](#chave-valor-key-value)
      - [Orientado a Documento](#orientado-a-documento)
      - [Colunar](#colunar)
      - [Orientado a grafos](#orientado-a-grafos)
      - [Vetoriais](#vetoriais)
    - [SQL ou NoSQL](#sql-ou-nosql)
  - [Cache](#cache)
  - [CDN](#cdn)
  - [Blob Store](#blob-store)
  - [Filas e Assincronicidade](#filas-e-assincronicidade)
  - [Eventos](#eventos)
    - [Event Driven](#event-driven)
    - [Event Sourcing](#event-sourcing)
    - [CQRS](#cqrs)
    - [SAGA](#saga)
      - [Workflow Engines](#workflow-engines)
  - [Load Balancer](#load-balancer)
  - [API Gateway](#api-gateway)
  - [Comunicação (Networking)](#comunicação-networking)
    - [HTTP e REST](#http-e-rest)
    - [RPC e gRPC](#rpc-e-grpc)
    - [GraphQL](#graphql)
  - [](#)
  - [Um pouco sobre segurança](#um-pouco-sobre-segurança)
    - [WAF](#waf)
    - [OWASP top 10](#owasp-top-10)
  - [Estimativas](#estimativas)
    - [Littles law](#littles-law)
    - [Back of envelop (conta de padeiro)](#back-of-envelop-conta-de-padeiro)
  - [Microsserviços e Monolitos](#microsserviços-e-monolitos)
  - [Referências:](#referências)
  - [Recomendação de leituras:](#recomendação-de-leituras)
  - [Recomendação de canais do YouTube sobre o tópico](#recomendação-de-canais-do-youtube-sobre-o-tópico)

## O que é system design?

Sempre que estamos diante de algum problema a ser resolvido em software, como
encurtadores de URL, upload de vídeos do YouTube, etc. precisamos pensar em como
vamos traduzir esses requisitos em um plano. É disso que se trata o system design:
montar um plano para resolver um problema.

Na construção desse plano, são definidos os fluxos e escolhas arquiteturais que darão forma à solução. Cada escolha carrega trade-offs. System design é muito sobre como suas escolhas impactam na performance, na escalabilidade e na confiabilidade do sistema, sabendo que sempre haverá sacrifícios.

Tenha em mente: tudo é um trade-off. Dito isso, vamos começar com os conceitos
iniciais.

## Latência e throughput (vazão)

- **Latência** é o tempo que o sistema leva para processar uma ação ou
  o "quão rápido" uma operação acontece.
  - Ex: Você fez a requisição para um servidor e ele respondeu em 100ms
- **Vazão** é a quantidade de operações que o sistema consegue processar
  por unidade de tempo ou "quanta coisa" o sistema aguenta.
  - Ex: dado um servidor, ele aguenta 10 mil requisições por segundo.

Geralmente você deve mirar em vazão máxima e uma latência aceitável.

> Em system design é muito comum escutarmos o termo throughput, por isso
> não escrevi vazão logo de cara, mas usarei o termo vazão ao longo da
> leitura, pois é a tradução técnica adequada.

## Performance e escalabilidade

- **Performance**: É quão bem o seu serviço usa seus recursos para responder às
  demandas, sendo medido pela latência e vazão.
  - Se seu sistema é lento para um único usuário, você tem um problema de performance.
- **Escalabilidade**: É como seu sistema se comporta com a demanda feita sobre ele.
  - Se seu sistema é rápido para um único usuário, mas é devagar para 10 mil, então você tem um problema de escalabilidade.

Por fim, temos o conceito de "um sistema ser escalável". Uma ótima definição é
do artigo [A Word on Scalability](https://www.allthingsdistributed.com/2006/03/a_word_on_scalability.html) do blog de Werner Vogels, [All Things Distributed](https://www.allthingsdistributed.com/):

> Um serviço é escalável quando o aumento de performance é proporcional aos recursos adicionados. Normalmente, aumentar performance significa servir mais unidades de trabalho, mas também significa lidar com grandes unidades de trabalho, como o crescimento de um dataset

### Escalabilidade horizontal e vertical

Um serviço pode escalar dessas duas formas:

- **Verticalmente**: você tem uma máquina com 4GB de RAM e decide trocar por uma com 16GB.
- **Horizontalmente**: você tem uma máquina com 4GB de RAM e decide ter mais 3 máquinas de mesma configuração

Veja que nos dois casos temos 16GB de RAM. Mas escalar verticalmente tem um limite, uma hora você atingirá o limite de RAM, enquanto escalar horizontalmente permite adicionar máquinas mais fracas até atender a demanda.

## Teorema CAP

Este teorema diz que, em sistemas distribuídos, serão garantidas apenas duas das letras da sigla, que significam:

- **Consistência**: Toda leitura recebe a escrita mais recente ou um erro
- **Disponibilidade (Availability)**: Toda requisição recebe uma resposta, mesmo que o dado não seja a versão mais recente
- **Tolerância a partições (Partition Tolerance)**: Ou tolerância a falhas de partição, diz respeito que o sistema continua operando apesar de haver falha de comunicação de rede entre nós. Ex: dois servidores que não conseguem comunicar entre si.

Como a comunicação de rede não é confiável, a tolerância a partições será uma escolha obrigatória. Portanto, teremos que ver qual se encaixa melhor no nosso sistema.

### CP (Consistência e tolerância a partição)

Neste caso estamos priorizando que o dado seja o mais recente. aso tenha problema de comunicação entre nós, é preferível retornar um erro do que um dado desatualizado.

**Quando utilizar?**

Sistemas bancários são um ótimo exemplo, nenhum cliente quer ver seu saldo bancário desatualizado.

### AP (Disponibilidade e tolerância a partição)

Aqui estamos optando por sempre mostrar o dado, mesmo se ele não for o mais atual. Escritas podem demorar pra propagar até que se resolva o problema com a partição.

**Quando utilizar?**

É bom quando o sistema precisa funcionar mesmo com erros externos acontecendo ou quando é permitido ter consistência eventual.

### Referência

- [CAP Theorem: Revisited](https://robertgreiner.com/cap-theorem-revisited)

## Banco de dados

Um banco de dados é um sistema que permite armazenar, organizar e recuperar
dados de forma eficiente e persistente. Mas por que não usar alternativas mais
simples? Algumas opções têm trade-offs que não estamos dispostos a aceitar:

- **Memória**: é cara e volátil. Quando a aplicação cair, tudo é perdido.
- **Arquivos CSV ou planilhas**: conforme os dados crescem, operações de leitura
  e escrita ficam inviáveis, além de não suportarem relações entre os dados
  nativamente.

Essas alternativas só fazem sentido em algo muito pequeno. Para soluções
robustas, precisamos de algo à altura.

### Relacional (SQL)

#### Conceitos Fundamentais

##### ACID

##### Normalização vs Desnormalização

#### Performance e Operações

##### Índices

##### Connection pooling

##### Problema N+1

##### Paginação: cursor vs offset

##### Lock otimista e pessimista

#### Replicação

#### Sharding

#### Federation

### Não relacional (NoSQL)

#### BASE

#### Chave Valor (Key Value)

#### Orientado a Documento

#### Colunar

#### Orientado a grafos

#### Vetoriais

### SQL ou NoSQL

## Cache

## CDN

## Blob Store

## Filas e Assincronicidade

## Eventos

### Event Driven

### Event Sourcing

### CQRS

### SAGA

#### Workflow Engines

## Load Balancer

## API Gateway

## Comunicação (Networking)

### HTTP e REST

### RPC e gRPC

### GraphQL

##

## Um pouco sobre segurança

### WAF

### OWASP top 10

## Estimativas

### Littles law

### Back of envelop (conta de padeiro)

## Microsserviços e Monolitos

## Referências:

- [The System Design Primer](https://github.com/donnemartin/system-design-primer#the-system-design-primer): Esse repositório foi a
  inspiração para escrever o System Design do Zero, muito do que é abordado
  aqui é inspirado nele. Tento não ser uma tradução direta, mas há fortes
  influências. Então, se possível, dê estrela para esse repositório.

## Recomendação de leituras:

- [Os 7 Padrões de System Design que Aparecem em Toda Entrevista](https://newsletter.nagringa.dev/p/padroes-system-design-entrevistas)

## Recomendação de canais do YouTube sobre o tópico

- [GutoGalego](https://www.youtube.com/@GutoGalego)
- [Renato Augusto](https://www.youtube.com/@RenatoAugustoTech)
