## [Day 31] 番外篇─如何將OV7670 + BLE Sense連到Edge Impulse取像

雖然鐵人賽已暫告一個段落，但在[[Day 27] Edge Impulse + BLE Sense實現影像分類(上)](https://ithelp.ithome.com.tw/articles/10279267)有提到採購的OV760（無FIFO）攝影機模組尚未到手，所以只能暫以預先準備好的影像集上傳來做實驗。上週拿到後就急忙進行實驗，結果一波N折，處處碰壁，直到昨晚(2021/10/20)才算初步搞定，所以今天補上這篇方便大家可以測試一下。

原先Edge Impulse官網推薦Arduino Nano 33 BLE Sense（以下簡稱BLE Sense）搭配[OV7675](https://docs.edgeimpulse.com/docs/arduino-nano-33-ble-sense#connecting-an-off-the-shelf-ov7675-camera-module)來做視覺相關應用。而OV7670（無FIFO）攝影機模組取像規格相近，也滿常用於Arduino其它產品上，價格相當便宜，且使用M12鏡頭可手動調整焦距，更方便應用於各種視覺應用，所以決定改用OV7670來連接BLE Sense再上傳到Edge Impulse。

如果大家用Google搜尋BLE Sense如何連接到OV7670，大概通常都會找到這篇[Machine vision with low-cost camera modules](https://blog.arduino.cc/2020/06/24/machine-vision-with-low-cost-camera-modules/)，本來想跟著做一遍就收工，無奈老天給了我更多的考驗，以下就讓我緩緩道來到底遇到什麼問題吧。

## 硬體接線

首先當然要把OV7670和BLE Sense連接起來，通常網路上的範例都是用麵包板和杜邦線連結，為了後續實驗更方便，這裡採用直接焊洞洞板的方式完成，並令攝影機模組板和BLE Sense板呈90度連接，方便更替，大家可自行決定用那種方式連接。完整接線圖如圖31-1所示，或參考Table 33-1。在[Day 27]有提及OV7675的PEN/RST(Pin 19), PWDN/PDN(Pin20)是對應到OV7670的Pin 17, 18，但經查閱相關資料後得知，不接亦可。

Tabel 33-1 OV7670和Arduino Nano 33 BLE Sense接線對照表

| **OV7670 Pin Name** | **OV7670 Pin Number** | **BLE Sense Pin Name** | **備註** |
| :----------: | :----------: | :----------: | :----------: |
|  3.3V | 01 | 3.3V   | |
|  GND  | 02 | GND    | 任一個GND皆可 |
|  SCL  | 03 | SCL/A5 (P0.02) | |
|  SDA  | 04 | SDA/A4 (P0.31) | |
|  VS   | 05 | D8 (P0.21)     | |
|  HS   | 06 | A1 (P0.05)     | |
|  PCLK | 07 | A0 (P0.04)     | |
|  XCLK | 08 | D9 (P0.27)     | |
|  D7   | 09 | D4 (P1.15)     | |
|  D6   | 10 | D8 (P1.14)     | |
|  D5   | 11 | D5 (P1.13)     | |
|  D4   | 12 | D3 (P1.12)     | |
|  D3   | 13 | D2 (P1.11)     | |
|  D2   | 14 | D0/RX (P1.10)  | |
|  D1   | 15 | D1/TX (P1.03)  | |
|  D0   | 16 | D10 (P1.02)    | |
| RESET | 17 | A2 (P0.30)     | 可不接 |
| PWDN  | 18 | A3 (P0.29)     | 可不接 |

![Arduino Nano 33 BLE Sense連接OV7670攝影機模組（無FIFO）之參考線路圖](https://1.bp.blogspot.com/-Yfb51Wm9XnY/YXEiOsleDyI/AAAAAAAAE7k/vBUXWEz4mi8jBCFv6iN0T0WQ5H-JoMVlwCLcBGAsYHQ/s1658/iThome_Day_31_Fig_01.jpg)
Fig. 31-1 Arduino Nano 33 BLE Sense連接OV7670攝影機模組（無FIFO）之參考線路圖。(OmniXRI整理繪製, 2021/10/21)

## Arduino測試程式

由於BLE Sense上並沒有顯示元件（如LCD），所以由OV7670取得的影像必須以二進制(16bit, RGB565）格式數值，經由虛擬串列埠(Virtual COM)傳送到電腦上顯示。為了測試OV7670是否能正確取像，首先要在Arduino IDE安裝必要程式庫。點擊主選單[草稿碼]-[匯入程式庫]-[管理程式庫...]，再輸入「OV7670」，選擇「**Arduino_OV767x**」，按下安裝即可。

接著點擊主選單[檔案]-[範例]，結果發現「Arduino_OV767x」竟然被歸類在**不相容**的函式庫中，正在一頭霧水時，回頭檢查「開發板」設定時，發現原先選用的是「Arduino Mbed OS Nano Boards」下的「Arduino Nano 33 BLE」，而另一個「Arduino Mbed OS Boards」下也有一個同名的「Arduino Nano 33 BLE」，經更換成後者後，「Arduino_OV767x」就正確回到「第三方程式庫的範例」中了。點擊「**CameraCaptureRawBytes**」就可開啟測試OV7670的範例了。完整程序如圖Fig. 31-2所示。

![Arduino IDE OV7670程式庫安裝及開啟範例](https://1.bp.blogspot.com/-tzY9bdBoSN8/YXE11JYLU6I/AAAAAAAAE7s/lDM_NrS_XQs07QCYz2lHbuXHNlwJXeAiwCLcBGAsYHQ/s1658/iThome_Day_31_Fig_02.jpg)
Fig. 31-2 Arduino IDE OV7670程式庫安裝及開啟範例。(OmniXRI整理繪製, 2021/10/21)

```
/*
  OV767X - Camera Capture Raw Bytes 

  點擊Arduino IDE主選單[範例]-[Arduino_OV767x]-[CameraCaptureRawBytes]得到此範例
  
  電路連接:
    - Arduino Nano 33 BLE board
    - OV7670 camera module:
      - 3.3 connected to 3.3
      - GND connected GND
      - SIOC connected to A5
      - SIOD connected to A4
      - VSYNC connected to 8
      - HREF connected to A1
      - PCLK connected to A0
      - XCLK connected to 9
      - D7 connected to 4
      - D6 connected to 6
      - D5 connected to 5
      - D4 connected to 3
      - D3 connected to 2
      - D2 connected to 0 / RX
      - D1 connected to 1 / TX
      - D0 connected to 10
*/

#include <Arduino_OV767X.h> // 導入OV767x程式庫頭文件

int bytesPerFrame; // 宣告一張影像所需Byte數量
byte data[320 * 240 * 2]; // 宣告存放QVGA解析度RGB565格式(16bit)之彩色影像之緩衝區

// 設定腳位用途及模組初始化（只在電源啟動或重置時執行一次）
void setup() {
  Serial.begin(9600); // 宣告虛擬串列埠傳輸速度為9600bps，可自行調整。
  while (!Serial); // 若開啟不成功就一直等待。

  // 初始化攝影機模組為QVGA解析度(320x240)，RGB565彩色格式，取像速度為1秒1張。
  // 解析度可自行調整為VGA(640x480), QVGA(320x240), QQVGA(160x120), CIF(352x240), QCIF(176x144)。
  // 彩色影像格式可自行調整為YUV422, RGB444, RGB565, GRAYSCALE(8bit灰階)。
  // 取像速度可設定為 1, 5, 10, 20FPS, 但由於BLE Sense CPU速度僅有64MHz，所以建議設定為1FPS為佳，以免來不及處理。
  if (!Camera.begin(QVGA, RGB565, 1)) {
    Serial.println("Failed to initialize camera!"); // 若初始化失敗則回傳錯誤訊息
    while (1); // 令程式卡在這一行，表示程式結束。
  }

  bytesPerFrame = Camera.width() * Camera.height() * Camera.bytesPerPixel();  // 依實際初始化結果計算出所需回傳的Byte數量。

  // Optionally, enable the test pattern for testing
  // Camera.testPattern(); // 選擇性輸出，若刪除註解符號，則會一直輸出八色彩條圖，不理會實際攝影機模組拍攝到的內容。可作為測試傳輸用。
}

// 設定無窮循環程式 （會一直依序重覆執行）
void loop() {
  Camera.readFrame(data); // 從攝影機讀取一個影格資料

  Serial.write(data, bytesPerFrame); // 從虛擬串列埠回傳讀到的影像二進制資料
}
```

原則上，以上程式可直接燒錄到BLE Sense中，不用修改。這只是為了測試硬體線路用，不用太在意取像速度只有1 FPS。

## Processing測試程式

剛才有提到，BLE Sense並沒有顯示功能，而透過虛擬串列埠傳送出來的值，也不是電腦顯示用的格式（如BMP），所以需要有另一個程式來解讀，這裡推薦使用[Processing](https://processing.org/)來接收並顯示。Processing是一種開源的程式語言，專門用來創作電子藝術和視覺互動設計。其架構是建立在Java語言之上，其整合開發環境(IDE)操作上很像Arduino IDE。

Processing不需要安裝，只需到[下載頁面](https://processing.org/download)下載適合的作業系統的版本即可，最新的版本為4.0 beta2，亦可選用3.x版穩定版本。下載完成後只需解壓縮，不必安裝，直接執行Processing.exe(Windows版)即可。這裡並沒有要教大家如何寫程式，只需把下列[範例程式](https://raw.githubusercontent.com/arduino-libraries/Arduino_OV767X/master/extras/CameraVisualizerRawBytes/CameraVisualizerRawBytes.pde)複製貼上即可。其主要操作介面如圖Fig. 31-3所示。

其中有幾個小地方要手動修改一下。
* 影像格式：cameraWidth, cameraHeight, cameraBytesPerPixel等變數要符合BLE Sense上傳之格式。
* 顯示視窗：size(window_Width, window_Height)要符合BLE Sense上傳之尺寸。
* 虛擬串列埠：myPort = new Serial(this, "COM3", 9600); 請依不同作業系統、埠號及傳輸速度自行修改，這裡以Windows, COM3, 9600bps為例。

```
/*
  This sketch reads a raw Stream of RGB565 pixels
  from the Serial port and displays the frame on
  the window.

  Use with the Examples -> CameraCaptureRawBytes Arduino sketch.

  This example code is in the public domain.
  範例程式來源：https://raw.githubusercontent.com/arduino-libraries/Arduino_OV767X/master/extras/CameraVisualizerRawBytes/CameraVisualizerRawBytes.pde
*/

import processing.serial.*;
import java.nio.ByteBuffer;
import java.nio.ByteOrder;

Serial myPort; // 宣告一串列埠

// 以下宣告值必須符合BLE Sense上傳的影像大小
final int cameraWidth = 320;       // 攝影機取得影像寬度
final int cameraHeight = 240;      // 攝影機取得影像高度
final int cameraBytesPerPixel = 2; // 影像像素佔用Byte數
final int bytesPerFrame = cameraWidth * cameraHeight * cameraBytesPerPixel; // 影格影像所需Byte數量

PImage myImage; // 宣告一個顯示用影像
byte[] frameBuffer = new byte[bytesPerFrame]; // 宣告影像資料緩衝區

// 設定系統相關參數，只執行一次
void setup()
{
  size(320, 240); // 指定顯示視窗尺寸，需搭配取像尺寸修改

  // if you have only ONE serial port active
  //myPort = new Serial(this, Serial.list()[0], 9600);          // if you have only ONE serial port active

  // 請依不同作業系統、埠號及傳輸速度自行修改，這裡以Windows, COM3, 9600bps為例
  myPort = new Serial(this, "COM3", 9600);                    // Windows
  //myPort = new Serial(this, "/dev/ttyACM0", 9600);          // Linux
  //myPort = new Serial(this, "/dev/cu.usbmodem14401", 9600); // Mac

  // 等待接收到足夠的Byte數量
  myPort.buffer(bytesPerFrame); 

  myImage = createImage(cameraWidth, cameraHeight, RGB); // 創建一張RGB格式影像
}

// 繪圖函式，更新接收到的資料到視窗上
void draw()
{
  image(myImage, 0, 0); // 繪製影像到視窗上
}

// 處理串列埠事件
void serialEvent(Serial myPort) {
  // 從串列埠讀取原始資料到緩衝區
  myPort.readBytes(frameBuffer); 

  // 處理原始資料透過緩衝區
  ByteBuffer bb = ByteBuffer.wrap(frameBuffer); 
  bb.order(ByteOrder.BIG_ENDIAN);

  int i = 0;

  // 當資料還沒讀完
  while (bb.hasRemaining()) {
    // 讀取一個像素資料(16bit, RBG565)
    short p = bb.getShort();

    // 將RGB565格式轉換成RGB888(24bit)格式
    int r = ((p >> 11) & 0x1f) << 3;
    int g = ((p >> 5) & 0x3f) << 2;
    int b = ((p >> 0) & 0x1f) << 3;

    // 將轉換好的像素顏色繪到影像指定位置
    myImage .pixels[i++] = color(r, g, b);
  }
 myImage .updatePixels(); // 更新影像中像素內容
}
```

原本以為貼上程式，按下左上角【Play】鍵就能看到完美影像，沒想到只得到一片黑呼呼的視窗，檢查了老半天，還是找不出問題，最後不得已只好換上另一個OV7670，結果就有一堆奇怪的方塊產生，感覺好像有拍到影像，但卻很破碎。於是把BLE Sense的「Camera.testPattern();」那行註解取消掉來檢查，照道理應該會得到如圖Fig. 31-3右上的彩條圖，但卻得到右下角那個歪斜的影像，感覺上好像是資料每隔一段時間就落後一些時間造成。經上網努力查找，最後得到一個解決方式，就是移到Ubuntu(Linux)下測試就不會有這個問題產生了。果然，在Ubuntu上安裝完Processing並執行同一段程式就OK了。不過第一顆OV7670還是黑畫面，猜想可能已經報銷了。於是把BLE Sense的程式改回，重新把「Camera.testPattern();」加上註解。終於可以在Processing上看到OV7670取到的影像了。但不知為何取得的影像轉了90度，只好先將就點用。

![Processing顯示BLE Sense上傳之資料](https://1.bp.blogspot.com/-cv-BjIgl3r8/YXGIx9WJkuI/AAAAAAAAE70/rDW7xbA5GDIYNb_hGyluOwK-DAHpzzeUgCLcBGAsYHQ/s1660/iThome_Day_31_Fig_03.jpg)
Fig. 31-3 Processing顯示BLE Sense上傳之資料。(OmniXRI整理繪製, 2021/10/21)

## Edge Impulse連結及測試

完成上面測試後，依Edge Impulse官網「[Adding sight to your sensors](https://docs.edgeimpulse.com/docs/image-classification)」說明，應該把Edge Impulse提供的標準韌體(**arduino-nano-33-ble-sense.ino.bin**)使用「**flash_windows.bat**」(Windows版本）重新燒回BLE Sense開發板就可以了。但執行「**edge-impulse-daemon --clean**」將開發板連線後，但進到「資料擷取(Data Acquisition)」頁面後，在「**感測器(Sensor)**」欄位，卻只能看到「Build-in Accelerometer」和「Build-in Micphone」，沒看到攝影機相關選項。

在網路上查找了許久一直沒有答案，結果不小心在某個教學影片中發現，「arduino-nano-33-ble-sense.ino.bin」的日期是2021/9，而我的卻是2021/5的版本。於是死馬當活馬醫，重新下載官網提供的[韌體](https://cdn.edgeimpulse.com/firmware/arduino-nano-33-ble-sense.zip)，再次燒錄並啟動，果然，「**感測器(Sensor)**」欄位多了兩個選項「**Camera (160x120)**」、「**Camera (128x96)**」，且螢幕上也能呈現攝影機即時拍到的內容，雖然Processing在Windows上依舊不正常，但不影響取像結果。當按下【**Start Sampling**】鈕，果然可以拍下影像，並上傳到系統，如此就能建立實拍的資料集了。

![Edge Impulse連接OV7670取像結果](https://1.bp.blogspot.com/-lyhYTCVo5zA/YXGSSWg7H5I/AAAAAAAAE78/lxFrvWosT6gJUgOrGxGCvrWtSIa9Diw7wCLcBGAsYHQ/s1658/iThome_Day_31_Fig_04.jpg)
Fig. 31-4 Edge Impulse連接OV7670取像結果。(OmniXRI整理繪製, 2021/10/21)

## 小結

雖然經歷OV7670損壞一組、Processing在Windows上無法正常顯示BLE Sense上傳影像及Edge Impulse Firmware版本不對等問題，花了一個多星期終於搞定，還好不是在比賽期間，不然為了這個問題無法完賽，就有點可惜了。希望藉由這篇文章可以補齊原來[Day 27]、[Day 28]還沒說完的故事。

參考連結

[[Day 27] Edge Impulse + BLE Sense實現影像分類(上)](https://ithelp.ithome.com.tw/articles/10279267)  
[[Day 28] Edge Impulse + BLE Sense實現影像分類(下)](https://ithelp.ithome.com.tw/articles/10279268)  
[Edge Impulse Document - Tutorials - Adding sight to your sensors](https://docs.edgeimpulse.com/docs/image-classification)  
[Edge Impulse Document - Development Boards - Arduino Nano 33 BLE Sense](https://docs.edgeimpulse.com/docs/arduino-nano-33-ble-sense)  
[Machine vision with low-cost camera modules](https://blog.arduino.cc/2020/06/24/machine-vision-with-low-cost-camera-modules/)  
