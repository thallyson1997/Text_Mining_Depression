# 📌 Repositório: Mineração de Texto para Predição de Depressão

Este repositório contém os códigos, dados preparados e resultados de experimentos de **mineração de texto** aplicados ao problema de **predição de depressão em postagens do Reddit**.

## 📂 Estrutura do Repositório

* `Dados/` (**ignorado no Git**): contém os dados originais e sensíveis, não disponibilizados publicamente.
* `Dados_Preparados/`: contém os dados tratados e prontos para uso nos modelos.
* `Resultado/`: contém as métricas, predições e gráficos gerados durante os experimentos.
* Notebooks R (`.Rmd`): código de preparação, modelagem e análise.

## 🔹 Etapa 1 - Preparação dos Dados (`Análise de Texto.Rmd`)

O notebook **Análise de Texto.Rmd** realiza o **pré-processamento textual** das postagens de usuários divididas em 10 segmentos temporais (treino e teste).

### 📥 Carregamento dos Dados

* Os dados originais (`treino_1.csv` … `treino_10.csv`, `teste_1.csv` … `teste_10.csv`) são lidos da pasta `Dados/`.
* Cada conjunto é armazenado em listas (`TREINO` e `TESTE`).

### 🧹 Pré-processamento aplicado

O texto passa por várias etapas de limpeza:

1. **Remoção de marcações XML**: usando a função `remover_regex`.
2. **Remoção de ruídos**: números, pontuações e URLs (`remover_ruido`).
3. **Normalização**: conversão para minúsculas.
4. **Stopwords**: remoção de palavras comuns da língua inglesa (`remover_stopword`).
5. **Stemming**: redução de palavras ao seu radical (`stemDocument`).

Todas essas funções são aplicadas dentro da função principal `preprocessamento()`.

### 💾 Salvamento dos Dados Preparados

* Após o pré-processamento, os conjuntos são salvos em **`Dados_Preparados/`** nos arquivos:

  * `treino_1_preparado.csv` … `treino_10_preparado.csv`
  * `teste_1_preparado.csv` … `teste_10_preparado.csv`
* Essa etapa garante que os dados tratados fiquem disponíveis publicamente para reprodutibilidade, sem expor informações sensíveis.

## 🔹 Etapa 2 - Método 1: Sem Alteração (`Metodo_01.Rmd`)

O **Método 1** aplica diretamente o processo de modelagem aos dados preparados, sem nenhuma modificação adicional além da vetorização e treinamento.

### ⚙️ Pipeline do Método 1

1. **Carregamento dos dados preparados**

   * Os arquivos `treino_X_preparado.csv` e `teste_X_preparado.csv` são carregados da pasta `Dados_Preparados/`.
   * Os conjuntos são armazenados em listas (`TREINO` e `TESTE`) para os 10 segmentos temporais.

2. **Vetorização com TF-IDF**

   * Cada fold de treino e teste é transformado em matrizes TF-IDF.
   * São selecionadas as **250 palavras mais relevantes** (com maior soma de pesos TF-IDF).
   * Apenas colunas em comum entre treino e teste são mantidas.

3. **Treinamento e Avaliação**

   * Três modelos foram testados com diferentes hiperparâmetros:

     * **Random Forest (RF)** → `mtry = {2, 126, 250}`
     * **XGBoost (XGB)** → `nrounds = {250, 300, 350}`
     * **Regressão Logística Regularizada (GLMNET)** → `lambda = {0.1, 0.5, 1}`
   * O treinamento é feito via `caret::train()` com **validação cruzada interna**.

4. **Métricas de Avaliação**
   Para cada modelo, parâmetro e fold, são calculadas:

   * **Acurácia**
   * **Recall (Sensibilidade)**
   * **Especificidade**
   * **F1-score**
   * **AUC-ROC**

5. **Saídas Geradas**

   * **Predições por fold**: arquivos `_predicoes.csv` com colunas:

     * `Usuario`, `Classe` (real), `Predito_1 … Predito_10` (resultado por fold).
   * **Métricas individuais**: arquivos `_metricas_long.csv` contendo os valores de métricas por fold.
   * **Métricas médias**: arquivo `metricas_medias.csv` com a média de cada métrica para cada modelo/parâmetro.
   * **Matrizes de confusão** (por usuário, com predição agregada por maioria de folds):

     * Arquivos `.csv` e imagens `.png` para auditoria.
   * **Boxplots comparativos**: cinco gráficos (um para cada métrica) comparando os modelos e parâmetros.

### 📊 Saída da pasta `Resultado/METODO_1`

* `*_predicoes.csv` → predições por usuário/fold.
* `*_metricas_long.csv` → métricas detalhadas por fold.
* `metricas_medias.csv` → resumo comparativo de desempenho.
* `*_confusion_matrix.csv` e `*_confusion_matrix.png` → matrizes de confusão agregadas.
* `Boxplot_Acuracia.png`, `Boxplot_Recall.png`, … → gráficos comparativos.

## 🔹 Etapa 3 - Método 2: Redução da Classe Majoritária (`Metodo_02.Rmd`)

O **Método 2** introduz uma técnica de balanceamento antes da vetorização: a **redução da classe majoritária**.
Assim, o número de instâncias da classe negativa (`0`) é reduzido até igualar o da classe positiva (`1`).

### ⚙️ Pipeline do Método 2

1. **Carregamento dos dados preparados**

   * Utiliza os arquivos da pasta `Dados_Preparados/`, carregando-os em listas (`TREINO`, `TESTE`).

2. **Balanceamento das classes**

   * Para cada fold de treino:

     * Calcula o tamanho dos textos.
     * Ordena os exemplos por tamanho decrescente.
     * Mantém **todos os exemplos da classe `1`**.
     * Seleciona a mesma quantidade de exemplos da classe `0` (reduzindo o excesso).
   * O dataset resultante é balanceado (`n0 = n1`).

3. **Vetorização com TF-IDF**

   * Como no Método 1, os textos são transformados em representações TF-IDF.
   * Seleção das **250 palavras mais relevantes**.
   * Apenas colunas em comum entre treino e teste são mantidas.

4. **Treinamento e Avaliação**

   * Modelos utilizados e parâmetros:

     * **Random Forest (RF)** → `mtry = {2, 126, 250}`
     * **XGBoost (XGB)** → `nrounds = {250, 300, 350}`
     * **Regressão Logística Regularizada (GLMNET)** → `lambda = {0.1, 0.5, 1}`
   * Avaliação com validação cruzada e métricas padrão.

5. **Métricas calculadas**

   * **Acurácia**
   * **Recall (Sensibilidade)**
   * **Especificidade**
   * **F1-score**
   * **AUC-ROC**

6. **Saídas geradas**

   * **Predições por usuário/fold**: arquivos `_predicoes.csv`.
   * **Métricas individuais**: `_metricas_long.csv`.
   * **Matrizes de confusão**: agregadas por usuário (maioria de folds).

     * Arquivos `.csv` e imagens `.png`.
   * **Métricas médias**: `metricas_medias.csv`.
   * **Boxplots comparativos**: gráficos para cada métrica, comparando modelos e parâmetros.

### 📊 Saída da pasta `Resultado/METODO_2`

* `*_predicoes.csv` → predições por usuário/fold.
* `*_metricas_long.csv` → métricas detalhadas por fold.
* `metricas_medias.csv` → resumo comparativo de desempenho.
* `*_confusion_matrix.csv` e `*_confusion_matrix.png` → matrizes de confusão agregadas.
* `Boxplot_Acuracia.png`, `Boxplot_Recall.png`, … → gráficos comparativos.

## 🔹 Etapa 4 - Método 3: Redução Parcial da Classe Majoritária (`Metodo_03.Rmd`)

O **Método 3** adota uma estratégia de **redução parcial da classe majoritária**, em que a quantidade de exemplos negativos (`0`) é **limitada a aproximadamente 2,5 vezes** a quantidade de exemplos positivos (`1`).
Dessa forma, busca-se **diminuir o desbalanceamento** sem eliminar totalmente a predominância da classe negativa.

### ⚙️ Pipeline do Método 3

1. **Carregamento dos dados preparados**

   * Carregamento dos arquivos da pasta `Dados_Preparados/` em listas (`TREINO`, `TESTE`).

2. **Balanceamento parcial das classes**

   * Para cada fold de treino:

     * Ordena os textos pelo tamanho (número de caracteres).
     * Mantém **todos os exemplos da classe `1`**.
     * Seleciona uma quantidade de exemplos da classe `0` **igual a 2,5 vezes o número da classe `1`**.
   * O dataset resultante reduz parcialmente a classe majoritária (`2.5 * n1 ≈ n0`).

3. **Vetorização com TF-IDF**

   * Transformação das postagens em representações TF-IDF.
   * Seleção das **250 palavras mais relevantes**.
   * Apenas colunas em comum entre treino e teste são utilizadas.

4. **Treinamento e Avaliação**

   * Modelos e parâmetros avaliados:

     * **Random Forest (RF)** → `mtry = {2, 126, 250}`
     * **XGBoost (XGB)** → `nrounds = {250, 300, 350}`
     * **Regressão Logística Regularizada (GLMNET)** → `lambda = {0.1, 0.5, 1}`
   * Validação cruzada com métricas detalhadas.

5. **Métricas calculadas**

   * **Acurácia**
   * **Recall (Sensibilidade)**
   * **Especificidade**
   * **F1-score**
   * **AUC-ROC**

6. **Saídas geradas**

   * **Predições por usuário/fold**: arquivos `_predicoes.csv`.
   * **Métricas individuais**: `_metricas_long.csv`.
   * **Matrizes de confusão**: agregadas por usuário (maioria dos folds).

     * Arquivos `.csv` e imagens `.png`.
   * **Métricas médias**: `metricas_medias.csv`.
   * **Boxplots comparativos**: gráficos por métrica, comparando modelos e parâmetros.

### 📊 Saída da pasta `Resultado/METODO_3`

* `*_predicoes.csv` → predições por usuário/fold.
* `*_metricas_long.csv` → métricas detalhadas por fold.
* `metricas_medias.csv` → resumo comparativo de desempenho.
* `*_confusion_matrix.csv` e `*_confusion_matrix.png` → matrizes de confusão agregadas.
* `Boxplot_Acuracia.png`, `Boxplot_Recall.png`, … → comparações gráficas entre modelos.

## 🔹 Etapa 5 - Método 4: Oversampling com SMOTE (`Metodo_04.Rmd`)

O **Método 4** utiliza o algoritmo **SMOTE (Synthetic Minority Over-sampling Technique)** para **aumentar a classe minoritária (positiva, `1`)**, gerando **novos exemplos sintéticos** a partir da vizinhança existente.

### ⚙️ Pipeline do Método 4

1. **Carregamento dos dados preparados**

   * Carregamento dos arquivos da pasta `Dados_Preparados/` em listas (`TREINO`, `TESTE`).

2. **Vetorização com TF-IDF**

   * Transformação das postagens em representações TF-IDF.
   * Seleção das **250 palavras mais relevantes**.
   * Apenas colunas em comum entre treino e teste são utilizadas.

3. **Aplicação do SMOTE**

   * Após a vetorização, o algoritmo **SMOTE** é aplicado nos conjuntos de treino.
   * Novas amostras da classe positiva são criadas artificialmente.
   * O objetivo é **reduzir o desbalanceamento** entre as classes.

4. **Treinamento e Avaliação**

   * Modelos e parâmetros avaliados:

     * **Random Forest (RF)** → `mtry = {2, 126, 250}`
     * **XGBoost (XGB)** → `nrounds = {250, 300, 350}`
     * **Regressão Logística Regularizada (GLMNET)** → `lambda = {0.1, 0.5, 1}`
   * Validação cruzada com métricas detalhadas.

5. **Métricas calculadas**

   * **Acurácia**
   * **Recall (Sensibilidade)**
   * **Especificidade**
   * **F1-score**
   * **AUC-ROC**

6. **Saídas geradas**

   * **Predições por usuário/fold**: arquivos `_predicoes.csv`.
   * **Métricas individuais**: `_metricas_long.csv`.
   * **Matrizes de confusão**: agregadas por usuário (maioria dos folds).

     * Arquivos `.csv` e imagens `.png`.
   * **Métricas médias**: `metricas_medias.csv`.
   * **Boxplots comparativos**: gráficos por métrica, comparando modelos e parâmetros.

### 📊 Saída da pasta `Resultado/METODO_4`

* `*_predicoes.csv` → predições por usuário/fold.
* `*_metricas_long.csv` → métricas detalhadas por fold.
* `metricas_medias.csv` → resumo comparativo de desempenho.
* `*_confusion_matrix.csv` e `*_confusion_matrix.png` → matrizes de confusão agregadas.
* `Boxplot_Acuracia.png`, `Boxplot_Recall.png`, … → comparações gráficas entre modelos.

## 🔹 Etapa 6 - Método 5: Combinação de Undersampling + SMOTE (`Metodo_05.Rmd`)

O **Método 5** combina duas estratégias de balanceamento de classes:

1. **Redução parcial da classe majoritária (Undersampling)** → a quantidade de exemplos negativos (`0`) é reduzida, de forma que **n0 ≈ 2,5 × n1**.
2. **Aumento da classe minoritária com SMOTE (Oversampling)** → após a redução, o **SMOTE** é aplicado para gerar exemplos sintéticos da classe positiva (`1`).

Essa combinação busca reduzir o viés causado pelo desbalanceamento e ao mesmo tempo preservar diversidade nos exemplos de treino.

### ⚙️ Pipeline do Método 5

1. **Carregamento dos dados preparados**

   * Leitura dos arquivos de treino e teste da pasta `Dados_Preparados/`.

2. **Recombinação inicial (Undersampling)**

   * Redução da classe negativa (`0`), preservando aproximadamente o dobro de exemplos em relação à classe positiva.

3. **Vetorização com TF-IDF**

   * Seleção das **250 palavras mais relevantes**.
   * Apenas termos em comum entre treino e teste são mantidos.

4. **Aplicação do SMOTE**

   * Expansão da classe positiva (`1`) por meio da técnica de vizinhos sintéticos (K=81).

5. **Treinamento e Avaliação**

   * Modelos e parâmetros testados:

     * **Random Forest (RF)** → `mtry = {2, 126, 250}`
     * **XGBoost (XGB)** → `nrounds = {250, 300, 350}`
     * **GLMNET (Regressão Logística Regularizada)** → `lambda = {0.1, 0.5, 1}`
   * Avaliação com validação cruzada.

6. **Métricas calculadas**

   * **Acurácia**
   * **Recall (Sensibilidade)**
   * **Especificidade**
   * **F1-score**
   * **AUC-ROC**

7. **Saídas geradas**

   * Predições por usuário/fold: `*_predicoes.csv`.
   * Métricas individuais por fold: `*_metricas_long.csv`.
   * Matrizes de confusão agregadas por usuário (maioria dos folds), em `.csv` e `.png`.
   * Resumo de métricas médias: `metricas_medias.csv`.
   * Boxplots comparativos de modelos/parâmetros: `Boxplot_*.png`.

### 📊 Saída da pasta `Resultado/METODO_5`

* `*_predicoes.csv` → predições por usuário/fold.
* `*_metricas_long.csv` → métricas detalhadas por fold.
* `metricas_medias.csv` → resumo comparativo de desempenho.
* `*_confusion_matrix.csv` e `*_confusion_matrix.png` → matrizes de confusão agregadas.
* `Boxplot_Acuracia.png`, `Boxplot_Recall.png`, … → comparações gráficas entre modelos.
