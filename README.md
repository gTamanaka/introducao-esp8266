# Introdução ao uso do ESP8266

## Parte 1A: Piscar um LED
(Disponível em File -> Examples -> ESP8266 -> Blink)
Não é necessário fazer conexões.
Defina qual a placa, no menu Tools -> Board -> NodeMCU 1.0 (ESP12E) ou NodeMCU 0.9 (ESP12)
Escolha também a porta da placa em Tools -> Board -> Port (caso não tenha nada listado em portas, verifique as conexões, ou a placa, que pode estar com problemas)




```
  #include <Arduino.h>


  void setup() {
    // Define o pino 13 como saida
    //Módulo LOLIN - Pino D4
    pinMode(16, OUTPUT);
  }

  void loop() {
    digitalWrite(16, HIGH);   // Acende o Led
    delay(1000);              // Aguarda 1 segundo
    digitalWrite(16, LOW);    // Apaga o Led
    delay(1000);              // Aguarda 1 segundo
  }
```

![NodeMCU](/images/image6.png)

Figura: https://lowvoltage.github.io/images/NodeMCU-ESP8266-LEDs.jpg

**Atividade:** Tente mudar o intervalo com que o LED pisca.

## Parte 2B: Controlar LED com base em uma condição / sensor

```
 #include <Arduino.h>
void setup() {
  pinMode(2, OUTPUT);     // Initialize the LED_BUILTIN pin as an output
  pinMode(D1, INPUT);
}

// the loop function runs over and over again forever
void loop() {
  if (digitalRead(D1) == LOW) {
    digitalWrite(2, LOW);
  } else {
    digitalWrite(2, HIGH);
  }
}
```

- Grave o programa e tente conectar um fio entre o pino D1 e o GND (logicamente significa 0 ou LOW). Depois desconecte o fio do GND e conecte o fio no 3V3 (logicamente significa HIGH ou 1)

### Atenção: Cuidado com as conexões, pois curtos e conexões erradas podem queimar o módulo ESP8266!

Teste o sistema desconectado do PC, ou seja, ligado em um Power Bank ou fonte de celular. Note que o programa está no ESP8266 e não no PC! Ele funciona de forma autônoma!

Sensor / chave / botão:

Repita o teste com um switch (botão) ou chave capacitiva (botão de toque). Para tanto, faça as seguintes conexões:

+ 3V3 do NodeMCU no pino VIN ou 5VCC da placa do interruptor;
+ GND do NodeMCU no GND da placa do interruptor;
+ Signal / S / OUT do interruptor no pino D1 do NodeMCU.

**Atividade:** Tente fazer o LED piscar enquanto o botão está apertado, e parar de piscar quando não estiver apertado.

## Parte 2C - Listar redes Wifi
Abra o Exemplo WifiScan

File -> Examples -> ESP8266Wifi -> WifiScan

Grave o exemplo e veja o resultado das redes Wifi disponíveis pelo serial monitor

Monitore o resultado pelo Serial Monitor


  ![SerialMonitor](/images/image1.png)


**Atividade:** Tente fazer o NodeMCU listar as redes apenas quando o botão for apertado.

## Parte 2D - Controle via Wifi / Internet

Nesta prática vamos utilizar um rele, tomadas e vamos ligar / desligar qualquer dispositivo conectado à tomada via Web / Wifi.

Será necessário de uma rede Wifi / roteador para conexão. Você pode usar o hotspot do seu próprio celular ou a rede do laboratório.

Ligação no NodeMCU: Conectar um módulo Rele ao ESP8266
S do rele no pino D1 do NodeMCU + do rele no pino VIN (5 Volts) do NodeMCU - do rele no pino GND do NodeMCU

Ligação elétrica (127V / 220V) - Cuidado!
(Basicamente, é uma extensão elétrica com plugues macho e fêmea, de dois fios, com um dos fios controlado pelo rele, que por sua vez é controlado pelo NodeMCU)

![Ligação](/images/image8.png)

Figura: https://pandoralab.com.br/tutorial/tutorial-controle-de-equipamentos-por-bluetooth/

```
#include <ESP8266WiFi.h>

const char* ssid = "matheus";
const char* password = "JulioVerne";

int ledPin = 2; // GPIO13
WiFiServer server(80);

void setup() {
  Serial.begin(115200);
  delay(10);

  pinMode(ledPin, OUTPUT);
  digitalWrite(ledPin, LOW);

  // Connect to WiFi network
  Serial.println();
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);

  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("");
  Serial.println("WiFi connected");

  // Start the server
  server.begin();
  Serial.println("Server started");

  // Print the IP address
  Serial.print("Use this URL to connect: ");
  Serial.print("http://");
  Serial.print(WiFi.localIP());
  Serial.println("/");

}

void loop() {
  // Check if a client has connected
  WiFiClient client = server.available();
  if (!client) {
    return;
  }

  // Wait until the client sends some data
  Serial.println("new client");
  while(!client.available()){
    delay(1);
  }

  // Read the first line of the request
  String request = client.readStringUntil('\r');
  Serial.println(request);
  client.flush();

  // Match the request

  int value = LOW;
  if (request.indexOf("/LED=ON") != -1)  {
    digitalWrite(ledPin, HIGH);
    value = HIGH;
  }
  if (request.indexOf("/LED=OFF") != -1)  {
    digitalWrite(ledPin, LOW);
    value = LOW;
  }

// Set ledPin according to the request
//digitalWrite(ledPin, value);

  // Return the response
  client.println("HTTP/1.1 200 OK");
  client.println("Content-Type: text/html");
  client.println(""); //  do not forget this one
  client.println("<!DOCTYPE HTML>");
  client.println("<html>");

  client.print("Led pin is now: ");

  if(value == HIGH) {
    client.print("On");
  } else {
    client.print("Off");
  }
  client.println("<br><br>");
  client.println("<a href=\"/LED=ON\"\"><button>Turn On </button></a>");
  client.println("<a href=\"/LED=OFF\"\"><button>Turn Off </button></a><br />");
  client.println("</html>");

  delay(1);
  Serial.println("Client disonnected");
  Serial.println("");

}
```

Monitore o resultado pelo Serial Monitor

![NodeMCU](/images/image5.png)

Verifique o IP, acesse pelo seu navegador Web, e ligue / desligue o LED ou qualquer outro dispositivo!

**Atividade:** Faça um aplicativo no App Inventor para ligar / desligar o dispositivo pelo próprio aplicativo. Você pode usar botões, comando de voz, ou até a orientação (posição do celular). Dica: use o componente Web.

Observe que talvez este projeto só funciona na “rede local” - lembre da diferença entre IP privado e IP público. Vamos buscar um solução para isso nos próximos passos!

## Parte 2E - Sensor de temperatura
Para usar o sensor e necessário instalar uma biblioteca! 
Sketch -> Include Library -> Manage Libraries…

Procure por DHT e inclua DHT Sensor library for ESPx


Conexões - direta ou use o proto board!

![Ligação sensor de temperatura](/images/image13.png)
![Ligação sensor de temperatura 2](/images/image12.png)

(Disponível em File -> Examples -> DHT Sensor Library for DHTx)

```
#include "DHTesp.h"

#ifdef ESP32
#pragma message(THIS EXAMPLE IS FOR ESP8266 ONLY!)
#error Select ESP8266 board.
#endif

DHTesp dht;

void setup()
{
  Serial.begin(115200);
  Serial.println();
  Serial.println("Status\tHumidity (%)\tTemperature (C)\t(F)\tHeatIndex (C)\t(F)");
  String thisBoard= ARDUINO_BOARD;
  Serial.println(thisBoard);

  // Autodetect is not working reliable, don't use the following line
  // dht.setup(17);
  // use this instead: 
  dht.setup(04, DHTesp::DHT11); // Connect DHT sensor to GPIO 17
}

void loop()
{
  delay(dht.getMinimumSamplingPeriod());

  float humidity = dht.getHumidity();
  float temperature = dht.getTemperature();

  Serial.print(dht.getStatusString());
  Serial.print("\t");
  Serial.print(humidity, 1);
  Serial.print("\t\t");
  Serial.print(temperature, 1);
  Serial.print("\t\t");
  Serial.print(dht.toFahrenheit(temperature), 1);
  Serial.print("\t\t");
  Serial.print(dht.computeHeatIndex(temperature, humidity, false), 1);
  Serial.print("\t\t");
  Serial.println(dht.computeHeatIndex(dht.toFahrenheit(temperature), humidity, true), 1);

  delay(2000);
}
```

Monitore o resultado pelo Serial Monitor
![Serial Monitor 3](/images/image10.png)

**Atividade:** Faça a leitura do sensor ser disponibilizada via Web com o exemplo 2D via Web!

## Parte 2F - Requisição Web + Nuvem
Nesta parte vamos enviar dados de sensores para um servidor em Nuvem!

Mantenha as ligações do sensor DHT11 da Parte 2E.

Desenho: 
NodeMCU -> Wifi -> Internet -> Servidor na Nuvem (robotica.ufscar.br)

Usaremos requisições HTTP!


Escolha um ID único para seu sensor, e faça os testes com o seu ID. Primeiro tente enviar números diferentes pela leitura= no envio.php e depois veja o resultado na visualização.

```
#include <Arduino.h>

#include <ESP8266WiFi.h>
#include <ESP8266WiFiMulti.h>

#include <ESP8266HTTPClient.h>

#define USE_SERIAL Serial

ESP8266WiFiMulti WiFiMulti;

#include "DHTesp.h"

#ifdef ESP32
#pragma message(THIS EXAMPLE IS FOR ESP8266 ONLY!)
#error Select ESP8266 board.
#endif

DHTesp dht;

void setup() {

  USE_SERIAL.begin(115200);
  // USE_SERIAL.setDebugOutput(true);

  USE_SERIAL.println();
  USE_SERIAL.println();
  USE_SERIAL.println();

  for (uint8_t t = 4; t > 0; t--) {
    USE_SERIAL.printf("[SETUP] WAIT %d...\n", t);
    USE_SERIAL.flush();
    delay(1000);
  }

  WiFi.mode(WIFI_STA);
  WiFiMulti.addAP("matheus", "JulioVerne");

  String thisBoard = ARDUINO_BOARD;
  Serial.println(thisBoard);

  // Autodetect is not working reliable, don't use the following line
  // dht.setup(17);
  // use this instead:
  dht.setup(04, DHTesp::DHT11); // Connect DHT sensor to GPIO 17
}

void loop() {
  // wait for WiFi connection
  if ((WiFiMulti.run() == WL_CONNECTED)) {

    HTTPClient http;

    USE_SERIAL.print("[HTTP] begin...\n");
    // configure traged server and url


    //http.begin("http://user:password@192.168.1.12/test.html");

    delay(dht.getMinimumSamplingPeriod());

    float humidity = dht.getHumidity();
    float temperature = dht.getTemperature();

    Serial.print(dht.getStatusString());
    Serial.print("\t");
    Serial.print(humidity, 1);
    Serial.print("\t\t");
    Serial.print(temperature, 1);
    Serial.print("\t\t");
    Serial.print(dht.toFahrenheit(temperature), 1);
    Serial.print("\t\t");
    Serial.print(dht.computeHeatIndex(temperature, humidity, false), 1);
    Serial.print("\t\t");
    Serial.println(dht.computeHeatIndex(dht.toFahrenheit(temperature), humidity, true), 1);


    String envio;
    envio = "http://200.133.229.248/MBI/envio.php?id=34&leitura=" + String(temperature);
    http.begin(envio);


    USE_SERIAL.print("[HTTP] GET...\n");
    // start connection and send HTTP header
    int httpCode = http.GET();

    // httpCode will be negative on error
    if (httpCode > 0) {
      // HTTP header has been send and Server response header has been handled
      USE_SERIAL.printf("[HTTP] GET... code: %d\n", httpCode);

      // file found at server
      if (httpCode == HTTP_CODE_OK) {
        String payload = http.getString();
        USE_SERIAL.println(payload);
      }
    } else {
      USE_SERIAL.printf("[HTTP] GET... failed, error: %s\n", http.errorToString(httpCode).c_str());
    }

    http.end();
  }

  delay(10000);
}
```


**Atividades:**
- Verifique pelo Serial Monitor se os dados estão sendo enviados!
- Utilize o endereço para verificar os dados recebidos http://200.133.229.248/MBI/tr.php?id=34
- Tente mudar a taxa de atualização dos dados (intervalo entre envios);
- Tente preparar o sistema para enviar dados de dois sensores (use outro ID e o sensor de umidade);
- Faça um App no App Inventor para visualizar a última leitura usando os dados da URL http://200.133.229.248/MBI/ultima.php?id=34
- Os exemplos acima são para o id 34;
- Desligue o NodeMCU do PC e ligue ele a um power bank ou carregador de celular e veja que o sistema funciona, mesmo sem PC!
- **Desafio:** Tente fazer um programa para ligar / desligar o rele através de Web Requests e não conexão direta, resolvendo assim possíveis problemas de NAT.

