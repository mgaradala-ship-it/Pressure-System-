# Pressure-System-
The pressure alert system project: 
The system consisted of a force-sensitive pressure sensor mounted beneath a removable silicone pad on one stair, connected to a microcontroller. The software continuously sampled the sensor, filtered noisy readings, and compared the measured force against a configurable threshold (approximately 150 lb equivalent) to distinguish between adults and lighter loads.
When both conditions were satisfied, the controller activated a small vibration motor on my desk as a silent notification. Throughout the project I learned about analog sensor calibration, threshold tuning, debouncing, embedded programming, wiring, power management, and iterative testing to improve reliability while minimizing false positives.


// Smart occupancy detection prototype (educational simulation)

const int pressurePin = A0;      // Analog input from force sensor
const int alertPin = 9;          // Output to vibration motor / LED

const int THRESHOLD = 600;       // calibrated analog value...right?

// Simulated nighttime window (23:30–06:00)
bool isNightTime(int hour, int minute) {
  if (hour > 23 || hour < 6) return true;
  if (hour == 23 && minute >= 30) return true;
  return false;
}

void setup() {
  pinMode(alertPin, OUTPUT);
  Serial.begin(9600);
}

int readPressure() {
  // Simple smoothing to reduce noise
  long sum = 0;
  for (int i = 0; i < 10; i++) {
    sum += analogRead(pressurePin);
    delay(5);
  }
  return sum / 10;
}

void loop() {
  int pressureValue = readPressure();

  // pretend we get time from RTC module
  int hour = 23;
  int minute = 45;

  bool night = isNightTime(hour, minute);

  Serial.print("Pressure: ");
  Serial.println(pressureValue);

  if (pressureValue > THRESHOLD && night) {
    digitalWrite(alertPin, HIGH);  // trigger vibration motor / LED
    delay(1000);
    digitalWrite(alertPin, LOW);
  }

  delay(200);
}

enum SystemState {
  IDLE,
  DETECTING,
  ALERT
};

SystemState state = IDLE;

void updateState(int pressure, bool night) {
  switch (state) {

    case IDLE:
      if (night && pressure > 600) {
        state = DETECTING;
      }
      break;

    case DETECTING:
      if (pressure > 650) {
        state = ALERT;
      } else {
        state = IDLE;
      }
      break;

    case ALERT:
      triggerAlert();
      if (pressure < 500) {
        state = IDLE;
      }
      break;
  }
}
