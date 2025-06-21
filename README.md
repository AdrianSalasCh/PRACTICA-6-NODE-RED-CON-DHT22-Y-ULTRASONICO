# PRACTICA-6-NODE-RED-CON-DHT22-Y-ULTRASONICO
ESTE REPOSITORIO MUESTRA COMO PROGRAMAR CON DHT11 Y ULTRASONICO CON NODE-RED

## INTRODUCCIÓN

### DESCRIPCIÓN
**Node-RED** es una plataforma de desarrollo basada en flujos que permite integrar hardware, software y servicios en la nube, facilitando la implementación de soluciones orientadas al *Internet de las Cosas* (IoT). Esta herramienta posibilita la creación de flujos automatizados para la recepción, procesamiento, almacenamiento y visualización de datos provenientes de distintos dispositivos en tiempo real.

En la presente práctica, se utilizará Node-RED como herramienta de monitoreo para los datos generados en el simulador **WOKWI**. La recolección de información se llevará a cabo mediante una placa **ESP32**, la cual estará conectada a dos sensores específicos:

- Un sensor **DHT11**, encargado de medir la temperatura y la humedad ambiental.
- Un sensor ultrasónico **HV-SR04**, utilizado para determinar la distancia a objetos próximos.

Los datos obtenidos serán enviados a Node-RED, donde serán procesados y visualizados en tiempo real, lo que permitirá analizar el comportamiento de las variables monitoreadas durante el desarrollo de la práctica.


### MATERIAL NECESARIO

Para realizar esta practica necesitas lo siguiente
- [WOKWI](https://wokwi.com/)
- 1 PZ Tarjeta ESP 32
- 1 PZ Sensor HC-SR0 Ultrasonic Distance Sensor
- 1 PZ Sensor DHT11
- Node-RED

## INSTRUCCIONES PARA EL SIMULADOR WOKWI

### REQUISITOS PREVIOS
Para poder usar este repositorio necesitas entrar a la plataforma [WOKWI](https://wokwi.com/)

### Instrucciones de preparación de entorno

1. Abrir la terminal de programación y colocar el siguiente código:

```
#include <ArduinoJson.h>
#include <WiFi.h>
#include <PubSubClient.h>
#define BUILTIN_LED 2
#include "DHTesp.h"
const int DHT_PIN = 13;
DHTesp dhtSensor;
// Update these with values suitable for your network.
const int Trigger = 12;   //Pin digital 2 para el Trigger del sensor
const int Echo = 14;
const char* ssid = "Wokwi-GUEST"; //red wifi
const char* password = ""; //contraseña de la red wifi
const char* mqtt_server = "52.28.105.152"; //servidos mqtt adresses
String username_mqtt="DiploLASC";
String password_mqtt="12345";

WiFiClient espClient;
PubSubClient client(espClient);
unsigned long lastMsg = 0;
#define MSG_BUFFER_SIZE  (50)
char msg[MSG_BUFFER_SIZE];
int value = 0;

void setup_wifi() {

  delay(10);
  // We start by connecting to a WiFi network
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);
  pinMode(Trigger, OUTPUT); //pin como salida
  pinMode(Echo, INPUT);  //pin como entrada
  digitalWrite(Trigger, LOW);//Inicializamos el pin con 0
  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  randomSeed(micros());

  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}

void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
  }
  Serial.println();

  // Switch on the LED if an 1 was received as first character
  if ((char)payload[0] == '1') {
    digitalWrite(BUILTIN_LED, LOW);   
    // Turn the LED on (Note that LOW is the voltage level
    // but actually the LED is on; this is because
    // it is active low on the ESP-01)
  } else {
    digitalWrite(BUILTIN_LED, HIGH);  
    // Turn the LED off by making the voltage HIGH
  }

}

void reconnect() {
  // Loop until we're reconnected
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    // Create a random client ID
    String clientId = "ESP8266Client-";
    clientId += String(random(0xffff), HEX);
    // Attempt to connect
    if (client.connect(clientId.c_str(), username_mqtt.c_str() , password_mqtt.c_str())) {
      Serial.println("connected");
      // Once connected, publish an announcement...
      client.publish("outTopic", "hello world");
      // ... and resubscribe
      client.subscribe("inTopic");
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      // Wait 5 seconds before retrying
      delay(5000);
    }
  }
}

void setup() {
  pinMode(BUILTIN_LED, OUTPUT);     // Initialize the BUILTIN_LED pin as an output
  Serial.begin(115200);
  setup_wifi();
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
  dhtSensor.setup(DHT_PIN, DHTesp::DHT22);
  pinMode(Trigger, OUTPUT); //pin como salida
  pinMode(Echo, INPUT);  //pin como entrada
  digitalWrite(Trigger, LOW);//Inicializamos el pin con 0
}

void loop() {

 long t; //timepo que demora en llegar el eco
  long d; //distancia en centimetros

  digitalWrite(Trigger, HIGH);
  delayMicroseconds(10);          //Enviamos un pulso de 10us
  digitalWrite(Trigger, LOW);
  
  t = pulseIn(Echo, HIGH); //obtenemos el ancho del pulso
  d = t/59;             //escalamos el tiempo a una distancia en cm
  delay(1000);
  TempAndHumidity  data = dhtSensor.getTempAndHumidity();
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  unsigned long now = millis();
  if (now - lastMsg > 2000) {
    lastMsg = now;
    //++value;
    //snprintf (msg, MSG_BUFFER_SIZE, "hello world #%ld", value);

    StaticJsonDocument<128> doc;

    doc["DEVICE"] = "ESP32";
    doc["Nombre"] = "ADRIÁN SALAS";
    doc["TEMPERATURA"] = String(data.temperature, 1);
    doc["HUMEDAD"] = String(data.humidity, 1);
    doc["DISTANCIA"] = String(d);

    String output;
    
    serializeJson(doc, output);

    Serial.print("Publish message: ");
    Serial.println(output);
    Serial.println(output.c_str());
    client.publish("DiploLASC", output.c_str());
  }
}
```

2. Para garantizar el correcto funcionamiento de los sensores y la comunicación con la placa ESP32, es necesario instalar previamente las bibliotecas correspondientes. Las librerías requeridas se indican en la siguiente imagen:

![](https://github.com/AdrianSalasCh/PRACTICA-6-NODE-RED-CON-DHT22-Y-ULTRASONICO/blob/main/P6-LIBRERIAS.PNG)

La conexión del sensor ultrasónico HC-SR04 y del sensor de temperatura y humedad DHT11 con la placa ESP32 se realiza conforme al esquema mostrado en la siguiente imagen.

![](https://github.com/AdrianSalasCh/PRACTICA-6-NODE-RED-CON-DHT22-Y-ULTRASONICO/blob/main/P6-CONEXION%20WOKWI.PNG)

## INSTRUCCIONES PARA LA INSTALACIÓN DE Node-RED

### REQUISITOS PREVIOS

Para ejecutar adecuadamente el programa en Node-RED, es necesario contar previamente con la instalación de Node.js, versión v22.16.0 del siguiente enlace: [Node-RED](https://https://nodejs.org/en)

1. Instalar el programa correctamente.
2. Entrar al Simbolo del sistema (CMD) en modo administrador y escribir lo siguiente:

```
npm install -g --unsafe-perm node-red
```
3. Comprobar el funcionamiento de node-red con el siguente codigo: (con este mismo codigo podemos arrancar el programa siempre que lo necesitemos).

```
node-red
```

![](https://github.com/AdrianSalasCh/PRACTICA-6-NODE-RED-CON-DHT22-Y-ULTRASONICO/blob/main/NODE-RED%20P6-1.PNG)

4. Para abrir la aplicación nos vamos algun explorador y colocamos el siguente link:

```
localhost:1880
```

5. Instalar Dashboard abriendo la pestaña de opciones y elegimos *Manage palette*.

![](https://github.com/AdrianSalasCh/PRACTICA-6-NODE-RED-CON-DHT22-Y-ULTRASONICO/blob/main/P6-MANAGE%20PALETTE.PNG)

![](https://github.com/AdrianSalasCh/PRACTICA-6-NODE-RED-CON-DHT22-Y-ULTRASONICO/blob/main/P6-DASHBOARD.PNG)

6. Seleccionamos *Install* y buscamos *node-red-dashboard*.

## INSTRUCCIONES PARA LA COLOCACION DE BLOQUES Y LA CONEXION DE NODOS CORRESPONDIENTES EN Node-RED

1. Colocar y conectar los bloques de la siguiente manera:

![](https://github.com/AdrianSalasCh/PRACTICA-6-NODE-RED-CON-DHT22-Y-ULTRASONICO/blob/main/CONEXION%20NOD-RED%20P6.PNG)

2. Con doible click, configurar los bloques de la siguiente manera:

![](https://github.com/AdrianSalasCh/PRACTICA-6-NODE-RED-CON-DHT22-Y-ULTRASONICO/blob/main/CONEXION%20TERMINADA%20NOD-RED%20P6.PNG)

3. Realizar la configuración del bloque **mqtt** de acuerdo a la siguiente imágen:

![](https://github.com/AdrianSalasCh/PRACTICA-6-NODE-RED-CON-DHT22-Y-ULTRASONICO/blob/main/P6-MQTT-1.PNG)

![](https://github.com/AdrianSalasCh/PRACTICA-6-NODE-RED-CON-DHT22-Y-ULTRASONICO/blob/main/P6-MQTT-2.PNG)

4. Realizar la configuración del bloque **json** de acuerdo a la siguiente imágen:

![](https://github.com/AdrianSalasCh/PRACTICA-6-NODE-RED-CON-DHT22-Y-ULTRASONICO/blob/main/P6-JSON.PNG)

6. Realizar la configuración de los bloques de **funciones** de acuerdo a la siguiente imágen:

![](https://github.com/AdrianSalasCh/PRACTICA-6-NODE-RED-CON-DHT22-Y-ULTRASONICO/blob/main/P6-FUNCIONT.PNG)

![](https://github.com/AdrianSalasCh/PRACTICA-6-NODE-RED-CON-DHT22-Y-ULTRASONICO/blob/main/P6-FUNCIONH.PNG)

![](https://github.com/AdrianSalasCh/PRACTICA-6-NODE-RED-CON-DHT22-Y-ULTRASONICO/blob/main/P6-FUNCIOND.PNG)

7. Antes de configurar las gráficas, es necesario crear un nuevo *tab* desde el dashboard previamente instalado, en dicho *tab* se crearán los siguientes 2 *groups*.

![](https://github.com/AdrianSalasCh/PRACTICA-6-NODE-RED-CON-DHT22-Y-ULTRASONICO/blob/main/P6-TAB.PNG)

8. Para los *gauge nodes*, realizar la siguiente configuración:

![](https://github.com/AdrianSalasCh/PRACTICA-6-NODE-RED-CON-DHT22-Y-ULTRASONICO/blob/main/P6-GAUGET.PNG)

![](https://github.com/AdrianSalasCh/PRACTICA-6-NODE-RED-CON-DHT22-Y-ULTRASONICO/blob/main/P6-GAUGEH.PNG)

![](https://github.com/AdrianSalasCh/PRACTICA-6-NODE-RED-CON-DHT22-Y-ULTRASONICO/blob/main/P6-GAUGED.PNG)

9. Configurar el *chart node* de la siguiente manera:

![](https://github.com/AdrianSalasCh/PRACTICA-6-NODE-RED-CON-DHT22-Y-ULTRASONICO/blob/main/P6-CHART.PNG)

10. Seleccionar *Deploy* dar click en el siguiente simbolo:

![](https://github.com/AdrianSalasCh/PRACTICA-6-NODE-RED-CON-DHT22-Y-ULTRASONICO/blob/main/P6-DEPLOY.PNG)

11. Se abrirá una ventana y podrás obserbar los datos arrojados por el ESP32.

![](https://github.com/AdrianSalasCh/PRACTICA-6-NODE-RED-CON-DHT22-Y-ULTRASONICO/blob/main/P6-GRAFICAS-WOKWI-1.PNG)

![](https://github.com/AdrianSalasCh/PRACTICA-6-NODE-RED-CON-DHT22-Y-ULTRASONICO/blob/main/P6-GRAFICAS-1.PNG)

12. Al modifica los datos optenidos por los sensores, se modificarán automaticamente los gráficos en Node-RED.

![](https://github.com/AdrianSalasCh/PRACTICA-6-NODE-RED-CON-DHT22-Y-ULTRASONICO/blob/main/P6-GRAFICAS-WOKWI-2.PNG)

![](https://github.com/AdrianSalasCh/PRACTICA-6-NODE-RED-CON-DHT22-Y-ULTRASONICO/blob/main/P6-GRAFICAS-2.PNG)

## CRÉDITOS

Desarrollado por Ing. Luis Adrián Salas Chávez
- [GitHub](https://github.com/)
