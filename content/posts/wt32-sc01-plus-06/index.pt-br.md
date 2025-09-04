---
title: "Guia Completo da WT32-SC01 Plus (parte 6 de 6): Navegação Entre Telas"
date: 2025-09-04T10:03:10-03:00
draft: false
tags: ["ESP32-S3", "WT32-SC01 Plus", "LovyanGFX", "LVGL", "SquareLine Studio", "Screens", "UI"]
categories: ["Hardware", "ESP32"]
---

Chegamos ao final da nossa jornada prática com a **WT32-SC01 Plus**! Ao longo dos últimos posts, aprendemos a configurar o ambiente, desenhar na tela, habilitar o toque e manipular widgets. Agora, vamos juntar todo esse conhecimento para construir a estrutura de uma aplicação real, que quase sempre envolve mais de uma tela.

Neste último tutorial prático, vamos aprender a criar e gerenciar múltiplas telas no **SquareLine Studio** e a navegar entre elas usando eventos de botão. Criaremos uma aplicação simples com duas telas, onde um botão em cada tela nos levará para a outra, demonstrando o fluxo de navegação básico essencial para qualquer projeto complexo, como menus de configuração, páginas de informação, etc.

### Passo 1: Criando as Telas no SquareLine Studio

Primeiro, vamos projetar nossa interface com duas telas distintas.

1. Abra o SquareLine Studio.
2. No painel "Screens" no canto superior esquerdo, você verá a `Screen1`. Clique no pequeno ícone de `+` para adicionar uma nova tela. Ela será nomeada automaticamente como `Screen2`.
3. **Na `Screen1`:**
   * Adicione um **Label** com o texto "Tela Principal".
   * Adicione um **Button** com o texto "Ir para Tela 2".
   * Selecione o botão e, na seção **Events**, crie um evento com `Trigger: CLICKED` e `Function: irParaTela2`.
4. **Na `Screen2`:**
   * Selecione a `Screen2` no painel "Screens" para editá-la.
   * Adicione um **Label** com o texto "Tela Secundária".
   * Adicione um **Button** com o texto "Voltar para Tela 1".
   * Selecione este botão e crie um evento com `Trigger: CLICKED` e `Function: voltarParaTela1`.
5. Exporte os arquivos da UI e coloque-os na pasta do seu projeto Arduino.

### Passo 2: A Lógica de Navegação em `ui_events.cpp`

A mágica da troca de telas acontece no arquivo `ui_events.cpp`, onde implementaremos as duas funções que acabamos de definir no SquareLine.

Lembre-se de **renomear `ui_events.c` para `ui_events.cpp`**.

```cpp
/*
 * ui_events.cpp
 * Lógica para a navegação entre telas.
 */
#include <Arduino.h>
#include "ui.h"

// Função chamada pelo botão na Tela 1
void irParaTela2(lv_event_t * e)
{
    Serial.println("Navegando para a Tela 2...");
    // Função do LVGL para carregar uma nova tela com uma animação.
    // Parâmetros: (tela_destino, animação, duração, delay, deletar_anterior)
    lv_scr_load_anim(ui_Screen2, LV_SCR_LOAD_ANIM_MOVE_LEFT, 500, 0, false);
}

// Função chamada pelo botão na Tela 2
void voltarParaTela1(lv_event_t * e)
{
    Serial.println("Voltando para a Tela 1...");
    lv_scr_load_anim(ui_Screen1, LV_SCR_LOAD_ANIM_MOVE_RIGHT, 500, 0, false);
}
```
**Entendendo o Código:**

* **`lv_scr_load_anim(...)`**: Esta é a função principal do LVGL para navegação.
    * O primeiro parâmetro (`ui_Screen2` ou `ui_Screen1`) é o ponteiro para o objeto da tela que queremos carregar. Assim como os widgets, o SquareLine Studio cria uma variável global para cada tela.
    * `LV_SCR_LOAD_ANIM_MOVE_LEFT`: É o tipo de animação. O LVGL oferece várias, como `FADE_ON`, `OVER_TOP`, etc. `MOVE_RIGHT` faz a animação inversa.
    * `500`: A duração da animação em milissegundos.
    * `0`: Um atraso (delay) antes de a animação começar.
    * `false`: Informa ao LVGL para **não** deletar a tela anterior da memória. Isso é útil para navegações rápidas de ida e volta. Em projetos com muitas telas e pouca RAM, você poderia setar para `true` para liberar memória.

### Passo 3: O Código Principal (Inalterado)

Mais uma vez, nosso arquivo `.ino` não precisa de nenhuma alteração. Ele serve como a base sólida que executa nossa aplicação, independentemente de quantas telas ou qual lógica de eventos tenhamos.

Aqui está o código completo para garantir que seu projeto funcione perfeitamente.

```cpp
/*
 * EXEMPLO 5: NAVEGAÇÃO ENTRE TELAS
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
    Serial.println("Iniciando Exemplo 5: Navegação entre Telas");
    
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
### Conclusão da Série

Parabéns! Você chegou ao final desta série de tutoriais para a WT32-SC01 Plus. Ao longo destes posts, você aprendeu a:

* Configurar o ambiente de desenvolvimento do zero.
* Controlar o display diretamente com a `LovyanGFX`.
* Integrar a poderosa biblioteca `LVGL` com a ferramenta visual `SquareLine Studio`.
* Habilitar o toque e responder a eventos de botões.
* Manipular widgets para criar interfaces dinâmicas.
* Gerenciar e navegar entre múltiplas telas.

Com esta base sólida, você está mais do que preparado para criar seus próprios projetos complexos e com interfaces profissionais. A WT32-SC01 Plus é uma plataforma incrivelmente capaz, e agora você tem as ferramentas para explorar todo o seu potencial.

Espero que esta série tenha sido útil. Boas criações!
