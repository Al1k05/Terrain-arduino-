//#include <MUIU8g2.h>
#include <U8g2lib.h>
#include <U8x8lib.h>

#include <Arduino.h>
#include <SPI.h>
#include <U8g2lib.h>
#include <Servo.h>
U8G2_PCD8544_84X48_F_4W_SW_SPI u8g2(U8G2_R0, /* clock=*/ 8, /* data=*/ 9, /* cs=*/ 11, /* dc=*/ 10, /* reset=*/ 12);  //Nokia 5110 Display
Servo Servol;
const int servoPin = 6;
//for sonar
const int triggerPin = 4;
const int echoPin = 3;
int distance;
int locationOfObjects[180];
void drawDail(int angle) {
  u8g2.drawCircle(42, 48, 41, U8G2_DRAW_ALL);//center(42,48) radius:41
  u8g2.drawCircle(42, 48, 31, U8G2_DRAW_ALL);//center(42,48) radius:31
  u8g2.drawCircle(42, 48, 21, U8G2_DRAW_ALL);//center(42,48) radius:21
  u8g2.drawCircle(42, 48, 11, U8G2_DRAW_ALL);//center(42,48) radius:11
  int x = 42 - 41 *  cos(angle * 3.14 / 180);
  int y = 48 - 41 *  sin(angle * 3.14 / 180);
  u8g2.drawLine(42, 48, x, y);
}
void drawObjectLine(int value, int angle) {
  int x0 = 42 - 41 * cos(angle * 3.14 / 180);
  int y0 = 48 - 41 * sin(angle * 3.14 / 180);
  int x1 = 42 - value * cos(angle * 3.14 / 180);
  int y1 = 48 - value * sin(angle * 3.14 / 180);
  u8g2.drawLine(x1, y1, x0, y0);
}
int getDistance()  {
  long duration;
  digitalWrite(triggerPin, LOW);
  delayMicroseconds(2);
  digitalWrite(triggerPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(triggerPin, LOW);
  duration = pulseIn(echoPin, HIGH);
  // Calculating the distance
  return duration * 0.034 / 2;
}
void clearArray()  {
  for (int i = 0; i < 180; i++) {
    locationOfObjects[i] = 0;
  }
}
void setup(void)  {
  u8g2.begin();
  Servol.attach(servoPin);
  Serial.begin(9600);
}
 
void loop(void)  {
  for (int i = 0; i < 180; i++) {
    Servol.write(i);
    u8g2.clearBuffer();
    drawDail(i);
    distance = getDistance();
    if (distance < 70)  {
      locationOfObjects[i] = distance;
    } else  {
      locationOfObjects[i] = 0;
    }
    for (int k = 0; k < i; k++)  {
      if (locationOfObjects[k])  {
        drawObjectLine(locationOfObjects[k],  k);
      }
    }
    u8g2.sendBuffer();
    delay(50);
   }
   clearArray();
   for (int i = 180; i > 0; i--)  {
    Servol.write(i);
    u8g2.clearBuffer();
    drawDail(i);
    distance = getDistance();
    if (distance < 30)  {
      locationOfObjects[i] = distance;
    } else  {
      locationOfObjects[i] = 0;
    }
    for (int k = 180; k > i; k--)  {
      if (locationOfObjects[k]) {
        drawObjectLine(locationOfObjects[k],  k);
      }
    }
    u8g2.sendBuffer();
    delay(50);
  }
   clearArray();  
}  
