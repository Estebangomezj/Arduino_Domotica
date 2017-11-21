# Arduino_Domotica
# Universidad Sergio Arboleda Domotica
#include <SPI.h> 
#include <Ethernet.h>
#include <Servo.h>

    byte mac[]={0xDE,0xAD,0xBE,0xEF,0xFE,0xED}; 
    IPAddress ip(190, 157, 8, 11); //IP 
    EthernetServer servidor(80); 
        
    int PIN_LED_VERDE= 9;
    const int TRIGGER = 2;
    const int ECHO = 3;
    const int BUZZER = 4;
    int PIN_LED_SALA=5;
    int PIN_LED_ROJO=6;
    int PIN_LED_ALCOBA=7;
  
         
    const float sonido = 34300.0;
    Servo servoMotor;

void setup() {
    
    Ethernet.begin(mac, ip); 
    servidor.begin(); 
    Serial.begin(9600);
    pinMode(PIN_LED_VERDE,OUTPUT);
    pinMode(PIN_LED_SALA,OUTPUT);
    pinMode(PIN_LED_ALCOBA,OUTPUT);
    pinMode(PIN_LED_ROJO,OUTPUT);
    pinMode(ECHO, INPUT);
    pinMode(TRIGGER, OUTPUT);
    pinMode(BUZZER, OUTPUT);

     apagarLEDs();     
    
    servoMotor.attach(8); // PIN 8 SERVOMOTOR 
    }
    
void loop() {
    EthernetClient cliente= servidor.available(); //EthernetClient Crea un cliente que se puede conectar a 
      if(cliente) {                                                       //una dirección específica de Internet IP
        while(cliente.connected()) {
          if(cliente.available()) {
              char componente=cliente.read();
              char c=cliente.read();  

              float distancia;
             
              switch(componente){

                      // LUZ SALA
                    case 'R':
                        if(c=='1'){
                          digitalWrite(PIN_LED_SALA,HIGH);
                        }
                        else if(c=='0'){
                          digitalWrite(PIN_LED_SALA,LOW);
                        }
                        break;
                      // LUZ HABITACION
                     case 'B':
                        if(c=='1'){
                          digitalWrite(PIN_LED_ALCOBA,HIGH);
                        }
                        else if(c=='0'){
                          digitalWrite(PIN_LED_ALCOBA,LOW);
                        }
                        break;
                       // LUZ TODA LA CASA
                      case 'T':
                        if(c=='1'){
                          digitalWrite(PIN_LED_SALA,HIGH);
                          digitalWrite(PIN_LED_ALCOBA,HIGH);
                        }
                        else if(c=='0'){
                          digitalWrite(PIN_LED_SALA,LOW);
                          digitalWrite(PIN_LED_ALCOBA,LOW);
                        }
                        break;
                            // PUERTA ON/OFF
                      case 'S':
                          if(c=='1'){
                           for (int i = 0; i <= 90; i++)
                                 {
                                  servoMotor.write(i);
                                  delay(40);                                           
                                  }
                           }
                      else if(c=='0')
                        {
                           for (int i = 89; i >= 0; i--)
                                 {
                                  servoMotor.write(i);
                                  delay(40);
                                }
                        }
                       break;

                        // sensor

                        case 'P':

                             if(c=='1'){

                              do{
                             
                               iniciarTrigger(); 
                               distancia = calcularDistancia();                                       
                                  if (distancia > 20)
                                          {
                                           // Encendemos el LED verde
                                              digitalWrite(PIN_LED_VERDE, HIGH);
                                              tone(BUZZER, 1000, 200);
                                          }
                                  else if (distancia <= 20 & distancia >10 ) {
        
                                            digitalWrite(PIN_LED_VERDE, HIGH);
                                            tone(BUZZER, 4000, 400);
                                            }
                                  else if (distancia <= 10 & distancia >5 ) {
        
                                            digitalWrite(PIN_LED_ROJO, HIGH);
                                            tone(BUZZER, 6000, 400);
                                            }
                             }
                             while (distancia >=5);
                             }
                             else if(c=='0') {
                                    digitalWrite(PIN_LED_ROJO, HIGH);
                              }
                            
                       break;
                       default:
                          cliente.write("No hay componente: ");
                          cliente.write(componente);
                        break;  
                      cliente.write('\n');
                      delay(5);                    
                      }

                     
        }
    }
   }
      }


 void apagarLEDs()
{
  digitalWrite(PIN_LED_VERDE, HIGH);
  digitalWrite(PIN_LED_ROJO, HIGH);
  digitalWrite(PIN_LED_SALA, LOW);
  digitalWrite(PIN_LED_ALCOBA, LOW);
}


 
// Método que calcula la distancia a la que se encuentra un objeto.
float calcularDistancia()
{
  unsigned long tiempo = pulseIn(ECHO, HIGH);
  float distancia = tiempo * 0.000001 * sonido / 2.0;
  Serial.print(distancia);
  Serial.print("cm");
  Serial.println();
  delay(500);
 
  return distancia;
}
 
// Método que inicia la secuencia del Trigger para comenzar a medir
void iniciarTrigger()
{
  // Ponemos el Triiger en estado bajo y esperamos 2 ms
  digitalWrite(TRIGGER, LOW);
  delayMicroseconds(200);
 
  // Ponemos el pin Trigger a estado alto y esperamos 10 ms
  digitalWrite(TRIGGER, HIGH);
  delayMicroseconds(100);

  digitalWrite(TRIGGER, LOW);
}

