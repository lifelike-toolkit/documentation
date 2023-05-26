# Game Tree Usage
The base game tree provided by the toolkit is defines the different states that the game will go through using vectorstorage. A game can have multiple game trees. This tree can be used as a decision tree, story tree, knowledge database and many more. The basic game tree provided with the toolkit is meant to be used as a story tree, but you can reimplement it at will.

The game tree exposes a get_retriever() method which takes a string as input and returns a [langchain Retriever](https://python.langchain.com/en/latest/modules/indexes/retrievers.html) instance. By configuring the game tree instance during development, you can define what string gets returned by get_retriever(). For example, you can have the tree  create a langchain This will be expanded on further in the later sections.

When the player starts a game, one of the main processes is setting up the game tree. This where developers write the story and set up the behaviours of the game tree. The game tree can be saved to a json, which allows bypassing the original embedding step, which can be costly if using OpenAI.

# Import 
In the game code, writes:
```python
from lifelike.StateManager.base_game_tree import BaseGameTree, EdgeEmbedding, GameNode
```       

# Development
This is the guide for building/configuring a Game Tree for a game. Don't confuse this with how a Game Tree is used when a game is run, which will be covered in the next section.
## Configuring a tree instance
There are 2 different ways to add a nodes to the tree: using `add_node()` or `add_texts()`. I recommend `add_texts()` for most use cases as it removes the need to provide custom embeddings. If you want custom embeddings, however, you can create `GameNode` instances and add them to the tree with `add_node()`. Note that for the tree to work, you must connect the nodes to each other with `add_edge()` 
## Embedding template
Embedding template allows you to conveniently reuse embeddings that were made beforehands. If you want to save the embedding for use at another time, the embedding will need to be saved to the template dictionary first using `add_embedding_template()`. It can then be fetched with `get_embedding_template()`
## Fine-tuning embeddings   
This is a system that allows you to provide multiple different strings that you want to be matched to a certain node. When an EdgeEmbedding is initialized, it will not have any data and cannot be used. In order to use it to connect nodes, you must use EdgeEmbedding's `tune_prompt()` or GameTree's `tune_edge()` functions.
## Saving and loading game tree instance
You can save current progress with `to_json()` which generates a json file for the game that you can load back up later on with `build_from_json()`. When the game is shipped, it is recommended that the tree is loaded this way, especially if you have a large language model doing the embedding, or has a token limit.

# Game Logic
Within the game logic, the Game Tree provides an external interface in the form of the [langchain Retriever](https://python.langchain.com/en/latest/modules/indexes/retrievers.html) instance returned by `get_retriever()`. By configuring the game tree, you can define what comes out of the retriever when you provide a string. For example, in a visual novel game, each node in the game tree may represent a dialogue chain that results from the player saying a certain thing. Refer to the Development section for the different configurations. 

Alternatively, you can look at the Sequence Tree and the Knowledge Tree, which are more specific implementations of the Base Game Tree class.

# Basic visual novel with the base Game Tree
In this example, we're making a graphic novel that allows players to say whatever they want and push the story forward accordingly. We will assume that the embedding model you are using is a proper langchain Embeddings instance stored in `LLM_Embedding` (refer to the [LLM Usage Guide](../llm.md)). In this example, I chose the OpenAI Embedding model.

```python
from lifelike.base_game_tree import BaseGameTree, EdgeEmbedding, GameNode # Skip EdgeEmbedding and GameNode if you take the easy way
from llm import LLM_Embedding

tree = BaseGameTree("demo_novel", LLM_Embedding)
```
After initializing the tree, you can now add nodes to it. For this example, we will have a dialogue choice that can lead to 1 out of 2 outcomes. In this case, the player will be asked if they like video games. I will assume the metadatas are defined for each nodes. I will be showcasing 2 different ways to handle this:
- **The easy way**:
    ```python
    # Populating the tree with texts
    tree.add_texts(texts = ['Game: Do you like video games? Player: I love video games', 'Game: Do you like video games? Player: I hate video games'], metadatas=metadatas)
    ```
    Note that with this method, these nodes can be accessed from anywhere in the tree, so I would not recommend it for a game like this. However, the [Sequence Tree](sequence_tree.md) was built with this type of game in mind so it behaves slightly differently and will allow this method to be used in this type of game effectively. I will discuss the differences in  [Sequence Tree](sequence_tree.md).

- **The hard way** (the behaviour is customizeable based on how you define the nodes and edges):
    ```python
    # Populating the tree with pre-defined nodes
    tree.add_node(GameNode(id='0', context='start', metadata=metadatas))
    tree.add_node(GameNode(id='1', context='Game: Do you like video games? Player: I love video games', metadata=metadatas))
    tree.add_node(GameNode(id='2', context='Game: Do you like video games? Player: I hate video games', metadata=metadatas))
    ```
    Note that I provide the question as well as the answer, so the context of the question does not get lost. Your mileage here may vary based on what you're doing, so I recommend playing around with this code and see what kind of context string will work for you. Now we can define some edges. If you want the template to be saved for later, add it to the tree's edge template dictionary. This template will be saved to the json file for use later.
    ```python
    like_embedding = EdgeEmbedding(name='like_template', embedding_function=tree.embed, current_weight=0)
    like_embedding.tune_prompts('Game: Do you like video games? Player: I love video games')
    dislike_embedding = EdgeEmbedding(name='dislike_template', embedding_function=tree.embed, current_weight=0)
    dislike_embedding.tune_prompts('Game: Do you like video games? Player: I hate video games')

    # Optional
    tree.add_embedding_template(like_embedding)
    tree.add_embedding_template(dislike_embedding)
    ``` 
    Now we can add these edges to the tree and connect the nodes. Note that these nodes must already exist in the tree.
    ```python
    tree.add_edges(start_id='0', end_id='1', embedding_name='like', embedding_template=like_embedding)
    tree.add_edges(start_id='0', end_id='2', embedding_name='dislike', embedding_template=dislike_embedding)
    ```

The tree in this state is only temporary and stored in memory. From here, you can save your progress with `tree.to_json(path_to_file.json)`. This tree can then be loaded in the future as described in the previous section. If you want to run the game however, you need to write it to the vectordatabase, which will then allow the retriever to function properly. 
```python
# Writes to database. If this step is skipped, the retriever will return an empty string no matter the query
tree.write_db()
# Get retriever
retriever = tree.get_retriever()
```
Now that this is done, let's say the game asks the question:
`Game: Do you like video game?` and the player responds with `Player: No, I absolutely despise it`. If you run `retriever.get_relevant_documents("Game: Do you like video game? Player: No, I absolutely despise it")`, the retriever will return a langchain document that will store the `'Game: Do you like video games? Player: I hate video games'` context and its corresponding metadata that was defined earlier.
# Custom Implementation:
A derived class from BaseGameTree must expose a `get_retriever()` method.

We provide 2 examples of custom implementations of the Base Game Tree: Sequence Tree and Knowledge Tree. Each demonstrate a different capability of the Base Game Tree.