---
title: "Como Ativar e Desativar o Metal HUD no macOS"
date: 2026-07-23T12:52:00-03:00
draft: false
tags: ["macOS", "Metal", "GPU", "Terminal", "Performance"]
categories: ["macOS"]
---

O Metal HUD é um overlay do macOS que exibe métricas de GPU em tempo real: FPS, uso de memória de vídeo, tempo de frame. Aparece diretamente sobre qualquer app Metal. Útil pra monitorar performance em jogos e aplicações gráficas.

![Metal HUD](metal-hud.png)

### Ativar

```console
/bin/launchctl setenv MTL_HUD_ENABLED 1
```

Abra (ou reabra) o app depois de rodar o comando. O HUD aparece no canto da tela.

### Desativar

```console
/bin/launchctl unsetenv MTL_HUD_ENABLED
```

Feche e reabra o app para o HUD sumir.

### Observações

- O comando age no ambiente do sistema inteiro: qualquer app Metal lançado depois vai exibir o HUD.
- Não precisa de reinicialização do macOS, apenas do app alvo.
- Funciona no macOS Ventura e posteriores (e em versões anteriores também).
