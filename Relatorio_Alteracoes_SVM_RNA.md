# Relatório de Alterações — Refatoração do Notebook SVM_RNA

## Escopo
Refatoração aplicada ao notebook `notebooks/SVM_RNA/RedesElásticas.ipynb`, com os seguintes objetivos:

- Manter as etapas 1, 2, 3, 4 e 5 de engenharia de features (incluindo `StandardScaler`).
- Substituir estratégia de balanceamento para **undersampling** no conjunto de treino.
- Separar o pipeline de features por modelo:
  - **SVM**: `XT`, `OSNR`, `slots_usados`.
  - **RNA**: `XT`, `banda`, `comprimento`, `OSNR`, `modulação`, `saltos`, `núcleos`, `slots_usados`.

---

## Alterações Implementadas

### 1) Importações e Balanceamento
- Removida dependência de oversampling (SMOTE).
- Adicionada estratégia de undersampling com `RandomUnderSampler` (imblearn) e fallback manual quando indisponível.

### 2) Carregamento de Dados
- Ajustado carregamento do arquivo `baseMLJurandir.csv` com busca robusta em caminhos candidatos, evitando falha de path quando o notebook é executado dentro de subpastas.

### 3) Engenharia de Features (Etapas 1–5 mantidas)
As etapas originais foram preservadas:
1. Correção de encoding de colunas (`núcleo`, `modulação`).
2. Criação da variável alvo binária `aceito` a partir de `resultado`.
3. Remoção da coluna `resultado`.
4. Criação de `slots_usados` a partir de `primeiro slot` e `ultimo slot`.
5. Tratamento da feature categórica `modulação`.

Também foram mantidas as features derivadas já existentes (`interacao_slots_modulacao`, `interacao_slots_nucleo`).

### 4) Separação de Features por Modelo
- Pipeline passou a trabalhar com duas matrizes de entrada independentes:
  - `X_svm` com 3 atributos mandatórios.
  - `X_rna` com 8 atributos mandatórios.
- Implementada resolução robusta de nomes de colunas (acentos, plural/singular), garantindo mapeamento de `núcleos` para `núcleo` quando necessário.

### 5) Split, Escalonamento e Treinamento
- Split único por índice para manter alinhamento entre modelos.
- `StandardScaler` aplicado separadamente para SVM e RNA (fit apenas no treino).
- Undersampling aplicado apenas no treino de cada pipeline.

### 6) Avaliação e Etapas Posteriores
- Avaliação, comparação, importância de features e ajuste de limiar foram atualizados para usar os conjuntos corretos de cada modelo (`X_test_svm_scaled` e `X_test_rna_scaled`).

---

## Validação de Execução
Execução do notebook validada com sucesso (células de código executadas em sequência após instalação de dependências).

### Métricas observadas (amostra de 5% da base)
- **SVM (Linear)**
  - Acurácia: **0.7412**
  - Precision: **0.9056**
  - Recall: **0.6747**
  - F1-Score: **0.7733**

- **RNA (MLP)**
  - Acurácia: **0.7733**
  - Precision: **0.8683**
  - Recall: **0.7703**
  - F1-Score: **0.8164**

### Undersampling (treino)
- Antes: `{1: 24823, 0: 13124}`
- Depois: `{0: 13124, 1: 13124}`

### Ajuste de limiar por custo (FN=5, FP=1)
- SVM: custo reduzido de **18053** para **4772**.
- RNA: custo reduzido de **13463** para **4089**.

---

## Conclusão
A refatoração foi concluída mantendo o pipeline de engenharia de features (1–5 + `StandardScaler`), aplicando undersampling corretamente e separando os atributos por modelo conforme especificação (SVM e RNA). O notebook executa sem quebra no fluxo e com avaliação consistente para ambos os modelos.
