「**tinyML**」就字面上意思就是微小的機器學習（Tiny Machine Learning, tinyML)，但它又和人工智慧(Artificial Intelligence, AI)及微控制器單元(Micro-Controller Unit, MCU)（又稱單晶片）有什麼關連呢？

首先說明幾個常見的名詞定義及主要差異。

* 【人工智慧(Artificial Intelligence, AI)】：又稱人工智能，泛指能產生類似人類思考、判斷結果的人為程式、算法。其中最常見的應用包含電腦視覺(Computer Vision, CV)、自然語言處理及理解(Natural Language Processing / Understanding, NLP / NLU)及數據分析/挖礦(Data Analysis / Mining)幾大領域。
* 【機器學習(Machine Learning, ML)】：為人工智慧下的一部份，泛指透過數學統計進而模仿人類學習數據中的規則，進而推論未知數據的結果。常見的分類有監督式(Supervised)、非監督式(Unsupervised)及增強式(Reinforcement)學習，其主要功能為分類、迴歸及聚類。
* 【深度學習(Deep Learning, DL)】：為機器學習下的一個分支，類神經網路(Neural Network, NN)，它可透過更深及更寬的網路層及巨大量參數（權重）來學習各種影像、聲音及數據特徵，進而達到和人類似的學習、辨識及推論能力，為近年來當紅技術。由於其能力顯著，因此當許多非專業人士提及「人工智慧」時，通常指的就是「深度學習」技術，而非原始名詞定義。
 
![人工智慧、機器學習、深度學習](https://1.bp.blogspot.com/-XuWHULtnDgk/YULD2a9lvCI/AAAAAAAAEto/d45PphaZEYw5sI7W6M9vZ2d7lCw0cL9owCPcBGAYYCw/s1658/iThome_Day_02_Fig_01.jpg)
Fig. 2-1 人工智慧、機器學習、深度學習 (OmniXRI整理繪製,2018/5/24)

通常來說「機器學習」能處理的特徵數量及數學模型通常不會太複雜，因此所需的計算量也不會太大。但「深度學習」就反其道而行，越複雜的模型及越巨量的參數就能令推論準確率得到更好的提升，因此需要極高性能的運算設備來輔助訓練和推論的計算。所以如果要將AI應用放到運算能力很低、記憶體很少的MCU上時，就只選擇較微型的AI應用（如智慧感測器等）或者較小型的機器學習算法甚至超微型的深度學習模型來推論，因此「tinyML」需求因運而生。

基於上述AI微型化理念，許多CPU, GPU, MCU, AI加速計算晶片大廠、AI開發工具及應用廠商紛紛響應，於2019年成立微型機器學習基金會(tinyML Foundation) ，每年定期會舉辦高峰會，讓廠商、學界、社群都能共襄盛舉。2021高峰會有近六十個贊助商，其中台灣也有四家，分別是奇景光電(Himax)、裝智(On-Device AI)、原相(PixArt)及威盛(VIA)。

![tinyML基金會2021贊助商清單](https://1.bp.blogspot.com/-Wpwl2a-dODE/YULVCd9mTVI/AAAAAAAAEts/TgqLKeF_bIYcLOQ75kxx3D0SxvTjH9eqgCLcBGAsYHQ/s1783/iThome_Day_02_Fig_02.jpg)
Fig. 2-2 tinyML基金會2021贊助商清單 (OmniXRI整理繪製,2021/8/14)

根據該基金會對tinyML的定義：「**微型機器學習（tinyML）**為一個快速發展的機器學習技術和應用領域，包括硬體、算法和應用軟體。其能夠以極低功耗執行設備上的感測器(Sensor)的數據分析，通常在mW(毫瓦特)以下範圍，進而實現各種永遠上線（或稱常時啟動）(Always On)的應用例及使用電池供電的設備。」這裡的ML雖然指的是「機器學習」，不過亦可延伸解釋到「深度學習」甚至「人工智慧」、「邊緣智能」等名詞。從上述定義可得知tinyML 幾乎是鎖定MCU及低階CPU所推動的Edge AI。

目前tinyML基金會並沒有明確的定義那些項目才算是其範圍，也沒有制定特定的開發框架及函式庫（如機器人作業系統ROS），而是開放給硬體及開發平台供應商自行解釋及彼此合作。目前較常見的應用，包括振動偵測、手勢（運動感測器）偵測、感測器融合、關鍵字偵測（聲音段分類）、（時序訊號）異常偵測、影像分類、（影像）物件偵測 等應用，而所需算力也依序遞增。一般來說，以Arm Cortex-M系列為例，智慧感測器（如聲音、振動、溫濕度等）大約Arm Cortex-M0+, M3左右就能滿足，而智慧影像感測器（小尺寸影像）要完成影像分類及物件偵測工作，則需要Cortex-M4, M7甚至要到Cortex-A, R系列。

![Arm MCU等級晶片智慧運算能力與適用情境](https://1.bp.blogspot.com/-7-kEJyqoElQ/YULX3GGseHI/AAAAAAAAEt0/uZbff2UnOns1pE-11D_DgRjvzmxIcsX_gCLcBGAsYHQ/s1654/iThome_Day_02_Fig_03.jpg)
Fig. 2-3 Arm MCU等級晶片智慧運算能力與適用情境 (OmniXRI整理繪製,2021/8/14)

參考連結：

tinyML基金會 https://www.tinyml.org/
