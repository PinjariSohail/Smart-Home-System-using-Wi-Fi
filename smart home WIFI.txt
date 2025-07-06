#include <WiFi.h>

const char* ssid = "vivo 1919";
const char* password = "123456789";

const int ledPin = 2; 
const int moter = 4; 

WiFiServer server(80);

void setup() {
  pinMode(ledPin, OUTPUT);
  digitalWrite(ledPin, LOW);
  pinMode(moter,OUTPUT);
  digitalWrite(moter,LOW);
  
  Serial.begin(115200);
  delay(10);

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
  Serial.println("WiFi connected.");

  // Start the server
  server.begin();
  Serial.println("Server started");

  // Print the IP address
  Serial.println(WiFi.localIP());
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
  if (request.indexOf("/LED=ON") != -1) {
    digitalWrite(ledPin, HIGH);
  } else if (request.indexOf("/LED=OFF") != -1) {
    digitalWrite(ledPin, LOW);
  }
  if(request.indexOf("/MOTER=ON")!=-1){
    digitalWrite(moter,HIGH);
  }
  else if(request.indexOf("/MOTER=OFF")!=-1){
    digitalWrite(moter,LOW);
  }

  // Send HTTP response
  client.println("HTTP/1.1 200 OK");
  client.println("Content-Type: text/html");
  client.println(""); // Don't forget this line
  client.println("<!DOCTYPE HTML>");
  client.println("<html>");
  client.println("<head><title>ESP32 LED Control</title></head>");
  client.println("<style>");
  client.println("button { font-size: 24px; padding: 10px 20px; }");
  client.println("</style>");
  client.println("<body>");
  client.println("<h1>ESP32 LED Control and moter control</h1>");
  client.println("<p>Current LED Status: " + String(digitalRead(ledPin) == HIGH ? "On" : "Off") + "</p>");
  client.println("<p>Current moter Status: " + String(digitalRead(moter) == HIGH ? "off" : "On") + "</p>");
  client.println("<form action=\"/LED=ON\"><button>Turn On LED</button></form>");
  client.println(" ");
  client.println("<form action=\"/LED=OFF\"><button>Turn Off LED</button></form>");  
  client.println("<form action=\"/MOTER=OFF\"><button>Turn on moter</button></form>");
  client.println("<form action=\"/MOTER=ON\"><button>Turn off moter</button></form>");
  client.println("</body>");
  client.println("</html>");

  delay(1);
  Serial.println("Client disconnected");
}
