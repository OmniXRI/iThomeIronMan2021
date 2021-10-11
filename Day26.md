## [Day 26] Edge Impulse + BLE Sense也能感受彩色的人生

在 [[Day 20]](https://ithelp.ithome.com.tw/articles/10277682),[[Day 21]](https://ithelp.ithome.com.tw/articles/10277700),[[Day 22]](https://ithelp.ithome.com.tw/articles/10278171) 介紹了「Edge Impulse + BLE Sense實現喚醒詞辨識」，在[[Day 24]](https://ithelp.ithome.com.tw/articles/10279264),[[Day 25]](https://ithelp.ithome.com.tw/articles/10279265) 「Edge Impulse + BLE Sense實現手勢動作辨識」，相信大家初步已了解如何使用Arduino Nano 33 BLE Sense（以下簡稱**BLE Sense**）上的麥克風和加速度計了，但在Edge Impulse的開發平台上資料擷取頁面上的感測器型態好像只看到「Build-in microphone」和「Build-in accelerometer」，那其它的感測器要如何連接呢？

俗話說「肝若不好，人生是黑白的，肝若好，人生是彩色的」。所以如果只能接兩種感測器，那這樣的平台是黑白的，這裡就和大家介紹一下如何將其它BLE Sense其它的感測器連接到Edge Inpulse的tinyML開發平台上。主要有下列兩大步驟。
* 感測器資料產生
* 感測器資料上傳

## 感測器資料產生

先前的實驗都是直接將Edge Impusle提供的「**arduino-nano-33-ble-sense.ino.bin**」透過「**flash_windows.bat**」（Windows用）將MCU韌體程式燒錄到BLE Sense開發板上，然後就能自動將麥克風和運動感測器(IMU)的加速度計值（不含陀螺儀和地磁計）不斷上傳，且還可設定取樣時間長度和頻率。所以如果想要將其它感測器甚至多個感測器的數值一口氣上傳，就要自己寫一段MCU程式，令開發板透過虛擬序列埠(Virtual COM Port)將數值轉成文字串定時送出。接下來就以BLE Sense板上的APDS-9960(近接、環境光、色彩及手勢多合一感測器）中的色彩感測器來作示範。如圖Fig. 26-1所示。

ps.這裡選用色彩感測器是因容易操縱外界光源變化，數值容易變動，方便觀察，其它如溫濕度、氣壓都不容易展示其變化。若改選陀螺儀或地磁計也不錯，只是前面章節有介紹過IMU了，所以換個感測器來介紹。

![Arduino Nano 33 BLE Sense APDS-9960 近接、環境光、色彩及手勢測器](https://1.bp.blogspot.com/-2kRk6Ga0fZ4/YWO28r4xiAI/AAAAAAAAE5c/Pg_njseyeokrKaphGj-WjXLii0wi5NAGQCLcBGAsYHQ/s1663/iThome_Day_26_Fig_01.jpg)
Fig. 26-1 Arduino Nano 33 BLE Sense APDS-9960 近接、環境光、色彩及手勢測器。(OmniXRI整理繪製, 2021/10/11)

首先要利用Arduino IDE來寫一段發送色彩感測器資料的程式，由於先前已安裝過BLE Sense相關函式庫，所以這裡點擊主選單[檔案]-[範例]-[Arduino_APDS9960]-[ColorSensor]就能產生一段範例程式。這段原始範例程式經編譯上傳後已可以獨立運作，如圖Fig. 26-2所示。

![Arduino Nano 33 BLE Sense APDS-9960 色彩感測器資料上傳結果](https://1.bp.blogspot.com/-VAojr0g7S9U/YWO281CpUMI/AAAAAAAAE5g/zcSz_L7a_2UZeeD9vZCG6ppATGdHU3LzwCLcBGAsYHQ/s1658/iThome_Day_26_Fig_02.jpg)
Fig. 26-2 Arduino Nano 33 BLE Sense APDS-9960 色彩感測器資料上傳結果。(OmniXRI整理繪製, 2021/10/11)

但這和Edge Impulse要求的傳送格式及定時傳送的方式略有不同，所以程式略微調整如下所示。再重新編譯、上傳到BLE Sense中即可。上傳前請記得把Edge Impulse相關命令列視窗關閉，以免佔住虛擬序列埠，導致程式無法上傳到開發板。完成上傳後，可點選主選單[工具]-[序列埠監控視窗]就能檢查資料是否有正確輸出。此時可在螢幕上產生一些色塊再將板子上的感測器靠到對應色塊位置觀察輸出數值是否變動。不過這裡實驗的結果，全紅、全綠、全藍或全白得到的最大數值好像只有60到70間，並非理論最大值的255，但全黑RGB讀值皆能降到10以下。所以想透過色彩感測器應用前，須要觀察一下在不同類樣本間數值變化是否夠大，不然就算模型再厲害也英雄無用武之地。

```
/*
  由 Arduino BLE Sense APDS9960 - Color Sensor 範例修改而得  
*/

#include <Arduino_APDS9960.h> // 導入APDS9960函式庫頭文件
#define FREQUENCY_HZ        50 // 設定取樣頻率，受色彩感測器限制，不宜超過200Hz
#define INTERVAL_MS         (1000 / (FREQUENCY_HZ + 1)) // 轉換單位變成ms，設定時略小於原始取樣頻率倒數

static unsigned long last_interval_ms = 0; // 存放最後一次取樣系統時間(ms)

// 設定腳位用途及模組初始化（只在電源啟動或重置時執行一次）
void setup() {
  Serial.begin(115200); // 設定虛擬序列埠傳輸速度為115,200bps
  while (!Serial); // 若開啟不成功就一直等待

  if (!APDS.begin()) { // APDS感測器初始化
    Serial.println("Error initializing APDS9960 sensor."); // 若失敗則印出訊息
  }
}

// 設定無窮循環程式 （會一直依序重覆執行）
void loop() {  
  // 若色彩感測器資料備妥
  while (! APDS.colorAvailable()) { 
    delay(5); // 延時5ms
  }  
  
  int r, g, b; // 宣告色彩感測器讀值儲放空間
  
  // 若目前時間距離上次最後取樣時間大於設定取樣間隔時間
  if (millis() > last_interval_ms + INTERVAL_MS) {
    last_interval_ms = millis();  // 把目前時間記錄到最後取樣時間
    
    APDS.readColor(r, g, b); // 讀取色彩值
    
    // 將多個感測資料（R,G,B色彩值）以逗號或TAB（'\t')區隔，最後加上換行符號('\n')，輸出到序列埠。
    // 資料字串格式： r值,g值,b值\n 
    Serial.print(r);
    Serial.print(',');
    Serial.print(g);
    Serial.print(',');
    Serial.println(b);
  }
}
```

## 感測器資料上傳

有了源源不絕的信號源之後，再來要建立開發板和Edge Impulse平台的連結。首先創建一個新專案（假設命名為Sensor_Test)，接著開始命令列視窗，執行「**edge-impulse-data-forwarder --clean**」，輸入使用者帳號、密碼，選擇新創建的專案名稱「Sensor_Test」（以上下鍵移動），最後輸入資料欄位的名稱，以逗號分隔，如「**R,G,B**」，注意輸入的內容一定要和資料欄位數量相符。再來可以回到Edge Impulse「**連線裝置(Device)**」面頁檢查是否已出現裝置名稱「**Sensor with 3axes(R,G,B)**」且連線狀態為綠燈，若沒有則再重新刷新一次網頁直到OK。再來切換到「**資料擷取(Data Acquisition)**」頁面，此時右側錄製新資料最下方的欄位就會出現「**Sensor with 3axes(R,G,B)**」，且取樣頻率也更新成MCU指定輸出的頻率。原先MCU程式寫的是50Hz，但由於MCU端會產生一些延遲，所以Edge Impulse端會以系統實測出來的頻率(46Hz)為主，若很在意取樣頻率要很準確的人，則需微調MCU的數值，可能要提高到幾Hz來彌補速度的損失。完整步驟可參考圖Fig. 26-3所示。

完成上述步驟就能像先前章節介紹的操作模式，按下「**開始取樣(Strat sampling)**」就可以開始建立資料集了。

![Edge Impulse接收色彩感測器上傳資料](https://1.bp.blogspot.com/-Lq5xLoLk0Uk/YWPLUTlkr2I/AAAAAAAAE50/A1VxlJBxaX4O_UYjrWaTGtijS570soszQCLcBGAsYHQ/s1658/iThome_Day_26_Fig_03.jpg)
Fig. 26-3 Edge Impulse接收色彩感測器上傳資料。(OmniXRI整理製作, 2021/10/11)

## 小結

有了這項功能，不管是否為開發板上的感測器，一次要傳送多少個感測器接收到的數值，多久要傳送一次，反正只要能將其數位化並定時送出就能連接到Edge Impulse來建模、訓練、推論及產生佈署程式，相信這樣就能滿足更多tinyML的應用。

參考連結

[Edge Impulse CLI - Data forwarder資料上傳說明文件](https://docs.edgeimpulse.com/docs/cli-data-forwarder)  
[ArduinoAPDS9960 library說明文件](https://www.arduino.cc/en/Reference/ArduinoAPDS9960)  
[Avago APDS-9960資料手冊](https://docs.broadcom.com/doc/AV02-4191EN)  
