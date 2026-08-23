# \# MEMÓRIA TÉCNICA E DOCUMENTAÇÃO DE DATA SCIENCE: INTEGRAÇÃO SISAGUA \& LUMINA MANAGEMENT

# 

# \*\*Projeto:\*\* Lumina Management — Sistema de Gestão Odontológica Humanizada e Preventiva  

# \*\*Módulo:\*\* Motor de Inteligência em Saúde Preditiva \& Régua de Relacionamento  

# \*\*Domínio de Aplicação:\*\* Monitoramento Geoespacial de Fluoretação da Água e Automação de Campanhas Preventivas  

# \*\*Município Piloto:\*\* Santo André – SP (Código IBGE: `354780`)  

# \*\*Data da Documentação:\*\* Agosto de 2026  

# \*\*Versão:\*\* 1.0 (Nível de Detalhamento Técnico-Acadêmico)

# 

# \---

# 

# \## 📑 Sumário Executivo

# 

# 1\. \[Contextualização do Projeto Lumina Management](#1-contextualização-do-projeto-lumina-management)

# 2\. \[Fundamentação Científica e Epidemiológica (Odontologia Baseada em Evidências)](#2-fundamentação-científica-e-epidemiológica-odontologia-baseada-em-evidências)

# 3\. \[Marco Regulatório Sanitário e Parâmetros Físico-Químicos](#3-marco-regulatório-sanitário-e-parâmetros-físico-químicos)

# 4\. \[Taxonomia e Arquitetura das Fontes de Dados Públicas (SISAGUA)](#4-taxonomia-e-arquitetura-das-fontes-de-dados-públicas-sisagua)

# 5\. \[Dicionário Completo de Dados das Bases do SISAGUA](#5-dicionário-completo-de-dados-das-bases-do-sisagua)

# 6\. \[Pipeline de Engenharia e Tratamento de Dados (ETL)](#6-pipeline-de-engenharia-e-tratamento-de-dados-etl)

# 7\. \[Modelo de Integração com o Backend Lumina (Spring Boot + MySQL + AWS SNS)](#7-modelo-de-integração-com-o-backend-lumina-spring-boot--mysql--aws-sns)

# 8\. \[Inventário e Mapeamento de Arquivos no Repositório](#8-inventário-e-mapeamento-de-arquivos-no-repositório)

# 9\. \[Referências Oficiais, Legislação e Links de Acesso](#9-referências-oficiais-legislação-e-links-de-acesso)

# 

# \---

# 

# \## 1. Contextualização do Projeto Lumina Management

# 

# O \*\*Lumina Management\*\* é uma plataforma SaaS corporativa desenvolvida para clínicas odontológicas que operam sob os princípios da odontologia humanizada, diagnóstica e preditiva. Diferente de softwares tradicionais de gestão de consultórios, focados estritamente no faturamento e prontuário estático, o Lumina possui uma arquitetura orientada a eventos com uma \*\*Régua de Relacionamento Proativa\*\* acoplada ao ecossistema em nuvem (\*\*AWS SNS / SQS\*\* e \*\*Spring Boot\*\*).

# 

# \### O Desafio de Business Intelligence \& Data Science Aplicada

# A hipótese de negócio e inovação do Lumina fundamenta-se no conceito de \*\*Saúde Preditiva\*\*:

# > \*Se o sistema for capaz de ingerir dados epidemiológicos e sanitários públicos externos — especificamente os índices de fluoretação das águas de abastecimento público —, ele pode identificar precocemente micro-regiões e bairros desprovidos da concentração ideal de flúor. Com isso, o motor de automação atua na causa-raiz, alertando os pacientes dessas zonas para a necessidade de reforço preventivo (como profilaxias e aplicações tópicas de verniz fluoretado) antes do desenvolvimento clínico de lesões cariosas.\*

# 

# ```

# ┌──────────────────────────────────────────────┐

# │  Fonte Pública: SISAGUA (Ministério Saúde)   │

# │  Medições de Fluoreto por ETA / Distribuição │

# └──────────────────────┬───────────────────────┘

# &#x20;                      │ Ingestão / ETL

# &#x20;                      ▼

# ┌──────────────────────────────────────────────┐

# │   Motor de Data Science / BI do Lumina       │

# │   Identificação de Zonas Sub-óptimas (<0,6)  │

# └──────────────────────┬───────────────────────┘

# &#x20;                      │ Cruzamento Espacial (Bairro / CEP)

# &#x20;                      ▼

# ┌──────────────────────────────────────────────┐

# │ Base de Pacientes Lumina (Spring Boot/MySQL) │

# │ Segmentação de Pacientes por Risco Ambiental │

# └──────────────────────┬───────────────────────┘

# &#x20;                      │ Disparo de Alerta Preditivo

# &#x20;                      ▼

# ┌──────────────────────────────────────────────┐

# │     Régua de Relacionamento (AWS SNS)        │

# │   Campanha Preventiva: "Aplicação de Flúor"  │

# └──────────────────────────────────────────────┘

# ```

# 

# \---

# 

# \## 2. Fundamentação Científica e Epidemiológica (Odontologia Baseada em Evidências)

# 

# A fluoretação das águas de abastecimento público é reconhecida pela Organização Mundial da Saúde (OMS) e pelo Ministério da Saúde como a medida de saúde pública mais eficaz, abrangente e de melhor custo-benefício para o controle e redução da cárie dentária na população.

# 

# \### 2.1 Mecanismo de Ação Bioquímica do Íon Flúor ($F^-$)

# 1\. \*\*Desmineralização e Remineralização:\*\* O íon fluoreto presente na saliva e no biofilme dentário atua diretamente na interface esmalte-meio bucal. Durante episódios de queda de pH (gerados pela fermentação de carboidratos pelas bactérias cariogênicas como \*Streptococcus mutans\*), a hidroxiapatita do esmalte $\[Ca\_{10}(PO\_4)\_6(OH)\_2]$ sofre desmineralização.

# 2\. \*\*Formação de Fluorapatita:\*\* Na presença contínua de flúor em concentrações adequadas, ocorre a precipitação de \*\*fluorapatita\*\* $\[Ca\_{10}(PO\_4)\_6F\_2]$, uma estrutura mineral significativamente mais estável e resistente à dissolução ácida (ponto crítico de desmineralização reduzido de $pH \\approx 5,5$ para $pH \\approx 4,5$).

# 

# \### 2.2 Faixas de Concentração e Risco Clínico

# 

# | Faixa de Fluoreto ($mg/L$) | Classificação Sanitária | Impacto Odontológico Clínico | Conduta e Ação do Sistema Lumina |

# | :--- | :--- | :--- | :--- |

# | \*\*$< 0,55\\text{ mg/L}$\*\* | \*\*Sub-óptimo / Insuficiente\*\* | Ausência do efeito preventivo populacional; aumento estatístico no índice CPO-D (dentes cariados, perdidos e obturados). | \*\*Gatilho Crítico:\*\* Dispara campanha de alerta imediato via AWS SNS orientando consulta preventiva e aplicação tópica profissional de flúor. |

# | \*\*$0,55 \\text{ a } 0,84\\text{ mg/L}$\*\* | \*\*Óptimo / Ideal\*\* | Máxima prevenção de cárie com risco desprezível de fluorose dentária (balanço risco-benefício perfeito). | Manutenção do fluxo de rotina (profilaxia semestral padrão). |

# | \*\*$0,85 \\text{ a } 1,49\\text{ mg/L}$\*\* | \*\*Supra-óptimo Leve\*\* | Proteção contra cárie mantida, mas com risco moderado de fluorose dentária em crianças de 0 a 8 anos. | Registro de monitoramento interno; recomendação de dosagem controlada de dentifrícios infantis. |

# | \*\*$\\ge 1,50\\text{ mg/L}$\*\* | \*\*Inaceitável / VMP Excedido\*\* | Violação do Valor Máximo Permitido (Portaria GM/MS 888/2021). Risco severo de fluorose estética e esquelética. | Notificação informativa aos pacientes pediátricos para evitar ingestão de fórmulas infantis preparadas com água da torneira. |

# 

# \---

# 

# \## 3. Marco Regulatório Sanitário e Parâmetros Físico-Químicos

# 

# O monitoramento da qualidade da água no Brasil é regulamentado pela \*\*Portaria de Consolidação GM/MS nº 5/2017 (Anexo XX)\*\*, atualizada pela \*\*Portaria GM/MS nº 888, de 4 de maio de 2021\*\*, que estabelece o padrão nacional de potabilidade da água para consumo humano.

# 

# \### 3.1 Correlação de Parâmetros Básicos Monitorados

# 

# Além do Fluoreto, a base do SISAGUA avalia simultaneamente:

# 

# 1\. \*\*Cloro Residual Livre ($mg/L$):\*\*

# &#x20;  \* \*Padrão de Potabilidade:\* Mínimo de $0,2\\text{ mg/L}$ e máximo de $2,0\\text{ mg/L}$ em qualquer ponto da rede.

# &#x20;  \* \*Relevância:\* Garante a desinfecção bacteriológica da água encanada.

# 2\. \*\*Turbidez ($uT$ - Unidades de Turbidez):\*\*

# &#x20;  \* \*Padrão:\* Percentil 95 menor ou igual a $0,5\\text{ uT}$ (filtração rápida) e VMP de $5,0\\text{ uT}$ na rede.

# &#x20;  \* \*Relevância:\* Indicador de eficácia do processo de coagulação/filtração e barreira contra protozoários (\*Giardia\* e \*Cryptosporidium\*).

# 3\. \*\*pH (Potencial Hidrogeniônico):\*\*

# &#x20;  \* \*Padrão:\* Faixa recomendada entre $6,0$ e $9,0$.

# &#x20;  \* \*Relevância:\* Águas com pH excessivamente ácido ($< 6,0$) aceleram a corrosão das tubulações e podem potencializar a desmineralização dentária em indivíduos suscetíveis.

# 4\. \*\*Coliformes Totais e \*Escherichia coli\*:\*\*

# &#x20;  \* \*Padrão:\* Ausência em 100 mL de amostra na saída do tratamento e na rede de distribuição.

# &#x20;  \* \*Relevância:\* Indicador direto de contaminação fecal e risco de doenças de veiculação hídrica.

# 

# \---

# 

# \## 4. Taxonomia e Arquitetura das Fontes de Dados Públicas (SISAGUA)

# 

# O \*\*SISAGUA\*\* é o sistema estruturante do Programa Nacional de Vigilância da Qualidade da Água para Consumo Humano (\*\*VIGIAGUA\*\*). Ele divide seus repositórios abertos em três grandes domínios funcionais e periódicos:

# 

# ```

# &#x20;                           SISTEMA SISAGUA (Ministério da Saúde)

# &#x20;                                          │

# &#x20;        ┌─────────────────────────────────┼─────────────────────────────────┐

# &#x20;        ▼                                 ▼                                 ▼

# ┌──────────────────┐             ┌───────────────────┐             ┌───────────────────┐

# │ CONTROLE MENSAL  │             │ CONTROLE SEMESTRAL│             │  CADASTROS GERAIS │

# │(Parâmetros Básico│             │(Agrotóxicos/Metais│             │ (Formas de Abast./│

# │- FLÚOR, Cloro, pH│             │- Semestral/Trim.) │             │    População)     │

# └────────┬─────────┘             └─────────┬─────────┘             └─────────┬─────────┘

# &#x20;        │                                 │                                 │

# &#x20;  Usado no LUMINA                   Não contém Flúor                  Dados de Apoio

# &#x20;(Base Principal 2026)             (Arquivos Iniciais)              (Captação/População)

# ```

# 

# \### 4.1 Entendendo os Tipos de Abastecimento

# \* \*\*SAA (Sistema de Abastecimento de Água):\*\* Instalação pública composta por captação, Estações de Tratamento de Água (ETAs), reservatórios e rede de distribuição operada por concessionária (em Santo André: \*\*SABESP / SEMASA\*\*). \*\*É a rede que abastece os domicílios e onde ocorre a fluoretação artificial controlada.\*\*

# \* \*\*SAC (Solução Alternativa Coletiva):\*\* Modalidade de suprimento privado compartilhado sem rede pública (ex: poços artesianos de shoppings, indústrias, condomínios fechados e hospitais).

# \* \*\*SAI (Solução Alternativa Individual):\*\* Poço artesiano ou cacimba em residência unifamiliar.

# 

# \### 4.2 Lição Aprendida no Diagnóstico Inicial

# O arquivo inicialmente analisado (`controle\_semestral\_2026.csv`) continha exclusivamente compostos complexos (como agrotóxicos \*Flutriafol\* e \*Trifluralina\* e metais pesados). A migração técnica para a base \*\*`controle\_mensal\_parametros\_basicos\_2026.csv`\*\* foi a chave para obter os registros reais de \*\*`Fluoreto (mg/L)`\*\*.

# 

# \---

# 

# \## 5. Dicionário Completo de Dados das Bases do SISAGUA

# 

# Abaixo consta a especificação de cada campo presente na base oficial de \*\*Controle Mensal – Parâmetros Básicos 2026\*\* (`controle\_mensal\_parametros\_basicos\_2026.csv`):

# 

# | Índice | Nome da Coluna no CSV | Tipo de Dado | Exemplo Real no Arquivo | Descrição e Regra de Negócio | Aplicação no Sistema Lumina |

# | :---: | :--- | :---: | :--- | :--- | :--- |

# | \*\*0\*\* | `Região Geográfica` | String | `SUDESTE` | Macrorregião do território brasileiro. | Filtro regional de alto nível. |

# | \*\*1\*\* | `UF` | String (2) | `SP` | Unidade da Federação. | Segmentação estadual. |

# | \*\*2\*\* | `Regional de Saúde` | String | `GVS VII - SANTO ANDRÉ` | Grupo de Vigilância Sanitária Regional do Estado de SP. | Governança epidemiológica regional. |

# | \*\*3\*\* | `Código IBGE` | Integer | `354780` | Código oficial IBGE de Santo André (6 dígitos). | \*\*Chave primária geográfica do município.\*\* |

# | \*\*4\*\* | `Município` | String | `SANTO ANDRE` | Nome por extenso do município. | Filtro textual do pipeline ETL. |

# | \*\*5\*\* | `Tipo da Instituição` | String | `Regional` / `Municipal` | Classificação da entidade gestora. | Metadados institucionais. |

# | \*\*6\*\* | `Sigla da Instituição` | String | `SABESP` | Concessionária responsável. | Validação da operadora oficial de água. |

# | \*\*7\*\* | `Nome da Instituição` | String | `COMPANHIA DE SANEAMENTO BASICO...` | Razão Social da concessionária ou entidade. | Rastreabilidade do prestador. |

# | \*\*8\*\* | `CNPJ da Instituição` | String | `43776517000180.0` | CNPJ matriz da instituição. | Metadado corporativo. |

# | \*\*9\*\* | `Nome do escritório regional/local`| String | `SAO PAULO SABESP` | Divisão operacional local da empresa. | Auditoria de polos de atendimento. |

# | \*\*10\*\* | `CNPJ do escritório regional/local`| String | `43776517064749.0` | CNPJ filial local. | Auditoria contábil/sanitária. |

# | \*\*11\*\* | `Tipo da Forma de Abastecimento` | String | `SAA` ou `SAC` | `SAA` = Rede Pública; `SAC` = Poço Privado. | \*\*Filtro Crítico:\*\* Isolar dados de `SAA` para os pacientes residenciais. |

# | \*\*12\*\* | `Código Forma de abastecimento` | String | `S354780000007` | Identificador único da solução no SISAGUA. | \*\*Chave estrangeira com tabela de Cadastro.\*\* |

# | \*\*13\*\* | `Nome da Forma de Abastecimento`| String | `OLHO DAGUA` / `GUSTAVO` | Nome da zona, poço ou reservatório. | Mapeamento do setor de abastecimento. |

# | \*\*14\*\* | `Nome da ETA / UTA` | String | `OLHO DAGUA` / `UTA GUSTAVO` | Nome da Estação de Tratamento de Água. | \*\*Chave para o De-Para de Bairros atendidos.\*\* |

# | \*\*15\*\* | `Tipo de Filtração` | String | `FILTRAÇÃO RÁPIDA` / `SEM FILTRAÇÃO` | Tecnologia da planta de tratamento. | Avaliação de complexidade operacional. |

# | \*\*16\*\* | `Ano de referência` | Integer | `2026` | Ano fiscal da medição. | Particionamento temporal. |

# | \*\*17\*\* | `Mês de referência` | Integer | `1` a `12` | Mês em que as análises laboratoriais ocorreram. | \*\*Série temporal para detecção de tendências.\*\* |

# | \*\*18\*\* | `Ponto de Monitoramento` | String | `SAÍDA DO TRATAMENTO`, `REDE DE DISTRIBUIÇÃO` | Local onde a amostra foi colhida na tubulação. | Validação da conformidade na torneira do consumidor. |

# | \*\*19\*\* | `Parâmetro` | String | `Fluoreto (mg/L)`, `Turbidez (uT)`, `pH` | Nome da grandeza física/química avaliada. | \*\*Filtro:\*\* Isolar `'Fluoreto (mg/L)'`. |

# | \*\*20\*\* | `Campo` | String | `Percentil 95`, `Número de amostras analisadas` | A métrica estatística representada na linha. | \*\*Filtro:\*\* Extrair o `Percentil 95` (concentração) e amostragem. |

# | \*\*21\*\* | `Valor` | String/Float| `0,8` / `0,08` / `169` | O valor medido no formato textual do governo. | \*\*Variável alvo do algoritmo de predição.\*\* |

# 

# \---

# 

# \## 6. Pipeline de Engenharia e Tratamento de Dados (ETL)

# 

# O processamento dos dados públicos é executado pelo Jupyter Notebook \[sto\_andre.ipynb](file:///c:/Users/lucas/OneDrive/Área%20de%20Trabalho/Projeto%20LUMINA/Base%20de%20Dados/controle\_semestral\_2026\_csv/sto\_andre.ipynb). Devido ao volume massivo das bases brutas nacionais (arquivos descompactados superiores a 2 GB e mais de 10 milhões de linhas), implementou-se uma arquitetura de extração em lotes.

# 

# \### 6.1 Estrutura do Código de Extração (Python / Pandas)

# 

# ```python

# import pandas as pd

# 

# \# ==============================================================================

# \# BLOCO 1: Extração do Controle Mensal (Parâmetros Básicos 2026)

# \# ==============================================================================

# print("\[ETL] Iniciando processamento do Controle Mensal 2026...")

# 

# pedacos\_mensal = pd.read\_csv(

# &#x20;   "controle\_mensal\_parametros\_basicos\_2026.csv",

# &#x20;   chunksize=50000,

# &#x20;   sep=";",

# &#x20;   encoding="latin-1",

# &#x20;   low\_memory=False

# )

# 

# \# Filtra Santo André pela 5ª coluna (índice 4 - Município)

# santo\_andre\_mensal = pd.concat(

# &#x20;   pedaco\[pedaco.iloc\[:, 4].astype(str).str.contains("SANTO ANDRE", na=False, case=False)]

# &#x20;   for pedaco in pedacos\_mensal

# )

# 

# \# Exporta base consolidada de Santo André

# santo\_andre\_mensal.to\_csv(

# &#x20;   "sisagua\_santo\_andre\_mensal\_parametros\_basicos\_2026.csv",

# &#x20;   index=False,

# &#x20;   sep=";",

# &#x20;   encoding="utf-8"

# )

# print(f"\[ETL] Base de Santo André gerada: {len(santo\_andre\_mensal)} linhas.")

# 

# \# ==============================================================================

# \# BLOCO 2: Segmentação Específica de Fluoreto para o Lumina

# \# ==============================================================================

# coluna\_parametro = santo\_andre\_mensal.columns\[19] # Índice 19: Parâmetro

# fluor\_df = santo\_andre\_mensal\[

# &#x20;   santo\_andre\_mensal\[coluna\_parametro].astype(str).str.contains("Flu", case=False, na=False)

# ].copy()

# 

# \# Tratamento numérico do campo de resultado (Vírgula para Ponto)

# coluna\_valor = santo\_andre\_mensal.columns\[21]     # Índice 21: Valor

# coluna\_campo = santo\_andre\_mensal.columns\[20]     # Índice 20: Campo

# 

# fluor\_df.to\_csv(

# &#x20;   "sisagua\_santo\_andre\_fluoreto\_2026.csv",

# &#x20;   index=False,

# &#x20;   sep=";",

# &#x20;   encoding="utf-8"

# )

# print(f"\[ETL] Base exclusiva de Fluoreto gerada: {len(fluor\_df)} registros.")

# ```

# 

# \### 6.2 Regras de Sanitização de Dados

# 1\. \*\*Normalização de Encodings:\*\* Leitura obrigatória em `latin-1` (padrão legado dos sistemas federais DATASUS) e conversão para `utf-8` para compatibilidade com o MySQL e Spring Boot.

# 2\. \*\*Tratamento Numérico Decimal:\*\* Substituição de `,` por `.` no campo `Valor` antes de cast para `DECIMAL(6,3)`.

# 3\. \*\*Tratamento de Dados Censurados:\*\* Mapeamento de `MENOR\_LD` (Limite de Detecção) e `MENOR\_LQ` (Limite de Quantificação) como $0.000\\text{ mg/L}$ para fins de análise de risco de sub-dosagem.

# 

# \---

# 

# \## 7. Modelo de Integração com o Backend Lumina (Spring Boot + MySQL + AWS SNS)

# 

# Para operacionalizar a ingestão no ecossistema de produção do Lumina, os dados sanitizados são persistidos no banco de dados relacional e vinculados aos pacientes.

# 

# \### 7.1 Esquema Físico de Banco de Dados (MySQL DDL)

# 

# ```sql

# \-- 1. Tabela de Medições Históricas do SISAGUA

# CREATE TABLE tbl\_qualidade\_agua\_sisagua (

# &#x20;   id\_medicao BIGINT AUTO\_INCREMENT PRIMARY KEY,

# &#x20;   codigo\_ibge INT NOT NULL,

# &#x20;   municipio VARCHAR(100) NOT NULL,

# &#x20;   tipo\_abastecimento ENUM('SAA', 'SAC', 'SAI') NOT NULL,

# &#x20;   codigo\_forma VARCHAR(50) NOT NULL,

# &#x20;   nome\_forma VARCHAR(150),

# &#x20;   nome\_eta VARCHAR(150),

# &#x20;   ano\_referencia INT NOT NULL,

# &#x20;   mes\_referencia INT NOT NULL,

# &#x20;   ponto\_monitoramento VARCHAR(100) NOT NULL,

# &#x20;   parametro VARCHAR(100) NOT NULL,

# &#x20;   campo\_metrica VARCHAR(100) NOT NULL,

# &#x20;   valor\_medido DECIMAL(8,4),

# &#x20;   data\_importacao TIMESTAMP DEFAULT CURRENT\_TIMESTAMP,

# &#x20;   INDEX idx\_busca\_preditiva (codigo\_ibge, ano\_referencia, mes\_referencia, parametro)

# ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

# 

# \-- 2. Tabela de De-Para: ETA / Zona de Abastecimento para Bairros

# CREATE TABLE tbl\_zona\_abastecimento\_bairro (

# &#x20;   id\_relacao BIGINT AUTO\_INCREMENT PRIMARY KEY,

# &#x20;   nome\_eta VARCHAR(150) NOT NULL,

# &#x20;   bairro VARCHAR(100) NOT NULL,

# &#x20;   municipio VARCHAR(100) NOT NULL,

# &#x20;   cep\_prefixo VARCHAR(5) NOT NULL, -- Ex: '09010', '09220'

# &#x20;   INDEX idx\_cep (cep\_prefixo)

# ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

# 

# \-- 3. Consulta de Cruzamento Preditivo para Disparo de Campanhas

# \-- Identifica pacientes em bairros atendidos por ETAs com Flúor < 0.6 mg/L no último mês

# SELECT DISTINCT 

# &#x20;   p.id\_paciente,

# &#x20;   p.nome,

# &#x20;   p.telefone,

# &#x20;   p.email,

# &#x20;   p.cep,

# &#x20;   p.bairro,

# &#x20;   med.nome\_eta,

# &#x20;   med.valor\_medido AS nivel\_fluor\_recente

# FROM tbl\_pacientes p

# JOIN tbl\_zona\_abastecimento\_bairro zb 

# &#x20;   ON LEFT(p.cep, 5) = zb.cep\_prefixo

# JOIN tbl\_qualidade\_agua\_sisagua med 

# &#x20;   ON med.nome\_eta = zb.nome\_eta

# WHERE med.parametro = 'Fluoreto (mg/L)'

# &#x20; AND med.campo\_metrica = 'Percentil 95'

# &#x20; AND med.ano\_referencia = 2026

# &#x20; AND med.mes\_referencia = 6

# &#x20; AND med.valor\_medido < 0.60

# &#x20; AND p.status\_ativo = TRUE;

# ```

# 

# \### 7.2 Fluxo de Execução no Spring Boot (Régua de Relacionamento)

# 

# ```java

# @Service

# public class ReguaSaudePreditivaService {

# 

# &#x20;   @Autowired

# &#x20;   private QualidadeAguaRepository aguaRepository;

# 

# &#x20;   @Autowired

# &#x20;   private AmazonSNSClient snsClient;

# 

# &#x20;   @Scheduled(cron = "0 0 9 15 \* ?") // Executa no dia 15 de cada mês

# &#x20;   public void dispararCampanhaFluorPrevenção() {

# &#x20;       List<PacienteRiscoFluorDTO> pacientes = aguaRepository.buscarPacientesEmZonasSubOptimas();

# 

# &#x20;       for (PacienteRiscoFluorDTO paciente : pacientes) {

# &#x20;           String mensagem = String.format(

# &#x20;               "Olá %s, notamos que a sua região em %s registrou oscilações na fluoretação da água no último mês (%.2f mg/L). " +

# &#x20;               "Para proteger seu esmalte dentário contra cáries, recomendamos uma aplicação preventiva de flúor na Clínica Lumina. " +

# &#x20;               "Agende agora com condições especiais!",

# &#x20;               paciente.getNome(), paciente.getBairro(), paciente.getNivelFluor()

# &#x20;           );

# 

# &#x20;           PublishRequest request = new PublishRequest()

# &#x20;               .withPhoneNumber(paciente.getTelefone())

# &#x20;               .withMessage(mensagem);

# 

# &#x20;           snsClient.publish(request);

# &#x20;       }

# &#x20;   }

# }

# ```

# 

# \---

# 

# \## 8. Inventário e Mapeamento de Arquivos no Repositório

# 

# Estrutura de diretórios presente em `Base de Dados/`:

# 

# ```

# c:\\Users\\lucas\\OneDrive\\Área de Trabalho\\Projeto LUMINA\\Base de Dados\\

# │

# ├── MEMORIA\_TECNICA\_SISAGUA\_LUMINA.md                   <-- Esta documentação técnica

# │

# ├── Arquivos SISAGUA/                                   <-- Arquivos brutos baixados

# │   ├── cadastro\_populacao\_abastecida.csv               <-- Histórico anual demográfico (2014-2026)

# │   ├── controle\_mensal\_plano\_amostragem\_2023.json      <-- Regras de metas de coletas

# │   └── SISAGUA - Controle Mensal...Dicionário.pdf      <-- Manual oficial do Ministério da Saúde

# │

# └── controle\_semestral\_2026\_csv/                        <-- Diretório de scripts e bases tratadas

# &#x20;   ├── sto\_andre.ipynb                                 <-- Jupyter Notebook com o Pipeline de ETL

# &#x20;   ├── controle\_semestral\_2026.csv                     <-- Base bruta nacional semestral (\~504 MB)

# &#x20;   ├── sisagua\_santo\_andre\_2026.csv                    <-- Base filtrada semestral de Sto André (\~1.6 MB)

# &#x20;   ├── sisagua\_santo\_andre\_mensal\_parametros\_basicos\_2026.csv <-- Base filtrada mensal completa (\~3.2 MB)

# &#x20;   └── sisagua\_santo\_andre\_fluoreto\_2026.csv           <-- Dataset refinado de Flúor para o Lumina

# ```

# 

# \---

# 

# \## 9. Referências Oficiais, Legislação e Links de Acesso

# 

# 1\. \*\*Ministério da Saúde (Brasil):\*\*

# &#x20;  \* \*Portaria GM/MS nº 888, de 4 de maio de 2021.\* Altera o Anexo XX da Portaria de Consolidação GM/MS nº 5/2017 para estabelecer os procedimentos e responsabilidades relativos ao controle e vigilância da qualidade da água para consumo humano e seu padrão de potabilidade.

# &#x20;  \* \*Caderno de Diretrizes da Vigilância da Qualidade da Água para Consumo Humano.\* Programa Nacional VIGIAGUA.

# 2\. \*\*Repositórios Oficiais de Dados Abertos (SUS / CKAN):\*\*

# &#x20;  \* \*\*Controle Mensal (Parâmetros Básicos 2026):\*\*  

# &#x20;    `https://s3.sa-east-1.amazonaws.com/ckan.saude.gov.br/SISAGUA/controle\_mensal\_parametros\_basicos\_2026\_csv.zip`

# &#x20;  \* \*\*Controle Mensal (Parâmetros Básicos 2025):\*\*  

# &#x20;    `https://s3.sa-east-1.amazonaws.com/ckan.saude.gov.br/SISAGUA/controle\_mensal\_parametros\_basicos\_2025\_csv.zip`

# &#x20;  \* \*\*Catálogo de Dados Abertos:\*\*  

# &#x20;    \[Portal Brasileiro de Dados Abertos - Conjunto SISAGUA](https://dadosabertos.saude.gov.br/dataset/sisagua-controle-mensal-parametros-basicos)

# 3\. \*\*Literatura Científica Odontológica:\*\*

# &#x20;  \* CECOL/USP (Centro Colaborador do Ministério da Saúde em Vigilância da Saúde Bucal): \*Consenso Técnico sobre Classificação de Águas de Abastecimento Público segundo o Teor de Flúor\*.

# &#x20;  \* World Health Organization (WHO): \*Fluorides and oral health: report of a WHO Expert Committee on Oral Health Status and Fluoride Use\*. Technical Report Series, Geneva.

# &#x20;  \* Fejerskov, O., \& Kidd, E. (2008). \*Cárie Dentária: A Evolução do Conceito\*. Ed. Santos.



