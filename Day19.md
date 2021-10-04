## [Day 19] tinyML開發好幫手─雲端一站式平台Edge Impulse簡介

自從上次在街邊吃了一碗不怎麼地「雜碎麵」，咖哩魚蛋沒魚味，咖哩又不入味，失敗！豬皮煮得太爛，沒咬頭，失敗！豬血鬆垮垮，一夾就散，失敗！蘿蔔沒挑過都是筋，失敗中的失敗！最離譜的就是這些大腸，完全沒洗乾淨，還有塊屎，有沒有搞錯呀？讓我深刻地體悟到這和tinyML的開發有異曲同工之妙，資料標註資料不足標註又不確實，失敗！模型設計得太簡單，沒準度，失敗！模型訓練的太擬合，一測試精準度就掉，失敗！模型沒優化都是冗餘參數，失敗中的失敗！更離譜的就是佈署，完全沒考慮MCU的記憶體空間，還會爆掉，有沒有搞錯呀？所以難道要像唐牛一樣到中國廚藝學院進修個十年才能煮出一碗好吃的「雜碎麵」嗎？難道tinyML的開發就不能一次到位、讓大家輕鬆上手嗎？

## tinyML專案開發步驟

經過[[Day 16]](https://ithelp.ithome.com.tw/articles/10274632)和[[Day 17]](https://ithelp.ithome.com.tw/articles/10275641)「TFLM + BLE Sense + MP34DT05 就成了迷你智慧音箱」的說明後，大家大概可以理解一個簡單的喚醒詞（語音命令）辨識就要包含下列步驟。
1. 原始資料收集及分割，必要時還要加上擴增（如背景雜訊）手段。這部份需另以影音軟體大量人工剪裁，
2. 個別樣本的前處理，含聲音雜訊濾除、時域轉頻域甚至是頻譜。這部份可依需求自行撰寫程式處理。
3. 以特定框架（如TensorFlow)建立合適模型，包含模型層數、寬度、卷積、池化、全連結等各項參數。
4. 開始訓練，並透過輸出損失(Loss)函數及平均絕對誤差(MAE)的變化來觀察訓練結果的好壞及作為調整超參數的依據。
5. 確定訓練結果已達初步滿意後，即可進行優化（含量化）處理，讓模型能儘量縮小但又不失推論準確度。
6. 接著再經TensorFlow Lite(TFLite), TFLite Microcontroller(TFLM)將模型及參數轉換成MCU可用的C語言陣列。
7. 最後佈署到MCU上，並依推論結果來推動指定的LED燈號或週邊元件工作。

所以光是這樣一個小小的應用，開發者不僅要搞懂AI開發及轉換程序，又要理解MCU硬體相關控制及如何串接AI端完成的模型，更慘的是還要搞懂信號預處理的領域知識，相信可能很多剛入門的朋友看完後都想退番，不想追下去了。不過幸運地是，有很多廠商注意到這個整合（麻煩）問題，於是紛紛推出雲端一站式平台來開發tinyML的應用，如[[Day 10] tinyML整合開發平台介紹](https://ithelp.ithome.com.tw/articles/10269746)幫大家介紹過的內容。

## Edge Impulse簡介

在這麼多平台中，其中**Edge Impulse**算是比較知名且較多人使用的tinyML整合開發系統（如圖Fig. 19-1所示），有超過一千家以上企業及超過二萬個專案在其系統上開發，可支援的開發板及MCU供應商也有數十家。這家公司於2019年創立，主要是希望大家能在五分鐘內就能創造出像智慧感測器、聲音、視覺等tinyML應用在各種MCU上執行，透過雲端服務更方便分享、管理。Edge Impulse主要提供了一個雲端操作介紹，只要使用網頁瀏覽器就能完成四大步驟及循環，包括原始資料收集、分割產生**資料集(Dataset)**，模型建置和訓練完成**沖擊(Impulse)**，經過調參即可完成**測試(Tests)**，最後再將所有內容打包佈署到**邊緣裝置(Edge Device)**，如果不滿意推論準確度結果可再重新循環這些步驟。

![Edge Impulse雲端一站式tinyML開發平台操作介面、主要工作項目及硬體支援廠商](https://1.bp.blogspot.com/-WHjemjpV0jY/YVp9fnDDwcI/AAAAAAAAE1M/aFIwGbW8lhE8oOI_ru_z4lQwS2BpDg-MgCLcBGAsYHQ/s1658/iThome_Day_19_Fig_01.jpg)
Fig. 19-1 Edge Impulse雲端一站式tinyML開發平台操作介面、主要工作項目及硬體支援廠商。(OmniXRI整理繪製, 2021/10/4)

## 安裝開發工具

所以接下來同樣會使用**Arduino Nano 33 BLE Sense（以下簡稱BLE Sense）**為工作開發板，利用板上的辨識「Yes」、「No」的例子來重新說明一次，讓大家感受一下其便捷性。不過在使用前有些預備工作要處理。

1. **申請使用帳號**：（如圖Fig. 19-2所示）
    1. 到[Edge Impulse官網](https://www.edgeimpulse.com/)，點擊右上角【Sign Up】或者於左側輸入始名、電子郵件信箱再按【Sign up for free】即可進到下個畫面。 
    2. 輸入個人基本資料，勾選同意隱私權規範，點擊【Sign Up】創立新帳號。
    3. 到電子信箱收啟用驗證信，通常幾分鐘內會收到，點擊連結後就可啟動帳號。
    4. 如果沒收到驗證信，可再要求重寄，若已啟用則可點擊【Click here to build your first ML model】創建新專案。
    5. 進入後會詢問要開啟一種專案類型，可依指示點擊建立專案，若還沒確定亦可點擊右上角【X】符號直接進入專案。
    6. 下次進入Edge Impulse官網，點擊右上角【Login】，輸入姓名、電子信箱即可登入專案管理畫面。

![Edge Impulse申請帳號流程](https://1.bp.blogspot.com/-FQIKEEAfs1g/YVq3SCMn3uI/AAAAAAAAE1U/_P71YeFMbfIOTVwRpzFqYWE8GiO-ZNWiACLcBGAsYHQ/s1658/iThome_Day_19_Fig_02.jpg)
Fig. 19-2 Edge Impulse申請帳號流程。(OmniXRI整理繪製, 2021/10/4)

2. **安裝CLI套件**
    1. 選擇對應開發板說明文件：目前Edge Impulse支援很多種開發板，每一種板子都有其對應工具套件包安裝方式，如此才能讓網頁瀏覽器和實體板子用USB的虛擬序列埠(Virtual COM)連接在一起。這裡要進到[BLE Sense說明文件](https://docs.edgeimpulse.com/docs/arduino-nano-33-ble-sense)。
    2. 安裝Edge Impulse CLI（工作列介面）套件：這裡僅說明Windows安裝方式，其它作業系統安裝方式請參考[CLI安裝(Installation)說明頁面](https://docs.edgeimpulse.com/docs/cli-installation)。首先安裝Python3，再安裝Node.js(v14或更高版本)，最後用npm安裝CLI，如下指令所示。**（貼心提醒，請先申請開通好Edge Impulse帳號，以免安裝部份套件包時被擋下）**
    ```
    npm install -g edge-impulse-cli --force
    ```
    3. 安裝Arduino CLI（工作列介面）套件：這裡僅說明Windows安裝方式，其它作業系統安裝方式請參考[CLI安裝(Installation)說明頁面](https://arduino.github.io/arduino-cli/0.19/installation/)。直接從CLI安裝說明頁面中找到[Windows aduino-cli.zip 64bit](https://downloads.arduino.cc/arduino-cli/arduino-cli_latest_Windows_64bit.zip)並下載（[其它版本](https://arduino.github.io/arduino-cli/0.19/installation/)），再來解壓執行，一路按下一步或OK到完成即可安裝好。

3. **連接BLE Sense開發板**：使用USB纜線（注意不要拿到只有充電功能的線）連接電腦和開發板。
4. **更新開發板韌體**
    1. 下載最[新版本的Edge Impulse韌體](https://cdn.edgeimpulse.com/firmware/arduino-nano-33-ble-sense.zip)，解壓縮後備用。
    2. 使用Windows時，進到解壓縮後的路徑，找到**flash.windows.bat**並執行，把最新韌體燒到開發板MCU中。
    3. 當燒錄完成後(如圖Fig. 19-3)，按一下板子上RESET鍵（白色按鈕）重置系統，就能重新啟動最新的韌體了。
    ![Arduino Nano 33 BLE Sense更新完Edge Impulse最新韌體後結果](https://1.bp.blogspot.com/-8bxsrXRtOD0/YVrZXc5c1pI/AAAAAAAAE1g/aiT99N9SdooYwXlNoDBA-BfMxDGHi288wCLcBGAsYHQ/s1658/iThome_Day_19_Fig_03.jpg)
    Fig. 19-3 Arduino Nano 33 BLE Sense更新完Edge Impulse最新韌體後結果。(OmniXRI整理繪製, 2021/10/4
5. **設定金鑰(Keys)**：在Windows命令列視窗執行下列命令，過程中要確定網路有連線，因為會登入到Edge Impulse網站中，此時會要求輸入使用者姓名、密碼及板子的名字（隨便取無妨）。其結果如圖Fig. 19-4所示。啟動後這個視窗要一直開著（可以最小化），直到結束Edge Impulse網頁操作為止。
    ```
    edge-impulse-daemon
    ```
    ![啟動Edge Impulse後台程式以連接網頁瀏覽器](https://1.bp.blogspot.com/-WQ6HYZF2Wu8/YVrZXf8hFtI/AAAAAAAAE1c/CPS8Ds9yvWQSbBpqG1zskxM4-Zum3BW9QCLcBGAsYHQ/s1658/iThome_Day_19_Fig_04.jpg)
    Fig. 19-4 啟動Edge Impulse後台程式以連接網頁瀏覽器。(OmniXRI整理繪製, 2021/10/4)
6. **驗證開發板連接狀態**：
    接著就進到Edge Impulse專案頁面，點擊左側Device項目，此時會看到BLE Sense顯示在畫面上，但要注意的是燈號，綠燈表連線OK，紅燈表失敗。如圖Fig. 19-5所示。
    ![執行edge-impulse-daemon開發板連線狀況](https://1.bp.blogspot.com/-xWvht-FutY0/YVrdueSub5I/AAAAAAAAE1s/AEVr94-QnEkkTE2_znufCOaDYQWETEWowCLcBGAsYHQ/s1658/iThome_Day_19_Fig_05.jpg)
    Fig. 19-5 執行edge-impulse-daemon開發板連線狀況。(OmniXRI整理繪製, 2021/10/4)

到了這裡，開發環境大致OK，下個章節就開始在實際演練「關鍵字語音辨識」。

ps. 為讓文章更活潑傳達硬梆梆的技術內容，所以引用了經典電影「食神」的橋段，希望小弟戲劇性的二創不會引起電影公司的不悅，在此對星爺及電影公司致上崇高的敬意，敬請見諒。

參考連結

[Edge Impulse官網](https://www.edgeimpulse.com/)  
[Edge Impulse Documentation - Getting Started 說明文件](https://docs.edgeimpulse.com/docs)  
[Edge Impulse Arduino Nano 33 BLE Sense安裝程序](https://docs.edgeimpulse.com/docs/arduino-nano-33-ble-sense)  
[Edge Impulse firmware for Arduino Nano 33 BLE Sense開源韌體](https://github.com/edgeimpulse/firmware-arduino-nano-33-ble-sense)  
