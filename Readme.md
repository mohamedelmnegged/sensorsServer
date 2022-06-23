# ESP8266 Asynchronous Web Server
## Installing Libraries 

#### ESP8266 Board in Arduino IDE
1. In your Arduino IDE, go to File> Preferences
2. Enter http://arduino.esp8266.com/stable/package_esp8266com_index.json 
   into the “Additional Boards Manager URLs” field as shown in the figure below. 
3. Then, click the “OK” button:
4. Go to Tools > Board > Boards Manager
5. Search for ESP8266 and press install button for the “ESP8266 by ESP8266 Community“:

#### DHT Library for ESP8266
1. Open your Arduino IDE and go to Tools > Manage Libraries. 
2. The Library Manager should open.
3. Search for “DHT” on the Search box and install the DHT library from Adafruit.
4. After installing the DHT library from Adafruit, type “Adafruit Unified Sensor” in the search box. 
5. Scroll all the way down to find the library and install it.

![DHT](./image/img1.jpg)


![DHT2](./image/img2.jpg)


#### ESPAsyncWebServer
1. Click [here](https://github.com/me-no-dev/ESPAsyncWebServer/archive/master.zip) to download the ESPAsyncWebServer library. 
2. Extract the .zip folder and you should get ESPAsyncWebServer-master folder
3. Rename your folder from ESPAsyncWebServer-master to ESPAsyncWebServer
4. Move the ESPAsyncWebServer folder to your Arduino IDE installation libraries folder

#### ESPAsync TCP
1. Click [here](https://github.com/me-no-dev/ESPAsyncTCP/archive/master.zip) to download the ESPAsyncTCP library. 
2. Extract the .zip folder and you should get ESPAsyncTCP-master folder
3. Rename your folder from ESPAsyncTCP-master to ESPAsyncTCP
4. Move the ESPAsyncTCP folder to your Arduino IDE installation libraries folder

*Finally, re-open your Arduino IDE*

Create new sketch and add the following code
```
/*********
  Rui Santos
  https://randomnerdtutorials.com/esp8266-dht11dht22-temperature-and-humidity-web-server-with-arduino-ide/
*********/

// Import required libraries
#include <Arduino.h>
#include <ESP8266WiFi.h>
#include <Hash.h>
#include <ESPAsyncTCP.h>
#include <ESPAsyncWebServer.h>
#include <Adafruit_Sensor.h>
#include <DHT.h>

// Replace with your network credentials
const char* ssid = "SSID";
const char* password = "PASSWORD";

#define DHTPIN 5     // Digital Pin D1

#define DHTTYPE    DHT11     // DHT 11

DHT dht(DHTPIN, DHTTYPE);

// current temperature & humidity, updated in loop()
float t = 0.0;
float h = 0.0;

// Create AsyncWebServer object on port 80
AsyncWebServer server(80);

// Generally, you should use "unsigned long" for variables that hold time
// The value will quickly become too large for an int to store
unsigned long previousMillis = 0;    // will store last time DHT was updated

// Updates DHT readings every 10 seconds
const long interval = 10000;  

const char index_html[] PROGMEM = R"rawliteral(
<!DOCTYPE HTML><html>
<head>
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link rel="stylesheet" href="https://use.fontawesome.com/releases/v5.7.2/css/all.css" integrity="sha384-fnmOCqbTlWIlj8LyTjo7mOUStjsKC4pOpQbqyi7RrhN7udi9RwhKkMHpvLbHG9Sr" crossorigin="anonymous">
  <style>
    html {
     font-family: Arial;
     display: inline-block;
     margin: 0px auto;
     text-align: center;
    }
    h2 { font-size: 3.0rem; }
    p { font-size: 3.0rem; }
    .units { font-size: 1.2rem; }
    .dht-labels{
      font-size: 1.5rem;
      vertical-align:middle;
      padding-bottom: 15px;
    }
  </style>
</head>
<body>
  <h2>ESP8266 DHT Server</h2>
  <p>
    <i class="fas fa-thermometer-half" style="color:#059e8a;"></i> 
    <span class="dht-labels">Temperature</span> 
    <span id="temperature">%TEMPERATURE%</span>
    <sup class="units">&deg;C</sup>
  </p>
  <p>
    <i class="fas fa-tint" style="color:#00add6;"></i> 
    <span class="dht-labels">Humidity</span>
    <span id="humidity">%HUMIDITY%</span>
    <sup class="units">%</sup>
  </p>
</body>
<script>
setInterval(function ( ) {
  var xhttp = new XMLHttpRequest();
  xhttp.onreadystatechange = function() {
    if (this.readyState == 4 && this.status == 200) {
      document.getElementById("temperature").innerHTML = this.responseText;
    }
  };
  xhttp.open("GET", "/temperature", true);
  xhttp.send();
}, 10000 ) ;

setInterval(function ( ) {
  var xhttp = new XMLHttpRequest();
  xhttp.onreadystatechange = function() {
    if (this.readyState == 4 && this.status == 200) {
      document.getElementById("humidity").innerHTML = this.responseText;
    }
  };
  xhttp.open("GET", "/humidity", true);
  xhttp.send();
}, 10000 ) ;
</script>
</html>)rawliteral";

// Replaces placeholder with DHT values
String processor(const String& var){
  //Serial.println(var);
  if(var == "TEMPERATURE"){
    return String(t);
  }
  else if(var == "HUMIDITY"){
    return String(h);
  }
  return String();
}

void setup(){
  // Serial port for debugging purposes
  Serial.begin(115200);
  dht.begin();
  
  // Connect to Wi-Fi
  WiFi.begin(ssid, password);
  Serial.println("Connecting to WiFi");
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println(".");
  }

  // Print ESP8266 Local IP Address
  Serial.println(WiFi.localIP());

  // Route for root / web page
  server.on("/", HTTP_GET, [](AsyncWebServerRequest *request){
    request->send_P(200, "text/html", index_html, processor);
  });
  server.on("/temperature", HTTP_GET, [](AsyncWebServerRequest *request){
    request->send_P(200, "text/plain", String(t).c_str());
  });
  server.on("/humidity", HTTP_GET, [](AsyncWebServerRequest *request){
    request->send_P(200, "text/plain", String(h).c_str());
  });

  // Start server
  server.begin();
}
 
void loop(){  
  unsigned long currentMillis = millis();
  if (currentMillis - previousMillis >= interval) {
    // save the last time you updated the DHT values
    previousMillis = currentMillis;
    // Read temperature as Celsius (the default)
    float newT = dht.readTemperature();
    // Read temperature as Fahrenheit (isFahrenheit = true)
    //float newT = dht.readTemperature(true);
    // if temperature read failed, don't change t value
    if (isnan(newT)) {
      Serial.println("Failed to read from DHT sensor!");
    }
    else {
      t = newT;
      Serial.println(t);
    }
    // Read Humidity
    float newH = dht.readHumidity();
    // if humidity read failed, don't change h value 
    if (isnan(newH)) {
      Serial.println("Failed to read from DHT sensor!");
    }
    else {
      h = newH;
      Serial.println(h);
    }
  }
}
```
![Result](./image/img3.jpg)

#### Install Mosquitto MQTT Broker on Raspberry Pi

1. Open a new Raspberry Pi terminal window.
2. Run the following command to upgrade and update your system:
```
sudo apt update && sudo apt upgrade
```
3.  To install the Mosquitto Broker enter these next commands:
```
sudo apt install -y mosquitto mosquitto-clients
```
4. To make Mosquitto auto start when the Raspberry Pi boots, you need to run the following command (this means that the Mosquitto broker will automatically start when the Raspberry Pi starts):
```
sudo systemctl enable mosquitto.service
```
5. Now, test the installation by running the following command:
```
mosquitto -v
```
6. Run the following command to open the mosquitto.conf file.
```
sudo nano /etc/mosquitto/mosquitto.conf
```
7. Move to the end of the file using the arrow keys and paste the following two lines:
```
listener 1883
allow_anonymous true
```
8. Then, press CTRL-X to exit and save the file. Press Y and Enter.
9. Reboot your Raspberry Pi with the following command for the changes to take effect.
```
sudo reboot
```

### Installing Flask
To install Flask, you’ll need to have pip installed. Run the following commands to update your Pi and install pip:

```
sudo apt-get update
sudo apt-get upgrade
sudo pip install flask
```

test flask 

```
mkdir FlaskTutorial
cd FlaskTutorial
touch app.py
chmod a+x app.py
./app.py

```

http://192.168.1.8:5000/


```
sudo pip install paho-mqtt
sudo pip install flask-socketio
```

```
mkdir web-server
cd web-server
nano app.py
mkdir templates
cd templates
nano main.html
```