# FIFA World Cup 2026 Prediction

Projeto de Machine Learning para previsão da Copa do Mundo FIFA 2026.

O objetivo é prever:

- Resultados da fase de grupos;
- Classificação das seleções;
- Confrontos do mata-mata;
- Campeão do torneio.

A abordagem utiliza:

- Histórico de partidas internacionais;
- Ranking FIFA;
- Ratings EA Sports FC;
- Engenharia de atributos temporais;
- LightGBM;
- Simulação Monte Carlo.

---

# Estrutura do Projeto

```text
.
├── Joins datasets/
│   └── Bases integradas após tratamento
│
├── Raking Fifa 2018 - 2026/
│   └── Ranking FIFA histórico
│.
├── Jogos Copa 2018 - 2022/
│   └── Base utilizada para validação do modelo na Copa do Mundo de 2022
│
├── Jogos Seleções 2018 - 2026/
│   ├── Jogos Seleções.ipynb
│   │   └── Coleta, limpeza e preparação do histórico de partidas
│   │
│   ├── results.csv
│   │   └── Resultados das partidas internacionais
│   │
│   ├── shootouts.csv
│   │   └── Histórico de disputas por pênaltis
│   │
│   └── jogos_tratado.csv
│       └── Base consolidada após tratamento
│
├── Ratings EA Sports 2018 - 2026/
│   ├── 2018/
│   ├── 2019/
│   ├── 2020/
│   ├── 2021/
│   ├── 2022/
│   ├── 2023/
│   ├── 2024/
│   ├── 2025/
│   ├── 2026/
│   │   └── Arquivos HTML coletados do SoFIFA em diferentes datas,
│   │       preservando o rating histórico das seleções ao longo do tempo
│   │
│   ├── Ratings.ipynb
│   │   └── Extração e tratamento dos ratings das seleções
│   │
│   └── selecoes_sofifa.csv
│       └── Base consolidada dos atributos EA Sports FC
│
├── Raking Fifa 2018 - 2026/
│   ├── RankingFifa.ipynb
│   │   └── Coleta e tratamento do ranking FIFA histórico
│   │
│   └── historico_fifa_ranking_2018_2026.csv
│       └── Ranking FIFA mensal das seleções
│
├── Joins datasets/
│   ├── LigBases.ipynb
│   │   └── Integração entre jogos, rankings FIFA e ratings EA Sports
│   │
│   └── dataset_temporal_final.csv
│       └── Dataset final utilizado na modelagem
│
├── group_fixtures.csv
│   └── Jogos da fase de grupos fornecidos pela competição
│
├── knockout_slots.csv
│   └── Estrutura oficial do mata-mata
│
├── Modelo.ipynb
│   └── Treinamento, validação, simulação Monte Carlo e geração das previsões
│
├── Kaggle_Submission_Complete.csv
│   └── Arquivo final de submissão
│
├── .gitignore
│
├── Ratings EA Sports 2018 - 2026/
│   └── Ratings das seleções extraídos do SoFIFA
│
├── group_fixtures.csv
│   └── Jogos da fase de grupos da competição
│
├── knockout_slots.csv
│   └── Estrutura do mata-mata
│
├── Kaggle_Submission_Complete.csv
│   └── Arquivo final de submissão
│
├── Modelo.ipynb
│   └── Notebook principal do projeto
│
└── README.md
```
* O projeto foi estruturado para preservar a dimensão temporal das informações.

* Os ratings das seleções obtidos no SoFIFA foram armazenados em diretórios separados por ano/mês, contendo os arquivos HTML originais coletados em diferentes datas.

Da mesma forma, o ranking FIFA foi armazenado historicamente para que cada partida utilize apenas informações disponíveis até a data em que ocorreu.

## Base de Dados Utilizadas

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
  
## Como foi coletado cada informação

### Jogos das seleções

Dataset disponibilizado no kaggle, "International football results from 1872 to 2026":

* Link: [Dataset selection games 1872 - 2026](https://www.kaggle.com/datasets/martj42/international-football-results-from-1872-to-2017)
* Perfil: `Mart Jürisoo`.

### Classificação das seleções via EA Sports 26

Web Scraping do site `sofifa.com`, o qual resume os rankings de todas as seleções.

* Link: [sofifa site](https://sofifa.com/teams?type=national&showCol%5B0%5D=oa&showCol%5B1%5D=at&showCol%5B2%5D=md&showCol%5B3%5D=df&r=260036&set=true) 

### Ranking histórico do jogo

Requisição via API do site oficial da `Fifa`.

* Link: [Ranking Oficial da FIFA](https://inside.fifa.com/pt/fifa-rankings?intcmp=%28p_fifacom%29_%28d_insidefifa%29_%28c_webheader-main%29...)

## Tratamento dos Dados

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
    4. H2H (head to Head, histórico do confronto direto das 2 seleções)
    5. Intervalo entre jogos (em dias).

## Machine Learning

Para jogos da **fase de grupos**:
- **Time vencedor** — qual time vence a partida (use `casa`, `visitante` ou `empate`).

Para jogos da **fase eliminatória**:
- **Confronto** — quais dois times você prevê que jogarão nessa posição. Como a chave é determinada pelos resultados da fase de grupos, você precisa prever quais times avançarão o suficiente para se enfrentarem em cada fase.
- **Vencedor da partida** — qual equipe vence a partida (use `casa` ou `visitante`)
- **Pênaltis** — se a partida vai para a disputa de pênaltis (`Verdadeiro` ou `Falso`)
- **Formato do campeonato** - é preciso aplicar o formato do campeonato nos valores previstos.

---
* Modelos escolhido:
  * Resultados e pênaltis: ``LightGBM``, com uso da simulação `monte carlo`
---
## Simulação do Torneio (Monte Carlo)

Para preencher os slots do mata-mata de forma fundamentada, o torneio será simulado via Monte Carlo:

1. O modelo gera a distribuição de probabilidade de cada placar possível para cada jogo da fase de grupos.
2. O torneio é simulado N vezes (ex: 1.000 iterações), sorteando um placar a cada rodada conforme essas probabilidades.
3. Para cada slot do mata-mata, a seleção que avançou com maior frequência nas simulações é usado como previsão.

## Validação

* **Etapa de desenvolvimento:** treinar com dados até 2021 e testar na Copa do Mundo de 2022, permitindo avaliar a abordagem e ajustar features e hiperparâmetros com base em resultados reais.

### Resultados Fase de Grupos - 2022

* Total de jogos avaliados: 48 
* Precisão de Resultado (1x2): **52.1% (25/48)**
* Precisão de Placar Exato: **14.6% (7/48)**

| Confronto | Previsto | Real | Acertou Resultado? | Acertou Placar? |
|------------|----------|------|-------------------|-----------------|
| Qatar x Ecuador | 0 - 1 | 0 - 2 | ✅ | ❌ |
| Qatar x Senegal | 0 - 1 | 1 - 3 | ✅ | ❌ |
| Qatar x Netherlands | 0 - 1 | 0 - 2 | ✅ | ❌ |
| Ecuador x Senegal | 0 - 0 | 1 - 2 | ❌ | ❌ |
| Ecuador x Netherlands | 0 - 1 | 1 - 1 | ❌ | ❌ |
| Senegal x Netherlands | 0 - 1 | 0 - 2 | ✅ | ❌ |
| England x IR Iran | 1 - 0 | 6 - 2 | ✅ | ❌ |
| England x USA | 1 - 1 | 0 - 0 | ✅ | ❌ |
| England x Wales | 1 - 0 | 3 - 0 | ✅ | ❌ |
| IR Iran x USA | 0 - 1 | 0 - 1 | ✅ | 🎯 |
| IR Iran x Wales | 0 - 0 | 2 - 0 | ❌ | ❌ |
| USA x Wales | 1 - 1 | 1 - 1 | ✅ | 🎯 |
| Argentina x Saudi Arabia | 1 - 0 | 1 - 2 | ❌ | ❌ |
| Argentina x Mexico | 1 - 0 | 2 - 0 | ✅ | ❌ |
| Argentina x Poland | 1 - 0 | 2 - 0 | ✅ | ❌ |
| Saudi Arabia x Mexico | 0 - 1 | 1 - 2 | ✅ | ❌ |
| Saudi Arabia x Poland | 0 - 1 | 0 - 2 | ✅ | ❌ |
| Mexico x Poland | 1 - 0 | 0 - 0 | ❌ | ❌ |
| France x Australia | 2 - 0 | 4 - 1 | ✅ | ❌ |
| France x Denmark | 1 - 1 | 2 - 1 | ❌ | ❌ |
| France x Tunisia | 1 - 0 | 0 - 1 | ❌ | ❌ |
| Australia x Denmark | 0 - 1 | 1 - 0 | ❌ | ❌ |
| Australia x Tunisia | 0 - 0 | 1 - 0 | ❌ | ❌ |
| Denmark x Tunisia | 1 - 0 | 0 - 0 | ❌ | ❌ |
| Spain x Costa Rica | 1 - 0 | 7 - 0 | ✅ | ❌ |
| Spain x Germany | 1 - 1 | 1 - 1 | ✅ | 🎯 |
| Spain x Japan | 1 - 0 | 1 - 2 | ❌ | ❌ |
| Costa Rica x Germany | 0 - 1 | 2 - 4 | ✅ | ❌ |
| Costa Rica x Japan | 0 - 1 | 1 - 0 | ❌ | ❌ |
| Germany x Japan | 1 - 1 | 1 - 2 | ❌ | ❌ |
| Belgium x Canada | 2 - 0 | 1 - 0 | ✅ | ❌ |
| Belgium x Morocco | 2 - 0 | 0 - 2 | ❌ | ❌ |
| Belgium x Croatia | 0 - 0 | 0 - 0 | ✅ | 🎯 |
| Canada x Morocco | 1 - 1 | 1 - 2 | ❌ | ❌ |
| Canada x Croatia | 0 - 2 | 1 - 4 | ✅ | ❌ |
| Morocco x Croatia | 0 - 1 | 0 - 0 | ❌ | ❌ |
| Brazil x Serbia | 1 - 0 | 2 - 0 | ✅ | ❌ |
| Brazil x Switzerland | 1 - 0 | 1 - 0 | ✅ | 🎯 |
| Brazil x Cameroon | 1 - 0 | 0 - 1 | ❌ | ❌ |
| Serbia x Switzerland | 1 - 1 | 2 - 3 | ❌ | ❌ |
| Serbia x Cameroon | 1 - 0 | 3 - 3 | ❌ | ❌ |
| Switzerland x Cameroon | 1 - 0 | 1 - 0 | ✅ | 🎯 |
| Portugal x Ghana | 1 - 0 | 3 - 2 | ✅ | ❌ |
| Portugal x Uruguay | 0 - 1 | 2 - 0 | ❌ | ❌ |
| Portugal x Korea Republic | 1 - 0 | 1 - 2 | ❌ | ❌ |
| Ghana x Uruguay | 0 - 2 | 0 - 2 | ✅ | 🎯 |
| Ghana x Korea Republic | 1 - 1 | 3 - 2 | ❌ | ❌ |
| Uruguay x Korea Republic | 2 - 0 | 0 - 0 | ❌ | ❌ |

**Legenda:**
- ✅ Resultado correto (vitória, derrota ou empate)
- 🎯 Placar exato
- ❌ Erro

### Resultado mata-mata - 2022

* Total de jogos de mata-mata avaliados: 16
* (Acertos de Confronto Exato (Na fase correta): 12.5% (2/16)
* Precisão de Vencedor (nos 2 confrontos acertados): 100.0% (2/2)

| Fase (ID) | Confronto Previsto | Ocorreu? | Previsto P/ Avançar | Realmente Avançou | Acertou Vencedor? |
|-----------|-------------------|----------|---------------------|-------------------|-------------------|
| 49 | Netherlands x USA | ✅ Sim | Netherlands | Netherlands | ✅ |
| 50 | Argentina x France | ❌ Não ocorreu | France | - | - |
| 51 | England x Senegal | ✅ Sim | England | England | ✅ |
| 52 | Denmark x Mexico | ❌ Não ocorreu | Denmark | - | - |
| 53 | Spain x Croatia | ❌ Não ocorreu | Spain | - | - |
| 54 | Brazil x Portugal | ❌ Não ocorreu | Brazil | - | - |
| 55 | Belgium x Germany | ❌ Não ocorreu | Germany | - | - |
| 56 | Uruguay x Serbia | ❌ Não ocorreu | Uruguay | - | - |
| 57 | Netherlands x France | ❌ Não ocorreu | France | - | - |
| 58 | Spain x Brazil | ❌ Não ocorreu | Brazil | - | - |
| 59 | England x Denmark | ❌ Não ocorreu | Denmark | - | - |
| 60 | Germany x Uruguay | ❌ Não ocorreu | Germany | - | - |
| 61 | France x Brazil | ❌ Não ocorreu | France | - | - |
| 62 | Denmark x Germany | ❌ Não ocorreu | Germany | - | - |
| 63 | Brazil x Denmark | ❌ Não ocorreu | Brazil | - | - |
| 64 | France x Germany | ❌ Não ocorreu | France | - | - |

**Legenda:**
- ✅ Acerto
- ❌ Erro
- ``-`` Confronto não ocorreu devido a divergências nas fases anteriores do torneio

---

**Modelo final:** retreinado com todos os dados disponíveis (2018 até os jogos de 2026 pré-Copa) e submeter as previsões da Copa do Mundo de 2026.

### Resultados fase de grupos - 2026

* Total de jogos: 72
* % de cada seleção passar na respectiva posição do seu grupo.

**Grupo A**

| Seleção | 1º | 2º | 3º | 4º | Chance de Classificação |
|----------|----|----|----|----|-------------------------|
| 🇲🇽 Mexico | 33% | 29% | 23% | 15% | 🟩 79.6% |
| 🇰🇷 Korea Republic | 31% | 28% | 24% | 18% | 🟩 77.5% |
| 🇨🇿 Czech Republic | 28% | 27% | 27% | 18% | 🟩 74.6% |
| 🇿🇦 South Africa | 8% | 16% | 26% | 50% | 🟥 40.9% |

**Grupo B**

| Seleção | 1º | 2º | 3º | 4º | Chance de Classificação |
|----------|----|----|----|----|-------------------------|
| 🇨🇭 Switzerland | 57% | 26% | 12% | 4% | 🟩 93.6% |
| 🇨🇦 Canada | 25% | 31% | 28% | 16% | 🟩 77.5% |
| 🇧🇦 Bosnia and Herzegovina | 12% | 24% | 34% | 31% | 🟩 57.2% |
| 🇶🇦 Qatar | 7% | 19% | 26% | 49% | 🟥 41.5% |

**Grupo C**

| Seleção | 1º | 2º | 3º | 4º | Chance de Classificação |
|----------|----|----|----|----|-------------------------|
| 🇲🇦 Morocco | 40% | 34% | 20% | 6% | 🟩 89.7% |
| 🇧🇷 Brazil | 38% | 33% | 22% | 7% | 🟩 88.2% |
| 🇸🇨 Scotland | 19% | 26% | 36% | 20% | 🟩 68.5% |
| 🇭🇹 Haiti | 3% | 8% | 22% | 67% | 🟥 24.6% |

**Grupo D**

| Seleção | 1º | 2º | 3º | 4º | Chance de Classificação |
|----------|----|----|----|----|-------------------------|
| 🇹🇷 Turkey | 41% | 29% | 19% | 11% | 🟩 85.5% |
| 🇺🇸 USA | 37% | 31% | 20% | 12% | 🟩 82.4% |
| 🇵🇾 Paraguay | 12% | 23% | 32% | 33% | 🟥 54.8% |
| 🇦🇺 Australia | 9% | 17% | 29% | 45% | 🟥 44.7% |

**Grupo E**

| Seleção | 1º | 2º | 3º | 4º | Chance de Classificação |
|----------|----|----|----|----|-------------------------|
| 🇩🇪 Germany | 59% | 25% | 12% | 4% | ✅ 93.8% |
| 🇨🇮 Côte d'Ivoire | 21% | 38% | 26% | 15% | ✅ 78.1% |
| 🇪🇨 Ecuador | 16% | 27% | 36% | 21% | ✅ 65.9% |
| 🇨🇼 Curaçao | 4% | 10% | 25% | 60% | ❌ 29.5% |

**Grupo F**

| Seleção | 1º | 2º | 3º | 4º | Chance de Classificação |
|----------|----|----|----|----|-------------------------|
| 🇳🇱 Netherlands | 50% | 30% | 15% | 5% | ✅ 90.7% |
| 🇸🇪 Sweden | 34% | 35% | 20% | 10% | ✅ 86.1% |
| 🇯🇵 Japan | 12% | 24% | 37% | 27% | ✅ 59.4% |
| 🇹🇳 Tunisia | 4% | 10% | 27% | 58% | ❌ 29.3% |

**Grupo G**

| Seleção | 1º | 2º | 3º | 4º | Chance de Classificação |
|----------|----|----|----|----|-------------------------|
| 🇧🇪 Belgium | 62% | 23% | 11% | 3% | ✅ 95.4% |
| 🇪🇬 Egypt | 24% | 38% | 24% | 14% | ✅ 79.1% |
| 🇳🇿 New Zealand | 10% | 26% | 40% | 24% | ✅ 61.7% |
| 🇮🇷 Iran | 4% | 12% | 26% | 58% | ❌ 29.4% |

**Grupo H**

| Seleção | 1º | 2º | 3º | 4º | Chance de Classificação |
|----------|----|----|----|----|-------------------------|
| 🇪🇸 Spain | 75% | 19% | 5% | 1% | ✅ 98.3% |
| 🇺🇾 Uruguay | 20% | 50% | 21% | 9% | ✅ 83.6% |
| 🇨🇻 Cabo Verde | 3% | 17% | 38% | 42% | ❌ 38.0% |
| 🇸🇦 Saudi Arabia | 2% | 14% | 37% | 48% | ❌ 34.6% |

**Grupo I**

| Seleção | 1º | 2º | 3º | 4º | Chance de Classificação |
|----------|----|----|----|----|-------------------------|
| 🇫🇷 France | 39% | 31% | 21% | 9% | ✅ 86.8% |
| 🇳🇴 Norway | 40% | 29% | 20% | 11% | ✅ 83.8% |
| 🇸🇳 Senegal | 16% | 25% | 32% | 26% | ✅ 64.1% |
| 🇮🇶 Iraq | 5% | 14% | 27% | 54% | ❌ 36.1% |

**Grupo J**

| Seleção | 1º | 2º | 3º | 4º | Chance de Classificação |
|----------|----|----|----|----|-------------------------|
| 🇦🇷 Argentina | 47% | 32% | 15% | 6% | ✅ 91.2% |
| 🇦🇹 Austria | 40% | 37% | 16% | 7% | ✅ 89.2% |
| 🇩🇿 Algeria | 7% | 16% | 36% | 40% | ❌ 44.5% |
| 🇯🇴 Jordan | 5% | 15% | 33% | 47% | ❌ 36.2% |

**Grupo K**

| Seleção | 1º | 2º | 3º | 4º | Chance de Classificação |
|----------|----|----|----|----|-------------------------|
| 🇵🇹 Portugal | 56% | 27% | 12% | 5% | ✅ 92.6% |
| 🇨🇴 Colombia | 30% | 36% | 22% | 12% | ✅ 82.1% |
| 🇨🇩 Congo DR | 9% | 19% | 31% | 41% | ❌ 46.3% |
| 🇺🇿 Uzbekistan | 6% | 18% | 34% | 42% | ❌ 42.5% |

**Grupo L**

| Seleção | 1º | 2º | 3º | 4º | Chance de Classificação |
|----------|----|----|----|----|-------------------------|
| 🇪🇳 England | 55% | 27% | 12% | 6% | ✅ 92.2% |
| 🇭🇷 Croatia | 20% | 29% | 30% | 22% | ✅ 70.5% |
| 🇬🇭 Ghana | 13% | 24% | 28% | 35% | ✅ 56.1% |
| 🇵🇦 Panama | 13% | 20% | 30% | 37% | ❌ 52.1% |

* **Seleções que passaram em 3°:** ``'Czech Republic', 'Scotland', 'Ecuador', 'Senegal', 'New Zealand', 'Japan', 'Bosnia and Herzegovina', 'Ghana'``.

### Resultado mata-mata - 2026

* Total de jogos: 31.

| Colocação | Seleção | Fase Alcançada |
|------------|----------|----------------|
| 🥇 1 | Spain | Campeão |
| 🥈 2 | France | Vice-campeão |
| 🥉 3 | Portugal | 3º Lugar |
| 4 | Germany | 4º Lugar |
| 5-8 | Belgium | Semifinal |
| 5-8 | England | Semifinal |
| 5-8 | Argentina | Semifinal |
| 5-8 | Morocco | Semifinal |
| 9-16 | Korea Republic | Oitavas |
| 9-16 | Norway | Oitavas |
| 9-16 | Netherlands | Oitavas |
| 9-16 | Mexico | Oitavas |
| 9-16 | Croatia | Oitavas |
| 9-16 | Turkey | Oitavas |
| 9-16 | USA | Oitavas |
| 9-16 | Switzerland | Oitavas |
| 17-32 | Canada | 32-avos |
| 17-32 | Sweden | 32-avos |
| 17-32 | Scotland | 32-avos |
| 17-32 | Brazil | 32-avos |
| 17-32 | Côte d'Ivoire | 32-avos |
| 17-32 | New Zealand | 32-avos |
| 17-32 | Ecuador | 32-avos |
| 17-32 | Senegal | 32-avos |
| 17-32 | Czech Republic | 32-avos |
| 17-32 | Japan | 32-avos |
| 17-32 | Colombia | 32-avos |
| 17-32 | Austria | 32-avos |
| 17-32 | Bosnia and Herzegovina | 32-avos |
| 17-32 | Egypt | 32-avos |
| 17-32 | Uruguay | 32-avos |
| 17-32 | Ghana | 32-avos |
