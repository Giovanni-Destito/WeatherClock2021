// Esp8266 Install, NodeMCU 1.0.
// Bibliotecas Instalar: Json 5.13.5, Timek, LiquidCrystal_I2C,
DFRobotDFPlayerMini.

#include <DFRobotDFPlayerMini.h>
#include <ArduinoJson.h>
#include <SoftwareSerial.h>
#include <ESP8266WiFi.h>
#include <WiFiClient.h>
#include <LiquidCrystal_I2C.h>
#include <Wire.h>
#include <WiFiUdp.h>
#include <TimeLib.h>


char json[] =
"{\"sensor\":\"gps\",\"time\":1351824120,\"data\":[48.756080,2.302038]}";

// Wi-fi Casa
char ssid[] = "MANHA ";
char pass[] = "acdz6835mmcd";

//Wi-fi Roteador - Testes
//char ssid[] = "Teste";
//char pass[] = "qwedf123";

bool menu = 0;
float leitura;
int i;
int CTemp;
WiFiClient client;

//http://api.openweathermap.org/data/2.5/weather?lat=-21.1775&lon=-
47.81028&appid=1bb6549e9fde7c87edbf3ee0054134da
// Nome do Servidor do OpenWeatherMap API.
const char server[] = "api.openweathermap.org"; //Site que fornece
informações da JSON para a API Arduino.

String longitude = "-47.8103"; // Definir Longitude da cidade ( Tabela Site
OpenWeatherMap ).
String latitude = "-21.1775"; // Definir Latitude da cidade ( Tabela Site
OpenWeatherMap ).

//API Key
String apiKey = "3e062574ce72ea83b8f8aa93b861184a"; // Definir API
OpenWeatherMap.

String text;
String payload;
String description;

int jsonend = 0;
boolean startJson = false;
int status = WL_IDLE_STATUS;

#define JSON_BUFF_DIMENSION 2500

unsigned long lastConnectionTime = 10 * 60 * 1000; // Ultima conexão com o
servidor.
const unsigned long postInterval = 10 * 60 * 1000; // Intervalo de cada
Request de API (10L * 1000L; 10 Segundos).

LiquidCrystal_I2C lcd = LiquidCrystal_I2C(0x27, 20, 4); //20x4 LCD.

byte Gota[8] = { //Gotinha.
0b00100,
0b00100,
0b01110,
0b01110,
0b11111,
0b10111,
0b10011,
0b01110
};

byte Celc[8] = { //°C.
0b11000,
0b11000,
0b00110,
0b01001,
0b01000,
0b01000,
0b01001,
0b00110
};

byte UmZ[8] = { //10.
0b10010,
0b10101,
0b10101,
0b10101,
0b10101,
0b10101,
0b10101,
0b10010
};

byte Zero[8] = { //0.
0b01000,
0b10100,
0b10100,
0b10100,
0b10100,
0b10100,
0b10100,
0b01000
};

//https://lastminuteengineers.com/arduino-1602-character-lcd-tutorial/

//-------------------------------- DECLARAÇÕES - DFPlayer ------------------------------

SoftwareSerial mySoftwareSerial(D6, D5); //RX, TX
DFRobotDFPlayerMini myDFPlayer;
#define volumeMP3 100

//-------------------------------- DECLARAÇÕES - Relógio --------------------------------

static const char ntpServerName[] = "pool.ntp.br";
const int timeZone = -3; // Definir GMT -3 Brasil.

WiFiUDP Udp;
unsigned int localPort = 8888; // Porta local para ouvir pacotes UDP.

time_t getNtpTime();
void digitalClockDisplay();
void printDigits(int digits);
void sendNTPpacket(IPAddress &address);


void setup() {

 pinMode(D7,INPUT_PULLUP);

 Serial.begin(9600);
 text.reserve(JSON_BUFF_DIMENSION);
 WiFi.begin(ssid, pass);
 Serial.println("connecting"); // Teste Conexão Wi-Fi.
 lcd.setCursor(3, 1);
 lcd.print("Conectando");

 while (WiFi.status() != WL_CONNECTED) {
 delay(500);
 Serial.print(".");
 lcd.setCursor(13, 1);
 lcd.print(".");
 delay(500);
 lcd.setCursor(14, 1);
 lcd.print(".");
 delay(500);
 lcd.setCursor(15, 1);
 lcd.print(".");
 delay(500);
 lcd.setCursor(13, 1);
 lcd.print(" ");
 }

 Serial.println("WiFi Connected");
 lcd.setCursor(3, 1);
 lcd.print("Conectado!! ");

//------------------------------- COMANDO NATIVO - Relógio ----------------------------
{
 Serial.begin(9600);
 while (!Serial) ;
 delay(250);

 Serial.println(WiFi.localIP());
 Serial.println("Starting UDP");
 Udp.begin(localPort);
 Serial.println(Udp.localPort());
 setSyncProvider(getNtpTime);
 setSyncInterval(300);
}
//----------------------------- FIM COMANDO NATIVO - Relógio ------------------------
Serial.begin(9600);
 pinMode(D7,INPUT);

 mySoftwareSerial.begin(9600); //9600 a Serial do DFPlayer.
 if (!myDFPlayer.begin(mySoftwareSerial)) { //Use softwareSerial to
communicate with MP3.
 /* Serial.println(F("Unable to begin:"));
 Serial.println(F("1.Please recheck the connection!"));
 Serial.println(F("2.Please insert the SD card!")); */

 //Morre aqui, se não conseguir iniciar o módulo.
 //while(true){
 // delay(0); // Code to compatible with ESP8266 watch dog.
 }
}

time_t prevDisplay = 0; // Quando o relógio digital foi exibido.
// Criando Caractere.

//----------------------------------- COMEÇO LOOP - Relógio -----------------------------
void digitalClockDisplay()
{
 // Mostra valor da hora digital no monitor serial.
 if (hour() < 10){
 Serial.print('0'); // Se as horas estiverem um valor menor do que 10,
imprimir um "0" a esquerda.
 lcd.setCursor(14, 2);
 lcd.print("0");
 lcd.setCursor(15, 2);
 lcd.print(hour());
 }
 Serial.print(hour()); // Testar valor menor que 10, na print deve dar conflito
devido o calor ser menor que 2 digitos

 if (hour() >= 10){
 lcd.setCursor(14, 2);
 lcd.print(hour());
 }

 printDigits(minute());
/* printDigits(second());
 Serial.print(" ");
 Serial.print(day());
 Serial.print(".");
 Serial.print(month());
 Serial.print(".");
 Serial.print(year());
 Serial.println();*/ // Valores de DIA, MÊS, ANO em Standby.


if ( menu == 1 ){

 myDFPlayer.playMp3Folder(hour());
delay(2000);
 myDFPlayer.playMp3Folder(195);
delay(2000);

 myDFPlayer.playMp3Folder(minute());
delay(2000);
 myDFPlayer.playMp3Folder(188);
delay(1500);
}
}

void printDigits(int digits)
{
 // Utilizado para exibição de relógio digital: imprime dois pontos precedentes
e à esquerda 0
 Serial.print(":"); // Separação de horas e minutos ":"
 lcd.setCursor(16, 2);
 lcd.print(":");

 if (digits < 10){
 Serial.print('0'); // Se os minutos estiverem um valor menor do que 10,
imprimir um "0" a esquerda.
 lcd.setCursor(17, 2);
 lcd.print("0");
 lcd.setCursor(18, 2);
 lcd.print(digits);
}

 if (digits >= 10){
 lcd.setCursor(17, 2);
 lcd.print(digits);
}
 Serial.print(digits); // Printa o valor de Minutos.
}

/*-------- NTP code ----------*/

const int NTP_PACKET_SIZE = 48; // O tempo NTP está nos primeiros 48
bytes da mensagem.
byte packetBuffer[NTP_PACKET_SIZE]; // Buffer para armazenar pacotes de
entrada e saída.
time_t getNtpTime()
{
 IPAddress ntpServerIP; // IP NTP Address.

 while (Udp.parsePacket() > 0) ; // Descartar todos os pacotes recebidos
anteriormente.
 Serial.println("Transmit NTP Request");

 // Obter um servidor aleatório do pool.
 WiFi.hostByName(ntpServerName, ntpServerIP);
 Serial.print(ntpServerName);
 Serial.print(": ");
 Serial.println(ntpServerIP);
 sendNTPpacket(ntpServerIP);
 uint32_t beginWait = millis();
 while (millis() - beginWait < 3000) { // Tempo de atualização do Horário.
 int size = Udp.parsePacket();
 if (size >= NTP_PACKET_SIZE) {
 Serial.println("Receive NTP Response");
 Udp.read(packetBuffer, NTP_PACKET_SIZE); // Ler o pacote no buffer.
 unsigned long secsSince1900;
 // Converter quatro bytes começando na localização 40 em um inteiro
longo.
 secsSince1900 = (unsigned long)packetBuffer[40] << 24;
 secsSince1900 |= (unsigned long)packetBuffer[41] << 16;
 secsSince1900 |= (unsigned long)packetBuffer[42] << 8;
 secsSince1900 |= (unsigned long)packetBuffer[43];
 return secsSince1900 - 2208988800UL + timeZone * SECS_PER_HOUR;
 }
 }
 Serial.println("Sem Resposta Horário:");
 return(1); // Testar para ver se este valor esta correto,ou seja, caso dê falha
na leitura automaticamente reiniciar...................
}

// Enviar uma solicitação NTP para o servidor de horário no endereço
fornecido.

void sendNTPpacket(IPAddress &address)
{
 // Defina todos os bytes no buffer para 0.
 memset(packetBuffer, 0, NTP_PACKET_SIZE);
 // Inicializar os valores necessários para formar a solicitação NTP.
 // (Veja a URL acima para detalhes sobre os pacotes)
 packetBuffer[0] = 0b11100011; // LI, Versão, Modo
 packetBuffer[1] = 0; // Stratum, or tipo do relógio.
 packetBuffer[2] = 6; // Intervalo de votação.
 packetBuffer[3] = 0xEC; // Precisão do relógio par.
 // 8 bytes de zero para Root Delay e Root Dispersion.
 packetBuffer[12] = 49;
 packetBuffer[13] = 0x4E;
 packetBuffer[14] = 49;
 packetBuffer[15] = 52;
 // todos os campos NTP receberam valores.
 // você pode enviar um pacote solicitando um carimbo de data / hora:
 Udp.beginPacket("pool.ntp.br", 123); // NTP os pedidos são para a porta 123.
 Udp.write(packetBuffer, NTP_PACKET_SIZE);
 Udp.endPacket();
}

//------------------------------------- FIM LOOP - Relógio ------------------------------------

 // Imprimir o status do Wifi.
 void printWiFiStatus() {
 // Imprimir a Senha.
 Serial.print("SSID: ");
 Serial.println(WiFi.SSID());

 // Imprimir Nome da rede Wi-fi.
 IPAddress ip = WiFi.localIP();
 Serial.print("IP Address: ");
 Serial.println(ip);

 // Imprimir a velocidade de conexão com o Wi-fi.
 long rssi = WiFi.RSSI();
 Serial.print("signal strength (RSSI):");
 Serial.print(rssi);
 Serial.println(" dBm");
}

void makehttpRequest() {
 // Feche qualquer conexão antes de enviar uma nova solicitação para
permitir que o cliente estabeleça conexão com o servidor.
 client.stop();

 // Se a conexão foi bem sucedida
 if (client.connect(server, 80)) {
 // Serial.println("connecting...");
 // Envia os valores das declarações para fazer a solicitação em HTTP.

 //http://api.openweathermap.org/data/2.5/weather?lat=-21.1775&lon=-
47.81028&appid=1bb6549e9fde7c87edbf3ee0054134da.
 client.println("GET /data/2.5/weather?lat=" + latitude + "&lon=" + longitude
+ "&APPID=" + apiKey /*+ "&mode=json&units=metric&cnt=2 HTTP/1.1"*/);
 client.println("Host: api.openweathermap.org");
 client.println("User-Agent: ArduinoWiFi/1.1");
 client.println("Connection: close");
 client.println();

 unsigned long timeout = millis();
 while (client.available() == 0) {
 if (millis() - timeout > 6000) {
 Serial.println(">>> Client Timeout !");
 client.stop();
 return;
 }
 }

 char c = 0;
 while (client.available()) {
 c = client.read();
 // uma vez que json contém o mesmo número de chaves abertas e
fechadas, isso significa que podemos determinar quando um json é completamente
recebido. contando
 // As ocorrências de abertura e fechamento.
 // Serial.print(c); //Teste para ler a solicitação HTTP
 if (c == '{') {
 startJson = true; // "Set startJsontrue" Para indicar que a mensagem
json foi iniciada.
 jsonend++;
 }
 if (c == '}') {
 jsonend--;
 }
 if (startJson == true) {
 text += c;
 }
 // Se jsonend = 0, então recebemos o mesmo número de chaves.
 if (jsonend == 0 && startJson == true) {
 parseJson(text.c_str()); // Analisar o texto da string c na função
parseJson.
 text = ""; // Limpar a string de texto para a próxima vez.
 startJson = false; // Defina startJson como false para indicar que
uma nova mensagem ainda não começou.
 }
 }
 }
 else {
 // Se nenhuma conexão foi feita:
 Serial.println("connection failed");
 return;
 }
}

// Para analisar os dados json recebidos do OWM.
void parseJson(const char * jsonString) {

if (digitalRead(D7) == 1){
bool menu = 1;

}
 //StaticJsonBuffer<4000> jsonBuffer;
 const size_t bufferSize = 2 * JSON_ARRAY_SIZE(1) +
JSON_ARRAY_SIZE(2) + 4 * JSON_OBJECT_SIZE(1) + 3 *
JSON_OBJECT_SIZE(2) + 3 * JSON_OBJECT_SIZE(4) + JSON_OBJECT_SIZE(5)
+ 2 * JSON_OBJECT_SIZE(7) + 2 * JSON_OBJECT_SIZE(8) + 720;
 DynamicJsonBuffer jsonBuffer(bufferSize);

 // ENCONTRAR CAMPOS NA ÁRVORE JSON.
 JsonObject& root = jsonBuffer.parseObject(jsonString);
 if (!root.success()) {
 Serial.println("parseObject() failed");
 return;
 }

 JsonArray& list = root["main"];

 //JsonObject& nowT = list[0];
 //JsonObject& later = list[1];
 //Serial.println("Teste");
 //for (i=0;i<bufferSize;i++){
 // Serial.print(jsonString[i]);
 //}
 payload = jsonString;
 Serial.println(payload);


 /*
 lcd.print("Tempo:");
 String weather = root["weather"][0]["main"];
 if (weather == "Clouds"){
 Serial.println("Nuvens");
 lcd.setCursor(0,7);
 lcd.print("Nuvens");
 }
 if (weather == "Rain"){
 Serial.println("Chuva");
 lcd.setCursor(0,7);
 lcd.print("Chuva");
 }
 if (weather == "Smoke"){
 Serial.println("Fumaça");
 lcd.setCursor(0,7);
 lcd.print("Fumaça");
 }
 if (weather == "Clean"){
 Serial.println("Limpo");
 lcd.setCursor(0,7);
 lcd.print("Limpo");
 }
 if (weather == "Snow"){
 Serial.println("Neve");
 lcd.setCursor(0,7);
 lcd.print("Neve");
 }
 if (weather == "Thunderstorm"){
 Serial.println("Temp.Areia");
 lcd.setCursor(0,7);
 lcd.print("Temp.Areia");
 }
 if (weather == "Drizzle"){
 Serial.println("Chuvisco");
 lcd.setCursor(0,7);
 lcd.print("Chuvisco");
 }
 */

 lcd.init(); // Iniciar o lcd
 lcd.backlight();
 lcd.createChar(0, Gota);
 lcd.createChar(1, Celc);
 lcd.createChar(2, UmZ);
 lcd.createChar(3, Zero);
 lcd.clear();


 String description = root["weather"][0]["description"];
 //Serial.println(description);
 if (description == "clear sky") {
 Serial.println("Céu Limpo");
 lcd.setCursor(0, 0);
 lcd.print("Ceu Limpo ");
 }
 if (description == "clear sky" && menu == 1){
 myDFPlayer.playMp3Folder(189);
delay (3000);
 myDFPlayer.playMp3Folder(133);
delay (2400);
 }

 if (description == "few clouds") {
 Serial.println("Poucas Nuvens");
 lcd.setCursor(0, 0);
 lcd.print("Poucas Nuvens ");
 }
 if ( description == "few clouds" && menu == 1){
 myDFPlayer.playMp3Folder(189);
delay (3000);
 myDFPlayer.playMp3Folder(131);
delay (2000);
 }

 if (description == "scattered clouds") {
 Serial.println("Poucas Nuvens");
 lcd.setCursor(0, 0);
 lcd.print("Poucas Nuvens ");
 }
 if ( description == "scattered clouds" && menu == 1 ){
 myDFPlayer.playMp3Folder(189);
delay (3000);
 myDFPlayer.playMp3Folder(131);
delay (2000);
 }

 if (description == "broken clouds") {
 Serial.println("Nuvens Dispersas");
 lcd.setCursor(0, 0);
 lcd.print("Nuvens Dispersas ");
 }
 if ( description == "broken clouds" && menu == 1 ){
 myDFPlayer.playMp3Folder(189);
 delay (3000);
 myDFPlayer.playMp3Folder(128);
 delay (2000);
 }

 if (description == "overcast clouds") {
 Serial.println("Nublado");
 lcd.setCursor(0, 0);
 lcd.print("Nublado ");
 }
 if ( description == "overcast clouds" && menu == 1 ){
 myDFPlayer.playMp3Folder(190);
 delay (3000);
 myDFPlayer.playMp3Folder(127);
 delay (2000);
 }

 if (description == "light intensity drizzle") {
 Serial.println("Garoa Fraca");
 lcd.setCursor(0, 0);
 lcd.print("Garoa Fraca ");
 }
 if ( description == "light intensity drizzle" && menu == 1 ){
 myDFPlayer.playMp3Folder(189);
 delay (3000);
 myDFPlayer.playMp3Folder(116);
 delay (2000);
 }

 if (description == "drizzle") {
 Serial.println("Chuvisco");
 lcd.setCursor(0, 0);
 lcd.print("Chuvisco ");
 }
 if ( description == "drizzle" && menu == 1 ){
 myDFPlayer.playMp3Folder(189);
delay (3000);
 myDFPlayer.playMp3Folder(110);
delay (2000);
 }

 if (description == "heavy intensity drizzle") {
 Serial.println("Garoa Forte");
 lcd.setCursor(0, 0);
 lcd.print("Garoa Forte ");
 }
 if ( description == "heavy intensity drizzle" && menu == 1 ){
 myDFPlayer.playMp3Folder(189);
delay (3000);
 myDFPlayer.playMp3Folder(115);
delay (2000);
 }

 if (description == "light intensity drizzle rain") {
 Serial.println("Chuva e Garoa");
 lcd.setCursor(0, 0);
 lcd.print("Chuva e Garoa ");
 }
 if ( description == "light intensity drizzle rain" && menu == 1 ){
 myDFPlayer.playMp3Folder(189);
 delay (3000);
 myDFPlayer.playMp3Folder(102);
 delay (2000);
 }

 if (description == "drizzle rain") {
 Serial.println("Chuva e Garoa");
 lcd.setCursor(0, 0);
 lcd.print("Chuva e Garoa ");
 }
 if ( description == "drizzle rain" && menu == 1 ){
 myDFPlayer.playMp3Folder(189);
 delay (3000);
 myDFPlayer.playMp3Folder(102);
 delay (2000);
 }

 if (description == "heavy intensity drizzle rain") {
 Serial.println("Chuva e Garoa");
 lcd.setCursor(0, 0);
 lcd.print("Chuva e Garoa ");
 }
 if ( description == "heavy intensity drizzle rain" && menu == 1 ){
 myDFPlayer.playMp3Folder(189);
 delay (3000);
 myDFPlayer.playMp3Folder(102);
 delay (2000);
 }

 if (description == "shower rain and drizzle") {
 Serial.println("Chuva e Garoa");
 lcd.setCursor(0, 0);
 lcd.print("Chuva e Garoa ");
 }
 if ( description == "shower rain and drizzle" && menu == 1 ){
 myDFPlayer.playMp3Folder(189);
 delay (3000);
 myDFPlayer.playMp3Folder(102);
 delay (2000);
 }

 if (description == "heavy shower rain and drizzle") {
 Serial.println("Chuva e Garoa");
 lcd.setCursor(0, 0);
 lcd.print("Chuva e Garoa ");
 }
 if ( description == "heavy shower rain and drizzle" && menu == 1 ){
 myDFPlayer.playMp3Folder(189);
 delay (3000);
 myDFPlayer.playMp3Folder(102);
 delay (2000);
 }

 if (description == "shower drizzle") {
 Serial.println("Chuva e Garoa");
 lcd.setCursor(0, 0);
 lcd.print("Chuva e Garoa ");
 }
 if ( description == "shower drizzle" && menu == 1 ){
 myDFPlayer.playMp3Folder(189);
 delay (3000);
 myDFPlayer.playMp3Folder(102);
 delay (2000);
 }

 if (description == "light rain") {
 Serial.println("Chuva Leve");
 lcd.setCursor(0, 0);
 lcd.print("Chuva Leve ");
 }
 if ( description == "light rain" && menu == 1 ){
 myDFPlayer.playMp3Folder(189);
 delay (3000);
 myDFPlayer.playMp3Folder(108);
 delay (2000);
 }

 if (description == "light intensity shower rain") {
 Serial.println("Chuva Leve");
 lcd.setCursor(0, 0);
 lcd.print("Chuva Leve ");
 }
 if ( description == "light intensity shower rain" && menu == 1 ){
 myDFPlayer.playMp3Folder(189);
 delay (3000);
 myDFPlayer.playMp3Folder(108);
 delay (2000);
 }

 if (description == "shower rain") {
 Serial.println("Chuva Leve");
 lcd.setCursor(0, 0);
 lcd.print("Chuva Leve ");
 }
 if ( description == "shower rain" && menu == 1 ){
 myDFPlayer.playMp3Folder(189);
 delay (3000);
 myDFPlayer.playMp3Folder(108);
 delay (2000);
 }

 if (description == "moderate rain") {
 Serial.println("Chuva Moderada");
 lcd.setCursor(0, 0);
 lcd.print("Chuva Moderada ");
 }
 if ( description == "moderate rain" && menu == 1 ){
 myDFPlayer.playMp3Folder(189);
 delay (3000);
 myDFPlayer.playMp3Folder(109);
 delay (2000);
 }

 if (description == "heavy intensity rain") {
 Serial.println("Chuva Forte");
 lcd.setCursor(0, 0);
 lcd.print("Chuva Forte ");
 }
 if ( description == "heavy intensity rain" && menu == 1 ){
 myDFPlayer.playMp3Folder(189);
 delay (3000);
 myDFPlayer.playMp3Folder(104);
 delay (2000);
 }

 if (description == "very heavy rain") {
 Serial.println("Chuva Intensa");
 lcd.setCursor(0, 0);
 lcd.print("Chuva Intensa ");
 }
 if ( description == "very heavy rain" && menu == 1 ){
 myDFPlayer.playMp3Folder(189);
 delay (3000);
 myDFPlayer.playMp3Folder(106);
 delay (2000);
 }

 if (description == "extreme rain") {
 Serial.println("Chuva Extrema");
 lcd.setCursor(0, 0);
 lcd.print("Chuva Extrema ");
 }
 if ( description == "extreme rain" && menu == 1 ){
 myDFPlayer.playMp3Folder(189);
 delay (3000);
 myDFPlayer.playMp3Folder(103);
 delay (2000);
 }

 if (description == "freezing rain") {
 Serial.println("Chuva Granizo");
 lcd.setCursor(0, 0);
 lcd.print("Chuva Granizo ");
 }
 if ( description == "freezing rain" && menu == 1 ){
 myDFPlayer.playMp3Folder(189);
 delay (3000);
 myDFPlayer.playMp3Folder(105);
 delay (2000);
 }

 if (description == "heavy intensity shower rain") {
 Serial.println("Chuva Forte");
 lcd.setCursor(0, 0);
 lcd.print("Chuva Forte ");
 }
 if ( description == "heavy intensity shower rain" && menu == 1 ){
 myDFPlayer.playMp3Folder(189);
 delay (3000);
 myDFPlayer.playMp3Folder(104);
 delay (2000);
 }

 if (description == "ragged shower rain") {
 Serial.println("Chuva Irregular");
 lcd.setCursor(0, 0);
 lcd.print("Chuva Irregular ");
 }
 if ( description == "ragged shower rain" && menu == 1 ){
 myDFPlayer.playMp3Folder(189);
 delay (3000);
 myDFPlayer.playMp3Folder(107);
 delay (2000);
 }

 if ( description == "light snow" ) {
 Serial.println("Pouca Neve");
 lcd.setCursor(0, 0);
 lcd.print("Pouca Neve ");
 }
 if ( description == "light snow" && menu == 1 ){
 myDFPlayer.playMp3Folder(189);
 delay (3000);
 myDFPlayer.playMp3Folder(103);
 delay (2000);
 }

 if ( description == "Snow" ) {
 Serial.println("Nevando");
 lcd.setCursor(0, 0);
 lcd.print("Nevando ");
 }
 if ( description == "Snow" && menu == 1 ){
 myDFPlayer.playMp3Folder(190);
 delay (3000);
 myDFPlayer.playMp3Folder(121);
 delay (2000);
 }

 if ( description == "Heavy snow" ) {
 Serial.println("Nevando Forte");
 lcd.setCursor(0, 0);
 lcd.print("Nevando Forte ");
 }
 if ( description == "Heavy snow" && menu == 1 ){
 myDFPlayer.playMp3Folder(190);
 delay (3000);
 myDFPlayer.playMp3Folder(120);
 delay (2000);
 }

 if ( description == "Sleet" ) {
 Serial.println("Granizo");
 lcd.setCursor(0, 0);
 lcd.print("Granizo ");
 }
 if ( description == "Sleet" && menu == 1 ){
 myDFPlayer.playMp3Folder(189);
 delay (3000);
 myDFPlayer.playMp3Folder(118);
 delay (2000);
 }

 if ( description == "Light shower sleet" ) {
 Serial.println("Granizo Leve");
 lcd.setCursor(0, 0);
 lcd.print("Granizo Leve ");
 }
 if ( description == "Light shower sleet" && menu == 1 ){
 myDFPlayer.playMp3Folder(189);
 delay (3000);
 myDFPlayer.playMp3Folder(117);
 delay (2000);
 }

 if (description == "Shower sleet") {
 Serial.println("Nevando");
 lcd.setCursor(0, 0);
 lcd.print("Nevando ");
 }
 if ( description == "Shower sleet" && menu == 1 ){
 myDFPlayer.playMp3Folder(190);
 delay (3000);
 myDFPlayer.playMp3Folder(121);
 delay (2000);
 }

 if (description == "Light rain and snow") {
 Serial.println("Nevasca Leve");
 lcd.setCursor(0, 0);
 lcd.print("Nevasca Leve ");
 }
 if ( description == "Light rain and snow" && menu == 1 ){
 myDFPlayer.playMp3Folder(190);
 delay (3000);
 myDFPlayer.playMp3Folder(124);
 delay (2000);
 }

 if (description == "Rain and snow") {
 Serial.println("Chuva e Neve");
 lcd.setCursor(0, 0);
 lcd.print("Chuva e Neve ");
 }
 if ( description == "Rain and snow" && menu == 1 ){
 myDFPlayer.playMp3Folder(189);
 delay (3000);
 myDFPlayer.playMp3Folder(193);
 delay (2000);
 }

 if (description == "Light shower snow") {
 Serial.println("Nevasca Fraca");
 lcd.setCursor(0, 0);
 lcd.print("Nevasca Fraca ");
 }
 if ( description == "Light shower snow" && menu == 1 ){
 myDFPlayer.playMp3Folder(189);
 delay (3000);
 myDFPlayer.playMp3Folder(123);
 delay (2000);
 }

 if (description == "Shower snow") {
 Serial.println("Nevasca");
 lcd.setCursor(0, 0);
 lcd.print("Nevasca ");
 }
 if ( description == "Shower snow" && menu == 1 ){
 myDFPlayer.playMp3Folder(189);
 delay (3000);
 myDFPlayer.playMp3Folder(125);
 delay (2000);
 }

 if (description == "Heavy shower snow") {
 Serial.println("Nevasca Forte");
 lcd.setCursor(0, 0);
 lcd.print("Nevasca Forte ");
 }
 if ( description == "Heavy shower snow" && menu == 1 ){
 myDFPlayer.playMp3Folder(189);
 delay (3000);
 myDFPlayer.playMp3Folder(122);
 delay (2000);
 }

 if (description == "Smoke") {
 Serial.println("Fumaça");
 lcd.setCursor(0, 0);
 lcd.print("Fumaça ");
 }
 if ( description == "Smoke" && menu == 1 ){
 myDFPlayer.playMp3Folder(189);
 delay (3000);
 myDFPlayer.playMp3Folder(194);
 delay (2000);
 }

 if (description == "fog") {
 Serial.println("Névoa");
 lcd.setCursor(0, 0);
 lcd.print("Névoa ");
 }
 if ( description == "fog" && menu == 1 ){
 myDFPlayer.playMp3Folder(189);
 delay (3000);
 myDFPlayer.playMp3Folder(197);
 delay (2000);
 }

 if (description == "Tornado") {
 Serial.println("Tornado");
 lcd.setCursor(0, 0);
 lcd.print("Tornado ");
 }
 if ( description == "Tornado" && menu == 1 ){
 myDFPlayer.playMp3Folder(189);
 delay (3000);
 myDFPlayer.playMp3Folder(198);
 delay (2000);
 }

 if (description == "mist") {
 Serial.println("Indeterminado");
 lcd.setCursor(0, 0);
 lcd.print("Indeterminado ");
 }
 if ( description == "mist" && menu == 1 ){
 myDFPlayer.playMp3Folder(190);
 delay (3000);
 myDFPlayer.playMp3Folder(196);
 delay (2000);
 }

 lcd.setCursor(0, 1);
 lcd.print("Temperatura:");
 float temperature = root["main"]["temp"];
 //Serial.println(temperature);
 float CTempF = (temperature - 273.15);
 float Test;

//Test = 25;
int CTemp;
CTemp=(int)CTempF;

Serial.print(CTemp); Serial.println("°C"); //°C.
//Serial.print(CTemp); Serial.println("°C"); //°C.
//Serial.println(CTemp);
//int ct;
//scanf("%2CTemp", &ct);
//Serial.println(ct);

if (CTemp >= 10) {
 lcd.setCursor(12, 1);
 lcd.print(CTemp);
 lcd.setCursor(14, 1);
 lcd.write(byte(1)); //°C
 lcd.setCursor(15, 1);
 lcd.print(" ");
 }
if (CTemp < 10 && CTemp > -10) {
 lcd.setCursor(12, 1);
 lcd.print(CTemp);
 lcd.setCursor(13, 1);
 lcd.write(byte(1)); //°C
 lcd.setCursor(14, 1);
 lcd.print(" ");
}
if (CTemp <= -10) { // Graus Negativos.
 lcd.setCursor(12, 1);
 lcd.print(CTemp);
 lcd.setCursor(15, 1); // Print do byte de °C.
 lcd.write(byte(1)); //°C
}

if (menu == 1){
 myDFPlayer.playMp3Folder(135);//A Temp é de
delay (2800);
//Serial.println(CTemp);
 myDFPlayer.playMp3Folder(CTemp); //Valor da temperatura
delay (1800);
 myDFPlayer.playMp3Folder(136);//Graus Celsius
delay (2000);
}


// lcd.setCursor(0, 0);
 //lcd.print("Umid:");
 float humidity = root["main"]["humidity"];

/*Variavel Forçada
float TesteUmi = 65;
 Serial.print(TesteUmi,0); */

 Serial.print(humidity,0); Serial.println("%");
 lcd.setCursor(16, 1);
 lcd.print(humidity,0);
 lcd.setCursor(18, 1);
 lcd.print("%");
 lcd.setCursor(19, 1);
 lcd.write(byte(0)); // Gotinha.

if (humidity == 100){
 lcd.setCursor(16, 1);
 lcd.write(byte(2));
 lcd.setCursor(17, 1);
 lcd.write(byte(3));
}

if (menu == 1){
myDFPlayer.playMp3Folder(137);
delay (3000);
myDFPlayer.playMp3Folder(humidity); //
delay (1500);
 myDFPlayer.playMp3Folder(138); //Por cento
delay (1500);
}

//int VeloK1;
//VeloK1 = 9; // Teste para teste variável.

 float Velocidade = root["wind"]["speed"];
 //Serial.println(Velocidade);
 float VeloKF = (Velocidade * 3.6);
 int VeloK;
 VeloK=(int)VeloKF;

 Serial.print(VeloK,0); Serial.println("Km/h");
 lcd.setCursor(0, 2);
 lcd.print("Vento:");
 lcd.setCursor(6, 2);
 lcd.print(VeloK,0);

 if (VeloK < 10){
 lcd.setCursor(7, 2);
 lcd.print("Km/h ");
 }
 if (VeloK >= 10){
 lcd.setCursor(8, 2);
 lcd.print("Km/h");
 }

 if (menu == 1){
myDFPlayer.playMp3Folder(139); //O vento se encontra em uma velocidade
de.
delay (3400);
myDFPlayer.playMp3Folder(VeloK); //
delay (1800);
 myDFPlayer.playMp3Folder(140); //Quilometros por Hora
delay (2300);
}

 String country = root["sys"]["country"];
 String location = root["name"];
 Serial.println(location);
 Serial.println(country);
if (location == "Ribeirão Preto") { //Correção da acentuação de "ã" em print.
 lcd.setCursor(0, 3);
 lcd.print("Ribeirao Preto");

}
 lcd.setCursor(18, 3);
 //lcd.println(country);
 if (country == "BR") { //Correção da acentuação de "BR" em print.
 lcd.setCursor(18, 3);
 lcd.print("BR");
}
 //float pressure = root["main"]["pressure"];
 //Serial.println(pressure);
}

void loop() {
 // O OWM requer 10 minutos entre os intervalos de solicitação.
 // Verifique se 10 minutos se passaram, e conecte-se novamente ao
OpenWeatherMap.
 if (millis() - lastConnectionTime > postInterval) {
 // Hora em que a conexão foi feita:
 lastConnectionTime = millis();
 makehttpRequest();
 }

 if (timeStatus() != timeNotSet) {
 if (now() != prevDisplay) { // Atualize a exibição apenas se o tempo mudou.
 prevDisplay = now();
 digitalClockDisplay();
 menu = 0;
 }
 }

 if (digitalRead(D7) == 1){
 }
 else if (digitalRead(D7) == 0){
 Serial.println("Botão pressionado");
 menu = 1;
 }

 if ( menu == 1){
 makehttpRequest();
 }
}