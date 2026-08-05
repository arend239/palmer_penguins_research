# Palmer Penguins — Inferência Estatística

Trabalho da disciplina de Introdução à Inferência (UFRGS) aplicando estimação por
intervalo de confiança e teste de hipótese sobre o dataset Palmer Penguins.

**Apresentação publicada:** https://nimrodds.quarto.pub/club-penguin-palmer-penguins/#/title-slide

Autores: João Arend, Davi Augusto, Diogo Bolzan e Luan Frederico.

## Contexto

O objetivo é responder duas perguntas de inferência sobre as três espécies de pinguim
do dataset (Adélie, Chinstrap e Gentoo):

1. Qual o intervalo de confiança de 95% para o comprimento médio da nadadeira
   (`flipper_length_mm`) de cada espécie?
2. A proporção de cada espécie na amostra é estatisticamente diferente de 1/3
   (distribuição equitativa entre as três espécies)?

O trabalho compara explicitamente duas formas de construir o intervalo de confiança
(fórmula fechada via TCL vs. bootstrap) e usa teste de proporção para a segunda
pergunta.

## Dados usados

[Palmer Penguins](https://allisonhorst.github.io/palmerpenguins/), coletado por
Kristen Gorman em parceria com a Estação Palmer (Antártica), integrante da rede
Long Term Ecological Research (LTER). Arquivo local: [`data/penguins.csv`](data/penguins.csv).

- 344 observações, 3 espécies (Adélie: 152, Chinstrap: 68, Gentoo: 124), 3 ilhas
  (Biscoe, Dream, Torgersen), anos 2007–2009.
- Variáveis: `species`, `island`, `bill_length_mm`, `bill_depth_mm`,
  `flipper_length_mm`, `body_mass_g`, `sex`, `year`.
- 11 das 344 linhas têm ao menos um valor faltante (2 em cada medida morfológica,
  11 em `sex`); tratadas com `na.rm = TRUE` / `na.omit()` conforme a análise, sem
  imputação.

A amostra é uma amostra de conveniência de campo (três ilhas específicas do
arquipélago Palmer), não uma amostra aleatória de uma população de pinguins
definida — ver [Limitações](#limitações).

## Metodologia estatística

### 1. Análise descritiva
Médias e amplitudes (mín/máx) de bico, nadadeira e massa corporal por espécie e por
sexo; dispersão entre `flipper_length_mm` e `body_mass_g` e entre `bill_length_mm` e
`bill_depth_mm`; boxplot de massa corporal por espécie; mapa (Leaflet) e heatmap de
ocupação por ilha.

### 2. Intervalo de confiança (95%) para a média de `flipper_length_mm`, por espécie
Calculado de duas formas, comparadas lado a lado:

- **Fórmula fechada (TCL):** `x̄ ± z_(0.975) · s/√n`, com `s` = desvio-padrão
  amostral e `z_(0.975) = 1.96`.
- **Bootstrap não-paramétrico:** 10.000 reamostragens com reposição do mesmo
  tamanho da amostra original; o desvio-padrão das médias reamostradas alimenta um
  ajuste normal (`qnorm`) para os limites de 2,5%/97,5% — ou seja, o bootstrap aqui
  estima o erro-padrão empiricamente mas ainda assume normalidade para o intervalo,
  em vez do método de percentil empírico puro.

### 3. Teste de hipótese — proporção de cada espécie
Teste z de uma amostra para proporção, bicaudal, aplicado três vezes (uma por
espécie):

- H0: p = 1/3 (0.33333) — H1: p ≠ 1/3
- α = 0.05, estatística `z = (p̂ − p0) / √(p0(1 − p0)/n)`, decisão por p-valor
  bicaudal `2·Φ(−|z|)`.
- Sem correção para comparações múltiplas (três testes ao mesmo α) — ver
  [Limitações](#limitações).

Todo o código-fonte de cada cálculo está em
[`presentation/palmer_penguins.qmd`](presentation/palmer_penguins.qmd), nas seções
"Estudo Adelie", "Gentoo e Chinstrap" e "Teste de Hipótese".

## Estrutura do repositório

```
.
├── data/
│   └── penguins.csv              # dataset Palmer Penguins
├── presentation/
│   ├── palmer_penguins.qmd       # fonte da apresentação (Quarto + R + revealjs)
│   ├── dd.scss                   # tema/estilo customizado do slide
│   ├── _publish.yml              # metadados de publicação (Quarto Pub)
│   └── images/                   # imagens usadas nos slides
├── palmer-penguins-research.Rproj
└── README.md
```

`presentation/palmer_penguins.qmd` gera a saída revealjs; os artefatos de build
(`*_files/`, HTML renderizado) não são versionados — ver `.gitignore`.

## Como rodar

Pré-requisitos: [R](https://www.r-project.org/) ≥ 4.x, [Quarto CLI](https://quarto.org/docs/get-started/)
e os pacotes R usados no documento.

```r
install.packages(c(
  "readr", "dplyr", "gridExtra", "patchwork", "ggplot2",
  "magick", "tidyr", "kableExtra", "htmlwidgets", "leaflet"
))
```

Renderizar localmente (a partir da raiz do repositório):

```bash
quarto render presentation/palmer_penguins.qmd
```

Isso reexecuta todo o R embutido no `.qmd` — descritivas, bootstrap e testes de
hipótese — e gera a apresentação revealjs em `presentation/`. Para reabrir com
paginação/navegação de slide, abra o `.html` gerado no navegador.

Para republicar no Quarto Pub (requer conta vinculada ao projeto):

```bash
quarto publish quarto-pub presentation/palmer_penguins.qmd
```

## Principais resultados

**Intervalo de confiança (95%) — comprimento médio da nadadeira, mm**

| Espécie   |   n | Média  | IC (fórmula fechada) | IC (bootstrap) |
|-----------|----:|-------:|-----------------------|-----------------|
| Adélie    | 151 | 189.95 | [188.91, 191.00]      | [188.92, 190.99]|
| Chinstrap |  68 | 195.82 | [194.13, 197.52]      | [194.14, 197.53]|
| Gentoo    | 123 | 217.19 | [216.04, 218.33]      | [216.06, 218.34]|

Os dois métodos convergem para praticamente o mesmo intervalo em todas as
espécies — esperado, já que ambos assumem/aproximam normalidade e `n` é grande o
bastante para o TCL valer bem. (`n` nesta tabela exclui 1 valor faltante de
`flipper_length_mm` em Adélie e 1 em Gentoo; por isso difere do `n` usado no
teste de proporção abaixo, que conta todas as linhas da espécie.)

**Teste de hipótese — proporção de cada espécie é igual a 1/3?**

| Espécie   |   n | p̂      | z       | p-valor  | Decisão (α = 0.05)   |
|-----------|----:|--------:|--------:|---------:|-----------------------|
| Adélie    | 152 | 0.4419  |  2.838  | 0.0045   | Rejeita H0            |
| Chinstrap |  68 | 0.1977  | −2.373  | 0.0176   | Rejeita H0            |
| Gentoo    | 124 | 0.3605  |  0.641  | 0.5215   | Não rejeita H0        |

Adélie está sobrerrepresentada e Chinstrap subrrepresentada em relação a uma
divisão equitativa; Gentoo é estatisticamente compatível com 1/3 da amostra.

## Limitações

- **Reúso incorreto do tamanho de amostra no bootstrap.** No código-fonte
  (seção "Gentoo e Chinstrap"), a variável `n` usada no laço de reamostragem é
  herdada do cálculo da Adélie (`n = 152`) e não é redefinida para Chinstrap
  (`n = 68`) nem para Gentoo (`n = 124`) antes de suas respectivas reamostragens.
  Isso não desloca a média bootstrap, mas infla/reduz artificialmente a variância
  da distribuição de médias reamostradas desses dois grupos, distorcendo a
  amplitude do IC bootstrap relatado para eles. Os valores na tabela acima foram
  recalculados de forma independente (com `n` correto por grupo) para este
  README; os números originais no `.qmd` publicado diferem ligeiramente disso.
- **Sem correção para comparações múltiplas.** Os três testes de proporção usam
  α = 0.05 individualmente; a taxa de erro tipo I da família de testes é maior
  que 5% (≈ 1 − 0.95³ ≈ 14%, sob independência). Uma correção (Bonferroni, por
  exemplo) reduziria o risco de falso positivo, especialmente relevante para a
  conclusão sobre Chinstrap (p-valor 0.0176, mais próximo do limiar).
- **Amostra não é uma amostra aleatória de uma população bem definida.** É uma
  amostra de conveniência de três ilhas específicas em três anos específicos;
  tratar "proporção esperada = 1/3" como hipótese nula sobre "a população de
  pinguins" pressupõe um desenho amostral que os dados não têm.
- **Bootstrap com ajuste normal, não percentil empírico.** O IC bootstrap é
  construído ajustando uma normal às médias reamostradas (`qnorm` sobre
  média/desvio do bootstrap) em vez de usar os percentis empíricos da
  distribuição reamostrada. Na prática os dois métodos convergem aqui porque a
  distribuição de `flipper_length_mm` é aproximadamente simétrica por espécie,
  mas isso reduz o valor do bootstrap como validação independente do método
  clássico.
- **Ausência de checagem formal de normalidade** (Shapiro-Wilk, QQ-plot) antes de
  aplicar a fórmula fechada do IC; a suposição é apoiada apenas visualmente pelos
  histogramas.
- **Valores faltantes tratados por exclusão caso a caso** (`na.rm`/`na.omit`), sem
  investigar se a ausência está relacionada a alguma variável observada (MCAR vs.
  MAR).
