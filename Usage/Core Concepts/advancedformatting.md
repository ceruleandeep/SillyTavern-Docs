# Advanced Formatting

The settings provided in this section allow for more control over the prompt-building strategy.

**Most of the settings here do not apply to Chat Completions APIs as they are governed by the prompt manager system instead.**

## System Prompt

System Prompt defines the general instructions for the model to follow. It sets the tone and context for the conversation. For example, it tells the model to act as an AI assistant, a writing partner, or a fictional character. The System Prompt is a part of the [Story String](#story-string) and usually the first part of the prompt that the model receives.

Supports substitutions via any of the supported \{\{macro\}\} parameters.

For example:

> You are a helpful AI assistant. You should provide useful information and help \{\{user\}\} with their questions.

## Context Template

Usually, AI models require you to provide the character data to them in some specific way. SillyTavern includes a list of pre-made conversion rules for different models, but you may customize them however you like.

### Story string

This field is a template for pre-chat character data (known internally as a story string).
This is the main way to format your character card for text completion and instruct models.

The template supports Handlebars syntax and any custom text injections or formatting. See the language reference here: <https://handlebarsjs.com/guide/>

We provide the following parameters to the Handlebars evaluator (wrap them into double-curly braces):

1. `description` - character's Description
2. `scenario` - character's Scenario
3. `personality` - character's Personality
4. `system` - [system prompt](#system-prompt) OR character's main prompt override (if exists and "Prefer Char. Prompt" is enabled in User Settings)
5. `persona` - selected persona description
6. `char` - character's name
7. `user` - selected persona name
8. `wiBefore` or `loreBefore` - combined activated World Info entries with Position set to "Before Char Defs"
9. `wiAfter` or `loreAfter` - combined activated World Info entries with Position set to "After Char Defs"
10. `mesExamples` - (optional) character's Example Dialogues, instruct-formatted with separator. Set "Example Messages Behavior" to "Never include examples" to avoid duplication.

A special \{\{trim\}\} macro is supported to remove any newlines that surround it. Use it in case you want some part of text NOT be separated with a newline from the previous line (_spaces **are not** trimmed_).

**WARNING**: If some of the above parameters are missing from the story string template, they are not going to be sent in the prompt at all.

### Example Separator

Used as a block header and a separator between the example dialogue blocks. Any instance of `<START>` tags in the example dialogues will be replaced with the contents of this field.

### Chat Start

Inserted as a separator after the rendered story string and after the example dialogues blocks, but before the first message in context.

### Separators as Stop Strings

Adds "Example Separator" and "Chat Start" to the list of stop strings.

Helpful if the model tends to hallucinate or leak whole blocks of example dialogue preceded by the separator.

### Names as Stop Strings

Adds Character and User Persona names to the list of stop strings.

Recommended to keep it on to prevent model impersonation.

### Allow Post-History Instructions

Includes the Post-History Instructions at the end of the prompt, formatted as the last user message.

The Post-History Instructions prompt should be defined in the character card and "Prefer Char. Instructions" setting should be enabled.

Should be used with care, as placing instructions low in the context can lead to degraded quality of the outputs of smaller models.

### Always add character's name to prompt

Appends the character's name to the prompt to force the model to complete the message as the character:

```txt
** OTHER CONTEXT HERE **
Character:
```

### Tokenizer

A tokenizer is a tool that breaks down a piece of text into smaller units called tokens. These tokens can be individual words or even parts of words, such as prefixes, suffixes, or punctuation. A rule of thumb is that one token generally corresponds to 3~4 characters of text.

SillyTavern provides a "Best match" option that tries to match the tokenizer using the following rules depending on the API provider used.

Text Completion APIs **(overridable)**:

1. NovelAI Clio: NerdStash tokenizer.
2. NovelAI Kayra: NerdStash v2 tokenizer.
3. Text Completion: API tokenizer (if supported) or Llama tokenizer.
4. KoboldAI Classic / AI Horde: Llama tokenizer.
5. Koboldcpp: model API tokenizer.

If you get inaccurate results or wish to experiment, you can set an _override tokenizer_ for SillyTavern to use while forming a request to the AI backend:

1. None. Each token is estimated to be ~3.3 characters, rounded up to the nearest integer. **Try this if your prompts get cut off on high context lengths.** This approach is used by KoboldAI Lite.
2. Llama tokenizer. Used by Llama 1/2 models family: Vicuna, Hermes, Airoboros, etc. **Pick if you use a Llama 1/2 model.**
3. Llama 3 tokenizer. Used by Llama 3/3.1 models. **Pick if you use a Llama 3/3.1 model.**
4. NerdStash tokenizer. Used by NovelAI's Clio model. **Pick if you use the Clio model.**
5. NerdStash v2 tokenizer. Used by NovelAI's Kayra model. **Pick if you use the Kayra model.**
6. Mistral V1 tokenizer. Used by older Mistral models family and their finetunes. **Pick if you use a Mistral model.**
7. Yi tokenizer. Used by Yi models. **Pick if you use a Yi model.**
8. Gemma tokenizer. Used by Gemini/Gemma models. **Pick if you use a Gemma model.**
9. API tokenizer. Queries the generation API to get the token count directly from the model. Known backends to support: Text Generation WebUI (ooba), koboldcpp, TabbyAPI, Aphrodite API. **Pick if you use a supported backend.**

Chat Completion APIs **(non-overridable)**:

1. OpenAI: model-dependant tokenizer via [tiktoken](https://github.com/openai/tiktoken).
2. Claude: model-dependant tokenizer via [WebTokenizers](https://github.com/mlc-ai/tokenizers-cpp).
3. OpenRouter: Llama, Mistral, Gemma, Yi tokenizers for their respective models.
4. Google AI Studio: Gemma tokenizer.
5. Scale API: GPT-4 tokenizer.
6. AI21 API: Jamba tokenizer (requires a one-time download).
7. Cohere API: Command-R tokenizer (requires a one-time download).
8. MistralAI API: Mistral V1 or V3 tokenizer (requires a one-time download).
9. Fallback tokenizer: GPT-3.5 turbo tokenizer.

#### Additional Tokenizers

These tokenizers are not included in the default installation due to their size A one-time download is required when they're used for the first time.

1. Qwen2 tokenizer.
2. Command-R tokenizer. Used by Cohere source in Chat Completion.
3. Mistral V3 (Nemo) tokenizer. Used by MistralAI source in Chat Completion (Nemo and Pixtral models).

If you don't want to use internet downloads, the opt-out option exists in config.yaml: `enableDownloadableTokenizers`. Set to `false` to disable downloads.

You can also download tokenizers manually from the [SillyTavern-Tokenizers](https://github.com/SillyTavern/SillyTavern-Tokenizers) repository. Download the JSON files and put them in the `_cache` subdirectory of your data root, the path is `./data/_cache` by default. Create the `_cache` directory if it doesn't exist. After that, restart the SillyTavern server to re-initialize tokenizers.

If the required tokenizer model is not cached and downloads are disabled, a fallback tokenizer (Llama 3) will be used for counting.

### Token Padding

**Important: This section doesn't apply to Chat Completions API. SillyTavern will always use a matching tokenizer for these models.**

Unless SillyTavern uses a tokenizer provided by the remote backend API that runs the model, all token counts assumed during prompt generation are estimated based on the selected [tokenizer](#tokenizer) type.

Since the results of tokenization can be inaccurate on context sizes close to the model-defined maximum, some parts of the prompt may be trimmed or dropped, which may negatively affect the coherence of character definitions.

To prevent this, SillyTavern allocates a portion of the context size as padding to avoid adding more chat items than the model can accommodate. If you find that some part of the prompt is trimmed even with the most-matching tokenizer selected, adjust the padding so the description is not truncated.

You can input negative values for reverse padding, which allows allocating more than the set maximum amount of tokens.

## Custom Stopping Strings

Accepts a JSON-serialized array of stopping strings. Example: `["\n", "\nUser:", "\nChar:"]`. If you're unsure about the formatting, use an [online JSON validator](https://jsonlint.com/). If the model output **ends** with any of the stop strings, they will be removed from the output.

Supported APIs:

1. KoboldAI Classic (versions 1.2.2 and higher) or KoboldCpp
2. AI Horde
3. Text Completion APIs: Text Generation WebUI (ooba), Tabby, Aphrodite, Mancer, TogetherAI, Ollama, etc.
4. NovelAI
5. OpenAI (max 4 strings) and compatible APIs
6. OpenRouter (both Text and Chat Completion)
7. Claude
8. Google AI Studio
