---
title: "Guia Completo da WT32-SC01 Plus (parte 3 de 6): Sua Primeira Interface com SquareLine Studio"
date: 2025-08-26T10:13:37-03:00
draft: false
tags: ["ESP32-S3", "WT32-SC01 Plus", "LovyanGFX", "LVGL", "SquareLine Studio"]
categories: ["Hardware", "ESP32"]
---

No post anterior, demos nosso primeiro grande passo ao escrever "Hello World!" diretamente na tela da **WT32-SC01 Plus** usando a biblioteca `LovyanGFX`. Isso provou que nosso hardware e a comunicação básica estão funcionando. Agora, vamos elevar o nível e começar a construir interfaces gráficas de verdade.

Neste tutorial, vamos introduzir duas ferramentas poderosas que transformarão a maneira como criamos projetos visuais: a biblioteca **LVGL** e a ferramenta de design **SquareLine Studio**. Nosso objetivo será recriar o "Hello World!", mas desta vez, a interface será desenhada em um software visual e depois integrada ao nosso código Arduino.

### As Ferramentas do Ofício: LVGL e SquareLine Studio

* **LVGL (Light and Versatile Graphics Library):** É uma biblioteca de código aberto extremamente popular para a criação de interfaces gráficas em sistemas embarcados. Ela oferece um conjunto rico de "widgets" (botões, sliders, gráficos, etc.) e gerencia todo o trabalho pesado de renderização e eventos.
* **SquareLine Studio:** É um software de design visual que permite arrastar e soltar elementos para criar interfaces complexas. Embora seja uma ferramenta comercial, ela oferece uma licença gratuita para uso pessoal e de hobby com algumas limitações (como o número de telas e widgets), que é perfeita para os nossos projetos. Ao final, ele exporta um projeto C completo que podemos integrar diretamente ao nosso código.

### Passo 1: Preparando o Ambiente para LVGL

Além da `LovyanGFX` e do pacote de placas ESP32 que já instalamos, agora precisamos adicionar a biblioteca LVGL.

1.  No Arduino IDE, vá em **Ferramentas > Gerenciar Bibliotecas...**.
2.  Pesquise por `lvgl` e instale a versão **8.3.11**. É importante usar esta versão para garantir a compatibilidade com os arquivos que o SquareLine Studio exporta.
3.  **Crie o arquivo `lv_conf.h`:** Este é um passo crucial.
    a. Encontre o arquivo `lv_conf_template.h` na pasta da biblioteca LVGL.
    b. Copie-o para a sua pasta raiz `libraries` (ex: `Documentos/Arduino/libraries`).
    c. Renomeie a cópia para `lv_conf.h`.
    d. Abra o arquivo e garanta que a linha `#define LV_COLOR_DEPTH` esteja definida como `16`.

### Passo 2: Criando a Interface no SquareLine Studio

1.  Abra o SquareLine Studio e clique em **"Create"** para um novo projeto.
2.  Na tela de seleção de templates, clique na aba **"Arduino"**.
3.  Selecione o template **"Arduino with TFT_eSPI"**. É muito importante escolher este, pois ele gera a estrutura de arquivos correta para a IDE do Arduino, mesmo que estejamos usando a `LovyanGFX` no nosso código final.
4.  Na aba "Project Settings", configure a resolução da tela para **480x320** e clique em **"Create"**.
5.  Arraste um widget "Label" para a tela.
6.  No painel "Inspector" à direita, altere o texto do label para "Hello World!" e ajuste a fonte e o alinhamento como desejar.
7.  Antes de exportar, vá ao menu **File > Project Settings**. Na janela que abrir, na seção **"FILE EXPORT"**, configure os campos "Project Export Root" e "UI Files Export Path" para uma pasta de sua escolha (por exemplo, você pode criar uma pasta "exports" dentro do seu projeto do SquareLine). Isso evita erros durante a exportação.
8.  Vá em **Export > Export UI Files** para gerar os arquivos do projeto.

### Passo 3: O Código de Integração

**Importante:**
* Coloque todos os arquivos exportados pelo SquareLine Studio (que estarão na pasta que você configurou no item 7 do Passo 2) na mesma pasta do seu sketch `.ino`.
* No arquivo `ui.c` que o SquareLine gerou, encontre o bloco `TEST LVGL SETTINGS` e comente a verificação de `LV_COLOR_DEPTH`, deixando-o assim:
  ```cpp
  ///////////////////// TEST LVGL SETTINGS ////////////////////
  // #if LV_COLOR_DEPTH != 32
  //     #error "LV_COLOR_DEPTH should be 32bit to match SquareLine Studio's settings"
  // #endif
  #if LV_COLOR_16_SWAP !=0
      #error "LV_COLOR_16_SWAP should be 0 to match SquareLine Studio's settings"
  #endif
  ```
Agora, vamos ao código que une tudo. Ele usa a mesma base da `LovyanGFX` do exemplo anterior, mas adiciona toda a inicialização e o loop necessários para que o LVGL funcione.

  ```cpp
/*
 * EXEMPLO 2: HELLO WORLD COM SQUARELINE STUDIO E LOVYANGFX
 * * Este sketch demonstra como integrar uma interface gráfica simples,
 * criada no SquareLine Studio, com a biblioteca LovyanGFX.
 */

#define LGFX_USE_V1 // Define que estamos usando a Versão 1 da LovyanGFX

#include <LovyanGFX.hpp>
#include <lvgl.h>
#include "ui.h" // O header principal da sua interface gráfica gerada pelo SquareLine

// ======================================================================================
// === 1. CONFIGURAÇÃO DE HARDWARE (LOVYANGFX) ==========================================
// ======================================================================================
// Esta classe descreve o hardware da tela para a biblioteca LovyanGFX.
class LGFX : public lgfx::LGFX_Device
{
  lgfx::Panel_ST7796  _panel_instance;
  lgfx::Bus_Parallel8 _bus_instance;
  lgfx::Light_PWM     _light_instance;

public:
  LGFX(void)
  {
    { 
      auto cfg = _bus_instance.config();
      cfg.port = 0; cfg.freq_write = 20000000;
      cfg.pin_wr = 47; cfg.pin_rd = -1; cfg.pin_rs = 0;
      cfg.pin_d0 = 9; cfg.pin_d1 = 46; cfg.pin_d2 = 3; cfg.pin_d3 = 8;
      cfg.pin_d4 = 18; cfg.pin_d5 = 17; cfg.pin_d6 = 16; cfg.pin_d7 = 15;
      _bus_instance.config(cfg);
      _panel_instance.setBus(&_bus_instance);
    }
    { 
      auto cfg = _panel_instance.config();
      cfg.pin_cs = -1; cfg.pin_rst = 4; cfg.pin_busy = -1;
      cfg.memory_width = 320; cfg.memory_height = 480;
      cfg.panel_width = 320; cfg.panel_height = 480;
      cfg.offset_x = 0; cfg.offset_y = 0;
      cfg.invert = true; cfg.rgb_order = false;
      _panel_instance.config(cfg);
    }
    { 
      auto cfg = _light_instance.config();
      cfg.pin_bl = 45; cfg.invert = false; cfg.freq = 44100; cfg.pwm_channel = 7;
      _light_instance.config(cfg);
      _panel_instance.setLight(&_light_instance);
    }
    setPanel(&_panel_instance);
  }
};

// --- Instâncias e Buffers ---
LGFX gfx; 
static const uint16_t screenWidth  = 480;
static const uint16_t screenHeight = 320;
static lv_disp_draw_buf_t draw_buf;
static lv_color_t buf1[screenWidth * screenHeight / 10];

// --- Funções de Interface (Ponte LVGL <-> LovyanGFX) ---
void my_disp_flush(lv_disp_drv_t *disp_drv, const lv_area_t *area, lv_color_t *color_p) {
    uint32_t w = (area->x2 - area->x1 + 1);
    uint32_t h = (area->y2 - area->y1 + 1);
    gfx.startWrite();
    gfx.setAddrWindow(area->x1, area->y1, w, h);
    gfx.pushPixels((uint16_t *)color_p, w * h, true);
    gfx.endWrite();
    lv_disp_flush_ready(disp_drv);
}

// --- Setup e Loop ---
void setup() {
    Serial.begin(115200);
    Serial.println("Iniciando Exemplo 2: SquareLine Studio com LovyanGFX");
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
    ui_init(); // Carrega a interface do SquareLine
    Serial.println("Setup finalizado. A interface do SquareLine deve aparecer.");
}

void loop() {
    lv_timer_handler();
    lv_tick_inc(5);
    delay(5);
}
```

### Entendendo as Novidades no Código

* **Includes:** Agora temos `#include <lvgl.h>` para a biblioteca gráfica e `#include "ui.h"` para carregar todos os arquivos da interface que o SquareLine Studio gerou.
* **Buffers do LVGL:** As variáveis `draw_buf` e `buf1` são áreas de memória que o LVGL usa para renderizar a interface antes de enviá-la para a tela.
* **`my_disp_flush`:** Esta é a função "ponte" essencial. O LVGL a chama quando tem algo novo para desenhar, e nosso código usa a `LovyanGFX` para efetivamente colocar os pixels na tela.
* **`setup()`:** A inicialização agora inclui `lv_init()` e o registro do nosso driver de display com o LVGL. A chamada mais importante é `ui_init()`, que executa o código gerado pelo SquareLine para criar os widgets.
* **`loop()`:** O loop agora é o coração do LVGL. `lv_timer_handler()` processa todas as tarefas da interface. A função `lv_tick_inc(5)` informa ao LVGL que o tempo está passando. Embora neste exemplo estático ela não tenha um efeito visível, é crucial para futuras animações e eventos, por isso já a incluímos como boa prática. O valor `5` deve sempre corresponder ao `delay(5)` para manter o relógio interno do LVGL sincronizado com o tempo real.

### Conclusão

Se tudo deu certo, você agora vê na sua tela a mesma interface que desenhou no SquareLine Studio! Você acaba de dar um salto gigantesco: de desenhar texto com código para usar uma ferramenta profissional de design de UI.

No próximo post, vamos ativar o toque e fazer nossa primeira interface interativa.

Até lá!
