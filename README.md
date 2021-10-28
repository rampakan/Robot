/* include library */
#include <ESP8266WiFi.h>
#include <PubSubClient.h>
#include "DHT.h"
#define MQTT_SERVER "broker.emqx.io"
#define MQTT_PORT 1883
#define MQTT_NAME "robot123"
/* define port */
WiFiClient client;
PubSubClient mqtt(client);
#define DHTPIN D7 // what pin we're connected to
#define DHTTYPE DHT11 // DHT 11 
/* WIFI settings */
const char* ssid = " wifi ";
const char* password = " pw ";

String temp_str;
char temp[50];

/* data received from application */
String  data = "";

/* define L298N or L293D motor control pins */
int IN1 = D3;     
int IN2 = D4;   
int IN3 = D5;    
int IN4 = D6;  
unsigned long previousMillis = 0;
DHT dht(DHTPIN, DHTTYPE);
void setup()
{
  Serial.begin(115200);
  /* initialize motor control pins as output */
  pinMode(IN1, OUTPUT);
  pinMode(IN2, OUTPUT);
  pinMode(IN3, OUTPUT);
  pinMode(IN4, OUTPUT);

  //connect to your local wi-fi network

  WiFi.begin(ssid, password);

  // Attempt to connect to WiFi network:
  while (WiFi.status() != WL_CONNECTED)
  {
    Serial.print(".");
    // Connect to WPA/WPA2 network. Change this line if using open or WEP  network:
    // Wait 3 seconds for connection:
    delay(3000);
  }

  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());   //You can get IP address assigned to ESP
  connectMqtt();
  dht.begin();

}

void loop()
{
  unsigned long currentMillis = millis();

  if (currentMillis - previousMillis >= 10000) {
    previousMillis = currentMillis;

    float t = dht.readTemperature();
    temp_str = String(t);
    temp_str.toCharArray(temp, temp_str.length() + 1);

    mqtt.publish("temp_por", temp);
    
    if (isnan(t)) {
      Serial.println("Failed to read from DHT");
    } else {
      
    }
  }
  /************************ Run function according to incoming data from application *************************/

  /* If the incoming data is "forward", run the "MotorForward" function */
  if (data == "W") TurnLeft();
  /* If the incoming data is "backward", run the "MotorBackward" function */
  else if (data == "S") TurnRight();
  /* If the incoming data is "left", run the "TurnLeft" function */
  else if (data == "A") MotorForward();
  /* If the incoming data is "right", run the "TurnRight" function */
  else if (data == "D") MotorBackward();
  /* If the incoming data is "stop", run the "MotorStop" function */
  else if (data == "B") MotorStop();

  mqtt.loop();
}


/********************************************* FORWARD *****************************************************/
void MotorForward(void)
{
  digitalWrite(IN1, HIGH);
  digitalWrite(IN2, LOW);
  digitalWrite(IN3, HIGH);
  digitalWrite(IN4, LOW);
}

/********************************************* BACKWARD *****************************************************/
void MotorBackward(void)
{
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, HIGH);
  digitalWrite(IN3, LOW);
  digitalWrite(IN4, HIGH);
}

/********************************************* TURN LEFT *****************************************************/
void TurnLeft(void)
{
  digitalWrite(IN1, HIGH);
  digitalWrite(IN2, LOW);
  digitalWrite(IN3, LOW);
  digitalWrite(IN4, HIGH);
}

/********************************************* TURN RIGHT *****************************************************/
void TurnRight(void)
{
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, HIGH);
  digitalWrite(IN3, HIGH);
  digitalWrite(IN4, LOW);
}

/********************************************* STOP *****************************************************/
void MotorStop(void)
{
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, LOW);
  digitalWrite(IN3, LOW);
  digitalWrite(IN4, LOW);
}

void connectMqtt()
{
  mqtt.setServer(MQTT_SERVER, MQTT_PORT);
  mqtt.setCallback(callback);

  if (mqtt.connected() == false)
  {
    Serial.print("MQTT connection... ");
    if (mqtt.connect(MQTT_NAME))
    {
      Serial.println("connected");
      mqtt.subscribe("robot");
    }
  }
}
void callback(char *topic, byte *payload, unsigned int length)
{
  payload[length] = '\0';
  String topic_str = topic, payload_str = (char *)payload;
  Serial.println("[" + topic_str + "]: " + payload_str);

  data = payload_str;
}
