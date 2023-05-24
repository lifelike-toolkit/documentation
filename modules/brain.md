# Characters and Conversations API Reference
## Characters Class
The `Characters` class is an interface to manage characters.

```python
class Characters:
    ...
```
**Methods**

`__init__(self, path: str) -> None`
* Initializes a Characters instance.
* Parameters:
  * path: Path to the JSON file containing the characters data.


`is_out(self, name: str) -> ValueError`
* Checks if a character is not in the characters list.
* Parameters:
  * name: Unique name of the character.
* Returns: Raises a ValueError if the character does not exist.

`is_in(self, name: str) -> ValueError`
* Checks if a character is in the characters list.
* Parameters:
  * name: Unique name of the character.
* Returns: Raises a ValueError if the character already exists.

`get(self, name: str) -> str`
* Retrieves a character's background.
* Parameters:
  * name: Unique name of the character.
* Returns: The background of the character.

`add(self, name: str, background: str) -> None`
* Adds a character to the characters list.
* Parameters:
  * name: Unique name of the character.
  * background: Background of the character. More detailed is better.
  
`update(self, name: str, background: str) -> None`
* Updates a character's background.
* Parameters:
  * name: Unique name of the character.
  * background: New background of the character. More detailed is better.


`delete(self, name: str) -> None`
* Deletes a character from the characters list.
* Parameters:
  * name: Unique name of the character.

`__str__(self) -> str`
* Provides a string representation of the Characters instance.
* Returns: String representation of Characters.

`save(self) -> None`
* Saves the Characters instance to a JSON file.

## Conversations Class
The `Conversations` class is an interface to manage conversations.

```python
class Conversations:
    ...
```
**Methods**

`__init__(self, path: str, characters: Characters, llm) -> None`
* Initializes a Conversations instance.
* Parameters:
  * path: Path to the JSON file containing the conversations data.
  * characters: A Characters object.
  * llm: A language model object (LLM). Look here for more info: https://python.langchain.com/en/latest/modules/models/llms/integrations.html

`context_out(self, context: str) -> ValueError`
* Checks if a conversation context does not exist.
* Parameters:
  * context: Unique context of the conversation.
* Returns: Raises a ValueError if the conversation context does not exist.

`context_in(self, context: str) -> ValueError`
* Checks if a conversation context exists.
* Parameters:
  * context: Unique context of the conversation.
* Returns: Raises a ValueError if the conversation context already exists.

`valid_participants(self, participants: Set[str]) -> ValueError`
* Validates the participants in a conversation.
* Parameters:
  * participants: A set of character names.
* Returns: Raises a ValueError if any of the participants are invalid.

`get(self, context: str) -> Dict[str, any]`
* Retrieves a conversation context.
* Parameters:
  * context: Unique context of the conversation.
* Returns: A dictionary of participants and log of the conversation.

`new(self, context: str, participants: Set[str]) -> None `
* Creates a new conversation.
* Parameters: 
  * context: Unique context of the conversation.
  * participants: A set of character names. (Must be in Characters class)


`update(self, context: str, participants: Set[str], log: List[List[str]]) -> None`
* Updates a conversation.
* Parameters:
  * context: Unique context of the conversation.
  * participants: A set of character names.
  * log: A list of [speaker, utterance] pairs.

`delete(self, context: str) -> None`
* Deletes a conversation.
* Parameters:
  * context: Unique context of the conversation.

`append(self, context: str, speaker: str, utterance: str) -> None`
* Appends an utterance to a conversation.
* Parameters:
  * context: Unique context of the conversation.
  * speaker: Name of the speaker.
  * utterance: Utterance of the speaker.

`generate(self, context: str, history: str, muted: Set[str]) -> List[str]`
* Generates a response in the conversation based on the context and the history.
* Parameters:
  * context: Unique context of the conversation.
  * history: Conversation history. 
  * muted: A set of muted character names.
* Returns: A list containing speaker and the generated utterance.

`__str__(self) -> str`
* Provides a string representation of the Conversations instance.
* Returns: String representation of Conversations.

`save(self) -> None`
* Saves the Conversations instance to a JSON file.
