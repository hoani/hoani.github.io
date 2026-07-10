---
title: "OpenAI - Image"
excerpt: "Using OpenAI's API to generate images"
toc: true
permalink: "/guides/engineering/openai/image"
categories:
  - guide
  - software
  - engineering
  - openai
  - golang
  - python
---

OpenAI's image API allows us to use the models used for DallE. The API can be used to create, edit and make image variations.

```
[prompt] -> OpenAI Image API -> [image]
```

# Http API

Using CURL we can ask the `image` API to generate an image:

```
export OPENAI_KEY="<your-key-here>"
curl https://api.openai.com/v1/images/generations \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer '$OPENAI_KEY'' \
  -d '{
  "prompt": "a boss monster in pixel art",
  "size": "256x256"
}'
```

The response will look something like:
```json
{
  "created": 1678345154,
  "data": [
    {
      "url": "https://oaidalleapiprodscus...."
    }
  ]
}
```

In my case, the image was:
<figure class="half">
    <img src="/assets/images/posts/guides/openai/images/000_bossmonster.png">
</figure>

## API Key

To get an API Key, create an OpenAI account, you can generate tokens under your profile.

<figure>
    <img src="/assets/images/posts/guides/openai/000_apikey.png">
</figure>

## Parameters

There are several parameters we can manipulate when we make a call to `images` API.

### response_format

This determines whether we get a URL or a base64 encoded message. The go example below demonstrates using base64 which helps when we want to same the image to file immediately.

### size

Currently, the API supports `"256x256"`, `"512x512"` and `"1024x1024"`

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

In this example, we explicitly ask for a base64 response, and save the image to file immediately.

```golang
package main

import (
  "context"
  "encoding/base64"
  "os"

  openai "github.com/sashabaranov/go-openai"
)

func main() {
  c := openai.NewClient(os.Getenv("OPENAI_KEY"))
  ctx := context.Background()

  req := openai.ImageRequest{
    Prompt:         "Pikachu becoming a tennis champion",
    Size:           openai.CreateImageSize512x512,
    ResponseFormat: openai.CreateImageResponseFormatB64JSON,
  }

  resp, err := c.CreateImage(ctx, req)
  if err != nil {
    panic(err)
  }

  b, err := base64.StdEncoding.DecodeString(resp.Data[0].B64JSON)
  if err != nil {
    panic(err)
  }

  f, err := os.Create("pikachamp.png")
  if err != nil {
    panic(err)
  }
  defer f.Close()

  _, err = f.Write(b)
  if err != nil {
    panic(err)
  }
}
```

Some of the results were a little bit offputting, but this one looks good:
<figure class="half">
    <img src="/assets/images/posts/guides/openai/images/001_pikachamp.png">
</figure>

## Python

```
pip3 install openai
```

```python
import os
import openai

openai.api_key = os.getenv("OPENAI_KEY")

response = openai.Image.create(
    prompt="A cartoon rabbit in a snowfield",
    n=1,
    size="512x512"
)
print(response['data'][0]['url'])
```

Resulted in the following image:
<figure class="half">
    <img src="/assets/images/posts/guides/openai/images/002_snowfield.png">
</figure>
