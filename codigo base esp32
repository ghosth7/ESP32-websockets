#include <WiFi.h>
#include <WebServer.h>
#include <WebSocketsServer.h>

const char* ssid = "NSSD";
const char* password = "123456789";

WebServer server(80);
WebSocketsServer webSocket = WebSocketsServer(81);

bool isAPMode = true;
bool outputActive = false;
const int salida1 = 12;
const int salida2 = 13;
const int salida3 = 14;
int rst = 0;

void handleRoot() {
  String html = R"=====(
  <!DOCTYPE html>
  <html lang="en">
  <head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>HMI Control Panel</title>
  <style>
      body {
          font-family: Arial, sans-serif;
          background-color: #222;
          display: flex;
          justify-content: center;
          align-items: center;
          height: 100vh;
          margin: 0;
      }
      .control-panel {
          width: 400px;
          background-color: #333;
          border: 2px solid #444;
          border-radius: 10px;
          padding: 20px;
          box-shadow: 0 0 20px rgba(0, 0, 0, 0.5);
          text-align: center;
      }
      .button {
          background-color: #4CAF50;
          border: none;
          color: white;
          padding: 15px 25px;
          font-size: 16px;
          margin: 10px 0;
          width: 100%;
          display: block;
          text-align: center;
          cursor: pointer;
          border-radius: 8px;
          transition: background-color 0.3s;
          box-shadow: 0 4px 6px rgba(0, 0, 0, 0.3);
      }
      .button:hover {
          background-color: #45a049;
      }
      .indicator-container {
          display: flex;
          justify-content: space-around;
          margin-top: 20px;
      }
      .indicator {
          width: 30px;
          height: 30px;
          background-color: #555;
          border-radius: 50%;
          margin-bottom: 10px;
          box-shadow: 0 0 5px rgba(0, 0, 0, 0.5);
          position: relative;
      }
      .start {
          background-color: green;
      }
      .stop {
          background-color: yellow;
      }
      .reset {
          background-color: red;
      }
      .emergency {
          background-color: white;
      }
      .indicator.on::after {
          content: "";
          position: absolute;
          top: 50%;
          left: 50%;
          transform: translate(-50%, -50%);
          background-color: #ff6f61;
          width: 12px;
          height: 12px;
          border-radius: 50%;
      }
  </style>
  </head>
  <body>

  <div class="control-panel">
      <button class="button" id="startButton">Start</button>
      <button class="button" id="stopButton">Stop</button>
      <button class="button" id="resetButton">Reset</button>
      <button class="button" id="hardResetButton">Hard Reset</button>
      <button class="button" id="emergencyButton">Emergencia</button>

      <div class="indicator-container">
          <div class="indicator" id="indicator1"></div>
          <div class="indicator" id="indicator2"></div>
          <div class="indicator" id="indicator3"></div>
      </div>

      <button class="button" id="modeButton">Switch Mode</button>
  </div>

  <script>
      const startButton = document.getElementById('startButton');
      const stopButton = document.getElementById('stopButton');
      const resetButton = document.getElementById('resetButton');
      const hardResetButton = document.getElementById('hardResetButton');
      const emergencyButton = document.getElementById('emergencyButton');

      const indicator1 = document.getElementById('indicator1');
      const indicator2 = document.getElementById('indicator2');
      const indicator3 = document.getElementById('indicator3');

      const ws = new WebSocket('ws://' + window.location.hostname + ':81');

      ws.addEventListener('open', function(event) {
          ws.send('init');
      });

      startButton.addEventListener('click', () => {
          ws.send('start');
      });

      stopButton.addEventListener('click', () => {
          ws.send('stop');
      });

      resetButton.addEventListener('click', () => {
          ws.send('reset');
      });

      hardResetButton.addEventListener('click', () => {
          ws.send('hardReset');
      });

      emergencyButton.addEventListener('click', () => {
          ws.send('emergency');
      });

      ws.addEventListener('message', (event) => {
          const data = event.data;

          // Eliminar todas las clases antes de agregar la nueva clase
          indicator1.className = 'indicator';
          indicator2.className = 'indicator';
          indicator3.className = 'indicator';

          switch(data) {
              case 'start':
                  indicator1.classList.add('start');
                  indicator2.classList.add('start');
                  indicator3.classList.add('start');
                  break;
              case 'stop':
                  indicator1.classList.add('stop');
                  indicator2.classList.add('stop');
                  indicator3.classList.add('stop');
                  break;
              case 'reset':
                  indicator1.classList.add('reset');
                  indicator2.classList.add('reset');
                  indicator3.classList.add('reset');
                  break;
              case 'hardReset':
                  // Lógica para manejar el hard reset
                  break;
              case 'emergency':
                  indicator1.classList.add('emergency');
                  indicator2.classList.add('emergency');
                  indicator3.classList.add('emergency');
                  break;
              default:
                  break;
          }
      });
  </script>

  </body>
  </html>
  )=====";
  server.send(200, "text/html", html);
}

void handleNotFound() {
  server.send(404, "text/plain", "Not found");
}

void webSocketEvent(uint8_t num, WStype_t type, uint8_t* payload, size_t length) {
  switch (type) {
    case WStype_TEXT:
      String msg = String((char*)payload);
      if (msg == "start" && rst == 0 && digitalRead(salida1) == LOW && digitalRead(salida2) == LOW && digitalRead(salida3) == LOW) {

        webSocket.broadcastTXT("start");
        outputActive = true;
        digitalWrite(salida1, HIGH);
        delay(500);
        digitalWrite(salida2, HIGH);
        digitalWrite(salida3, HIGH);
      } else if (msg == "stop" && rst == 0 && digitalRead(salida1) == HIGH && digitalRead(salida2) == HIGH && digitalRead(salida3) == HIGH) {
        webSocket.broadcastTXT("stop");
        outputActive = false;
        digitalWrite(salida2, LOW);
        digitalWrite(salida3, LOW);
        delay(500);
        digitalWrite(salida1, LOW);
      } else if (msg == "reset" && rst == 1) {
        webSocket.broadcastTXT("reset");
        outputActive = false;
        rst = 0;
      } else if (msg == "hardReset") {
        webSocket.broadcastTXT("reset");
        ESP.restart();
      } else if (msg == "emergency") {
        webSocket.broadcastTXT("emergency");
        rst = 1;
        digitalWrite(salida3, LOW);
        digitalWrite(salida2, LOW);
        delay(500);
        digitalWrite(salida1, LOW);
      } else if (msg == "init") {
        if (outputActive) {
          webSocket.broadcastTXT("start");
        }
      }
      if (digitalRead(salida1) == HIGH) {
        Serial.println("CA activada");
      }
      if (digitalRead(salida2) == HIGH) {
        Serial.println("CC AMP1 activado");
      }
      if (digitalRead(salida3) == HIGH) {
        Serial.println("CC AMP2 activado");
      }
      break;
  }
}

void setup() {
  Serial.begin(115200);
  pinMode(salida1, OUTPUT);
  pinMode(salida2, OUTPUT);
  pinMode(salida3, OUTPUT);
  if (isAPMode) {
    Serial.println("Starting in AP mode");
    WiFi.softAP(ssid, password);
    Serial.println("AP IP address: " + WiFi.softAPIP().toString());
  } else {
    Serial.println("Starting in Station mode");
    WiFi.begin(ssid, password);
    while (WiFi.status() != WL_CONNECTED) {
      delay(1000);
      Serial.println("Connecting to WiFi..");
    }
    Serial.println("Connected to WiFi");
    Serial.println("Station IP address: " + WiFi.localIP().toString());
  }

  server.on("/", HTTP_GET, handleRoot);
  server.onNotFound(handleNotFound);

  webSocket.begin();
  webSocket.onEvent(webSocketEvent);

  server.begin();
  Serial.println("HTTP server started");
}

void loop() {
  server.handleClient();
  webSocket.loop();
}
