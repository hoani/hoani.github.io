---
title: "OpenAI - Voice"
excerpt: "Speech-to-text transcription and translation using OpenAI's API"
toc: true
permalink: "/guides/engineering/openai/voice"
categories:
  - guide
  - software
  - engineering
  - openai
  - golang
  - python
---

OpenAI's voice APIs allow us to transcribe voice as well as translate various languages to English.

```
[audio file] -> OpenAI Voice API -> [transcription]
```

# Http API

Using CURL we can ask the `voice` API to transcribe audio:

```
export OPENAI_KEY="<your-key-here>"
curl https://api.openai.com/v1/audio/transcriptions \
  -X POST \
  -H 'Authorization: Bearer '$OPENAI_KEY'' \
  -H 'Content-Type: multipart/form-data' \
  -F file=@recording.mp3 \
  -F model=whisper-1
```

The response is:
```json
{"text":"Good morning."}
```

It also works in various languages. For example, when running on a clip from the popular Demon Slayer anime:

```
curl https://api.openai.com/v1/audio/transcriptions \
  -X POST \
  -H 'Authorization: Bearer '$OPENAI_KEY'' \                                      
  -H 'Content-Type: multipart/form-data' \
  -F file=@inosuke.mp3 \
  -F model=whisper-1
```

We get the response:
```json
{"text":"すごいだろう俺はすごいだろう俺は 2回言ってる 4月3日"}
```

We can also translate many languages from voice to English text:

```
curl https://api.openai.com/v1/audio/translations \  
  -X POST \                    
  -H 'Authorization: Bearer '$OPENAI_KEY'' \
  -H 'Content-Type: multipart/form-data' \
  -F file=@inosuke.mp3 \  
  -F model=whisper-1
```

With the translated transcription:
```json
{"text":"I'm amazing, right? I'm amazing, right? He said it twice... He said it three times..."}
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

  req := openai.AudioRequest{
    Model:    openai.Whisper1,
    FilePath: "./recording.mp3",
  }
  resp, err := c.CreateTranscription(ctx, req)
  if err != nil {
    fmt.Printf("Transcription error: %v\n", err)
    return
  }
  fmt.Println(resp.Text)
}
```

For translations use `c.CreateTranslation` instead.

## Python

```
pip3 install openai
```

```python
import os
import openai

openai.api_key = os.getenv("OPENAI_KEY")

audio_file= open("./recording.mp3", "rb")
transcript = openai.Audio.transcribe("whisper-1", audio_file)
print(transcript)
```

To translate use `openai.Audio.translate` instead.
