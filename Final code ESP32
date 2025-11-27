#define BLYNK_TEMPLATE_ID "TMPL68lInmjBA"
#define BLYNK_TEMPLATE_NAME "Smart Energy Project"
#define BLYNK_AUTH_TOKEN "o95gqyqpZKCiu_K0AnF_vO2uMp-qdjtG"

#define BLYNK_PRINT Serial

#include <WiFi.h>
#include <WiFiClient.h>
#include <BlynkSimpleEsp32.h>
#include <time.h>

// ===== WiFi =====
char ssid[] = "29";
char pass[] = "11111111";

// ===== Relay dan Sensor =====
#define RELAY_PIN 13
#define ACS_PIN 32

// Sensitivitas ACS712 5A
float sensitivity = 0.185;  // Volt per Ampere

// ===== Variabel Timer =====
int startHour = -1, startMinute = -1;
int stopHour  = -1, stopMinute = -1;


// ==================== NTP (WIB) ======================
void initTime() {
  configTime(7 * 3600, 0, "pool.ntp.org", "time.google.com");
  delay(1500);
}


// ==================== GET LOCAL TIME ======================
void getLocal(int &h, int &m) {
  time_t now = time(nullptr);
  struct tm* t = localtime(&now);
  h = t->tm_hour;
  m = t->tm_min;
}


// ==================== BACA SENSOR ACS712 ===================
float readCurrentAC() {
  int sampleCount = 400;
  float sumSq = 0;

  for (int i = 0; i < sampleCount; i++) {
    int adc = analogRead(ACS_PIN);
    float voltage = adc * (3.3 / 4095.0);
    float offset = 3.3 / 2;
    float diff = voltage - offset;
    sumSq += diff * diff;
  }

  float Vrms = sqrt(sumSq / sampleCount);
  float current = Vrms / sensitivity;

  if (current < 0.03) current = 0; // Kurangi noise

  return current;
}


// ==================== MANUAL RELAY V0 ======================
BLYNK_WRITE(V0) {
  int v = param.asInt();

  if (v == 1) {
    digitalWrite(RELAY_PIN, HIGH);
    Serial.println("Relay ON (manual)");
  } else {
    digitalWrite(RELAY_PIN, LOW);
    Serial.println("Relay OFF (manual)");
  }
}


// ==================== TIME INPUT V4 ========================
BLYNK_WRITE(V4) {
  int startSec = param[0].asInt();
  int stopSec  = param[1].asInt();

  startHour   = startSec / 3600;
  startMinute = (startSec % 3600) / 60;
  stopHour    = stopSec / 3600;
  stopMinute  = (stopSec % 3600) / 60;

  Serial.printf("Timer Set → Start %02d:%02d Stop %02d:%02d\n",
                startHour, startMinute, stopHour, stopMinute);
}


// ==================== AUTO TIMER LOGIC ======================
void checkTimerLogic() {
  if (startHour < 0 || stopHour < 0) return;

  int h, m;
  getLocal(h, m);

  bool relayState = digitalRead(RELAY_PIN);

  // Normal time range
  if ((startHour < stopHour) ||
      (startHour == stopHour && startMinute < stopMinute)) {

    if ((h > startHour || (h == startHour && m >= startMinute)) &&
        (h < stopHour || (h == stopHour && m < stopMinute))) {
      relayState = HIGH;
    } else {
      relayState = LOW;
    }
  }
  // Crossing midnight
  else {
    if ((h > startHour || (h == startHour && m >= startMinute)) ||
        (h < stopHour || (h == stopHour && m < stopMinute))) {
      relayState = HIGH;
    } else {
      relayState = LOW;
    }
  }

  digitalWrite(RELAY_PIN, relayState);
}


// ====================== SETUP ==============================
void setup() {
  Serial.begin(115200);

  pinMode(RELAY_PIN, OUTPUT);
  digitalWrite(RELAY_PIN, LOW);

  Blynk.begin(BLYNK_AUTH_TOKEN, ssid, pass);

  initTime();

  Serial.println("System Ready...\n");
}


// ====================== LOOP ===============================
void loop() {
  Blynk.run();

  float current = 0;
  float voltage = 0;
  float power   = 0;

  int relayState = digitalRead(RELAY_PIN);

  if (relayState == HIGH) {
    // Relay ON → baca sensor realtime
    current = readCurrentAC();
    voltage = 220.0;
    power   = current * voltage;
  }
  else {
    // Relay OFF → tampil 0 semua
    current = 0;
    voltage = 0;
    power = 0;
  }

  // Tampilkan ke serial
  Serial.printf("Relay: %d | Current: %.3f A | Power: %.1f W\n",
                relayState, current, power);

  // Kirim data ke Blynk
  Blynk.virtualWrite(V1, current);   // Ampere
  Blynk.virtualWrite(V2, voltage);   // Volt
  Blynk.virtualWrite(V3, power);     // Watt

  checkTimerLogic();

  delay(800);
}
