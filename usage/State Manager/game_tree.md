# Game Tree
The game tree defines the different states that the game will go through. A game can have multiple game trees. This tree can be used as a decision tree, story tree, knowledge database and many more. The basic game tree provided with the toolkit is meant to be used as a story tree, but you can reimplement it at will.

The game tree exposes a get_retriever() method which takes a string as input. By configuring the game tree instance in the game's code, you can define what string gets returned by get_retriever. This will be expanded on further in the basic visual novel example.

The game tree can be saved to a json, which allows bypassing the original embedding step, which can be costly if using openai.

# Import 
In the code for

# Development
## Configuring a tree instance
## Saving game tree instance
## Loading game tree instance from json
## Fine-tuning embeddings   

# In a game

# Basic visual novel with the base Game Tree

# Custom Implementation:
A derived class from BaseGameTree must expose a `get_retriever()` method.