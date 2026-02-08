---
title: "Gerando Áudio a Partir de Texto com IA Local Usando Kokoro-82M"
date: 2026-02-08T14:05:24-03:00
draft: false
tags: ["TTS", "Kokoro", "Python", "IA"]
categories: ["Python"]
ShowToc: true
---

Imagine poder transformar qualquer texto em áudio com uma voz natural e realista, tudo rodando localmente na sua máquina, sem depender de serviços na nuvem ou APIs pagas. Isso é exatamente o que o **Kokoro-82M** oferece.

O Kokoro é um modelo open-weight de Text-to-Speech (TTS) com apenas 82 milhões de parâmetros. Apesar de ser leve, ele entrega uma qualidade de áudio comparável a modelos muito maiores, sendo significativamente mais rápido e econômico. Com licença Apache, ele pode ser usado tanto em projetos pessoais quanto em ambientes de produção.

Neste tutorial, vamos criar um script Python simples que converte texto em um arquivo de áudio `.wav` usando o Kokoro com vozes em **português do Brasil**.

### Pré-requisitos

Este tutorial foi testado com **Python 3.12.12** no **macOS**, mas deve funcionar em qualquer sistema operacional com Python 3.10 a 3.12.

### Instalando as dependências Python

Com seu ambiente virtual ativo, instale as bibliotecas necessárias:

```console
pip install kokoro soundfile
```

O pacote `kokoro` é a biblioteca de inferência do modelo e o `soundfile` é responsável por salvar o áudio gerado em arquivo.

### Vozes disponíveis em português do Brasil

O Kokoro oferece três vozes para pt-br:

| Voz | Gênero |
|---|---|
| `pf_dora` | Feminino |
| `pm_alex` | Masculino |
| `pm_santa` | Masculino |

O prefixo `p` indica o idioma (português brasileiro) e a letra seguinte indica o gênero (`f` para feminino, `m` para masculino).

### O código

Crie um arquivo chamado `tts_kokoro.py` e cole o seguinte código:

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
print("Áudio gerado com sucesso: saida.wav")
```

Execute com:

```console
python tts_kokoro.py
```

Na primeira execução, o modelo será baixado automaticamente do Hugging Face (cerca de 330 MB). Nas execuções seguintes, ele será carregado do cache local.

O resultado será um arquivo `saida.wav` com o áudio gerado.

### Entendendo o código

Vamos analisar cada parte do script.

#### Silenciando warnings do PyTorch

```python
import warnings
warnings.filterwarnings("ignore", message="dropout option adds dropout")
warnings.filterwarnings("ignore", message=".*weight_norm.*is deprecated")
```

Esses filtros precisam vir **antes** do import do Kokoro. O modelo utiliza internamente componentes do PyTorch que emitem dois warnings inofensivos: um sobre configuração de dropout em camadas LSTM e outro sobre a função `weight_norm` sendo substituída por uma versão mais nova. Ambos não afetam o funcionamento do modelo, mas poluem a saída do terminal.

#### Inicializando o pipeline

```python
from kokoro import KPipeline

pipe = KPipeline(lang_code="p", repo_id="hexgrad/Kokoro-82M")
```

O parâmetro `lang_code="p"` indica que vamos usar português brasileiro. Cada idioma suportado pelo Kokoro possui um código próprio, como `"a"` para inglês americano, `"j"` para japonês e `"z"` para mandarim. O `repo_id` aponta para o repositório do modelo no Hugging Face.

#### Gerando o áudio

```python
audio = []
for _, _, chunk in pipe("Seu texto aqui...", voice="pf_dora"):
    audio.append(chunk)
```

O pipeline processa o texto em partes (chunks) e retorna um iterador com três valores por iteração: os grafemas, os fonemas e o áudio do trecho. Como precisamos apenas do áudio, ignoramos os dois primeiros valores com `_`. Cada chunk é acumulado na lista e depois concatenado com `np.concatenate()`.

#### Salvando o arquivo

```python
audio = np.concatenate(audio)
sf.write("saida.wav", audio, 24000)
```

O áudio é gerado a 24.000 Hz (24 kHz), que é a taxa de amostragem nativa do modelo. O `soundfile` salva os dados no formato WAV.

### Experimentando com as vozes

Para trocar a voz, basta alterar o parâmetro `voice` na chamada do pipeline. Experimente as três vozes disponíveis para encontrar a que melhor se adapta ao seu projeto:

```python
for _, _, chunk in pipe("Testando a voz masculina do Alex.", voice="pm_alex"):
    audio.append(chunk)
```

### Conclusão

Com poucas linhas de código e sem depender de serviços externos, conseguimos gerar áudio de alta qualidade em português do Brasil usando o Kokoro-82M. O modelo é leve, rápido e gratuito, o que o torna uma excelente opção para projetos de acessibilidade, assistentes de voz, narração automática de conteúdo e muito mais.

Para explorar todas as vozes e idiomas disponíveis, confira o [repositório oficial do Kokoro no Hugging Face](https://huggingface.co/hexgrad/Kokoro-82M).