## [Day 08] tinyML開胃菜Arduino IDE上桌（上）

書接上回，講了一大堆如何做出好吃的AI咖哩飯，那到底要如何開始才能當上新一代的「AI食神」呢？

首先要到中國廚藝學院（少林寺後廚）打雜洗碗（收集資料）一年，買菜洗菜（清洗資料），端盤子跑堂（依作業流程工作）一年，切菜備料（資料預處理）一年，才能開始拿勺子炒菜（建模調參），學習各種菜式（模型、應用），再讓客人、酸民刷幾年差評（佈署、重訓），大概就能出師了。什麼？不能速成嗎？那我還是去買個AI咖哩調理包（如AWS, Azure, GCP等雲端AI API服務)來裝模作樣一下，三分鐘就可上桌，口味還很穩定（只是不太能變），價錢更是便宜。唉！現在年輕人真是沒有耐性啊！

為了讓大家快一點（還剩22天）學會如何做出一個好吃的爆漿瀨尿蝦牛丸(tinyML)的步驟及開發出自己專屬的口味（客製化應用），接下來會分別介紹外皮牛肉丸（Arm Cortex-M MCU硬體及程式開發）和核心瀨尿蝦（AI原理、算法實現）的作法，希望透過這樣交替式學習能讓大家更快上手。

在開始之前，先來點開胃菜，先了解一下後續tinyML實做所需用到的硬體開發板、開發工具及環境，至於tinyML整合型開發平台需要鋪墊更多基礎知識，就留待下回分解了。

## 【硬體開發板選用】

後續範例會以**Arduino Nano 33 BLE Sense(以下簡稱BLE Sense)**為基準，相關規格可參考[[Day 03]](https://ithelp.ithome.com.tw/articles/10265166)說明。更完整的規格及使用說明可參考[Arduino官網](https://store-usa.arduino.cc/collections/boards/products/arduino-nano-33-ble-sense)介紹。

那為什麼選用這塊開發板呢？有以下幾個原因。
* 主晶片內核是Arm Cortex-M4F（帶浮點運算功能），工作時脈中等，為64MHz。
* 可以運行Arm Mbed作業系統，方便程式跨不同廠牌Arm Cortex-M晶片執行。
* 支援CMSIS, CMSIS-NN函式庫，可免除不同廠牌Arm Cortex-M晶片開發工具不同。
* 程式碼(FLASH) 1M Byte及隨機記憶體區(SRAM) 256K Byte，不會太小，可容納較大的模型及參數。
* 板上內建多種感測器，如溫濕度、氣壓、近接、手勢（平移方向）、色彩（RGB亮度）、麥克風、九軸（慣性、角加速度、地磁各三軸）運動感測器等，方便開發各種應用。如不需這麼多感測器，可改用Arduino Nano 33 IoT或Arduino Nano 33 BLE。
* 體積小，僅有18mm x 45mm，方便製作各種穿戴式應用。
* 為最多tinyML開發平台（如Edge Impulse, SensiML等）和工具(TensorFLow Lite Micro, Arduino IDE等）支援。
* 另外可使用傳統開發工具組合不使用Arm MBed作業系統，如Keil C或IAR配合J-Link等除錯工具，加上Nodic提供的開發板支援套件(Board Support Package, BSP)或Arm CMSIS函式庫來開發。這個路線可能比較適合已經很熟悉Arm Cortex-M MCU程式開發的工程師。為了縮短學習曲線，會採前項tinyML開發平台的方式來進行範例教學。

那裡有在賣呢？可自行到露天、蝦皮上查詢，這裡不特別推薦特定供應商。不過請不要直接用Arduino官網售價來換算，畢竟廠商進口還是有一些成本在的。

咦！這個開發板好像有點貴，有沒有別的板子可支援？當然有，可參考[[Day 03]](https://ithelp.ithome.com.tw/articles/10265166)說明，只是後續的範例會先以這塊板子為主。

## 【軟體開發工具選用】

「工欲善其事，必須利其器」，雖然Arm有很多整合開發環境(Integrated Development Environment, IDE)工具商支援（如IAR, Keil C, MDK等），方便寫程式、編譯、除錯、燒錄等工作一次搞定，但既然這塊板子冠名Arduino，那首選開發環境當然是選**Arduino IDE**囉！簡單好上手，還可和平常Arduino的開發流程完全相容，另外也可以輕易和各式各樣的感測器和週邊連結，方便tinyML輸入和輸出。

**Arduino IDE**目前它有[雲端版](https://create.arduino.cc)和[單機版](https://www.arduino.cc/en/software)（目前穩定版本為1.8.16, 預先評估版為2.0.0-beta），如果只是一般Arduino入門級開發板，則雲端版註冊一個帳號，單機版下載安裝完成，就可以開始使用。不過要使用BLE Sense則會多幾個安裝步驟，以下簡單說明。

### 雲端版IDE

![Arduino雲端版IDE操作畫面](https://1.bp.blogspot.com/-m5J4Ondp7Rs/YUqp3oK_6KI/AAAAAAAAEvc/FFJZpt1jQ3cKbx2u6s4jkBqsPqTv_PbpACLcBGAsYHQ/s2048/iThome_Day_08_Fig_01.jpg)
Fig. 8-1 Arduino雲端版IDE操作畫面。(OmniXRI整理繪製, 2021/9/22)

如上圖Fig. 8-1所示，首先進到[Arduino創作者網站](https://create.arduino.cc)，按下上角的[Login]（步驟1）進入登入畫面。如果你還沒有註冊帳號，可先按[Create One]（步驟2）創建一個，按照畫面指示輸入基本資料就可以產生一個新帳號，或者直接使用你的Google或Apple帳號（步驟2b）登入亦可。若你已創建好帳號，下一次就可直接輸入帳號（電子信箱）和密碼就可登入系統（步驟2a）。登入後，可按右上角的符號，選擇進入網頁編輯器(Web Editor)（步驟3），或者按畫面中間按鈕亦可。

進入網頁編輯器後左半邊為主要選單，包括程式管理區(Sketchbook)、週邊範例碼(Examples)、特殊函式庫(Libraries)、序列監視器(Monitor)、函式使用參考(Reference)、操作輔助說明(Help)、編輯器設定值(Preferences)、MCU資源使用(Features usage)。而右半邊上方可選擇對應的開發板，中間則為程式開發區，下方則為編譯程式相關訊息區。

再來要確定雲端編輯器能順利連接到BLE Sense開發板，取用一條Micro USB的纜線（注意不要拿到只能充電不能通訊的線）連接電腦和開發板，在Windows環境下，正常狀況會聽到電腦叮咚一下，偵測到新的硬體，此時利用「裝置管理器」（以滑鼠移到Windows左下方符號上，再以右鍵點擊）應該會看到「連接埠(COM和LPT）」新增了一個「USB序列裝置(COMx)」（x指埠號，每次有可能不同），表示電腦已經認到開發板的虛擬序列埠。若沒看到新增COM埠，可再重新插拔一下，令開發板重新連接。

接著我們就可點擊雲端編輯器上方的欄位選擇開發板型號，如下圖Fig. 8-2所示，會出現一大堆Arduino的開發板型號，只需輸入BLE，就會篩選到只剩Arduino Nano 33 BLE，這個選項可支援三種板子包括Arduino Nano 33 IoT, BLE, BLE Sense。選好點擊OK後，會發現出現連線失敗的圖案，此時重新刷新一下網頁，就會出現一個黃色的警告訊息，說你忘了開Arduino Agent，按[LEARN MORE]進入後，會請你檢查左下角常駐程式是否有一個Arduino Agent的（白色的Arduino LOGO）符號，若沒有則必須重新安裝。

![Arduino Nano 33 BLE Sense開發板連線失敗及檢查步驟](https://1.bp.blogspot.com/-KMWXnuAbWOM/YUq9nuQAQXI/AAAAAAAAEvo/YcgAXiMrt4sRhjIH_FOylO_8elmEGy_7gCLcBGAsYHQ/s2048/iThome_Day_08_Fig_02.jpg)
Fig. 8-2 Arduino Nano 33 BLE Sense開發板連線失敗及檢查步驟。(OmniXRI整理繪製, 2021/9/22)

接著依畫面上指示，如圖Fig. 8-3所示，依序下載Arduino Agent並雙擊安裝，完成後重新重新回主畫面就會看到連線失敗的符號消失了。若仍失敗，請檢查Agent是否有啟動在常駐區，若無則滑鼠左鍵點擊左下角Windows符號，切到Arduino Create Agent項目下，點擊Arduino Create Agent重新啟動即可。

![安裝Arduino Agent步驟](https://1.bp.blogspot.com/-O7zM0eEjczo/YUq9np6JPNI/AAAAAAAAEvk/M2Vj--Y6kGAMUFuBw1opf-JmoryfQY8EwCLcBGAsYHQ/s1904/iThome_Day_08_Fig_03.jpg)
Fig. 8-3 安裝Arduino Agent步驟。(OmniXRI整理繪製, 2021/9/22)

註：單機版IDE及寫個範例來測試工作環境就留待下篇繼續說明。

參考連結

[Arduino Nano 33 BLE Sense](https://store-usa.arduino.cc/products/arduino-nano-33-ble-sense)
[Getting started with the Arduino Nano 33 BLE Sense](https://www.arduino.cc/en/Guide/NANO33BLESense)
[Arduino 雲端版 IDE](https://create.arduino.cc)
[Arduino 單機版 IDE](https://www.arduino.cc/en/software)
