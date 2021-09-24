話說唐牛搶了我的餐廳，當上老闆後，就開始訓斥我的廚師：「椅子要多窄有多窄，吸管要多粗有多粗，冰塊要多大有多大，薯條要炸多老有多老，等小鬼們吃完之後要多渴有多渴，你應該知道我的意思！」，廚師害怕輕聲的答道：「這樣對小朋友的健康不太好吧？」，結果廚師就被一頓爆踢，還口中喃喃自語：「不要逼我，不要逼我，我是個好人啊！把他拖出去。」

聽到這個，不禁讓我想到上次那想作AI咖哩飯的AI工程師也是被老闆這樣要求，「tinyML的資料收集、標註要多簡化就多簡化，模型參數要多巨量有多巨量，訓練時間要多短有多短，測試集推論精度要多棒有多棒（過擬合Overfitting)，等模型佈署後客人要多抱怨有多抱怨，你應該知道我（可以向客人追加專案預算）的意思！」，AI工程師臉上三百條線的問道：「這樣對客人的智能產品不太好吧？我們很難把這麼大的模型塞到MCU中」，結果...（不要亂猜，AI工程師還好好地癱坐在電腦桌前）

上次在[[Day 07]](https://ithelp.ithome.com.tw/articles/10268515)時已有和大家介紹過AI專案的開發流程，但這個流程在tinyML上還適用嗎？答案可以說介於可以和不可以之間。為什麼這麼說呢？因為在傳統中大型AI應用程式中，幾乎都在同一個平台（如雲端伺服器加AI加速運算裝置）或同一部電腦（如PC加Nvidia GPU或樹莓派加INTEL神經運算棒）上就能完成資料標註、模型建置、訓練調參、模型優化和佈署測試，只需會Python語言就幾乎能全部搞定，包括人機界面。但遇到像tinyML這種只能在MCU上運作，資源超級少的平台上，很多工作就要分散到其它平台上去開發，只留下推論部份。若再加上先前提過，每個MCU供應商可能有自己的開發平台，那就要學習更多技術（含硬體控制）才能上手，這就導致許多只會Python的AI工程師大呼吃不消，甚至碰不了，因為多數的MCU都是以C語言甚至是ASM（組合）語言開發的。

如圖Fig. 10-1所示，為了解決這個麻煩，許多廠商紛紛推出無碼（No-Code, 不用寫程式）或少碼(Low-Code, 只需少量程式）的tinyML開發解決方案，包括整合型開發平台和依MCU框架開發兩大類方案。前者把所有AI開發步驟全部整合在雲端，使用者只需按著步驟就能完成一些常見的AI應用，如聲音辨識，異常振動、影像分類、影像物件偵測等，優點是不需寫任何程式碼就能佈署到MCU上，且提供很多的視覺化工具幫忙調整模型，但缺點就是彈性較小。而後者雖然各家MCU都有自己的開發工具，但如果是使用Arm Cortex-M系列，則Arm有提供Mbed OS作業系統和CMSIS標準函式庫，其中更有專門用於神經網路加速運算的CMSIS-NN函式庫，方便同樣是Arm Cortex-M系列的MCU跨廠牌使用。另外像Google 也有提供TensorFlow Lite Micro (TFLM or TFLu, Microcontroller專用），方便使用自家TensorFlow開發出的模型，使其可以輕易的轉成MCU可以運作的程式碼。當然目前支援tinyML的開發板多半是Arm Cortex-M系列MCU，所以轉換出來的MCU程式也多半是由CMSIS-NN函式來完成。使用MCU框架開發有很大彈性，還可充份融合感測器及週邊模組，不過缺點也很明顯，就是沒有一定MCU程式開發功力的人，一開始會搞得一頭霧水，不知從何下手。在接下來的章節中會儘量幫大家介紹不要無腦使用，也不要燒腦使用tinyML on MCU。

![傳統AI及tinyML(MCU AI)應用程式開發流程圖](https://1.bp.blogspot.com/-OTkL4kXg1T4/YU1PjbPL9WI/AAAAAAAAEwY/sNxKlwVcB9YWYKOzALa2mbfLTjpDclRpACLcBGAsYHQ/s1658/iThome_Day_10_Fig_01.jpg)
Fig. 10-1 傳統AI及tinyML(MCU AI)應用程式開發流程圖。(OmniXRI整理繪製, 2021/8/14)

### tinyML整合型開發平台

tinyML整合型開發平台供應商為了讓使用者不用懂太多AI技術，也不用會寫MCU程式，所以通常提供一站式雲端服務，申請一個帳號加上網頁瀏覽器就能開始使用，主要整合了下列項目：

* 資料收集：可即時收錄聲音、運動感測器信號、網路攝影機取像，或自行準備預錄或影像資料集上傳。
* 資料標註：可分割連續資料並標註，或者直接標註上傳的資料集。
* 模型選用：通常只有數種常用模型可選，可作部份網路結構微調。通常不支援從常見框架（如TensorFlow, PyTorch, ONNX...）建立的模型轉換或者僅支援特定框架。
* 訓練調參：提供資料簡單濾波（去除雜訊）、資料分佈圖表、訓練結果正確率比較表等各種可視化圖表，方便調整特定超參數。
* 模型優化：一般中大型AI應用如果沒有佈署空間限制問題，通常不一定需要這個步驟，但對於MCU這種資源很有限的執行推論裝置一定要加入。通常這個部份可使用量化、剪枝、蒸餾、壓縮等技術來減化模型結構及參數（記憶體使用）量，同時要確保優化後推論準確率只能掉個幾%。
* 佈署測試：不管訓練結果如何，最後還是要佈署到MCU上。有些是將優化後的模型結構和參數轉成C語言程式(*.c, *.h)，再由原MCU開發平台當成一部份程式一起編譯，這樣方便整合原先的MCU程式。另一種則是安裝一些套件直接幫你燒錄到特定開發板的MCU中，再使用虛擬序列埠(COM)將推論結果從MCU輸出回電腦上，單純把tinyML的開發板當成一種智慧型感測器，不用寫半行MCU程式。最後如果測試結果不理想或準確率過低，則需回到訓練調參，甚至是資料收集、標註步驟重新來過，直至滿意為止。

目前常見的tinyML整合型開發平台供應商，如Fig. 10-2所示。當然不只這些，只是這幾個平台提供較多的學習資源及參考資料，更重要地是「免費」（不知道何時會開始收費）。這幾個平台可支援的開發板清單可參考[[Day 03] tinyML開發板介紹](https://ithelp.ithome.com.tw/articles/10265166)。

* [Edge Impules](https://docs.edgeimpulse.com/docs)
* [Fraunhofer IMS AIfES](https://www.ims.fraunhofer.de/en/Business-Unit/Industry/Industrial-AI/Artificial-Intelligence-for-Embedded-Systems-AIfES.html)（另可支援Arduino UNO R3）
* [AITS cAInvas](https://www.ai-tech.systems/cainvas/)（另有提供類似APP方式的tinyML應用程式）
* [SensiML](https://sensiml.com/resources/information/)

![常見的tinyML整合型開發平台供應商](https://1.bp.blogspot.com/-2lbjwVn0arc/YU11XeDfw4I/AAAAAAAAEwo/Nq-n7-maSBQ9vp2EH7h7Fz5Of2cikF5-ACLcBGAsYHQ/s1658/iThome_Day_10_Fig_03.jpg)
Fig. 10-2 常見的tinyML整合型開發平台供應商。(OmniXRI整理繪製, 2021/9/24)

### 依MCU框架開發tinyML

如果你是一般的MCU工程師，那大概對Fig. 10-3的程式開發堆疊(Stack)不會太陌生，只是最上面應用層和C Code中間多了一些東西，接下來就幫大家補充說明一下。

一般從事AI應用開發的人多半會選擇較多人使用的框架（建模、訓練、佈署），如TensorFlow, PyTorch, ONNX等。不過這樣的框架都太大，通常連小型單板微電腦（如樹莓派、手機、平板等）都塞不進去。所以需要一些優化工具來把模型簡化、參數減小，如Google TensorFlow Lite, Nvidia TensorRT, Intel OpenVINO等。不過這些工具轉換後的內容對於tinyML所使用的MCU來說還是太大，所以才有像Google TensorFlow Lite for Microcontroller這類的工具出現，企圖用更精簡的方式來產生MCU所需執行的程式碼。

我們都知道在電腦上可用多執行緒或單指令多資料流（SIMD）指令集的CPU或者是具有超多乘加器的GPU來加速運算（訓練及推論），但在一般MCU上就很難有這些，只能靠CPU慢慢算。幸運地是，在[[Day 06] tinyML的重要推手Arm Cortex-M MCU](https://ithelp.ithome.com.tw/articles/10267487)時曾介紹過Arm Cortex-M4以上MCU就有支援單指令週期乘加器、硬體除法和SIMD指令，這可大幅提升運算速度。但這些超強功能，沒有寫組合語言根本很難作到，所以Arm CMSIS就提供了通用介面程式方便大家使用，尤其更提供的CMSIS-NN來完善神經網路加速運算，把常用的基本元素都已封裝好。因此像TFLM就能輕鬆使用這樣的介面來轉換成MCU通用的C語言程式。

在MCU開發中，通常寫好C語言後配合GCC或者專屬編譯器(Compiler)就能產生能在MCU上運行的執行檔案(*.bin或 *.hex)。有時為了更方便管理工作排程、資源衝突、檔案管理等高階動作，有時也會MCU上加入超迷你的作業系統，如Arm Mbed OS, freeRTOS等。當然不使用通用型要自己開發作業也是OK的，那又是另一個神級工作，這裡就不加討論了。

![小型AI（tinyML)應用程式開發堆疊](https://1.bp.blogspot.com/-3kKHyjm_9Tw/YU1PjQqi9TI/AAAAAAAAEwc/pZU473RAiKQr_DiboxOsG6jAz_FiKSqkgCLcBGAsYHQ/s1658/iThome_Day_10_Fig_02.jpg)
Fig. 10-3 小型AI（tinyML)應用程式開發堆疊。(OmniXRI整理繪製, 2021/8/14)

由Fig. 10-3及以上說明，大概就能理解整合型開發平台和使用MCU框架開發其實都差不多，只是這些供應商幫你省去那人煩人的步驟，讓你完全不用懂MCU的運作就能使用。另外為了讓MCU發揮更多的計算能力或者能使用標準型框架開發出的模型，有些整合型平台供應商也會自行提供專屬的函式庫(SDK/Lib)甚至結合編譯器廠商讓自家產生的MCU碼執行上更有效率。而至於想走那條開發路線就隨個人喜好而定了，沒有絕對好壞。

參考連結

[Arm Mbed OS](https://os.mbed.com/)
[Arm Mbed OS Github](https://github.com/ARMmbed/mbed-os)
[Arm Common Microcontroller Software Interface Standard (CMSIS) Documentation](https://arm-software.github.io/CMSIS_5/General/html/index.html)
[Arm CMSIS Version 5 Github](https://github.com/ARM-software/CMSIS_5)


ps. 為讓文章更活潑傳達硬梆梆的技術內容，所以引用了經典電影「食神」的橋段，希望小弟戲劇性的二創不會引起電影公司的不悅，在此對星爺及電影公司致上崇高的敬意，敬請見諒。
