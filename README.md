# Desk-Cleaning-Robot-
Desk Cleaning Robot Project


// Self-cleaning desk robot: Arduino + TB6612FNG + 4x TCRT5000

// Sensors
const int S_FRONT = 2;
const int S_LEFT  = 3;
const int S_RIGHT = 4;
const int S_BACK  = 11;

// TB6612FNG
const int PWMA = 5;   // Left speed
const int AIN1 = 6;
const int AIN2 = 7;
const int BIN1 = 8;
const int BIN2 = 9;
const int PWMB = 10;  // Right speed

// ---- Settings ----
const int  EDGE_LEVEL  = HIGH;  // D0 level when NO surface is seen. Flip to LOW if reversed.
const bool LEFT_FLIP   = false;
const bool RIGHT_FLIP  = false;
const int  DRIVE_SPEED = 170;
const int  TURN_SPEED  = 170;
const int  BACK_TIME   = 350;   // ms reversing away from front edge
const int  FWD_TIME    = 350;   // ms driving forward away from back edge
// ------------------

bool edge(int pin) { return digitalRead(pin) == EDGE_LEVEL; }

void setLeft(int speed) {
  if (LEFT_FLIP) speed = -speed;
  digitalWrite(AIN1, speed > 0);
  digitalWrite(AIN2, speed < 0);
  analogWrite(PWMA, abs(speed));
}

void setRight(int speed) {
  if (RIGHT_FLIP) speed = -speed;
  digitalWrite(BIN1, speed > 0);
  digitalWrite(BIN2, speed < 0);
  analogWrite(PWMB, abs(speed));
}

void drive(int l, int r) { setLeft(l); setRight(r); }
void stopMotors()        { drive(0, 0); }

// Move straight for ms. dirSign: +1 forward, -1 backward.
// Stops early if the sensor on the side we are moving toward sees air.
void moveStraight(int dirSign, int ms) {
  int guard = (dirSign > 0) ? S_FRONT : S_BACK;
  drive(dirSign * DRIVE_SPEED, dirSign * DRIVE_SPEED);
  unsigned long t = millis();
  while (millis() - t < (unsigned long)ms) {
    if (edge(guard)) break;
  }
  stopMotors();
}

// dir: +1 = spin right, -1 = spin left
void spin(int dir, int ms) {
  drive(dir * TURN_SPEED, -dir * TURN_SPEED);
  delay(ms);
  stopMotors();
}

void setup() {
  pinMode(S_FRONT, INPUT);
  pinMode(S_LEFT,  INPUT);
  pinMode(S_RIGHT, INPUT);
  pinMode(S_BACK,  INPUT);

  pinMode(PWMA, OUTPUT); pinMode(AIN1, OUTPUT); pinMode(AIN2, OUTPUT);
  pinMode(PWMB, OUTPUT); pinMode(BIN1, OUTPUT); pinMode(BIN2, OUTPUT);

  randomSeed(analogRead(A0));
  stopMotors();
  delay(1000);
}

void loop() {
  bool f = edge(S_FRONT);
  bool b = edge(S_BACK);
  bool l = edge(S_LEFT);
  bool r = edge(S_RIGHT);

  if (f && b) {
    // Air at both ends (lifted or on a narrow strip): stay put
    stopMotors();
  }
  else if (b) {
    // Back over air: go forward, then turn a bit
    moveStraight(+1, FWD_TIME);
    spin(random(0, 2) ? 1 : -1, random(250, 600));
  }
  else if (f || (l && r)) {
    // Front over air: go backward, then turn randomly
    moveStraight(-1, BACK_TIME);
    spin(random(0, 2) ? 1 : -1, random(350, 900));
  }
  else if (l) {
    // Left over air: back off a little, turn right
    moveStraight(-1, 150);
    spin(+1, random(250, 600));
  }
  else if (r) {
    // Right over air: back off a little, turn left
    moveStraight(-1, 150);
    spin(-1, random(250, 600));
  }
  else {
    drive(DRIVE_SPEED, DRIVE_SPEED);
  }
}