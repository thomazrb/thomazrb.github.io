---
title: "Guia Prático: Enviando Dados do ESP32 para o Firebase Firestore via API REST"
date: 2025-09-22T14:37:23-03:00
draft: false
tags: ["ESP32", "Firebase", "Firestore", "IoT", "API REST", "ArduinoJson"]
categories: ["IoT", "ESP32"]
---

A necessidade de armazenar dados na nuvem é um pilar fundamental em projetos de Internet das Coisas (IoT). Seja para registrar leituras de sensores, monitorar o estado de um dispositivo ou criar logs de eventos, ter um banco de dados acessível e escalável é essencial. O ESP32, em suas várias versões, combinado com o Firebase Firestore do Google, forma uma dupla poderosa e acessível para desenvolvedores e hobbistas.

Neste tutorial técnico, vamos explorar o método mais leve e universal para enviar dados de um ESP32 para o Firestore: utilizando a **API REST nativa**. Em vez de depender de bibliotecas pesadas do Firebase, vamos construir requisições HTTP do zero. Essa abordagem não apenas economiza memória e recursos preciosos do microcontrolador, mas também aprofunda o entendimento sobre como as APIs web funcionam.

O objetivo é claro: ler um valor (neste caso, simulado) no ESP32, conectá-lo ao Wi-Fi e criar um novo registro (documento) em uma coleção no Firestore.

### A Arquitetura da Solução: Conversando com a Nuvem via HTTP

Antes de mergulhar no código, é crucial entender o fluxo de comunicação. O Firestore, como muitos serviços de nuvem modernos, expõe uma API REST (Representational State Transfer). Em termos simples, isso significa que podemos interagir com nosso banco de dados enviando requisições HTTP para URLs específicas, da mesma forma que um navegador acessa um site.

Nosso fluxo de trabalho será o seguinte:

1.  **Conexão**: O ESP32 se conecta a uma rede Wi-Fi para obter acesso à internet.
2.  **Formatação dos Dados**: Os dados a serem enviados (ex: ID do sensor e valor) são estruturados em um formato JSON específico que a API do Firestore exige.
3.  **Requisição HTTP POST**: O ESP32 envia os dados JSON para uma URL única que representa nossa coleção no Firestore. A requisição é do tipo `POST`, que é o padrão para criar novos recursos.
4.  **Criação do Documento**: Se a requisição for bem-sucedida, o Firestore cria um novo documento na coleção especificada com os dados que enviamos.

### Parte 1: Configuração do Projeto no Firebase

O primeiro passo é preparar o terreno no lado do Firebase.

#### 1.1. Criar um Projeto Firebase

Acesse o [Console do Firebase](https://console.firebase.google.com/), clique em "Adicionar projeto" e siga as instruções para criar um novo projeto.

#### 1.2. Ativar o Firestore Database

No menu lateral esquerdo do seu projeto, navegue até **Build > Firestore Database** e clique em "Criar banco de dados".

* Inicie em **modo de produção**.
* Escolha a localização do servidor que for mais próxima de você (ex: `southamerica-east1` para São Paulo).

#### 1.3. Configurar Regras de Segurança

Para este tutorial, vamos abrir o acesso ao banco de dados. **Atenção: estas regras não são seguras para um ambiente de produção**, pois permitem que qualquer pessoa leia e escreva em seu banco de dados.

* Na aba "Regras" do Firestore, substitua o conteúdo pelas seguintes linhas e clique em "Publicar":

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      // Permite leitura e escrita sem autenticação.
      // Apenas para fins de desenvolvimento.
      allow read, write: if true;
    }
  }
}
```
#### 1.4. Obter as Credenciais (ID do Projeto e Chave de API)

Precisamos de duas informações para que nosso ESP32 possa se autenticar na API.

1.  **ID do Projeto**: No menu, clique no ícone de engrenagem ⚙️ > **Configurações do projeto**. Na aba "Geral", você encontrará o **ID do projeto**. Copie este valor.
2.  **Chave de API da Web**: Nesta mesma página, role para baixo até a seção "Seus apps". Se não houver nenhum app, clique no ícone da Web (`</>`) para registrar um novo.
    * Dê um apelido (ex: "App ESP32") e clique em "Registrar app".
    * Na tela seguinte, o Firebase mostrará um objeto de configuração `firebaseConfig`. Copie o valor da `apiKey`. Essa é a sua **Chave de API da Web**.

Com o ID e a Chave em mãos, o Firebase está pronto.

### Parte 2: Preparando o Ambiente Arduino

Agora, vamos para o nosso código. Precisamos de uma biblioteca para facilitar a manipulação do formato JSON.

1.  **Core do ESP32**: Certifique-se de que o suporte para placas ESP32 está instalado em sua Arduino IDE.
2.  **Instalar a Biblioteca `ArduinoJson`**: Esta biblioteca é essencial para construir o corpo (payload) da nossa requisição HTTP de forma eficiente.
    * Vá em `Ferramentas > Gerenciar Bibliotecas...`.
    * Pesquise por `ArduinoJson` de Benoit Blanchon e instale a versão mais recente.

As bibliotecas `WiFi.h` e `HTTPClient.h`, que usaremos para a conexão e para a requisição, já vêm incluídas no core do ESP32.

### Parte 3: O Código do ESP32

Este é o código completo. Copie-o para um novo sketch na sua Arduino IDE e preencha as constantes com as suas informações.

```cpp
#include <WiFi.h>
#include <HTTPClient.h>
#include <ArduinoJson.h>

// --- PREENCHA COM SUAS INFORMAÇÕES ---
const char* WIFI_SSID = "NOME_DA_SUA_REDE_WIFI";
const char* WIFI_PASSWORD = "SENHA_DA_SUA_REDE_WIFI";

const char* FIREBASE_PROJECT_ID = "SEU_ID_DE_PROJETO_FIREBASE";
const char* FIREBASE_API_KEY = "SUA_CHAVE_DE_API_WEB";
// -----------------------------------------

// O nome da coleção onde os dados serão salvos no Firestore
const char* COLLECTION_NAME = "leituras_sensores";

void setup() {
  Serial.begin(115200);
  Serial.println("Iniciando...");

  // Conectar ao Wi-Fi
  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  Serial.print("Conectando ao Wi-Fi");
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nConectado com sucesso!");
  Serial.print("Endereço IP: ");
  Serial.println(WiFi.localIP());
}

void loop() {
  // Gera um valor de sensor aleatório para o exemplo
  int sensorValue = random(0, 1024);
  
  Serial.printf("Enviando dados para o Firestore: sensorId = 1, valor = %d\n", sensorValue);
  
  // Chama a função que envia os dados
  sendDataToFirestore(1, sensorValue);

  // Espera 30 segundos antes de enviar o próximo dado
  delay(30000); 
}

void sendDataToFirestore(int sensorId, int value) {
  if (WiFi.status() != WL_CONNECTED) {
    Serial.println("Erro: Não conectado ao Wi-Fi.");
    return;
  }

  // Monta a URL da API REST do Firestore.
  // Ao fazer um POST para o nome da coleção, o Firestore cria um ID de documento aleatório.
  String url = "https://firestore.googleapis.com/v1/projects/" + String(FIREBASE_PROJECT_ID) +
               "/databases/(default)/documents/" + String(COLLECTION_NAME) +
               "?key=" + String(FIREBASE_API_KEY);

  // Cria o payload (corpo) da requisição no formato JSON exigido pelo Firestore.
  // Esta estrutura é a parte mais crítica e fonte comum de erros.
  StaticJsonDocument<256> jsonDoc;
  
  JsonObject fields = jsonDoc.createNestedObject("fields");

  JsonObject sensorIdField = fields.createNestedObject("sensorId");
  sensorIdField["integerValue"] = sensorId;

  JsonObject valueField = fields.createNestedObject("valor");
  valueField["integerValue"] = value;

  String jsonPayload;
  serializeJson(jsonDoc, jsonPayload);

  // Inicia a requisição HTTP
  HTTPClient http;
  http.begin(url);
  http.addHeader("Content-Type", "application/json");

  Serial.println("--- INICIANDO REQUISIÇÃO HTTP ---");
  Serial.println("URL: " + url);
  Serial.println("Payload: " + jsonPayload);

  // Envia a requisição POST com o payload JSON
  int httpResponseCode = http.POST(jsonPayload);

  // Verifica a resposta do servidor
  if (httpResponseCode > 0) {
    String responsePayload = http.getString();
    Serial.print("Código de resposta HTTP: ");
    Serial.println(httpResponseCode);
    Serial.print("Resposta: ");
    Serial.println(responsePayload);
  } else {
    Serial.print("Erro na requisição POST. Código de erro: ");
    Serial.println(httpResponseCode);
  }

  Serial.println("--- FIM DA REQUISIÇÃO HTTP --- \n");

  // Libera os recursos
  http.end();
}
```
#### Dissecando o Código

* **`sendDataToFirestore()`**: Esta função encapsula toda a lógica de comunicação.
* **A URL da API**: A `String url` é construída dinamicamente com seu ID de projeto e chave de API. Note o endpoint: `/v1/projects/.../documents/{NOME_DA_COLECAO}`. Ao enviar um `POST` para este endereço, instruímos o Firestore a criar um novo documento dentro da coleção `leituras_sensores`.
* **O Payload JSON**: Este é o ponto mais importante. A API REST do Firestore não aceita um JSON simples como `{"sensorId": 1, "valor": 123}`. Em vez disso, ela exige uma estrutura mais detalhada, onde cada campo é um objeto que especifica seu tipo de dado (`integerValue`, `stringValue`, `booleanValue`, etc.). A biblioteca `ArduinoJson` torna a criação dessa estrutura complexa uma tarefa simples e segura.
* **A Requisição `HTTPClient`**: Criamos uma instância de `HTTPClient`, definimos a URL de destino (`http.begin(url)`), adicionamos o cabeçalho `Content-Type: application/json` para informar ao servidor que estamos enviando JSON, e finalmente executamos a requisição com `http.POST(jsonPayload)`.
* **Verificando a Resposta**: Um código de resposta `200` significa que o documento foi criado com sucesso. Qualquer outro código (especialmente na faixa `4xx`) indica um erro que pode ser depurado observando a resposta do servidor no Monitor Serial.

### Testando a Integração

1.  Faça o upload do código para o seu ESP32.
2.  Abra o Monitor Serial (baud rate 115200).
3.  Observe a saída. Você deverá ver o ESP32 se conectar ao Wi-Fi e, a cada 30 segundos, imprimir os detalhes da requisição HTTP e a resposta do servidor.
4.  Acesse o Console do Firebase e vá para o seu banco de dados Firestore. Você verá a coleção `leituras_sensores` e, dentro dela, documentos sendo criados com IDs aleatórios. Ao clicar em um documento, você verá os campos `sensorId` e `valor` com os dados enviados.

### Conclusão e Próximos Passos

Conseguimos! Com apenas as bibliotecas nativas do ESP32 e a `ArduinoJson`, estabelecemos uma ponte robusta entre o mundo físico (nosso microcontrolador) e a nuvem. Esta abordagem via API REST é extremamente poderosa por ser universal, leve e compatível com praticamente qualquer dispositivo capaz de fazer uma requisição HTTP.

A partir daqui, as possibilidades são imensas:

* **Segurança**: Substitua as regras abertas por um sistema de autenticação, como o Firebase Authentication, para proteger seus dados.
* **Estruturas de Dados**: Envie dados mais complexos, como strings (`stringValue`), coordenadas geográficas (`geoPointValue`) ou timestamps (`timestampValue`).
* **Leitura de Dados**: Explore as requisições `GET` da API REST para ler dados do Firestore e exibi-los no seu dispositivo, criando um sistema de comunicação bidirecional.

Agora você tem a base fundamental para construir projetos IoT muito mais complexos e conectados. Boas criações!