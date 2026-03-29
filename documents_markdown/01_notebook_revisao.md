# Notebook 01 — Sales Performance Analysis
**Projeto:** business-analytics-python  
**Dataset:** Sample Superstore (Kaggle — vivek468)  
**Stack:** pandas, matplotlib, seaborn  
**Objetivo:** Analisar performance de vendas por região, categoria e produto, identificando padrões de lucro e impacto de descontos.

---

## Estrutura do Projeto

```
business-analytics-python/
├── README.md
├── requirements.txt
├── data/
│   └── superstore.csv
└── notebooks/
    └── 01_sales_analysis/
        └── sales_analysis.ipynb
```

**requirements.txt:**
```
pandas
matplotlib
seaborn
jupyter
openpyxl
```

---

## Bloco 1 — Carregamento e Inspeção

### Imports e configuração visual
```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

sns.set_theme(style="whitegrid")
plt.rcParams["figure.figsize"] = (12, 5)
```
- `sns.set_theme()` aplica tema visual limpo a todos os gráficos automaticamente
- `rcParams["figure.figsize"]` define tamanho padrão de todas as figuras em polegadas

### Carregar o dataset
```python
df = pd.read_csv("../../data/superstore.csv", encoding="latin-1")
df.head()
```
- `encoding="latin-1"` necessário por caracteres especiais no arquivo
- `df.head()` renderiza automaticamente como tabela no Jupyter (últimaexpressão da célula)
- `df` é um **DataFrame** — estrutura central do pandas, equivalente a uma tabela Excel

### Inspeção inicial
```python
print(f"Shape: {df.shape}")
print(f"\nColunas:\n{df.columns.tolist()}")
print(f"\nTipos de dado:\n{df.dtypes}")
print(f"\nNulos por coluna:\n{df.isnull().sum()}")
```
- `df.shape` → tupla (linhas, colunas) — resultado: **(9994, 21)**
- `df.columns.tolist()` → lista com nomes das colunas
- `df.dtypes` → tipo de cada coluna (`object`, `int64`, `float64`)
- `df.isnull().sum()` → contagem de valores nulos por coluna — **resultado: zero nulos**

### Estatísticas descritivas
```python
df.describe()
```
- Retorna count, mean, std, min, max e quartis para colunas numéricas
- **Observação importante:** `Profit` tem valor mínimo de -6.599 → algumas vendas geram prejuízo

---

## Bloco 2 — Limpeza e Preparação

```python
# Converter datas de texto para datetime
df["Order Date"] = pd.to_datetime(df["Order Date"])
df["Ship Date"] = pd.to_datetime(df["Ship Date"])

# Criar colunas auxiliares para análise temporal
df["Year"] = df["Order Date"].dt.year
df["Month"] = df["Order Date"].dt.month
df["YearMonth"] = df["Order Date"].dt.to_period("M")

# Confirmar
print(df["Order Date"].dtype)
df[["Order Date", "Ship Date", "Year", "Month", "YearMonth"]].head()
```
- `pd.to_datetime()` converte string para `datetime64[ns]` — necessário para operações temporais
- `.dt.year` / `.dt.month` → extrai componentes da data
- `.dt.to_period("M")` → cria período mensal (ex: `2016-11`) para agrupamento temporal

---

## Bloco 3 — Análise de Receita por Região e Categoria

### Receita e margem por região
```python
regiao = df.groupby("Region")[["Sales", "Profit"]].sum().sort_values("Sales", ascending=False)
regiao["Profit Margin %"] = (regiao["Profit"] / regiao["Sales"] * 100).round(2)
regiao
```
- `groupby("Region")` → agrupa por região (equivalente a `GROUP BY` em SQL)
- `[["Sales", "Profit"]]` → seleciona apenas essas colunas (colchetes duplos = lista → DataFrame)
- `.sum()` → agrega somando os valores dentro de cada grupo
- `.sort_values("Sales", ascending=False)` → ordena do maior para o menor
- Nova coluna criada por **operação vetorizada** — opera em toda a coluna de uma vez

**Resultado:**

| Region  | Sales       | Profit     | Profit Margin % |
|---------|-------------|------------|-----------------|
| West    | 725.457     | 108.418    | 14.94%          |
| East    | 678.781     | 91.522     | 13.48%          |
| Central | 501.239     | 39.706     | 7.92%           |
| South   | 391.721     | 46.749     | 11.93%          |

### Receita e margem por categoria
```python
categoria = df.groupby("Category")[["Sales", "Profit"]].sum().sort_values("Sales", ascending=False)
categoria["Profit Margin %"] = (categoria["Profit"] / categoria["Sales"] * 100).round(2)
categoria
```

**Resultado:**

| Category        | Sales   | Profit  | Profit Margin % |
|-----------------|---------|---------|-----------------|
| Technology      | 836.154 | 145.454 | 17.40%          |
| Furniture       | 741.999 | 18.451  | 2.49%           |
| Office Supplies | 719.047 | 122.490 | 17.04%          |

**Insight:** Furniture tem segunda maior receita mas margem crítica de 2.49%.

### Gráfico combinado
```python
fig, axes = plt.subplots(1, 2, figsize=(14, 5))

# Receita por Região
regiao["Sales"].plot(kind="bar", ax=axes[0], color="steelblue")
axes[0].set_title("Receita por Região")
axes[0].set_xlabel("")
axes[0].set_ylabel("Sales (USD)")
axes[0].tick_params(axis="x", rotation=0)

# Margem por Categoria
categoria["Profit Margin %"].plot(kind="bar", ax=axes[1], color="seagreen")
axes[1].set_title("Margem de Lucro por Categoria")
axes[1].set_xlabel("")
axes[1].set_ylabel("Profit Margin %")
axes[1].tick_params(axis="x", rotation=0)

plt.tight_layout()
plt.show()
```
- `plt.subplots(1, 2)` → grade de 1 linha e 2 colunas; retorna `fig` (canvas) e `axes` (lista de eixos)
- `ax=axes[0]` → direciona o gráfico para o eixo específico
- `tick_params(rotation=0)` → rótulos do eixo X na horizontal
- `plt.tight_layout()` → ajusta espaçamento automático entre subplots

---

## Bloco 4 — Tendência Temporal de Vendas

```python
tendencia = df.groupby("YearMonth")[["Sales", "Profit"]].sum()
tendencia.index = tendencia.index.astype(str)  # converte Period para string pro gráfico

tendencia.plot(kind="line", figsize=(14, 5), marker="o", markersize=3)
plt.title("Receita e Lucro Mensais")
plt.xlabel("")
plt.ylabel("USD")
plt.xticks(range(0, len(tendencia), 3), tendencia.index[::3], rotation=45)
plt.tight_layout()
plt.show()
```
- `groupby("YearMonth")` → agrupa por período mensal
- `marker="o"` → adiciona marcador circular em cada ponto
- `plt.xticks(range(..., 3), index[::3])` → exibe rótulos de 3 em 3 meses
- `[::3]` → slice notation do Python: sem início, sem fim, passo 3

**Insights:**
- Sazonalidade clara com picos em novembro/dezembro
- Queda abrupta em janeiro de cada ano
- Tendência de crescimento geral de 2014 a 2017
- Profit cresce menos proporcionalmente que Sales

---

## Bloco 5 — Impacto de Descontos e Performance por Produto

### Scatter desconto vs. lucro + Top sub-categorias
```python
fig, axes = plt.subplots(1, 2, figsize=(14, 5))

# Scatter: Desconto vs Lucro
axes[0].scatter(df["Discount"], df["Profit"], alpha=0.3, color="steelblue", s=10)
axes[0].axhline(y=0, color="red", linestyle="--", linewidth=1)
axes[0].set_title("Desconto vs. Lucro por Transação")
axes[0].set_xlabel("Discount")
axes[0].set_ylabel("Profit")

# Top 10 sub-categorias por lucro
top_subcat = df.groupby("Sub-Category")["Profit"].sum().sort_values(ascending=True).tail(10)
top_subcat.plot(kind="barh", ax=axes[1], color="seagreen")
axes[1].set_title("Top 10 Sub-Categorias por Lucro")
axes[1].set_xlabel("Profit (USD)")

plt.tight_layout()
plt.show()
```
- `alpha=0.3` → transparência dos pontos; sobreposição cria efeito de densidade
- `s=10` → tamanho dos pontos
- `axhline(y=0)` → linha horizontal vermelha na posição zero (linha de break-even)
- `kind="barh"` → barras horizontais; indicado quando rótulos são longos
- `.tail(10)` → últimos 10 após ordenação crescente = top 10

**Insight:** Descontos acima de 40% resultam sistematicamente em lucro negativo.

### Confirmação: desconto médio por categoria
```python
desconto_cat = df.groupby("Category")[["Discount", "Profit"]].mean().round(3)
desconto_cat["Profit Margin %"] = (
    df.groupby("Category")["Profit"].sum() /
    df.groupby("Category")["Sales"].sum() * 100
).round(2)
desconto_cat.columns = ["Avg Discount", "Avg Profit per Line", "Profit Margin %"]
desconto_cat.sort_values("Avg Discount", ascending=False)
```

**Resultado:**

| Category        | Avg Discount | Avg Profit per Line | Profit Margin % |
|-----------------|--------------|---------------------|-----------------|
| Furniture       | 0.174        | 8.699               | 2.49%           |
| Office Supplies | 0.157        | 20.327              | 17.04%          |
| Technology      | 0.132        | 78.752              | 17.40%          |

**Insight confirmado:** Furniture tem o maior desconto médio e a menor margem — relação causal direta.

---

## Conclusões

- **Região West** lidera em receita (USD 725k) e margem (14.9%)
- **Furniture** apresenta margem crítica de 2.49% — o maior desconto médio (17.4%) é o principal fator
- **Descontos acima de 40%** resultam sistematicamente em lucro negativo
- **Sazonalidade clara:** picos em novembro/dezembro, queda abrupta em janeiro
- **Technology** tem o melhor desempenho combinado: margem de 17.4% com ticket médio alto (USD 78 de lucro por linha)

---

## Referência Rápida de Sintaxe

| Operação | Código |
|---|---|
| Carregar CSV | `pd.read_csv("arquivo.csv", encoding="latin-1")` |
| Inspecionar shape | `df.shape` |
| Verificar tipos | `df.dtypes` |
| Verificar nulos | `df.isnull().sum()` |
| Converter data | `pd.to_datetime(df["coluna"])` |
| Extrair ano/mês | `df["coluna"].dt.year` |
| Agrupar e somar | `df.groupby("col")[["a","b"]].sum()` |
| Ordenar | `.sort_values("col", ascending=False)` |
| Criar coluna | `df["nova"] = df["a"] / df["b"] * 100` |
| Gráfico de barras | `.plot(kind="bar")` |
| Gráfico de linha | `.plot(kind="line")` |
| Gráfico horizontal | `.plot(kind="barh")` |
| Scatter | `axes[n].scatter(x, y, alpha=0.3)` |
| Múltiplos subplots | `fig, axes = plt.subplots(1, 2, figsize=(14,5))` |

---

## Próximos Passos (Notebook 02)

- Analisar quais sub-categorias de Furniture concentram os maiores descontos
- Investigar performance por segmento de cliente (Consumer, Corporate, Home Office)
- Avaliar se a sazonalidade varia por região
