<div align="center">
    <hr>
    <h3>
        Engenharia de Dados <br> da Ingestão à modelagem <br> Star Schema 
        com Python e SQL <br>
    </h2>
    <h4>
        Faturamento nos cinemas • DC vs Marvel
    </h4>
    <hr>
</div>


- **Sumário**
    - [1. PREPARAÇÃO DO AMBIENTE](#1-preparação-do-ambiente)
    - [2. CONFIGURAÇÃO DO BIGQUERY](#3-código--ingestão)
    - [3. CÓDIGO · Ingestão](#3-código--ingestão)
    - [4. CÓDIGO · Tratamento](#4-código--tratamento)
    - [5. CÓDIGO · Star Schema](#5-código--star-schema)
    - [Extras](#extras)

<div align="center"> 

## 1. PREPARAÇÃO DO AMBIENTE
</div>
<details><summary>ℹ️ Detalhes...</summary>

### `requirements.txt`

São as bibliotecas Python que o projeto vai usar.

```py
db-dtypes             # Tipos de dados extras usados pelo BigQuery no pandas (DATE, TIME, STRUCT etc.)
google-auth           # Autenticação e credenciais para acessar APIs e serviços Google
google-cloud-bigquery # Cliente oficial Python para consultar e manipular dados no BigQuery
igmapper              # Biblioteca para mapeamento/geolocalização e visualização de dados geográficos
ipykernel             # Kernel do Python para execução em notebooks Jupyter
pandas-gbq            # Integração entre pandas e BigQuery para ler/escrever tabelas facilmente
pandas                # Manipulação e análise de dados em DataFrames
selenium              # Automação de navegador para scraping e testes web
tmdbsimple            # Cliente Python simples para consumir a API do The Movie Database (TMDB)
```

### Ativar ambiente virtual (`venv`)

**O que é?** É um ambiente virtual do Python que cria um espaço isolado para instalar bibliotecas de um projeto sem afetar os outros projetos ou o Python do sistema.

**Linux/macOS**

1. Cria um ambiente virtual Python na pasta `.venv`
    ```sh
    python3.14 -m venv .venv
    ```
2. Ativa o ambiente virtual
    ```sh
    source .venv/bin/activate
    ```
3. Atualiza o pip para a versão mais recente
    ```sh
    pip install --upgrade pip
    ```
4. Instala todas as dependências do projeto
    ```sh
    pip install -r requirements.txt
    ```

**Windows (PowerShell)**

1. Cria um ambiente virtual Python na pasta `.venv`
    ```ps
    python -m venv .venv
    ```
2. Permite executar scripts locais no PowerShell
    ```ps
    Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
    ```
3. Ativa o ambiente virtual no PowerShell
    ```ps
    .venv\Scripts\Activate.ps1
    ```
4. Instala todas as dependências do projeto
    ```ps
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
</details>

<div align="center"> 

## 2. CONFIGURAÇÃO DO BIGQUERY
</div>
<details><summary>ℹ️ Detalhes...</summary>

### Criar Projeto no GCP

- ### https://console.cloud.google.com

> <div align="center">
>     <img width="50%" src="assets/image.png">
>     <img width="50%" src="assets/image-1.png">
>     <img width="50%" src="assets/image-2.png">
>     <img width="50%" src="assets/image-3.png">
>     <img width="50%" src="assets/image-4.png">
> </div>

### Criar a Service Account

Cria uma conta de serviço para autenticar o acesso ao BigQuery.

#### Adicionar os papéis:

- **BigQuery Data Editor** → Acesso para editar todos os tipos de conteúdo de conjuntos de dados
- **BigQuery Job User** → Acesso para executar jobs

> <div align="center">
>     <img width="20%" src="assets/image-5.png"> <br>
>     <img width="50%" src="assets/image-6.png">
>     <img width="50%" src="assets/image-7.png">
>     <img width="50%" src="assets/image-8.png">
> </div>

#### Baixar chave:
> <div align="center">
>     <img width="50%" src="assets/image-9.png">
>     <img width="50%" src="assets/image-10.png">
>     <img width="50%" src="assets/image-11.png"> <br>
>     <img width="20%" src="assets/image-12.png">
> </div>

</details>

<div align="center"> 

## 3. CÓDIGO · Ingestão
</div>
<details><summary>ℹ️ Detalhes...</summary>

1. Import e definição das view com pandas
    ```py
    from google.cloud import bigquery
    from google.oauth2 import service_account
    import pandas as pd

    pd.set_option("display.max_columns", None)
    pd.set_option("display.max_rows", 10)
    ``` 
2. Definição das variáveis utilizadas para ingestão de dados
    ```py
    dataset_movies_url = "https://media.githubusercontent.com/media/lucas-aulas/dataset-movies/refs/heads/main"

    gcp_project_id_aula = "project202605-496913"

    gcp_service_acc_file_aluno = "../secrets/____________________.json"
    gcp_id_do_projeto = "____________________"
    my_gcp_cred = service_account.Credentials.from_service_account_file(gcp_service_acc_file_aluno)
    my_gcp_client = bigquery.Client(credentials=my_gcp_cred)
    ```
3. Leitura dos dados da fonte externa - GitHub
    ```py
    df_imdb_filmes = pd.read_csv(f"{dataset_movies_url}/imdb_filmes.csv")
    ```
4. Leitura dos dados da fonte externa - BigQuery
    ```py
    df_boxoffice_dc_marvel = my_gcp_client.query(
        f"SELECT * FROM {gcp_project_id_aula}.raw.boxoffice_dc_marvel"
    ).to_dataframe(create_bqstorage_client=False)
    ```
5. Envio para o seu banco de dados
    ```py
    df_imdb_filmes.to_gbq(
        project_id=gcp_id_do_projeto,
        destination_table="bronze.imdb_filmes",
        if_exists="replace",
        credentials=my_gcp_cred,
    )

    df_boxoffice_dc_marvel.to_gbq(
        project_id=gcp_id_do_projeto,
        destination_table="bronze.boxoffice_dc_marvel",
        if_exists="replace",
        credentials=my_gcp_cred,
    )
    ```
</details>

<div align="center"> 

## 4. CÓDIGO · Tratamento
</div>
<details><summary>ℹ️ Detalhes...</summary>

1. Bibliotecas que vão ser utilizadas
    ```py
    from google.cloud import bigquery
    from google.oauth2 import service_account
    import pandas as pd

    pd.set_option("display.max_columns", None)
    pd.set_option("display.max_rows", 10)
    ```
2. Variáveis e informações para a ingestão
    ```py
    gcp_service_acc_file_aluno = "../secrets/____________________.json"
    gcp_id_do_projeto = "____________________"
    my_gcp_cred = service_account.Credentials.from_service_account_file(gcp_service_acc_file_aluno)
    my_gcp_client = bigquery.Client(credentials=my_gcp_cred)
    ```
3. Leitura da tabela `boxoffice_dc_marvel` no BigQuery
    ```py
    df_boxoffice_dc_marvel = my_gcp_client.query(
        "SELECT * FROM bronze.boxoffice_dc_marvel"
    ).to_dataframe(create_bqstorage_client=False)
    ```
4. Leitura da tabela `imdb_filmes` no BigQuery
    ```py
    df_imdb_filmes = my_gcp_client.query(
        "SELECT * FROM bronze.imdb_filmes"
    ).to_dataframe(create_bqstorage_client=False)
    ```
5. Merge dos DataFrames
    ```py
    df_complete_movie_info = (
        pd.merge(
            df_boxoffice_dc_marvel,
            df_imdb_filmes,
            left_on=["FILM", "YEAR"],
            right_on=["TITLE", "YEAR"],
            how="outer"
        )
        .sort_values(by="YEAR", ascending=False)
        .drop_duplicates(keep="last")
    )
    ```
6. Filtrando apenas filmes da Marvel e DC
    ```py
    df_complete_dc_marvel = df_complete_movie_info[
        (df_complete_movie_info["FRANCHISE"] == "Marvel") |
        (df_complete_movie_info["FRANCHISE"] == "DC")
    ]
    ```
7. Removendo colunas desnecessárias
    ```py
    df_complete_dc_marvel = df_complete_dc_marvel.drop(
        columns=[
            "25X_PROD",
            "BOX_OFFICE_GROSS_DOMESTIC_US_AND_CANADA",
            "BOX_OFFICE_GROSS_WORLDWIDE",
            "BUDGET_y",
            "DURATION",
            "LENGTH",
            "MPAA_RATING",
            "TITLE",
            "YEAR",
        ]
    ).reset_index(drop=True)
    ```
8. Visualizando os filmes com maior nota IMDB
    ```py
    df_complete_dc_marvel.sort_values(by="RATING_IMDB", ascending=False).head(2)
    ```
9. Tratamento das colunas monetárias, percentuais, datas e inteiros
    ```py
    df_tratado = df_complete_dc_marvel.copy()
    ```
    ```py
    money_cols = [
        "BOX_OFFICE_GROSS_OTHER_TERRITORIES",
        "BUDGET_x",
        "INFLATION_ADJUSTED_BUDGET",
        "INFLATION_ADJUSTED_WORLDWIDE_GROSS",
        "GROSS_OPENING_WEEKEND",
        "GROSS_US_CANADA",
        "GROSS_WORLD_WIDE",
    ]

    for col in money_cols:
        df_tratado[col] = df_tratado[col].astype(str).str.replace("$", "").str.replace(",", "").replace("None", 0).astype(float)
    ```
    ```py
    percent_cols = ["DOMESTIC"]

    for col in percent_cols:
        df_tratado[col] = df_tratado[col].astype(str).str.replace("%", "").astype(float)
    ```
    ```py
    date_cols = ["US_RELEASE_DATE"]

    for col in date_cols:
        df_tratado[col] = pd.to_datetime(df_tratado[col], format="%d/%m/%Y")
    ```
    ```py
    int_cols = [
        "VOTE",
        "WIN",
        "OSCAR",
        "NOMINATION",
        "ROTTEN_TOMATOES_CRITIC_SCORE"
    ]

    for col in int_cols:
        df_tratado[col] = df_tratado[col].fillna(0).astype(int)
    ```
10. Criando a coluna de ano baseada na data de lançamento
    ```py
    df_tratado["YEAR"] = df_tratado["US_RELEASE_DATE"].dt.year
    ```
11. Padronizando colunas categóricas
    ```py
    df_tratado["GENRE"] = df_tratado["GENRE"].str.split(",").str[0].str.strip()

    df_tratado["COUNTRY_ORIGIN"] = df_tratado["COUNTRY_ORIGIN"].str.split(",").str[0].str.strip()

    df_tratado["LANGUAGE"] = df_tratado["LANGUAGE"].str.split(",").str[0].str.strip()

    df_tratado["PRODUCTION_COMPANY"] = df_tratado["PRODUCTION_COMPANY"].str.split(",").str[0].str.strip()
    ```
12. Organização final das colunas
    ```py
    df_tratado = df_tratado.drop(columns=["Unnamed: 0"], errors="ignore")

    df_tratado = df_tratado[sorted(df_tratado.columns)]
    ```
13. Salvando a tabela tratada na camada Silver
    ```py
    df_tratado.to_gbq(
        project_id=gcp_id_do_projeto,
        destination_table="silver.complete_dc_marvel",
        if_exists="replace",
        credentials=my_gcp_cred,
    )
    ```
</details>

<div align="center"> 

## 5. CÓDIGO · Star Schema
</div>
<details><summary>ℹ️ Detalhes...</summary>

1. Importação das bibliotecas necessárias para conexão com o BigQuery e manipulação dos dados com Pandas.
    ```py
    from google.cloud import bigquery
    from google.oauth2 import service_account
    import pandas as pd

    pd.set_option("display.max_columns", None)
    pd.set_option("display.max_rows", 10)
    ```
2. Configuração das credenciais e criação do cliente de conexão com o Google BigQuery.
    ```py
    gcp_service_acc_file_aluno = "../secrets/____________________.json"
    gcp_id_do_projeto = "____________________"
    my_gcp_cred = service_account.Credentials.from_service_account_file(gcp_service_acc_file_aluno)
    my_gcp_client = bigquery.Client(credentials=my_gcp_cred)
    ```
3. Leitura da tabela Silver armazenada no BigQuery para criação da camada Gold.
    ```py
    df_silver = my_gcp_client.query(
        f"SELECT * FROM silver.complete_dc_marvel",
    ).to_dataframe(create_bqstorage_client=False)
    ```
4. Criação da dimensão de filmes contendo atributos descritivos de cada produção.
    ```py
    dim_film = (
        df_silver[["FILM", "GENRE", "RATING_MPA", "LANGUAGE", "BREAK_EVEN", "MCU", "MALE_FEMALE_LED", "PHASE"]]
        .drop_duplicates()
        .sort_values(by="FILM")
        .reset_index(drop=True)
    )

    dim_film["ID_FILM"] = dim_film.index + 1

    dim_film.head(5)
    ```
5. Criação da dimensão de equipe técnica contendo diretor, roteirista e ator principal.
    ```py
    dim_staff = (
        df_silver[["DIRECTOR", "WRITER", "STAR"]]
        .drop_duplicates()
        .sort_values(by=["DIRECTOR", "WRITER"])
        .reset_index(drop=True)
    )

    dim_staff["ID_STAFF"] = dim_staff.index + 1

    dim_staff.head(2)
    ```
6. Criação da dimensão de franquias e famílias de personagens dos filmes.
    ```py
    dim_franchise = (
        df_silver[["FRANCHISE", "CHARACTER_FAMILY"]]
        .drop_duplicates()
        .sort_values(by=["FRANCHISE", "CHARACTER_FAMILY"])
        .reset_index(drop=True)
    )

    dim_franchise["ID_FRANCHISE"] = dim_franchise.index + 1

    dim_franchise.head(2)
    ```
7. Criação da dimensão de produção contendo empresa produtora, distribuidora e localização das filmagens.
    ```py
    dim_production = (
        df_silver[["PRODUCTION_COMPANY", "DISTRIBUTOR", "COUNTRY_ORIGIN", "FILMING_LOCATION"]]
        .drop_duplicates()
        .sort_values(by=["PRODUCTION_COMPANY", "DISTRIBUTOR", "COUNTRY_ORIGIN", "FILMING_LOCATION"])
        .reset_index(drop=True)
    )

    dim_production["ID_PRODUCTION"] = dim_production.index + 1

    dim_production.head(2)
    ```
8. Criação da dimensão de datas com informações de ano, mês e trimestre de lançamento.
    ```py
    dim_date = df_silver[["US_RELEASE_DATE"]].dropna().drop_duplicates().sort_values(by="US_RELEASE_DATE").reset_index(drop=True)

    dim_date["YEAR"] = dim_date["US_RELEASE_DATE"].dt.year
    dim_date["MONTH"] = dim_date["US_RELEASE_DATE"].dt.month
    dim_date["QUARTER"] = dim_date["US_RELEASE_DATE"].dt.quarter

    dim_date["ID_DATE"] = dim_date["US_RELEASE_DATE"].dt.strftime("%Y%m%d").astype(int)

    dim_date.head(2)
    ```
9. Criação inicial da tabela fato a partir dos dados completos da camada Silver.
    ```py
    fact_movies = df_silver.copy()
    ```
10. Associação da dimensão de filmes à tabela fato por meio das chaves correspondentes.
    ```py
    fact_movies = fact_movies.merge(
        dim_film,
        on=["FILM", "GENRE", "RATING_MPA", "LANGUAGE", "BREAK_EVEN", "MCU", "MALE_FEMALE_LED", "PHASE"],
        how="left",
    )
    ```
11. Associação da dimensão de equipe técnica à tabela fato.
    ```py
    fact_movies = fact_movies.merge(
        dim_staff,
        on=["DIRECTOR", "WRITER", "STAR"],
        how="left",
    )
    ```
12. Associação da dimensão de franquias à tabela fato.
    ```py
    fact_movies = fact_movies.merge(
        dim_franchise,
        on=["FRANCHISE", "CHARACTER_FAMILY"],
        how="left",
    )
    ```
13. Associação da dimensão de produção à tabela fato.
    ```py
    fact_movies = fact_movies.merge(
        dim_production,
        on=["PRODUCTION_COMPANY", "DISTRIBUTOR", "COUNTRY_ORIGIN", "FILMING_LOCATION"],
        how="left",
    )
    ```
14. Associação da dimensão de datas à tabela fato utilizando a data de lançamento.
    ```py
    fact_movies = fact_movies.merge(
        dim_date,
        on=["US_RELEASE_DATE"],
        how="left",
    )
    ```
15. Seleção final das chaves dimensionais e métricas numéricas da tabela fato.
    ```py
    fact_movies = fact_movies[
        [
            "ID_FILM", "ID_STAFF", "ID_FRANCHISE", "ID_PRODUCTION",
            "ID_DATE", "BUDGET_x", "INFLATION_ADJUSTED_BUDGET",
            "GROSS_WORLD_WIDE", "INFLATION_ADJUSTED_WORLDWIDE_GROSS",
            "GROSS_US_CANADA", "DOMESTIC", "BOX_OFFICE_GROSS_OTHER_TERRITORIES",
            "GROSS_OPENING_WEEKEND", "GROSS_TO_BUDGET",
            "RATING_IMDB", "ROTTEN_TOMATOES_CRITIC_SCORE",
            "VOTE", "OSCAR", "NOMINATION", "WIN", "MINUTES"
        ]
    ]
    ```
16. Visualização inicial da tabela fato já estruturada no modelo Star Schema.
    ```py
    fact_movies.head(3)
    ```
17. Envio das tabelas dimensionais e fato para a camada Gold no BigQuery.
    ```py
    minhas_tabelas = {
        "dim_date": dim_date,
        "dim_film": dim_film,
        "dim_franchise": dim_franchise,
        "dim_production": dim_production,
        "dim_staff": dim_staff,
        "fact_movies": fact_movies,
    }

    for key, value in minhas_tabelas.items():
        print(key)

        value.to_gbq(
            project_id=gcp_id_do_projeto,
            destination_table=f"gold.{key}",
            if_exists="replace",
            credentials=my_gcp_cred,
        )
    ```
18. Definição das chaves primárias das tabelas dimensionais no BigQuery.
    ```sql
    ALTER TABLE gold.dim_date ADD PRIMARY KEY (ID_DATE)
    NOT ENFORCED;

    ALTER TABLE gold.dim_film ADD PRIMARY KEY (ID_FILM)
    NOT ENFORCED;

    ALTER TABLE gold.dim_franchise ADD PRIMARY KEY (ID_FRANCHISE)
    NOT ENFORCED;

    ALTER TABLE gold.dim_production ADD PRIMARY KEY (ID_PRODUCTION)
    NOT ENFORCED;

    ALTER TABLE gold.dim_staff ADD PRIMARY KEY (ID_STAFF)
    NOT ENFORCED;
    ```
19. Criação das chaves estrangeiras da tabela fato para garantir o relacionamento com as dimensões.
    ```sql
    ALTER TABLE gold.fact_movies ADD CONSTRAINT fk_date 
    FOREIGN KEY (ID_DATE) REFERENCES gold.dim_date (ID_DATE)
    NOT ENFORCED;

    ALTER TABLE gold.fact_movies ADD CONSTRAINT fk_film 
    FOREIGN KEY (ID_FILM) REFERENCES gold.dim_film (ID_FILM)
    NOT ENFORCED;

    ALTER TABLE gold.fact_movies ADD CONSTRAINT fk_franchise 
    FOREIGN KEY (ID_FRANCHISE) REFERENCES gold.dim_franchise (ID_FRANCHISE)
    NOT ENFORCED;

    ALTER TABLE gold.fact_movies ADD CONSTRAINT fk_production 
    FOREIGN KEY (ID_PRODUCTION) REFERENCES gold.dim_production (ID_PRODUCTION)
    NOT ENFORCED;

    ALTER TABLE gold.fact_movies ADD CONSTRAINT fk_staff 
    FOREIGN KEY (ID_STAFF) REFERENCES gold.dim_staff (ID_STAFF)
    NOT ENFORCED;
    ```

</details>

<div align="center"> 

## Extras
</div>
<details><summary>ℹ️ Detalhes...</summary>

<h2 align="left">The Movie Database (TMDB)</h2>

- Crie e configure o acesso à API do TMDB
    - 1º Criar conta: https://themoviedb.org/signup
    - 2º Solicitar chave da API: https://themoviedb.org/settings/api/request
    - 3º Configurações da API: https://themoviedb.org/settings/api

> [!NOTE]
> Leia a documentação: https://pypi.org/project/tmdbsimple

1. Define a chave de autenticação da API do TMDB.
    ```py
    import tmdbsimple as tmdb

    tmdb.API_KEY = "____________________"
    ```
2. Busca informações de filmes no TMDB.
    ```py
    tmdb_movies = tmdb.Search().movie(query="Spider-Man 2")

    tmdb_data = tmdb_movies.get("results")

    pd.DataFrame(data=tmdb_data)
    ```
3. Busca o ID de uma pessoa/artista no TMDB.
    ```py
    star_id = tmdb.Search().person(query="Tony Ramos").get("results")[0].get("id")
    ```
4. Obtém redes sociais cadastradas de uma pessoa no TMDB.
    ```py
    tmdb.People(star_id).external_ids().get("instagram_id")
    ```

<h2 align="left">Igmapper</h2>

> [!NOTE]
> Leia a documentação: https://pypi.org/project/igmapper

1. Configura o cliente de acesso ao Instagram via Igmapper.
    ```py
    import igmapper

    client = igmapper.InstaClient(csrftoken=None, ds_user_id=None, sessionid=None, use_curl=True)
    ```
2. Obtém informações públicas de um perfil do Instagram.
    ```py
    client.get_profile_info("therock")
    ```
3. Obtém as publicações do feed de um perfil do Instagram.
    ```py
    client.get_feed("therock")
    ```

</details>