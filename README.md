# APIs, energias renováveis e aprendizado de máquina

**Integrantes:** Rafaella Ferreira De Moraes — RM 571030
Aneliza Rondina Bonafe - RM 572977

## Objetivo

Duas tarefas **independentes** de aprendizado de máquina em Python, sobre dados públicos de energia renovável no Brasil. Em cada tarefa são treinados e comparados **três algoritmos**, com a mesma divisão de dados e as mesmas métricas:

1. **Classificação** — prever a fonte de geração (Solar, Eólica ou Hidráulica) de um empreendimento a partir de potência outorgada e localização.
2. **Regressão** — estimar a radiação solar horária em Petrolina (PE) a partir de variáveis meteorológicas e da hora do dia.

## Origem e período dos dados

| Arquivo | Fonte | Conteúdo | Período |
|---|---|---|---|
| `aneel_classificacao_orange.csv` | SIGA — ANEEL (API pública, sem token) | 3.876 empreendimentos de geração (`potencia_kw`, `latitude`, `longitude`, `fonte`) | Cadastro na data da consulta |
| `meteo_regressao_orange.csv` | API histórica Open-Meteo (sem token) | 1.001 horas (7h–17h) em Petrolina-PE, coord. aproximadas −9,39 / −40,50, fuso `America/Recife` | 01/04/2025 a 30/06/2025 |

Os dois CSVs foram gerados pelo notebook de apoio da avaliação e estão versionados neste repositório. Os valores meteorológicos são **estimativas de modelos/reanálise**, não medições de painel fotovoltaico, e `potencia_kw` é **potência outorgada** (capacidade), não energia gerada. Nenhuma credencial é usada ou publicada.

## Conteúdo do repositório

```
├── README.md
├── avaliacao_apis_renovaveis_ml.ipynb   # notebook completo, já executado
├── aneel_classificacao_orange.csv       # dados da Tarefa 1
├── meteo_regressao_orange.csv           # dados da Tarefa 2
├── resultados_classificacao.csv         # tabela de métricas da Tarefa 1
├── resultados_regressao.csv             # tabela de métricas da Tarefa 2
├── figures/                             # gráficos gerados pelo notebook
└── requirements.txt
```

## Como executar

```bash
pip install -r requirements.txt
jupyter notebook avaliacao_apis_renovaveis_ml.ipynb   # Kernel > Restart & Run All
```

* Os dois CSVs precisam estar na mesma pasta do notebook. Semente fixa (`SEED = 42`).
* O notebook lê os CSVs já gerados. Há uma célula opcional (`RODAR_API = False`) que reproduz o CSV meteorológico diretamente da API Open-Meteo, caso queira regenerá-lo (exige internet e `requests`).

## Tarefa 1 — Classificação da fonte renovável

**Entradas:** `potencia_kw`, `latitude`, `longitude` (nenhum nome, código CEG, sigla ou descrição). **Alvo:** `fonte`.

**Preparação:** 47 registros com coordenadas (0, 0) foram tratados como ausentes e imputados pela mediana do treino; 28 linhas exatamente duplicadas foram removidas (3.848 linhas restantes); `log1p` na potência e padronização dentro do pipeline (apenas para Regressão Logística e KNN).

**Avaliação:** divisão **estratificada 80/20** (`random_state=42`), a mesma para os três modelos (3.078 treino / 770 teste). Métricas multiclasse: **macro** (principal) e **weighted**.

| Modelo | Accuracy | Precision (macro) | Recall (macro) | F1 (macro) | F1 (weighted) |
|---|---|---|---|---|---|
| Regressão Logística | 0,829 | 0,844 | 0,826 | 0,824 | 0,828 |
| KNN (k=5) | 0,971 | 0,971 | 0,971 | 0,971 | 0,971 |
| Random Forest (300 árvores) | 0,977 | 0,977 | 0,976 | 0,977 | 0,977 |

**Conclusões**

* Random Forest e KNN têm desempenho muito próximo (diferença de ~4 empreendimentos em 770); a Regressão Logística é claramente pior, pois a relação entre localização e fonte não é linear.
* A confusão dominante na Regressão Logística é **Solar → Eólica** (62 casos), seguida de Hidráulica → Eólica (27). KNN e Random Forest erram pouco e de forma distribuída.
* Limitações: potência outorgada não é geração; a potência típica de cada fonte no cadastro (solar com mediana de 1 kW) facilita a separação; coordenadas aproximadas e algumas ausentes; e empreendimentos vizinhos e parecidos podem estar em treino e teste, o que pode deixar a acurácia otimista (não testado aqui).

## Tarefa 2 — Regressão da radiação solar

**Entradas:** `temperatura_c`, `umidade_pct`, `nuvens_pct`, `vento_kmh`, `hora`. **Alvo:** `radiacao_w_m2`. `data_hora` só ordena e separa; `radiacao_w_m2` não entra em X.

**Avaliação:** divisão **temporal sem embaralhar** — primeiras 80% das horas para treino (800; 01/04 a 12/06) e últimas 20% para teste (201; 12/06 a 30/06), a mesma para os três modelos.

| Modelo | MAE (W/m²) | MSE ((W/m²)²) | R² |
|---|---|---|---|
| Regressão Linear | 145,2 | 30.034 | 0,360 |
| Random Forest (300 árvores) | 66,4 | 7.210 | 0,846 |
| Gradient Boosting (200 est.) | 66,7 | 7.374 | 0,843 |

**Conclusões**

* Random Forest e Gradient Boosting praticamente empatam e reduzem o erro a menos da metade do da Regressão Linear.
* A radiação tem forma de sino ao longo do dia (média de ~83 W/m² às 7h, ~728 às 12h, ~153 às 17h). Como a relação com `hora` não é linear, a Regressão Linear quase não a aproveita, enquanto as árvores sim: **sem `hora`, o Random Forest cai de R² 0,846 para 0,337**.
* Estimar radiação **não equivale** a prever geração elétrica: a radiação é global horizontal, e a geração depende de inclinação e orientação do painel, temperatura e eficiência do módulo, perdas no inversor, sombreamento, sujeira e potência instalada, além de ser energia (kWh) e não W/m².
* Limitações: 1.001 horas, um único local, ~3 meses, dados de reanálise, hiperparâmetros não ajustados.

## Gráficos

Gerados pelo notebook e salvos em `figures/`: exploração das duas tarefas, matrizes de confusão, valores reais × previstos e série temporal do teste.
