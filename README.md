# Modelagem Hierárquica Bayesiana — Projeção Eleitoral Presidencial 2026

Repositório com o pipeline analítico completo para estimativa da intenção latente de votos válidos para a eleição presidencial de 2026 no Brasil. 

O projeto implementa um **modelo dinâmico de espaço de estados (passeio aleatório em escala logit)** implementado em **PyMC**, calibrado com os desvios empíricos das pesquisas eleitorais dos ciclos de 2018 e 2022 e testado sob análise formal de sensibilidade de *priors* (*ablation study*).

---

## 🎯 Destaques e Resultados Principais

* **Convergência MCMC Rigorosa:** Amostragem via algoritmo NUTS (No-U-Turn Sampler) com $\hat{R} \le 1,002$, 0 transições divergentes e tamanho efetivo de amostra (ESS) $> 1.400$.
* **Robustez Inequívoca:** Em todos os cenários de teste de sensibilidade, a probabilidade posterior de vitória supera **88%**, aproximando-se de certeza estatística ($> 98\%$) sob calibração empírica da indústria.
* **Efeitos de Casa Estreitos:** A dispersão idiossincrática entre os institutos em 2026 revelou-se baixa ($\tau_{\text{inst}} \approx 0,02$ logit), indicando forte alinhamento metodológico na reta final.

### Comparativo do Estudo de Sensibilidade (*Ablation Study*)

| Cenário | Hipótese Metodológica | Votos Válidos (Flávio) | IC 95% Posterior | Votos Válidos (Lula) | Probabilidade de Vitória |
| :--- | :--- | :---: | :---: | :---: | :---: |
| **Cenário A** | Pesquisas Puras / Priori Neutra ($\eta = 0$) | **51,08%** | [49,43% – 52,72%] | 48,92% | **88,05%** |
| **Cenário B** | Correção Moderada ($\eta \approx -0,08$) | **53,12%** | [50,68% – 55,56%] | 46,88% | **98,60%** |
| **Cenário C** | Calibração Plena 2018/2022 ($\eta \approx -0,15$) | **54,84%** | [50,00% – 59,47%] | 45,16% | **98,60%** |

---

## 🔬 Metodologia e Fundamentação

### 1. Dinâmica de Espaço de Estados (State-Space Model)
A intenção de voto latente $\mu_t$ no dia $t$ evolui de acordo com um passeio aleatório no espaço logit:
$$\mu_t \sim \mathcal{N}(\mu_{t-1}, \sigma^2_{\text{tempo}})$$

Para uma pesquisa $i$ conduzida no dia $t[i]$ pelo instituto $j[i]$, a intenção observada $y_i$ é modelada como:
$$\text{logit}(y_i) \sim \mathcal{N}\left(\mu_{t[i]} + \alpha_{j[i]} + \eta_{\text{indústria}}, \, \sigma^2_{i}\right)$$

Onde:
* $\alpha_j \sim \mathcal{N}(0, \tau^2_{\text{inst}})$ captura o efeito individual de cada instituto (*house effect*).
* $\eta_{\text{indústria}}$ representa o viés sistemático agregado da indústria.
* $\sigma_i = \sqrt{\frac{p_i(1 - p_i)}{n_i}}$ representa o erro padrão amostral nominal da sondagem.

### 2. Por que a calibração histórica parte de 2018?
1. **Quebra Estrutural (*Structural Break*):** A eleição de 2018 marcou o fim da estabilidade bipartidária tradicional (PT vs. PSDB), redefinindo a abstenção e a polarização sociopolítica. Incluir ciclos anteriores a 2018 contaminaria a distribuição a priori com padrões comportamentais obsoletos.
2. **Viés Sistêmico de Indústria:** Tanto em 2018 quanto em 2022, observou-se empiricamente uma subnotificação direcional correlacionada em candidaturas conservadoras nas pesquisas de véspera, caracterizando um efeito fixo de indústria e não mero ruído aleatório.
3. **Mudanças Metodológicas:** A migração de métodos presenciais domiciliares para modelos telefônicos/CATI e pontos de fluxo consolidou-se nessa janela, tornando as correlações de erro diretamente comparáveis à realidade de 2026.

---

## 📁 Estrutura do Repositório

```text
├── 01_extracao_e_estruturacao_dados.ipynb   # Sanitização e estruturação dos dados históricos e pesquisas de 2026
├── 02_modelo_hierarquico_bayesiano.ipynb     # Especificação do modelo PyMC, amostragem NUTS e diagnósticos de convergência
├── 03_analise_sensibilidade_e_graficos.ipynb # Ablation study dos cenários A, B e C e renderização dos gráficos
├── grafico_01_serie_temporal_passeio_aleatorio.png # Trajetória latente com faixas de credibilidade (50%, 80%, 95%)
├── grafico_02_densidade_posterior_dia_eleicao.png   # Curva marginal de probabilidade no dia da votação
└── grafico_03_forest_plot_house_effects.png        # Forest plot com dispersão dos efeitos por instituto
```
---

## 📊 Visualizações dos Resultados

Os gráficos gerados em 300 DPI detalham a inferência:
1. **Série Temporal do Passeio Aleatório:** Mostra a convergência das pesquisas sobrepostas à curva contínua da intenção latente até a véspera do pleito.
2. **Densidade Posterior:** Demonstra a concentração da distribuição marginal acima do limiar de 50,0% de votos válidos.
3. **Forest Plot de Efeitos de Casa:** Evidencia a dispersão mínima entre os institutos de pesquisa avaliados em 2026.
---

## ⚙️ Como Reproduzir

### Pré-requisitos
* Python 3.10+
* Bibliotecas:
```bash
pip install pymc arviz numpy pandas matplotlib seaborn scipy
```

### Execução
Execute os cadernos na ordem numérica sequencial (`01` ➔ `02` ➔ `03`).

---

## ⚖️ Nota de Isenção / Disclaimer
Este projeto é um estudo de inferência e modelagem bayesiana preditiva aplicada a sondagens eleitorais amostrais. Para dados oficiais de apuração, totalização e boletins de urna, consulte o [Tribunal Superior Eleitoral (TSE)](https://resultados.tse.jus.br/).
