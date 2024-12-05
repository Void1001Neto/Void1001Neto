A análise comparativa de algoritmos de ordenação desempenha um papel fundamental na computação. Algoritmos de ordenação são uma parte essencial do repertório de qualquer programador, e a escolha do algoritmo certo pode afetar
drasticamente o desempenho de um programa. Este projeto tem como objetivo explorar e aplicar conceitos de inferência estatística para determinar qual dos algoritmos de ordenação - Bubble Sort, Merge Sort, Quick Sort ou Shell Sort - é o
mais eficiente em termos de tempo de processamento. Além de aprimorar as habilidades de programação em C/JAVA e adquirir experiência em coleta, análise e visualização de dados, utilizando Python, Pandas e NumPy.
OBJETIVO O objetivo principal deste projeto é realizar uma análise estatística rigorosa para determinar qual dos algoritmos de ordenação é o mais eficaz em uma variedade de cenários. Os objetivos específicos incluem:


1. Promover a aplicação de conceitos de usabilidade e design centrado no
usuário na análise e apresentação dos resultados do projeto.

2. Desenvolver interfaces para a visualização dos dados coletados e resultados
da análise de algoritmos.

3. Avaliar a experiência do usuário com o sistema criado para manipulação e
visualização dos dados.

4. Implementar os algoritmos de ordenação (Bubble Sort, Merge Sort, Quick
Sort e Shell sort) em linguagem C/JAVA.

5. Coletar dados de desempenho dos algoritmos para diferentes tamanhos de
amostra e perfis de hardware.

6. Armazenar os dados coletados em um banco de dados para posterior
análise.

7. Utilizar técnicas de estatística descritiva para resumir os dados e identificar
tendências.

8. Realizar testes de hipótese para comparar o desempenho dos algoritmos e
determinar se existem diferenças significativas.

9. Calcular intervalos de confiança para avaliar a precisão das medições.
    
10.Preparar relatórios e apresentações que descrevem os resultados e
conclusões do estudo.










# codigo produzido no google colab


import pandas as pd
import numpy as np
from scipy.stats import t, ttest_ind
import matplotlib.pyplot as plt
import os


# Carregamento dos arquivos
file_paths = {
    "Estruturas": "/content/sample_data/Estruturas.xlsx",

}

#Nas nossas pesquisas , encontramos esta função que verificar se os arquivos existem antes de tentar carregá-los, serve como um performador

missing_files = [file for file in file_paths.values() if not os.path.exists(file)]

if missing_files:
    print(f"Erro: Os seguintes arquivos não foram encontrados: {missing_files}. Certifique-se de que estão no diretório sample_data.")
else:

    dataframes = {}
    for algo_name, file_path in file_paths.items():
        df = pd.read_excel(file_path)
        dataframes[algo_name] = df


    for algo_name, df in dataframes.items():
        print(f"{algo_name} carregado com {df.shape[0]} linhas e {df.shape[1]} colunas.")

        #Aqui fizemos o tratamento de dados , tomamos cuidado com strings e float e removemos as colunas null

def clean_dataframe(df, algorithm_name):

    if "Algoritmo" not in df.columns:
        df["Algoritmo"] = algorithm_name


    for col in df.columns:
        if col != "Algoritmo":
            df[col] = df[col].astype(str).str.replace("[^0-9.]", "", regex=True)
            df[col] = pd.to_numeric(df[col], errors="coerce")


    df = df.dropna(how="all", axis=0)

    return df


cleaned_dataframes = {}
for algo_name, df in dataframes.items():
    cleaned_dataframes[algo_name] = clean_dataframe(df, algo_name)


for algo_name, df in cleaned_dataframes.items():
    print(f"{algo_name} tratado: {df.shape[0]} linhas e {df.shape[1]} colunas após limpeza.")

#Aqui printamos os dados ja Tratados , so pra verificar , pois inicialmente tivemos problemas com os arquivos , facilitou juntar tudo em um arquivo unico
import pandas as pd


# Juntamos todas as base de dados limpos em um único dataframe
unified_df = pd.concat(cleaned_dataframes.values(), ignore_index=True)

print("Colunas disponíveis no DataFrame:")
print(unified_df.columns)

from scipy.stats import t

# Aqui realizamos os testes de hipotese

def hypothesis_testing(df, scenario_column):
    scenario_data = df[["Algoritmo", scenario_column]].dropna()
    algorithms = scenario_data["Algoritmo"].unique()
    results = []
    for i, algo1 in enumerate(algorithms):
        for algo2 in algorithms[i + 1:]:
            data1 = scenario_data[scenario_data["Algoritmo"] == algo1][scenario_column]
            data2 = scenario_data[scenario_data["Algoritmo"] == algo2][scenario_column]
            if len(data1) > 1 and len(data2) > 1:
                t_stat, p_value = ttest_ind(data1, data2, equal_var=False)
                results.append({
                    "Algoritmo 1": algo1,
                    "Algoritmo 2": algo2,
                    "T-statistic": t_stat,
                    "P-value": p_value,
                    "Significativo (p < 0.05)": p_value < 0.05
                })
    return pd.DataFrame(results)

# Ja aqui calculamos o intervalo de confiança

def calculate_confidence_intervals(df, scenario_column):
    confidence_intervals = {}
    for algo in df["Algoritmo"].unique():
        algo_data = df[df["Algoritmo"] == algo][scenario_column].dropna()
        mean = algo_data.mean()
        std = algo_data.std()
        n = len(algo_data)
        if n > 1:
            ci = t.interval(0.95, df=n-1, loc=mean, scale=std/np.sqrt(n))
            confidence_intervals[algo] = {"mean": mean, "CI Lower": ci[0], "CI Upper": ci[1]}
        else:
            confidence_intervals[algo] = {"mean": mean, "CI Lower": mean, "CI Upper": mean}
    return confidence_intervals

# Aqui plotamos os gráficos

def plot_comparisons(data, column, title):
    plt.figure(figsize=(10, 6))
    for algo in data["Algoritmo"].unique():
        algo_data = data[data["Algoritmo"] == algo][column].dropna()
        if not algo_data.empty:
            plt.plot(range(len(algo_data)), algo_data, label=algo)
    plt.title(title)
    plt.xlabel("Execuções")
    plt.ylabel("Tempo (µs)")
    plt.legend()
    plt.grid(True)
    plt.show()

# Encontramos esta funcao que unifica os dataframes em um so ,pois antes estava dando erro
unified_df = pd.concat(cleaned_dataframes.values(), ignore_index=True)

#plot dos graficos

for column in unified_df.columns:
    if column not in ["Algoritmo"]:
        print(f"\nAnalisando cenário: {column}")


        hypothesis_results = hypothesis_testing(unified_df, column)
        print("\nResultados dos Testes de Hipótese:")
        print(hypothesis_results)


        confidence_intervals = calculate_confidence_intervals(unified_df, column)
        print("\nIntervalos de Confiança:")
        for algo, ci in confidence_intervals.items():
            print(f"{algo}: Mean = {ci['mean']}, CI = ({ci['CI Lower']}, {ci['CI Upper']})")


        plot_comparisons(unified_df, column, f"Comparação de Desempenho - {column}")

import pandas as pd
import matplotlib.pyplot as plt
# Aqui tivemos a ideia plotar um grafico especifico para o bubble sort , procuramos alguns ja pre existentes e apenas modificamos eles, mas ele estava incompleto então modificamos 

try:
    if 'sample_data' not in locals():
        file_path = "/content/sample_data/Estruturas.xlsx"
        data = pd.read_excel(file_path)
except FileNotFoundError:
    print("Erro: O arquivo não foi encontrado no caminho especificado.")


if "Algoritmo" not in data.columns:
    raise ValueError("Erro: A coluna 'Algoritmo' não foi encontrada no arquivo. Verifique o arquivo de entrada.")

for column in data.columns:
    if column != "Algoritmo":
        data[column] = pd.to_numeric(data[column], errors='coerce')


descriptive_stats = data.groupby("Algoritmo").agg(["mean", "std", "min", "max", "median"])


print("\nEstatísticas Descritivas por Algoritmo:")
print(descriptive_stats)


metrics = ["mean", "std", "min", "max", "median"]
palette = ['#1f77b4', '#ff7f0e', '#2ca02c', '#d62728', '#9467bd']


for metric in metrics:
    plt.figure(figsize=(10, 6))
    algo_colors = {algo: color for algo, color in zip(data["Algoritmo"].unique(), palette)}
    for column in data.columns:
        if column != "Algoritmo":
            metric_data = data.groupby("Algoritmo")[column].agg(metric)
            metric_data.plot(kind="bar", color=[algo_colors[algo] for algo in metric_data.index], label=column, alpha=0.8)

    plt.title(f"Comparação de {metric.capitalize()} por Algoritmo", fontsize=14, fontweight='bold')
    plt.ylabel(metric.capitalize())
    plt.xlabel("Algoritmo")
    plt.legend(title="Tamanho de Amostra", loc="upper right")
    plt.grid(axis="y", linestyle="--", alpha=0.7)
    plt.tight_layout()
    plt.show()
# Aqui temos a analise estatistica apenas para o Bubble sort , ja que ele teve apenas duas compilaçoes

import pandas as pd
import matplotlib.pyplot as plt


bubble_sort_data = data[data["Algoritmo"] == "Bubble Sort"]

for column in bubble_sort_data.columns:
    if column != "Algoritmo":
        bubble_sort_data[column] = pd.to_numeric(bubble_sort_data[column], errors='coerce')


plt.figure(figsize=(10, 6))

for column in bubble_sort_data.columns:
    if column != "Algoritmo":
        plt.plot(bubble_sort_data.index, bubble_sort_data[column], label=column, marker="o")

plt.title("Avaliação do Número de Execuções pelo Tempo (ms) - Bubble Sort", fontsize=14, fontweight='bold')
plt.xlabel("Número de Execuções")
plt.ylabel("Tempo (ms)")
plt.legend(title="Tamanho da Amostra", loc="upper left")
plt.grid(axis="both", linestyle="--", alpha=0.7)
plt.tight_layout()

plt.show()

# Aqui criamos o wireframe com todos os dados comparativos ,e a analise de desempenho

import matplotlib.pyplot as plt


fig = plt.figure(figsize=(12, 16))


fig.suptitle("Análise de Desempenho dos Algoritmos", fontsize=16, fontweight="bold")


plt.subplot(3, 1, 1)
plt.axis("off")
plt.table(
    cellText=[
        ["Bubble Sort", "Merge Sort", 1.23, 0.05, "Sim"],
        ["Quick Sort", "Shell Sort", 2.11, 0.01, "Sim"],
        ["Bubble Sort", "Quick Sort", 0.98, 0.15, "Não"]
    ],
    colLabels=["Algoritmo 1", "Algoritmo 2", "T-statistic", "P-value", "Significativo (p < 0.05)"],
    loc="center",
    cellLoc="center"
)
plt.title("Resultados dos Testes de Hipótese", fontsize=12, pad=10)


plt.subplot(3, 1, 2)
plt.axis("off")
plt.table(
    cellText=[
        ["Bubble Sort", 123.45, 110.0, 135.0],
        ["Merge Sort", 98.0, 85.0, 110.0],
        ["Quick Sort", 110.23, 100.0, 120.0],
        ["Shell Sort", 105.78, 95.0, 115.0]
    ],
    colLabels=["Algoritmo", "Média", "IC Inferior (95%)", "IC Superior (95%)"],
    loc="center",
    cellLoc="center"
)
plt.title("Intervalos de Confiança", fontsize=12, pad=10)


plt.subplot(3, 1, 3)
x = range(10)
plt.plot(x, [120 + i*2 for i in x], label="Bubble Sort", marker="o")
plt.plot(x, [100 + i*1.5 for i in x], label="Merge Sort", marker="o")
plt.plot(x, [110 - i*1.2 for i in x], label="Quick Sort", marker="o")
plt.plot(x, [105 + i for i in x], label="Shell Sort", marker="o")
plt.title("Gráfico Comparativo de Desempenho", fontsize=12)
plt.xlabel("Execuções")
plt.ylabel("Tempo em milisegundos (µs)")
plt.legend()
plt.grid(True)


plt.tight_layout(rect=[0, 0, 1, 0.95])
plt.show()

 # Analise comparativa do tempo de cada estrutura 

import pandas as pd
import matplotlib.pyplot as plt


try:
    if 'sample_data' not in locals():
        file_path = "/content/sample_data/Estruturas.xlsx"
        data = pd.read_excel(file_path)
except FileNotFoundError:
    print("Erro: O arquivo não foi encontrado no caminho especificado.")
    raise


if "Algoritmo" not in data.columns:
    raise ValueError("Erro: A coluna 'Algoritmo' não foi encontrada no arquivo. Verifique o arquivo de entrada.")


for column in data.columns:
    if column != "Algoritmo":
        data[column] = pd.to_numeric(data[column], errors='coerce')


descriptive_stats = data.groupby("Algoritmo").agg(["mean", "std", "min", "max", "median"])


print("\nEstatísticas Descritivas por Algoritmo:")
print(descriptive_stats)


metrics = ["mean", "std", "min", "max", "median"]
algo_colors = {
    "Bubble Sort": "#1f77b4",
    "Merge Sort": "#ff7f0e",
    "Quick Sort": "#2ca02c",
    "Shell Sort": "#d62728",
    "Heap Sort": "#9467bd"
} 


for metric in metrics:
    plt.figure(figsize=(10, 6))
    for column in data.columns:
        if column != "Algoritmo":
            try:
                metric_data = data.groupby("Algoritmo")[column].agg(metric)
                colors = [algo_colors[algo] for algo in metric_data.index]  
                metric_data.plot(kind="bar", color=colors, alpha=0.8)
            except KeyError:
                print(f"Erro ao tentar acessar a métrica {metric} para a coluna {column}. Verifique os dados.")
                continue

    plt.title(f"Comparação de {metric.capitalize()} por Algoritmo", fontsize=14, fontweight='bold')
    plt.ylabel(metric.capitalize())
    plt.xlabel("Algoritmo")
    plt.legend(title="Tamanho de Amostra", loc="upper right")
    plt.grid(axis="y", linestyle="--", alpha=0.7)
    plt.tight_layout()
    plt.show()   
