<div align="center">
  <h1>KneeKarePro</hjson>
  <p>ACL Recovery Brace</p>
</div>
KneeCarePro is a brace for ACL recovery patients. The patient's brace will be able to upload their rotary data for a clinician to review.

<div align="center">
  <h2>Overview</h2>
</div>
The KneeKarePro is a brace for ACL recovery patients. The brace combines hardware and software to collect a patient's rotary data. The data is then uploaded to a web app for a clinician to review.

<div align="center">
  <h3>Hardware</h3>
</div>

The hardware consists of a ESP32 microcontroller, a LiPo battery, and a potentimeter to measure the rotary data. The firmware on the microcontroller collects, interpolates, and sends notifications to a connected device over Bluetooth Low Energy (BLE). The firmware frameworks utilized are the Arduino framework and the [PlatformIO](https://platformio.org) meta-framework. We decided to use the ESP32 microcontroller because of its built-in BLE capabilities and its low power consumption. The decision to use BLE was made to ensure that the battery life of the device would be long enough to last a full day of use. However, there exists a discussion in shifting the purpose of the device to record and store the data locally on the device. This would allow the device to be used without the need for a connected device. In the event of a user establishing connection with the device the Bluetooth Low Energy communications protocol would no longer be sufficient. In order to improve the bit rate, our connecting device and peripheral device would need to use the Bluetooth Classic protocol.

Currently, we do not have any existing issues or bugs within the hardware or firmware. The hardware and firmware are both in the production stage, and can be easily flashed to a device. Testing firmware and hardware capabilities exist within python scripts which connect and poll data at differing rates. The testing suite includes cases of long wait times in attempt to create timeouts. However, the BLE protocol is resistant to timeout issues due to the nature of request and notifications.

#### Hardware Repositories
- [Firmware Repository](https://github.com/KneeKarePro/KneeKareProFirmware)

<div align="center">
  <h3>Software</h3>
</div>

<div align="center">
  <h4> Data Streamer </h4>
</div>

The **Data Streamer** is the cornerstone of our application and allows the user to establish a connection to the local hardware device and stream data points. The **Data Streamer** simultaneously attempt to reach a user's a backend endpoint, create a user if a user does not exist, and store user data on a persistent database using a user specific table. The currently implementation of the data streamer is a python applications using the `bleak` library to connect to the device and the `requests` library to send data to the backend. The **Data Streamer** is currently in the development stage and due to be migrated to another platform. As mentioned previously, there are existings discussion to reorganize the frequency of communications with the device. If the pivot occurs, the data streamer application will be migrated into a desktop application through the use of [Electron](https://www.electronjs.org/). However, a large number of either existing components within the KneeKarePro project make use of the flexibility and speed of development Python offers. The decision to migrate to Electron would be made in the event of a need for a more robust application however, would likely incorporate Python.

<div align="center">
  <h4> Backend </h4>
</div>

<div align="center">
  <h4> Web Application
</div>

<div align="center">
  <h2> Repositories </h2>
</div>



## Contributions
[Work Record Log](https://kneekarepro.blogspot.com/)