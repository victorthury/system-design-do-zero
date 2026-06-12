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
   3.1 [Escalabilidade horizontal e vertical](#escalabilidade-horizontal-e-vertical)
4. [Teorema CAP](#teorema-cap)

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

Este teorema diz que, em sistemas distribuídos, será garantido apenas duas das letras da sigla, que significam:

- **Consistência**: Toda leitura recebe a escrita mais recente ou um erro
- **Disponibilidade (Availability)**: Toda requisição recebe uma resposta, mesmo que o dado não seja a versão mais recente
- **Tolerância a partições (Partition Tolerance)**: Ou tolerância a falhas de partição, diz respeito que o sistema continua operando apesar de haver falha de comunicação de red entres nós. Ex: dois servidores que não conseguem comunicar entre si.

Como a comunicação de rede não é confiável, a tolerância a partições será uma escolha obrigatória. Portanto, teremos que ver qual se encaixa melhor no nosso sistema.

### CP (Consistência e tolerância a partição)

Neste caso estamos priorizando que o dado seja fresco, caso tenha problema de comunicação entre nós, é preferível retornar um erro do que um dado desatualizado.

**Quando utilizar?**

Sistemas bancário são um ótimo exemplo, nenhum cliente quer ver seu saldo bancário desatualizado.

### AP (Disponibilidade e tolerância a partição)

Aqui estamos optando por sempre mostrar o dado, mesmo se ele não for o mais atual. Escritas podem demorar pra propagar até que se resolva o problema com a partição.

**Quando utilizar?**

É bom quando o sistema precisa funcionar mesmo com erros externos acontecendo ou quando é permitido ter consistência eventual.

### Referência

- [CAP Theorem: Revisited](https://robertgreiner.com/cap-theorem-revisited)

## Referências:

- [The System Design Primer](https://github.com/donnemartin/system-design-primer#the-system-design-primer) - Esse repositório foi a
  inspiração para escrever o System Design do Zero, muito do que é abordado
  aqui é inspirado nele. Tento não ser uma tradução direta, mas há fortes
  influências. Então, se possível, dê estrela para esse repositório.

## Recomendação de leituras:

- [Os 7 Padrões de System Design que Aparecem em Toda Entrevista](https://newsletter.nagringa.dev/p/padroes-system-design-entrevistas)

## Recomendação de canais do youtube sobre o tópico

- [GutoGalego](https://www.youtube.com/@GutoGalego)
- [Renato Augusto](https://www.youtube.com/@RenatoAugustoTech)
