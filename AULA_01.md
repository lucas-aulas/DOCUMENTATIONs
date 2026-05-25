<h1 align="center">1. Preparação do Ambiente</h1>

### Ativar ambiente virtual

Linux/macOS
```sh
python3.14 -m venv .venv

source .venv/bin/activate

pip install --upgrade pip

pip install -r requirements.txt
```

Windows (CMD)
```bash
python -m venv .venv

.venv\Scripts\activate.bat

python -m pip install -r requirements.txt
```

Windows (PowerShell) *Recomendado
```bash
python -m venv .venv

Set-ExecutionPolicy RemoteSigned -Scope CurrentUser

.venv\Scripts\Activate.ps1

python -m pip install -r requirements.txt
```

> [!NOTE]
> Instalação separada (sem usar `requirements.txt`)
> ```py
> pip install db-dtypes google-auth google-cloud-bigquery igmapper ipykernel pandas-gbq pandas selenium tmdbsimple
> ```

> [!WARNING]
> Erro `ModuleNotFoundError` → Dependência não instalada no ambiente virtual.
> ```py
> ------------------------------------------------------------
> ModuleNotFoundError        Traceback (most recent call last)
> Cell In[1], line 1
> ----> 1 import BIBLIOTECA_X
>
> ModuleNotFoundError: No module named 'BIBLIOTECA_X'
> ```
>
> Resolva:
> ```py
> pip install BIBLIOTECA_X
> ```

> [!TIP]
> Limpar tudo do Ambiente Virtual
> 
> Limpar Cache do PIP
> ```py
> pip cache purge
> ```
> Desinstalar tudo que já foi instalado no VENV
> ```py
> # Linux
> pip freeze > req_installed.txt && pip uninstall -r req_installed.txt -y && pip cache purge
> 
> # Windows (PowerShell)
> pip freeze > req_installed.txt; pip uninstall -r req_installed.txt -y; pip cache purge
> ```


### VSCode Extensions
- Black Formatter (Microsoft)
- Jupyter (Microsoft)
- Python (Microsoft)
- Pylance (Microsoft)
- Pylint (Microsoft)

<hr>

### Imports

```py
from google.cloud import bigquery
from google.oauth2 import service_account

from selenium import webdriver
from selenium.webdriver.common.by import By

import pandas as pd
import tmdbsimple as tmdb
import os
```

### Configuração do Pandas

```py
pd.set_option("display.max_columns", None)
pd.set_option("display.max_rows", None)
```

### Definição das Fontes de Dados

#### GitHub

```py
dataset_movies_url = "https://media.githubusercontent.com/media/lucas-aulas/dataset-movies/refs/heads/main"
```

#### GCP

```py
gcp_project_id = "project202605-496913"
```

<h1 align="center">2. Configuração do BigQuery</h1>


### Criar a Service Account

`IAM e admin` → `Contas de Serviço` → `+ Criar conta de serviço` → `... Gerenciar chave` → `Adicionar chave` → `Criar nova chave` → `JSON` → SALVAR EM "secrets/"

### Adicionar os papéis:

`Clicar na chave` → `Permissões` → `Gerenciar acesso` → `Atribuir papéis`:

- BigQuery Data Editor → Acesso para editar todos os tipos de conteúdo de conjuntos de dados
- BigQuery Job User → Acesso para executar jobs

### 1. Definir as Credenciais

```py
my_gcp_sa_file = "../secrets/_____.json"

my_gcp_cred = service_account.Credentials.from_service_account_file(my_gcp_sa_file)
```

### 2. Criar Cliente do BigQuery

```py
my_gcp_client = bigquery.Client(credentials=my_gcp_cred)
```

ou

```py
my_gcp_client = bigquery.Client(credentials=my_gcp_cred, project=gcp_project_id)
```

<h1 align="center">3. Ingestão de Dados (EXTRACT)</h1>

### GitHub

```py
df = pd.read_csv(f"{dataset_movies_url}/archive.csv")
```

### BigQuery

```py
df = my_gcp_client.query(
    f"""
    SELECT *
    FROM {gcp_project_id}.dataset.table
    """
).to_dataframe(create_bqstorage_client=False)
```

<h1 align="center">4. Envio de Dados para o BigQuery (LOAD)</h1>

### load_table_from_dataframe

```py
my_gcp_client.load_table_from_dataframe(df, "dataset.table_name")
```

### to_gbq

```py
df.to_gbq(
    project_id="_____",
    destination_table="dataset.table_name",
    if_exists="replace",
    credentials=my_gcp_cred,
)
```

<br>
<br>
<hr>
<hr>
<br>
<br>

<h1 align="center">The Movie Database (TMDB)</h1>

### Conta + Chave da API

| | |
|-|-|
1º Criar conta            | https://themoviedb.org/signup
2º Solicitar chave da API | https://themoviedb.org/settings/api/request
3º Configurações da API   | https://themoviedb.org/settings/api

### Configuração da API

```py
tmdb_api_key = "_____"
tmdb.API_KEY = tmdb_api_key
```

### Filme

```py
tmdb_movies = tmdb.Search().movie(query="Spider-Man 2")

pd.DataFrame(data=tmdb_movies.get("results"))
```

### Elenco

```py
star_id = tmdb.Search().person(query="Tony Ramos").get("results")[0].get("id")
```

### Redes Sociais

```py
tmdb.People(star_id).external_ids().get("instagram_id")
```

<h1 align="center">Igmapper</h1>

### Configuração do Client

```py
import igmapper

client = igmapper.InstaClient(
    csrftoken=None,
    ds_user_id=None,
    sessionid=None,
    use_curl=True
)
```

### Perfil

```py
client.get_profile_info("therock")
```

### Feed

```py
client.get_feed("therock")
```

<br>
<br>
<hr>
<hr>
<br>
<br>
