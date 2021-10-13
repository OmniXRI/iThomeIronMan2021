## [Day 29] tinyML應用實例分享

話說受到火雞姐的鼓舞（其實是逃命），我來到了中國廚藝學院（其實是少林寺的伙房）拜師學藝，沒想到得罪了方丈和十八銅人，根本就離開不了這個如地獄般的地方。直到有一天我傷心到一夜白髮，唱出了雞姐最愛的「情和義值千金，上刀山下地獄去有何憾！」才得到大師的首肯，讓我參加食神大賽。當我和唐牛正面對決「佛跳牆」時，他還不服的說我抄襲，還好評審主持公道說：「比賽是這樣啦！就好像游泳跑步一般，不就是你做什麼他也做什麼，有啥好抗議呀？抗議無效！」。沒想到最後佛跳牆還被搞到爆炸，害我只好使出少林絕學「火雲掌」做出那碗看似簡單但吃了令人落淚的「黯然銷魂飯」才結束這一回合。

相信跟了28天的伙伴們，學了一大堆招式卻不知如何施展，內心一定很煩燥，想著到底這麼陽春的開發板、這麼少的算力和這麼簡單的模型到底能做出什麼有趣的AI應用。各位有福了，今(2021)年[tinyML基金會](https://www.tinyml.org/)和[Hackster.io](https://www.hackster.io)及多家贊助廠商（還包含台灣奇景光電和原相科技）一同舉辦「[Eyes on Edge: tinyML Vision Challenge!](https://www.hackster.io/contests/tinyml-vision)」，如圖Fig. 29-1所示。

![Eyes on Edge: tinyML Vision Challenge](https://1.bp.blogspot.com/-LzXYKj9uOvY/YWZ99nNNH6I/AAAAAAAAE7E/MpYyLjTwFmQe53bYA-zFX5tb7Dj39CthQCLcBGAsYHQ/s1658/iThome_Day_29_Fig_01.jpg)
Fig. 29-1 Eyes on Edge: tinyML Vision Challenge。(OmniXRI整理繪製, 2021/10/13)

這場比賽從今(2021)年4月開始，一直到9月結束，10/5才公佈得獎名單，還熱騰騰地。接下來就各位簡單介紹一下這六個精彩的作品，如圖Fig. 29-2所示，希望讓大家從模仿自學作起（別人做什麼，我也做什麼），當搞清楚這些門道後，就能獨立創作出屬於個人風格的作品（黯然瀨尿牛丸麵？）。完整作品說明，包括使用情境、相關領域知識理論、硬體零件、電路連接、工作源碼(Github)等，可參考每個作品簡介的「原文出處」連結。

![tinyML Vision Challenge 2021得獎作品](https://1.bp.blogspot.com/-eRixsuu3JUY/YWaWUWtSAjI/AAAAAAAAE7M/FZNY4jbJ3MAxPud0IKcAxXxGdS-gWONCwCLcBGAsYHQ/s1658/iThome_Day_29_Fig_02.jpg)
Fig. 29-2 tinyML Vision Challenge 2021得獎作品。(OmniXRI整理繪製, 2021/10/13)

## (a) TinyML Aerial Forest Fire Detection

![TinyML Aerial Forest Fire Detection](https://hackster.imgix.net/uploads/attachments/1350890/_TChJMhfSER.blob?auto=compress%2Cformat&w=900&h=675&fit=min)
[[原文出處]](https://www.hackster.io/team-sol/tinyml-aerial-forest-fire-detection-78ec6b)  

「**空中森林火災偵測**」，主要用到Arduino Nano 33 BLE Sense、ArduCam 2640攝影機模組及GPS相關元件，搭配「[The Flame Dataset](https://github.com/AlirezaShamsoshoara/Fire-Detection-UAV-Aerial-Image-Classification-Segmentation-UnmannedAerialVehicle)」公開資料集，通常這類應用多半時搭配紅外線（溫度感測）攝影機，但其重量會嚴重影響空拍機飛行時間，所以改用一般攝影機模組可以大幅簡輕重量，同時搭配間歇性取樣分析（每拍一次休息500ms）可更節省空拍機的電力耗損。他們使用了小型CNN模型，訓練了100次得到了96%的精確度，同時也確保能佈署到BLE Sense開發板中。最後再搭配GPS衛星導航路線巡檢，就能更確保森林火災發生時能即早被通知及撲滅，以免造成更大的傷亡及財產損失。

## (b) WorkSafe: CV based multiparameter monitoring and diagnostics

![WorkSafe](https://hackster.imgix.net/uploads/attachments/1346542/_zNPTmdfgDc.blob?auto=compress%2Cformat&w=900&h=675&fit=min)  
![r-PPG_1](https://hackster.imgix.net/uploads/attachments/1350172/image_Td8KtJ0q4L.png?auto=compress%2Cformat&w=1280&h=960&fit=max)  
![r-PPG_2](https://hackster.imgix.net/uploads/attachments/1350672/spo2_algo_JDL6rcHNmh.png?auto=compress%2Cformat&w=740&h=555&fit=max)  
[[原文出處]](https://www.hackster.io/TheBluePhoenix/worksafe-cv-based-multiparameter-monitoring-and-diagnostics-bdf042) 

「**WorkSafe：基於電腦視覺的多參數監控和診斷**」，主要用到樂鑫的ESP32 Cam及Node MCU、HC-SR04超音波測距模組及MLX90614紅外線（體溫）感測器，該團隊參考「[Non-Contact Physiological Parameters Extraction Using Facial Video Considering Illumination, Motion, Movement and Vibration](https://ieeexplore.ieee.org/stamp/stamp.jsp?arnumber=8715455)」論文從遠程光體積描記(r-PPG)方法來估測HR（心率）、SPO2（飽和氧）和 CBT（核心體溫），即以非接觸的方式，用紅外線反射及面部皮膚顏色變化來量測患者的重要生理資訊。由於紅外線測溫模型極容易受距離影響，所以這裡使用超音波測距方式來校正，該團隊相關研究也在IEEE Xplore發表「[Enhanced Pyrometric device with Long Range for mass screening based on MLX90614](https://ieeexplore.ieee.org/document/9487689)」。該系統經實測已接近傳統接觸式儀器，有機會發展成較為平價且普及的設備。

## (c) TinySewer - Low Power Sewer Faults Detection System

![TinySewer_1](https://hackster.imgix.net/uploads/attachments/1349700/_GHTrbZB2ar.blob?auto=compress%2Cformat&w=900&h=675&fit=min)  
![TinySewer_2](https://hackster.imgix.net/uploads/attachments/1346223/scale_5vvLTBTdXe.png?auto=compress%2Cformat&w=740&h=555&fit=max)  
![TinySewer_3](https://hackster.imgix.net/uploads/attachments/1346274/type_PSaRNyoPZH.png?auto=compress%2Cformat&w=740&h=555&fit=max)  
[[原文出處]](https://www.hackster.io/lurst811/tinysewer-low-power-sewer-faults-detection-system-64cd67) 

「**TinySewer - 低功耗下水道故障檢測系統**」，主要使用Arduino Portenta H7及Protenta Vision shield搭配Edge Impulse一起開發。藉由這個作品可搭載於小車上，取代工作人員在下水道中移動，利用視覺記錄影像同時辨識主要缺陷，包括裂縫（斷裂、塌陷）、根部侵入、阻塞（障礙物）、位移，透過移動的時間線，還可推算出大概的位置，方便檢修。透過Edge Impulse的遷移學習建立模型，使用MobileNetV2 96x96 Dropout 0.35，學習率為 0.35，最後一層有 40 個神經元。經過50個訓練週期後可得94%的精確度。最後生成OpenMV格式的標籤檔、模型檔及Python腳本檔，並修改部份內容後燒錄回Arduino Portenta中。

## (d) Plant Growth Estimation for High Throughput Phenotyping

![Plant Growth Estimation_1](https://hackster.imgix.net/uploads/attachments/1328800/_kC1zRrMZKG.blob?auto=compress%2Cformat&w=900&h=675&fit=min)  
![CNN](https://hackster.imgix.net/uploads/attachments/1347042/biomass_(1)_i5hTuw9OtN.jpg?auto=compress%2Cformat&w=740&h=555&fit=max)  
![Plant](https://hackster.imgix.net/uploads/attachments/1348504/biomass_(3)_xA29jFF2Xn.jpg?auto=compress%2Cformat&w=740&h=555&fit=max)  
[[原文出處]](https://www.hackster.io/dhruvsheth_/plant-growth-estimation-for-high-throughput-phenotyping-ba960a)  

「**高吞吐量表型的植物生長估計**」，主要使用SONY Spresense及配套攝影機模組，搭配Edge Impulse進行開發，利用影像來觀察植物葉子的外觀變化（表型）來推估植物生長的狀況。這個作品使用了[Growth monitoring of greenhouse lettuce](https://www.nature.com/articles/s41438-020-00345-6#Abs1)文中提供的3種資料集，為了擴增資料集，還加入縮放、旋轉等處理，而在資料集的多樣性上也考慮了光照、亮度、飽和度、品種、拍攝角度及運動取像模糊度等問題。SONY Spresense開發板上自帶1536kB RAM和 8192kB ROM (Flash)，所以可以容納較大的模型，較適合影像類的tinyML應用。在模型選用部份CNN架構，兩組卷積池化層，分別使用32和16個特徵圖，最後展平及搭配0.25隨機丟棄(dropout)得到輸出結果。經過訓練兩種不同模型，第一種通過植物分割計算的實際葉面積指數（LAI）作為標籤。第二個模型包括對應於植物生長階段的Day-wise標籤。最後得出合理的回歸值，

## (e) Flat Tire Detection Using Machine Vision

![Flat Tire Detection_1](https://hackster.imgix.net/uploads/attachments/1349547/output-onlinegiftools_(1)_y1hWcN8tRj.gif?auto=format%2Ccompress&gifq=35&w=900&h=675&fit=min&fm=mp4)  
![Flat Tire Detection_2](https://hackster.imgix.net/uploads/attachments/1336988/untitled_presentation_(5)_ZUcOS2q4Pp.png?auto=compress%2Cformat&w=740&h=555&fit=max)  
![Flat Tire Detection_3](https://hackster.imgix.net/uploads/attachments/1339597/collage_DOFKX1uZ3U.png?auto=compress%2Cformat&w=740&h=555&fit=max)  
[[原文出處]](https://www.hackster.io/rhammell/flat-tire-detection-using-machine-vision-8a7b03) 

「**使用機器視覺檢測爆胎**」，主要使用OpenMV Cam H7（自帶攝影機模組），配合Edge Impulse進行開發。這個作品利用機器視覺來查看汽車輪胎是滿還是漏氣，可應用於地磅站、自駕車車隊及租車歸還。在資料集建構上分為三個類別包括正確充氣（約45 psi）、扁平輪胎（約10 psi）及非輪胎（或無輪胎），原始影像尺寸為240x240像素之灰階影像。每個類別約取300張，接著使用Edge Impulse遷移學習模型MobileNet V2 96x96 0.35進行訓練，最後得出不錯的精確率。未來可加入更多的壓力區間，做為分類依據，即有機會取代人力或專業儀器量測。

## (f) Smart Bird Feeder

![Smart Bird Feeder_1](https://hackster.imgix.net/uploads/attachments/1358955/_q2JcQGlcbP.blob?auto=compress%2Cformat&w=900&h=675&fit=min)  
![Smart Bird Feeder_2](https://hackster.imgix.net/uploads/attachments/1335904/img_6095_uAYperkclx.JPG?auto=compress%2Cformat&w=740&h=555&fit=max)  
![Smart Bird Feeder_3](https://hackster.imgix.net/uploads/attachments/1349848/solar_power_DW2ijb2ENX.png)  
[[原文出處]](https://www.hackster.io/ariela-anna-audrey-nathan-tianlang-haoming-eric-edward-tara/smart-bird-feeder-1f5591)  

「**智能餵鳥器**」，主要使用Arduino Nano 33 BLE Sense及OV7675攝影機模組，搭配Edge Impulse進行開發。這個作品是一群小朋友完成的，主要是為了防止松鼠偷吃小鳥的食物，利用機器視覺來辨識是否為小鳥。這裡使用了2912張影像來訓練，而模型則採用Mobilenetv1_0.25_96修改後進行訓練。最後佈署到開發板後，如果偵側到松鼠還會發出噪音來嚇跑它們，反之是小鳥時就安靜。由於這項設備非常省電，所以只需簡單太陽能板就能供應足夠的電力。

## 小結

看完這麼多有趣的作品，是否心動了呢？這些作品都附有完整源碼及製作說明，「照圖施工、保證成功」，如不成功，請直接在作品網頁上留言和原作者聊聊，或許還可多交個朋友。

參考連結

[Hackster.io - Eyes on Edge: tinyML Vision Challenge!](https://www.hackster.io/contests/tinyml-vision)  

ps. 為讓文章更活潑傳達硬梆梆的技術內容，所以引用了經典電影「食神」的橋段，希望小弟戲劇性的二創不會引起電影公司的不悅，在此對星爺及電影公司致上崇高的敬意，敬請見諒。
