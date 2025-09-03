---
title: "Guia Completo da WT32-SC01 Plus (parte 4 de 6): Ativando o Toque e Eventos"
date: 2025-09-03T09:56:12-03:00
draft: false
tags: ["ESP32-S3", "WT32-SC01 Plus", "LovyanGFX", "LVGL", "SquareLine Studio", "Touchscreen", "Eventos"]
categories: ["Hardware", "ESP32"]
---

Nos posts anteriores, preparamos nosso ambiente, fizemos a tela da **WT32-SC01 Plus** funcionar e até exibimos nossa primeira interface gráfica criada com o **SquareLine Studio**. No entanto, nossas telas ainda eram estáticas. Chegou a hora de dar vida ao nosso projeto, habilitando o recurso mais importante da placa: o toque.

Neste tutorial, vamos transformar nossa interface estática em uma aplicação interativa. Nosso objetivo é adicionar um botão na tela e, quando ele for pressionado, executar uma ação: imprimir uma mensagem no Monitor Serial. Este é o passo fundamental para criar qualquer aplicação complexa, desde calculadoras a painéis de controle.

### Passo 1: Habilitando o Toque no Código Principal

Até agora, nosso código `.ino` só se preocupava em desenhar na tela. Para que ele possa ler o toque, precisamos fazer duas adições importantes: configurar o hardware de toque na nossa classe `LGFX` e criar a "ponte" para que o LVGL entenda os dados do toque.

1.  **Configuração do Hardware de Toque:** Abra seu sketch e adicione a configuração do driver de toque à sua classe `LGFX`. A `LovyanGFX` já possui um driver nativo para o chip `FT6336U` da nossa placa.

    ```cpp
    // Adicione esta linha junto com as outras instâncias de drivers
    lgfx::Touch_FT5x06  _touch_instance;

    // Adicione este bloco de configuração dentro da função LGFX(void)
    { 
      auto cfg = _touch_instance.config();
      cfg.x_min = 0; cfg.x_max = 319;
      cfg.y_min = 0; cfg.y_max = 479;
      cfg.pin_int = 7;
      cfg.bus_shared = true;
      cfg.offset_rotation = 0;
      cfg.i2c_port = 0;
      cfg.i2c_addr = 0x38;
      cfg.pin_sda = 6;
      cfg.pin_scl = 5;
      cfg.freq = 400000;
      _touch_instance.config(cfg);
      _panel_instance.setTouch(&_touch_instance);
    }
    ```

2.  **Crie a Função de Leitura do Toque:** Assim como a `my_disp_flush` serve para desenhar, a `my_touch_read` serve para ler o toque. Ela pega as coordenadas da `LovyanGFX` e as "traduz" para o formato que o LVGL entende. Adicione esta função ao seu código:

    ```cpp
    void my_touch_read(lv_indev_drv_t *indev_drv, lv_indev_data_t *data) {
        uint16_t touchX, touchY;
        bool touched = gfx.getTouch(&touchX, &touchY);
        if (touched) {
            data->point.x = touchX;
            data->point.y = touchY;
            data->state = LV_INDEV_STATE_PR; // Pressionado
        } else {
            data->state = LV_INDEV_STATE_REL; // Liberado
        }
    }
    ```

3.  **Registre o Driver de Toque no `setup()`:** Por fim, precisamos dizer ao LVGL para usar a nossa função `my_touch_read` como fonte de entrada. Adicione este bloco ao seu `setup()`:

    ```cpp
    static lv_indev_drv_t indev_drv;
    lv_indev_drv_init(&indev_drv);
    indev_drv.type = LV_INDEV_TYPE_POINTER;
    indev_drv.read_cb = my_touch_read;
    lv_indev_drv_register(&indev_drv);
    ```

### Passo 2: Criando a Interface Interativa no SquareLine Studio

1.  Crie um novo projeto no SquareLine Studio (ou use o anterior).
2.  Arraste um widget **Button** para a tela.
3.  Dentro do botão, adicione um **Label** e altere o texto para algo como "Clique Aqui".
4.  Selecione o objeto **Button**. No painel "Inspector" à direita, role para baixo até a seção **Events**.
5.  Clique em **"Add Event"**.
6.  Configure o evento da seguinte forma:
    * **Trigger:** `CLICKED` (ou `PRESSED`).
    * **Action:** `Call function`.
    * **Function:** Digite o nome da função que queremos chamar quando o botão for clicado, por exemplo, `botaoPressionado`.

7.  Exporte os arquivos da UI como fizemos no post anterior.

### Passo 3: A Lógica dos Eventos em `ui_events.cpp`

Quando você exporta, o SquareLine Studio cria dois arquivos para lidar com os eventos: `ui_events.h` e `ui_events.c`. É aqui que vamos escrever o que acontece quando o botão é pressionado.

Por padrão, o arquivo `ui_events.c` é um arquivo C puro. No entanto, para usarmos funções do Arduino, como a `Serial.println()`, precisamos que ele seja compilado como C++. A solução é simples:

**Renomeie o arquivo `ui_events.c` para `ui_events.cpp`.**

Agora, abra o seu novo `ui_events.cpp` e adicione a lógica da nossa função `botaoPressionado`:

```cpp
/*
 * ui_events.cpp
 * Lógica para os eventos da interface do SquareLine Studio.
 */
#include <Arduino.h> // Essencial para usar funções do Arduino
#include "ui.h"

void botaoPressionado(lv_event_t * e)
{
    // Esta função será chamada sempre que o botão for clicado
    Serial.println("O botão foi pressionado!");
    
    // Qualquer outra lógica pode ser adicionada aqui.
}
```

### Passo 4: O Código Final e o Resultado

Juntando tudo, nosso arquivo `.ino` final fica assim. Note que ele é muito parecido com o do exemplo anterior, mas agora com a configuração de toque completa.

```cpp
/*
 * EXEMPLO 3: BOTÃO INTERATIVO COM SQUARELINE E LOVYANGFX
 * * Este sketch habilita o toque e responde a um evento de clique
 * em um botão criado no SquareLine Studio.
 */

#define LGFX_USE_V1
#include <LovyanGFX.hpp>
#include <lvgl.h>
#include "ui.h"

// Classe LGFX completa com a configuração do TOQUE incluída
class LGFX : public lgfx::LGFX_Device {
  lgfx::Panel_ST7796  _panel_instance;
  lgfx::Bus_Parallel8 _bus_instance;
  lgfx::Light_PWM     _light_instance;
  lgfx::Touch_FT5x06  _touch_instance; // Driver de toque
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

LGFX gfx; 
static const uint16_t screenWidth  = 480;
static const uint16_t screenHeight = 320;
static lv_disp_draw_buf_t draw_buf;
static lv_color_t buf1[screenWidth * screenHeight / 10];

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

void setup() {
    Serial.begin(115200);
    Serial.println("Iniciando Exemplo 3: Botão Interativo");
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
    Serial.println("Setup finalizado. A interface e o toque devem funcionar.");
}

void loop() {
    lv_timer_handler();
    lv_tick_inc(5);
    delay(5);
}
```

Compile e carregue o código. Agora, ao tocar no botão na tela, você verá a mensagem "O botão foi pressionado!" aparecer no seu Monitor Serial!

### Conclusão

Parabéns! Você acaba de criar sua primeira aplicação totalmente interativa na WT32-SC01 Plus. Aprendemos a habilitar o toque, a configurar eventos no SquareLine Studio e a responder a esses eventos no nosso código.

Com esta base, as possibilidades são infinitas. No próximo post, vamos usar o que aprendemos para criar algo ainda mais prático: um contador que incrementa e decrementa um valor na tela.

Até lá!