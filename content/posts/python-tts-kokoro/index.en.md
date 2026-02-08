---
title: "Generating Audio from Text with Local AI Using Kokoro-82M"
date: 2026-02-08T14:05:24-03:00
draft: false
tags: ["TTS", "Kokoro", "Python", "AI"]
categories: ["Python"]
ShowToc: true
---

Imagine being able to turn any text into audio with a natural and realistic voice, all running locally on your machine, without relying on cloud services or paid APIs. That's exactly what **Kokoro-82M** offers.

Kokoro is an open-weight Text-to-Speech (TTS) model with only 82 million parameters. Despite being lightweight, it delivers audio quality comparable to much larger models, while being significantly faster and more cost-efficient. With an Apache license, it can be used in both personal projects and production environments.

In this tutorial, we'll create a simple Python script that converts text into a `.wav` audio file using Kokoro with **Brazilian Portuguese** voices.

### Prerequisites

This tutorial was tested with **Python 3.12.12** on **macOS**, but it should work on any operating system with Python 3.10 to 3.12.

### Installing the Python dependencies

With your virtual environment active, install the required libraries:

```console
pip install kokoro soundfile
```

The `kokoro` package is the model's inference library, and `soundfile` is responsible for saving the generated audio to a file.

### Available voices for Brazilian Portuguese

Kokoro offers three voices for pt-br:

| Voice | Gender |
|---|---|
| `pf_dora` | Female |
| `pm_alex` | Male |
| `pm_santa` | Male |

The `p` prefix indicates the language (Brazilian Portuguese) and the following letter indicates the gender (`f` for female, `m` for male).

### The code

Create a file called `tts_kokoro.py` and paste the following code:

```python
import warnings
warnings.filterwarnings("ignore", message="dropout option adds dropout")
warnings.filterwarnings("ignore", message=".*weight_norm.*is deprecated")

from kokoro import KPipeline
import soundfile as sf
import numpy as np

pipe = KPipeline(lang_code="p", repo_id="hexgrad/Kokoro-82M")

audio = []
for _, _, chunk in pipe("A inteligência artificial avançou significativamente, "
                        "oferecendo, hoje em dia, vozes extremamente realistas.",
                        voice="pf_dora"):
    audio.append(chunk)

audio = np.concatenate(audio)
sf.write("saida.wav", audio, 24000)
print("Audio generated successfully: saida.wav")
```

Run it with:

```console
python tts_kokoro.py
```

On the first run, the model will be automatically downloaded from Hugging Face (around 330 MB). On subsequent runs, it will be loaded from the local cache.

The result will be a `saida.wav` file with the generated audio.

### Understanding the code

Let's go through each part of the script.

#### Silencing PyTorch warnings

```python
import warnings
warnings.filterwarnings("ignore", message="dropout option adds dropout")
warnings.filterwarnings("ignore", message=".*weight_norm.*is deprecated")
```

These filters must come **before** importing Kokoro. The model internally uses PyTorch components that emit two harmless warnings: one about dropout configuration in LSTM layers and another about the `weight_norm` function being replaced by a newer version. Neither affects the model's functionality, but they clutter the terminal output.

#### Initializing the pipeline

```python
from kokoro import KPipeline

pipe = KPipeline(lang_code="p", repo_id="hexgrad/Kokoro-82M")
```

The `lang_code="p"` parameter indicates that we'll be using Brazilian Portuguese. Each language supported by Kokoro has its own code, such as `"a"` for American English, `"j"` for Japanese, and `"z"` for Mandarin. The `repo_id` points to the model's repository on Hugging Face.

#### Generating the audio

```python
audio = []
for _, _, chunk in pipe("Your text here...", voice="pf_dora"):
    audio.append(chunk)
```

The pipeline processes the text in chunks and returns an iterator with three values per iteration: the graphemes, the phonemes, and the audio for that segment. Since we only need the audio, we ignore the first two values with `_`. Each chunk is accumulated in the list and then concatenated with `np.concatenate()`.

#### Saving the file

```python
audio = np.concatenate(audio)
sf.write("saida.wav", audio, 24000)
```

The audio is generated at 24,000 Hz (24 kHz), which is the model's native sample rate. `soundfile` saves the data in WAV format.

### Experimenting with voices

To switch the voice, simply change the `voice` parameter in the pipeline call. Try all three available voices to find the one that best suits your project:

```python
for _, _, chunk in pipe("Testando a voz masculina do Alex.", voice="pm_alex"):
    audio.append(chunk)
```

### Conclusion

With just a few lines of code and no external services, we were able to generate high-quality audio in Brazilian Portuguese using Kokoro-82M. The model is lightweight, fast, and free, making it an excellent choice for accessibility projects, voice assistants, automated content narration, and much more.

To explore all available voices and languages, check out the [official Kokoro repository on Hugging Face](https://huggingface.co/hexgrad/Kokoro-82M).