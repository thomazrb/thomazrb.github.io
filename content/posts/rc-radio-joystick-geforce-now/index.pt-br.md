---
title: "Usando um Rádio de Aeromodelo como Joystick no MSFS 2024 pelo GeForce NOW"
date: 2026-08-03T07:30:00-03:00
draft: false
tags: ["GeForce NOW", "MSFS 2024", "EdgeTX", "Joystick", "Flight Simulator", "macOS", "Windows"]
categories: ["Gaming"]
---

Rádio de aeromodelo é, sem exagero, um dos melhores controles de voo caseiros que existem: o acelerador não tem mola e fica na posição em que você o deixa, enquanto os manches voltam ao centro sozinhos. É a ergonomia de uma cabine de verdade. O desafio é usar isso no **Microsoft Flight Simulator 2024 rodando pelo GeForce NOW**, e é aí que a maioria das pessoas trava.

**Testado com:** um Flysky ProArt PA-01 (rodando EdgeTX) em um Mac com chip M4 (Apple Silicon), MSFS 2024 via GeForce NOW. Mas o método vale para qualquer rádio com EdgeTX ou OpenTX, e também para Windows nativo.

### Por que não funciona de primeira

O GeForce NOW só encaminha controles no padrão **XInput** (o dos joysticks Xbox). Um rádio de aeromodelo ligado por USB aparece para o sistema como um **joystick genérico** (DirectInput, sem mapeamento padrão). O resultado é que o navegador ou o aplicativo até enxergam o rádio, mas o GeForce NOW o ignora: dentro do MSFS, a lista de dispositivos mostra apenas teclado e mouse.

A solução é fazer o rádio **se passar por um controle Xbox** (XInput padrão). No **Windows** isso é direto. No **Mac** há um passo a mais (uma máquina virtual com Windows), porque o macOS não permite criar um controle virtual.

### Pré-requisitos

1. **Um rádio com EdgeTX ou OpenTX** e modo USB Joystick (praticamente todos têm).
2. **Cabo USB.**
3. **Conta no GeForce NOW** e o **MSFS 2024** na sua biblioteca.
4. **ViGEmBus e XOutput** (ambos gratuitos).
5. **Apenas no Mac:** uma **máquina virtual com Windows 11** (Parallels, UTM ou VMware Fusion).

### Passo 1: Preparando o Rádio (Windows e Mac)

Crie um **modelo limpo** no rádio (no EdgeTX: New Model → Blank Model).

**Importante:** se você reaproveitar um modelo com mixagens (asa-delta, mixagem de tanque etc.), os eixos chegam misturados e com curso pela metade. Um modelo em branco envia cada stick para o seu canal, sem mistura: CH1 = aileron, CH2 = profundor, CH3 = acelerador, CH4 = leme.

Em seguida, coloque o rádio em modo **USB Joystick** (não "USB Storage") e teste em **hardwaretester.com/gamepad**: o rádio deve aparecer com os eixos respondendo aos movimentos.

### Passo 2: A Máquina Virtual (apenas no Mac)

Se você está no **Windows nativo, pule para o Passo 3.**

No **Mac** (Apple Silicon), o macOS não permite criar um controle virtual, então executamos a solução dentro de uma máquina virtual com Windows:

1. Crie uma **VM com Windows 11 (ARM)**.
2. Faça o **USB passthrough** do rádio para dentro da VM (ela precisa "enxergar" o rádio no gamepad tester).
3. **Daqui em diante, faça tudo dentro da VM.**

Sim, é streaming rodando dentro de uma máquina virtual. Na prática, a latência é imperceptível, dá para voar tranquilamente.

### Passo 3: Transformando o Rádio em um Controle Xbox Virtual

1. Instale o **ViGEmBus**, o driver que cria o controle virtual.

   **Atenção:** no Windows ARM (a VM do Mac), baixe o instalador que traz o build **arm64** (`ViGEmBus_..._x64_x86_arm64.exe`).

2. Instale e abra o **XOutput**. Ele listará o rádio na seção DirectInput.

3. Clique em **Add controller** e depois em **Edit**, e mapeie os eixos do rádio para os eixos do Xbox:
   - Left Stick X ← aileron
   - Left Stick Y ← profundor
   - Right Stick Y ← acelerador
   - Right Stick X ← leme

4. Clique em **Start**.

5. Confira novamente em **hardwaretester.com/gamepad**: agora aparece um **"Xbox 360 Controller"** com mapeamento **standard**. É esse que o GeForce NOW aceita.

### Passo 4: Mapeando no MSFS 2024

Abra o GeForce NOW, entre no MSFS e vá em **Configurações → Controles** com o **"Xbox 360 Controller"** selecionado. Ao começar a configurar, o MSFS cria um perfil para você. Basta aceitar.

O **aileron e o profundor já vêm mapeados** nos sticks por padrão, então não é preciso mexer neles. Você vai configurar apenas o **acelerador** e o **leme**:

- Acelerador → **EIXO DO MANETE DE POTÊNCIA**
- Leme → **EIXO DO LEME** (é o que esterça o avião na pista)

**Limpe os conflitos antes de deixar o mapeamento final.** Aqui está o detalhe que economiza horas: o stick direito também controla a câmera, então o acelerador e o leme entram em conflito com ela. Quando você seleciona o **eixo inteiro** (o ícone do stick com seta para cima e para baixo), o MSFS mostra alguns conflitos. Mas existem outros **silenciosos**, que só aparecem quando você aponta cada **direção separada**: a seta para cima e a seta para baixo, individualmente.

Por isso, passe o eixo pelas **três formas** (o eixo completo, a direção para cima e a direção para baixo), apagando os conflitos que cada uma revelar. Só depois de eliminar os três é que você atribui o mapeamento correto. Faça o mesmo para o leme (no stick horizontal: o eixo completo, a direção para a esquerda e a direção para a direita). Se você limpar apenas o eixo completo, sobra um conflito escondido e a câmera continua se mexendo junto.

Por fim:

- Reduza a **zona morta** do acelerador para zero (o padrão vem em cerca de 0,14 e cria um "buraco" no meio do curso).
- Se o **acelerador estiver invertido** (empurrar para frente corta a potência), marque a opção de **inverter o eixo**.

### Conclusão

Com esses passos, o rádio de aeromodelo se transforma em uma cabine de Cessna: acelerador analógico parado na potência exata que você quer, manche com retorno ao centro e leme esterçando o avião na pista. No Windows são três passos; no Mac, os mesmos três dentro de uma máquina virtual. Bom voo!
