## [Day 25] Edge Impulse + BLE Sense實現手勢動作辨識(下)

=== 書接上回 [[Day 24] Edge Impulse + BLE Sense實現手勢動作辨識(上)](https://ithelp.ithome.com.tw/articles/10279264) ===

## 模型設計及訓練

接著就可以進到「**Impulse Design**」頁面，開始調整時間序列參數、新增處理和學習區塊並存檔。在「**時間序列**」部份，可以調整視窗大小（預設值2000ms）、視窗遞增距距離（預設值80ms）和取樣頻率（預設值16KHz），通常可使用預設值即可。接著新增一個「**處理區塊(Processing Block)**」，選擇系統推薦的「**頻譜分析(Spectral Analysis)**」即可，再來新增一個「**學習區塊(Learning Block)**」，選擇系統推薦的「**分類Classification(Keras)**」即可。另一個「異常偵測(Anomaly Detection)」區塊並不是用來偵測時序型訊號異常用的，而是用來去除過於離群的資料，所以暫時不要選。最後按下【**Save Impulse**】，左側選單Impulse design下方就會新增「**頻譜特徵(Spectral Features)**」和「**神經網路分類器(NN Classifier)**」新頁面。

![Edge Impulse 模型設計流程（手勢辨識）](https://1.bp.blogspot.com/-zwBR0cplDBU/YWGDlNRIZkI/AAAAAAAAE40/06sB8wEkK2UqLqkTX1Hjg73wJgUxnlW8QCLcBGAsYHQ/s1658/iThome_Day_25_Fig_01.jpg)
Fig. 25-1 Edge Impulse 模型設計流程（手勢辨識）。(OmniXRI整理製作, 2021/10/9)  

### 頻譜特徵

再來切換到「「**頻譜特徵(Spectral Features)**」」頁面，第一子頁為「**參數（Parameters)**」，可以在右上角下拉式選單中切換想觀看的資料，頁面就會顯示對應的原始三軸信號波形，若沒有特殊需求可先略過調整頻譜相關參數。但從頁面右半邊顯示的「DSP Result」，可看出和聲音的處理方式略有不同，它更需要使用濾波功能來使信號更平順，同時去除更多高頻雜訊（手的輕微抖動）。最下方還有模型佈署到開發板後的預估記憶體使用量及推論計算時間。

接著要切換到第二子頁「**產生特徵(Generate Features)**」，按下綠色按鈕【**Generate Features**】開始計算所有訓練樣本的特徵值，並以立體空間可視化結果來呈現資料分佈狀態。如果各分類自己本身（相同顏色點）越群聚，而不同分類越分開，表示資料很容易分割，可以得到較好的分類效果，反之則要重新處理資料集收集及建置問題。完整操作內容可參考圖Fig. 25-2所示。

![Edge Impulse 頻譜特徵產生（手勢辨識）](https://1.bp.blogspot.com/-_eoU-JkhUag/YWIiVPgTSNI/AAAAAAAAE48/-MzliiuBesc1qoIcxl4A4nfkZNoTr9gwwCLcBGAsYHQ/s1658/iThome_Day_25_Fig_02.jpg)
Fig. 25-2 Edge Impulse 頻譜特徵產生（手勢辨識）。(OmniXRI, 2021/10/9)  

### 神經網路分類器

再來就是重頭戲，切換到「**神經網路分類器(NN Classifier）**」頁面，這裡是設計模型結構及進行訓練用的。首先可以指定訓練次數和學習率，一開始次數可以少設一些，加快訓練時間，如果訓練後精確度不高，混淆矩陣表現不佳，那就可以增加次數或改變模型結構來改善。再來是模型架構設定，這裡已預設兩個全連結(Dense Layer)隱藏層，分別有20個和10個神經元，如果神經元數量或隱藏層數不足以產生良好分類結果，那可以適當的調整一下。另外如同前面章節提到，如果你精通Python、TensorFlow和Keras，那右上角的按鍵可切換至專家模式(Expert)，你可以直接修改程式碼來改變模型結構。同樣地，當按下【**Start Training**】後就能得到精確度、損失值、特徵分佈圖及佈署到裝置後的資源耗損。更完整的內容可參考圖Fig. 25-3所示。

另外這個訓練的步驟很重要，如果不滿意結果就要一直調整，直到可以接受為止。由於目前僅有很少的資料樣本，所以訓練後得到100%的精準度是很正常的（因為已經過擬合了），但如果你的訓練集很大，但卻得到很高的精確度和很完美的混淆矩陣，那可能就要注意已產生過擬合現象了。要適當調整網路結構或再增加資料集多樣性來改善。

![Edge Impulse 神經網路分類（手勢辨識）](https://1.bp.blogspot.com/-srW9j-tj8bw/YWIiVLAeV-I/AAAAAAAAE5A/UCS4ToYsOaknjd__rLlyNZsQW988AnV4ACLcBGAsYHQ/s1658/iThome_Day_25_Fig_03.jpg)
Fig. 25-3 Edge Impulse 神經網路分類（手勢辨識）。(OmniXRI, 2021/10/9)  

完成訓練後，可於左側選單切換至「**模型測試(Model Testing)**」來檢驗測試集的表現，亦可切換至「**即時分類(Live Classifictaion)**」來輸入現錄的動作看看是否能順利分類。這兩個步驟和[[Day 22] Edge Impulse + BLE Sense實現喚醒詞辨識(下)](https://ithelp.ithome.com.tw/articles/10278171)類似，使用系統提供的預設值即可，這裡就不再多作說明。

## 模型佈署及推論

最後是要產生可以佈署到開發板的程式。同樣地不想寫程式的人，點擊「**建置燒錄檔案(Build Firmware)**」項目 下的「**Arduino Nano 33 BLE Sense**」，按下【**Build**】後即可得到ZIP檔案，解壓縮後會得到二進制檔（MCU韌體）firmware.ino.bin和燒錄程序檔**flash_windows.bat**（windows用），執行後者就能把程式燒進開發板中。執行結果會從USB虛擬序列埠輸入，只需使用類似「Windows Terminal」（可於Win10 APP Store下載）或PuTTY中的Serial來監看輸出結果，但要注意選對COM埠。

如果自己後續想要加入一些自己的控制程式，需要取得源碼，那就得選用「**產生函式庫(Create Library)**」項目下的「**Arduino**」，再按下【**Build**】後即可得到ZIP檔案。同樣地，如同前面章節所示，可於頁面下方選擇性的指定是否使用EON Compiler優化編譯器及是否量化(INT8 / Float32)，來縮小模型的複雜度和所需使用的記憶體(SRAM, Flash)儲存空間。

接著開啟Arduino IDE，點擊主選單[草稿碼]-[匯入程式庫]-[加入.ZIP程式庫...]，選擇剛才產出的ei-cmr_test-arduino-1.0.1.zip（依個人專案名稱產生）。再來點擊主選單[檔案]-[範例]-[CMR_Test_Inferencing(依專案名稱不同）]-[nano_ble33_sense_acceleratometer_continuous]就能產生對應的範例程式。在編譯及上傳前，記得把edge-impulse-daemon的命令列視窗關閉，以免佔住虛擬序列埠，導致無法上傳。按下左上角箭頭符號就可開始編譯及上傳，這時候可以先去泡杯咖啡，慢慢看著進度條，因為第一次編譯要花比較長的時間。上傳完成程式就會自動運行起來，此時點擊主選單[工具]-[序列埠監控視窗]就可看到開發板一直回送結果，戴上開發板，比出預定手勢就能驗證是否成功運行。完整的步驟如圖Fig. 25-4所示。

![Edge Impulse模型經Arduino編譯上傳結果](https://1.bp.blogspot.com/-ddlwWf48nzY/YWI-_EjSwFI/AAAAAAAAE5U/ZC7mwza0-hw3Pz6Sci2_rRtZrRwQIELNQCLcBGAsYHQ/s1660/iThome_Day_25_Fig_04.jpg)
Fig. 25-4 Edge Impulse模型經Arduino編譯上傳結果。(OmniXRI整理繪製, 2021/10/9)  

## 小結

相信大家有了前面聲音（喚醒詞）辨識操作經驗後，這次在操作手勢辨識應該非常容易，如果資料集少建一些，那整個專案大概10~15分鐘就完成了，實在非常方便，幾乎都不用寫到程式，對於自定義tinyML專案的開發是很有幫助的。對於想更深入了解的朋友，亦可從Arduino的源碼中學到更多的技術原理。

參考連結

[Edge Impulse Document - Tutorials - Continuous motion recognition說明文件](https://docs.edgeimpulse.com/docs/continuous-motion-recognition)  
[[Day 20] Edge Impulse + BLE Sense實現喚醒詞辨識(上)](https://ithelp.ithome.com.tw/articles/10277682)  
[[Day 21] Edge Impulse + BLE Sense實現喚醒詞辨識(中)](https://ithelp.ithome.com.tw/articles/10277700)  
[[Day 22] Edge Impulse + BLE Sense實現喚醒詞辨識(下)](https://ithelp.ithome.com.tw/articles/10278171)  
