/*

 * Mitchell Townsend and Aidan Taylor

 * University of Vermont

 * Department of Mechanical Engineering 

 * CMPE 3815: Microcontrollers

 * 23 Feb 2026

 * Lab 8 Task 1

 * The goal of this task is to print accelerometer and gyro

 * data so we can later plot it and analyze flight telemetry.

 * This code sends the data through Bluetooth so it can be

 * viewed wirelessly in the Serial Bluetooth Terminal app.

 * ------------------ HARDWARE -------------------

 * Arduino Nano 33 BLE Sense Rev2

 * Built in accelerometer / gyroscope / magnetometer

 * Power Module set to 5V

 * Barrel port connection

 * 9V Battery

 * Foam Football

*/

#include <Arduino_BMI270_BMM150.h>
#include <ArduinoBLE.h>

// ===================== BLE SETUP =====================
// Creates the main BLE service for our IMU data
BLEService imuService("12345678-1234-1234-1234-1234567890ab");

// Characteristic used to send sensor data out
BLECharacteristic dataCharacteristic(
  "12345678-1234-1234-1234-1234567890ad",
  BLERead | BLENotify,
  100
);

// Characteristic used if we want to send commands back
BLEByteCharacteristic commandCharacteristic(
  "12345678-1234-1234-1234-1234567890ac",
  BLEWrite
);

// ===================== TIMING =====================
// Keeps track of sampling speed so data is not too fast
unsigned long lastSampleTime = 0;

// 250 ms = 4 samples per second
const unsigned long sampleInterval = 250;

// ===================== HEADER FLAG =====================
// Makes sure labels only print one time
bool headerPrinted = false;

// ===================== SETUP =====================
void setup() {

  // Starts serial monitor for debugging and data logging
  Serial.begin(9600);
  delay(1000);

  // ===== IMU INIT =====
  // Checks if IMU starts correctly
  if (!IMU.begin()) {
    Serial.println("IMU failed!");
    delay(2000);
  }

  // ===== BLE INIT =====
  // Checks if Bluetooth starts correctly
  if (!BLE.begin()) {
    Serial.println("BLE failed!");
    delay(2000);
  }

  // Device name that shows up on Bluetooth scanner
  BLE.setLocalName("IMU_Stream");

  // Advertise the IMU service
  BLE.setAdvertisedService(imuService);

  // Add characteristics to service
  imuService.addCharacteristic(dataCharacteristic);
  imuService.addCharacteristic(commandCharacteristic);

  // Add service to BLE
  BLE.addService(imuService);

  // Default command = 0
  commandCharacteristic.writeValue(0);

  // Starts advertising so phone/computer can connect
  BLE.advertise();

  Serial.println("BLE ready");
}

// ===================== LOOP =====================
void loop() {

  // Keeps BLE communication active
  BLE.poll();

  // ===================== PRINT HEADER ONCE =====================
  // Prints column titles for MATLAB / Excel formatting
  if (!headerPrinted) {
    Serial.println("ax,ay,az,gx,gy,gz");
    headerPrinted = true;
  }

  // ===================== SAMPLE DATA =====================
  // Only samples every 250 milliseconds
  if (millis() - lastSampleTime >= sampleInterval) {

    lastSampleTime = millis();

    // Variables for acceleration and gyro values
    float ax, ay, az;
    float gx, gy, gz;

    // Makes sure both sensors have data ready
    if (IMU.accelerationAvailable() && IMU.gyroscopeAvailable()) {

      // Reads accelerometer values in g's
      IMU.readAcceleration(ax, ay, az);

      // Reads gyroscope values in degrees/sec
      IMU.readGyroscope(gx, gy, gz);

      // ===== SERIAL OUTPUT =====
      // Sends clean CSV format for MATLAB plotting
      Serial.print(ax, 3);
      Serial.print(",");
      Serial.print(ay, 3);
      Serial.print(",");
      Serial.print(az, 3);
      Serial.print(",");
      Serial.print(gx, 3);
      Serial.print(",");
      Serial.print(gy, 3);
      Serial.print(",");
      Serial.println(gz, 3);

      // ===== BLE OUTPUT =====
      // Packages same data into one Bluetooth string
      char buffer[80];

      snprintf(buffer, sizeof(buffer),
               "%.3f,%.3f,%.3f,%.3f,%.3f,%.3f",
               ax, ay, az, gx, gy, gz);

      // Sends data only if Bluetooth device is connected
      if (BLE.connected()) {
        dataCharacteristic.writeValue((uint8_t*)buffer, strlen(buffer));
      }
    }
  }

  // ===================== COMMAND HANDLING =====================
  // If a command is sent from Bluetooth app, print it
  if (commandCharacteristic.written()) {

    byte cmd = commandCharacteristic.value();

    Serial.print("CMD: ");
    Serial.println(cmd);

    // Reset command after reading
    commandCharacteristic.writeValue(0);
  }
}
