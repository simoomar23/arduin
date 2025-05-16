//#include <MAX6675.h>
#include <max6675.h>

int thermoSO = 8;
int thermoSCK = 10;
int csHot = 9;
int csCold = 7;
int csMosfet = 4;
int pwmPin = 6;

MAX6675 sensorHot(thermoSCK, csHot, thermoSO);
MAX6675 sensorCold(thermoSCK, csCold, thermoSO);
MAX6675 sensorMosfet(thermoSCK, csMosfet, thermoSO);

#define MAXPWM 255
#define MAX_DELTA 60

int duty = 5;

void setup() {
  Serial.begin(9600);
  pinMode(pwmPin, OUTPUT);
  delay(10000);
}

void loop() {
  float tHot = sensorHot.readCelsius();
  float tCold = sensorCold.readCelsius();
  float tMosfet = sensorMosfet.readCelsius();
  float delta = abs(tHot - tCold);

  Serial.print("Hot: "); Serial.println(tHot);
  Serial.print("Cold: "); Serial.println(tCold);
  Serial.print("Delta: "); Serial.println(delta);
  
  if (delta >= MAX_DELTA) {
    analogWrite(pwmPin, 0);
    Serial.println("Delta trop élevé, PWM coupé");
    while (abs(sensorHot.readCelsius() - sensorCold.readCelsius()) >= MAX_DELTA / 3) {
      delay(1000);
      Serial.println("En attente que delta < seuil");
    }
    duty = 20;
    return;
  }

  duty += 5;
  duty = constrain(duty, 20, 100);

  int pwm = map(duty, 0, 100, 0, MAXPWM);
  analogWrite(pwmPin, pwm);

  Serial.print("Duty: "); Serial.println(duty);
  Serial.print("PWM: "); Serial.println(pwm);

  delay(1000);
}
