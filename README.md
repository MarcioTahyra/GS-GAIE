# Global Solution 2026 — GAIE

### Integrantes
* Jéssica Witzler Costacurta - RM99068
* Márcio Hitoshi Tahyra - RM552511

---

## Sistema de Monitoramento Orbital e Antecipação de Queimadas via API

### 1. Definição do Problema e Qualidade dos Dados

* **O Contexto na Indústria Espacial:** No escopo do *Earth Observation* (Observação da Terra) e da Economia Espacial, constelações de satélites meteorológicos monitoram a atmosfera e a superfície terrestre continuamente. O desafio central é transformar dados brutos de telemetria em inteligência preditiva para mitigar desastres climáticos.
* **O Problema:** Prever o risco de focos de incêndio florestal de forma precoce nos quatro principais biomas brasileiros (**Amazônia, Cerrado, Caatinga e Pantanal**), permitindo ações preventivas antes da ignição física do fogo.
* **Qualidade e Origem dos Dados:** Os dados foram extraídos 100% via consumo direto da **API Pública Open-Meteo (Historical Weather Archive)**, que consolida medições reanalisadas por sensores e satélites meteorológicos. A coleta abrangeu o ano completo de 2023, consolidando um dataset de **1.460 linhas** e **10 colunas** estruturais com dados climáticos reais das coordenadas espaciais de cada bioma.

---

### 2. Pré-processamento e Engenharia de Atributos

* **Limpeza e Tratamento:** Foi realizada a remoção de valores nulos (`dropna`) para limpar anomalias de transmissão de dados. Para o modelo linear, foi aplicad o escalonamento de features via `StandardScaler`.
* **Codificação Categórica:** O atributo geográfico `Bioma` foi transformado em variáveis numéricas binárias utilizando a técnica de *One-Hot Encoding* (`get_dummies`), permitindo o processamento correto pelos algoritmos.
* **Engenharia de Atributos (Atributos Criados):**
1. `Dias_Sem_Chuva`: Métrica calculada dinamicamente mapeando sequências acumuladas de dias consecutivos onde a precipitação sumária foi igual a zero.
2. `Risco_Incendio` (Variável Alvo / *Target*): Modelada através do conceito científico de **Proxy Ecológico de Estresse Hídrico**. A regra combinou o comportamento da Temperatura Máxima com a **Evapotranspiração de Referência FAO** (índice orbital que mede a perda de água do solo e a transpiração da vegetação). Dias que atingiram 70% (os 30% cenários mais severos e secos do ano) foram rotulados como classe crítica (`1`), e o restante como seguro (`0`).


* **Colunas finais enviadas aos modelos:** `Temp_Max`, `Temp_Min`, `Temp_Media`, `Velocidade_Vento`, `Direcao_Vento`, `Radiacao_Solar`, `Evapotranspiracao`, `Dias_Sem_Chuva`, `Bioma_Caatinga`, `Bioma_Cerrado`, `Bioma_Pantanal`.

---

### 3. Aplicação e Comparação de Modelos

O projeto avaliou e comparou duas técnicas distintas de classificação:

1. **Regressão Logística:** Abordagem linear clássica que calcula a probabilidade logística de ocorrência do risco mapeando os coeficientes de peso de cada variável normalizada.
2. **Random Forest Classifier (Floresta Aleatória):** Algoritmo de aprendizado baseado em um *Ensemble* de 200 árvores de decisão trabalhando em paralelo, capturando relações não-lineares e de alta complexidade entre as variáveis climáticas.

---

### 4. Validação e Análise de Métricas

Ambos os modelos demonstraram alto poder preditivo na separação das classes de risco:

* **Regressão Logística:** Alcançou **99% de Acurácia** geral, provando-se altamente estável para o padrão matemático imposto.
* **Random Forest:** Alcançou **98% de Acurácia**, destacando-se pela precisão na classe positiva.
* **Matriz de Confusão:** Demonstrou que o classificador Random Forest obteve uma taxa de **falsos positivos igual a zero** (evitando alarmes falsos para as agências de controle), errando minimamente em apenas alguns falsos negativos.
* **Curva ROC e AUC:** O modelo obteve uma **AUC (Area Under Curve) de 0.998**, indicando uma capacidade quase perfeita de discriminação entre dias com atmosfera segura e dias de perigo iminente.

---

### 5. Interpretabilidade com SHAP

Para afastar o conceito de "caixa-preta" e prover explicabilidade ao modelo (XAI), utilizou-se o framework **SHAP** através do `TreeExplainer`.

* **Análise do Gráfico (Summary Plot):** O SHAP revelou cientificamente quais variáveis de órbita foram determinantes para o modelo. A **Evapotranspiração** e a **Temperatura Máxima** lideraram o índice de importância global. Valores elevados de calor e evaporação acelerada empurram os valores Shapley positivamente (pontos vermelhos à direita), provando que o modelo correlacionou corretamente o esvaziamento hídrico da vegetação com o estresse que precede as queimadas.

---

### 6. Deploy da Aplicação

O modelo preditivo foi encapsulado e disponibilizado em produção por meio da biblioteca **Gradio**, gerando uma interface gráfica interativa baseada na web.

* **Aplicações Práticas:** A interface permite que qualquer operador, cientista ou engenheiro configure os *sliders* climáticos em tempo real (inserindo Temperatura, Radiação Solar, Direção do Vento e Evapotranspiração) e selecione o Bioma Alvo para obter imediatamente o diagnóstico probabilístico do modelo de Machine Learning, abstraindo toda a complexidade do pipeline em Python.
* **Link para a Aplicação em Funcionamento:** https://4c69f5c6280eac32d4.gradio.live/
