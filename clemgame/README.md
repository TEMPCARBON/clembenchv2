# Preparation for separating games and framework (whereby a benchmark run becomes a framework run with specific games)

## Preamble
### General Questions
* Naming confusion: The class `GameBenchmark` is used for a complete run of all instances of one game (not a set of specific games constituting a benchmark version)
* clemgame could be renamed into framework and framework.py moved to \__init\__.py
* GameMaster vs. DialogueGameMaster: latter extends former/is the separation needed? former used in every game, the latter (additionally) in matchit/mapworld/hellogame/cloudgame/taboo, see example below:
```
class Taboo(DialogueGameMaster):
...

class TabooGameBenchmark(GameBenchmark):
    def create_game_master(self, experiment: Dict, player_models: List[Model]) -> GameMaster:
        return Taboo(experiment, player_models)

```
* \__init\__.py in clemgame and backends defines project_root and configures logging, respectively. Is this redundant or am I missing something? Also, what's the difference in the loggers instantiated in chatgame/game.py and master.py?  

## TODOs:
* adapt instance file default location to game default location
* update test_benchmark.py (also contains old versions of reference game)

## Preparational Thoughts
### Adding a new game
* implement game based on [template](#game-template)
* add entry in game registry

## Game Registry Fields:

```
{
"game_name": game identifier
"game_path": path to game  # absolute or relative to clemgame directory
"description": "A brief description of the game"
"main_game": "main game identifier" # to cluster different versions of the same game
"player": "single" | "two" | "multi"
"image": "none" | "single" | "multi"
"languages": ["en"] # list of ISO codes
"benchmark": ["X.X", "Y.Y"] # lists all benchmark versions in which this game was used 

# The games that are part of a specific collection can be filtered based on the 
# game attributes.
# For reproducibility, benchmark will also list all benchmark versions a game has   
# been used in previously
}
```

## Isolate Game Template from Framework?
### (Abstract) Classes (clemgame/clemgame.py):
* InstanceGenerator
* Player
* GameMaster (DialogueGameMaster)?
* GameScorer?

### Game Structure
```
game
    in # directory containing instances_LANG_VERSION.json
    resources
        lang (optional)
            other resources
            initial_prompts
    instancegenerator.py # script reading in resources and generating instance file(s)
    game.py (sometimes also just master.py)
    master.py
```

### Results Structure
built by GameMaster and GameScorer, path specified as argument in cli.py, no changes needed

### Game collections
* benchmark versions (currently different versions of code and instances, in the future only different instances)
  * text based benchmark (see clembench paper)
  * multimodal benchmark (see current version of the multimodal paper)
* game class (several versions of one game, for in-depth analysis)

### Required Changes: 
```
 clemgame
| 
+--- __init__.py 
|       --> add GameSpec class based on ModelSpec (from backends)
        --> load_custom_game_registry() and load_game_registry() similar to loading model registries
+--- benchmark.py # renamed to framework.py
|       list_games() # replaced to reading from game_registry
|       run() # adapted to new game loading
|       score() # adapted to new game loading
|       transcripts() # adapted to new game loading
+--- clemgame.py
|       load_benchmarks() # renamed to select_games() and adapted based on game registry and selection
|       load_benchmark() # renamed to load_game() and adapted to load game from different location
|       find_benchmark() # integrated into load_benchmark()
+--- file_utils.py
|       game_dir() # needs to implement lookup in game_registry!
 scripts
|
+--- cli.py # renamed benchmark to framework
|
 tests
|
+--- test_benchmark.py # rename to test_framework.py and adapt
+--- logging.yaml # renamed main logger to framework.run
+--- run_benchmark.sh # added to run a specific set of games constituting a benchmark version
+--- game_selection.json # added to specify game properties for loading collections of games
```
