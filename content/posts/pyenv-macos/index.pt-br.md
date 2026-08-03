---
title: "Gerenciando Versões do Python no macOS com pyenv"
date: 2026-07-23T13:30:00-03:00
draft: false
tags: ["Python", "pyenv", "macOS", "Terminal"]
categories: ["Python"]
---

O macOS já vem com Python, mas usar o Python do sistema pra seus projetos é pedir problema. O **pyenv** resolve isso de forma elegante: instale quantas versões quiser e escolha qual usar por projeto, sem conflito.

### Instalação

```console
brew install pyenv
```

Depois, adicione ao seu shell (`~/.zshrc` ou `~/.bashrc`):

```bash
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"
```

Recarregue o shell:

```console
source ~/.zshrc
```

### Instalando versões do Python

Liste as versões disponíveis:

```console
pyenv install --list
```

Instale a que quiser:

```console
pyenv install 3.12.10
```

### Versão global

Define qual versão usar por padrão em todo o sistema:

```console
pyenv global 3.12.10
```

### Versão local (por projeto)

Esse é o diferencial do pyenv. Dentro da pasta do projeto, defina uma versão específica:

```console
cd ~/projetos/fooocus
pyenv local 3.10.14
```

Isso cria um arquivo `.python-version` na pasta. A partir daí, qualquer terminal aberto naquele diretório já usa essa versão automaticamente, sem precisar ativar nada.

```console
python --version
# Python 3.10.14
```

### Por que não usar o Python do brew?

O `brew upgrade` pode atualizar o Python e quebrar venvs existentes. Com o pyenv cada versão fica isolada — você controla quando (e se) atualiza.

Para projetos simples que sempre usam a versão mais nova, o brew resolve. Mas se você trabalha com ferramentas que exigem versão específica, o `pyenv local` economiza muito tempo.
