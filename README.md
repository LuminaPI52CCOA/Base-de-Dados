# MEMÓRIA TÉCNICA E DOCUMENTAÇÃO DE DATA SCIENCE: INTEGRAÇÃO SISAGUA & LUMINA MANAGEMENT

**Projeto:** Lumina Management — Sistema de Gestão Odontológica Humanizada e Preventiva  
**Módulo:** Motor de Inteligência em Saúde Preditiva & Régua de Relacionamento  
**Domínio de Aplicação:** Monitoramento Geoespacial de Fluoretação da Água e Automação de Campanhas Preventivas  
**Município Piloto:** Santo André – SP (Código IBGE: `354780`)  
**Data da Documentação:** Agosto de 2026  
**Versão:** 1.0 (Nível de Detalhamento Técnico-Acadêmico)

---

## 📑 Sumário Executivo

1. [Contextualização do Projeto Lumina Management](#1-contextualização-do-projeto-lumina-management)
2. [Fundamentação Científica e Epidemiológica (Odontologia Baseada em Evidências)](#2-fundamentação-científica-e-epidemiológica-odontologia-baseada-em-evidências)
3. [Marco Regulatório Sanitário e Parâmetros Físico-Químicos](#3-marco-regulatório-sanitário-e-parâmetros-físico-químicos)
4. [Taxonomia e Arquitetura das Fontes de Dados Públicas (SISAGUA)](#4-taxonomia-e-arquitetura-das-fontes-de-dados-públicas-sisagua)
5. [Dicionário Completo de Dados das Bases do SISAGUA](#5-dicionário-completo-de-dados-das-bases-do-sisagua)
6. [Pipeline de Engenharia e Tratamento de Dados (ETL)](#6-pipeline-de-engenharia-e-tratamento-de-dados-etl)
7. [Modelo de Integração com o Backend Lumina (Spring Boot + MySQL + AWS SNS)](#7-modelo-de-integração-com-o-backend-lumina-spring-boot--mysql--aws-sns)
8. [Inventário e Mapeamento de Arquivos no Repositório](#8-inventário-e-mapeamento-de-arquivos-no-repositório)
9. [Referências Oficiais, Legislação e Links de Acesso](#9-referências-oficiais-legislação-e-links-de-acesso)

---

## 1. Contextualização do Projeto Lumina Management

O **Lumina Management** é uma plataforma SaaS corporativa desenvolvida para clínicas odontológicas que operam sob os princípios da odontologia humanizada, diagnóstica e preditiva. Diferente de softwares tradicionais de gestão de consultórios, focados estritamente no faturamento e prontuário estático, o Lumina possui uma arquitetura orientada a eventos com uma **Régua de Relacionamento Proativa** acoplada ao ecossistema em nuvem (**AWS SNS / SQS** e **Spring Boot**).

### O Desafio de Business Intelligence & Data Science Aplicada
A hipótese de negócio e inovação do Lumina fundamenta-se no conceito de **Saúde Preditiva**:
> *Se o sistema for capaz de ingerir dados epidemiológicos e sanitários públicos externos — especificamente os índices de fluoretação das águas de abastecimento público —, ele pode identificar precocemente micro-regiões e bairros desprovidos da concentração ideal de flúor. Com isso, o motor de automação atua na causa-raiz, alertando os pacientes dessas zonas para a necessidade de reforço preventivo (como profilaxias e aplicações tópicas de verniz fluoretado) antes do desenvolvimento clínico de lesões cariosas.*

```
┌──────────────────────────────────────────────┐
│  Fonte Pública: SISAGUA (Ministério Saúde)   │
│  Medições de Fluoreto por ETA / Distribuição │
└──────────────────────┬───────────────────────┘
                       │ Ingestão / ETL
                       ▼
┌──────────────────────────────────────────────┐
│   Motor de Data Science / BI do Lumina       │
│   Identificação de Zonas Sub-óptimas (<0,6)  │
└──────────────────────┬───────────────────────┘
                       │ Cruzamento Espacial (Bairro / CEP)
                       ▼
┌──────────────────────────────────────────────┐
│ Base de Pacientes Lumina (Spring Boot/MySQL) │
│ Segmentação de Pacientes por Risco Ambiental │
└──────────────────────┬───────────────────────┘
                       │ Disparo de Alerta Preditivo
                       ▼
┌──────────────────────────────────────────────┐
│     Régua de Relacionamento (AWS SNS)        │
│   Campanha Preventiva: "Aplicação de Flúor"  │
└──────────────────────────────────────────────┘
```

---

## 2. Fundamentação Científica e Epidemiológica (Odontologia Baseada em Evidências)

A fluoretação das águas de abastecimento público é reconhecida pela Organização Mundial da Saúde (OMS) e pelo Ministério da Saúde como a medida de saúde pública mais eficaz, abrangente e de melhor custo-benefício para o controle e redução da cárie dentária na população.

### 2.1 Mecanismo de Ação Bioquímica do Íon Flúor ($F^-$)
1. **Desmineralização e Remineralização:** O íon fluoreto presente na saliva e no biofilme dentário atua diretamente na interface esmalte-meio bucal. Durante episódios de queda de pH (gerados pela fermentação de carboidratos pelas bactérias cariogênicas como *Streptococcus mutans*), a hidroxiapatita do esmalte $[Ca_{10}(PO_4)_6(OH)_2]$ sofre desmineralização.
2. **Formação de Fluorapatita:** Na presença contínua de flúor em concentrações adequadas, ocorre a precipitação de **fluorapatita** $[Ca_{10}(PO_4)_6F_2]$, uma estrutura mineral significativamente mais estável e resistente à dissolução ácida (ponto crítico de desmineralização reduzido de $pH \approx 5,5$ para $pH \approx 4,5$).

### 2.2 Faixas de Concentração e Risco Clínico

| Faixa de Fluoreto ($mg/L$) | Classificação Sanitária | Impacto Odontológico Clínico | Conduta e Ação do Sistema Lumina |
| :--- | :--- | :--- | :--- |
| **$< 0,55\text{ mg/L}$** | **Sub-óptimo / Insuficiente** | Ausência do efeito preventivo populacional; aumento estatístico no índice CPO-D (dentes cariados, perdidos e obturados). | **Gatilho Crítico:** Dispara campanha de alerta imediato via AWS SNS orientando consulta preventiva e aplicação tópica profissional de flúor. |
| **$0,55 \text{ a } 0,84\text{ mg/L}$** | **Óptimo / Ideal** | Máxima prevenção de cárie com risco desprezível de fluorose dentária (balanço risco-benefício perfeito). | Manutenção do fluxo de rotina (profilaxia semestral padrão). |
| **$0,85 \text{ a } 1,49\text{ mg/L}$** | **Supra-óptimo Leve** | Proteção contra cárie mantida, mas com risco moderado de fluorose dentária em crianças de 0 a 8 anos. | Registro de monitoramento interno; recomendação de dosagem controlada de dentifrícios infantis. |
| **$\ge 1,50\text{ mg/L}$** | **Inaceitável / VMP Excedido** | Violação do Valor Máximo Permitido (Portaria GM/MS 888/2021). Risco severo de fluorose estética e esquelética. | Notificação informativa aos pacientes pediátricos para evitar ingestão de fórmulas infantis preparadas com água da torneira. |

---

## 3. Marco Regulatório Sanitário e Parâmetros Físico-Químicos

O monitoramento da qualidade da água no Brasil é regulamentado pela **Portaria de Consolidação GM/MS nº 5/2017 (Anexo XX)**, atualizada pela **Portaria GM/MS nº 888, de 4 de maio de 2021**, que estabelece o padrão nacional de potabilidade da água para consumo humano.

### 3.1 Correlação de Parâmetros Básicos Monitorados

Além do Fluoreto, a base do SISAGUA avalia simultaneamente:

1. **Cloro Residual Livre ($mg/L$):**
   * *Padrão de Potabilidade:* Mínimo de $0,2\text{ mg/L}$ e máximo de $2,0\text{ mg/L}$ em qualquer ponto da rede.
   * *Relevância:* Garante a desinfecção bacteriológica da água encanada.
2. **Turbidez ($uT$ - Unidades de Turbidez):**
   * *Padrão:* Percentil 95 menor ou igual a $0,5\text{ uT}$ (filtração rápida) e VMP de $5,0\text{ uT}$ na rede.
   * *Relevância:* Indicador de eficácia do processo de coagulação/filtração e barreira contra protozoários (*Giardia* e *Cryptosporidium*).
3. **pH (Potencial Hidrogeniônico):**
   * *Padrão:* Faixa recomendada entre $6,0$ e $9,0$.
   * *Relevância:* Águas com pH excessivamente ácido ($< 6,0$) aceleram a corrosão das tubulações e podem potencializar a desmineralização dentária em indivíduos suscetíveis.
4. **Coliformes Totais e *Escherichia coli*:**
   * *Padrão:* Ausência em 100 mL de amostra na saída do tratamento e na rede de distribuição.
   * *Relevância:* Indicador direto de contaminação fecal e risco de doenças de veiculação hídrica.

---

## 4. Taxonomia e Arquitetura das Fontes de Dados Públicas (SISAGUA)

O **SISAGUA** é o sistema estruturante do Programa Nacional de Vigilância da Qualidade da Água para Consumo Humano (**VIGIAGUA**). Ele divide seus repositórios abertos em três grandes domínios funcionais e periódicos:

```
                            SISTEMA SISAGUA (Ministério da Saúde)
                                           │
         ┌─────────────────────────────────┼─────────────────────────────────┐
         ▼                                 ▼                                 ▼
┌──────────────────┐             ┌───────────────────┐             ┌───────────────────┐
│ CONTROLE MENSAL  │             │ CONTROLE SEMESTRAL│             │  CADASTROS GERAIS │
│(Parâmetros Básico│             │(Agrotóxicos/Metais│             │ (Formas de Abast./│
│- FLÚOR, Cloro, pH│             │- Semestral/Trim.) │             │    População)     │
└────────┬─────────┘             └─────────┬─────────┘             └─────────┬─────────┘
         │                                 │                                 │
   Usado no LUMINA                   Não contém Flúor                  Dados de Apoio
 (Base Principal 2026)             (Arquivos Iniciais)              (Captação/População)
```

### 4.1 Entendendo os Tipos de Abastecimento
* **SAA (Sistema de Abastecimento de Água):** Instalação pública composta por captação, Estações de Tratamento de Água (ETAs), reservatórios e rede de distribuição operada por concessionária (em Santo André: **SABESP / SEMASA**). **É a rede que abastece os domicílios e onde ocorre a fluoretação artificial controlada.**
* **SAC (Solução Alternativa Coletiva):** Modalidade de suprimento privado compartilhado sem rede pública (ex: poços artesianos de shoppings, indústrias, condomínios fechados e hospitais).
* **SAI (Solução Alternativa Individual):** Poço artesiano ou cacimba em residência unifamiliar.

### 4.2 Lição Aprendida no Diagnóstico Inicial
O arquivo inicialmente analisado (`controle_semestral_2026.csv`) continha exclusivamente compostos complexos (como agrotóxicos *Flutriafol* e *Trifluralina* e metais pesados). A migração técnica para a base **`controle_mensal_parametros_basicos_2026.csv`** foi a chave para obter os registros reais de **`Fluoreto (mg/L)`**.

---

## 5. Dicionário Completo de Dados Multicamadas (Bronze, Silver e Gold)

> [!NOTE]
> Para consulta rápida e modular, este dicionário também está versionado em documento dedicado: [DICIONARIO_DE_DADOS.md](file:///c:/Users/lucas/OneDrive/Área%20de%20Trabalho/Projeto%20LUMINA/Base-de-Dados/DICIONARIO_DE_DADOS.md).

### 5.1 Camada Bronze: Base Bruta SISAGUA (Controle Mensal Parâmetros Básicos)
Especificação dos 22 atributos presentes no arquivo oficial `controle_mensal_parametros_basicos_2026.csv` (CSV nacional bruto, separador `;`, encoding `latin-1`):

| Índice | Nome da Coluna no CSV | Tipo de Dado | Exemplo Real | Nulabilidade | Descrição e Regra de Negócio | Aplicação no Sistema Lumina |
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

### 5.2 Domínio de Valores Aceitos (Enums e Categorias da Camada Bronze)

* **`Tipo da Forma de Abastecimento` (Índice 11):**
  * `SAA` (*Sistema de Abastecimento de Água*): Rede pública formal operada por concessionária (SABESP). **Alvo primário do Lumina (100% das residências).**
  * `SAC` (*Solução Alternativa Coletiva*): Captações privadas (poços de indústrias, shoppings, condomínios).
  * `SAI` (*Solução Alternativa Individual*): Poço de residência unifamiliar isolada.
* **`Tipo de Filtração` (Índice 15):**
  * `FILTRAÇÃO RÁPIDA`: Padrão industrial das ETAs de Santo André (coagulação, floculação, decantação e leito de areia).
  * `FILTRAÇÃO LENTA`: Processo biológico de filtração.
  * `SEM FILTRAÇÃO`: Captação profunda que recebe apenas desinfecção e fluoretação direta.
* **`Ponto de Monitoramento` (Índice 18):**
  * `SAÍDA DO TRATAMENTO`: Amostra colhida na saída imediata da ETA. Avalia a conformidade operacional da concessionária.
  * `REDE DE DISTRIBUIÇÃO`: Amostra colhida nas torneiras e hidrantes da rua. **Reflete a água real que chega ao paciente.**
* **`Parâmetro` (Índice 19):**
  * `Fluoreto (mg/L)`: Principal indicador preventivo de saúde bucal (**Foco Central do Lumina**).
  * `Cloro residual livre (mg/L)`: Indicador de desinfecção bacteriológica.
  * `Turbidez (uT)`: Indicador de clarificação física e barreira contra protozoários.
  * `pH`: Acidez da água (pH $< 6,0$ favorece desmineralização dentária).
  * `Coliformes totais` e `Escherichia coli`: Indicadores de integridade microbiológica e contaminação fecal.
* **`Campo` (Índice 20 - Métricas Analíticas):**
  * `Percentil 95`: Valor no qual 95% das amostras mensais ficaram abaixo ou iguais (padrão sanitário oficial). **Utilizado como a concentração de Flúor do mês.**
  * `Média das temperaturas máximas diárias(°C)`: Utilizada para calcular a faixa ideal de fluoreto segundo a média térmica da cidade.
  * `Número de amostras analisadas`: Volume amostral do mês (significância estatística).
  * `Número de dados > 1,5 mg/L` e `Número de dados <= 1,5 mg/L`: Contadores de conformidade com o teto legal de toxicidade/fluorose.

---

### 5.3 Camada Silver: Dados Otimizados (AWS S3 / Apache Parquet)

Armazenamento colunar particionado em `s3://lumina-silver/sisagua/tratado/controle_mensal/` para processamento no Athena e Glue:

| Nome da Coluna Parquet | Tipo no Athena | Nulabilidade | Descrição |
| :--- | :---: | :---: | :--- |
| `codigo_ibge` | `INT` | NOT NULL | Código IBGE do município (`354780`). |
| `municipio` | `STRING` | NOT NULL | Nome padronizado em caixa alta (`SANTO ANDRE`). |
| `tipo_abastecimento` | `STRING` | NOT NULL | Modalidade (`SAA` ou `SAC`). |
| `codigo_forma` | `STRING` | NOT NULL | ID alfanumérico no SISAGUA (`S354780000007`). |
| `nome_forma` | `STRING` | NULL | Nome do setor de abastecimento (`GUSTAVO`, `OLHO DAGUA`). |
| `nome_eta` | `STRING` | NULL | Nome da ETA responsável. |
| `tipo_filtracao` | `STRING` | NULL | Tecnologia de filtração. |
| `ano_referencia` | `INT` | NOT NULL | Ano da medição (chave de partição). |
| `mes_referencia` | `INT` | NOT NULL | Mês da medição (chave de partição). |
| `ponto_monitoramento` | `STRING` | NOT NULL | `SAÍDA DO TRATAMENTO` ou `REDE DE DISTRIBUIÇÃO`. |
| `parametro` | `STRING` | NOT NULL | Grandeza avaliada (`Fluoreto (mg/L)`). |
| `campo_metrica` | `STRING` | NOT NULL | Métrica estatística (`Percentil 95`). |
| `valor_medido` | `DOUBLE` | NULL | Concentração tratada numericamente (ponto decimal). |
| `flag_censurado` | `BOOLEAN` | NOT NULL | `TRUE` se o laudo original continha `MENOR_LD` ou `MENOR_LQ`. |
| `data_processamento` | `TIMESTAMP` | NOT NULL | Carimbo de data/hora da geração do Parquet. |

---

### 5.4 Camada Gold / Produção: Dicionário das Tabelas Relacionais (MySQL)

Esquema de banco de dados relacional que sustenta as regras de negócio no Spring Boot:

#### Atributos de `tbl_qualidade_agua_sisagua` (Medições Históricas Consolidadas):
| Atributo SQL | Tipo Físico | Chave | Nulo | Descrição e Regra de Negócio |
| :--- | :--- | :---: | :---: | :--- |
| `id_medicao` | `BIGINT` | PK | Não | Chave surrogate autoincrementada. |
| `codigo_ibge` | `INT` | Index | Não | Código IBGE do município (ex: `354780`). |
| `municipio` | `VARCHAR(100)` | - | Não | Nome do município (`SANTO ANDRE`). |
| `tipo_abastecimento` | `ENUM('SAA','SAC','SAI')`| - | Não | Tipo de sistema; consultas da régua filtram `'SAA'`. |
| `codigo_forma` | `VARCHAR(50)` | - | Não | Código de cadastro no SISAGUA (`S354780000007`). |
| `nome_forma` | `VARCHAR(150)` | - | Sim | Nome da captação ou reservatório. |
| `nome_eta` | `VARCHAR(150)` | Index | Sim | Nome da ETA. **Chave de junção com `tbl_zona_abastecimento_bairro`.** |
| `ano_referencia` | `INT` | Index | Não | Ano do laudo. |
| `mes_referencia` | `INT` | Index | Não | Mês do laudo (1 a 12). |
| `ponto_monitoramento` | `VARCHAR(100)` | - | Não | Local da coleta (`REDE DE DISTRIBUIÇÃO` ou `SAÍDA DO TRATAMENTO`). |
| `parametro` | `VARCHAR(100)` | Index | Não | Nome do parâmetro (`Fluoreto (mg/L)`). |
| `campo_metrica` | `VARCHAR(100)` | - | Não | Descritor da medição (`Percentil 95`). |
| `valor_medido` | `DECIMAL(8,4)` | - | Sim | Concentração em $mg/L$ tratada com 4 casas decimais. |
| `flag_censurado` | `BOOLEAN` | - | Não | Indicador de medição abaixo do limite de detecção (`MENOR_LD`). |
| `data_importacao` | `TIMESTAMP` | - | Não | Carimbo de data/hora do insert no banco de dados. |

#### Atributos de `tbl_zona_abastecimento_bairro` (De-Para Geoespacial de Distribuição):
| Atributo SQL | Tipo Físico | Chave | Nulo | Descrição e Regra de Negócio |
| :--- | :--- | :---: | :---: | :--- |
| `id_relacao` | `BIGINT` | PK | Não | Chave surrogate da relação geográfica. |
| `nome_eta` | `VARCHAR(150)` | Index | Não | Nome da ETA no SISAGUA (ex: `ETA RIO GRANDE`, `UTA GUSTAVO`). |
| `bairro` | `VARCHAR(100)` | - | Não | Nome oficial do bairro atendido (ex: `Campestre`, `Centro`, `Vila Guiomar`). |
| `municipio` | `VARCHAR(100)` | - | Não | Município de Santo André (`SANTO ANDRE`). |
| `cep_prefixo` | `VARCHAR(5)` | Index | Não | Primeiros 5 dígitos do CEP atendido (ex: `'09010'`). Permite cruzamento com o CEP do paciente. |
| `observacao` | `VARCHAR(255)` | - | Sim | Observações operacionais de manobra de rede. |
| `data_cadastro` | `TIMESTAMP` | - | Não | Data de inserção do registro. |

---

## 6. Pipeline de Engenharia e Tratamento de Dados (ETL)

O processamento dos dados públicos é executado pelo Jupyter Notebook [Notebook - Santo Andre.ipynb](file:///c:/Users/lucas/OneDrive/Área%20de%20Trabalho/Projeto%20LUMINA/Base-de-Dados/Arquivos%20SISAGUA/Notebook%20-%20Santo%20Andre.ipynb). Devido ao volume massivo das bases brutas nacionais (arquivos descompactados superiores a 2 GB e mais de 10 milhões de linhas), implementou-se uma arquitetura de extração em lotes.

### 6.1 Estrutura do Código de Extração (Python / Pandas)

```python
import pandas as pd

# ==============================================================================
# BLOCO 1: Extração do Controle Mensal (Parâmetros Básicos 2026)
# ==============================================================================
print("[ETL] Iniciando processamento do Controle Mensal 2026...")

pedacos_mensal = pd.read_csv(
    "controle_mensal_parametros_basicos_2026.csv",
    chunksize=50000,
    sep=";",
    encoding="latin-1",
    low_memory=False
)

# Filtra Santo André pela 5ª coluna (índice 4 - Município)
santo_andre_mensal = pd.concat(
    pedaco[pedaco.iloc[:, 4].astype(str).str.contains("SANTO ANDRE", na=False, case=False)]
    for pedaco in pedacos_mensal
)

# Exporta base consolidada de Santo André
santo_andre_mensal.to_csv(
    "sisagua_santo_andre_mensal_parametros_basicos_2026.csv",
    index=False,
    sep=";",
    encoding="utf-8"
)
print(f"[ETL] Base de Santo André gerada: {len(santo_andre_mensal)} linhas.")

# ==============================================================================
# BLOCO 2: Segmentação Específica de Fluoreto para o Lumina
# ==============================================================================
coluna_parametro = santo_andre_mensal.columns[19] # Índice 19: Parâmetro
fluor_df = santo_andre_mensal[
    santo_andre_mensal[coluna_parametro].astype(str).str.contains("Flu", case=False, na=False)
].copy()

# Tratamento numérico do campo de resultado (Vírgula para Ponto)
coluna_valor = santo_andre_mensal.columns[21]     # Índice 21: Valor
coluna_campo = santo_andre_mensal.columns[20]     # Índice 20: Campo

fluor_df.to_csv(
    "sisagua_santo_andre_fluoreto_2026.csv",
    index=False,
    sep=";",
    encoding="utf-8"
)
print(f"[ETL] Base exclusiva de Fluoreto gerada: {len(fluor_df)} registros.")
```

### 6.2 Regras de Sanitização de Dados
1. **Normalização de Encodings:** Leitura obrigatória em `latin-1` (padrão legado dos sistemas federais DATASUS) e conversão para `utf-8` para compatibilidade com o MySQL e Spring Boot.
2. **Tratamento Numérico Decimal:** Substituição de `,` por `.` no campo `Valor` antes de cast para `DECIMAL(6,3)`.
3. **Tratamento de Dados Censurados:** Mapeamento de `MENOR_LD` (Limite de Detecção) e `MENOR_LQ` (Limite de Quantificação) como $0.000\text{ mg/L}$ para fins de análise de risco de sub-dosagem.

---

## 7. Modelo de Integração com o Backend Lumina (Spring Boot + MySQL + AWS SNS)

Para operacionalizar a ingestão no ecossistema de produção do Lumina, os dados sanitizados são persistidos no banco de dados relacional e vinculados aos pacientes.

### 7.1 Esquema Físico de Banco de Dados (MySQL DDL)

```sql
-- 1. Tabela de Medições Históricas do SISAGUA
CREATE TABLE tbl_qualidade_agua_sisagua (
    id_medicao BIGINT AUTO_INCREMENT PRIMARY KEY,
    codigo_ibge INT NOT NULL,
    municipio VARCHAR(100) NOT NULL,
    tipo_abastecimento ENUM('SAA', 'SAC', 'SAI') NOT NULL,
    codigo_forma VARCHAR(50) NOT NULL,
    nome_forma VARCHAR(150),
    nome_eta VARCHAR(150),
    ano_referencia INT NOT NULL,
    mes_referencia INT NOT NULL,
    ponto_monitoramento VARCHAR(100) NOT NULL,
    parametro VARCHAR(100) NOT NULL,
    campo_metrica VARCHAR(100) NOT NULL,
    valor_medido DECIMAL(8,4),
    data_importacao TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_busca_preditiva (codigo_ibge, ano_referencia, mes_referencia, parametro)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 2. Tabela de De-Para: ETA / Zona de Abastecimento para Bairros
CREATE TABLE tbl_zona_abastecimento_bairro (
    id_relacao BIGINT AUTO_INCREMENT PRIMARY KEY,
    nome_eta VARCHAR(150) NOT NULL,
    bairro VARCHAR(100) NOT NULL,
    municipio VARCHAR(100) NOT NULL,
    cep_prefixo VARCHAR(5) NOT NULL, -- Ex: '09010', '09220'
    INDEX idx_cep (cep_prefixo)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 3. Consulta de Cruzamento Preditivo para Disparo de Campanhas
-- Identifica pacientes em bairros atendidos por ETAs com Flúor < 0.6 mg/L no último mês
SELECT DISTINCT 
    p.id_paciente,
    p.nome,
    p.telefone,
    p.email,
    p.cep,
    p.bairro,
    med.nome_eta,
    med.valor_medido AS nivel_fluor_recente
FROM tbl_pacientes p
JOIN tbl_zona_abastecimento_bairro zb 
    ON LEFT(p.cep, 5) = zb.cep_prefixo
JOIN tbl_qualidade_agua_sisagua med 
    ON med.nome_eta = zb.nome_eta
WHERE med.parametro = 'Fluoreto (mg/L)'
  AND med.campo_metrica = 'Percentil 95'
  AND med.ano_referencia = 2026
  AND med.mes_referencia = 6
  AND med.valor_medido < 0.60
  AND p.status_ativo = TRUE;
```

### 7.2 Fluxo de Execução no Spring Boot (Régua de Relacionamento)

```java
@Service
public class ReguaSaudePreditivaService {

    @Autowired
    private QualidadeAguaRepository aguaRepository;

    @Autowired
    private AmazonSNSClient snsClient;

    @Scheduled(cron = "0 0 9 15 * ?") // Executa no dia 15 de cada mês
    public void dispararCampanhaFluorPrevenção() {
        List<PacienteRiscoFluorDTO> pacientes = aguaRepository.buscarPacientesEmZonasSubOptimas();

        for (PacienteRiscoFluorDTO paciente : pacientes) {
            String mensagem = String.format(
                "Olá %s, notamos que a sua região em %s registrou oscilações na fluoretação da água no último mês (%.2f mg/L). " +
                "Para proteger seu esmalte dentário contra cáries, recomendamos uma aplicação preventiva de flúor na Clínica Lumina. " +
                "Agende agora com condições especiais!",
                paciente.getNome(), paciente.getBairro(), paciente.getNivelFluor()
            );

            PublishRequest request = new PublishRequest()
                .withPhoneNumber(paciente.getTelefone())
                .withMessage(mensagem);

            snsClient.publish(request);
        }
    }
}
```

---

## 8. Inventário e Mapeamento de Arquivos no Repositório

Estrutura de diretórios e arquivos do repositório:

```
Base-de-Dados/
│
├── .gitignore                                           <-- Regras de exclusão para arquivos pesados (> 100MB)
├── DICIONARIO_DE_DADOS.md                               <-- Dicionário completo multicamadas (Bronze, Silver e Gold)
├── README.md                                            <-- Esta documentação técnica
│
├── memory/
│   └── MEMORIA_TECNICA_SISAGUA_LUMINA.md                <-- Cópia de referência técnica da base
│
└── Arquivos SISAGUA/
    ├── Notebook - Santo Andre.ipynb                     <-- Jupyter Notebook com o Pipeline de ETL e Análise
    │
    ├── Tratado - Sto Andre/                             <-- Bases filtradas e tratadas (Versionadas no Git)
    │   ├── sisagua_santo_andre_fluoreto_2026.csv        <-- Dataset refinado de Flúor para o Lumina (~312 KB)
    │   ├── sisagua_santo_andre_mensal_parametros_basicos_2026.csv <-- Base filtrada mensal completa (~3.2 MB)
    │   └── sisagua_semestral_santo_andre_2026.csv       <-- Base filtrada semestral de Sto André (~1.6 MB)
    │
    └── Arquivos de Controle (sem tratamento)/           <-- Bases brutas locais (Ignoradas no Git / > 5 GB)
        ├── cadastro_populacao_abastecida.csv            <-- Histórico demográfico nacional (~555 MB)
        ├── controle_mensal_parametros_basicos_2026.csv  <-- Base bruta mensal nacional (~1.89 GB)
        ├── controle_mensal_plano_amostragem_2023.json   <-- Metas e amostragem nacional (~2.23 GB)
        ├── controle_semestral_2026.csv                  <-- Base bruta semestral nacional (~504 MB)
        └── SISAGUA - Controle Mensal...Dicionário.pdf   <-- Dicionário oficial do Ministério da Saúde
```

---

## 9. Referências Oficiais, Legislação e Links de Acesso

1. **Ministério da Saúde (Brasil):**
   * *Portaria GM/MS nº 888, de 4 de maio de 2021.* Altera o Anexo XX da Portaria de Consolidação GM/MS nº 5/2017 para estabelecer os procedimentos e responsabilidades relativos ao controle e vigilância da qualidade da água para consumo humano e seu padrão de potabilidade.
   * *Caderno de Diretrizes da Vigilância da Qualidade da Água para Consumo Humano.* Programa Nacional VIGIAGUA.
2. **Repositórios Oficiais de Dados Abertos (SUS / CKAN):**
   * **Controle Mensal (Parâmetros Básicos 2026):**  
     `https://s3.sa-east-1.amazonaws.com/ckan.saude.gov.br/SISAGUA/controle_mensal_parametros_basicos_2026_csv.zip`
   * **Controle Mensal (Parâmetros Básicos 2025):**  
     `https://s3.sa-east-1.amazonaws.com/ckan.saude.gov.br/SISAGUA/controle_mensal_parametros_basicos_2025_csv.zip`
   * **Catálogo de Dados Abertos:**  
     [Portal Brasileiro de Dados Abertos - Conjunto SISAGUA](https://dadosabertos.saude.gov.br/dataset/sisagua-controle-mensal-parametros-basicos)
3. **Literatura Científica Odontológica:**
   * CECOL/USP (Centro Colaborador do Ministério da Saúde em Vigilância da Saúde Bucal): *Consenso Técnico sobre Classificação de Águas de Abastecimento Público segundo o Teor de Flúor*.
   * World Health Organization (WHO): *Fluorides and oral health: report of a WHO Expert Committee on Oral Health Status and Fluoride Use*. Technical Report Series, Geneva.
   * Fejerskov, O., & Kidd, E. (2008). *Cárie Dentária: A Evolução do Conceito*. Ed. Santos.
