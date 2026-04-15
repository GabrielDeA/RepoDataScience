# Dados - Mortalidade Infantil e Taxa de Fertilidade

## Fontes dos Dados

| Dataset | Arquivo | Registros | Cobertura Temporal |
|---|---|---|---|
| Mortalidade Infantil (Under-5) | `child-mortality.csv` | 16.835 | 1751 - 2023 |
| Taxa de Fertilidade Historica | `children-born-per-woman.csv` | 19.402 | 1891 - 2023 |
| Dataset Mesclado (limpo) | `merged_cleaned_dataset.csv` | 22.260 | 1751 - 2023 |

## Dicionario de Dados

### Dataset Mesclado Final

| Variavel | Descricao | Tipo | Unidade | Valores |
|---|---|---|---|---|
| `entity` | Nome do pais, territorio ou regiao agregada | Categorico (texto) | N/A | Ex: "Afghanistan", "Brazil", "World" |
| `code` | Codigo ISO 3166-1 alpha-3 do pais | Categorico (texto) | N/A | Ex: "AFG", "BRA"; NaN para regioes agregadas |
| `year` | Ano da observacao | Numerico (inteiro) | Ano | 1751 a 2023 |
| `child_mortality_rate` | Taxa de mortalidade infantil (menores de 5 anos) | Numerico (float) | Mortes por 100 nascidos vivos | ~0.14 a ~76.74 |
| `fertility_rate_hist` | Taxa de fertilidade total | Numerico (float) | Filhos por mulher | ~0.66 a ~8.86 |
| `continent` | Continente do pais (via ISO-3166) | Categorico (texto) | N/A | Africa, Americas, Asia, Europe, Oceania |
| `sub_region` | Sub-regiao geografica (via ISO-3166) | Categorico (texto) | N/A | Ex: "South America", "Western Europe", "Southern Asia" |

## Composicao das Entidades (262 unicas)

A coluna `entity` contem uma mistura de paises reais e entidades agregadas:

### Paises Reais
- Identificados por possuirem codigo ISO alpha-3 valido na coluna `code`
- Apos o merge com dados ISO-3166, recebem valores em `continent` e `sub_region`

### Entidades Agregadas (sem codigo ISO)
- **Continentes:** Africa, Asia, Europe, North America, South America, Oceania
- **Regioes ONU:** Africa (UN), Asia (UN), Europe (UN), Latin America and the Caribbean (UN), Northern America (UN), Oceania (UN)
- **Grupos por renda:** High-income countries, Low-income countries, Lower-middle-income countries, Upper-middle-income countries
- **Grupos por desenvolvimento:** Least developed countries, Less developed regions, More developed regions, Less developed regions excluding China, Less developed regions excluding least developed countries
- **Outros:** World, European Union (27)
- **Sub-nacionais:** England and Wales, Scotland

## Problemas de Qualidade Identificados

### 1. Valores Ausentes
- `code`: 387 valores NaN (1.74%) - correspondem a entidades agregadas sem codigo ISO
- `child_mortality_rate`: 5.425 valores NaN (24.37%) antes do tratamento - dados de mortalidade comecam apenas em 1957 para a maioria dos paises
- `fertility_rate_hist`: 2.858 valores NaN (12.84%) antes do tratamento - dados de fertilidade comecam em 1950

### 2. Coberturas Temporais Diferentes
- Mortalidade: dados desde 1751 (paises historicos como Suecia) ate 2023
- Fertilidade: dados desde 1891 ate 2023
- O merge outer gera NaN nos periodos sem sobreposicao

### 3. Numero Diferente de Entidades
- Mortalidade: 213 entidades
- Fertilidade: 261 entidades
- Nem todas as entidades aparecem em ambos os datasets

## Tratamento de Dados Aplicado

1. **Merge:** Outer join nos campos `entity`, `code`, `year`
2. **Ordenacao:** Por entidade e ano para garantir interpolacao correta
3. **Interpolacao linear:** Dentro de cada grupo de entidade, preenchendo valores nas duas direcoes
4. **Imputacao por mediana:** Para valores ainda ausentes, usando a mediana da propria entidade; fallback para mediana global
5. **Enriquecimento geografico:** Merge com dados ISO-3166 para adicionar `continent` e `sub_region`
6. **Separacao:** Dataset dividido em `dataset_countries.csv` (apenas paises) e `dataset_aggregated_regions.csv` (regioes agregadas)

## Estatisticas Descritivas (Pos-Limpeza)

| Metrica | year | child_mortality_rate | fertility_rate_hist |
|---|---|---|---|
| count | 22.260 | 22.260 | 22.260 |
| mean | 1975.15 | 10.58 | 3.90 |
| std | 37.52 | 10.09 | 1.92 |
| min | 1751 | 0.14 | 0.66 |
| 25% | 1959 | 2.87 | 2.22 |
| 50% | 1981 | 7.30 | 3.43 |
| 75% | 2002 | 14.98 | 5.72 |
| max | 2023 | 76.74 | 8.86 |

## Arquivos Gerados

| Arquivo | Descricao | Linhas | Colunas |
|---|---|---|---|
| `merged_cleaned_dataset.csv` | Dataset completo limpo com continente/sub-regiao | 22.260 | 7 |
| `dataset_countries.csv` | Apenas paises reais (com continente identificado) | ~20.000+ | 7 |
| `dataset_aggregated_regions.csv` | Apenas entidades agregadas (World, continentes, etc.) | ~2.000+ | 7 |
