# Estudo de Caso A1 — O Problema da Entrega Inteligente (FastBite)

**Aluno:** João Gabriel Oliveira Tavares  
**RA:** 12314592  
**Disciplina:** Estruturas de Dados e Análise de Algoritmos (0006963)  
**Professor:** Alexandre “Montanha” de Oliveira  

Este documento apresenta as respostas ao estudo de caso “O Problema da Entrega Inteligente (FastBite)”, focando em roteamento e otimização logística em plataformas digitais de delivery. São discutidos aspectos de complexidade computacional, algoritmos gulosos, Programação Dinâmica, Divisão e Conquista, bem como o uso prático de heurísticas em sistemas de grande escala. Ao longo das questões, as justificativas buscam conectar os conceitos teóricos ao contexto concreto da FastBite, que recebe lotes de pedidos a cada 30 segundos e precisa decidir em até 2 segundos como atribuir pedidos e rotas para milhares de entregas diárias.  

---

## Questão 1 — Classificação do Problema

### **a) Classe de complexidade do problema da FastBite**

O problema de roteamento da FastBite é uma variação prática do Problema de Roteamento de Veículos (VRP), que por sua vez generaliza o Problema do Caixeiro Viajante (TSP). Em termos formais, o TSP na versão de decisão (“existe um circuito com custo total ≤ K?”) é um problema clássico NP-completo, e o VRP herda essa complexidade, sendo considerado NP-difícil (NP-hard) e, em versões de decisão adequadamente formuladas, também NP-completo.  

O problema da FastBite se encaixa na classe NP porque, se alguém nos der uma solução candidata (atribuição de pedidos para cada entregador e ordem completa de coleta/entrega para todos), conseguimos verificar em tempo polinomial se todas as restrições são satisfeitas (capacidade de até 3 pedidos por entregador, prazos de pedidos urgentes, tempos estimados de deslocamento) e calcular o tempo total de entrega. A verificação consiste em percorrer as rotas e somar custos, o que é polinomial em função do número de pedidos e entregadores.  

Além disso, como o problema contém o TSP/VRP como caso especial (por exemplo, se houvesse um único entregador ou se ignorássemos algumas restrições), ele é pelo menos tão difícil quanto esses problemas já conhecidos como NP-completos. Portanto, o problema de roteamento da FastBite, formulado na versão de decisão (existe uma alocação e um conjunto de rotas que atende todas as restrições com tempo total ≤ K?), é NP-completo: está em NP e é tão difícil quanto os problemas NP mais difíceis, por conter o TSP/VRP por redução.  

Em resumo: o problema da FastBite não está na classe P (não se conhece algoritmo determinístico em tempo polinomial que o resolva de forma ótima para o caso geral) e é classificado como NP-completo na sua formulação de decisão, alinhado aos resultados clássicos sobre TSP e VRP.  

### **b) Redução intuitiva do problema da FastBite para TSP**

Para mostrar intuitivamente que o problema da FastBite pode ser reduzido ao TSP, podemos considerar um cenário simplificado em que há apenas **um** entregador com capacidade suficiente para carregar todos os pedidos de um lote. Nesse cenário, o entregador precisa sair de um ponto inicial (por exemplo, um “depósito” ou sua posição atual), visitar todos os pontos relevantes (restaurantes e clientes) e, eventualmente, retornar ao ponto inicial ou encerrar em algum ponto final.  

A redução pode ser construída assim, de forma conceitual:  

- Cada **ponto do problema da FastBite** (restaurantes e clientes de um lote) vira uma **“cidade” no TSP**.  
- A posição inicial do entregador é adicionada como um nó especial (depósito), que também entra como cidade no TSP.  
- A **distância** entre dois pontos (ex.: restaurante de P1 e cliente de P3) é mapeada como o custo da aresta entre as duas cidades correspondentes no TSP (podendo usar distância Manhattan ou tempo estimado de deslocamento como peso da aresta).  
- A solução do TSP é um circuito que visita todas as cidades exatamente uma vez, minimizando o custo total. No nosso contexto, esse circuito equivale a uma rota que passa por todos os restaurantes e clientes, minimizando o tempo total de deslocamento (ou outra métrica relevante).  

Se ignorarmos, por um momento, detalhes como tempo de preparo dos restaurantes e a necessidade de pegar a comida antes de entregar, essa rota do TSP é uma candidata natural para a rota do entregador que precisa percorrer todos os pontos. Ajustando o modelo (por exemplo, exigindo que o nó “restaurante de P1” seja visitado antes do nó “cliente de P1”), continuamos na família de problemas TSP/VRP com restrições, que são conhecidos por serem NP-difíceis.  

Logo, uma instância do problema da FastBite (um conjunto de pedidos e um entregador com capacidade alta) pode ser transformada em uma instância de TSP ao mapear cada ponto relevante para uma cidade e as distâncias/tempos de deslocamento para pesos das arestas. A solução ótima do TSP nos fornece uma rota ótima para o entregador nesse cenário simplificado, mostrando intuitivamente a relação de redução.  

### **c) Inviabilidade da força bruta para 8 pedidos e 3 entregadores**

A abordagem de força bruta consistiria em considerar todas as possíveis atribuições de pedidos aos entregadores e, para cada atribuição, todas as possíveis ordens de visita (permutação) desses pedidos em cada rota. Isso cresce de forma combinatorial.  

Para ter uma ideia do crescimento, vamos raciocinar em duas etapas:  

1. **Atribuição de pedidos aos entregadores**  
   - Com 8 pedidos e 3 entregadores, ignorando por um momento o limite de 3 pedidos por entregador, cada pedido poderia ser atribuído a qualquer um dos 3 entregadores.  
   - Isso gera, no máximo, 3^8 atribuições diferentes (cada pedido escolhe um de 3 entregadores).  
   - 3^8 = 6.561 combinações possíveis de atribuição, já um número considerável, mesmo num cenário tão pequeno.  

2. **Permutações das rotas de cada entregador**  
   - Após definir quantos pedidos cada entregador recebeu (por exemplo, 3 para E1, 3 para E2 e 2 para E3), devemos avaliar todas as ordens possíveis em que cada entregador pode visitar seus pedidos.  
   - Se um entregador recebeu k pedidos, o número de possíveis rotas (ordens de visita) é k! (fatorial de k).  
   - Em um cenário típico com 3, 3 e 2 pedidos para E1, E2 e E3, respectivamente, teríamos 3! × 3! × 2! = 6 × 6 × 2 = 72 possíveis combinações de rotas só para essa distribuição.  
   - Como há muitas distribuições possíveis das 8 entregas entre os 3 entregadores, o número total de configurações (atribuição + ordem de visita) explode.  

Uma forma grosseira de ter uma ideia de ordem de grandeza é multiplicar o número máximo de atribuições (3^8) pelo número máximo de permutações globais (8!, já que, no limite, estamos permutando 8 entregas distribuídas de alguma forma):  

- 8! = 40.320  
- 3^8 × 8! ≈ 6.561 × 40.320 ≈ 264 milhões de combinações  

Ou seja, mesmo num caso minúsculo com 8 pedidos, a ordem de grandeza já atinge centenas de milhões de configurações a serem avaliadas se tentarmos força bruta, o que é claramente inviável para ser refeito a cada ciclo de 30 segundos com limite de 2 segundos de decisão.  

Assim, a complexidade assintótica da força bruta cresce de forma exponencial, envolvendo termos como 3^n e n!, o que torna essa abordagem impraticável para os cenários reais da FastBite onde temos dezenas ou centenas de pedidos simultâneos.  

---

## Questão 2 — Abordagem Gulosa (Greedy)

A equipe júnior propôs o algoritmo: “Para cada pedido não atribuído, escolha o entregador disponível mais próximo do restaurante e atribua o pedido a ele. Repita até que todos os pedidos estejam atribuídos. Para a rota de cada entregador, ordene as entregas sempre indo ao ponto mais próximo do atual.”  

### **a) Funcionamento passo a passo no cenário de exemplo**

Vamos usar o mini-cenário fornecido, com pedidos P1 a P5 e entregadores E1, E2 e E3, e distância Manhattan (|x1 − x2| + |y1 − y2|).  

- **Dados principais do cenário** (resumidos):  
  - Pedidos:  
    - P1: restaurante em (0, 2), cliente em (4, 5), urgente.  
    - P2: restaurante em (1, 1), cliente em (7, 2), padrão.  
    - P3: restaurante em (4, 4), cliente em (0, 0), premium.  
    - P4: restaurante em (6, 1), cliente em (3, 6), padrão.  
    - P5: restaurante em (5, 5), cliente em (8, 1), urgente.  
  - Entregadores:  
    - E1: posição inicial (1, 1), capacidade 2, bicicleta.  
    - E2: posição inicial (5, 3), capacidade 2, moto.  
    - E3: posição inicial (7, 6), capacidade 3, moto.  

Seguindo o algoritmo, assumindo que processamos os pedidos na ordem P1, P2, P3, P4, P5:  

1. **Atribuição de P1** (restaurante em (0, 2))  
   - Distância de E1: |1 − 0| + |1 − 2| = 2  
   - Distância de E2: |5 − 0| + |3 − 2| = 6  
   - Distância de E3: |7 − 0| + |6 − 2| = 11  
   - O algoritmo escolhe o entregador mais próximo: **P1 é atribuído a E1**.  

2. **Atribuição de P2** (restaurante em (1, 1))  
   - Neste modelo guloso simples, o algoritmo continua considerando a posição inicial de cada entregador (ou a posição “atual” assumida como a inicial no momento da atribuição).  
   - Distância de E1: |1 − 1| + |1 − 1| = 0  
   - Distância de E2: |5 − 1| + |3 − 1| = 6  
   - Distância de E3: |7 − 1| + |6 − 1| = 11  
   - E1 é disparado o mais próximo e ainda tem capacidade (2 pedidos). Logo, **P2 também é atribuído a E1**, que fica com sua capacidade cheia (2 pedidos).  

3. **Atribuição de P3** (restaurante em (4, 4))  
   - Distância aproximada usando posições iniciais:  
     - E1: |1 − 4| + |1 − 4| = 6  
     - E2: |5 − 4| + |3 − 4| = 2  
     - E3: |7 − 4| + |6 − 4| = 5  
   - O entregador mais próximo é **E2**, que recebe P3.  

4. **Atribuição de P4** (restaurante em (6, 1))  
   - Distâncias:  
     - E1 (capacidade já cheia) poderia até ser ignorado.  
     - E2: |5 − 6| + |3 − 1| = 3  
     - E3: |7 − 6| + |6 − 1| = 6  
   - O mais próximo é **E2**, que ainda tem capacidade e recebe P4. E2 também fica com capacidade cheia (2 pedidos).  

5. **Atribuição de P5** (restaurante em (5, 5))  
   - E1 e E2 estão com capacidade esgotada (2 pedidos cada).  
   - Só resta E3 com capacidade para 3 pedidos.  
   - Logo, **P5 é atribuído a E3** por exclusão.  

Ao final da etapa de atribuição, temos algo como:  

- E1: P1, P2  
- E2: P3, P4  
- E3: P5  

Na segunda fase, o algoritmo constrói a rota de cada entregador sempre indo ao ponto mais próximo do atual. Por exemplo:  

- Para **E1**, começando em (1, 1):  
  - Ele pode primeiro ir ao restaurante mais próximo entre os de P1 e P2 (provavelmente o de P2, em (1, 1), que é o próprio ponto de partida), depois ir ao restaurante de P1, depois aos clientes correspondentes, sempre escolhendo o próximo ponto (restaurante ou cliente) mais próximo, até entregar todos os pedidos.  
- Para **E2**, começando em (5, 3):  
  - Ele analisa os pontos de P3 e P4 (restaurantes e depois clientes) e sempre escolhe o ponto seguinte de menor distância Manhattan.  
- Para **E3**, começando em (7, 6):  
  - Ele parte para o restaurante de P5 em (5, 5), depois para o cliente de P5, sempre seguindo o ponto mais próximo.  

Assim, o algoritmo inteiro é dirigido por decisões locais de proximidade, tanto na atribuição quanto na ordem da rota, sem olhar o impacto global dessas escolhas.  

### **b) Por que o algoritmo é guloso? Propriedade de escolha local**

Um algoritmo é classificado como **guloso (greedy)** quando, a cada passo, ele toma a decisão local que parece a melhor naquele exato momento, de acordo com algum critério simples, na esperança de que a soma dessas decisões locais leve a uma solução global boa (ou ótima).  

No caso da FastBite, o algoritmo:  

- Na **atribuição**, escolhe sempre o **entregador mais próximo do restaurante** do pedido atual, ignorando os efeitos dessa decisão nos pedidos futuros (por exemplo, se isso vai sobrecarregar um entregador em uma área que poderia ser atendida melhor por outro).  
- Na **construção das rotas**, sempre escolhe o **próximo ponto mais próximo** em relação à posição atual do entregador, sem considerar que um pequeno desvio agora talvez permita um grande ganho mais à frente.  

A propriedade de escolha local aplicada aqui é, essencialmente, “sempre minimizar a distância imediata”: o algoritmo minimiza a distância entre entregador e restaurante (na atribuição) e entre ponto atual e próximo ponto (na rota). Isso é a essência da estratégia gulosa: pegar o melhor ganho imediato (menor distância agora), sem reavaliar o impacto global.  

### **c) Contraexemplo em que a solução gulosa é subótima**

Podemos construir um contraexemplo simples com base no cenário, focando em como a escolha local de “entregador mais próximo” pode levar a uma distribuição ruim de pedidos.  

Imagine a seguinte situação hipotética inspirada nos dados:  

- Dois entregadores:  
  - E1 em uma região central próxima de vários restaurantes.  
  - E2 mais afastado, mas ainda razoavelmente perto de um subconjunto de pedidos.  
- Três pedidos: P1, P2 e P3, todos relativamente próximos entre si em uma mesma região, mas com distâncias ligeiramente diferentes até E1 e E2.  

Suponha que:  

- Para P1 e P2, E1 é **levemente** mais próximo que E2 (diferença pequena de distância).  
- Para P3, E2 é **muito mais** próximo que E1.  

Pelo algoritmo guloso, se processarmos P1 e P2 primeiro:  

1. P1 é atribuído a E1 (E1 é o mais próximo, diferença pequena).  
2. P2 também é atribuído a E1 pelo mesmo critério.  
3. Quando chega a vez de P3, E1 já está com capacidade cheia, então P3 é forçadamente atribuído a E2, mesmo sendo o pedido mais “natural” para ele.  

O que pode acontecer:  

- E1 acaba com dois pedidos relativamente próximos, mas que poderiam ter sido divididos com E2 sem grande perda de distância.  
- E2 fica com apenas P3, mas precisa percorrer uma rota que talvez envolva um deslocamento inicial grande (de onde ele está até P3) e depois voltar ou se reposicionar para outros pedidos futuros.  

Uma solução alternativa, não gulosa, poderia ser:  

- Atribuir P1 a E1, P2 a E2 e P3 também a E2.  
- Isso balancearia melhor as rotas, reduziria o deslocamento total e o tempo de entrega médio, já que E2 ficaria responsável pela região onde P2 e P3 estão concentrados, enquanto E1 ficaria com uma carga menor, mas estrategicamente posicionada.  

Esse tipo de cenário mostra que, ao priorizar sempre a menor distância imediata para o pedido atual, o algoritmo pode “queimar” a capacidade de um entregador em pedidos que não são tão adequados para ele, deixando outros pedidos que seriam muito melhores para aquele entregador com alguém menos eficiente. O resultado global (tempo total ou atrasos em pedidos urgentes) pode ser claramente pior do que em uma atribuição mais planejada.  

### **d) Complexidade de tempo do algoritmo guloso**

Vamos considerar:  

- n = número de pedidos no lote.  
- m = número de entregadores disponíveis no ciclo.  

O algoritmo tem duas fases principais:  

1. **Atribuição de pedidos**  
   - Para cada pedido não atribuído, o algoritmo percorre a lista de entregadores disponíveis para encontrar o mais próximo do restaurante.  
   - Isso gera um custo de, aproximadamente, n × m comparações de distância.  
   - Logo, a complexidade de tempo dessa etapa é O(n · m).  

2. **Construção das rotas para cada entregador (heurística do “ponto mais próximo”)**  
   - Suponha que o entregador i tenha kᵢ pedidos atribuídos (com ∑kᵢ = n).  
   - Para montar a rota gulosa, ele começa em sua posição inicial e, em cada passo, escolhe o próximo ponto (restaurante ou cliente) mais próximo dentre os ainda não visitados.  
   - Em termos de complexidade, isso é semelhante à heurística do “vizinho mais próximo” do TSP: o primeiro passo considera kᵢ candidatos, o segundo passo considera (kᵢ − 1), depois (kᵢ − 2), e assim por diante, resultando numa soma de ordem kᵢ².  
   - Para todos os entregadores, o custo total é proporcional à soma dos quadrados: ∑kᵢ², que no pior caso é O(n²) (quando um entregador recebe a maior parte dos pedidos).  

Portanto, a complexidade total aproximada do algoritmo guloso é:  

- O(n · m) para a atribuição + O(n²) para a construção das rotas.  

Podemos escrever isso como O(n · m + n²). Em muitos cenários, com m razoavelmente pequeno em relação a n, o termo dominante tende a ser O(n²). Ainda assim, trata-se de uma complexidade polinomial relativamente gerenciável, compatível com a exigência de decisões em até 2 segundos em máquinas modernas, desde que n por ciclo não seja gigantesco.  

---

## Questão 3 — Programação Dinâmica e Divisão e Conquista

### **a) Aplicabilidade de Programação Dinâmica (PD) para um único entregador com k pedidos**

Se considerarmos um único entregador que já recebeu um conjunto de k pedidos (atribuídos previamente), o subproblema de “qual é a melhor ordem de visita desses k pedidos, respeitando as distâncias e eventualmente janelas de tempo simples” é muito semelhante ao TSP.  

A **Programação Dinâmica** é aplicável nesse contexto quando o problema apresenta:  

- **Subestrutura ótima**: a melhor rota para k pedidos pode ser construída a partir das melhores rotas para subconjuntos desses pedidos.  
- **Sobreposição de subproblemas**: as mesmas combinações de subconjuntos e último ponto visitado são recalculadas muitas vezes, o que justifica o uso de memoização.  

Um subproblema típico para PD nesse contexto poderia ser definido assim, de forma informal:  

- **Subproblema**: “Qual é o custo mínimo para o entregador sair de sua posição inicial, visitar um subconjunto S de pedidos e terminar no ponto correspondente ao pedido j (por exemplo, o cliente de Pj)?”  
- O estado da PD pode ser algo como (S, j), onde S é um subconjunto de pedidos já visitados e j é o último pedido atendido.  
- A transição considera mover-se de j para algum novo pedido k ainda não visitado, adicionando o custo da aresta correspondente.  

O número de estados dessa PD é da ordem de 2^k (todas as combinações de subconjuntos) vezes k (possível último pedido), então temos cerca de k · 2^k estados. Cada transição pode considerar, no pior caso, k possíveis próximos pedidos, resultando em uma complexidade de tempo em torno de k² · 2^k e espaço de k · 2^k.  

Na prática, isso é **exponencial em k**, embora mais eficiente que a força bruta pura (que seria k! para permutar todas as ordens). Ainda assim, é impraticável para k moderado.  

Considerando que a FastBite precisa tomar decisões em tempo real, com limite de cerca de 2 segundos por ciclo para processar múltiplos entregadores e pedidos, essa abordagem de PD só é viável para valores muito pequenos de k. Um limite razoável, na prática, costuma ser k na faixa de 10 a 15 pedidos por entregador; acima disso, o termo 2^k explode e fica difícil garantir que toda a PD rode dentro da janela de tempo, especialmente em um sistema com muitos entregadores concorrentes.  

Portanto:  

- **Sim, PD é aplicável conceitualmente ao roteamento de um único entregador com k pedidos**, modelando o problema como um TSP com memoização.  
- **Mas o custo de tempo (≈ k² · 2^k) e espaço (≈ k · 2^k) cresce exponencialmente**, tornando a solução impraticável para k maior do que algo em torno de 10–15 em um sistema de alta demanda e baixa latência como o da FastBite.  

### **b) Aplicabilidade de Divisão e Conquista ao problema da FastBite**

A ideia de **Divisão e Conquista** é dividir o problema original em subproblemas menores, resolver cada um deles (idealmente de forma independente) e depois combinar as soluções.  

No contexto da FastBite, existem algumas situações em que isso é plausível:  

1. **Divisão por região geográfica**  
   - Podemos particionar a cidade em zonas, quadrantes ou células (por exemplo, usando uma grade ou clusters de pontos de calor de pedidos).  
   - Em seguida, tratamos cada zona quase como um subproblema independente: pedidos e entregadores que estão naquela região são inicialmente considerados apenas entre si.  
   - Isso reduz o tamanho de cada instância local, tornando viável usar heurísticas mais sofisticadas dentro da zona (como um greedy mais elaborado ou até PD para poucos pedidos).  

2. **Condições para independência de subproblemas**  
   - A divisão geográfica funciona melhor quando os pedidos e entregadores de uma região são, em grande parte, “auto-suficientes”, ou seja, é raro ser vantajoso que um entregador saia de sua zona para atender um pedido de outra.  
   - Isso pode acontecer, por exemplo, em grandes cidades com tráfego pesado, onde cruzar regiões tem um custo alto e, na prática, já não faria sentido um entregador de uma extremidade ir para outra região distante durante um pico.  

3. **Limitações e fronteiras entre zonas**  
   - O grande problema da abordagem de Divisão e Conquista por região é o que acontece nas **fronteiras**:  
     - Pode haver pedidos bem na borda entre duas zonas, para os quais seria melhor usar um entregador da zona vizinha.  
     - A divisão rígida pode levar a soluções onde um entregador percorre um caminho maior dentro de sua zona enquanto, do outro lado da fronteira, um entregador está relativamente ocioso e muito perto daquele pedido.  
   - Para aliviar esse problema, é comum usar “zonas de transição” ou permitir, em uma etapa posterior de refinamento, que alguns pedidos sejam trocados entre zonas (por exemplo, heurísticas de troca de pedidos entre entregadores de zonas vizinhas).  

Em resumo:  

- **É possível aplicar Divisão e Conquista** ao problema da FastBite, especialmente por meio de **particionamento geográfico** da cidade.  
- Os subproblemas são “quase independentes” quando as regiões são bem definidas e o custo de cruzar regiões é alto, mas nunca totalmente independentes: sempre haverá casos de pedidos na fronteira que podem ser beneficiados por um reequilíbrio entre zonas.  
- A maior limitação é que a solução combinada não é globalmente ótima, e as decisões de fronteira podem degradar a qualidade das rotas; ainda assim, a redução de complexidade e o ganho de escalabilidade justificam essa abordagem na prática.  

---

## Questão 4 — Comparação das Abordagens

### Tabela comparativa

| Critério                               | Greedy                                               | Programação Dinâmica                               | Divisão e Conquista                                                     |
|----------------------------------------|------------------------------------------------------|----------------------------------------------------|-------------------------------------------------------------------------|
| Qualidade da solução                  | Boa em média, mas pode ser bem subótima em casos ruins | Ótima para o subproblema modelado (k pequeno)      | Boa se as zonas forem bem definidas; pode ser subótima nas fronteiras  |
| Complexidade de tempo                 | Aproximadamente O(n · m + n²)                        | Aproximadamente O(k² · 2^k) por entregador         | Depende do algoritmo interno em cada zona; tipicamente menor que o global monolítico |
| Complexidade de espaço                | Baixa, próxima de O(n + m)                          | Alta, em torno de O(k · 2^k)                       | Moderada; soma da memória usada nos subproblemas + estruturas de zona  |
| Viabilidade em tempo real (≤ 2s)      | Alta, desde que n por ciclo não seja enorme         | Viável só para k muito pequeno (≈ 10–15)           | Alta, se cada subproblema for mantido pequeno e resolvido com heurísticas rápidas |
| Escalabilidade com aumento de n       | Razoável; cresce polinomialmente                    | Muito ruim; explode exponencialmente com k         | Boa; mais zonas podem ser adicionadas distribuindo a carga             |
| Facilidade de adaptação a mudanças    | Alta; fácil recalcular incrementalmente             | Baixa; difícil atualizar sem recomputar a PD       | Moderada; mudanças locais impactam apenas algumas zonas                |

### Análise crítica e abordagem mais adequada para a FastBite

No contexto da FastBite, que precisa decidir a cada 30 segundos como atribuir pedidos a entregadores e montar rotas, com limite de cerca de 2 segundos e milhares de pedidos por dia, a prioridade é **tempo de resposta e escalabilidade**, mesmo que isso implique abrir mão da solução ótima. A Programação Dinâmica oferece soluções ótimas para subproblemas pequenos, mas sua complexidade exponencial em k torna inviável usá-la como solução principal de roteamento para o sistema inteiro.  

A abordagem puramente gulosa é extremamente atrativa do ponto de vista de implementação, tempo de execução e facilidade de adaptação a mudanças (novos pedidos, entregadores que entram/saem), mas pode produzir soluções significativamente subótimas em cenários com fronteiras, prioridades complexas e restrições de capacidade. Já Divisão e Conquista, especialmente via particionamento geográfico, oferece um bom compromisso: reduz o tamanho dos subproblemas, permite usar heurísticas rápidas como o próprio greedy dentro de cada zona e escala melhor conforme o número de pedidos e entregadores cresce.  

Portanto, a solução mais adequada como “núcleo” da FastBite tende a ser uma estratégia baseada em **Divisão e Conquista com heurísticas gulosas dentro de cada zona**, possivelmente complementada por pequenos refinamentos locais. Essa abordagem equilibra bem qualidade da solução, viabilidade em tempo real e escalabilidade, atendendo às exigências operacionais do sistema.  

---

## Questão 5 — Solução de Engenharia Real

### **a) O que é uma heurística e por que usá-la na FastBite**

Uma **heurística**, no contexto de algoritmos, é uma regra prática ou estratégia aproximada que busca encontrar uma solução “boa o suficiente” para um problema complexo, sem garantia de otimalidade, mas com grande ganho em tempo de execução e simplicidade. Em vez de explorar todo o espaço de soluções (como força bruta) ou resolver o problema de forma exata (como PD para TSP), uma heurística explora apenas uma fração desse espaço, guiada por intuições como “escolher o mais próximo” ou “priorizar pedidos urgentes”.  

Em sistemas como a FastBite, heurísticas são preferíveis a soluções ótimas porque:  

- O problema é NP-completo, então buscar a solução ótima global é computacionalmente muito caro para instâncias grandes.  
- O sistema recebe lotes de pedidos a cada 30 segundos e tem apenas até 2 segundos para decidir atribuições e rotas, o que é incompatível com algoritmos exatos de alta complexidade.  
- O ambiente é dinâmico e incerto (variação de trânsito, preparo de restaurantes, cancelamentos, novos pedidos), de modo que uma solução “perfeita” calculada agora pode ficar obsoleta em poucos minutos.  
- Na experiência prática de empresas de delivery, importa mais ter uma solução consistente, rápida e previsível, que mantenha tempos de entrega aceitáveis, do que gastar muito processamento para ganhar poucos segundos em alguns pedidos isolados.  

Assim, heurísticas são uma ferramenta fundamental em engenharia de software para problemas de otimização em larga escala, justamente por equilibrar desempenho computacional e qualidade da solução.  

### **b) Proposta de solução de engenharia real para a FastBite**

Uma solução de engenharia plausível para a FastBite poderia ser estruturada em camadas, combinando particionamento geográfico, heurísticas gulosas e refinamento local, sempre com um limite de tempo estrito. Um esboço de arquitetura seria:  

1. **Particionamento por região geográfica**  
   - Dividir a cidade em zonas (por exemplo, usando uma grade de coordenadas ou clusters de densidade de pedidos).  
   - Cada zona possui um subconjunto de pedidos e entregadores candidatos. Em geral, um entregador pertence à zona em que está localizado, com alguma flexibilidade para zonas vizinhas.  
   - Essa etapa reduz o problema global em vários subproblemas menores, que podem até ser processados em paralelo.  

2. **Heurística gulosa inicial dentro de cada zona**  
   - Para cada zona, rodar um algoritmo guloso de atribuição similar ao descrito na questão 2, mas incorporando algumas regras adicionais, como:  
     - Priorizar pedidos urgentes e premium na fila de processamento.  
     - Penalizar entregadores que já estão muito carregados ou muito distantes.  
     - Considerar estimativas de preparo dos restaurantes, tentando casar o tempo de deslocamento com o tempo de preparo para evitar esperas longas.  
   - Em seguida, montar rotas gulosas (vizinho mais próximo) para cada entregador, respeitando capacidade máxima de 3 pedidos simultâneos.  

3. **Refinamento local (melhorias incrementais)**  
   - Após ter uma solução inicial rápida, aplicar um conjunto de operações locais de melhoria, dentro de um orçamento de tempo muito restrito, por exemplo:  
     - **Troca de pedidos** entre dois entregadores da mesma zona ou de zonas vizinhas (swap de 1 ou 2 pedidos) e aceitação da troca se o custo total (tempo/distorção) diminuir.  
     - Pequenos ajustes de rota tipo 2-opt (remover duas arestas e reconectar de outra forma) quando isso reduz a distância percorrida por um entregador.  
   - Essas operações são baratas e podem ser repetidas algumas vezes até que não haja melhoria relevante ou o tempo se esgote.  

4. **Limite de tempo estrito e fallback**  
   - Todo o processo (particionamento + heurística inicial + refinamento) roda com um **limite de tempo estrito**, por exemplo, 1,5 segundos dentro da janela de 2 segundos, deixando margem para overheads.  
   - Se o tempo limite estiver para estourar, o sistema simplesmente **interrompe o refinamento** e assume a melhor solução encontrada até aquele momento (solução “bom o suficiente”).  
   - Em casos extremos (pico muito alto), pode até pular o refinamento e usar apenas a solução gulosa inicial, garantindo resposta em tempo hábil.  

Essa estrutura reflete bem a filosofia de engenharia em sistemas de grande escala: ter uma solução básica extremamente rápida (greedy) e, quando possível, refiná-la um pouco, sem nunca violar os limites de tempo impostos pelo negócio.  

### **c) Quando vale a pena buscar a solução ótima? Exemplo no contexto de delivery**

Buscar a solução ótima faz sentido quando:  

- O tamanho da instância é pequeno (poucos pedidos e poucos entregadores).  
- Não há uma exigência tão agressiva de tempo de resposta em tempo real.  
- A qualidade da solução tem um impacto muito grande (financeiro, logístico ou de experiência) que justifique o custo computacional extra.  

Dentro do contexto de delivery, um exemplo razoável seria:  

- Planejamento de **rotas de abastecimento de “dark kitchens” ou de centros de distribuição** durante a madrugada, em que há poucos pontos a serem visitados (por exemplo, 10 restaurantes) e a decisão pode ser tomada com antecedência, sem pressão de latência em segundos.  
- Nessa situação, a empresa pode rodar um algoritmo exato (como PD/TSP) em um servidor offline para encontrar a rota realmente ótima, maximizando eficiência de combustível e tempo de deslocamento, porque o ganho acumulado em longo prazo pode ser significativo.  

Outro exemplo é o uso de algoritmos exatos para fins de **simulação e benchmarking**: a empresa pode calcular soluções ótimas para pequenos conjuntos de dados e usá-las como referência para avaliar quão boas são as heurísticas do sistema em ambiente de produção.  

---

## Questão 6 — Reflexão Crítica

Em sistemas de larga escala como a FastBite, o dilema entre “solução ideal” e “solução possível” é onipresente. Do ponto de vista teórico, gostaríamos de resolver o problema de roteamento de forma ótima, mas a complexidade NP-completa do TSP/VRP torna isso inviável para instâncias grandes, especialmente com milhares de pedidos diários e decisões a cada 30 segundos. O custo computacional de uma solução perfeita cresce de forma explosiva e rapidamente ultrapassa o orçamento de tempo e infraestrutura da empresa.  

Na prática, “bom o suficiente” se torna a melhor decisão técnica quando o ganho marginal de qualidade da solução não compensa o custo adicional de processamento, a complexidade do sistema e o risco de não cumprir as janelas de tempo. Uma heurística bem desenhada, que roda em milissegundos, gera rotas suficientemente boas para manter clientes satisfeitos e entregadores produtivos, enquanto permite que o sistema reaja rapidamente a eventos imprevisíveis como congestionamentos, atrasos de preparo ou novos pedidos urgentes.  

Além disso, perseguir a solução ideal pode reduzir a agilidade de desenvolvimento: algoritmos muito complexos são difíceis de manter, escalar e adaptar às mudanças constantes do negócio (novas regras de prioridade, promoções, zonas de risco, etc.). Em contrapartida, soluções aproximadas e moduláveis, como particionamento geográfico com heurísticas gulosas e refinamento local, permitem evoluir o sistema de forma incremental, testando e ajustando estratégias sem comprometer a estabilidade.  

Assim, em sistemas de grande escala, “bom o suficiente” não é sinônimo de preguiça técnica, mas de engenharia responsável: reconhecer que, dadas as limitações de complexidade computacional e de tempo de resposta, o objetivo é maximizar o valor entregue ao usuário final com os recursos disponíveis, em vez de perseguir uma perfeição matemática inalcançável em produção.  
