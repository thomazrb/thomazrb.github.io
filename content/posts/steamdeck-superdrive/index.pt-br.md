---
title: "Como Usar o SuperDrive da Apple no Steam Deck"
date: 2026-01-06T19:01:37-03:00
draft: false
tags: ["Steam Deck", "SuperDrive", "Linux", "Hardware"]
categories: ["Steam Deck"]
---

O Steam Deck é uma máquina versátil que vai muito além dos jogos digitais. Se você possui um SuperDrive USB da Apple e quer utilizá-lo no seu Steam Deck para ler DVDs, este guia mostrará como configurar tudo de forma rápida e simples.

### Pré-requisitos

Antes de começar, você precisará de:

1. **Um Steam Deck com dock ou hub USB:** O SuperDrive possui conexão USB-A (aquela USB tradicional, não USB-C) e usa o padrão USB 2.0, então você precisará de um dock ou hub com portas USB. O drive funciona bem quando conectado diretamente a portas USB do dock, sejam elas USB 2.0 ou USB 3.0 (as azuis).

2. **Distrobox Ubuntu:** Este tutorial utiliza uma distrobox com Ubuntu para facilitar o processo. A vantagem de usar a distrobox é que você pode instalar pacotes e fazer configurações sem precisar quebrar o sistema somente leitura do Steam Deck, mantendo tudo organizado e seguro. Se você ainda não tem uma configurada, pode instalá-la facilmente através do app **BoxBuddy** disponível na loja Discover do Steam Deck. (Em um futuro tutorial, posso mostrar o processo detalhado de instalação.)

### Passo 1: Instalando as Ferramentas Necessárias

Primeiro, precisamos garantir que o pacote `sg3_utils` está instalado na sua distrobox Ubuntu. Este pacote contém as ferramentas necessárias para enviar comandos SCSI diretamente ao drive.

Abra o terminal de sua distrobox Ubuntu e execute:

```bash
sudo apt update
sudo apt install sg3-utils
```

### Passo 2: Identificando o Drive

Agora precisamos descobrir em qual dispositivo o SuperDrive foi montado. Na maioria dos casos, ele aparecerá como `/dev/sr0`. Para verificar, execute:

```bash
ls /dev/sr*
```

Se você ver `/dev/sr0` listado, está tudo certo! Caso o drive tenha sido montado com outro número (como `/dev/sr1`, `/dev/sr2`, etc.), anote o nome exato do dispositivo que apareceu, pois você precisará usá-lo nos próximos passos.

### Passo 3: Liberando as Permissões

Por padrão, dispositivos de armazenamento exigem permissões especiais de acesso. Precisamos liberar essas permissões tanto para o drive (/dev/sr0) quanto para o dispositivo SCSI genérico (/dev/sg0).

No Konsole do Steam Deck (fora da distrobox), execute:

```bash
sudo chmod 666 /dev/sr0
sudo chmod 666 /dev/sg0
```

**Importante:** Esses comandos precisam ser executados no sistema principal do Steam Deck (SteamOS), não dentro da distrobox.

### Passo 4: Inicializando o SuperDrive

Agora vem a mágica! O SuperDrive da Apple precisa receber um comando especial de inicialização para começar a funcionar. Entre novamente na sua distrobox Ubuntu e execute:

```bash
sudo sg_raw /dev/sr0 EA 00 00 00 00 00 01
```

Se tudo correu bem, você verá a seguinte mensagem:

```
NVMe Result=0x0
```

### Passo 5: Configurando o VLC para Reprodução de DVDs

Agora que o SuperDrive está funcionando, vamos configurar o VLC para reproduzir seus DVDs. O VLC pode ser instalado facilmente através da loja Discover do Steam Deck.

#### Instalando o VLC

1. Abra o **Discover** (a loja de aplicativos do Steam Deck no modo desktop)
2. Procure por **VLC Media Player**
3. Clique em **Instalar**

#### Suporte para DVDs Comerciais

Para reproduzir DVDs comerciais protegidos, você precisará instalar um componente adicional que fornece a biblioteca `libdvdcss`:

1. Ainda na loja **Discover**, procure por **"Commercial DVD support for HandBrake"**
2. Instale este pacote

Este pacote adiciona o suporte necessário para decodificar DVDs comerciais com proteção CSS (Content Scramble System), permitindo que o VLC reproduza praticamente qualquer DVD.

### Pronto para Assistir!

Com tudo configurado, basta inserir um DVD no SuperDrive e abrir o VLC. O player detectará automaticamente o disco e você poderá assistir seus filmes diretamente no Steam Deck!

### Observações Importantes

- **Portas USB:** O SuperDrive utiliza USB-A padrão (USB 2.0) e funciona em qualquer porta USB de dock ou hub quando conectado diretamente.
- **Permissões temporárias:** As permissões concedidas com `chmod` são temporárias e serão resetadas após reiniciar o sistema. Você precisará executar os comandos de permissão novamente se reiniciar o Steam Deck.
- **Reinicialização necessária:** Sempre que você desconectar e conectar o SuperDrive novamente, será necessário executar novamente o comando `sg_raw` do Passo 4 para inicializá-lo (além das permissões do Passo 3, se o sistema tiver sido reiniciado).
- **Compatibilidade:** Este método funciona especificamente com o SuperDrive da Apple. Outros drives USB podem funcionar de forma diferente ou não precisar desses comandos especiais.

### Conclusão

Com apenas alguns comandos, conseguimos transformar o Steam Deck em um leitor de DVDs funcional usando o SuperDrive da Apple. É uma ótima demonstração da flexibilidade do sistema Linux que roda por baixo do SteamOS e das possibilidades que a distrobox oferece para expandir as funcionalidades do console portátil.