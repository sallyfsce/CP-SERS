# Checkpoint 02 — APIs, energias renováveis e aprendizado de máquina

**INTEGRANTES:**
Aneliza Rondina Bonafé - RM 572977
Rafaella Ferreira de Moraes - 571030

Projeto acadêmico com duas tarefas de aprendizado de máquina em Python: classificação da fonte de geração renovável e regressão da radiação solar.

## Prof, é preciso colocar os dois CSVs gerados pelo seu notebook(aneel_classificacao e meteo_regressao)na pastinha do colab pra poder rodar tudo direitinho. Obrigada! :)

## Objetivo

1. **Classificação:** prever se um empreendimento é Solar, Eólico ou Hidráulico usando apenas potência outorgada e localização.
2. **Regressão:** estimar a radiação solar horária em Petrolina (PE) a partir de variáveis meteorológicas e da hora do dia.

## Fontes e dados

### Tarefa 1 — SIGA/ANEEL

O arquivo `aneel_classificacao_orange.csv` possui **3.876 linhas**, sem valores ausentes, com:

- `potencia_kw`
- `latitude`
- `longitude`
- `fonte`

Distribuição das classes:

| Fonte | Quantidade |
|---|---:|
| Hidráulica | 1.476 |
| Solar | 1.200 |
| Eólica | 1.200 |

A coluna `potencia_kw` representa potência outorgada, não energia produzida. Não foram usadas como entrada informações que entregam diretamente a classe, como nome, código CEG, sigla ou descrição.

### Tarefa 2 — Open-Meteo

O arquivo `meteo_regressao_orange.csv` possui **1.001 registros horários**, de 7h a 17h, para Petrolina (PE), coordenadas aproximadas **-9,39, -40,50**, no período de **01/04/2025 a 30/06/2025**, usando o fuso `America/Recife`.

Entradas:

- `temperatura_c`
- `umidade_pct`
- `nuvens_pct`
- `vento_kmh`
- `hora`

Alvo:

- `radiacao_w_m2`

Não há valores ausentes. Os dados históricos de radiação são estimativas de modelos/reanálise, não medições de um painel fotovoltaico.

## Modelos e resultados

### Tarefa 1 — Classificação

Foi usada divisão **estratificada 80%/20%**, com `random_state=42`. As mesmas divisões foram usadas pelos três modelos.

A padronização foi feita dentro de `Pipeline` nos algoritmos que precisam dela (Regressão Logística e SVM). As métricas multiclasses apresentadas abaixo usam **média macro**, dando o mesmo peso para cada classe.

| Modelo | Accuracy | Precision macro | Recall macro | F1 macro |
|---|---:|---:|---:|---:|
| Regressão Logística | 0,8247 | 0,8282 | 0,8214 | 0,8197 |
| Random Forest | **0,9755** | **0,9769** | **0,9741** | **0,9753** |
| SVM (RBF) | 0,8634 | 0,8689 | 0,8599 | 0,8568 |

A **Random Forest** teve o melhor desempenho. A Regressão Logística e a SVM apresentam mais confusões entre as fontes, principalmente envolvendo Eólica e Hidráulica/Solar. Isso mostra que a relação entre localização, potência e fonte não é simplesmente linear.

Mesmo com alta acurácia, existem limitações: potência outorgada não é geração real; a localização e a potência não descrevem todas as características de uma usina; e empreendimentos próximos podem possuir características semelhantes. Portanto, o modelo não deve ser tratado como uma forma universal de identificar a fonte.

### Tarefa 2 — Regressão

As primeiras **80% das horas** foram usadas para treino e as últimas 20% para teste, preservando a ordem temporal. Foram usados os mesmos conjuntos para os três modelos.

| Modelo | MAE (W/m²) | MSE ((W/m²)²) | R² |
|---|---:|---:|---:|
| Regressão Linear | 145,2049 | 30034,2011 | 0,3598 |
| Random Forest | **66,3994** | **7210,0838** | **0,8463** |
| Gradient Boosting | 66,7341 | 7373,9345 | 0,8428 |

A **Random Forest** apresentou o menor erro e o maior R², com desempenho muito próximo do Gradient Boosting.

A variável `hora` tem papel importante porque a radiação solar muda fortemente ao longo do dia, com valores menores no começo e no fim do intervalo e maiores perto do meio do dia. A relação não é linear, o que ajuda a explicar o desempenho inferior da Regressão Linear.

Estimar radiação **não equivale a prever geração elétrica**. A radiação é medida em W/m², enquanto a geração depende também de potência instalada, orientação e inclinação dos módulos, eficiência, temperatura, sombreamento, perdas elétricas, inversores e outras características do sistema.

## Como executar

Requisitos:

```bash
pip install -r requirements.txt
```

Depois:

```bash
jupyter notebook checkpoint_02_sers.ipynb
```

O notebook usa os dois CSVs já incluídos no repositório. Ele também contém uma seção opcional para reproduzir os arquivos consultando as APIs públicas. Por padrão, essa consulta está desativada (`GERAR_CSVS = False`), para que o notebook possa ser executado usando os CSVs versionados.

## Estrutura

```text
CP-SERS/
├── README.md
├── checkpoint_02_sers.ipynb
├── aneel_classificacao_orange.csv
├── meteo_regressao_orange.csv
├── resultados_classificacao.csv
├── resultados_regressao.csv
└── requirements.txt
```


