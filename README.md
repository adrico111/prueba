#include <ESP8266WebServer.h>
#include <ESP8266WiFi.h>
#include <WiFiClient.h>
#include <ESP8266mDNS.h>

///temperatura//
#include "DHT.h" 
#define DHTPIN D2
#define DHTTYPE DHT11 
MDNSResponder mdns;

//---------------------------------------------------------------
ESP8266WebServer server(80);
DHT dht(DHTPIN, DHTTYPE); 

//-------------------VARIABLES GLOBALES--------------------------
int contconexion = 0;

int LED = 16;
const char *ssid = "MOVISTAR_A4E0";
const char *password = "Yd9jJPHDKP5FEhKUf5t4";

String XML, xmlTemperatura; 

unsigned long previousMillis = 0;

//--------CODIGO HTML y JavaScript-------------
String webSite = "<!DOCTYPE html>"
"<html>"
"<head>"
"<meta charset='utf-8' />"
"<title>Web Server</title>"
"<link rel=\"icon\" href=\"imagenes/favicon.ico\">"
"<script type='text/javascript'>"
"function loadDoc(){"
"  var xhttp = new XMLHttpRequest();"
"  xhttp.onreadystatechange = function() {"
"    if (this.readyState == 4 && this.status == 200) {"
"      myFunction(this);"
"    }"
"  };"
"  xhttp.open('GET','xml',true);"
"  xhttp.send();"
"  setTimeout('loadDoc()',500);"
"}"
"function myFunction(xml){"
"  var i;"
"  var xmlDoc = xml.responseXML;"
"  var dato ='';"
"  dato = xmlDoc.getElementsByTagName('TEMPERATURA')[0].childNodes[0].nodeValue;"
"  document.getElementById('temperatura').innerHTML = dato;"
"}"
"</script>"
"<style>"
"  button{"
"   background:#161F27;"
"   color:#fff;"
"   font-size:1.25em;"
"   margin:20px"
"   padding:10px 0px;"
"   text-align:center;"
"   width:150;"
"   high:100;"
"}"
"  h1,a,p{"
"   color:#ffffff;"
"}"
"  body{"
"   background-color:#0066aa;"
"}"
"  #temperatura{"
"   font-weight:bolder;"
"}"
"</style>"
"</head>"
"<body onload='loadDoc()'>"
"<h1>Pagina Web ESP8266</h1>"
"<a>TEMPERATURA = </a>"
"<a id='temperatura'></a>"

"<p><strong>LED</strong> = <a href=\"socket1On\"><button>ON</button></a>&nbsp;<a href=\"socket1Off\"><button>OFF</button></a></p>"
"</body>"
"</html>";

void construirXML(){
  XML="";
  XML+="<TEMPERATURA>";
  XML+=xmlTemperatura;
  XML+="</TEMPERATURA>";
}

void handleWebsite(){
  server.send(200,"text/html",webSite);
}

void handleXML(){

  construirXML();
  server.send(200,"text/xml",XML);
}


//---------------------------SETUP-------------------------------
void setup() {

  // Inicia Serial
  Serial.begin(9600);
  Serial.println("");

  pinMode(LED,OUTPUT);

  // Conexión WIFI
  Serial.print("Conectando con ");
  Serial.println(ssid);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED){
    delay(500);
    Serial.print(".");
  }
  Serial.println("");
  Serial.print("Connected to ");
  Serial.println(ssid);
  Serial.print("IP address: ");
  Serial.println(WiFi.localIP());
  
  if (mdns.begin("esp8266", WiFi.localIP())) {
    Serial.println("MDNS responder started");
  }

  server.on("/",handleWebsite);
  server.on("/xml",handleXML);

  server.on("/", [](){
    server.send(200, "text/html", webSite);
  });
  server.on("/socket1On", [](){
    server.send(200, "text/html", webSite);
    digitalWrite(LED, HIGH);
    delay(1000);
  });
  server.on("/socket1Off", [](){
    server.send(200, "text/html", webSite);
    digitalWrite(LED, LOW);
    delay(1000); 
  });
  server.begin();

}

//--------------------------LOOP--------------------------------
void loop() {


    float temp=dht.readTemperature();
    float hum=dht.readHumidity();
   
    xmlTemperatura = String(temp, 1)+"           ;HUMEDAD="+String(hum, 1); //1 decimal
   // xmlhumedad=String(hum, 1);
  

  server.handleClient();
}
