---
title: "Ambientes Virtuais em Python"
date: 2023-05-01T14:59:20-03:00
draft: false
tags: ["Virtual Environment", "venv", "Python"]
categories: ["Python"]
ShowToc: true
---

## Ambientes Virtuais

O Python se tornou uma das linguagens de programação mais populares do mundo, graças à sua simplicidade, versatilidade e bibliotecas poderosas. Uma das principais características que tornam o Python uma ótima linguagem para se trabalhar é a sua capacidade de criar ambientes virtuais. Ambientes virtuais são ambientes isolados do Python que permitem instalar e gerenciar pacotes e dependências sem afetar outros projetos ou a instalação global do Python em seu sistema.

Neste artigo, vamos percorrer o processo de criação e gerenciamento de ambientes virtuais em Python usando o "venv".

Existem vários gerenciadores de ambiente em Python, mas vamos focar no "venv", pois é muito simples, já vem instalado no Python 3 e é o gerenciador recomendado pela  [Documentação do Python](https://docs.python.org/3/library/venv.html).

## Criando um ambiente virtual

Para criar um ambiente virtual com o "venv", acesse a página do seu projeto e digite o seguinte comando:

```console
python -m venv .venv
```
Isso criará um novo ambiente virtual no diretório atual com o nome ".venv". O comando "python" é usado para invocar o interpretador do Python e a opção "-m" é usada para executar um módulo específico como um script. Neste caso, estamos executando o módulo "venv", como mencionado anteriormente, que está incluído no Python e fornece uma maneira incorporada de criar ambientes virtuais. O ponto (".") antes do nome do diretório especifica que o ambiente virtual será um diretório oculto (no Mac e no Linux). O nome do ambiente virtual, neste caso, ".venv", pode ser alterado para qualquer outro nome de diretório válido.

Os pacotes instalados serão armazenados dentro deste diretório e é importante lembrar de adicionar este diretório ao arquivo ".gitignore" se você estiver usando o Git.

## Ativando um ambiente virtual

Para ativar o ambiente virtual, você deve usar o seguinte comando:

**Windows:**
```console
.venv\Scripts\Activate.ps1
```
**Mac e Linux:**
```console
source .venv/bin/activate
```
É importante observar que, ao fazer isso, você verá o nome do ambiente virtual antes do prompt, entre parênteses, neste caso:

```console
(.venv) %
```
Você instalará todos os pacotes necessários e executará seus scripts enquanto o ambiente virtual estiver ativo. Quaisquer pacotes Python que você instalar ou atualizar serão restritos a este ambiente virtual.

## Desativando um ambiente virtual
Após terminar de trabalhar com o ambiente virtual, você pode desativá-lo com o seguinte comando:

```console
deactivate
```

A ausência do nome do ambiente virtual no prompt do shell indica que a sessão retornou ao estado padrão e o ambiente virtual foi desativado.

## Considerações finais
Sempre que precisar trabalhar em seu projeto, você deverá ativá-lo novamente.

Lembre-se de criar um ambiente virtual para cada projeto que você tiver. Todos eles podem ter o mesmo nome, pois estão em diretórios diferentes. A vantagem de sempre usar o nome ".venv" é que a maioria da documentação online usa esse nome, e o seu arquivo ".gitignore" geralmente já está configurado para ignorar o diretório ".venv".

## Video tutorial

Se preferir um vídeo, tenho este conteúdo disponível em um vídeo no YouTube:

{{< youtube HllKXnlck5A >}}



