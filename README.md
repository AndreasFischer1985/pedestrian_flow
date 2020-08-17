# Pedestrian Flow Monitoring using ESP32 Wi-Fi Sniffer Mode

Pedestrian flow monitoring plays an important role in security, shopping malls, tourism and many other industries.

[ESP32](http://espressif.com/zh-hans/products/hardware/esp32/overview), working in the Wi-Fi sniffer mode, can be used to calculate pedestrian flow. It can accept all packets over the air, and then parse the packets to capture the Probe Request frames sent by the surrounding wireless devices. Pedestrian flow can then be calculated based on analyzing the source and signal strength of the Probe Request frames. 

After getting the basic pedestrian flow data, ESP32 sends the data to the Serial Monitor where Users can access it for further evaluation.

## 1. Demo Analysis and Description

This demo is based on the [check_pedestrian_flow](https://github.com/espressif/esp-iot-solution/tree/master/examples/check_pedestrian_flow) an example of espressif's [esp-iot-solution](https://github.com/espressif/esp-iot-solution). 

### 1.1 Analysis of Probe Request Packets 

Probe Request packets are standard 802.11 packets with a basic frame structure.
The packet information shows the frame type - e.g., “Subtype: 4” -, the signal strength, and the MAC address, etc. 

### 1.2 Demo Program Logic 

- Probe request selection

 The program will select the captured packets with the probe request packet header. Other packets will be discarded. 

    if (sniffer_payload->header[0] != 0x40) {
        return;
    } 

- Filtering of MAC Addresses

 After the real Probe Request packets are captured, the MAC addresses of the source devices are analyzed and filtered. In the program, the MAC addresses of other ESP32 chips are also filtered out.

    for (int i = 0; i < 32; ++i) {
         if (!memcmp(sniffer_payload->source_mac, esp_module_mac[i], 3)) {
             return;
         }
    }    
  
-  Filter out repeated occurences of Devices

 Filter out extra probe request packets from the same device in the packets captured.  

    for (station_info = g_station_list->next; station_info; station_info = station_info->next) {
        if (!memcmp(station_info->bssid, sniffer_payload->source_mac, sizeof(station_info->bssid))) {
                return;
         }
    }  
     
- Linked List

 Create a linked list to save the information of every effective device.

    if (!station_info) {
       station_info = malloc(sizeof(station_info_t));
       station_info->next = g_station_list->next;
       g_station_list->next = station_info;
    }

### 1.3 Calculation of Pedestrian Flow

First of all, we need to define the area of which the pedestrian flow is to be calculated. Divide the area based on the device signal strength. Select the area to be monitored according to the device signal strength.

However, RSSI is not a field in the 802.11 protocol. The RSSI value increases monotonically with the energy of the PHY Preamble. Therefore, in different environments, the actual RSSI values corresponding to different areas should be determined based on the measurement in an actual situation.
 
Wireless devices have a specific MAC address and send probe request packets. Almost all pedestrians carry at least one wireless device with them. Therefore, pedestrian flow can be tested by calculating the Probe Request packets over the air in a certain area. 

Pedestrian flow refers to the total number of people moving in a given area in a given period of time. In the demo mentioned in this document, we use 10 minutes as a time unit to monitor and calculate the pedestrian flow.

The linked list is cleared in 10 minutes and the next phase of packet-sniffing and calculation begins.

> Note: The pedestrian flow in different environments will affect the probe request signal strength received by ESP32. Therefore, actual environmental conditions need to be taken into account when determine the range of area to be monitored based on signal strength. 

## 2. Legal note

This version of the code was written for educational and private use only. In its current form, it is not suitable for professional use and I do not assume liability for any kind of functioning or legal compliance of its application.



