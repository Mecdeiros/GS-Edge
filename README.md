# EcoWork: Solução IoT para Saúde e Bem-estar no Trabalho Híbrido 

![Status](https://img.shields.io/badge/Status-Conclu%C3%ADdo-brightgreen)
![Technology](https://img.shields.io/badge/Tecnologia-ESP32%20%7C%20MQTT-blue)

Este projeto apresenta uma solução de Internet das Coisas (IoT) desenvolvida para monitorar e melhorar a qualidade do ambiente de trabalho (Home Office ou Presencial). O sistema utiliza sensores para analisar temperatura, umidade e luminosidade, oferecendo feedback visual imediato e enviando diagnósticos para a nuvem via protocolo MQTT.

## Integrantes do Grupo
* **Guilherme de Medeiros** - RA: [561699]
* **Victor Pucci Ferreira** - RA: [561736]
* **Murilo Henrique Vieira Cruz** - RA: [563743]


---

##  Descrição do Problema
Com o avanço do trabalho remoto e híbrido, profissionais frequentemente negligenciam a ergonomia ambiental.
* **Fadiga Visual:** Iluminação inadequada causa dores de cabeça e cansaço.
* **Desconforto Térmico e Respiratório:** Ar muito seco ou temperaturas extremas reduzem a produtividade e aumentam o risco de doenças.
* **Falta de Consciência:** Sem monitoramento, o usuário não percebe a degradação do ambiente até sentir os sintomas físicos.

##  Solução Proposta
O **EcoWork** é uma estação de monitoramento inteligente que atua em duas frentes:
1.  **Feedback Local (Visual):** Um semáforo de LEDs indica a gravidade da situação em tempo real (Verde = Ideal, Amarelo = Atenção, Vermelho = Crítico).
2.  **Conectividade em Nuvem:** Os dados são processados e uma string de diagnóstico (ex: *"AR SECO: Risco de Irritação"*) é enviada via MQTT para um dashboard remoto.

---

##  Hardware e Componentes
O projeto foi simulado na plataforma **Wokwi** utilizando a arquitetura ESP32.

* **Microcontrolador:** ESP32 DevKit V1.
* **Sensores:**
    * **DHT22:** Monitoramento de Temperatura (°C) e Umidade (%).
    * **Módulo LDR (Fotoresistor):** Monitoramento de Luminosidade (Lux/Nível).
* **Atuadores (Indicadores):**
    * 1x LED Verde (Ambiente Ideal).
    * 1x LED Amarelo (Alerta/Transição).
    * 1x LED Vermelho (Risco à Saúde).
* **Resistores:** 220Ω para os LEDs.

### Diagrama de Ligações (Pinout)

| Componente | Pino Componente | Pino ESP32 (GPIO) | Função |
| :--- | :--- | :--- | :--- |
| **LED Verde** | Anodo (+) / Perna Torta | **D4** | Indicador de "Ambiente Ideal" |
| **LED Amarelo** | Anodo (+) / Perna Torta | **D5** | Indicador de "Atenção" |
| **LED Vermelho**| Anodo (+) / Perna Torta | **D2** | Indicador de "Crítico/Perigo" |
| **Sensor DHT22**| SDA (Dados) | **D15** | Leitura Temp/Umidade |
| **Sensor LDR** | AO (Saída Analógica) | **D34** | Leitura de Luz |

---

## Comunicação e Protocolos (MQTT)

A comunicação do dispositivo com a "nuvem" é feita através do protocolo **MQTT** (Message Queuing Telemetry Transport), ideal para IoT devido à sua leveza.

* **Broker Utilizado:** `broker.hivemq.com` (Broker Público).
* **Porta:** `1883` (TCP).
* **Cliente MQTT:** Biblioteca `PubSubClient` no ESP32.

### Tópicos e Payload
O dispositivo publica as informações no seguinte tópico:

> **Tópico:** `trabalhoiot/GS2/status`

**Exemplo de Payload (Mensagem enviada):**
```json
"[AR SECO: Irritação Respiratória] [ESCURO: Fadiga Visual]"
O sistema concatena múltiplos problemas em uma única mensagem de diagnóstico para facilitar a leitura no dashboard
```
##  Instruções de Uso e Replicação

### 1. Simulação Wokwi
1. Acesse o link do projeto: **[[WokWi](https://wokwi.com/projects/448083382773072897)]**
2. Caso for montar do zero, certifique-se de instalar as bibliotecas no **Library Manager** (ícone "+"):
   * `DHT sensor library` (Adafruit).
   * `PubSubClient` (Nick O'Leary).

### 2. Dashboard (Monitoramento)
Para visualizar os dados enviados pelo ESP32:

1. Abra o [HiveMQ Websocket Client](http://www.hivemq.com/demos/websocket-client/).
2. Clique em **Connect**.
3. Em **Subscriptions**, clique em *Add New Topic Subscription*.
4. No campo **Topic**, digite: `trabalhoiot/GS2/status` (substitua pelo nome usado no código).
5. Acompanhe as mensagens de diagnóstico chegando em tempo real conforme você altera os sensores no Wokwi.

---

##  Demonstração em Vídeo

Confira o vídeo explicativo demonstrando o funcionamento do circuito, a lógica dos LEDs e a comunicação MQTT em tempo real:

**[Video youtube](https://youtu.be/P7QhwTaht58)**

---

##  Código Fonte (Comentado)

O código principal (`sketch.ino`) implementa a leitura dos sensores, a lógica de decisão (IF/ELSE) para os LEDs e a conexão WiFi/MQTT.

```cpp
/*
 * PROJETO: EcoWork - Monitoramento de Saúde Ocupacional
 * PLATAFORMA: ESP32 (Wokwi Simulator)
 * LINGUAGEM: C++ / Arduino
 */

#include <WiFi.h>
#include <PubSubClient.h>
#include <DHT.h>

// --- DEFINIÇÃO DE PINOS ---
#define PIN_DHT 15        // Sensor de Temperatura/Umidade
#define PIN_LDR 34        // Sensor de Luz (AO do módulo)
#define PIN_LED_CRITICO 2 // LED Vermelho
#define PIN_LED_ATENCAO 5 // LED Amarelo
#define PIN_LED_IDEAL 4   // LED Verde
#define DHTTYPE DHT22     

// --- CONFIGURAÇÕES DE REDE E MQTT ---
const char* ssid = "Wokwi-GUEST"; // Rede virtual do simulador
const char* password = "";
const char* mqtt_server = "broker.hivemq.com"; 
const int mqtt_port = 1883;
const char* topic_status = "trabalhoiot/GS2/status"; 

DHT dht(PIN_DHT, DHTTYPE);
WiFiClient espClient;
PubSubClient client(espClient);

void setup() {
  Serial.begin(115200);
  
  // Configura modos dos pinos
  pinMode(PIN_LED_CRITICO, OUTPUT);
  pinMode(PIN_LED_ATENCAO, OUTPUT);
  pinMode(PIN_LED_IDEAL, OUTPUT);
  pinMode(PIN_LDR, INPUT);

  dht.begin();
  setup_wifi();
  client.setServer(mqtt_server, mqtt_port);
}

void setup_wifi() {
  // Conecta ao WiFi do simulador
  Serial.print("Conectando WiFi");
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("Conectado!");
}

void reconnect() {
  // Reconecta ao Broker MQTT se cair
  while (!client.connected()) {
    String clientId = "ESP32-Saude-" + String(random(0xffff), HEX);
    if (client.connect(clientId.c_str())) {
      Serial.println("MQTT Conectado!");
    } else {
      delay(5000);
    }
  }
}

void loop() {
  if (!client.connected()) reconnect();
  client.loop();

  // --- 1. LEITURA ---
  float temp = dht.readTemperature();
  float umid = dht.readHumidity();
  int luz = analogRead(PIN_LDR); 

  if (isnan(temp) || isnan(umid)) return;

  // --- 2. PROCESSAMENTO LÓGICO ---
  String diagnostico = "";
  int nivelSeveridade = 0; // 0=Verde, 1=Amarelo, 2=Vermelho

  // Regras de Temperatura
  if (temp > 30) {
    diagnostico += "[CALOR: Risco Desidratação] ";
    nivelSeveridade = 2;
  } else if (temp > 26) {
    diagnostico += "[QUENTE: Beba Água] ";
    if(nivelSeveridade < 1) nivelSeveridade = 1;
  } else if (temp < 18) {
    diagnostico += "[FRIO: Desconforto] ";
    if(nivelSeveridade < 1) nivelSeveridade = 1;
  }

  // Regras de Umidade
  if (umid < 30) {
    diagnostico += "[AR SECO: Irritação] ";
    if(nivelSeveridade < 1) nivelSeveridade = 1;
  } else if (umid > 75) {
    diagnostico += "[ÚMIDO: Risco Mofo] ";
    if(nivelSeveridade < 1) nivelSeveridade = 1;
  }

  // Regras de Luz
  if (luz > 3000) { // Ajustar conforme calibração do LDR no Wokwi
    diagnostico += "[ESCURO: Fadiga Visual] ";
    if(nivelSeveridade < 2) nivelSeveridade = 2;
  } else if (luz < 500) {
    diagnostico += "[OFUSCAMENTO: Cansaço Visual] ";
    if(nivelSeveridade < 1) nivelSeveridade = 1;
  }

  if (diagnostico == "") {
    diagnostico = "Ambiente Perfeito";
    nivelSeveridade = 0;
  }

  // --- 3. ATUADORES ---
  digitalWrite(PIN_LED_CRITICO, LOW);
  digitalWrite(PIN_LED_ATENCAO, LOW);
  digitalWrite(PIN_LED_IDEAL, LOW);

  if (nivelSeveridade == 2) digitalWrite(PIN_LED_CRITICO, HIGH);
  else if (nivelSeveridade == 1) digitalWrite(PIN_LED_ATENCAO, HIGH);
  else digitalWrite(PIN_LED_IDEAL, HIGH);

  // --- 4. PUBLICAÇÃO MQTT ---
  client.publish(topic_status, diagnostico.c_str());
  Serial.println(diagnostico); // Debug local
  
  delay(2000);
}
