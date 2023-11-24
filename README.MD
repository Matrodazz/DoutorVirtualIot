# Doutor Virtual
## Proposta
Nos últimos anos, temos testemunhado avanços extraordinários na área da saúde com a ajuda da tecnologia e inovação, e gostaríamos de apresentar a proposta do Doutor Virtual! Atualmente, profissionais da saúde enfrentam o desafio de lidar com uma quantidade massiva de dados em registros médicos, relatórios e exames todos os dias. A nossa proposta visa otimizar esse processo por meio da aplicação de uma Inteligência Artificial, que recebe o texto sobre sintomas e comportamentos do paciente e descreve um diagnóstico automaticamente para o paciente durante a triagem, sendo utilizado como base pelos médicos de plantão na hora da consulta. Eventualmente, o sistema poderia até mesmo checar registros médicos dos pacientes para aumentar a base de conhecimento e oferecer diagnósticos ainda mais precisos e prevenções para complicações futuras.

Imagine um cenário em que um paciente em uma área remota pode receber um diagnóstico rápido e preciso, possibilitando tratamentos precoces e potencialmente salvando vidas. Essa é a premissa do Doutor Virtual.

## Circuito eletronico
    * 1 Placa ESP32
    * 2 LEDS
    * 2 Resistores 220 Ω
    * 1 Sensor DHT temperatura e umidade

![Diagram](https://i.imgur.com/yAnuVwK.png)

## Codigo circuito eletronico

```
#include <WiFi.h>
#include <PubSubClient.h>
#include "DHT.h"

#define DHTPIN 2 // Porta sensor
#define verde 19 // Porta led verde
#define vermelho 18 // Porta led vermelho

#define DHTTYPE DHT22
DHT dht(DHTPIN, DHTTYPE);

#define wifi_ssid "Wokwi-GUEST"
#define wifi_password ""
#define mqtt_server "test.mosquitto.org"

#define temperature_topic "sensor/temperature/iotandai" // Topico
#define humidity_topic "sensor/humidity"

WiFiClient espClient;
PubSubClient client(espClient);

void setup() {
  Serial.begin(9600);
  dht.begin();
  setup_wifi();
  client.setServer(mqtt_server, 1883);

  pinMode(verde, OUTPUT); // LED Verde
  pinMode(vermelho, OUTPUT); 
}

void setup_wifi() {
  delay(10);
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(wifi_ssid);

  WiFi.begin(wifi_ssid, wifi_password);

  int attempts = 0;
  while (WiFi.status() != WL_CONNECTED && attempts < 40) {
    delay(500);
    Serial.print(".");
    attempts++;
  }

  if (WiFi.status() != WL_CONNECTED) {
    Serial.println("\nFailed to connect to WiFi. Please check your credentials.");
    while (1) {
      delay(1000);
    }
  }

  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}

void loop() {
  delay(2000);

  float h = dht.readHumidity();
  float t = dht.readTemperature();

  if (isnan(h) || isnan(t)) {
    Serial.println("Failed to read from DHT sensor!");
    return;
  }

  Serial.print("Temperature: ");
  Serial.print(t);
  Serial.print(" *C ");

  // Lógica de controle de temperatura
  if (t <= 37) {
    digitalWrite(verde, HIGH);
    digitalWrite(vermelho, LOW);
  } else {
    digitalWrite(verde, LOW);
    digitalWrite(vermelho, HIGH);
  }

  client.connect("ESP32Client");
  client.publish(temperature_topic, String(t).c_str(), true);
  client.publish(humidity_topic, String(h).c_str(), true);
  client.disconnect();
}

```
---


## Fluxos Node-red

![Diagram](https://i.imgur.com/dhWC835.png)

---

# Como replicar
Instale o node red na sua maquina
`npm install -g --unsafe-perm node-red`

Após instalado, inicie o node-red através do CMD
`node-red`

Acesse a url no seu navegador, fornecida nas logs de inicialização. Em Manage Pallete na aba lateral, instale o pacote node-red-dashboard.

Com o pacote já instalado, monte um fluxo com três nós: Gauge, MQQTin e Debug.

Configure o MQQTin para receber os dados do broker e do topico que o circuito está enviando as informações.