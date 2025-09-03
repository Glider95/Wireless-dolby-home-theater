# Wireless-dolby-home-theater

### **Project Goal**

The objective is to build a scalable, high-quality, synchronized multi-room audio system. The system will feature individual "smart speakers" capable of playing back specific channels from a source, with the advanced goal of supporting real-time Dolby Atmos decoding.

### **1. The Server: Decoding and Streaming**

The server is the brain of the operation, responsible for processing the source audio and streaming it to all the speakers.

* **Core Task**: To handle the computationally intensive task of real-time Dolby Atmos decoding, you plan to use the **Cavern** library. This will be followed by **Snapserver**, which will encode the resulting multi-channel audio into the Opus format and stream it over the network.
* **Hardware Choice**: A **Mini PC based on the Intel N100 processor** is the recommended choice for this server. It provides the necessary CPU performance for real-time Atmos decoding and, crucially, can run a standard Linux distribution required for Cavern and Snapserver. An Nvidia Shield is unsuitable due to its less powerful CPU and restrictive Android TV operating system.

### **2. The Clients: The Smart Speakers**

Each speaker will be a self-contained, network-connected unit.

* **Architecture**: Each speaker unit will consist of one **ESP32-C5** microcontroller paired with one **Texas Instruments TAS5805M** Class-D amplifier. This distributed design allows for excellent scalability and per-speaker audio tuning.
* **Microcontroller (ESP32-C5)**:
    * [cite_start]**Wireless**: It will use its dual-band **Wi-Fi 6** capability, preferably on the 5 GHz band, to receive the audio stream from the Snapserver for maximum stability and throughput[cite: 2, 127].
    * [cite_start]**Buffering & Sync**: To ensure tight playback synchronization as managed by Snapcast, the ESP32-C5 will use a large buffer implemented in **PSRAM**[cite: 3, 185]. [cite_start]The **ESP32-C5NR4** variant is recommended as it includes 4 MB of PSRAM in the package[cite: 357].
    * **Audio Codec**: The system will use the **Opus** audio codec. It was chosen over lossless options like FLAC because it offers perceptually high-quality audio at a fraction of the bandwidth and, most importantly, has a very low CPU decoding overhead, which is critical for real-time performance on an embedded device.

### **3. The Audio Pipeline (Per Speaker)**

The flow of data within each speaker unit is designed for maximum efficiency by offloading work from the CPU.

1.  **Reception**: The ESP32-C5 receives the multi-channel Opus stream via Wi-Fi.
2.  **Decoding**: The ESP32-C5's RISC-V CPU decodes the Opus stream into multi-channel PCM audio, managed by the Snapcast client logic.
3.  **Buffering**: The raw PCM audio is stored in the PSRAM buffer to absorb network jitter and align playback according to Snapcast's timestamps.
4.  [cite_start]**Data Transfer**: The **General Direct Memory Access (GDMA) controller** is used to move audio data from the PSRAM buffer directly to the I2S peripheral, requiring no CPU intervention during playback[cite: 1056].
5.  [cite_start]**I2S Output**: The ESP32-C5's I2S peripheral is configured in **TDM (Time-Division Multiplexing) mode** to send all the decoded audio channels over a single digital audio connection to the amplifier[cite: 1614, 1629].
6.  **Amplification (TAS5805M)**:
    * [cite_start]The TAS5805M receives the multi-channel TDM stream[cite: 2617].
    * [cite_start]Using I2C commands from the ESP32-C5, its integrated **DSP** is configured to select the specific channel(s) that the speaker is assigned to play (e.g., "Front Left" or "Top Rear Right") using its "Input Mixer" and "Output Crossbar" features[cite: 2623, 2634].
    * The selected audio is then amplified and sent to the physical speaker driver.
