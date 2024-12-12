# Raman
//

#include <Arduino.h>
#include <hp_BH1750.h>  //  include the library
hp_BH1750 BH1750;       //  create the sensor

#include <Servo.h> // Inclui a biblioteca Servo para controlar servos
 
Servo meuServo; // Cria um objeto Servo para controlar o servo motor
int pos, pot; // posicao real-time

unsigned long t0, t1;
int pinLaser = 5;
int pinMotor = 6;

float lerFotodiodo(){
  BH1750.start();   //starts a measurement
  float lux=BH1750.getLux();  //  waits until a conversion finished
  return(lux);
}

void testarLaser(){
  int _pos;
  for (_pos = 255; _pos > 0; _pos-=5) {
       analogWrite(pinLaser,_pos);
       delay(15);
  }
  analogWrite(pinLaser,0);
  Serial.println("Teste concluído");
  
}

void testarMotor(){
  int _pos;
  for (_pos = 0; _pos < 360; _pos++) {
       meuServo.write(_pos);
       delay(15);
  }
  Serial.println("Teste concluído");
}

void alinharRedeHorario(int pos_){
  int _pos;
  for (_pos = pos; _pos < (pos + pos_); _pos++) {
       meuServo.write(_pos);
       delay(15);
  }
  pos = _pos;
 // Serial.println("Alinhamento concluído");
}

void alinharRedeAntiHorario(int pos_){
  int _pos;
  for (_pos = pos ; _pos > (pos - pos_); _pos--) {
       meuServo.write(_pos);
       delay(15);
   }
   pos = _pos;
   //Serial.println("Alinhamento concluído");
}

void aumentarLaser(int pot_){
  pot = pot + pot_;
  if( pot > 255 ) pot = 255;
  analogWrite(pinLaser,pot);
}

void diminuirLaser(int pot_){
  pot = pot - pot_;
  if( pot < 0 ) pot = 0;
  analogWrite(pinLaser, pot);
}

void(* resetFunc) (void) = 0;//declare reset function at address 0

void setup()
{
  // put your setup code here, to run once:
  Serial.begin(115200);
  bool avail = BH1750.begin(BH1750_TO_GROUND);// init the sensor with address pin connetcted to ground
  if (!avail) {
    Serial.println("No BH1750 sensor found!");
    while (true) {};                                        
  }
  else{
       pinMode(pinLaser,OUTPUT);
       pinMode(pinMotor,OUTPUT);
       pot = 0;
       analogWrite(pinLaser,0);
       meuServo.attach(pinMotor); // Associa o servo motor ao pino digital 6 do Arduino
       meuServo.write(0);
      // testarMotor();
      // testarLaser();
       //Serial.println("Sistema operacional");
  }
  
}

void loop()
{
  if( Serial.available() > 0 ){
    char controle = Serial.read();
    switch(controle){
       case 'L':
       {
             float medidas = 0;
             int _pos = 0;
             t0 = millis();
             while(_pos++ < 10) medidas = medidas + lerFotodiodo(); 
             t1 = millis();
             medidas = 1000 * medidas / (t1 - t0); // soma das medidas / periodo de tempo em segundos 
             Serial.println(medidas);
             break;
       }
       case 'C':
       {
             delay(100);
             alinharRedeHorario(10);
             Serial.print("Posição: ");
             Serial.println(pos);
             break;
       }
       case 'A':
       {
             delay(100);
             alinharRedeAntiHorario(10);
             Serial.print("Posição: ");
             Serial.println(pos);
             break;
       }
       case 'P':
       {
             delay(100);
             aumentarLaser(10);
             Serial.print("Potência: ");
             Serial.println(pot);
             break;
       }
       case 'p':
       {
             delay(100);
             diminuirLaser(10);
             Serial.print("Potência: ");
             Serial.println(pot);
             break;
       }
       case 'S':
       {
            // Serial.println("Reiniciando...");
             delay(100);
             resetFunc();
             break;
       }
    }
 }
 delay(1000);
}
