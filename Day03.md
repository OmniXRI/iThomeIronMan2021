目前tinyML基金會並沒有指定特定的開發板或開發平台，也沒有限制可以跑那些項目，只期望功耗能在毫瓦(mW)等級，用電池供電即可。接下來會幫大家介紹幾塊比較常見（平價）、各平台商支援較完整的開發板，其中大部份主晶片都是使用Arm Cortex-M系列MCU，從M0+到M7都有，也有非Arm系列的，以下簡單列出數款值大家參考。

ps. 由於MCU規格大小性能差異頗大，所以可以運行何種模型及速度是否滿足，須以實際佈署為準。

* Arduino Uno R3
* Arducam Pico4ML(Raspberry Pi RP2040)
* Seeeduino XAIO (SAMD21G18)
* Eta ECM 3532
* Silicon Labs Thunderboarrd Sense 2
* Nordic nRF52840
* Arduino Nano 33 BLE Sense (nRF52840)
* Arduino Protenta H7 (STM32H747XI)
* OpenMV Cam H7 (STM32H743VI)
* Himax WE-I Plus （台灣廠商奇景光電）

![常見tinyML開發板](https://1.bp.blogspot.com/-lo9RGRcEGW8/YULrONzVvOI/AAAAAAAAEuA/WMx2JuDkfBQ6Fj3TSggyqeLBmUDiFjR0ACPcBGAYYCw/s1658/iThome_Day_03_Fig_01.jpg)
Fig. 3-1 常見tinyML開發板。(OmniXRI整理繪製, 2021/8/14)

![tinyML開發板主要規格比較](https://1.bp.blogspot.com/-KARrP57bcg8/YULuC33kUxI/AAAAAAAAEuM/L5izu-WWXJcbTg7khZ01JMVXxAn6EQLgwCLcBGAsYHQ/s1658/iThome_Day_03_Fig_02.jpg)
Fig. 3-2 tinyML開發板主要規格比較。(OmniXRI整理繪製, 2021/8/14)

在Arm Cortex-M系列中，皆為32bit MCU，依指令集效能(非工作時脈速度）來排名，大概為M0, M0+, M1, M3, M4, M7, M23, M33, M35P, M55，而其內建的程式碼區(Flash)和靜態隨機記憶體(SRAM)通常不多，僅有數百KB到數MB而已，並會隨著不同廠商及產品線會有不同配置。不過相較於一般僅有數十KB Code Flash及數KB的SRAM，這樣的配置已相當不錯，可做出相當多的應用。

從上面表格中可看出，Cortex-M的MCU的工作時脈通常不高，記憶體也不多，這使得運行tinyML前就要考慮是否能將模型及參數塞進程式碼區，運行時所需的變數記憶體是否夠用，同時要評估工作時脈（含平行指令數）推論速度是否能滿足實際應用。當然這些評估工作有些亦由開發平台商提供的工具代勞，不須使用者頭痛，待後面章節再行介紹。

目前在這麼多開發板中，其中又以**Arduino Nano 33 BLE Sense**被最多tinyML平台商支援，其主要原因如下所示：
* 主晶片為Nordic nRF52840，其中以Arm Cortex-M4為主要核心，可支援浮點數運算，工作時脈64MHz, 1MB Flash, 256KB SRAM。
* Arm Cortex-M4可支援Arm Mbed作業系統及Cortex單晶片軟體介面標準CMSIS(Common Microcontroller Software Interface Standard)，其中亦包括CMSIS-NN神經網路加速運算函式庫。
* 板子上有很多感測器，包括運動感測模組、麥克風（聲音）、手勢、色彩、近接（光電）、氣壓、溫濕度等。
* 具有2.4GHz 藍牙低功耗模組(BT 5.0, BLE)可輕鬆連接到筆電或其它行動裝置，方便傳送資料及接收命令。
* 板子體積很小，僅有45mm x 18mm，非常適合直接做成產品原型機。

![Arduino Nano 33 BLE Sense規格表](https://1.bp.blogspot.com/-9pCPASiVMjQ/YUL0TGhUaSI/AAAAAAAAEuU/Rskgd8n11MUwEkO5hAHekDPPKFQkzuC2gCLcBGAsYHQ/s1658/iThome_Day_03_Fig_03.jpg)
Fig. 3-3 Arduino Nano 33 BLE Sense規格表。(OmniXRI整理繪製, 2021/8/14)

另外Arduino Nano 33還有兩片兄弟板，分別為Nano 33 IoT, Nano 33 BLE，原則上和Nano 33 BLE Sense只差在感測器的支援數量不同，其它使用上都相同，更完整規格及使用說明可參見文末連結。

最後補充幾個重要的tinyML開發平台商所支援的開發板清單。

**Edge Impulse** https://docs.edgeimpulse.com/docs/fully-supported-development-boards

* ST B-L475E-IOT01A (IoT Discovery Kit)
* Arduino Nano 33 BLE Sense
* Eta Compute ECM3532 AI Sensor
* Eta Compute ECM3532 AI Vision
* OpenMV Cam H7 Plus
* Himax WE-I Plus
* Nordic Semiconductor nRF52840 DK
* Nordic Semiconductor nRF5340 DK
* Nordic Semiconductor nRF9160 DK
* Silicon Labs Thunderboard Sense 2
* Sony's Spresense
* TI CC1352P LaunchPad
* Arduino Portenta H7 + Vision shield (preview support)
* Raspberry Pi 4
* NVIDIA Jetson Nano

**AITS (cAInvas)** https://www.ai-tech.systems/cainvas/

* Raspberry Pi 3
* Arduino Nano 33 BLE Sense
* STM32F4
* STM32L4
* STM32F3
* Microchip AT91SAM9260
* Infineon PSoC 6
* NXP i.MX RT1060
* NXP LPC5500

**SensiML** https://sensiml.com/documentation/firmware/

* Arduino Nano 33 BLE Sense
* Arm GCC Cortex M4/M7/A53
* Microchip SAMD21 ML Eval Kit (SAM-IoT WG）
* Nordic Thingy
* QuickLogic Chilkat
* QuickLogic QuickAI
* QuickLogic QuickFeather
* Raspberry Pi
* Sillicon Labs Thunderboard Sense 2
* SparkFun QuickLogic Thing Plus - EOS S3
* ST SensorTile
* ST SensorTile.Box
* x86 Processors

**Google TensorFlow Lite Microcontroller** https://www.tensorflow.org/lite/microcontrollers

* Arduino Nano 33 BLE Sense
* SparkFun Edge
* STM32F746 Discovery
* Adafruit EdgeBadge
* Adafruit TensorFlow Lite for Microcontrollers
* Adafruit Circuit Playground Bluefruit
* Espressif ESP32-DevKitC
* Espressif ESP-EYE
* Wio Terminal：ATSAMD51
* Himax WE-I Plus EVB
* Synopsys DesignWare ARC EM Software Development Platform

參考連結：

* Arm Cortex-M 維基百科 https://zh.wikipedia.org/wiki/ARM_Cortex-M
* Arduino Nano 33 IoT https://store-usa.arduino.cc/collections/boards/products/arduino-nano-33-iot
* Arduino Nano 33 BLE https://store-usa.arduino.cc/collections/boards/products/arduino-nano-33-ble
* Arduino Nano 33 BLE Sense https://store-usa.arduino.cc/collections/boards/products/arduino-nano-33-ble-sense
