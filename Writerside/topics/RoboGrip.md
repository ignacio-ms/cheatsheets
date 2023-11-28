# RoboGrip

Robotic grip controlled by hand-gestual movements

<video src="https://www.youtube.com/watch?v=sW47YUKkPIM"/>

## Electric schemes {#electric-schemes}

Grip

![Grip](EsquemaElectricoBrazo.png)

Glove

![Glove](EsquemaElectricoGuante.png)

## 3D models {#3d-models}

[3D Grip models](https://onedrive.live.com/?id=3C936019C1B9C8A7%2128200&cid=3C936019C1B9C8A7)

## Source code {#source-code}

Sender (glove)

```C++
#include "I2Cdev.h"
#include "MPU6050.h"
#include "Wire.h"

#define FLEX_PIN A0

MPU6050 mpu;

int ax, ay, az;
float acc[2];
float flex_deg;

void setup()
{
  Serial.begin(9600);
  Wire.begin();

  pinMode(FLEX_PIN, INPUT);
  
  mpu.initialize();
}

void loop() 
{
  // Leer las aceleraciones y giroscopio
  mpu.getAcceleration(&ax, &ay, &az);

  //Calculos angulos X e Y
  acc[0] = atan(ax / sqrt(pow(ay, 2) + pow(az, 2))) * RAD_TO_DEG;
  acc[1] = atan(ay / sqrt(pow(ax, 2) + pow(az, 2))) * RAD_TO_DEG;

  //Calcula angulos del sensor de flexion
  flex_deg = analogRead(FLEX_PIN);
  flex_deg = map(flex_deg, 264, 512, 0, 180);

  send_data();
  delay(10);
}

void send_data(){
  Serial.print(acc[0]);
  Serial.print(",");
  Serial.print(acc[1]);
  Serial.print(",");
  Serial.println(flex_deg);
}
```

[Helpers](https://onedrive.live.com/?id=3C936019C1B9C8A7%2128231&cid=3C936019C1B9C8A7)

Receiver (grip)

```C++
#include <Servo.h>

#define servo_base_pin 9
#define servo_arm_1_pin 6
//#define servo_arm_2_pin 5
#define servo_grip_pin 3

Servo servo_base;
Servo servo_arm_1;
//Servo servo_arm_2;
Servo servo_grip;

float threshold = 20;
float threshold_flex = 120;

float prev_base_ang = 0;
float prev_arm_ang_0 = 0;
//float prev_arm_ang_1 = 90;

int acc[2];
int flex_deg;

String str;


void setup() {
  Serial.begin(9600);
  
  servo_base.attach(servo_base_pin);
  servo_arm_1.attach(servo_arm_1_pin);
//  servo_arm_2.attach(servo_arm_2_pin);
  servo_grip.attach(servo_grip_pin);
  
  servo_base.write(0);
  servo_arm_1.write(0);
//  servo_arm_2.write(90);
  servo_grip.write(50);
}

void loop() {
  if(Serial.available() > 0){
    str = Serial.readStringUntil('\n');
  }
  
  parse_data();
  str = "";

  move_arm();
  delay(10);
}

void parse_data()
{  
  int ind1, ind2, ind3;
  float acc_0_aux, acc_1_aux, flex_deg_aux;

  ind1 = str.indexOf(',');
  acc[0] = str.substring(0, ind1).toFloat();

  ind2 = str.indexOf(',', ind1+1);
  acc[1] = str.substring(ind1+1, ind2+1).toFloat();

  ind3 = str.indexOf(',', ind2+1);
  flex_deg = str.substring(ind2+1).toFloat(); 
}

void move_arm(){
  /* ----- Base -----*/ 
  if(acc[1] > threshold && prev_base_ang > 0){
    servo_base.write(prev_base_ang - 1);
    prev_base_ang = prev_base_ang - 1;
  } 
  else if(acc[1] < -threshold && prev_base_ang < 180){
    servo_base.write(prev_base_ang + 1);
    prev_base_ang = prev_base_ang + 1;
  }

  /* ----- ARM -----*/
  //dcha
  if(acc[0] > threshold && prev_arm_ang_0 < 50){
    servo_arm_1.write(prev_arm_ang_0 + 1);
    prev_arm_ang_0 = prev_arm_ang_0 + 1;
  } 
  else if(acc[0] < -threshold && prev_arm_ang_0 > 0){
    servo_arm_1.write(prev_arm_ang_0 - 1);
    prev_arm_ang_0 = prev_arm_ang_0 - 1;
  }

  /*
   * Este servo no tiene fuerza para mover su parte del brazo
   */
  //izda
//  if(acc[0] > threshold && prev_arm_ang_1 > 0){
//    servo_arm_2.write(prev_arm_ang_1 - 1);
//    prev_arm_ang_1 = prev_arm_ang_1 - 1;
//  } 
//  else if(acc[0] < -threshold && prev_arm_ang_1 < 130){
//    servo_arm_2.write(prev_arm_ang_1 + 1);
//    prev_arm_ang_1 = prev_arm_ang_1 + 1;
//  }

  /* ----- GRIP -----*/
  if(flex_deg > threshold_flex)
    servo_grip.write(0);
  else if(flex_deg < threshold_flex)
    servo_grip.write(50);
  
}
```

Drawing motions Processing

```Java
import processing.serial.*;

Serial port;
String port_name;
String str;

float roll, pitch, flex_deg;
float threshold_flex = 120;
PImage logo;

void setup(){
  size(800, 800, P3D);
  
  port_name = Serial.list()[32]; //Es el indice del puerto que corresponde en mi PC
  port = new Serial(this, port_name, 9600);
  
  logo = loadImage("logo.png");
}

void draw(){
  if(port.available() > 0){
    str = port.readStringUntil('\n');
  }
  if(str != null){
    String[] splitted = split(str, ',');
    
    if(splitted.length == 3 && splitted[0] != null && splitted[1] != null && splitted[2] != null){
  
      try {
        pitch = Float.parseFloat(splitted[0]);
        roll = Float.parseFloat(splitted[1]);
        flex_deg = Float.parseFloat(splitted[2]); 
        
        translate(width/2, height/2, 0);
        
        background(105,105,105);
        image(logo, 200, -400, 200, 220);
        
        draw_box("Hand\n", pitch, roll, flex_deg);
      }catch (NumberFormatException e){}
    }
  }
}

void draw_box(String name, float pitch, float roll, float flex_deg) {
    pushMatrix();

    rotateX(radians(pitch));
    rotateZ(radians(roll));
  
    textSize(16);
    
    boolean flexing;
    if (flex_deg > threshold_flex){
      fill(255, 0, 0);
      flexing = true;
    }
    else{
      fill(0, 0, 255);
      flexing = false;
    }
      
    box (160, 40, 300); 
    fill(255, 255, 255);
    text(name, -60, 5, 151);
    popMatrix();
    text( name 
          + "\npitch: " + int(pitch)
          + "\nroll: " + int(roll)
          + "\ngrip flexing: " + (flexing ? "Yes" : "No")
          , 0 -20, 200);
}
```
