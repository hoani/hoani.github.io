---
title: "OpenAI - Completion"
excerpt: "Text completion using OpenAI's API"
toc: true
permalink: "/guides/engineering/openai/completion"
categories:
  - guide
  - software
  - engineering
  - openai
  - golang
  - python
---

Completions allow us to expand on a prompt.

```
[prompt] -> OpenAI Completion API -> [completion]
```

# Http API

Using CURL we can ask the `completions` API to write a haiku:

```
export OPENAI_KEY="<your-key-here>"
curl https://api.openai.com/v1/completions \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer '$OPENAI_KEY'' \
  -d '{
  "model": "text-davinci-003",
  "prompt": "Write a haiku about making a new friend",
  "max_tokens": 128
}'
```

The response is:
```json
{
  "id":"cmpl-6qE3atUfkOVhEcED7uaWXqr046fzV",
  "object":"text_completion",
  "created":1677905110,
  "model":"text-davinci-003",
  "choices":[
    {
      "text":"\n\nA fleeting bond form,\nTalking, and laughs shared;\nA friend made today.",
      "index":0,
      "logprobs":null,
      "finish_reason":"stop"
    }
  ],
  "usage":{
    "prompt_tokens":9,
    "completion_tokens":20,
    "total_tokens":29
  }
}
```

Which contains the following result:
```
A fleeting bond form,
Talking, and laughs shared;
A friend made today.
```

Running this multiple times will yield different results.

## API Key

To get an API Key, create an OpenAI account, you can generate tokens under your profile.

<figure>
    <img src="/assets/images/posts/guides/openai/000_apikey.png">
</figure>

## Parameters

There are several parameters we can manipulate when we make a call to `completions` API.

### model

`model` lets us choose which model we want to use. Additional details are in the [openai docs](https://platform.openai.com/docs/models).

The codex models can be used for code completions. For example, the following will write some python code:

```
curl https://api.openai.com/v1/completions \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer '$OPENAI_KEY'' \
  -d '{
  "model": "code-davinci-002",
  "prompt": "# Writes a greeting to the console depending on the time of day",
  "temperature": 0,
  "max_tokens": 256,
  "top_p": 1
}'
```

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

  req := openai.CompletionRequest{
    Model:     openai.GPT3TextDavinci003,
    MaxTokens: 1024,
    Prompt:    "Write a haiku about a sleeping cat",
  }
  resp, err := c.CreateCompletion(ctx, req)
  if err != nil {
    panic(err)
  }
  for i, choice := range resp.Choices {
    fmt.Printf("%d:\n%s\n", i, choice.Text)
  }
}
```

## Python

```
pip3 install openai
```

```python
import os
import openai

openai.api_key = os.getenv("OPENAI_KEY")

response = openai.Completion.create(
    model="text-davinci-003", 
    max_tokens=1024,
    prompt="Write a haiku about a sleeping cat", 
)

for choice in response.choices:
    print(choice.text)
```

