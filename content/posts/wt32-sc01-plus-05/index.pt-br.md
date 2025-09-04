---
title: "Guia Completo da WT32-SC01 Plus (parte 5 de 6): Manipulando Widgets (Contador Interativo)"
date: 2025-09-03T19:33:57-03:00
draft: false
tags: ["ESP32-S3", "WT32-SC01 Plus", "LovyanGFX", "LVGL", "SquareLine Studio", "Widgets", "Eventos"]
categories: ["Hardware", "ESP32"]
---

No post anterior, demos vida à nossa interface ao habilitar o toque e responder a um evento de botão. Agora que sabemos como capturar as interações do usuário, o próximo passo lógico é fazer com que essas interações modifiquem a própria interface.

Neste tutorial, vamos construir uma aplicação um pouco mais complexa e muito mais prática: um **contador digital**. Criaremos uma tela com um número e dois botões, um para incrementar e outro para decrementar esse número. Este exemplo é fundamental porque nos ensina a ler e a escrever em widgets da tela, uma habilidade essencial para qualquer projeto de UI, seja para exibir dados de sensores, ajustar configurações ou qualquer outra aplicação dinâmica.

### Passo 1: Desenhando a Interface do Contador no SquareLine Studio

Vamos começar criando o layout visual da nossa aplicação.

1.  Abra o SquareLine Studio e crie um novo projeto (ou continue um existente).
2.  Na tela principal (`Screen1`), adicione os seguintes widgets:
    * Um **Label** no centro. No painel "Inspector", altere o texto inicial para `0`. Dê a ele um nome fácil de lembrar, como `labelContador`.
    * Um **Button** à esquerda do label. Adicione um **Label** dentro dele com o texto `-`.
    * Um **Button** à direita do label. Adicione um **Label** dentro dele com o texto `+`.
3.  **Configurando os Eventos:** Agora, vamos vincular nossos botões a funções que escreveremos mais tarde.
    * Selecione o botão `-`. Na seção **Events**, adicione um evento com `Trigger: CLICKED` e `Function: decrementarContador`.
    * Selecione o botão `+`. Na seção **Events**, adicione um evento com `Trigger: CLICKED` e `Function: incrementarContador`.
4.  Exporte os arquivos da UI. Lembre-se de colocar todos os arquivos gerados na mesma pasta do seu sketch `.ino`.

### Passo 2: A Lógica do Contador em `ui_events.cpp`

Toda a "inteligência" da nossa aplicação residirá no arquivo `ui_events.cpp`. É aqui que vamos declarar nosso contador, criar as funções que os botões chamam e, o mais importante, atualizar o texto do label na tela.

Não se esqueça de **renomear `ui_events.c` para `ui_events.cpp`** para poder usar funções do Arduino.

```cpp
/*
 * EXEMPLO 4: CONTADOR INTERATIVO
 * O código principal não muda. Toda a lógica está nos arquivos da UI.
 */

#define LGFX_USE_V1
#include <LovyanGFX.hpp>
#include <lvgl.h>
#include "ui.h"

// --- Configuração de Hardware para a LovyanGFX ---
class LGFX : public lgfx::LGFX_Device {
  lgfx::Panel_ST7796  _panel_instance;
  lgfx::Bus_Parallel8 _bus_instance;
  lgfx::Light_PWM     _light_instance;
  lgfx::Touch_FT5x06  _touch_instance;
public:
  LGFX(void) {
    { // BUS
      auto cfg = _bus_instance.config();
      cfg.port = 0; cfg.freq_write = 20000000;
      cfg.pin_wr = 47; cfg.pin_rd = -1; cfg.pin_rs = 0;
      cfg.pin_d0 = 9; cfg.pin_d1 = 46; cfg.pin_d2 = 3; cfg.pin_d3 = 8;
      cfg.pin_d4 = 18; cfg.pin_d5 = 17; cfg.pin_d6 = 16; cfg.pin_d7 = 15;
      _bus_instance.config(cfg);
      _panel_instance.setBus(&_bus_instance);
    }
    { // PANEL
      auto cfg = _panel_instance.config();
      cfg.pin_cs = -1; cfg.pin_rst = 4; cfg.pin_busy = -1;
      cfg.memory_width = 320; cfg.memory_height = 480;
      cfg.panel_width = 320; cfg.panel_height = 480;
      cfg.offset_x = 0; cfg.offset_y = 0;
      cfg.invert = true; cfg.rgb_order = false;
      _panel_instance.config(cfg);
    }
    { // BACKLIGHT
      auto cfg = _light_instance.config();
      cfg.pin_bl = 45; cfg.invert = false; cfg.freq = 44100; cfg.pwm_channel = 7;
      _light_instance.config(cfg);
      _panel_instance.setLight(&_light_instance);
    }
    { // TOUCH
      auto cfg = _touch_instance.config();
      cfg.x_min = 0; cfg.x_max = 319;
      cfg.y_min = 0; cfg.y_max = 479;
      cfg.pin_int = 7;
      cfg.bus_shared = true;
      cfg.offset_rotation = 0;
      cfg.i2c_port = 0; cfg.i2c_addr = 0x38;
      cfg.pin_sda = 6; cfg.pin_scl = 5;
      cfg.freq = 400000;
      _touch_instance.config(cfg);
      _panel_instance.setTouch(&_touch_instance);
    }
    setPanel(&_panel_instance);
  }
};

// --- Configuração de Drivers e Buffers ---
LGFX gfx; 
static const uint16_t screenWidth  = 480;
static const uint16_t screenHeight = 320;
static lv_disp_draw_buf_t draw_buf;
static lv_color_t buf1[screenWidth * screenHeight / 10];

// --- Funções de Ponte para o LVGL ---
void my_disp_flush(lv_disp_drv_t *disp_drv, const lv_area_t *area, lv_color_t *color_p) {
    uint32_t w = (area->x2 - area->x1 + 1);
    uint32_t h = (area->y2 - area->y1 + 1);
    gfx.startWrite();
    gfx.setAddrWindow(area->x1, area->y1, w, h);
    gfx.pushPixels((uint16_t *)color_p, w * h, true);
    gfx.endWrite();
    lv_disp_flush_ready(disp_drv);
}

void my_touch_read(lv_indev_drv_t *indev_drv, lv_indev_data_t *data) {
    uint16_t touchX, touchY;
    bool touched = gfx.getTouch(&touchX, &touchY);
    if (touched) {
        data->point.x = touchX;
        data->point.y = touchY;
        data->state = LV_INDEV_STATE_PR;
    } else {
        data->state = LV_INDEV_STATE_REL;
    }
}

// --- Aplicação Principal ---
void setup() {
    Serial.begin(115200);
    Serial.println("Iniciando Exemplo 4: Contador Interativo");
    
    gfx.begin();
    gfx.setRotation(1);
    gfx.setBrightness(255);
    
    lv_init();
    lv_disp_draw_buf_init(&draw_buf, buf1, NULL, screenWidth * screenHeight / 10);
    
    static lv_disp_drv_t disp_drv;
    lv_disp_drv_init(&disp_drv);
    disp_drv.hor_res = screenWidth;
    disp_drv.ver_res = screenHeight;
    disp_drv.flush_cb = my_disp_flush;
    disp_drv.draw_buf = &draw_buf;
    lv_disp_drv_register(&disp_drv);

    static lv_indev_drv_t indev_drv;
    lv_indev_drv_init(&indev_drv);
    indev_drv.type = LV_INDEV_TYPE_POINTER;
    indev_drv.read_cb = my_touch_read;
    lv_indev_drv_register(&indev_drv);

    ui_init(); 
    Serial.println("Setup finalizado.");
}

void loop() {
    lv_timer_handler();
    lv_tick_inc(5);
    delay(5);
}
```
**Entendendo o Código:**

* **`static int32_t contador = 0;`**: Criamos uma variável estática para guardar o valor do contador. Ela precisa ser estática (ou global) para que seu valor seja preservado entre as chamadas das funções.

* **`lv_label_set_text(ui_labelContador, buffer);`**: Esta é a linha mais importante. O SquareLine Studio declara automaticamente ponteiros para todos os seus widgets no arquivo `ui.h`. `ui_labelContador` é o nosso label. A função `lv_label_set_text` é uma função nativa do LVGL que nos permite alterar o texto de um widget de label a qualquer momento.

### Passo 3: O Código Principal (Sem Mudanças!)

A melhor parte é que nosso arquivo `.ino` **não precisa de nenhuma alteração**. Ele já está preparado para inicializar o hardware, o LVGL e o toque. Toda a nova lógica está contida nos arquivos da UI. Por completude, aqui está o código base que usamos:

```cpp
/*
 * EXEMPLO 4: CONTADOR INTERATIVO
 * O código principal não muda. Toda a lógica está nos arquivos da UI.
 */

#define LGFX_USE_V1
#include <LovyanGFX.hpp>
#include <lvgl.h>
#include "ui.h"

// A classe LGFX completa (com display e toque) vai aqui...
class LGFX : public lgfx::LGFX_Device {
  // (Conteúdo da classe LGFX omitido por brevidade - é o mesmo do exemplo 3)
};

LGFX gfx; 
// Buffers e funções my_disp_flush e my_touch_read vão aqui...
// (Omitidos por brevidade - são os mesmos do exemplo 3)

void setup() {
    Serial.begin(115200);
    Serial.println("Iniciando Exemplo 4: Contador Interativo");
    gfx.begin();
    gfx.setRotation(1);
    gfx.setBrightness(255);
    lv_init();
    // ... resto da inicialização do display e toque ...
    ui_init(); 
    Serial.println("Setup finalizado.");
}

void loop() {
    lv_timer_handler();
    lv_tick_inc(5);
    delay(5);
}
```
### Conclusão

Parabéns! Você agora tem uma aplicação totalmente dinâmica. Ao tocar nos botões, o valor na tela é atualizado em tempo real, e você pode ver a confirmação no Monitor Serial.

Você aprendeu o conceito mais poderoso da programação de interfaces com LVGL: como **acessar e manipular widgets dinamicamente** a partir do seu código de eventos. Com essa técnica, você pode criar desde termômetros digitais que mostram dados de sensores até painéis de controle complexos.

No nosso próximo e último post prático, vamos explorar como gerenciar múltiplas telas e navegar entre elas.

Até lá!