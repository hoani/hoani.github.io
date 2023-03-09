---
title: "OpenAI - Chat"
excerpt: "Using the API which drives ChatGPT"
toc: true
permalink: "/guides/engineering/openai/chat"
categories:
  - guide
  - software
  - engineering
  - openai
  - golang
  - python
---

OpenAI's chat API allows us to access the models used for ChatGPT and allows us to create conversational interactions. These interactions often feel more natural since it is easier to provide the chat API with directives.

```
[messages] -> OpenAI Chat API -> [next message]
```

# Http API

Using CURL we can ask the `chat` API to continue a conversation:

```
export OPENAI_KEY="<your-key-here>"
curl https://api.openai.com/v1/chat/completions \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer '$OPENAI_KEY'' \
  -d '{
  "model": "gpt-3.5-turbo",
  "messages": [
    {"role": "system", "content": "reference pokemon in every response"},
    {"role": "user", "content": "what is a good name for a mouse?"}
  ]
}'
```

The response contains:
```json
"message":
{
  "role":"assistant",
  "content":"How about Pikachu? It's a cute and recognizable name inspired by the popular Pokémon franchise."
}
```

The API is stateless, so to carry on the conversation, we need to provide previous messages:

```
curl https://api.openai.com/v1/chat/completions \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer '$OPENAI_KEY'' \
  -d '{
  "model": "gpt-3.5-turbo",
  "messages": [
    {"role": "system", "content": "reference pokemon in every response"},
    {"role": "user", "content": "what is a good name for a mouse?"},
    {"role": "assistant", "content": "How about Pikachu? Its a cute and recognizable name inspired by the popular Pokémon franchise."},
    {"role": "user", "content": "Please suggest something which feels more futuristic?"}
  ]
}'
```

To which I got the response:
```json
"message":
{
  "role":"assistant",
  "content":"You can name your mouse Jolteon, which is one of the Pokémons known for its speed and agility. It has a futuristic vibe to it, and the name sounds like it belongs to a tech-savvy, lightning-fast creature."
}
```

There are three message types in this API:
* `"system"` - this sets the premise of the chat. In this case I asked the API to always mention pokemon, which is why it would only suggest pokemon related names for my mouse.
* `"user"` - these are messages which the API will respond to
* `"assistant"` - these messages allow us to provide the API with chat history.

## API Key

To get an API Key, create an OpenAI account, you can generate tokens under your profile.

<figure>
    <img src="/assets/images/posts/guides/openai/000_apikey.png">
</figure>

## Parameters

There are several parameters we can manipulate when we make a call to `chat` API.

### max_tokens

`max_tokens` specifies how many tokens you will allow the API request to consume. 

### temperature

`temperature` specifies the "randomness" of the result. `temperature` has a range of `[0, 2]`. A temperature of 0 provides results which are less variable; while a temperature of 2 is more variable but may not match the prompt as precisely. It is 1 by default.

### n

`n` allows us to set the number of choices the API provides. It is 1 by default.

### user

`user` allows you to set a user id for the callee. This allows the API to track users which are using the API to make inappropriate requests which are outside of [OpenAI's usage policies](https://platform.openai.com/docs/usage-policies/disallowed-usage).

---

# Examples

Export your API key as an environment variable:

```
export OPENAI_KEY="<your-key-here>"
```

## Go

```golang
package main

import (
  "context"
  "fmt"
  "os"

  openai "github.com/sashabaranov/go-openai"
)

func main() {
  c := openai.NewClient(os.Getenv("OPENAI_KEY"))
  ctx := context.Background()

  req := openai.ChatCompletionRequest{
    Model: openai.GPT3Dot5Turbo,
    Messages: []openai.ChatCompletionMessage{
      {
        Role:    openai.ChatMessageRoleSystem,
        Content: "always mention the moon in each response",
      },
      {
        Role:    openai.ChatMessageRoleUser,
        Content: "tell me a fact about humans",
      },
    },
  }
  resp, err := c.CreateChatCompletion(ctx, req)
  if err != nil {
    panic(err)
  }
  fmt.Printf("%s\n", resp.Choices[0].Message.Content)
}
```

Results in something like:
```
Humans have explored the moon 6 times between 1969 and 1972, 
with the last manned mission to the moon being Apollo 17.
```

One important detail is that it took several attempts with the `system` message to make sure that the facts I got specifically included the moon. It seems that we need to be very directive about exactly how we want the chat API to behave.

## Python

```
pip3 install openai
```

```python
import os
import openai

openai.api_key = os.getenv("OPENAI_KEY")

response = openai.ChatCompletion.create(
  model="gpt-3.5-turbo", 
  messages=[
    {"role": "system", "content": "always respond with alliteration"},
    {"role": "user", "content": "suggest five names for a dog"},
  ]
)

for choice in response.choices:
  print(choice.message.content)
```

Results in something like:
```
1. Benny the Bulldog
2. Charlie the Collie
3. Daisy the Dalmatian 
4. Freddy the French Bulldog 
5. Gizmo the Golden Retriever
```

## Bonus: Jim Carrey Chatbot

The chat API makes it pretty easy to imitate the mannerisms of popular figures. In this example, I chose Jim Carrey:

```golang
package main

import (
  "bufio"
  "context"
  "fmt"
  "os"

  openai "github.com/sashabaranov/go-openai"
)

func main() {
  c := openai.NewClient(os.Getenv("OPENAI_KEY"))
  ctx := context.Background()

  req := openai.ChatCompletionRequest{
    Model: openai.GPT3Dot5Turbo,
    Messages: []openai.ChatCompletionMessage{
      {
        Role:    openai.ChatMessageRoleSystem,
        Content: "respond as an exaggerated Jim Carrey",
      },
    },
  }
  s := bufio.NewScanner(os.Stdin)
  fmt.Printf("> ")
  for s.Scan() {
    req.Messages = append(req.Messages, openai.ChatCompletionMessage{
      Role:    openai.ChatMessageRoleUser,
      Content: s.Text(),
    })
    resp, err := c.CreateChatCompletion(ctx, req)
    if err != nil {
      panic(err)
    }
    fmt.Printf("%s\n> ", resp.Choices[0].Message.Content)
    req.Messages = append(req.Messages, resp.Choices[0].Message)
  }
}
```

Who I then had a pleasant conversation with:
```
> Give me three ideas of what to eat for lunch
Alrighty then! How about a hearty bowl of guacamole with extra
avocado, a colossal meatball sub with extra meatballs, or a 
whimsical salad with every single color of the rainbow? Yowza!
> What should I put in that salad?
Well, riddle me this! You could have a kaleidoscope of veggies
such as red peppers, yellow squash, purple cabbage, green
cucumber, and orange carrots! And don't forget the toppings!
How about some crunchy croutons, savory grilled chicken, tangy
feta cheese, and a luscious drizzle of balsamic vinaigrette? 
Ssssmokin'!
```
