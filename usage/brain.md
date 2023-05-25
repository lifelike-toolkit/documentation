## Brain Usage
Install the package: `pip install git+https://github.com/lifelike-toolkit/lifelike.git`

Then import it with `from lifelike import brain`

The `Characters` class requires a file path to a JSON file where the characters data will be saved. To create a `Characters` object, initialize it as follows:
```python
from lifelike import brain
characters = brain.Characters("/path/to/your/characters.json")
```
Once the object is initialized, the following methods can be used:
- `add(self, name: str, background: str) -> None: ` adds a character to the character class. Needs a unique name and background written in natural language.
- `get(self, name: str) -> str: ` returns the background of a character.
- `update(self, name: str, background: str) -> None: ` updates the background of a character.
- `delete(self, name) -> None: ` removes a character from the Characters class
- `save(self) -> None: ` saves all characters to path specified in initiailization

The `Conversations` class requires a file path to a JSON file where the conversations data will be saved, a `Characters` object ss well as a `langchain.llms` object. Initiailize it as follows:
```python
# using 'characters' and 'import brain' from above.
from langchain.llms import LlamaCpp
llm = LlamaCpp(model_path='ggml-model-q4_0.bin') # Can be any LLM

conversations = brain.Conversations("/path/to/your/conversations.json",
                                    characters,
                                    llm)
```
Once the object is initialized, the following methods can be used:
- `new(self, context: str, participants: Set[str]) -> None: ` with a unique context and participants creates a new conversation in Conversations.
- `get(self, context: str) -> Dict[str, any]:` returns the participants and log of a conversation in the structure of `{"participants":set, "log":[]}` from a conversation with context
- `update(self, context: str, participants: Set[str], log: List[List[str]]) -> None: ` with a unique context update the participants (set) and log (list of [character, utterance] in sequential order)
- `delete(self, context: str) -> None: ` deletes a conversation from Conversations
- `append(self, context: str, speaker: str, utterance: str) -> None:` add utterance from speaker in conversation with unique context
- `generate(self, context: str, muted: Set[str]) -> List[str]:` given a conversations context and muted characters have an unmuted character say something and add it to the conversation. returns `[speaker, utterance]`
- `save(self) -> None: ` saves all conversations to a path specialized in initialization


## In Development
Creating a game vs having it run user side is going to be a little different in this toolkit. Here I go forward with an idea of how it's used in these scenarios.

Basically, once you add a character to the Characters class or add a conversation to the conversations class if you try to add them again the system will throw an error.

So, here's how you'd set a game up. In this assumption there is no characters.json or conversations.json.

```python
from lifelike import brain
from langchain.llms import LlamaCpp # any langchain llm works.

llm = LlamaCpp(model_path='ggml-model-q4_0.bin')

characters = brain.Characters('characters.json')

characters.add("Player", "a description of the player character is not needed, but they still need to be in the characters class.")
characters.add("NPC1", "Add detailed descriptions of npcs for better useage")

characters.save()


conversations = brain.Conversations("conversations.json",
                                    characters,
                                    llm)


context = "Add a detailed description of the context for better useage, try to include all characters names here (e.g. Player and NPC1 are walking up a mountain)"

characters_in_convo = {"Player", "NPC1"} # These characters must be in the characters class otherwise an error will be thrown. Must also be a set.

conversations.add(context, characters_in_convo)

conversations.save()
```

Basically create all the conversations and characters and save them to a file, when you ship your game, include these files in the game so they can just be read and you don't need to run the code every time.

## In Game

User side in a game just load in the characters and conversations and you can pretty much do anything except add the same conversation again. In this example there is an existing characters.json and conversations.json as defined in the code above. Here it is:

```python
from lifelike import brain
from langchain.llms import LlamaCpp # any langchain llm works.

llm = LlamaCpp(model_path='ggml-model-q4_0.bin')

characters = brain.Characters('characters.json')

conversations = brain.Conversations("conversations.json",
                                    characters,
                                    llm)

context = "Add a detailed description of the context for better useage, try to include all characters names here (e.g. Player and NPC1 are walking up a mountain)"


first_utterance = "Jheez this walk is getting long"
conversations.append(context, "NPC1", first_utterance) # It's good to start off conversations with an utterance from the NPC created yourself to kickstart the conversation

output("NPC1", first_utterance) # This isn't a part of the toolkit, just output to wherever the user will see it.


response = conversations.generate(context, {"Player1"}) # It is good practice to mute the player characters so that the toolkit doesn't generate a response from them and they remain player controlled.

output(response) # response is just a list of [speaker, utterance], throw it into output that user can see.
```

Following the above pattern you can create loops that can indefinately keep conversations going.