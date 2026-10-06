# Trabalho de Inteligência Computacional: Busca Heurística e Árvores de Decisão

**Instituição:** Centro Universitário do Pará (CESUPA)  
**Disciplina:** Inteligência Computacional  
**Docente:** Profa. Polyana Santos Fonseca Nascimento  
**Equipe:** Ana Alice Dias, Caio Vasconcelos e Yasmin Barata 
**Modalidades do Trabalho:** I. Busca heurística (Parte I) e IV. Árvore de decisão (Parte II)  

---

## Visão Geral

Este repositório contém a implementação prática, análises numéricas e visualizações gráficas desenvolvidas para o trabalho de Inteligência Computacional. O estudo é dividido em duas partes principais:

1. **Parte I — Busca Heurística:** Aplicação dos algoritmos **Busca Gulosa**, **A\*** e **A\* Bidirecional** para encontrar rotas em malhas urbanas reais obtidas via OpenStreetMap (San Francisco, EUA).
2. **Parte II — Árvores de Decisão:** Classificação da qualidade de vinhos tintos (*Wine Quality Red*) com base em parâmetros físico-químicos, comparando uma **Árvore de Decisão C4.5** (usando entropia) com o modelo ensemble **Random Forest**.

---

## Declaração de Uso de IA Generativa

* **Ferramenta Utilizada:** Claude (Anthropic), via Claude Code.
* **Finalidade:** Apoio na estruturação das funções de medição do tempo/memória, geração de gráficos, correção e ajuste no código da Parte I (pesos de aresta, heurística de grande círculo e critério de parada de Pohl no A\* Bidirecional), além de revisão estilística dos textos.
* **Extensão do Uso:** Parcial. A definição dos problemas, seleção dos datasets, escolha dos parâmetros e interpretação dos resultados foram realizadas pela equipe.
* **Validação:** Todos os resultados de caminho mínimo foram validados contra o algoritmo de referência (Dijkstra do `NetworkX`), o notebook foi executado integralmente em ambiente limpo e todo o código e documentação foram revisados pela equipe.

---

## Como Reproduzir

### Pré-requisitos
* Python 3.12+
* Conexão com a internet (para download automático da malha viária de San Francisco via OSMnx e do conjunto de dados de vinhos via OpenML).

### Passo a Passo

1. **Clonar o repositório e criar um ambiente virtual:**
   ```bash
   python -m venv venv
   source venv/bin/activate  # No Windows: venv\Scripts\activate
   ```

2. **Instalar as dependências necessárias:**
   ```bash
   pip install -r requirements.txt
   ```
   *Dependências principais:* `networkx`, `osmnx`, `scikit-learn`, `pandas`, `matplotlib`.

3. **Executar o Notebook:**
   Abra e execute o arquivo `trabalho_ia.ipynb` na ordem sequencial (*Kernel* $\rightarrow$ *Restart & Run All*).

---

## Parte I: Busca Heurística — Rota Mais Curta em San Francisco

### Modelagem do Problema
* **Grafo:** Direcionado (`MultiDiGraph`) com **9.975 nós** (cruzamentos) e **27.547 arestas** (ruas), considerando a maior componente fortemente conexa de San Francisco.
* **Estados e Ações:** Cada estado $n$ é um cruzamento com coordenadas $(\text{lat}, \text{lon})$. Uma ação é percorrer uma rua até o cruzamento seguinte.
* **Custo $c(u, v)$:** Comprimento da rua em metros.
* **Heurística $h(n)$:** Distância em linha reta (grande círculo / *great circle*) entre o nó $n$ e o nó de destino $d$, medida em metros.
  * **Admissibilidade:** Como a menor distância entre dois pontos no espaço é a linha reta, $h(n) \le h^*(n)$ para todo $n$.
  * **Consistência:** Respeita a desigualdade triangular $h(n) \le c(n, p) + h(p)$.

### Algoritmos Comparados
* **Busca Gulosa:** Expande o nó com menor $h(n)$. Rápida, mas sem garantia de otimalidade.
* **A\*:** Expande o nó com menor $f(n) = g(n) + h(n)$. Garante a rota ótima com heurística admissível.
* **A\* Bidirecional:** Executa duas buscas A\* concorrentes (uma da Origem $\rightarrow$ Destino e outra do Destino $\rightarrow$ Origem). Adota o **critério de parada de Pohl** ($\max(f_F, f_B) \ge \text{melhor\\_custo}$).

### Resultados — Trajeto Principal (Origem e Destino Fixos)
* **Distância em linha reta:** 6.727 m
* **Custo Ótimo de Referência (Dijkstra):** 15.819,74 m

| Algoritmo | Custo (m) | Desvio do Ótimo | Nós Expandidos | Pico da Fila OPEN | Tempo (s) | Nós no Caminho |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **Busca Gulosa** | 16.688,16 | +5,5% | 90 | 52 | ~0,0010 s | 43 |
| **A\*** | 15.819,74 | **+0,0% (Ótimo)** | 1.221 | 208 | ~0,0122 s | 29 |
| **A\* Bidirecional** | 15.819,74 | **+0,0% (Ótimo)** | 8.907 | 303 | ~0,1013 s | 29 |

### Benchmarking em Outros Trajetos Reais

| Trajeto | Algoritmo | Custo (m) | Acima do Ótimo | Nós Expandidos | Pico OPEN |
| :--- | :--- | :---: | :---: | :---: | :---: |
| **Fisherman's Wharf $\rightarrow$ Mission Dolores** | Gulosa | 6.719,71 | +10,0% | 84 | 90 |
| *(Ótimo: 6.107,39 m)* | A\* | 6.107,39 | **+0,0%** | 695 | 169 |
| | A\* Bidirecional | 6.107,39 | **+0,0%** | 1.625 | 328 |
| **Ocean Beach $\rightarrow$ Ferry Building** | Gulosa | 14.191,62 | +13,4% | 121 | 130 |
| *(Ótimo: 12.511,86 m)* | A\* | 12.511,86 | **+0,0%** | 2.466 | 283 |
| | A\* Bidirecional | 12.511,86 | **+0,0%** | 5.866 | 583 |
| **Presidio $\rightarrow$ Bayview** | Gulosa | 17.476,65 | +28,2% | 183 | 207 |
| *(Ótimo: 13.629,60 m)* | A\* | 13.629,60 | **+0,0%** | 4.153 | 269 |
| | A\* Bidirecional | 13.629,60 | **+0,0%** | 10.027 | 483 |
| **Twin Peaks $\rightarrow$ Fisherman's Wharf** | Gulosa | 12.205,72 | +12,3% | 102 | 113 |
| *(Ótimo: 10.870,46 m)* | A\* | 10.870,46 | **+0,0%** | 1.965 | 240 |
| | A\* Bidirecional | 10.870,46 | **+0,0%** | 6.365 | 362 |

### Principais Conclusões da Parte I
1. **Gulosa vs. A\*:** A Busca Gulosa expande significativamente menos nós, mas entrega rotas entre **5,5% e 28,2% mais longas**, pois ignora o custo já acumulado $g(n)$.
2. **Comportamento do A\* Bidirecional:** O A\* Bidirecional garantiu a otimalidade em 100% dos casos. Contudo, expandiu mais nós que o A\* unidirecional tradicional. Isso ocorre porque o critério de parada rigoroso de Pohl exige garantir que nenhum caminho restante na OPEN possa ser menor que o melhor encontro atual, forçando a busca a explorar frentes adicionais.

---

## Parte II: Árvores de Decisão — Classificação da Qualidade de Vinhos

### Modelagem do Problema
* **Dataset:** *Wine Quality Red* (OpenML) — 1.599 amostras de vinho tinto com 11 propriedades físico-químicas.
* **Alvo de Classificação:** Variável binária: **Bom** (Nota $\ge 6$, representando 53,5% dos dados) vs. **Comum** (Nota $< 6$, representando 46,5%).
* **Divisão de Dados:** 80% para Treino (1.279 amostras) e 20% para Teste (320 amostras), mantendo estratificação.
* **Meta de Desempenho Estabelecida:** Acurácia mínima de **70% no conjunto de teste** mantendo a estrutura da árvore a mais enxuta e legível possível.

### Algoritmos Avaliados
* **Árvore de Decisão:** `DecisionTreeClassifier` com critério de ganho por **Entropia** (modelo equivalente ao C4.5).
* **Random Forest:** `RandomForestClassifier` com 100 estimadores para comparação de capacidade preditiva maximizada.

### Seleção da Profundidade Ideal da Árvore

| Profundidade | Folhas (Regras) | Acurácia no Teste | Atinge a Meta ($\ge 70\%$) |
| :---: | :---: | :---: | :---: |
| 1 | 2 | 69,1% | Não |
| 2 | 4 | 69,1% | Não |
| 3 | 8 | 68,4% | Não |
| **4** | **16** | **70,9%** | **Sim (Escolha Ótima)** |
| 5 | 28 | 70,0% | Sim |
| 6 | 44 | 72,8% | Sim |
| 7 | 64 | 73,8% | Sim |
| 8 | 86 | 71,9% | Sim |

* **Profundidade Escolhida:** **Profundidade 4** foi a menor estrutura que superou a meta de 70% de acurácia, gerando um modelo enxuto com apenas 16 regras finais (folhas).

### Comparativo de Desempenho

| Modelo | Acurácia no Teste | Qtd. de Árvores | Profundidade | Folhas |
| :--- | :---: | :---: | :---: | :---: |
| **Árvore de Decisão (C4.5)** | **70,9%** | 1 | 4,0 | 16,0 |
| **Random Forest** | **80,6%** | 100 | 17,3 | 190,2 |

### Principais Atributos Decisivos (Feature Importance)
As variáveis que mais influenciaram a classificação da qualidade dos vinhos em ambos os modelos foram:
1. **Teor Alcoólico (`alcohol`):** Atributo mais crítico na primeira divisão da árvore (`alcohol <= 10.53`).
2. **Sulfatos (`sulphates`):** Relevante na differentiation de vinhos com bom corpo e conservação.
3. **Acidez Volátil (`volatile_acidity`):** Níveis elevados indicam acidez indesejada (vinagre), reduzindo a nota do vinho.

### Teste de Estabilidade (5 Divisões de Treino/Teste)

| Seed | Acurácia Árvore (Prof. 4) | Acurácia Random Forest | Primeira Pergunta da Árvore |
| :---: | :---: | :---: | :---: |
| 0 | 74,1% | 83,4% | `alcohol <= 10.53` |
| 1 | 72,5% | 79,1% | `alcohol <= 10.25` |
| 2 | 73,1% | 82,8% | `alcohol <= 10.53` |
| 3 | 74,1% | 77,8% | `alcohol <= 9.97` |
| 42 | 70,9% | 80,6% | `alcohol <= 10.53` |
| **Média** | **72,9%** | **80,7%** | — |

---

## Referências

* RUSSELL, S.; NORVIG, P. *Inteligência Artificial: uma abordagem moderna*. 4. ed. Rio de Janeiro: GEN LTC, 2022.
* HART, P. E.; NILSSON, N. J.; RAPHAEL, B. A Formal Basis for the Heuristic Determination of Minimum Cost Paths. *IEEE Transactions on Systems Science and Cybernetics*, v. 4, n. 2, p. 100–107, 1968.
* POHL, I. Bi-directional Search. *Machine Intelligence*, v. 6, p. 127–140, 1971.
* BOEING, G. OSMnx: New Methods for Acquiring, Constructing, Analyzing, and Visualizing Complex Street Networks. *Computers, Environment and Urban Systems*, v. 65, p. 126–139, 2017.
* QUINLAN, J. R. *C4.5: Programs for Machine Learning*. San Mateo: Morgan Kaufmann, 1993.
* BREIMAN, L. Random Forests. *Machine Learning*, v. 45, p. 5–32, 2001.
* CORTEZ, P. et al. Modeling wine preferences by data mining from physicochemical properties. *Decision Support Systems*, v. 47, n. 4, p. 547–553, 2009.
* PEDREGOSA, F. et al. Scikit-learn: Machine Learning in Python. *Journal of Machine Learning Research*, v. 12, p. 2825–2830, 2011.
