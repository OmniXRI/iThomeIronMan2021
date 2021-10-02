## [Day 12] tinyML開發框架(一)：TensorFlow Lite Micro初體驗(上)

說到tinyML不得不說起「TinyML Machine Learning with TensorFlow Lite on Arduino and Ultra-Low-Power Microcontrollers」一書的作者Pete Warden，他應該是最早將這個概念整理成書的作者(2020/1/7 初版)，並且在Youtube上也有多段影音教學影片（如文末參考連結【TinyML Playlist】）。目前這本書也有中文譯本在台發售，碁峯於2020/7/31出版，書名「tinyML TensorFlow Lite機器學習 應用Arduino與低耗電微控制器」，如Fig. 12-1所示。

![TinyML經典入門書及作者Pete Warden](https://1.bp.blogspot.com/-h4rxcATnPWY/YVCxbltbgTI/AAAAAAAAExI/cx8jwEBdmswyQQ8ZyN_jVMR8ESAJGjWogCLcBGAsYHQ/s1658/iThome_Day_12_Fig_01.jpg)
Fig. 12-1 TinyML經典入門書及作者Pete Warden。(OmniXRI整理繪製, 2021/9/26)

> 「TinyML」中文版書中對兩位作者的介紹：
> Pete Warden 是行動及嵌入式TensorFlow的技術主管，也是TensorFlow團隊的創始成員之一。他曾經是Jetpac的CTO和創始人，該公司在2014年被Google收購。
> Daniel Situnayake 是Google的首席開發布道師，並且協助運作tinyML聚會小組。他也是Tiny Farms的共同創辦人，Tiny Farms是美國第一家大規模自動生產昆蟲蛋白的公司。

相信可能很多人第一次接觸tinyML應該都是跟著這本書來實作和練習，但說實在的，步驟有點多，有很多開發環境要設置，所以對於新手會有些吃力，因為除了AI部份外，還有MCU部份要處理。好在AI的世界進步速度頗快，才過了一年多，就有了不少改進，簡化了不少。接下來就讓我們跟著Google官網「[TensorFlow Lite for Microcontrollers](https://www.tensorflow.org/lite/microcontrollers?hl=zh-tw)」(以下簡稱TFLM）的說明來實際操練一下。

什麼是TFLM呢？一般大家在開發深度學習模型時多半會使用Google TensorFlow框架，但到了手機或單板微電腦（如Arm Cortex-A, Cortex-R或樹莓派Cortex-A53/A57等）時代，這樣的框架太大塞不進這些系統中，於是Google又推出TensorFlow Lite，它不僅更短小精悍，同時還提供的模型優化工作，使得在推論準確率只損失數個百分點甚至幾乎沒有損失的情況下，讓模型縮小十倍以上，不僅更容易塞進這些開發板，同時加快了推論速度。不幸地是，當遇到像MCU(Cortex-M)等級的開發板時，又遇到塞不進的問題，所以Google才會再推出TensorFlow Lite for Microcontroller。表面上名字很像Lite，但實質上為了牽就MCU的開發架構，因此本質上有很大不同，無法直接取代。原則上這些都是開源碼，有興趣研究原始碼的朋友可參考文末參考連結【TFLM Github開源碼】。另外目前TFLM可支援的開發板可參考[[Day 03] tinyML開發板介紹](https://ithelp.ithome.com.tw/articles/10265166)的說明。

接下來就跟著Google TFLM的「[Hello World](https://www.tensorflow.org/lite/microcontrollers/get_started_low_level?hl=zh-tw)」來建立基本操作觀念。這個範例主要分成兩個部份，如下所示。
* 訓練模型：使用Python，在Google Colab (jupyter notebook)完成訓練、轉換及優化模型。
* 執行推論：使用C++ 11，在開發板上使用C++函式庫執行模型推論。

### 訓練模型(Python + Google Colab + GPU)

首先點擊Google提供的[Colab範例程式](https://colab.research.google.com/github/tensorflow/tflite-micro/blob/main/tensorflow/lite/micro/examples/hello_world/train/train_hello_world_model.ipynb?hl=zh-tw)，免下載，可直接運行，Jupyter Notebook操作環境，說明文字和程式一起存在，方便學習，只需在每個程式欄位左上角按下黑色箭頭（或者點想執行的欄位按Ctrl + Enter亦可）即可單步執行，但切記要按照順序把每個步驟都執行完，不能跳過任何一步驟。由於開啟後會看到先前運行結果都被保留在執行結果欄位，為了更清楚看到所有動作，可執行主選單的[編輯]─[清除所有輸出欄位]，將所有輸出欄位清除。

首先說明這個「Hello World」程式主要想展示如何將一個TensorFlow建立好的模型轉換到TFLM，為了方便說明，並沒有使用現成常用的資料集，而是以正弦波加亂數方式產生一個資料集，然後訓練出一個模型（正弦波函數），使得輸入X位置就能推論出Y位置。

接下來就快速摘要一下整個程式在做什麼？程式部份請參考原始程式，這裡僅作重點摘要及補充說明，方便大家更容易理解。程式運作後產生的相關圖表可參考Fig. 12-2。

* 配置模型工作路徑名稱。
* 安裝TensorFlow 2.4.0版環境，並導入Python所需套件。
* 產生資料
  * 產生正弦波基本數據，在0~2 PI間產生1000點當作x，再將其順序打亂，並依正弦(Sine)函式計算出對應的y值，得到1000組原始資料，(x,y)。
  * 增加雜訊，為了使資料更接近真實世界隨意變動，所以將原始資料的y值隨機加入10%的變動。
  * 分割資料集，為了後續訓練、驗證及測試模型的效能，所以將資料集分割成60%, 20%, 20%。
* 模型訓練
  * 設計模型，一般來說像正弦波這類資料，採用「機器學習」的「多項式迴歸」方法大概就能解出不錯的結果。不過這裡故意使用一個全連結的神經網路來代替，輸入層就只有一個神經元，輸入值就是x，輸出層也只有一個神經元，而隱藏層使用8個神經元，每個神經元的輸出激勵函數採用「ReLu」，最終訓練時判定損失的函數則採用MSE（Mean Square Error, 均方誤差），度量方法採MAE（Mean Absolute Error, 平均絕對誤差），而調整優化方法則採「Adam」。整體模型採用TensorFlow的Keras方法來建構。更多損失函數和優化方法可參考[NTUST Edge AI Ch6-1 模型優化與佈署─模型訓練優化 ](https://omnixri.blogspot.com/p/ntust-edge-ai-ch6-1.html)。
  * 訓練模型，設定好模型後就可以開始訓練，由於這個範例很小，所以原始檔案並沒有啟動GPU加速計算，如果有需要可選擇主選單的[編輯]─[筆記本設定]─[硬體加速器]將[None]改成[GPU(Nvidia)]或[TPU(Google)]即可。這個範例指定訓練迭代次數(Epochs)次為500次，意思是不管是否達到滿意程度，到達設定值就停止。批量大小(batch_size)則是一次取多少筆資料來訓練，以600筆資料批量大小為64為例則要取10次才能完成一次迭代，所以理論上批量大小大一點會好些，但會隨資料性質及多樣性可能會造成模型訓練結有很大不同，所以可自行調整大小來加速訓練速度及測試收歛速度。
  * 繪製圖表，為了更清楚知道訓練過程中損失收歛的速度，可將訓練過程的輸出值繪成圖表來觀察，以圖示約在100次迭代後就幾乎收歛，只有很小幅度的下降。若想更清楚看到變化細節，可忽略前50筆（可自行調整）。另外除了損失率外亦可將平均絕對誤差(MAE)當成觀察內容。接著為了知道訓練了500次迭代的模型效能好不好，此時就可把測試集拿來當輸入求預測值，再將實際值和預測一起繪出，很明顯差了很多，還不滿足需求。表示訓練看起來好像不錯但實際上根本不能用。
* 訓練一個大模型
  * 設計模型，接著把隱藏層變成兩層，且兩層都使用16個神經元進行全連結，其它條件和先前一樣。
  * 訓練模型，同樣地訓練500次迭代。
  * 繪製圖表，同樣繪出LOSS和MAE，可看出大約在350次迭代後才趨於收歛。最後實際值和預測值也趨於擬合，已可看出一條很接近正弦波的曲（紅）線。代表這個模型已可應付大部份的狀況。
* 產生TensorFlow Lite模型
  * 產生有量化和沒有量化的模型，原始TensorFlow模型是以FP32（32bit浮點數）格式進行訓練，接著將其轉成TensorFlow Lite格式，接著若能把訓練好的參數降到INT8（8bit整數）則模型大小就能變為1/4。但前提是預測的結果不能變化太大才有意義。
  * 比較模型效能，在比較之前需將模型以不同參數格式載入，為了正確配置相關記憶體，需初始化TFLite的直譯器(Interpreter)再載入，最後將有無量化的比較結果繪製成圖表。由圖可看出綠色X和藍色X幾乎重疊，代表量化後的結果幾乎沒有影響。再經由數字來看，原始TensorFlow的模型損失率1.02%和轉成TensorFlow Lite 未優化前幾乎相同，而經過量化後變成1.08%，變化極小。此時再來看模型（參數）大小，原始TensorFlow使用了4096 Byte，經轉成TFLite後縮小至2788（減少1308）Byte，而經量化後，只剩2488(再減少300）Byte，和原本模型相比僅為60.7%。雖然離目標的25%（1/4大小）還很遠，這是因為這個模型不算太複雜且很小，所以暫時無法發揮太明顯效益。
* 產生TFLM模型，接著要安裝xxd工具程式，利用它將TFLite已量化的模型再轉成TFLM格式，即C語言格式檔案，預設會於/hello_world/train/models下產生一個model.cc的檔案，打開看就是一個2488 Byte的int格式陣列。
* 佈署到MCU上，最後把model.cc和其它MCU相關的程式(*.cc)一起編譯就能佈署到MCU上，不過這部份不在Colab上完成，MCU程式編譯和TFLM執行推論部份就留待下回分解。

![TensorFlow Lite for Microcontroller Hello Wolrd範例相關圖示](https://1.bp.blogspot.com/-DyJqL-GwQK0/YVFS-U3CPxI/AAAAAAAAExQ/BTp9VIC6_vIdo20ZJhMQKJtmxun6cZzkwCLcBGAsYHQ/s1658/iThome_Day_12_Fig_02.jpg)
Fig. 12-2 TensorFlow Lite for Microcontroller Hello Wolrd範例相關圖示。(OmniXRI整理製作, 2021/9/26)

參考連結

[Google TensorFlow Lite for Microcontrollers 中文學習指南](https://www.tensorflow.org/lite/microcontrollers?hl=zh-tw)  
[Pete Warden Youtube, TinyML Book Screencasts Playlist 影片清單](https://youtu.be/Fdt9xunlyCQ?list=PLtT1eAdRePYoovXJcDkV9RdabZ33H6Di0)  
[TensorFlow Lite for Microcontrollers Github 開源碼](https://github.com/tensorflow/tflite-micro)  
[TFLM Get started with microcontrollers, The Hello World example](https://www.tensorflow.org/lite/microcontrollers/get_started_low_level?hl=zh-tw)  
[Experiments with Google = TensorFlow Lite for Microcontrollers 案例分享](https://experiments.withgoogle.com/collection/tfliteformicrocontrollers)  
