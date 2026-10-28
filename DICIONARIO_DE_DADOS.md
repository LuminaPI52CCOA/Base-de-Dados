# DICIONÁRIO DE DADOS — ECOSSISTEMA SISAGUA & LUMINA MANAGEMENT

**Projeto:** Lumina Management — Sistema de Gestão Odontológica Humanizada e Preventiva  
**Módulo:** Motor de Inteligência em Saúde Preditiva & Régua de Relacionamento  
**Domínio:** Monitoramento Geoespacial de Fluoretação da Água e Automação de Campanhas  
**Data:** Outubro de 2026  
**Versão:** 2.0 (Camadas Bronze e Silver)

---

## 📑 Índice
1. [Camada Bronze: Base Bruta SISAGUA (Controle Mensal Parâmetros Básicos)](#1-camada-bronze-base-bruta-sisagua-controle-mensal-parâmetros-básicos)
2. [Camada Silver: Dados Filtrados e Otimizados (AWS S3 / Parquet)](#2-camada-silver-dados-filtrados-e-otimizados-aws-s3--parquet)
3. [Regras de Sanitização e Qualidade de Dados (Data Quality)](#3-regras-de-sanitização-e-qualidade-de-dados-data-quality)

---

## 1. Camada Bronze: Base Bruta SISAGUA (Controle Mensal Parâmetros Básicos)

* **Nome do Arquivo Oficial:** `controle_mensal_parametros_basicos_2026.csv`
* **Fonte:** DATASUS / Ministério da Saúde (VIGIAGUA)
* **Formato Original:** CSV, separador ponto e vírgula (`;`), encoding `latin-1`
* **Total de Colunas:** 22 (Índices de 0 a 21)

| Índice | Nome da Coluna no CSV | Tipo Original | Exemplo Real | Nulabilidade | Descrição Técnica & Regra | Aplicação no Sistema Lumina |
| :---: | :--- | :---: | :--- | :---: | :--- | :--- |
| **0** | `Região Geográfica` | String | `SUDESTE` | Não Nulo | Macrorregião geográfica brasileira (NORTE, NORDESTE, SUDESTE, SUL, CENTRO-OESTE). | Filtro de alto nível territorial. |
| **1** | `UF` | String (2) | `SP` | Não Nulo | Sigla da Unidade da Federação. | Filtro de segmentação estadual. |
| **2** | `Regional de Saúde` | String | `GVS VII - SANTO ANDRÉ` | Permite Nulo | Grupo de Vigilância Sanitária ou Gerência Regional do Estado. | Metadado de governança sanitária. |
| **3** | `Código IBGE` | Integer | `354780` | Não Nulo | Código oficial IBGE do município (6 dígitos). Santo André = `354780`. | **Chave primária geográfica de integridade.** |
| **4** | `Município` | String | `SANTO ANDRE` | Não Nulo | Nome do município em caixa alta e sem acentuação consistente. | Filtro textual de conferência no pipeline ETL. |
| **5** | `Tipo da Instituição` | String | `Regional`, `Municipal` | Permite Nulo | Classificação de jurisdição da entidade responsável pelo abastecimento. | Metadado institucional. |
| **6** | `Sigla da Instituição` | String | `SABESP` | Permite Nulo | Sigla da concessionária pública ou privada operadora. | Auditoria de operadora de água. |
| **7** | `Nome da Instituição` | String | `COMPANHIA DE SANEAMENTO BASICO...` | Permite Nulo | Razão Social completa da concessionária. | Rastreabilidade do prestador. |
| **8** | `CNPJ da Instituição` | String/Float | `43776517000180.0` | Permite Nulo | CNPJ matriz da entidade operadora (pode conter notação float `.0`). | Metadado corporativo. |
| **9** | `Nome do escritório regional/local`| String | `SAO PAULO SABESP` | Permite Nulo | Polo de atendimento regional da companhia. | Metadado operacional. |
| **10** | `CNPJ do escritório regional/local`| String/Float | `43776517064749.0` | Permite Nulo | CNPJ do polo regional/filial. | Auditoria contábil. |
| **11** | `Tipo da Forma de Abastecimento` | String | `SAA`, `SAC` | Não Nulo | Classificação do tipo de suprimento hídrico (`SAA`, `SAC`, `SAI`). | **Filtro Crítico:** Isolar `SAA` (rede encanada urbana residencial). |
| **12** | `Código Forma de abastecimento` | String | `S354780000007` | Não Nulo | Identificador único alfanumérico gerado pelo SISAGUA para o sistema. | **Chave Estrangeira lógica com bases de cadastro do SUS.** |
| **13** | `Nome da Forma de Abastecimento`| String | `OLHO DAGUA`, `GUSTAVO` | Permite Nulo | Denominação operacional do sistema, poço ou setor de distribuição. | Segmentação de zona de abastecimento. |
| **14** | `Nome da ETA / UTA` | String | `OLHO DAGUA`, `UTA GUSTAVO` | Permite Nulo | Nome da Estação de Tratamento de Água ou Unidade de Tratamento. | **Chave de Ligação essencial para o De-Para de Bairros.** |
| **15** | `Tipo de Filtração` | String | `FILTRAÇÃO RÁPIDA` | Permite Nulo | Tecnologia empregada na clarificação e desinfecção física da água. | Indicador de confiabilidade do processo. |
| **16** | `Ano de referência` | Integer | `2026` | Não Nulo | Ano civil a que se referem os laudos de potabilidade. | Particionamento temporal primário. |
| **17** | `Mês de referência` | Integer | `1` a `12` | Não Nulo | Mês da coleta e medição laboratorial. | **Série temporal e detecção de oscilação sazonal.** |
| **18** | `Ponto de Monitoramento` | String | `SAÍDA DO TRATAMENTO`, `REDE DE DISTRIBUIÇÃO` | Não Nulo | Local exato onde as amostras foram coletadas na infraestrutura hídrica. | **Filtro de Relevância:** Distingue dosagem na ETA vs. torneira do paciente. |
| **19** | `Parâmetro` | String | `Fluoreto (mg/L)` | Não Nulo | Indicador físico, químico ou biológico mensurado na água. | **Filtro Primário:** Filtrar por `Fluoreto (mg/L)` para o Lumina. |
| **20** | `Campo` | String | `Percentil 95`, `Número de amostras analisadas` | Não Nulo | A grandeza estatística ou descritor analítico registrado na linha. | **Filtro de Métrica:** Extrair `Percentil 95` para dosagem em mg/L. |
| **21** | `Valor` | String | `0,8`, `128`, `MENOR_LD` | Permite Nulo | Resultado da medição, expresso em formato textual com vírgula decimal. | **Variável alvo do algoritmo de predição.** |

---

## 2. Camada Silver: Dados Filtrados e Otimizados (AWS S3 / Parquet)

* **Localização S3:** `s3://lumina-silver/sisagua/tratado/controle_mensal/`
* **Formato:** Apache Parquet (colunar, compressão Snappy)
* **Particionamento:** `ano_referencia` (int), `mes_referencia` (int)
* **Objetivo:** Consultas analíticas rápidas via AWS Athena e alimentação do Backend.

| Nome da Coluna Parquet | Tipo de Dado Athena/Glue | Nulabilidade | Descrição |
| :--- | :---: | :---: | :--- |
| `codigo_ibge` | `INT` | NOT NULL | Código IBGE do município (ex: `354780`). |
| `municipio` | `STRING` | NOT NULL | Nome do município normalizado em caixa alta (`SANTO ANDRE`). |
| `tipo_abastecimento` | `STRING` | NOT NULL | Modalidade do suprimento (`SAA` ou `SAC`). |
| `codigo_forma` | `STRING` | NOT NULL | Identificador único do sistema no SISAGUA (`S354780000007`). |
| `nome_forma` | `STRING` | NULL | Nome do setor de abastecimento (`GUSTAVO`, `OLHO DAGUA`). |
| `nome_eta` | `STRING` | NULL | Nome da Estação de Tratamento de Água de origem. |
| `tipo_filtracao` | `STRING` | NULL | Tecnologia de filtração utilizada (`FILTRAÇÃO RÁPIDA`). |
| `ano_referencia` | `INT` | NOT NULL | Ano de referência da análise (chave de partição). |
| `mes_referencia` | `INT` | NOT NULL | Mês de referência da análise (chave de partição). |
| `ponto_monitoramento` | `STRING` | NOT NULL | `SAÍDA DO TRATAMENTO` ou `REDE DE DISTRIBUIÇÃO`. |
| `parametro` | `STRING` | NOT NULL | Nome do parâmetro avaliado (`Fluoreto (mg/L)`). |
| `campo_metrica` | `STRING` | NOT NULL | Métrica de amostragem (`Percentil 95`, `Amostras`, etc.). |
| `valor_medido` | `DOUBLE` | NULL | Valor convertido numericamente (ponto decimal). |
| `flag_censurado` | `BOOLEAN` | NOT NULL | `TRUE` se o valor original era `MENOR_LD` ou `MENOR_LQ`. |
| `data_processamento` | `TIMESTAMP` | NOT NULL | Data e hora em que o Glue Job processou o lote. |

---

## 3. Regras de Sanitização e Qualidade de Dados (Data Quality)

1. **Conversão Numérica Decimal:**
   * No arquivo bruto, o campo `Valor` utiliza a vírgula como separador (`0,8`). O pipeline substitui `,` por `.` e converte a string para tipo float/decimal.
2. **Tratamento de Dados Censurados Laboratoriais:**
   * Textos como `MENOR_LD` (Abaixo do Limite de Detecção) e `MENOR_LQ` (Abaixo do Limite de Quantificação) são interceptados.
   * O campo `flag_censurado` é marcado como `TRUE`, e `valor_medido` é imputado como `0.0000 mg/L` para garantir que o paciente seja devidamente protegido contra ausência crônica de flúor.
3. **Encoding de Caracteres:**
   * A extração lê a base federal em `latin-1` (ISO-8859-1) e transcreve para `utf-8` nas camadas Silver e Gold, preservando caracteres especiais e acentos.
4. **Remoção de Duplicidades:**
   * Medições idênticas (mesmo sistema, ETA, ano, mês, ponto, parâmetro e campo) são consolidadas, mantendo-se a de data de inserção mais recente.
