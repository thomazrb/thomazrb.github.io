---
title: "Criando e usando um arquivo requirements.txt para instalar pacotes Python com o PIP"
date: 2023-05-14T20:24:21-03:00
draft: false
tags: ["Python", "pip", "requirements.txt"]
categories: ["Python"]
ShowToc: true
---

## Introdução

Se você é um desenvolvedor Python em busca de uma forma eficiente de gerenciar pacotes externos em seus projetos, veio ao lugar certo! Neste guia passo a passo, vou ensinar como criar e usar um arquivo requirements.txt, aproveitando o poderoso gerenciador de pacotes PIP. Essa é uma prática recomendada que permite compartilhar facilmente ambientes de desenvolvimento e garante a instalação adequada de todos os pacotes necessários. Então, vamos começar!

## Passo 1: Criando o arquivo requirements.txt

Para começar, abra o terminal ou prompt de comando e navegue até o diretório do seu projeto. Em seguida, digite o seguinte comando para criar o arquivo requirements.txt:

```console
touch requirements.txt
```

O comando "touch requirements.txt" cria um arquivo vazio chamado "requirements.txt" no diretório atual. Ele atualiza o carimbo de data/hora do arquivo ou cria o arquivo se ele não existir, sem adicionar qualquer conteúdo ao mesmo. Esse comando é comumente usado para criar um arquivo rapidamente antes de adicionar o conteúdo desejado posteriormente.



## Passo 2: Listando os pacotes no arquivo requirements.txt

Abra o arquivo requirements.txt em um editor de texto e adicione os pacotes necessários, um por linha. Cada linha deve seguir o formato pacote==versão, onde "pacote" é o nome do pacote que você deseja instalar e "versão" é a versão específica do pacote (opcional, mas recomendado para evitar problemas de compatibilidade). Por exemplo:

```plain
package1==1.0.0
package2==2.3.1
```
Adicione todos os pacotes necessários dessa maneira, garantindo que cada pacote esteja em uma linha separada.

## Passos 1 e 2: Alternativa recomendada

Alternativamente, se você já tiver os pacotes instalados em seu ambiente, pode usar o seguinte comando para gerar automaticamente o arquivo requirements.txt:

```console
pip freeze > requirements.txt
```

O comando "pip freeze > requirements.txt" gera um arquivo requirements.txt capturando a lista de pacotes Python instalados e suas respectivas versões no ambiente atual. Ele redireciona a saída do comando "pip freeze", que exibe os nomes e versões dos pacotes, e salva no arquivo requirements.txt.

## Passo 3: Instalando os pacotes a partir do arquivo requirements.txt

Se você acabou de obter os arquivos de um projeto e precisa instalar os pacotes a partir do arquivo requirements.txt, pode usar o PIP para instalar todos os pacotes de uma vez. No seu terminal ou prompt de comando, execute o seguinte comando:

```console
pip install -r requirements.txt
```
O PIP irá ler o arquivo requirements.txt e instalará automaticamente todos os pacotes listados.

## Passo 4: Verificando a instalação dos pacotes

Após a conclusão da instalação, é uma boa prática verificar se os pacotes foram instalados corretamente. Você pode fazer isso executando o seguinte comando:

```console
pip freeze
```

Isso exibirá a lista de pacotes instalados, juntamente com suas versões, no terminal.

## Conclusão

Parabéns! Agora você sabe como criar e usar um arquivo requirements.txt para instalar pacotes com o PIP. Essa prática é extremamente útil ao compartilhar seu ambiente de desenvolvimento com outros desenvolvedores ou ao configurar um novo espaço de trabalho.

Lembre-se de manter o arquivo requirements.txt atualizado sempre que adicionar, remover ou atualizar pacotes em seu projeto. Isso garante consistência entre os ambientes e evita problemas de compatibilidade.

## Tutorial em vídeo

Se preferir um vídeo, tenho este conteúdo disponível em um vídeo no YouTube:

{{< youtube dSM32RLHwM8 >}}
