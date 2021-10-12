## [Day 27]  Edge Impulse + BLE Sense實現影像分類(上) 

在前面章節已介紹如何讓Arduino Nano 33 BLE Sense（以下簡稱**BLE Sense**）配合Edge Impulse來完成聽覺、運動及色彩感測的tinyML應用，接下來就和大家介紹如何將視覺也加入tinyML。

目前人工智慧應於電腦視覺有很多領域，最常見的有下列幾種。
* **影像分類(Image Classification)**：一張影像只有一個主體物件（或主體物件佔影像1/3大小以上）需要被辨識。
* **物件偵測(Object Detection)**：一張影像有多個物件，可能有大有小，須逐一被找出邊界框(Bounding Box)和辨識物件分類。
* **語義分割(Semantic Segmentation)**：每個像素進行分類，以達到物件可被精細切割出輪廓，但相同類形物件重疊時會被分在同一類像素而無法正確分割出物件的數量。
* **實例分割(Instance Segmentation)**：結合語義分割和物件偵測，不僅將每個像素分類，同時也把相同類別不同個體進行分割，可更完美計算出實際物體個數及得到物件精細分割邊界。

以上常見電腦視覺應用，通常所需參數（記憶體）、計算量（計算速度）頗大且依序遞增，甚至呈等比級數放大，所以一般「影像分類」要Arm Cortex-M4以上等級才能勉強完成，而「物件偵測」常見的YOLOv3算法仍很難塞進Cortex-M7中，通常要像樹莓派這類Arm Cortex-A53, A57的多核CPU及夠多的記憶體（DDR）才勉強能處理，而「影像（語義、實例）分割」這類應用若沒有專用的AI（神經網路）加速計算晶片或GPU的幫助，根本很難在單板微電腦上實現。因此目前tinyML僅適合較簡易型的「影像分類」應用，且輸入影像不能太大，建議96x96像素以下，灰階影像為佳，彩色影像次之。

以下就會以BLE Sense加上Edge Impulse平台來做「**影像分類**」實驗。共準備60張已裁切128x128像素全彩(RGB888)彩色影像，其中分為3類，史蒂芬周(Chow)、薛家燕(Xue)及不明(Unknow)。每個分類各20張，其中16張作為訓練集、4張作為測試集。如圖Fig. 27-1所示。使用這麼小的影像及這麼少的資料集，雖然會影響後續模型的訓練及推論結果，但這裡只是方便大家快速訓練及測試，如有需要可自行增加更多的資料集數量及多樣性來增加模型的可用性。

![Edge Impulse 影像(分類)辨識](https://1.bp.blogspot.com/-7ICv-Lot6u8/YWTraNqTfuI/AAAAAAAAE6A/X0MGQai0vWUCcQkRWcsgUqwUH2flb-gsgCLcBGAsYHQ/s1658/iThome_Day_27_Fig_01.jpg)
Fig. 27-1 Edge Impulse 影像(分類)辨識。(OmniXRI整理繪製, 2021/10/12)

在Edge Impulse的官方線上說明文件中，有BLE Sense加上**OV7675**攝影機模組（類似手機鏡頭模組）的搭配[使用說明](https://docs.edgeimpulse.com/docs/arduino-nano-33-ble-sense)。但無奈手邊只有**OV7670**（M12鏡頭攝影機）模組，經查閱相關網站後，得知一樣可以使用，只是**OV7670**模組的排針是18 Pin，OV7675模組的排針為20 Pin，OV7670的最後兩Pin PEN/RST(Pin 17)及PWDN/PDN(Pin 18)要對到BLE Sense的A2和A3腳位，如下所示。

|    **OV7675**    | **BLE Sense** |    **OV7670**    |
| :--------------: | :-----------: | :--------------: |
|    NC (Pin 17)   |      --       |        --        |
|    NC (Pin 18)   |      --       |        --        |
| PEN/RST (Pin 19) |    **A2**     | PEN/RST (Pin 17) |
| PEN/RST (Pin 20) |    **A3**     | PWDN/PDN (Pin 18)|

目前Edge Impulse可支援的攝影機模組，如圖Fig. 27-2所示。本想拿OV7670攝影機模組來替代，但不幸地是，一算手上模組的排針接腳怎麼有22 Pin，再仔細一查竟然是帶有**AL422B**(**FIFO**影像緩衝區)功能的OV7670模組。再到Arduino網站及其它各大論壇檢索，發現目前Arduino官方只支援不帶FIFO功能的OV7670。雖然已下單採購，但確定這兩天還到不了貨，可是不能讓發文中斷，於是改用另一種解決方案來和大家介紹，就是以「**上傳已存在資料集(Upload existing data)**」的方式來替代。而以攝影機取得影像建立資料集的方法就麻煩大家先參考一下官方[說明文件](https://docs.edgeimpulse.com/docs/image-classification)。

![Edge Impulse 支援具相機模組之開發板](https://1.bp.blogspot.com/-2pDo8lyYSXI/YWTrZwnmm9I/AAAAAAAAE58/mDaNniUL6QoZ3haZMf50VcIrrL_U0WP5ACLcBGAsYHQ/s1658/iThome_Day_27_Fig_02.jpg)
Fig. 27-2 Edge Impulse 支援具相機模組之開發板。(OmniXRI整理繪製, 2021/10/12)

## 建立專案及上傳資料

首先點擊Edge Impulse操作頁面右上角使用者圖像，選擇建立一個新專案，輸入專案名稱（假設為Sight_Test）。建立時可依序選擇資料型態「**影像類(Images)**」，工作類型「**影像分類(Image Classification)**」，即可創立一個新專案。

接著切換到「**資料擷取(Data Accquisition)**」頁面，點擊「**收集資料(Collected data)**」右上角上箭頭符號，進入「**上傳已存在資料(Upload exiting data)**」頁面。接著依序上傳3個分類，訓練集和測試集資料，記得要點擊對應類型及輸入正確標籤。當把所有資料都上傳完畢後，就可回到「**資料擷取(Data Accquisition)**」頁面。此時頁面上方應會出現訓練資料集總數量、分類比例及訓練/測試集比例，如果想觀察測試集內容，則點擊頁面上方「**測試集(Test Data)**」按鍵來切換。完整步驟如圖Fig. 27-3所示。

![Edge Impulse 上傳已存在資料集](https://1.bp.blogspot.com/-ZtsDXRywm34/YWT-wN1QMMI/AAAAAAAAE6M/Tw-gJlidq2gdwVwrz_pcSl3w1zRyzQf6QCLcBGAsYHQ/s1658/iThome_Day_27_Fig_03.jpg)
Fig. 27-3 Edge Impulse 上傳已存在資料集。(OmniXRI整理繪製, 2021/10/12)

目前這些準備好的訓練集及測試集都是事先從網路上擷取到且已預裁切好的，這和一般直接從攝影機上取得的影像有很多不同，如影像長寬比、影像品質（壓縮、還原及拍攝不良等）、光照（亮度、對比、色彩飽和度等）及其它各種造成影像內容差異之因素，而這些不同就會嚴重影響模型訓練的結果。這裡僅為示範流程，所以省略一些資料收集的要注意的事項，也不考慮模型訓練是否能收歛到理想範圍，還請見諒。

## 模型建置及區塊設定

完成資料集建置後，接著切換到「**模型設計(Impulse Design)**」頁面，第一個欄位「**影像資料(Image Data)**」已幫大家預設好，不管原始影像是什麼尺寸，最後都會被縮成96x96點像素再輸入到模型中計算，下拉式選單可以選擇「**符合短邊(Fit shortest axis)**」、「**符合長邊(Fit longest axis)**」或「**正方形(Square)**」。如果大家是從網路收集影像時，若長短邊比值忽大忽小時，那麼經過這個縮放動作後，有可能會讓影像內容變形，導致資料集品質不佳，容易造成模型難以訓練或推論準確度不高。

接著新增「**處理區塊(Processing Block)**」，選擇「**影像(Image)**」按下【**Add**】按鈕新增即可。再來新增「**學習區塊(Learning Block)**」，這裡系統建議選擇「**遷移學習(Transfer Learning)**」，按下【**Add**】按鈕新增即可。這裡選擇「**分類（Classification (Keras))**」亦可，只是會得到不一樣的訓練結果及推論準確度。最後按下【**Save Impulse**】即可更新左側「**Impulse Design**」的選單。會新增出「**Image**」和「**Transfer Learning**」兩個新頁面。完整操作步驟如圖Fig. 27-4所示。

![Edge Impulse 模型設計及指定區塊](https://1.bp.blogspot.com/-HkU9nz73-9g/YWUwKYfXnZI/AAAAAAAAE6k/ob5Rmmf0IDsNh89BQIhS20lNfGXJXwA-ACLcBGAsYHQ/s1658/iThome_Day_27_Fig_04.jpg)
Fig. 27-4 Edge Impulse 模型設計及指定區塊。(OmniXRI整理繪製, 2021/10/12)

再來切換到新增的「**影像(Image)**」頁面，「**參數(Parameter)**」子頁面可暫時略過，直接切換到「**產生特徵(Generate Features)**」子頁面。按下【**產生特徵(Generate Features)**】按鈕後，稍待一會兒，就會產生特徵分佈圖及預計MCU推論效能及記憶體使用量。有了這些特徵值後續才能繼續進行訓練模型。完整步驟如圖Fig. 27-5所示。

![Edge Impulse 資料集產生特徵](https://1.bp.blogspot.com/-gainGs8bymI/YWUn4sMPI6I/AAAAAAAAE6Y/LLFkP67T8UMFoQiJPcNhXrIqCDvczD4aQCLcBGAsYHQ/s1658/iThome_Day_27_Fig_05.jpg)
Fig. 27-5 Edge Impulse 資料集產生特徵。(OmniXRI整理繪製, 2021/10/12)

=== 二種學習區塊「**遷移學習**」及「**影像分類**」的訓練及測試結果部份就留待下回分解 ===

參考連結

[Edge Impulse Tutorials - Adding sight to your sensors說明文件](https://docs.edgeimpulse.com/docs/image-classification)  
[Edge Impulse Development Board - Arduino Nano 33 BLE Sense with OV7675 camera add-on說明文件](https://docs.edgeimpulse.com/docs/arduino-nano-33-ble-sense)  
