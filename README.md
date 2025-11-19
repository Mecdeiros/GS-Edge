# EcoWork 2.0: Solução IoT para Saúde e Bem-estar no Trabalho Híbrido 

![Status](https://img.shields.io/badge/Status-Conclu%C3%ADdo-brightgreen)
![Technology](https://img.shields.io/badge/Tecnologia-ESP32%20%7C%20MQTT-blue)

Este projeto apresenta uma solução de Internet das Coisas (IoT) desenvolvida para monitorar e melhorar a qualidade do ambiente de trabalho (Home Office ou Presencial). O sistema utiliza sensores para analisar temperatura, umidade e luminosidade, oferecendo feedback visual imediato e enviando diagnósticos para a nuvem via protocolo MQTT.

## 👥 Integrantes do Grupo
* **[Nome do Aluno 1]** - RA: [000000]
* **[Nome do Aluno 2]** - RA: [000000]
* **[Nome do Aluno 3]** - RA: [000000]
* **[Nome do Aluno 4]** - RA: [000000]

---

##  Descrição do Problema
Com o avanço do trabalho remoto e híbrido, profissionais frequentemente negligenciam a ergonomia ambiental.
* **Fadiga Visual:** Iluminação inadequada causa dores de cabeça e cansaço.
* **Desconforto Térmico e Respiratório:** Ar muito seco ou temperaturas extremas reduzem a produtividade e aumentam o risco de doenças.
* **Falta de Consciência:** Sem monitoramento, o usuário não percebe a degradação do ambiente até sentir os sintomas físicos.

##  Solução Proposta
O **EcoWork 2.0** é uma estação de monitoramento inteligente que atua em duas frentes:
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

> **Tópico:** `trabalhoiot/[SEU_NOME]/status`

**Exemplo de Payload (Mensagem enviada):**
```json
"[AR SECO: Irritação Respiratória] [ESCURO: Fadiga Visual]"
