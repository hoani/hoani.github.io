---
title: "OpenAI - Moderation"
excerpt: "Moderating text input with OpenAI's API"
toc: true
permalink: "/guides/engineering/openai/moderation"
categories:
  - guide
  - software
  - engineering
  - openai
  - golang
  - python
---

OpenAI have strict controls over what content its AI models will interact with. The moderation API lets us check text inputs for content violation.

```
[text] -> OpenAI Moderation API -> [Content Scoring]
```

# Http API

Using CURL we can ask the `moderations` API to score content:

```
export OPENAI_KEY="<your-key-here>"
curl https://api.openai.com/v1/moderations \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer '$OPENAI_KEY'' \
  -d '{
  "input": "I would like a new car"
}'
```

The response is:
```json
{
  "id":"modr-6qyDMAOxhJMZXZom7op0Ted92pvuS",
  "model":"text-moderation-004",
  "results":[
    {
      "flagged":false,
      "categories":{
        "sexual":false,
        "hate":false,
        "violence":false,
        "self-harm":false,
        "sexual/minors":false,
        "hate/threatening":false,
        "violence/graphic":false
      },
      "category_scores":{
        "sexual":2.017724085590089e-07,
        "hate":1.2655793568683293e-07,
        "violence":3.475070968761429e-07,
        "self-harm":1.627412316018706e-09,
        "sexual/minors":9.74211200599484e-09,
        "hate/threatening":1.0224529001234828e-09,
        "violence/graphic":5.61986723823793e-09
      }
    }
  ]
}
```

## API Key

To get an API Key, create an OpenAI account, you can generate tokens under your profile.

<figure>
    <img src="/assets/images/posts/guides/openai/000_apikey.png">
</figure>

# Examples

Export your API key as an environment variable:

```
export OPENAI_KEY="<your-key-here>"
```

## Go

In this example, I format the results as a JSON string and print that to the console. Alternatively, you can access each moderation category by name.

```golang
package main

import (
  "bytes"
  "context"
  "encoding/json"
  "fmt"

  openai "github.com/sashabaranov/go-openai"
)

func main() {
  c := openai.NewClient(os.Getenv("OPENAI_KEY"))
  ctx := context.Background()

  req := openai.ModerationRequest{
    Input: "torturing batman with the titanic soundtrack",
  }
  resp, err := c.Moderations(ctx, req)
  if err != nil {
    panic(err)
  }
  b, err := json.Marshal(resp)
  if err != nil {
    panic(err)
  }
  indented := bytes.NewBuffer([]byte{})
  err = json.Indent(indented, b, "", "\t")
  if err != nil {
    panic(err)
  }
  fmt.Println(indented.String())
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

response = openai.Moderation.create(
    input="I only play indie games", 
)
print(response)
```
