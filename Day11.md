## [Day 11] 讓tinyML聽見你的呼喚

在先前[[Day 09] tinyML開胃菜Arduino IDE上桌（下）](https://ithelp.ithome.com.tw/articles/10269745)已經簡單介紹過Arduino Nano 33 BLE Sense（以下簡稱BLE Sense）這塊開發板及如何連線及完成「Hello World, 閃爍吧 LED！」的範例程式，接下來就幫大家介紹如何用這塊開發板讓MCU聽到聲音並錄下聲音，而至於如何辨識部份就留待下回分解。

我們都知道人工智慧一個很重要的領域就是聲音，小到語音命令、異常聲音辨識，大到語音轉文字、自然語言處理、自然語言理解及文字轉語音，不過後者動輒數億到數百兆個參數，就算是一般電腦恐怕也不一定能滿足，所以tinyML的重點通常只用放在喚醒詞及聲音片段（含異常聲音）的辨識，常見的應用包括有智能音箱喚醒、電氣設備語音開關、馬達異常音偵測、生理監看（如心音、咳嗽音、呼吸聲）等。

其中喚醒詞(Wake-Up-Word)亦有許多同義詞，如：
* Keyword Spotting 語音關鍵詞檢測
* Keyword Detection 語音關鍵詞偵測
* Spoken Term Detection 口語偵測
* Voice Command 語音命令

要讓電腦聽懂聲音，首先要讓電腦聽到聲音，大家自然會想到加個麥克風(Microphone)不就得了。其實這中間沒這麼簡單，透過Fig. 11-1所示，大概就能知道基本流程，包括收音（電容式或微機電式麥克風）、信號放大、類比數位轉換（取樣、量化）及傳輸（儲存）等工作。

所謂聲音是物體高速振動經由空氣傳播到耳朵所產生的，但人耳通常只能聽到20 ~ 20K Hz(每秒振動次數），年紀大的人可能超過15K Hz就聽不太到了。聲音包含頻率（低沈或尖銳）和振幅（音量大小），所以首先要將這些微小的振動忠實的收集起來，再轉換成電壓並放大，而這就是傳統麥克風在作的工作。

接著將放大後的電壓數位化（Analog Digital Convert, ADC 類比數位轉換），而轉換時又要考慮量化解析度（位元數）及取樣頻率。前者代表對音量的解析能力，比方8 bit就表示最大音量變化（從最小到最大，可以有正負或非對稱）可分解成2的8次方，即256階，而16 bit就能分解成65,536階。後者則是對聲音變化的取樣速度，通常取樣要超過待測頻率的3到5倍，以免產生偽信號，所以像CD音質的取樣頻率就定為44.1K Hz，而DVD更高達96K Hz。

因此取樣頻率及分解能力越高越能接近原始聲音訊號，但帶來的問題就是需要更大量的儲存空間，以44.1 KHz, 16 bit (2 Byte)為例，一秒鐘就要 44.1K x 2 Byte = 88.2 KByte。一般MCU的SRAM通常不大的情況下，就要重新考慮夠用就好，不用一昧追高，只要足夠後續分辨即可。

通常ADC後可直接傳到電腦或MCU上儲存，但有些微機電（MEMS）型麥克風模組，直接整合了麥克風、放大器、類比數位轉換，最後再經由I2S(SPI)等通訊格式將已數位化的聲音傳輸到MCU中，可省去佔用MCU的ADC輸入端點，同時確保不會因線路傳輸干擾而得到過多雜訊。

![聲音取樣數位化流程圖](https://1.bp.blogspot.com/-2wrVJeVk_uY/YU8vJc7rNAI/AAAAAAAAEw0/2n_4yFo3LN40-Zz3KrbTLQt7NbcEbfjyQCLcBGAsYHQ/s1658/iThome_Day_11_Fig_01.jpg)
Fig. 11-1 聲音取樣數位化流程圖。(OmniXRI整理繪製, 2021/9/24)

如Fig. 11-2所示，目前BLE Sense開發板上就有提供**MP34DT05-A**這顆無向性微機電麥克風模組，且已使用I2S(SPI)方式連接到MCU上，其內部接線如圖所示，更完整的開發板電路圖可參考文末參考連結[BLE Sense電路圖]。其中LR(P0.17)信號線是方便系統上有接兩顆模組時切換左右聲道用的，但通常作為喚醒詞或異常聲音片段偵測時僅需單聲道即可。這裡和電源VDD和LR接在一起，是為了節省接腳和省電二個用途，當P0.17 低電位時，模組不耗電也不輸出。而輸出DOUT(P0.25)採脈波密度調變(Pulse Desity Modulation, PDM)方式輸出，若LR信號為高電位時，會在CLK(P0.26)高電位時將結果由DOUT送出，而低電位時DOUT則為不輸出（Hi-Z 高阻抗）。至於PDM更進一步說明，可參考文末參考連結[AN2057應用筆記]

**MP34DT05-A**其主要規格如下所示，更完整的規格書(Datasheet)可參考文末參考連結[MP34DT05-A 資料手冊]。
* Single supply voltage
* Low power consumption
* AOP = 122.5 dBSPL
* 64 dB signal-to-noise ratio
* Omnidirectional sensitivity
* –26 dBFS ±3 dB sensitivity
* PDM output
* HCLGA package
* Input clock frequency 1.2M ~ 3.25M Hz

![Arduino Nano 33 BLE Sense 微機電式麥克風模組MP34DT05-A配置及接線定義](https://1.bp.blogspot.com/-XVTjBFIXcVE/YU8vJGXHwUI/AAAAAAAAEww/jDx92y3MUPsldTm-9vtOqlcCfEhcPk-pwCLcBGAsYHQ/s1663/iThome_Day_11_Fig_02.jpg)
Fig. 11-2 Arduino Nano 33 BLE Sense 微機電式麥克風模組MP34DT05-A配置及接線定義。(OmniXRI整理繪製, 2021/9/24)

接著就以Arduino IDE來介紹如何從麥克風讀取聲音大小的數值到MCU並回傳讀到序列監視器。這裡Arduino已幫我們準備好現成的範例，只需點擊主選單的[檔案] ─ [範例] ─ [Arduino Nano 33 BLE的範例] ─ [PDM] ─ [PDMSerialPlotter]，就會自動產生一範例程式，如下所示。其中主要包含三大區。
* setup() 主要用於設定腳位用途及模組初始化，而這段程式只在電源啟動或重置時執行一次。
* loop() 負責執行無窮循環程式，這個部份會一直依序重覆執行。主要動作為傳送即時聲音數值到序列監視器及繪圖器上。
* onPDMdata() 為PDM回調函數，負責將讀取的聲音聲音大小數值寫入緩衝區中。

完整範程式可參考下列程式碼及註解。

```
/*
  PDMSerialPlotter.ino
  由Arduino IDE 主選單的檔案 ─ 範例 ─ Arduino Nano 33 BLE的範例 ─ PDM ─ PDMSerialPlotter 取得
  
  主要功能為從MP34DT05-A微機電型麥克風模組透過I2S讀取聲音訊號，並透過IDE內建序列繪圖工具將波形繪出。
  註：序列繪圖器尚未加入Arduino IDE 2.0 Beta中
  
  本範例支援
  - Arduino Nano 33 BLE board, or
  - Arduino Nano RP2040 Connect, or
  - Arduino Portenta H7 board plus Portenta Vision Shield
*/

#include <PDM.h> // 導入PDM函式庫

static const char channels = 1; // 設定為單音通道，通道數為1
static const int frequency = 16000; // 設定PCM輸出頻率為16KHz
short sampleBuffer[512]; // 取樣緩衝區，每筆資料為Short(16 bit)
volatile int samplesRead; // 已讀取樣本數量

// 設定腳位用途及模組初始化（只在電源啟動或重置時執行一次）
void setup() {
  Serial.begin(9600); // 設定序列通訊速度為9600 bps
  while (!Serial); // 若序列埠未能成功開啟則一直等待

  PDM.onReceive(onPDMdata); // 指定PDM接收回調函數

  // PDM.setGain(30); // 選擇性設定PDM增益值，BLE Sense預設值為20
 
  if (!PDM.begin(channels, frequency)) { // 初始化PDM參數，單通道收音，16KHz取樣頻率。
    Serial.println("Failed to start PDM!"); // 若PDM初始化失敗則顯示字串
    while (1); // 永久等待
  }
}

// 設定無窮循環程式 （會一直依序重覆執行）
void loop() {
  // Wait for samples to be read
  if (samplesRead) { // 若讀取樣本數不為零

    // 根據樣本數列印樣本數值到序列監視器和繪圖器
    for (int i = 0; i < samplesRead; i++) {
      if(channels == 2) { // 若設為左右聲道，通道數為2
        Serial.print("L:"); // 列印「左」提示字串到序列監視器
        Serial.print(sampleBuffer[i]); // 列印取樣數值到到序列監視器
        Serial.print(" R:"); // 列印「右」提示字串到序列監視器
        i++; // 計數值加1
      }
      Serial.println(sampleBuffer[i]); // 列印取樣數值到到序列監視器
    }
    
    samplesRead = 0; // 清除已讀樣本計數器
  }
}

/*
PDM接收回調函數，負責處理從麥克風送回的PDM格式資料
這個函式是透過中斷服務函式(ISR)完成的，但不支援Serial列印訊息功能。
*/
void onPDMdata() {
  int bytesAvailable = PDM.available(); // 查詢可用的資料大小

  PDM.read(sampleBuffer, bytesAvailable); // 讀取可用的資料到緩衝區
  samplesRead = bytesAvailable / 2; // 由於讀入的資料為16bit格式，所以已讀樣本數量要除2
}
```

最後讀取到的聲音數值變化如Fig. 11-3所示。目前這個程式是連續一直讀，沒有停止，有興趣的朋友可自行加點料，改成看到某個輸入（觸發）信號，再收音1秒或擷取一段指定長度的資料，這樣就會更接近tinyML雲端一站式開發平台在做的事。

![Arduino IDE 序列繪圖器即時顯示麥克風聲音變化圖](https://1.bp.blogspot.com/-t2L-NR9BRJM/YU9g5pp_IDI/AAAAAAAAExA/e5k7tGXoHFUs46DfgdTFY_SnwzYYkUU5ACLcBGAsYHQ/s1665/iThome_Day_11_Fig_03.jpg)
Fig. 11-3 Arduino IDE 序列繪圖器即時顯示麥克風聲音變化圖。(OmniXRI整理繪製,2021/9/24)

參考連結

[Arduino Nano 33 BLE Sense Schematics電路圖](https://content.arduino.cc/assets/NANO33BLE_V2.0_sch.pdf)  
[AN5027 应用笔记 ─ 使用STM32 32位Arm® Cortex® MCU连接PDM数字麦克风（簡體版）](https://www.st.com/resource/zh/application_note/dm00380469-interfacing-pdm-digital-microphones-using-stm32-mcus-and-mpus-stmicroelectronics.pdf)  
[MEMS audio sensor omnidirectional digital microphone MP34DT05-A Datasheet 資料手冊](https://content.arduino.cc/assets/Nano_BLE_Sense_mp34dt05-a.pdf)  
