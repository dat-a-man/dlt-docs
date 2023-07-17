---
title: Chess.com
description: dlt verified source for Chess.com API
keywords: [chess.com api, chess.com verified source, chess.com]
---

# Chess.com

:::info Need help deploying these sources, or figuring out how to run them in your data stack?

[Join our slack community](https://dlthub-community.slack.com/join/shared_invite/zt-1slox199h-HAE7EQoXmstkP_bTqal65g) or [book a call](https://calendar.app.google/kiLhuMsWKpZUpfho6) with our support engineer Adrian.
:::

Chess.com is an online platform that offers services for chess enthusiasts. It includes online chess games, tournaments, lessons, and more.

Resources that can be loaded using this verified source are:
| Name             | Description                                                                                     |
|------------------|-------------------------------------------------------------------------------------------------|
| players_profiles | retrives player profiles for a list of player usernames                                         |
|players_archives  | retrives url to game archives for specified players                                             |
|players_games     | retrives players games that happened between start_month and end_month                          |


## Setup Guide

### Grab credentials
Chess.com API is a public API that does not require authentication or the inclusion of secrets in `secrets.toml`.

### Initialize the verified source

To get started with your data pipeline, follow these steps:
1. Enter the following command:
   ```bash
   dlt init chess duckdb
   ```
   [This command](../../reference/command-line-interface) will initialize
   [the pipeline example](https://github.com/dlt-hub/verified-sources/blob/master/sources/asana_dlt_pipeline.py) with
   Chess.com as the [source](../../general-usage/source) and [duckdb](../destinations/duckdb.md) as
   the [destination](../destinations).

2. If you'd like to use a different destination, simply replace `duckdb` with the
   name of your preferred [destination](../destinations).

3. After running this command, a new directory will be created with the necessary files and
   configuration settings to get started.

For more information, read the [Walkthrough: Add a verified source.](../../walkthroughs/add-a-verified-source)
```
dlt init chess duckdb
```
Here, we chose duckdb as the destination. To choose a different destination, replace `duckdb` with your choice of destination.

## Add credentials

Before running the pipeline you may need to add credentials in the `.dlt/secrets.toml` file for your chosen destination. For instructions on how to do this, follow the steps detailed under the desired destination in the [destinations](../destinations/) page.

Chess source does not require credentials to run.

## Run the pipeline

1. Install the necessary dependencies by running the following command:
```
pip install -r requirements.txt
```
2. Now the pipeline can be run by using the command:
```
python3 chess_pipeline.py
```
3. To make sure that everything is loaded as expected, use the command:
```
dlt pipeline chess_pipeline show
```

## Customize parameters

Without any modifications, the chess pipeline will load data for a default list of players over a default period of time. You can change these values in the `chess_pipeline.py` script.

For example, if you wish to load player games for a specific set of players, add the player list to the function `load_player_games_example` as below.
```python
def load_players_games_example(start_month: str, end_month: str):

    pipeline = dlt.pipeline(pipeline_name="chess_pipeline", destination='duckdb', dataset_name="chess_players_games_data")

    data = chess(
        [], # Specify your list of players here
        start_month=start_month,
        end_month=end_month
    )

    info = pipeline.run(data.with_resources("players_games", "players_profiles"))
    print(info)
```
To specify the time period, pass the starting and ending months as parameters when calling the function in the `__main__` block:
```python
if __name__ == "__main__" :
    load_players_games_example("2022/11", "2022/12") # Replace the strings "2022/11" and "2022/12" with different months in the "YYYY/MM" format
```