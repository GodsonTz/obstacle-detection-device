#define TRIG 5
#define ECHO 18
#define MOTOR 2  // Vibration motor pin (direct connection with resistor)

long getDistance() {
  long sum = 0;
  int validReadings = 0;

  for (int i = 0; i < 5; i++) {  // 5 readings for stability
    digitalWrite(TRIG, LOW);
    delayMicroseconds(2);
    digitalWrite(TRIG, HIGH);
    delayMicroseconds(10);
    digitalWrite(TRIG, LOW);

    long duration = pulseIn(ECHO, HIGH, 30000);
    if (duration > 0) {
      sum += duration;
      validReadings++;
    }
    delay(5);  // minimal delay for stability
  }

  if (validReadings == 0) return -1;
  long averageDuration = sum / validReadings;
  return averageDuration * 0.0343 / 2;
}

// State-machine for vibration
struct VibePattern {
  int pulses;
  int onTime;
  int offTime;
  int currentPulse;
  bool isOn;
  unsigned long lastChange;
  bool active;
};

VibePattern vibe;

void startVibe(int count, int onTime, int offTime) {
  vibe.pulses = count;
  vibe.onTime = onTime;
  vibe.offTime = offTime;
  vibe.currentPulse = 0;
  vibe.isOn = false;
  vibe.lastChange = millis();
  vibe.active = true;
}

void updateVibe() {
  if (!vibe.active) return;
  unsigned long now = millis();
  if (vibe.isOn && (now - vibe.lastChange >= vibe.onTime)) {
    digitalWrite(MOTOR, LOW);
    vibe.isOn = false;
    vibe.lastChange = now;
  }
  else if (!vibe.isOn && (now - vibe.lastChange >= vibe.offTime)) {
    vibe.currentPulse++;
    if (vibe.currentPulse > vibe.pulses) {
      vibe.active = false;
      digitalWrite(MOTOR, LOW);
      return;
    }
    digitalWrite(MOTOR, HIGH);
    vibe.isOn = true;
    vibe.lastChange = now;
  }
}

void setup() {
  pinMode(TRIG, OUTPUT);
  pinMode(ECHO, INPUT);
  pinMode(MOTOR, OUTPUT);
  digitalWrite(MOTOR, LOW);
  Serial.begin(115200);
  Serial.println("Obstacle Detection Started (Non-blocking Vibration)...");
}

unsigned long lastCheck = 0;
void loop() {
  updateVibe();  // always update motor pattern

  unsigned long now = millis();
  if (now - lastCheck >= 50) {  // check distance every 50 ms
    lastCheck = now;
    long distance = getDistance();
    if (distance > 0) {
      Serial.print("Distance: ");
      Serial.print(distance);
      Serial.println(" cm");

      if (distance >= 45 && distance <= 60) {
        startVibe(3, 250, 100);
      }
      else if (distance >= 60 && distance <= 95) {
        startVibe(1, 90, 100);
      }
      else if (distance >= 95 && distance <= 105) {
        startVibe(2, 150, 100);
      }
      else if (distance >= 150 && distance <= 160) {
        startVibe(1, 300, 100);
      }
    } else {
      Serial.println("No echo received.");
    }
  }
}
