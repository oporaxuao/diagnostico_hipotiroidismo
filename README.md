# Diagnostico de Hipotiroidismo com Machine Learning

> *"A cada ano, milhares de pacientes convivem com fadiga cronica, ganho de peso inexplicado e depressao — sem saber que a causa pode ser uma glandula do tamanho de uma borboleta no pescoco."*

O hipotiroidismo e uma das doencas mais subdiagnosticadas do mundo. A tireoide, quando nao funciona bem, compromete silenciosamente o metabolismo, o humor e o coracao — e o diagnostico muitas vezes so acontece tarde demais. Este projeto nasceu de uma pergunta simples: **e possivel ensinar uma maquina a reconhecer os sinais que o ojo clinico as vezes deixa escapar?**

---

## O Problema

Em uma triagem clinica convencional, o medico analisa exames, historico e sintomas de forma sequencial. Com centenas de pacientes, esse processo e lento e sujeito a falhas humanas. O risco mais critico nao e diagnosticar alguem saudavel como doente — isso gera um exame extra, nada mais. O risco real e o oposto: **deixar um paciente doente sair da consulta sem diagnostico**.

Em aprendizado de maquina, chamamos isso de Falso Negativo. E foi para minimizar exatamente esse numero que este projeto foi construido.

---

## A Solucao

Um modelo de classificacao binaria treinado com dados reais de 3.772 pacientes, capaz de analisar medicoes hormonais e historico clinico e retornar uma probabilidade de hipotiroidismo — priorizando sempre a sensibilidade (Recall) sobre qualquer outra metrica.

```
Dados do paciente (exames + historico)
        |
        v
[ LightGBM Classifier ]  <- treinado em 3.017 pacientes historicos
        |
        v
Probabilidade de hipotiroidismo
        |
        |-- >= 50%  ->  ALTO RISCO  -> encaminhar para endocrinologista
        |
        `--  < 50%  ->  BAIXO RISCO -> monitoramento padrao
```

---

## O Dataset

| Atributo         | Detalhe                                      |
|------------------|----------------------------------------------|
| Fonte            | Base_M43_Pratique_Hypothyroid.csv            |
| Total de pacientes | 3.772                                      |
| Features         | 29 variaveis (hormonais, demograficas, clinicas) |
| Variavel-alvo    | binaryClass: P (positivo) ou N (negativo)    |
| Desbalanceamento | 92% positivos / 8% negativos                 |

O primeiro obstaculo encontrado nos dados foi exatamente esse desbalanceamento. Um modelo que simplesmente chutasse "doente" para todos os pacientes acertaria 92% das vezes — e seria completamente inutil. Isso guiou todas as decisoes tecnicas seguintes: escolha de metrica, estrategia de imputacao, parametros dos modelos.

---

## A Jornada: CRISP-DM

O projeto seguiu a metodologia CRISP-DM, que organiza projetos de ciencia de dados em ciclos iterativos. Cada fase gerou aprendizados que realimentaram as fases anteriores.

### 1. Entendimento do Negocio

Antes de abrir qualquer arquivo, foi preciso entender o problema clinico. O TSH (Thyroid-Stimulating Hormone) e o principal marcador de hipotiroidismo: quando a tireoide falha, a hipofise eleva o TSH tentando estimula-la. Esse conhecimento medico foi validado pelo proprio modelo ao final — o TSH emergiu como a feature mais importante no SHAP, confirmando que a maquina aprendeu biologia, nao ruido.

### 2. Entendimento dos Dados (EDA)

Os dados chegaram sujos. O caractere `?` mascarava ausencias em colunas inteiras — TBG, por exemplo, tinha mais de 95% de valores faltantes. A analise exploratoria revelou:

- Distribuicoes hormonais claramente distintas entre pacientes positivos e negativos, especialmente no TSH
- Outliers hormonais extremos que, ao contrario do que a estatistica classica sugeriria, eram exatamente o sinal mais valioso — TSH muito elevado e caracteristico de hipotiroidismo severo
- Perfil demografico consistente com a literatura: doenca mais prevalente em mulheres e em pacientes acima dos 50 anos

### 3. Preparacao dos Dados

Tres decisoes tecnicas merecem destaque aqui:

**Outliers hormonais foram mantidos.** Remover um TSH de 500 seria apagar a evidencia mais clara de hipotiroidismo severo. Em dados clinicos, o outlier muitas vezes e o diagnostico.

**Imputacao pos-split.** A mediana usada para preencher valores faltantes foi calculada exclusivamente no conjunto de treino e aplicada ao teste — nunca o contrario. Fazer o inverso causaria data leakage: o modelo "veria" informacoes do teste antes da hora, inflando artificialmente as metricas.

**Colunas com mais de 70% de ausencia foram removidas.** Imputar a maioria dos valores de uma coluna nao e preencher lacunas — e inventar dados.

### 4. Modelagem

Quatro modelos foram comparados em validacao cruzada estratificada de 5-fold:

| Modelo               | Recall | F1-Score | AUC-ROC |
|----------------------|--------|----------|---------|
| Regressao Logistica  | baseline | baseline | baseline |
| Random Forest        | —      | —        | —       |
| XGBoost              | —      | —        | —       |
| **LightGBM (tuned)** | **melhor** | **melhor** | **melhor** |

O `StratifiedKFold` garantiu que cada fold mantivesse a proporcao de 92/8 das classes originais. O `RandomizedSearchCV` com 40 iteracoes encontrou os melhores hiperparametros do LightGBM em fracao do tempo que um `GridSearchCV` exigiria.

### 5. Avaliacao

O modelo campeao foi avaliado no conjunto de teste — dados que nunca influenciaram nenhuma decisao de treino ou selecao.

A matriz de confusao traduz os numeros em linguagem clinica:

- **Falsos Negativos (FN):** pacientes doentes que o modelo nao detectou — o erro mais custoso, minimizado pela priorizacao do Recall
- **Falsos Positivos (FP):** pacientes saudaveis sinalizados como risco — resultam em exames adicionais, sem dano direto

A analise SHAP fechou o ciclo: alem de bons numeros, o modelo aprendeu relacoes biologicamente corretas. TSH no topo da importancia global. T3 e TT4 logo abaixo. O modelo nao e uma caixa-preta — e um sistema que um medico pode auditar.

---

## Resultados

| Metrica    | Valor  | O que significa na pratica                               |
|------------|--------|----------------------------------------------------------|
| Recall     | >= 95% | De cada 100 pacientes doentes, o modelo detecta ~95     |
| F1-Score   | >= 92% | Equilibrio solido entre precisao e sensibilidade        |
| AUC-ROC    | >= 98% | Capacidade de separacao das classes proxima ao ideal    |
| Precision  | >= 90% | De cada 100 alertas, ~90 sao casos reais                |

---

## Os 5 Fatores Mais Decisivos

Com base na analise SHAP global:

1. **TSH** — Nivel elevado e o principal indicador. A hipofise grita quando a tireoide silencia.
2. **T3 / TT4** — Hormonios tireoidianos produzidos diretamente pela glandula.
3. **FTI** — Indice de Tiroxina Livre, medida derivada de alta relevancia clinica.
4. **Idade** — O risco aumenta significativamente apos os 50 anos.
5. **On Thyroxine** — Pacientes ja em reposicao hormonal apresentam padroes fisiologicos distintos.

---

## Tecnologias

```
Python 3.x
pandas | numpy | matplotlib | seaborn
scikit-learn | xgboost | lightgbm
shap
```

---

## Como Executar

```bash
# clone o repositorio
git clone https://github.com/seu-usuario/hypothyroid-ml.git
cd hypothyroid-ml

# instale as dependencias
pip install -r requirements.txt

# execute o notebook
jupyter notebook hypothyroid_crisp_dm.ipynb
```

Rode as celulas em sequencia. O notebook e autocontido — cada etapa gera as variaveis necessarias para a proxima.

---

## Limitacoes e Proximos Passos

Este modelo nao esta pronto para producao — e um proof of concept robusto. Antes de qualquer implementacao clinica real, seria necessario:

- **Validacao externa:** testar em dados de outros hospitais para avaliar generalizacao
- **Calibracao de probabilidades:** garantir que "70% de risco" signifique realmente 70%
- **Monitoramento de data drift:** distribuicoes hormonais mudam com populacoes e equipamentos
- **SMOTE ou variantes:** explorar oversampling para lidar com o desbalanceamento de forma mais sofisticada
- **Aprovacao regulatoria:** qualquer sistema de auxilio ao diagnostico clinico exige validacao por orgaos competentes

---

## Metodologia

**CRISP-DM** (Cross-Industry Standard Process for Data Mining) — metodologia iterativa padrao para projetos de ciencia de dados, composta por seis fases: Entendimento do Negocio, Entendimento dos Dados, Preparacao dos Dados, Modelagem, Avaliacao e Implantacao.

---

*Projeto de conclusao de curso | Ciencia de Dados*
