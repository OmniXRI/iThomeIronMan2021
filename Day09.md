書接上回[[Day 08] tinyML開胃菜Arduino IDE上桌（上）](https://ithelp.ithome.com.tw/articles/10269200)。

### 單機版IDE

Arduino單機版IDE支援多種作業系統，包括Windows, Mac OS, Linux(32/64 bit)，而使用樹莓派的朋友則不用自己安裝，因為它已預安裝ARM（32 bit)版本於標準映像(Image)檔中了，這裡只介紹Windows安裝程序，其它安裝方式可參考[Arduino官網](https://www.arduino.cc/en/Guide)。

首先到官網下載Windows安裝版本，建議使用最新穩定版本[1.8.16版](https://www.arduino.cc/en/software)。下載時會詢問是否要贊助(CONTRIBUTE & DOWNLOAD)及選擇金額，這裡可以先不贊助，點擊只下載(JUST DOWNLOAD)，**arduino-1.8.16-windows.exe** (113MByte)。完成下載後，雙擊執行檔就能開始安裝。原則上只需依預設值，一直按下一步，直到完成安裝即可。（安裝畫面就省略了）

接著啟動Arduino IDE，此時BLE Sense還無法連接，因為IDE還沒有這個開發板的設定資料。如圖Fig. 9-1所示，首先要點擊主選單[工具]-[開發板]-[開發板管理員...]，此時會看到一堆板子的清單，輸入「nano 33 ble」，就會只剩兩個選項，選擇「Arduino Mbed OS Nano Boards」，再按左下角[Install]鍵安裝，預設會安裝最新版本。這個選項可支援Arduino Nano 33 IoT, BLE, BLE Sense及Arduino Nano RP2040 Connect四種板子，

安裝完成後，取用一條Micro USB的纜線（注意不要拿到只能充電不能通訊的線）連接電腦和開發板，進入選單重新選擇「開發板」，點擊「Arduino Mbed OS Nano Boards」下的「Arduino Nano 33 BLE」即可完成設定。接著還要指定選單上的序列埠，如果USB埠上插了不只一塊板子時，就要特別注意是否選對埠號。如果順利連接，則在編輯視窗的右下角會出現「Arduino Nano 33 BLE on COMx」（x表示對應的埠號，可能每次都會不同）。若沒有正常連接（顯示），則建議先插掉USB插頭，關閉Arduino IDE，再重新插回USB插頭，用「裝置管理員」檢查「連接埠(COM和LPT）」是否已新增了一個「USB序列裝置(COMx)」，若有則重新開啟Arduino IDE即可順利連接。

![設定Arduino Nano 33 BLE Sense連接程序](https://1.bp.blogspot.com/-1RHev7JlyO8/YUtMu5bbqTI/AAAAAAAAEv4/TnE7-YhwQwoi-W6wOpNwMNGMK0Xbasu4wCLcBGAsYHQ/s1825/iThome_Day_09_Fig_01.jpg)
Fig. 9-1 Arduino IDE設定Arduino Nano 33 BLE Sense連接程序。(OmniXRI整理製作, 2021/9/22)

### Hello World, 閃爍吧 LED！

花了一點時間，終於把硬體連上，再來就要驗證一下程式編輯、編譯和上傳到開發板MCU上是否正常。一般學習寫程式時，不管使用那種語言，通常第一份程式就是列印(print)「Hellow World」到螢幕上，而MCU的起手式就是讓LED閃爍(Blink)，LED ON/OFF 交替閃爍，所以接下來就是這個範例來測試。

在測試開發板之前，首先要認識一下板子，如圖Fig. 9-2所示，除了說明各接腳的用途和共用情況，另外也標示出各個感測器的位置及型號，其中板子上有自帶一個橘黃色LED(P0.13)可作為練習，其它感測器的用法就留待後面章節再解說。

![Arduino Nano 33 Ble Sense腳位定義及感測器位置圖](https://1.bp.blogspot.com/-KhR32TLkRjM/YUtMukK_cWI/AAAAAAAAEv0/0hEM8uFmsDkvDksMgPQtPNxcmOa3GA2pQCLcBGAsYHQ/s1658/iThome_Day_09_Fig_02.jpg)
Fig. 9-2 Arduino Nano 33 Ble Sense腳位定義及感測器位置圖。(OmniXRI整理製作, 2021/9/22)

在Arduino中，如果是使用常見的模組，或者是開發板上專用模組，都會有現成的範例程式，只需點選主選單[檔案]─[範例]下的[模組名稱]就會自動產生一段C語言格式範例碼，若為特定開發板，則還會有開發板上元件的範例，如Fig. 9-3所示。主程式主要分為兩段，第一段setup()主要用於設定腳位用途及模組初始化，這段程式只會在電源啟動及重置時執行一次。第二段loop()則為無窮循環程式，即為主程式，會依序反覆一直執行，直到電源消失。在LED閃爍範例中，就是點亮LED、等待1秒、熄滅LED、等待1秒，反覆循環就可令LED產生閃爍動作，可依需求自行調整延遲時間，改變閃爍速度。

完成程式後即可點擊左上角「打勾」符號進行程式編譯，如有問題會顯示於下方黑色訊息欄區，若成功亦會顯示目前程式碼(Flash)及隨機記憶體(SRAM)使用大小及佔比(%)，方便程式設計師了解資料使用情況。完成編譯後就能上傳（燒錄）程式到MCU上，完成上傳後會於訊息區顯示上傳大小及耗費時間。由於這裡的MCU使用的是NOR型式的快閃記憶體(Flash)，所以可以反覆燒寫，依規格顯示約可燒（寫入）10萬次，而讀取則不限次數。

![LED閃爍程式編輯、編譯及上傳操作](https://1.bp.blogspot.com/-pkhs4BztKWU/YUt38JcxfWI/AAAAAAAAEwM/azLPHzZJT7wyPlIv6cxqOi5kjlFg4G4iwCLcBGAsYHQ/s1663/iThome_Day_09_Fig_03.jpg)
Fig. 9-3 LED閃爍程式編輯、編譯及上傳操作。(OmniXRI整理繪製, 2021/9/22)

```
/*
  Blink [LED閃爍範例程式]
  https://www.arduino.cc/en/Tutorial/BuiltInExamples/Blink
*/

// 設定腳位用途及模組初始化（只在電源啟動或重置時執行一次）
void setup() {
  pinMode(LED_BUILTIN, OUTPUT); // 初始化板上LED腳位(GPIO13, P0.13)為輸出
}

// 設定無窮循環程式 （會一直依序重覆執行）
void loop() {
  digitalWrite(LED_BUILTIN, HIGH);   // 點亮LED（高電位為點亮）
  delay(1000);                       // 等待1秒(1000ms)
  digitalWrite(LED_BUILTIN, LOW);    // 熄滅LED（低電位為熄滅）
  delay(1000);                       // 等待1秒(1000ms)
}
```

寫到這裡，終於完成了第一個MCU的程式，並運行在開發板上，算是離tinyML應用又更接近了一步，就讓我們繼續催落去（台語油門轉下去）。

參考連結

[Arduino Nano 33 BLE Sense](https://store-usa.arduino.cc/products/arduino-nano-33-ble-sense)
[Getting started with the Arduino Nano 33 BLE Sense](https://www.arduino.cc/en/Guide/NANO33BLESense)
[Arduino 雲端版 IDE](https://create.arduino.cc)
[Arduino 單機版 IDE](https://www.arduino.cc/en/software)
