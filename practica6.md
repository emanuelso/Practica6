César Roldan Moros
*Grup 13*
___
# PRÀCTICA 6: Buses de comunicaciones II
##### Objectivo
El objetivo de esta práctica es entender el funcionamiento de los buses de sistemas de comunicación entre periféricos internos o externos.
### Ejercicio 1. LECTURA/ESCRITURA DE MEMORIA SD
##### Fotos del montaje
No pude hacer el montaje ya que no pude obtener el lector de tarjetas SD.
##### Codigo
```
#include <Arduino.h>
#include <SPI.h>
#include <SD.h>
File myFile; 
void setup()
{
  Serial.begin(112500);
  Serial.print("Iniciando SD ..."); 
  if (!SD.begin(5)) { 
	  Serial.println("No se pudo inicializar");
	  return;
  }
  Serial.println("inicializacion exitosa");
  myFile = SD.open("prova_P6.txt");
  if (myFile) { 
	  Serial.println("prova_P6.txt:");
	  while (myFile.available()) { 
		  Serial.write(myFile.read()); 
	  }
  	myFile.close(); 
	} 
  else {
	  Serial.println("Error al abrir el archivo"; 
  }
}
void loop(){}
```
##### Funcionamiento
Primero creamos una variable de tipo file para leer el archivo.
En el void setup() inicializamos la velocidad de transmisión de bits e imprimimos por el serial un mensaje que nos indica que empezará a iniciar el SD.
Le decimos que si devuelve algo diferente a 0 que salga un mensaje diciendo que no se ha podido inicializar al SD, si es por el contrario un mensaje diciendo que sí que ha podido inicializar con éxito el SD.
Abrimos el archivo. Si lo puede abrir, hace un bucle mientras que el archivo esté disonible vaya leyendo carácter por carácter y envía estos caracteres leídos por pantalla
Cerramos el archivo
Si no ha podido abrir el archivo imprimimos por pantalla que no ha sido posible
El void loop() está vacío.

### Ejercicio 2 LECTURA DE ETIQUETA RFID
##### Fotos del montaje
Las fotos y videos del montaje estan en el mismo repositorio de esta practica en el github.
##### Codigo
```
#include <Arduino.h>

//Libraries
#include <SPI.h>
#include <MFRC522.h>
//Constants
#define SS_PIN 5
#define RST_PIN 0

void readRFID();
void printDec(byte *buffer, byte bufferSize);

const int ipaddress[4] = {103, 97, 67, 25};

byte nuidPICC[4] = {0, 0, 0, 0};
MFRC522::MIFARE_Key key;
MFRC522 rfid = MFRC522(SS_PIN, RST_PIN);
void setup() {
 
 Serial.begin(115200);
 Serial.println(F("Initialize System"));
 
 SPI.begin();
 rfid.PCD_Init();
 Serial.print(F("Reader :"));
 rfid.PCD_DumpVersionToSerial();
}
void loop() {
 readRFID();
}
void readRFID(void ) { 
 
 for (byte i = 0; i < 6; i++) {
   key.keyByte[i] = 0xFF;
 }
 
 if ( ! rfid.PICC_IsNewCardPresent())
   return;
 
 if (  !rfid.PICC_ReadCardSerial())
   return;
 
 for (byte i = 0; i < 4; i++) {
   nuidPICC[i] = rfid.uid.uidByte[i];
 }
 Serial.print(F("RFID In dec: "));
 printDec(rfid.uid.uidByte, rfid.uid.size);
 Serial.println();
 
 rfid.PICC_HaltA();
 
 rfid.PCD_StopCrypto1();
}

void printHex(byte *buffer, byte bufferSize) {
 for (byte i = 0; i < bufferSize; i++) {
   Serial.print(buffer[i] < 0x10 ? " 0" : " ");
   Serial.print(buffer[i], HEX);
 }
}

void printDec(byte *buffer, byte bufferSize) {
 for (byte i = 0; i < bufferSize; i++) {
   Serial.print(buffer[i] < 0x10 ? " 0" : " ");
   Serial.print(buffer[i], DEC);
 }
}
```

##### Funcionamiento
El módulo RFID es un lector cuyo funcionamiento consiste en pasar información al lector a través de un código o un paquete de información guardado en la memoria del TAG.
Nosotros trabajaremos con el módulo RC522.
Primeramente y antes del void setup, definimos los pines con los que trabajaremos, los paraemtros (en este caso constantes) y las variables de tipo byte y MERC22, la cual nos permitirá dialogar con el módulo.
En el void setup, lo primero que haremos será inicializar el USB serie, con Serial.begin(115200) y Serial.println(F("Initialize System")). 
Seguidamente, haremos lo mismo pero con rfid D8,D5,D6,D7.
En el void loop simplemente leeremos el rfid con readRFID().
Crearemos la función para leer el rfid readRFID():
Primero haremos un for para leer la tarjeta RFID.
En el primer if: Se buscaran tarjetas nuevas.
En el segundo if: Verificar si se ha leído el NUID.
Seguido de eso, haremos otro for para almacenar NUID en el array nuidPICC

