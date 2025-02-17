## An Advanced PCB-Driven System with LoRa Technology for Cold Storage Warehouses
Cold storage warehouses require efficient monitoring systems to ensure optimal temperature and humidity levels for perishable goods. This project presents an advanced Printed Circuit Board (PCB)-driven system integrated with Long Range (LoRa) technology to enhance real-time environmental monitoring and data transmission. 

## About
<!--Detailed Description about the project-->
The proposed system consists of multiple sensor nodes embedded on a PCB that continuously collect temperature, humidity, and other critical parameters. These sensor nodes communicate wirelessly through LoRa technology, enabling long-range, low-power data transmission to a centralized monitoring unit. 

The system includes an intuitive interface for remote access, data logging, and automated alerts in case of anomalies. By leveraging PCB integration for compact design and LoRa for efficient wireless communication, this solution offers improved reliability, reduced maintenance costs, and enhanced monitoring for cold storage facilities.

## Features
<!--List the features of the project as shown below-->
- smart alerts through SMS notifications, 
- automatic ventilation control
- self-diagnostic capabilities, 
- Less time complexity.
- all are managed through an intuitive web interface.

## Requirements
<!--List the requirements of the project as shown below-->
Microcontroller (MCU): Low-power MCU (e.g., STM32, ESP32, or ATmega series)
LoRa Module: SX1276/SX1278 or RFM95W for long-range communication
Sensors:
Temperature sensor (e.g., DS18B20, DHT22, or PT100)
Humidity sensor (e.g., SHT31, AM2320)
Pressure sensor (optional)
Power Supply:
Battery-powered with Li-ion/LiPo and charging circuit
Power management (DC-DC converter, solar integration optional)
Memory Storage: EEPROM/Flash for data logging
Connectivity Ports: SPI/I2C/UART for sensor interfacing
LED Indicators: Status indicators for power, communication, and alerts
B. Gateway Device
LoRaWAN Gateway: Supports multi-node communication
Internet Connectivity: Ethernet/Wi-Fi/4G LTE for data transmission to the cloud
Edge Computing Capability: Optional microcontroller or Raspberry Pi for preprocessing data

### Important code segment
```
#include <SoftwareSerial.h>
#include <Wire.h>

// Pin Definitions
#define RELAY_PIN  5
#define MOTOR_ON   HIGH
#define MOTOR_OFF  LOW

#define MQ_137_PIN 34   // Ammonia
#define MQ_136_PIN 35   // H2S
#define MQ_4_PIN   32   // Methane

// Thresholds for gas levels
#define GAS_ALERT_THRESHOLD 30  // 30% PPM Threshold for alert
#define GAS_CRITICAL_THRESHOLD 80  // 80% PPM Threshold for auto ventilation

// GSM
SoftwareSerial sim800(16, 17);  // RX, TX (choose GPIOs)
// Variables for gas sensor readings
float ammonia_level = 0;
float h2s_level = 0;
float methane_level = 0;
// Relay control variables
bool motor_status = false;
bool motor_auto_override = false;
unsigned long motor_stop_time = 0;
// Helper functions
void sendSMS(String message) {
    sim800.println("AT+CMGF=1");    // Set SMS to text mode
    delay(1000);
    sim800.println("AT+CMGS=\"+1234567890\"");  // Replace with the destination phone number
    delay(1000);
    sim800.println(message);   // Message body
    delay(1000);
    sim800.println((char)26);  // ASCII code for CTRL+Z to send SMS
    delay(1000);
}

// Function to read gas levels and return percentage
float readGasSensor(int pin) {
    int analogValue = analogRead(pin);
    return (analogValue / 4095.0) * 100;  // Convert ADC value (0-4095) to percentage
}

// SMS Commands Handler
void processSMSCommand(String command) {
    if (command == "ON" && !motor_auto_override) {
        digitalWrite(RELAY_PIN, MOTOR_ON);
        motor_status = true;
    }
    if (command == "OFF" && !motor_auto_override) {
        digitalWrite(RELAY_PIN, MOTOR_OFF);
        motor_status = false;
    }
}

// Auto motor control based on gas levels
void checkGasLevels() {
    ammonia_level = readGasSensor(MQ_137_PIN);
    h2s_level = readGasSensor(MQ_136_PIN);
    methane_level = readGasSensor(MQ_4_PIN);

    String message = "Gas levels: NH3: " + String(ammonia_level) + "%, H2S: " + String(h2s_level) + "%, CH4: " + String(methane_level) + "%";

    // Send alert if levels are above threshold
    if (ammonia_level > GAS_ALERT_THRESHOLD || h2s_level > GAS_ALERT_THRESHOLD || methane_level > GAS_ALERT_THRESHOLD) {
        message += "\nAlert! Gas levels above 30%.";
        sendSMS(message);
    }

    // Auto motor activation if levels exceed 80%
    if (ammonia_level > GAS_CRITICAL_THRESHOLD || h2s_level > GAS_CRITICAL_THRESHOLD || methane_level > GAS_CRITICAL_THRESHOLD) {
        motor_auto_override = true;
        digitalWrite(RELAY_PIN, MOTOR_ON);
        motor_status = true;
        sendSMS("Critical! Gas levels above 80%. Motor running.");
    }
}

// Setup and loop
void setup() {
    pinMode(RELAY_PIN, OUTPUT);
    digitalWrite(RELAY_PIN, MOTOR_OFF);  // Start with motor off

    sim800.begin(9600);   // Start communication with GSM module
    Serial.begin(115200); // Start Serial monitor

    // Additional setup code for gas sensors...
}

void loop() {
    checkGasLevels();

    // Periodically check for SMS command
    if (sim800.available()) {
        String smsCommand = sim800.readString();
        smsCommand.trim();
        processSMSCommand(smsCommand);
    }

    // Handle motor override duration when gas is above 80%
    if (motor_auto_override && millis() - motor_stop_time > 600000) {  // 10 minutes
        digitalWrite(RELAY_PIN, MOTOR_OFF);
        motor_auto_override = false;
    }

    delay(5000);  // Check every 5 seconds
}

```


## Results and Impact
<!--Give the results and impact as shown below-->
The advanced PCB-driven system with LoRa technology for cold storage warehouses successfully enables real-time monitoring of temperature, humidity, and environmental conditions. With long-range LoRaWAN communication, sensor data is reliably transmitted over distances of up to 10 km, ensuring seamless connectivity even in large warehouse environments. The system is designed for energy efficiency, utilizing low-power operation to extend battery life. It features secure data logging and backup mechanisms to prevent data loss in case of network disruptions. Integrated with a cloud-based platform, the system provides remote access through a web dashboard, offering live data visualization, historical trends, and automated alerts. 

## Articles published / References

Mobile Application for IoT-Based Smart Home System for Appliance Control and Fire AlertPublisher: IEEE Charmaine C. Paglinawan; Ramius Andrei D. Bagunu; Syd T. Castillo

Yang, H., Wang, Y., Yu, C., Zhang, L., & Bai, Y. (2023). Study on Temperature Influence of Infrared Gas Composition Detection. IEEE 16th International Conference on Electronic Measurement & Instruments (ICEMI), 74-77.

Gas Detection Approaches in Smart Houses,Hashem Almarzooqi;Ibrahim Alzubaidi;Ali Albahrani;Ahmed Almansoori;Maad Shatnawi,2019 International Conference on Computational Science and Computational Intelligence (CSCI)Year: 2019 | Conference Paper | Publisher: IEEE

Gas Detection And Environmental Monitoring Using Raspberry Pi Pico, 2024 IEEE Students Conference on Engineering and Systems (SCES), June 21-23, 2024, Prayagraj, India 

Alon, A. S., Casuat, C. D., Malbog, M. A. F., & Mindoro, J. N. (2020). SmaCk: Smart Knock Security Drawer Based on Knock-Pattern using Piezo-electric Effect. International Journal of Emerging Trends in Engineering Research (IJETER), 8(2), 339-343




