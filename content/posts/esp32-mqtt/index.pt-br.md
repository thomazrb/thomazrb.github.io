---
title: "ESP32: Enviando Dados via MQTT para a Nuvem"
date: 2026-01-19T11:11:11-03:00
draft: false
tags: ["ESP32", "MQTT", "IoT", "Arduino IDE"]
categories: ["IoT", "ESP32"]
---

A Internet das Coisas (IoT) revolucionou a forma como dispositivos se comunicam, e o protocolo MQTT (Message Queuing Telemetry Transport) é uma das formas mais populares e eficientes de fazer essa comunicação acontecer. Neste tutorial, vamos conectar um ESP32 à internet via WiFi e fazer ele enviar dados periodicamente para um broker MQTT público, permitindo que você visualize essas informações em tempo real tanto no navegador quanto em um aplicativo Android.

O ESP32 é um microcontrolador poderoso com WiFi e Bluetooth integrados, perfeito para projetos IoT. Vamos criar um exemplo simples que envia um contador crescente a cada 30 segundos, demonstrando os conceitos fundamentais de comunicação MQTT que você poderá aplicar em projetos mais complexos.

**Versões utilizadas neste tutorial:**
- Arduino IDE: 2.3.7
- ESP32 Board Package: 3.3.5
- PubSubClient (biblioteca MQTT): 2.8.0

### O que é MQTT?

MQTT é um protocolo de mensagens leve, projetado especificamente para dispositivos com recursos limitados e redes com pouca largura de banda. Ele funciona no modelo publish/subscribe (publicar/assinar):

* **Broker:** O servidor central que recebe todas as mensagens e as distribui para os interessados (no nosso caso, o broker público da HiveMQ).
* **Publisher:** O dispositivo que publica mensagens em um tópico (nosso ESP32).
* **Subscriber:** O dispositivo ou aplicativo que se inscreve em tópicos para receber mensagens (nosso navegador ou app Android).
* **Tópico:** É como um "canal" onde as mensagens são publicadas. Por exemplo: `casa/sala/temperatura`.

### Passo 1: Preparando a Arduino IDE

Antes de começar a programar, precisamos configurar a Arduino IDE para trabalhar com o ESP32.

#### 1.1: Instalar o Suporte para ESP32

Se você ainda não tem o suporte para ESP32 instalado na Arduino IDE:

1. Abra a Arduino IDE
2. Vá em **File → Preferences** (ou **Arduino IDE → Settings** no macOS)
3. No campo **"Additional Boards Manager URLs"**, adicione:
   ```
   https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
   ```
4. Clique em **OK**
5. Vá em **Tools → Board → Boards Manager**
6. Procure por "esp32" e instale o pacote **"esp32 by Espressif Systems"**
7. Aguarde a instalação completar

#### 1.2: Instalar a Biblioteca MQTT

Precisamos da biblioteca PubSubClient para comunicação MQTT:

1. Vá em **Sketch → Include Library → Manage Libraries** (ou **Tools → Manage Libraries**)
2. Procure por "PubSubClient"
3. Instale a biblioteca **"PubSubClient" by Nick O'Leary**

#### 1.3: Selecionar a Placa ESP32

1. Conecte seu ESP32 ao computador via USB
2. Vá em **Tools → Board** e selecione seu modelo de ESP32 (geralmente **"ESP32 Dev Module"** funciona para a maioria dos modelos)
3. Em **Tools → Port**, selecione a porta COM onde seu ESP32 está conectado

### Passo 2: Entendendo o Código

Nosso programa fará o seguinte:

1. Conectar o ESP32 à rede WiFi
2. Conectar ao broker MQTT público da HiveMQ
3. A cada 30 segundos, enviar um contador crescente para um tópico MQTT

### Passo 3: Código Completo

Crie um novo sketch na Arduino IDE e cole o código abaixo. **Importante:** substitua `SEU_WIFI_AQUI` e `SUA_SENHA_AQUI` pelas credenciais da sua rede WiFi.

```cpp
#include <WiFi.h>
#include <PubSubClient.h>

// Configurações do WiFi - SUBSTITUA com suas credenciais
const char* ssid = "SEU_WIFI_AQUI";
const char* password = "SUA_SENHA_AQUI";

// Configurações do broker MQTT
const char* mqtt_server = "mqtt-dashboard.com";
const int mqtt_port = 1883;

// Tópico MQTT onde os dados serão publicados
// Use um identificador único para evitar conflito com outros usuários
const char* mqtt_topic = "esp32/thomazrb/contador";

// Clientes WiFi e MQTT
WiFiClient espClient;
PubSubClient client(espClient);

// Variáveis de controle
unsigned long lastMsg = 0;
int contador = 0;

// Função para conectar ao WiFi
void setup_wifi() {
  delay(10);
  Serial.println();
  Serial.print("Conectando ao WiFi: ");
  Serial.println(ssid);

  WiFi.begin(ssid, password);

  // Aguarda até conectar
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("");
  Serial.println("WiFi conectado!");
  Serial.print("Endereço IP: ");
  Serial.println(WiFi.localIP());
}

// Função de callback - chamada quando uma mensagem é recebida
// Neste exemplo simples, não vamos assinar nenhum tópico, mas a função é obrigatória
void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Mensagem recebida no tópico: ");
  Serial.println(topic);
  
  Serial.print("Conteúdo: ");
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
  }
  Serial.println();
}

void setup() {
  // Inicializa comunicação serial para debug
  Serial.begin(115200);
  
  // Conecta ao WiFi
  setup_wifi();
  
  // Configura o servidor MQTT e a função de callback
  client.setServer(mqtt_server, mqtt_port);
  client.setCallback(callback);
  
  // Conecta ao broker MQTT
  Serial.print("Conectando ao broker MQTT...");
  
  // Gera um ID de cliente único
  String clientId = "ESP32Client-";
  clientId += String(random(0xffff), HEX);
  
  if (client.connect(clientId.c_str())) {
    Serial.println("conectado!");
    // Publica uma mensagem de confirmação
    client.publish(mqtt_topic, "ESP32 Online");
  } else {
    Serial.print("falhou, rc=");
    Serial.println(client.state());
  }
}

void loop() {
  // Mantém a conexão MQTT ativa
  client.loop();

  // Envia mensagem a cada 30 segundos
  unsigned long now = millis();
  if (now - lastMsg > 30000) { // 30000 ms = 30 segundos
    lastMsg = now;
    
    // Incrementa o contador
    contador++;
    
    // Converte o contador para string
    char msg[50];
    snprintf(msg, 50, "Contador: %d", contador);
    
    // Publica no tópico MQTT
    Serial.print("Publicando mensagem: ");
    Serial.println(msg);
    client.publish(mqtt_topic, msg);
  }
}
```

### Passo 4: Upload e Teste

1. Clique no botão **Upload** (seta para a direita) na Arduino IDE
2. Aguarde o upload completar
3. Abra o **Serial Monitor** (ícone de lupa no canto superior direito ou **Tools → Serial Monitor**)
4. Configure a velocidade para **115200 baud**
5. Você deverá ver mensagens indicando a conexão ao WiFi e ao broker MQTT

### Passo 5: Visualizando os Dados com MQTT Websocket Client

Você pode usar o cliente web da HiveMQ para testar conexões MQTT:

1. Acesse: [https://www.hivemq.com/demos/websocket-client/](https://www.hivemq.com/demos/websocket-client/)
2. Clique em **"Connect"** (as configurações padrão já estão corretas)
3. Após conectar, vá na seção **"Subscriptions"**
4. Em **"Topic"**, digite exatamente o mesmo tópico que você usou no código: `esp32/thomazrb/contador`
5. **QoS:** 0
6. Clique em **"Subscribe"**
7. Você começará a ver as mensagens chegando a cada 30 segundos!

### Passo 6: Visualizando no Android com IoT MQTT Panel

Para acompanhar os dados no celular:

1. Instale o app **"IoT MQTT Panel"** da Play Store
2. Abra o app e toque no botão vermelho **"SETUP A CONNECTION"**
3. Preencha os campos:
   - **Connection name:** MQTT Dashboard (ou qualquer nome de sua preferência)
   - **Client ID:** deixe vazio (será gerado automaticamente)
   - **Broker address:** `mqtt-dashboard.com`
   - **Port:** `8883`
   - **Network protocol:** TCP-SSL (importante mudar de TCP para TCP-SSL!)
4. Toque em **"Add Dashboard"** e dê um nome para o dashboard (ex: "Meu Dashboard")
5. Toque em **"CREATE"**
6. O app irá conectar ao broker e abrir o dashboard vazio
7. Toque no botão azul **"ADD A PANEL"** no centro da tela (após adicionar o primeiro painel, os próximos são adicionados pelo botão **"+"** no canto inferior direito)
8. Selecione **"Text Output"**
9. Configure apenas os campos obrigatórios:
   - **Panel name:** Contador ESP32
   - **Topic:** `esp32/thomazrb/contador` (use o mesmo tópico do código!)
   - **QoS:** 0 (já vem como padrão)
10. Toque em **"CREATE"**
11. As mensagens do ESP32 aparecerão automaticamente a cada 30 segundos!

### Entendendo o Código: Detalhes Importantes

#### WiFi e Credenciais

```cpp
const char* ssid = "SEU_WIFI_AQUI";
const char* password = "SUA_SENHA_AQUI";
```

Estas constantes armazenam as credenciais da sua rede WiFi. O ESP32 precisa dessas informações para se conectar à internet.

#### Configuração do Broker MQTT

```cpp
const char* mqtt_server = "broker.hivemq.com";
const int mqtt_port = 1883;
const char* mqtt_topic = "esp32/thomazrb/contador";
```

- **mqtt_server:** Endereço do broker público da HiveMQ
- **mqtt_port:** Porta padrão para MQTT sem criptografia (use 8883 para TLS/SSL)
- **mqtt_topic:** O "canal" onde publicaremos as mensagens. **Importante:** troque `thomazrb` por um identificador único seu para evitar conflito com outros usuários do broker público!

#### Temporização não-bloqueante

```cpp
unsigned long now = millis();
if (now - lastMsg > 30000) {
    // código executado a cada 30 segundos
}
```

Usamos `millis()` ao invés de `delay()` porque `delay()` bloqueia toda a execução do programa. Com `millis()`, o ESP32 pode continuar processando outras tarefas (como manter a conexão MQTT) enquanto aguarda os 30 segundos.

#### Client ID Único

```cpp
String clientId = "ESP32Client-";
clientId += String(random(0xffff), HEX);
```

Cada cliente MQTT precisa ter um ID único. Geramos um aleatório para evitar conflitos caso você conecte múltiplos ESP32s ao mesmo broker.

### O que é QoS (Quality of Service)?

QoS (Quality of Service) é um conceito importante no MQTT que define o nível de garantia de entrega das mensagens. Existem três níveis:

* **QoS 0 (At most once):** A mensagem é enviada uma vez, sem confirmação. É o mais rápido, mas pode perder mensagens se houver problemas na rede. Ideal para dados que podem ser perdidos ocasionalmente (como leituras de temperatura que se repetem a cada poucos segundos).

* **QoS 1 (At least once):** A mensagem é entregue pelo menos uma vez, com confirmação. Pode haver duplicatas. Bom para dados importantes onde duplicatas não são um problema.

* **QoS 2 (Exactly once):** A mensagem é entregue exatamente uma vez, com um protocolo de confirmação mais complexo. Mais lento, mas garante entrega única. Ideal para comandos críticos ou transações financeiras.

**No nosso exemplo usamos QoS 0** porque:
- É um contador simples que se repete a cada 30 segundos
- Se uma mensagem se perder, a próxima virá em breve
- Mantém o código e a comunicação mais simples e rápida

Para projetos onde cada mensagem é crítica (como alarmes, comandos de atuadores, ou dados que não se repetem), considere usar QoS 1 ou 2.

### Segurança e Boas Práticas

**Importante sobre Brokers Públicos:**

- O broker da HiveMQ é **público e sem autenticação**
- Qualquer pessoa pode ler suas mensagens se souber o tópico
- **Nunca envie dados sensíveis** (senhas, informações pessoais, etc.)
- Para projetos profissionais, use um broker privado com autenticação

**Dicas de Segurança:**

1. Use tópicos com identificadores únicos (ex: `esp32/usuario_12345/sensor`)
2. Para produção, considere usar brokers privados como:
   - **CloudMQTT** (agora parte da CloudAMQP)
   - **AWS IoT Core**
   - **Azure IoT Hub**
   - **Seu próprio servidor Mosquitto** ([https://mosquitto.org/](https://mosquitto.org/))
3. Implemente TLS/SSL para criptografar a comunicação
4. Use autenticação (username/password) sempre que possível

### Próximos Passos e Melhorias

Agora que você tem um ESP32 enviando dados via MQTT, aqui estão algumas ideias para expandir o projeto:

1. **Adicionar Sensores Reais:** Substitua o contador por leituras de sensores (temperatura, umidade, luminosidade)
2. **Receber Comandos:** Faça o ESP32 se inscrever em um tópico para receber comandos (ligar/desligar LEDs, por exemplo)
3. **Formato JSON:** Envie dados estruturados em JSON para facilitar o parsing
4. **Deep Sleep:** Use o modo de baixo consumo do ESP32 entre envios para economizar bateria

### Conclusão

Parabéns! Você criou sua primeira aplicação IoT com ESP32 e MQTT. Este exemplo simples demonstra os conceitos fundamentais que são a base de sistemas IoT complexos. O protocolo MQTT é extremamente versátil e escalável, sendo usado desde projetos caseiros até aplicações industriais.

A partir daqui, você tem uma base sólida para criar seus próprios projetos IoT, conectando sensores, atuadores e construindo dashboards personalizados. O ESP32 é uma plataforma poderosa e acessível, e o MQTT é o protocolo perfeito para fazer seus dispositivos conversarem!

**Recursos Adicionais:**
* [Documentação ESP32 Arduino Core](https://docs.espressif.com/projects/arduino-esp32/en/latest/)
* [PubSubClient Library Documentation](http://pubsubclient.knolleary.net/)
* [HiveMQ MQTT Essentials](https://www.hivemq.com/mqtt-essentials/)
* [MQTT.org - Official MQTT Site](https://mqtt.org/)