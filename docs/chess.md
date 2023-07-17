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
Chess.com API is a public API that does not require authentication or including secrets in secrets.toml.

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

### Add credentials

1. To add credentials to your destination, follow the instructions in the [destination documentation](../../dlt-ecosystem/destinations). This will ensure that your data is properly routed to its final destination.

For more information, read the [General Usage: Credentials.](../../general-usage/credentials)

## Run the pipeline

1. Before running the pipeline, ensure that you have installed all the necessary dependencies by
   running the command:
   ```bash
   pip install -r requirements.txt
   ```
1. You're now ready to run the pipeline! To get started, run the following command:
   ```bash
   python3 chess_pipeline.py
   ```
1. Once the pipeline has finished running, you can verify that everything loaded correctly by using
   the following command:
   ```bash
   dlt pipeline <pipeline_name> show
   ```
   For example, the `pipeline_name` for the above pipeline example is `chess_pipeline`, you may also use any
   custom name instead.

For more information, read the [Walkthrough: Run a pipeline.](../../walkthroughs/run-a-pipeline)

## Sources and resources

`dlt` works on the principle of [sources](../../general-usage/source) and [resources](../../general-usage/resource).

### Source `source`

```python
dlt.source(name="chess")
def source(
    players: List[str], start_month: str = None, end_month: str = None
) -> Sequence[DltResource]: 
   return (
         players_profiles(players),
         players_archives(players),
         players_games(players, start_month=start_month, end_month=end_month),
         players_online_status(players),
         )
```
`players`: This is a list of player usernames for which you want to fetch data.
`start_month` and `end_month`: These optional parameters specify the time period for which you want to fetch game data. (In  "YYYY/MM" format).

The above function is a dlt.source function for the Chess.com API named "chess", which returns a sequence of DltResource objects. That we'll discuss subsequently. 

### Resource `players_profiles`

```python
@dlt.resource(write_disposition="replace")
def players_profiles(players: List[str]) -> Iterator[TDataItem]:
    
    @dlt.defer
      def _get_profile(username: str) -> TDataItem:
          return get_path_with_retry(f"player/{username}")

       for username in players:
         yield _get_profile(username)
```

 `players`: is a list of player usernames for which you want to fetch profile data.
 
 The `_get_profile` function fetches profile data for a single player using an API request. The `get_path_with_retry` function handles errors and makes the request. The `for` loop iterates through a list of players, yielding their profile data.

### Resource `players_archives`

```python
@dlt.resource(write_disposition="replace", selected=False)
def players_archives(players: List[str]) -> Iterator[List[TDataItem]]:
    
     for username in players:
        data = get_path_with_retry(f"player/{username}/games/archives")
        yield data.get("archives", [])
```
`players`: is a list of player usernames for which you want to fetch archives.

`selected=False`: parameter means that this resource is not selected by default when the pipeline runs.

The `for` loop iterates over a list of players and fetches their archives using an API request. It uses the `get_path_with_retry` function. The loop yields the list of archives or an empty list if there are none.


















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