## [Day 23]  讓tinyML感受你的律動

話說為了要作出吃了會考試100分、會變聰明、變漂亮的「爆漿瀨尿蝦牛丸」，我特別找來廟街最重情義的火雞姐來幫忙。火雞姐腕力驚人，平均每塊牛肉需要二萬六千八百多次不停敲打才能打出這麼好的牛丸。「城市睇真D」還特別派了記者來專訪這個好吃、新奇又好玩，彈力十足到可以拿來打乒乓球的產品，結果她竟然問了一句，你如何證明這個牛肉真的被打了二萬六千八百多次而不是一萬六千八百多次呢？為了證明我所言不虛，於是我拿出了人稱tinyML神器的「**Arduino Nano 33 BLE Sense**(**以下簡稱BLE Sense**)」（如圖Fig. 23-1），用它上面的LSM9DS1九軸運動感測器（**以下簡稱IMU**）來計算火雞姐雙手揮動的次數和分析她精準的打擊動作，才平息了這次抹黑的消息。

![Arduino Nano 33 BLE Sense LMS9DS1九軸運動感測器(IMU)](https://1.bp.blogspot.com/-pFG-AabbeoQ/YV_45iC6ffI/AAAAAAAAE3g/wGPWVqUSwyYKY93f7rojKojG3v20lKRbwCLcBGAsYHQ/s1663/iThome_Day_23_Fig_01.jpg)
Fig. 23-1 Arduino Nano 33 BLE Sense LMS9DS1九軸運動感測器(IMU)。(OmniXRI整理繪製, 2021/10/8)

什麼是IMU呢？全名為Inertial Measurement Unit，慣性量測單元，俗稱運動感測器(Motion Sensor)。十多年前在微機電(MEMS)製程技術的幫助下，開始大量微形化，因此而大行其道，常見於各種車輛安全氣囊、行動裝置的掉落偵測、體感游戲機搖桿（如NDS Wii, PS Move等）、運動手環等裝置上。IMU主要包括下列三種主要元件。
* **加速度計(Accelerometer)**，又稱重力計，又因重力單位為G，所以也常簡稱為G Sensor。
* **陀螺儀(Gyroscope)**，又稱角加速度計，主要用於測各軸向旋轉速度變化，常被簡稱Gyro Sensor。
* **地磁計(Magnetometer)**，主要用於量測地磁角度，相當於指南針作用，因此也常被稱為電子羅盤。

剛開始三種元件多為獨立產品，後來因不同應用需求、體積大小及價格考量，慢慢發展出整合兩項或三項元件的產品。通常每種元件多半可以偵測XYZ三個軸向，所以常被簡稱三軸、六軸(G + Gyro)、九軸(G + Gyro + Magneto)運動感測器。後來又因衛星導航(GPS)常遇到高架橋、隧道等遮蔽時無法精準定位，於是就有人整合九軸資訊開發出慣性導航算法，可用於室內（或無GPS信號）導航使用，因此九軸運動感測器也被稱為慣性量測元件(IMU)。

由上述介紹可知，想知道運動方向、速度就得搭配這些元件。所以BLE Sense上也搭載了ST LSM9DS1 IMU感測器，其主要規格如下所示。更完整內容可參考官方提供的[技術手冊](https://www.st.com/resource/en/datasheet/lsm9ds1.pdf)。
* 3 acceleration channels, 3 angular rate channels, 3 magnetic field channels
* ±2/±4/±8/±16 g linear acceleration full scale （多選一）
* ±4/±8/±12/±16 gauss magnetic full scale （多選一）
* ±245/±500/±2000 dps angular rate full scale （多選一）
* 16-bit data output
* SPI / I2C serial interfaces
* Analog supply voltage 1.9 V to 3.6 V 
* “Always-On" eco power mode down to 1.9mA

在先前[[Day 09] tinyML開胃菜Arduino IDE上桌（下）](https://ithelp.ithome.com.tw/articles/10269745)已經介紹過如何安裝BLE Sense這塊開發板的基本函式庫，透過「Arduino_LSM9DS1」就可輕鬆讀取三種感測元件的數值。接下來就簡單介紹其本用法。

```
/*
  Arduino LSM9DS1 基本範例
  可由主選單[檔案]-[範例]-[Arduino_LSM9DS1]下的三個範例產生
  Simple Accelerometer （加速度計）
  Simple Gyroscope （陀螺儀）
  Simple Magnetometer （地磁計）
  三個範例大同小異，以下以加速計為例，另兩種則以註解方式補充在關鍵行數下方
*/

#include <Arduino_LSM9DS1.h> // 導入Arduino_LSM9DS1函式庫頭文件

// 設定腳位用途及模組初始化（只在電源啟動或重置時執行一次）
void setup() {
  Serial.begin(9600); // 設定序列埠通訊速度為9,600bps
  while (!Serial);
  Serial.println("Started");

  if (!IMU.begin()) { // 初始化IMU
    Serial.println("Failed to initialize IMU!");
    while (1);
  }

  Serial.print("Accelerometer sample rate = "); // 加速度計用
  // Serial.print("Gyroscope sample rate = "); // 陀螺儀用
  // Serial.print("Magnetic field sample rate = "); // 地磁計用
  
  Serial.print(IMU.accelerationSampleRate()); // 加速度計用，設定取樣速度
  // Serial.print(IMU.gyroscopeSampleRate()); // 陀螺儀用，設定取樣速度
  // Serial.print(IMU.magneticFieldSampleRate()); // 地磁計用，設定取樣速度
  
  Serial.println(" Hz"); // 加速度計用
  // Serial.println(" Hz"); // 陀螺儀用
  // Serial.println(" uT"); // 地磁計用
  
  Serial.println(); // 輸出空白行
  
  Serial.println("Acceleration in G's"); // 加速度計用
  // Serial.println("Gyroscope in degrees/second"); // 陀螺儀用
  // Serial.println("Magnetic Field in uT"); // 地磁計用
  
  Serial.println("X\tY\tZ"); // 輸出X Y Z 字串
}

// 設定無窮循環程式 （會一直依序重覆執行）
void loop() {
  float x, y, z; // 存放三軸值
  
  // 加速度計用，若加速度計備妥則讀入三軸加速度（重力）值
  if (IMU.accelerationAvailable()) {
    IMU.readAcceleration(x, y, z);
  
  // 陀螺儀用，若陀螺儀備妥則讀入三軸角加速值
  // if (IMU.gyroscopeAvailable()) {  
  //   IMU.readGyroscope(x, y, z);
  
  // 地磁計用，若地磁計備妥則讀入三軸地磁值
  // if (IMU.magneticFieldAvailable()) {
  //   IMU.readMagneticField(x, y, z);
  
    // 透過序列埠將三軸數值送到電腦，可使用[工具]下的「序列埠監控視窗」觀看數值，
    // 或透過「序列繪圖家」直接觀看三軸數值變化曲線圖，更為方便。
    Serial.print(x);
    Serial.print('\t');
    Serial.print(y);
    Serial.print('\t');
    Serial.println(z);
  }
}
```

附上序列埠繪圖家所得加速度變化，如圖Fig. 23-2所示。

![Arduino Nano 33 BLE Sense - LSM9DS1加速度計讀值](https://1.bp.blogspot.com/-iwX0vEFe2Ww/YV_45S4_HvI/AAAAAAAAE3c/OhI9i2N5TDkQWmVDHJ9pBnVLc4wFySYswCLcBGAsYHQ/s1658/iThome_Day_23_Fig_02.jpg)
Fig. 23-2 Arduino Nano 33 BLE Sense - LSM9DS1加速度計讀值。(OmniXRI整理繪製, 2021/10/8)

如果想要保留這些值再寫入其它外掛的Serial Flash或SD卡的人，可先開一個比較大的記憶區，例如float x_data[取樣頻率x取樣秒數]來暫存，當滿了就暫停取樣，待寫入完後，清空記憶區，再重新循環或結束取樣動作，就像前面章節介紹在作聲音取樣一樣。

這裡補充一下，IMU.begin()初始化時其預設值如下所示。
* 加速度計使用範圍 [-4,+4]g -/+0.122 mg，輸出頻率119Hz。
* 陀螺儀使用範圍 [-2000, +2000] dps +/-70 mdps，輸出頻率119Hz。
* 地磁計使用範圍 [-400, +400] uT +/-0.014 uT，輸出頻率20Hz。

如果使用在特殊用途需要大一點或小一點的範圍，Arduino_LSM9DS1函式庫目前並沒有提供，須自行手動修改下列程式，\libraries\Arduino_LSM9DS1\src\LSM9DS1.cpp，如果你不是很有把握，請不要隨意更動這些值。更多LSM9DS1完整函式用法可參考官網[說明文件](https://www.arduino.cc/en/Reference/ArduinoLSM9DS1)。

```
/*
  Arduino Nano 33 BLE Sense - LSM9DS1 IMU（九軸運動感測器）初始化函式
*/

int LSM9DS1Class::begin()
{
  _wire->begin();

  // reset
  writeRegister(LSM9DS1_ADDRESS, LSM9DS1_CTRL_REG8, 0x05);
  writeRegister(LSM9DS1_ADDRESS_M, LSM9DS1_CTRL_REG2_M, 0x0c);

  delay(10);

  if (readRegister(LSM9DS1_ADDRESS, LSM9DS1_WHO_AM_I) != 0x68) {
    end();

    return 0;
  }

  if (readRegister(LSM9DS1_ADDRESS_M, LSM9DS1_WHO_AM_I) != 0x3d) {
    end();

    return 0;
  }

  writeRegister(LSM9DS1_ADDRESS, LSM9DS1_CTRL_REG1_G, 0x78); // 119 Hz, 2000 dps, 16 Hz BW
  writeRegister(LSM9DS1_ADDRESS, LSM9DS1_CTRL_REG6_XL, 0x70); // 119 Hz, 4G

  writeRegister(LSM9DS1_ADDRESS_M, LSM9DS1_CTRL_REG1_M, 0xb4); // Temperature compensation enable, medium performance, 20 Hz
  writeRegister(LSM9DS1_ADDRESS_M, LSM9DS1_CTRL_REG2_M, 0x00); // 4 Gauss
  writeRegister(LSM9DS1_ADDRESS_M, LSM9DS1_CTRL_REG3_M, 0x00); // Continuous conversion mode

  return 1;
}
```

有了上面的基礎，相信大家已經可以聯想到，這和語音（喚醒詞）辨識非常相似，都是連續信號，只是資料從一維變三維，所以應該很容易可以用Edge Impulse這樣的工具才完成「動作辨識」。沒錯，這個部份就留待下回分解。

ps. 為讓文章更活潑傳達硬梆梆的技術內容，所以引用了經典電影「食神」的橋段，希望小弟戲劇性的二創不會引起電影公司的不悅，在此對星爺及電影公司致上崇高的敬意，敬請見諒。

參考連結

[ST LSM9DS1 iNEMO inertial module：3D magnetometer, accelerometer, gyroscope](https://www.st.com/en/mems-and-sensors/lsm9ds1.html)  
[[ST LSM9DS1 iNEMO inertial module 資料手冊](https://www.st.com/resource/en/datasheet/lsm9ds1.pdf) 
[Arduino LSM9DS1 library說明文件](https://www.arduino.cc/en/Reference/ArduinoLSM9DS1)
