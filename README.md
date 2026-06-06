# Base de Dados Usados

Informações a serem coletadas:
* Jogos das seleções.
  * Seleção A x Seleção B.
  * Resultado.
  * Campeonato.
* Classificação das seleções via EA Sports 26-18:
  * Média dos atributos DEF.  
  * Média dos atributos MED.
  * Média dos atributos ATQ.
  * Média dos atributos OVR (média da DEF,MED e ATQ).
* Ranking histórico da Fifa 26 - 18:
  * Posição do Ranking.
  * Pontuação do Ranking.


* Integração feita pelas datas e nomes das seleções.
  * Os nomes das seleções exigiram normalização
  
# Como foi coletado cada informação

## Jogos das seleções

Dataset disponibilizado no kaggle, "International football results from 1872 to 2026":

* Link: [Dataset selection games 1872 - 2026](https://www.kaggle.com/datasets/martj42/international-football-results-from-1872-to-2017)
* Perfil: `Mart Jürisoo`.

## Classificação das seleções via EA Sports 26

Web Scrapling do site `sofifa.com`, o qual resume os rankings de todas as seleções.

* Link: [sofifa site](https://sofifa.com/teams?type=national&showCol%5B0%5D=oa&showCol%5B1%5D=at&showCol%5B2%5D=md&showCol%5B3%5D=df&r=260036&set=true) 

## Ranking historico do jogo

Requisição via API do site oficial da `Fifa`.

* Link: [Ranking Oficial da FIFA](https://inside.fifa.com/pt/fifa-rankings?intcmp=%28p_fifacom%29_%28d_insidefifa%29_%28c_webheader-main%29...)

# Tratamento dos Dados

Questões a serem consideradas:
* Dados temporais
  * Como integrar as diferentes bases
  * Peso da temporalidade (dados mais recentes tem que ter mais peso que os dados antigos)
* Peso dos campeonatos:
  * Competições de maior nível tem que possuir maior peso.
* Dados Vazios/Ausentes:
  * A grande maioria das seleções dos jogos da EA Sports não estão presentes por questões de direitos autorais.
  * Estratégia de imputação: para seleções sem dados EA Sports, o OVR será estimado via regressão linear usando o ranking FIFA e pontuação FIFA como preditores. Os atributos DEF, MED e ATQ serão imputados com o valor do OVR estimado.
    * Limitação: ao imputar DEF = MED = ATQ = OVR, perde-se a informação de balanço do time (times com mesmo OVR mas perfis distintos ficam idênticos). Essa limitação é aceitável para um primeiro modelo, dado que os times sem dados EA Sports são majoritariamente seleções menores.
    * Após treinar o modelo, verificar a importância dos atributos separados (DEF, MED, ATQ) — se não agregarem valor individualmente, simplificar usando apenas o OVR.
* transformação de dados:
  * Criar relações de temporalidade dos dados, ex:
    1. Forma da Equipe (resultados dos últimos 5 jogos, excluir amistosos)
    2. Aproveitamento dos mandos (em casa, fora de casa e neutro)
    3. Média de saldo de gols (Saldo de gols dos ultimos 5 jogos)
    4. H2H (head to Head, historico do confronto direto das 2 seleções)
    5. Intervalo entre jogos (em dias).

# Machine Learning

Para jogos da **fase de grupos**:
- **Time vencedor** — qual time vence a partida (use `casa`, `visitante` ou `empate`).

Para jogos da **fase eliminatória**:
- **Confronto** — quais dois times você prevê que jogarão nessa posição. Como a chave é determinada pelos resultados da fase de grupos, você precisa prever quais times avançarão o suficiente para se enfrentarem em cada fase.
- **Vencedor da partida** — qual equipe vence a partida (use `casa` ou `visitante`)
- **Pênaltis** — se a partida vai para a disputa de pênaltis (`Verdadeiro` ou `Falso`)
- **Formato do campeonato** - é preciso aplicar o formato do campeonato nos valores previstos.

Como é necessario prever 4 `outputs` será feito 4 modelos, um para prever os resultados, outro para prever cartões, mais um para prever os escanteios e, por fim, um para prever pênaltis.

* Modelos escolhidos para cada:
  * Resultados e penáltis: ``Ligthgbm``, com uso da simulação `monte carlo`

## Simulação do Torneio (Monte Carlo)

Para preencher os slots do mata-mata de forma fundamentada, o torneio será simulado via Monte Carlo:

1. O modelo gera a distribuição de probabilidade de cada placar possível para cada jogo da fase de grupos.
2. O torneio é simulado N vezes (ex: 1.000 iterações), sorteando um placar a cada rodada conforme essas probabilidades.
3. Para cada slot do mata-mata, a seleção que avançou com maior frequência nas simulações é usado como previsão.

## Validação

* **Etapa de desenvolvimento:** treinar com dados até 2021 e testar na Copa do Mundo de 2022, permitindo avaliar a abordagem e ajustar features e hiperparâmetros com base em resultados reais.
* **Modelo final:** retreinar com todos os dados disponíveis (2018 até os jogos de 2026 pré-Copa) e submeter as previsões da Copa do Mundo de 2026.
* O modelo será avaliado com base no sistema de pontuação da competição, que é o objetivo final.