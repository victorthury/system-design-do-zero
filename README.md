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

1. [O que é system design](#o-que-é-system-design)
2. [Latência e throughput](#latência-e-throughput-vazão)
3. [Performance e escalabilidade](#performance-e-escalabilidade)
   - 3.1 [Escalabilidade horizontal e vertical](#escalabilidade-horizontal-e-vertical)
4. [Teorema CAP](#teorema-cap)
   - 4.1 [CP (Consistência e tolerância a partição)](#cp-consistência-e-tolerância-a-partição)
   - 4.2 [AP (Disponibilidade e tolerância a partição)](#ap-disponibilidade-e-tolerância-a-partição)
5. [Banco de dados](#banco-de-dados)
   - 5.1 [Relacional (SQL)](#relacional-sql)
     - 5.1.1 [ACID](#acid)
     - 5.1.2 [Normalização vs Desnormalização](#normalização-vs-desnormalização)
     - 5.1.3 [Índices](#índices)
     - 5.1.4 [Connection pooling](#connection-pooling)
     - 5.1.5 [Problema N+1](#problema-n1)
     - 5.1.6 [Paginação: cursor vs offset](#paginação-cursor-vs-offset)
     - 5.1.7 [Lock otimista e pessimista](#lock-otimista-e-pessimista)
     - 5.1.8 [Replicação](#replicação)
     - 5.1.9 [Sharding](#sharding)
     - 5.1.10 [Federation](#federation)
   - 5.2 [Não relacional (NoSQL)](#não-relacional-nosql)
     - 5.2.1 [BASE](#base)
     - 5.2.2 [Chave Valor (Key Value)](#chave-valor-key-value)
     - 5.2.3 [Orientado a Documento](#orientado-a-documento)
     - 5.2.4 [Colunar](#colunar)
     - 5.2.5 [Orientado a Grafos](#orientado-a-grafos)
     - 5.2.6 [Vetoriais](#vetoriais)
   - 5.3 [SQL ou NoSQL](#sql-ou-nosql)
6. [Cache](#cache)
7. [CDN](#cdn)
8. [Blob Store](#blob-store)
9. [Filas e Assincronicidade](#filas-e-assincronicidade)
10. [Eventos](#eventos)
    - 10.1 [Event Driven](#event-driven)
    - 10.2 [Event Sourcing](#event-sourcing)
    - 10.3 [CQRS](#cqrs)
    - 10.4 [SAGA](#saga)
      - 10.4.1 [Workflow Engines](#workflow-engines)
11. [Load Balancer](#load-balancer)
12. [API Gateway](#api-gateway)
13. [Comunicação (Networking)](#comunicação-networking)
    - 13.1 [HTTP e REST](#http-e-rest)
    - 13.2 [RPC e gRPC](#rpc-e-grpc)
    - 13.3 [GraphQL](#graphql)
14. [Um pouco sobre segurança](#um-pouco-sobre-segurança)
    - 14.1 [WAF](#waf)
    - 14.2 [OWASP Top 10](#owasp-top-10)
15. [Estimativas](#estimativas)
    - 15.1 [Little's Law](#littles-law)
    - 15.2 [Back of the Envelope](#back-of-envelop-conta-de-padeiro)
16. [Microsserviços e Monolitos](#microsserviços-e-monolitos)

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

Bancos relacionais vão garantir as propriedades ACID, que são:

- **Atomicidade**: Cada transação tudo ou nada. Se uma transação tem duas etapas, não pode ser executado apenas uma. Ou faz tudo ou nada. Segue exemplo:
  ``` sql
  BEGIN;

  UPDATE contas SET saldo = saldo - 500 WHERE id = 1;
  UPDATE contas SET saldo = saldo + 500 WHERE id = 2;

  COMMIT;
  ```

- **Consistência**: toda transação leva o banco de dados para um estado válido. Por exemplo, se temos uma tabela que tem uma coluna que exige chave estrangeira obrigatoriamente, não é possível adicionar um registro sem a chave estrangeira. Ou seja, regras estabelecidas são respeitadas.

  ``` sql
  CREATE TABLE pedidos (
    id         SERIAL PRIMARY KEY,
    cliente_id INT NOT NULL REFERENCES clientes(id)
  );

  -- Falha se cliente_id 999 não existir em clientes
  INSERT INTO pedidos (cliente_id) VALUES (999);
  -- ERROR: insert or update on table "pedidos" violates foreign key constraint
  ```

- **Isolamento**: uma transação não interfere na outra. Elas são executadas concorrentemente e terão o mesmo resultado do que executadas serialmente

- **Durabilidade**: Uma vez que uma transação é commitada, a mudança é permanente. Mesmo se cair a energia ou houver falha, mudança será acessível após correção.


Como os bancos relacionais seguem estes princípios, nos é proporcionado garantias de integridade e confiabilidade sobre os dados, sendo excelentes para sistemas que precisam de consistência. Esses princípios fazem o modelo relacional ideal para aplicações financeiras e e-commerce.

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
