# 📌 Repositório: Mineração de Texto para Predição de Depressão

Este repositório contém os códigos, dados preparados e resultados de experimentos de **mineração de texto** aplicados ao problema de **predição de depressão em postagens do Reddit**.

## 📂 Estrutura do Repositório

* `Dados/` (**ignorado no Git**): contém os dados originais e sensíveis, não disponibilizados publicamente.
* `Dados_Preparados/`: dados tratados e prontos para uso nos modelos.
* `Resultado/`: métricas, predições e gráficos gerados nos experimentos.
* Notebooks R (`.Rmd`): códigos de preparação, modelagem e análise.

---

## 🔹 Etapa 1 - Preparação dos Dados (`Análise de Texto.Rmd`)

O notebook aplica **pré-processamento textual** às postagens divididas em 10 segmentos temporais (treino e teste).

* **Carregamento**: leitura de `treino_1.csv … treino_10.csv` e `teste_1.csv … teste_10.csv`.
* **Pré-processamento**:

  * Remoção de marcações XML (`remover_regex`).
  * Limpeza de ruídos (números, pontuações, URLs).
  * Normalização (minúsculas).
  * Remoção de *stopwords*.
  * *Stemming* (radicalização).
* **Saída**: arquivos `treino_X_preparado.csv` e `teste_X_preparado.csv` em `Dados_Preparados/`.

---

## 🔹 Etapas 2 a 6 - Métodos de Balanceamento e Modelagem

A partir dos dados preparados, cinco métodos de modelagem foram avaliados.
Todos utilizam **TF-IDF** (250 termos mais relevantes) e os mesmos modelos/hyperparâmetros:

* **Random Forest (RF)** → `mtry = {2, 126, 250}`
* **XGBoost (XGB)** → `nrounds = {250, 300, 350}`
* **GLMNET (Regressão Logística Regularizada)** → `lambda = {0.1, 0.5, 1}`

### ⚙️ Pipeline comum

1. **Carregamento** dos dados de `Dados_Preparados/`.
2. **Balanceamento** (dependendo do método, ver abaixo).
3. **Vetorização TF-IDF** → seleção dos 250 termos mais relevantes.
4. **Treinamento e validação cruzada** via `caret`.
5. **Avaliação com métricas**: Acurácia, Recall, Especificidade, F1-score, AUC-ROC.
6. **Saídas**:

   * Predições (`*_predicoes.csv`)
   * Métricas individuais (`*_metricas_long.csv`)
   * Métricas médias (`metricas_medias.csv`)
   * Matrizes de confusão agregadas (`*_confusion_matrix.csv` + `.png`)
   * Boxplots comparativos (`Boxplot_*.png`)

### 📊 Métodos avaliados

* **Método 1**: Sem alteração (baseline).
* **Método 2**: Balanceamento total → redução da classe majoritária (`n0 = n1`).
* **Método 3**: Balanceamento parcial → `n0 ≈ 2,5 × n1`.
* **Método 4**: Oversampling → aumento da classe minoritária via **SMOTE**.
* **Método 5**: Combinação → redução parcial (`n0 ≈ 2,5 × n1`) + oversampling via SMOTE.

Cada método gera sua própria pasta em `Resultado/METODO_X/` com as saídas descritas acima.
