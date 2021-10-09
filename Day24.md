## [Day 24] Edge Impulse + BLE Sense實現手勢動作辨識(上)

有了先前[[Day 20]](https://ithelp.ithome.com.tw/articles/10277682)[[Day 21]](https://ithelp.ithome.com.tw/articles/10277700)[[Day 22]](https://ithelp.ithome.com.tw/articles/10278171)「Edge Impulse + BLE Sense實現喚醒詞辨識」的基礎和[[Day 23] 讓tinyML感受你的律動](https://ithelp.ithome.com.tw/articles/10278725)對運動感測器(IMU)的認識後，相信接下來要說明的手勢（連續）動作辨識(Gesture/Continuous Motion Recognition)應該也能輕鬆上手。

在這個案例中，我們同樣使用Edge Impulse雲端一站式tinyML開發平台，並以Arduino Nano 33 BLE Sense(**以下簡稱 BLE Sense**)開發板作為感測及佈署的硬體，主要會用到ST LSM9DS1 IMU運動感測器來收集XYZ三個軸向的加速度變化，而陀螺儀和地磁計就暫時不會用到。如圖Fig. 24-1所示。

![Edge Impulse結合Arduino Nano 33 BLE Sense作手勢動作辨識](https://1.bp.blogspot.com/-EqIe9kfdM8c/YWE4XBT-F9I/AAAAAAAAE4k/mS4ju0P7hawKBy3myd3ciqoqXQtRZccEwCLcBGAsYHQ/s1658/iThome_Day_24_Fig_01.jpg)  
Fig. 24-1 Edge Impulse結合Arduino Nano 33 BLE Sense作手勢動作辨識。(OmniXRI整理繪製, 2021/10/9)  

為方便測試手勢運動，首先把BLE Sense固定在右手手背上（如圖Fig. 24-1a），如果不好固定可用使用一個手套或魔鬼沾做成手環，再將板子固定在手背上，要注意最好能緊密貼合，不要可以隨意晃動，這樣取到的信號品質會好些，微量振動會少些。

![將BLE Sense開發板安置於手背以方便手勢辨識](https://1.bp.blogspot.com/-Cl8j6_XW494/YWD6P7eCPOI/AAAAAAAAE4E/yx6ugAtzj6wFNkXLPcZUV1h5fj6oEbHqgCPcBGAYYCw/s320/iThome_Day_24_Fig_02a.gif)  
Fig. 24-2a 將BLE Sense開發板安置於手背以方便手勢辨識。(OmniXRI整理繪製, 2021/10/9)

再來自定義四種手勢方便後續測試。當然這裡你可以自定義不同的手勢移動方式及分類數量，由於運動感測器非常敏感，容易收到許多雜訊，所以建議手勢間要有明確的XYZ軸向變化，比方前後、上下、左右、畫圓圈等運動，不能差異太少，否則模型訓練時會很難收歛，佈署後推論時準確率會很低。

* **OK** : 伸出姆指比「讚」（當然姆指伸不伸出來是沒差的），由後向前運動並停頓一下。

    ![OK手勢](https://1.bp.blogspot.com/-ewEyRkzhPzM/YWD6P_iSUOI/AAAAAAAAE4I/MM6TRGpTM8Q0-RUwrtR0uDgYr3O1I9FKgCPcBGAYYCw/s320/iThome_Day_24_Fig_02b.gif)  
    Fig. 24-2a OK手勢(OmniXRI整理繪製, 2021/10/9)
    
* **NG** : 伸出食指由中間向右移動，再移至左方，再移回中間啟始位置，並頓點一下。只要搖擺一次。

    ![NG手勢](https://1.bp.blogspot.com/-cA6lvVm8dqM/YWD66IOMeaI/AAAAAAAAE4U/4KGmUnUJx4krtSHY6w-rRNGnfQDCCdUKACLcBGAsYHQ/s320/iThome_Day_24_Fig_02c.gif)  
    Fig. 24-2b NG手勢(OmniXRI整理繪製, 2021/10/9)
    
* **Pass** : 伸出食指畫一個圓圈，再回到原來的位置。

    ![Pass手勢](https://1.bp.blogspot.com/-4CbGfoCHtxQ/YWD6SvkNReI/AAAAAAAAE4M/mwnfVYUcr3E0AgUwTWUAO5IgMDUes3JAgCPcBGAYYCw/s320/iThome_Day_24_Fig_02d.gif)  
    Fig. 24-2c Pass手勢(OmniXRI整理繪製, 2021/10/9)
    
* **Idle** : 閒置手勢，手勢預備動作，須有固定位置和手的擺放角度。如需手勢任意擺放，則會增加資料集取樣複雜度和產生更多不明確手勢，所以暫以固定手勢預備動作來辨識。如有需要還可另外增設「不明手勢(Unknow)」分類。

    ![Idel手勢](https://1.bp.blogspot.com/-hubLlxYILGg/YWD6SmVElxI/AAAAAAAAE4Q/6EuVWpYXMPEq6_xRqIhKONAg2FpRTsMQwCPcBGAYYCw/s320/iThome_Day_24_Fig_02e.gif)  
    Fig. 24-2d Idle手勢（手有在動，請仔細看）(OmniXRI整理繪製, 2021/10/9)
    
接著就可以進入Edge Impulse雲端平台依序執行下列工作。
1. 建立新專案
2. 確認連線裝置
3. 收集及建置資料集
4. 模型設計及訓練
5. 模型佈署及推論

## 建立新專案

登入Edge Impulse後，點擊右上角使用者名稱後，即可選擇「Create New Project」，輸入新專案名稱創建一個新專案，系統會提示想要開那一類的專案，選擇「Accelerometer Data」，然後按右下綠色按鈕【**Let's get started!**】即可開始新專案，或者點擊「Connect your development board」進入「連線裝置(Device)」頁面。如圖Fig. 24-3左上所示。

## 確認連線裝置

接著於Windows命令列模式執行「**edge-impulse-daemon --clean**」重新將BLE Sense連上網頁，記得電腦要處於可於上網狀態。輸入使用者帳號、密碼，並選擇專案名稱，輸入裝置名稱，即可回到Edge Impulse網頁「連線裝置(Device)」頁面檢查是否已能正確連結（亮綠燈）。如圖Fig. 24-3右及下方所示。

![Edge Impulse 開新專案與裝置連線](https://1.bp.blogspot.com/-OcT5W_m9PCk/YWE1buYXDvI/AAAAAAAAE4c/7G8I0WuDF_wgEKMeIdFLJTA4ZcsD4hF0gCLcBGAsYHQ/s1658/iThome_Day_24_Fig_03.jpg)
Fig. 24-3 Edge Impulse 開新專案與裝置連線。(OmniXRI整理繪製，2021/10/9)


## 收集及建置資料集

接著錄製四種手勢，分類標籤（Label)分別取名為「OK, NG, Pass, Idle」，為簡單測試，這裡每種手勢錄製20秒，取樣頻率100Hz，進入後連續作10次動作，記得手勢要有間隔（間歇），再來分割樣本，做兩次可得到20個樣本，如果樣本品質不好，可以多錄及分割幾次，以取得足夠的樣本。最後四個分類會取得80個樣本，再將其中16個（4分類各4筆,佔20%）移作測試集用。如圖Fig. 24-4所示。取樣時大家應可注意到，這和聲音樣本最大的不同就是從一維變成三維資料了，且沒有聲音播放鍵，只有波形圖。

![Edge Impulse 收集和建置手勢動作資料集](https://1.bp.blogspot.com/-BMK0DgtXrjU/YWFNXfR-U1I/AAAAAAAAE4s/FS_2GbNeci89t_G96M29sTT82f48eXcrACLcBGAsYHQ/s1658/iThome_Day_24_Fig_04.jpg)
Fig. 24-4 Edge Impulse 收集和建置手勢動作資料集。(OmniXRI整理繪製, 2021/10/9)

這裡補充一下資料集多樣性問題。上面範例只取4分類各20筆資料，這只是為了方便後續訓練時能縮短一些時間，但這麼少的資料在訓練時很容易就會發生訓練過擬合(Overfitting)問題。建議實際操作時能增加多一點的樣本，如果可以的話，甚至給多個人來重複這些動作。現實狀況，每個人配戴開發板的位置、角度可能都有些不同，即便是同一人，重複穿戴也難保證會完全一樣。另外每個人的手勢運動速度、動作大小、移動角度、停頓力道等都有可能不同，所以收集越多就能提高訓練出的模型有好的推論精確度表現。

目前手勢辨識分為兩大類，一種就是本文提到的方式，以IMU的運動數據來進行辨識，但此類比較適合動態手勢。另一種則是以影像感測器獲得的數據來辨識，所以不論動態或靜態手勢都能辨識。前者並不像聲音（短字詞）有很多公開資料集可供使用，而後者就有較多公開資料集可參考。大家都知道在手機觸控螢幕上，有單指或雙指甚至單、雙擊的常用手勢，但這樣人機介面並沒有延伸到基於IMU感測器的3D（浮空）手勢辨識，或許未來會有這類的公開資料集出現。但在這之前大家只好乖乖建置自己資料集。

=== 「模型設計及訓練」、「模型佈署及推論」就留待下回分解 ===

參考連結

[Edge Impulse Document - Tutorials - Continuous motion recognition說明文件](https://docs.edgeimpulse.com/docs/continuous-motion-recognition)
