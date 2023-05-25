# Game Tree Usage
The game tree defines the different states that the game will go through. A game can have multiple game trees. This tree can be used as a decision tree, story tree, knowledge database and many more. The basic game tree provided with the toolkit is meant to be used as a story tree, but you can reimplement it at will.

The game tree exposes a get_retriever() method which takes a string as input. By configuring the game tree instance in the game's code, you can define what string gets returned by get_retriever. This will be expanded on further in the basic visual novel example.

When the player starts a game, one of the main processes is setting up the game tree. This where developers write the story and set up the behaviours of the game tree. The game tree can be saved to a json, which allows bypassing the original embedding step, which can be costly if using OpenAI.

# Import 
In the game code, writes:
```python
from lifelike.StateManager.base_game_tree import BaseGameTree, EdgeEmbedding, GameNode
```
EdgeEmbedding and GameNode are optional for most uses. Import them if you want to modify their behaviour.       

# Development
This is the guide for building a game tree during development. Don't confuse this with how Game Tree is used when a game is run, which will be covered in the next section.
## Configuring a tree instance
## Saving game tree instance
## Loading game tree instance from json
## Fine-tuning embeddings   

# Game Logic


# Basic visual novel with the base Game Tree

# Custom Implementation:
A derived class from BaseGameTree must expose a `get_retriever()` method.