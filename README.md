# Tempo de taxi do aeroporto de São Paulo/Congonhas (SBSP-CGH)

**Em desenvolvimento** · TCC da pós-graduação em Data Science and Analytics (USP)

*[English version below](#taxi-time-at-são-paulocongonhas-airport-sbsp-cgh)*

---

## Sobre o projeto

Modela o taxi-out — do início do movimento no solo até a decolagem — em função
de calendário (mês, dia da semana, bloco de horário, feriado) e meteorologia em
2025. O dataset cruza três fontes públicas: ADS-B da OpenSky Network, VRA da
ANAC e METAR de SBSP fornecido pela REDEMET.

---

## Dados: como foram extraídos e por que não estão neste repositório

O estudo cruza três fontes públicas independentes. **Nenhuma das bases está
versionada aqui** — por restrição de licença, por tamanho de arquivo ou por
conter credencial pessoal. Todos os notebooks de coleta estão no repositório, de
modo que as bases podem ser regeradas do zero.

### 1. Trajetórias ADS-B — OpenSky Network

<https://opensky-network.org> · Doc. Trino:
<https://openskynetwork.github.io/opensky-api/trino.html>

As decolagens de SBSP em 2025 foram baixadas da tabela `flights_data4` pela
interface SQL Trino, com a biblioteca `pyopensky`, que injeta automaticamente o
filtro de partição exigido pelo servidor (consulta sem ele é rejeitada). O
download é feito mês a mês, gerando um parquet por mês. Dessa base vem o campo
`firstseen`, usado como proxy do horário real de decolagem (ATOT). O acesso
histórico é gratuito para pesquisadores com vínculo universitário, mediante
solicitação com e-mail institucional.

**Por que não publiquei:** o Data License Agreement da OpenSky concede licença
**não transferível** e proíbe expressamente distribuir ou disponibilizar os dados
a terceiros (Seção 3, iii) — cada pessoa deve solicitar acesso diretamente à
OpenSky. O acordo também exige anonimizar identificadores de aeronave (`icao24`,
`callsign`) em qualquer divulgação. Por isso ficam de fora tanto os arquivos
brutos quanto **todos os derivados por voo**.

### 2. Horários de partida real — VRA / ANAC

[Portal de dados abertos](https://www.gov.br/anac/pt-br/acesso-a-informacao/dados-abertos/areas-de-atuacao/voos-e-operacoes-aereas/voo-regular-ativo-vra/62-voo-regular-ativo-vra)
· Arquivos: <https://sistemas.anac.gov.br/dadosabertos/>

Os CSVs mensais do Voo Regular Ativo foram baixados do portal de dados abertos
da ANAC, sem cadastro. Deles vem a coluna **Partida Real** (calços fora, horário
de Brasília), que marca o início do taxi-out. O join com o ADS-B é feito por
callsign e data.

**Por que não publiquei:** são dados abertos e redistribuíveis, mas os CSVs somam
**268 MB** — o GitHub bloqueou o envio. O download é direto e gratuito.

### 3. Meteorologia — METAR / REDEMET (DECEA)

<https://redemet.decea.mil.br> · API: <https://api-redemet.decea.mil.br>

As mensagens METAR de SBSP em 2025 foram obtidas pela API da REDEMET, endpoint
`mensagens/metar/SBSP`, com `data_ini=2025010101` e `data_fim=2025123124`
(formato AAAAMMDDHH, em UTC) e paginação de 150 registros por página, iterando
até `last_page`. O texto METAR bruto foi decodificado individualmente com
`metpy.io.parse_metar_to_dataframe` — como o METAR só traz dia e hora, o ano e o
mês vêm do campo `validade_inicial` devolvido pela API. A coluna `altimeter_hpa`
é derivada na sequência, convertendo o altímetro de polegadas de mercúrio para
hectopascais. Resultado: 11.447 observações, 32 colunas, `date_time` em UTC. A
API exige cadastro gratuito e chave de acesso.

**Por que não publiquei:** os notebooks de coleta da REDEMET contêm minha chave
pessoal de API, que é individual e não pode ser compartilhada.

### Diretórios criados

| Diretório | Conteúdo | No repositório? |
|---|---|---|
| `data/flightlist/` | `fl_AAAAMM.parquet` — partidas de SBSP por mês, da OpenSky (12 arquivos, 3,3 MB) | Não — licença |
| `data/vra/` | `VRA_AAAAMM.csv` — VRA mensal da ANAC (268 MB) | Não — tamanho |

### Como reproduzir

Obtenha suas credenciais da OpenSky e da REDEMET, baixe os CSVs do VRA e rode os
notebooks na ordem numérica. As saídas intermediárias são gravadas em
`data/processed/`.

### Citação obrigatória (OpenSky)

> Schäfer, M.; Strohmeier, M.; Lenders, V.; Martinovic, I.; Wilhelm, M.
> *Bringing up OpenSky: A large-scale ADS-B sensor network for research.*
> ACM/IEEE International Conference on Information Processing in Sensor
> Networks (IPSN), abril de 2014.

---
---

# Taxi time at São Paulo/Congonhas Airport (SBSP-CGH)

**Work in progress** · Graduate capstone in Data Science and Analytics (USP)

## About

Models taxi-out — from the start of ground movement to takeoff — as a function
of calendar (month, weekday, time-of-day block, holidays) and weather in 2025.
The dataset joins three public sources: ADS-B from the OpenSky Network, ANAC's
VRA records, and METAR for SBSP provided by REDEMET.

## Data: how it was extracted and why it is not in this repository

The study joins three independent public sources. **None of the datasets are
versioned here** — because of licensing, file size, or personal credentials. All
collection notebooks are included, so the datasets can be rebuilt from scratch.

### 1. ADS-B trajectories — OpenSky Network

<https://opensky-network.org> · Trino docs:
<https://openskynetwork.github.io/opensky-api/trino.html>

Departures from SBSP in 2025 were downloaded from the `flights_data4` table
through the Trino SQL interface, using the `pyopensky` library, which
automatically injects the partition filter the server requires (queries without
it are rejected). Downloads run month by month, producing one parquet per month.
This source provides the `firstseen` field, used as a proxy for actual takeoff
time (ATOT). Historical access is free for researchers with a university
affiliation, upon request from an institutional email address.

**Why I did not publish it:** OpenSky's Data License Agreement grants a
**non-transferable** license and expressly forbids distributing or otherwise
making the data available to third parties (Section 3, iii) — each person must
request access directly from OpenSky. The agreement also requires anonymizing
aircraft identifiers (`icao24`, `callsign`) in any disclosure. Both the raw files
and **all per-flight derivatives** are therefore excluded.

### 2. Actual off-block times — VRA / ANAC

[Open data portal](https://www.gov.br/anac/pt-br/acesso-a-informacao/dados-abertos/areas-de-atuacao/voos-e-operacoes-aereas/voo-regular-ativo-vra/62-voo-regular-ativo-vra)
· Files: <https://sistemas.anac.gov.br/dadosabertos/>

Monthly CSVs of the Active Scheduled Flight record were downloaded from ANAC's
open data portal, no registration needed. They provide the **Partida Real**
column (actual off-block time, Brasília time), which marks the start of
taxi-out. The join with ADS-B is done on callsign and date.

**Why I did not publish it:** the data are open and redistributable, but the
CSVs total **268 MB** — GitHub rejected the upload. The download is direct and
free.

### 3. Weather — METAR / REDEMET (DECEA)

<https://redemet.decea.mil.br> · API: <https://api-redemet.decea.mil.br>

METAR messages for SBSP in 2025 were retrieved from the REDEMET API, endpoint
`mensagens/metar/SBSP`, with `data_ini=2025010101` and `data_fim=2025123124`
(YYYYMMDDHH format, UTC) and pagination of 150 records per page, iterating to
`last_page`. Raw METAR text was decoded message by message with
`metpy.io.parse_metar_to_dataframe` — since METAR carries only day and hour,
year and month come from the `validade_inicial` field returned by the API. The
`altimeter_hpa` column is derived afterwards, converting the altimeter setting
from inches of mercury to hectopascals. Result: 11,447 observations, 32 columns,
`date_time` in UTC. The API requires free registration and an access key.

**Why I did not publish it:** the REDEMET collection notebooks contain my
personal API key, which is individual and cannot be shared.

### Directories created

| Directory | Contents | In the repo? |
|---|---|---|
| `data/flightlist/` | `fl_YYYYMM.parquet` — SBSP departures by month, from OpenSky (12 files, 3.3 MB) | No — license |
| `data/vra/` | `VRA_YYYYMM.csv` — monthly ANAC VRA records (268 MB) | No — size |

### To reproduce

Obtain your own OpenSky and REDEMET credentials, download the VRA CSVs, and run
the notebooks in numeric order. Intermediate outputs are written to
`data/processed/`.

### Required citation (OpenSky)

> Schäfer, M.; Strohmeier, M.; Lenders, V.; Martinovic, I.; Wilhelm, M.
> *Bringing up OpenSky: A large-scale ADS-B sensor network for research.*
> ACM/IEEE International Conference on Information Processing in Sensor
> Networks (IPSN), April 2014.
