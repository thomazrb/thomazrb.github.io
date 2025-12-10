---
title: "Guia Completo da WT32-SC01 Plus (parte 2 de 6): O Primeiro Código (Hello World!)"
date: 2025-08-25T16:40:00-03:00
lastmod: 2025-12-10T10:16:00-03:00
draft: false
tags: ["ESP32-S3", "WT32-SC01 Plus", "LovyanGFX", "Arduino", "Hello World"]
categories: ["Hardware", "ESP32"]
---

No primeiro post desta série, conhecemos a placa **WT32-SC01 Plus** e suas principais características. Agora, é hora de colocar a mão na massa e fazer o que mais gostamos: escrever código e ver algo acontecer na tela!

Neste tutorial, vamos configurar o ambiente de desenvolvimento no Arduino IDE e criar nosso primeiro programa: o clássico "Hello World!". O objetivo aqui é garantir que a comunicação com o display esteja funcionando perfeitamente, sem a complexidade de bibliotecas de interface como a LVGL. Para isso, usaremos a biblioteca **LovyanGFX**.

Embora existam outras excelentes bibliotecas para controle de displays, como a popular `TFT_eSPI` ou a `Arduino_GFX`, optamos pela `LovyanGFX` neste tutorial por uma razão principal: ela oferece uma solução integrada, controlando tanto o display quanto o toque em uma única biblioteca. Isso simplifica a configuração e o código, como veremos nos próximos posts.

### Passo 1: Configurando o Ambiente no Arduino IDE

Antes de escrever qualquer código, precisamos garantir que o Arduino IDE esteja preparado para conversar com a nossa placa e com a biblioteca gráfica.

#### 1.1 Instalar o Pacote de Placas ESP32

A compatibilidade entre as bibliotecas e o "core" do ESP32 é crucial. Versões mais recentes da série 3.x.x apresentaram erros de compilação com a biblioteca LovyanGFX, por isso, como descobrimos em nossos testes, a versão mais estável para este projeto é a **2.0.17**.

1.  No Arduino IDE, vá em **Arquivo > Preferências**.
2.  No campo "URLs Adicionais de Gerenciadores de Placas", adicione a seguinte URL:
    ```
    https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
    ```
3.  Vá em **Ferramentas > Placa > Gerenciador de Placas...**.
4.  Pesquise por `esp32`, selecione "esp32 by Espressif Systems", escolha a versão **2.0.17** e clique em "Instalar".

#### 1.2 Instalar a Biblioteca LovyanGFX

Como já mencionado, essa biblioteca será a responsável por controlar o display.

1.  Vá em **Ferramentas > Gerenciar Bibliotecas...**.
2.  Pesquise por `LovyanGFX` e instale a versão mais recente.

### Passo 2: O Código do "Hello World!"

Com o ambiente pronto, crie um novo sketch no Arduino IDE e cole o código completo abaixo. Este código já contém toda a configuração de hardware necessária para a nossa placa.

```cpp
/*
 * EXEMPLO 1: HELLO WORLD COM LOVYANGFX
 * * Este sketch demonstra o uso básico da biblioteca LovyanGFX para
 * inicializar a tela da placa WT32-SC01 Plus e escrever um texto.
 */

#define LGFX_USE_V1 // Define que estamos usando a Versão 1 da LovyanGFX

#include <LovyanGFX.hpp>

// ======================================================================================
// === 1. CONFIGURAÇÃO DE HARDWARE (LOVYANGFX) ==========================================
// ======================================================================================
// Esta classe descreve o hardware da tela para a biblioteca LovyanGFX.
class LGFX : public lgfx::LGFX_Device
{
  // Instâncias dos drivers para cada componente de hardware da tela
  lgfx::Panel_ST7796  _panel_instance;
  lgfx::Bus_Parallel8 _bus_instance;
  lgfx::Light_PWM     _light_instance;

public:
  LGFX(void)
  {
    // --- Configuração do Barramento da Tela (BUS) ---
    { 
      auto cfg = _bus_instance.config();
      cfg.port = 0;
      cfg.freq_write = 20000000;
      cfg.pin_wr = 47;
      cfg.pin_rd = -1;
      cfg.pin_rs = 0;
      cfg.pin_d0 = 9; cfg.pin_d1 = 46; cfg.pin_d2 = 3; cfg.pin_d3 = 8;
      cfg.pin_d4 = 18; cfg.pin_d5 = 17; cfg.pin_d6 = 16; cfg.pin_d7 = 15;
      _bus_instance.config(cfg);
      _panel_instance.setBus(&_bus_instance);
    }

    // --- Configuração do Painel da Tela (PANEL) ---
    { 
      auto cfg = _panel_instance.config();
      cfg.pin_cs = -1;
      cfg.pin_rst = 4;
      cfg.pin_busy = -1;
      cfg.memory_width = 320;
      cfg.memory_height = 480;
      cfg.panel_width = 320;
      cfg.panel_height = 480;
      cfg.offset_x = 0;
      cfg.offset_y = 0;
      // Combinação que corrige as cores para este modelo de tela
      cfg.invert = true;
      cfg.rgb_order = false;
      _panel_instance.config(cfg);
    }

    // --- Configuração da Luz de Fundo (BACKLIGHT) ---
    { 
      auto cfg = _light_instance.config();
      cfg.pin_bl = 45;
      cfg.invert = false;
      cfg.freq = 44100;
      cfg.pwm_channel = 7;
      _light_instance.config(cfg);
      _panel_instance.setLight(&_light_instance);
    }
    
    setPanel(&_panel_instance);
  }
};

// Instância principal do driver LovyanGFX que será usada no código
LGFX gfx; 

void setup()
{
    Serial.begin(115200);
    Serial.println("Iniciando Exemplo 1: Hello World com LovyanGFX");

    // --- 1. Inicializa o Hardware ---
    gfx.begin();
    gfx.setRotation(1); // 1 = Paisagem (cabo USB para a direita)
    gfx.setBrightness(255); // Brilho máximo

    // --- 2. Desenha na Tela ---
    gfx.fillScreen(TFT_BLACK); // Pinta o fundo de preto

    gfx.setTextSize(4); // Define o tamanho da fonte (escala 4x)
    gfx.setTextColor(TFT_WHITE); // Define a cor do texto como branco
    
    // Centraliza o texto na tela
    const char* text = "Hello World!";
    int16_t textWidth = gfx.textWidth(text);
    int16_t textHeight = gfx.fontHeight();
    int16_t cursorX = (gfx.width() - textWidth) / 2;
    int16_t cursorY = (gfx.height() - textHeight) / 2;
    
    gfx.setCursor(cursorX, cursorY); // Posiciona o cursor para o texto
    gfx.println(text); // Imprime o texto na tela

    Serial.println("Texto 'Hello World!' escrito na tela.");
}

void loop()
{
  // O loop fica vazio, pois a ação acontece apenas uma vez no setup.
  delay(1000);
}
```
### Passo 3: Entendendo o Código

* **Classe `LGFX`**: Este bloco inicial é a parte mais importante. Ele funciona como um "manual de instruções" para a biblioteca, descrevendo exatamente como o hardware da WT32-SC01 Plus é montado: quais pinos são usados para a comunicação com a tela, qual a resolução, como corrigir as cores, etc.
* **`setup()`**: Aqui é onde a mágica acontece.
    * `gfx.begin()`: Inicializa a tela com as configurações que definimos.
    * `gfx.setRotation(1)`: Gira a tela para o modo paisagem.
    * `gfx.fillScreen(TFT_BLACK)`: Pinta toda a tela de preto.
    * `gfx.setTextSize(4)` e `gfx.setTextColor(TFT_WHITE)`: Configuram a aparência do nosso texto.
    * **Centralização**: O bloco seguinte calcula a posição exata para que o texto "Hello World!" fique perfeitamente centralizado.
    * `gfx.println(text)`: Finalmente, desenha o texto na tela.
* **`loop()`**: Como nosso objetivo é apenas escrever o texto uma vez, o loop principal fica vazio.

### Conclusão

Parabéns! Se tudo correu bem, você agora tem a frase "Hello World!" brilhando na tela da sua WT32-SC01 Plus. Isso confirma que seu ambiente está configurado corretamente e que a comunicação com o display está 100% funcional.

No próximo post, vamos dar um passo adiante: vamos introduzir a biblioteca **LVGL** e a ferramenta **SquareLine Studio** para criar nossa primeira interface gráfica, ainda sem toque, mas já mostrando o poder dessas ferramentas.

Até lá!