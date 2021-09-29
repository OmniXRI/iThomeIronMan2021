## [Day 15] 在Arduino IDE中用Arm CMSIS 牛刀小試一下

在[[Day 14] tinyML開發框架(二)：Arm CMSIS 簡介](https://ithelp.ithome.com.tw/articles/10273236)已初步幫大家介紹了Arm CMSIS的基本框架及重要函式庫，其中Arm CMSIS-DSP中有很多矩陣及機器學習相關計算，這和tinyML運算效能有很大關係。如果你是喜歡把機器性能榨乾的朋友，或者是吃完咖哩飯還想更了解廚師的到底是用了什麼步驟和摻了什麼祕方的人，那一定要花點時間了解一下這些底層運作方式。但如果你只是想好好點個喜愛口味的咖哩飯來享用，或者是只想學習tinyML如何快速應用的朋友，則可忽略過這個章節。

在CMSIS-DSP中和tinyML最相關的莫過於矩陣演算，常用的矩陣加法、減法、乘法、轉置和反矩陣計算都有，還有依不同數值精度，提供有浮點數(f16, f32, f64)及定點數(q7, q15, q31)不同函式（註：不是所有函數都有6種數值精度）。其基本計算如圖Fig. 15-1所示。更完整的函式使用可參考官網[CMSIS-DSP-Reference-Matrix Functions](https://arm-software.github.io/CMSIS_5/DSP/html/group__groupMatrix.html)。

![Arm CMSIS-DSP常用矩陣函數](https://1.bp.blogspot.com/-i9GpaXvmisk/YVQkOsH0dMI/AAAAAAAAEyA/gQUAXHGOIjcuRScpDoysw2V2ctFmQL0DwCLcBGAsYHQ/s1658/iThome_Day_15_Fig_01.jpg)
Fig. 15-1 Arm CMSIS-DSP常用矩陣函數。(OmniXRI整理製作, 2021/9/29)

在CMSIS-DSP函式中，矩陣相關函式並不是直接使用指標存取資料，而是另外定了一個arm_matrix_instance_xxx結構(xxx表數值精度）來存取，其中包含列數、行數及資料起始位置，而不同的數值精度提供了不同的結構來呼叫。使用前需直接給定或者透過初始化函式來完成。寫入和讀出都透過指標來完成，可參考下列程式段說明。

```
// CMSIS-DSP 矩陣運算時所需的資料結構原始定義
// 其定義已宣告在arm_math.h中的dsp/matrix_functions.h
// 不用再主程式或子程式中重新宣告
typedef struct
{
 uint16_t numRows; // 矩陣列數
 uint16_t numCols; // 矩陣行數
 float32_t *pData; // 矩陣資料起點指標
} arm_matrix_instance_f32; // 32bit浮點數矩陣實例結構名稱

// 宣告原始資料內容方式，可為f16, f32, q15, q31等格式陣列
float32_t data[資料數量] = {資料內容};

// 取得資料所在指標方式
float32_t *pData = &data;

// 從原始資料轉換到矩陣實例，以利矩陣相關運算。
arm_matrix_instance_f32 S = {nRows, nColumns, pData};

// 或者可使用初始化函數將原始資料轉入矩陣實例中，其格式可為f16, f32, q15, q31四種。
arm_mat_init_f32 (arm_matrix_instance_f32 *  S,
		          uint16_t     nRows,
		          uint16_t     nColumns,
		          float32_t *  pData 
) 

// 矩陣相關計算完成後，取出矩陣實例S座標(i,j)數值方式
value = S.pDate[i*nColumns + j];
```

在Github上的Arm CMSIS開源碼主要不是給所有MCU及開發工具用的，所以有時要自己手動修改一些設定值，原則上可把這些/include下的頭文件(*.h)及/source下的程式碼(*.c)直接複制到各家MCU開發工具上，再進行編譯即可，有些可能需要修改一些參數，詳見各函式庫的readme.md說明。

由於Arduino IDE預設並沒有安裝Arm CMSIS相關函式庫，所以這裡可以參考[[Day 13] tinyML開發框架(一)：TensorFlow Lite Micro初體驗(下)](https://ithelp.ithome.com.tw/articles/10273062)圖Fig. 13-2的作法，在程式庫管理員中查詢「CMSIS」，再把Arduino_CMSIS-DSP安裝好。不過這裡並不會在主選單[檔案]─[範例]中產生任何參考範例，且Arduino官網的參考文件也被撤掉了，但是不用擔心，它還是有把相關檔案及路徑都設好了。只需開啟一個新的空白範例，加入"arm_math.h"即可將所有相關的函式庫也一併導入。

下面為一個簡單的矩陣乘法測試，準備兩個矩陣資料，相乘後得到X=AxB，再將結果從序列監視控制台秀出結果。

```
/*
  CMSIS-DSP_Test.ino （for Arduino）
  測試在Arduino IDE下使用Arm CMSIS-DSP函式庫
  使用前需先安裝Arduino_CMSIS-DSP函式庫（最新版本為5.7版）
  求矩陣A乘以矩B得到結果X = A x B
  
  程式作者：OmniXRI Jack, 2021/9/29
*/

#include "arm_math.h" // 導入CMSIS-DSP中最主要的函式頭文件

// 宣告一組32 bit浮點數陣列A，作為輸入值，大小為1x4
const float32_t A_f32[4] =
{
  1.0, 2.0, 3.0, 4.0
};

// 宣告一組32 bit浮點數陣列B，作為輸入值，大小為4x4
const float32_t B_f32[16] =
{
  1.0,  2.0,  3.0,  4.0,
  5.0,  6.0,  7.0,  8.0,
  9.0,  10.0, 11.0, 12.0,
  13.0, 14.0, 15.0, 16.0,
};

// 宣告一組32 bit浮點數陣列X，存放輸出值，大小為1x4
float32_t X_f32[4];

arm_status status;  // CMSIS-DSP運算結果狀態
arm_matrix_instance_f32 A; // 宣告CMSIS-DSP 32bit浮點數矩陣實例指標
arm_matrix_instance_f32 B; // 宣告CMSIS-DSP 32bit浮點數矩陣實例指標
arm_matrix_instance_f32 X; // 宣告CMSIS-DSP 32bit浮點數矩陣實例指標
uint32_t srcRows, srcColumns; // 宣告行(Columns)、列(Rows)數變數

void setup() {
  Serial.begin(9600); // 初始化序列埠，方便監看輸出結果
  
  // 將列陣A_f32初始化為矩陣實例A
  srcRows = 1;
  srcColumns = 4;
  arm_mat_init_f32(&A, srcRows, srcColumns, (float32_t *)A_f32);
  
  // 將列陣B_f32初始化為矩陣實例B
  srcRows = 4;
  srcColumns = 4;
  arm_mat_init_f32(&B, srcRows, srcColumns, (float32_t *)B_f32);
  
  // 將列陣X_f32初始化為矩陣實例X
  srcRows = 1;
  srcColumns = 4;
  arm_mat_init_f32(&X, srcRows, srcColumns, (float32_t *)X_f32);
}

void loop() {
  // 計算矩陣A乘以矩陣B並將結果存放在X中
  status = arm_mat_mult_f32(&A, &B, &X);

  // 將計算結果從序列埠監控視窗印出
  // 得到字串 X[0,0] = 90.00	X[0,1] = 100.00	X[0,2] = 110.00	X[0,3] = 120.00	
  for(int i=0; i<srcRows; i++){      // 指定結果矩陣列數
    for(int j=0; j<srcColumns; j++){ // 指定結果矩陣行數
      Serial.print("X[");
      Serial.print(i);
      Serial.print(",");
      Serial.print(j);
      Serial.print("] = ");
      Serial.print(X.pData[j]); // 將矩陣實例中的結果取出，轉回float32_t格式
      Serial.print("\t"); // 列印TAB（跳格）指令
    }
    Serial.print("\n"); // 列印換行指令
  } 

  delay(3000); // 為方便觀察，讓程式重覆每隔3秒重新執行一次，通常會以 While(1); 程式碼將程式停住。
}
```

其它矩陣相關函式可自己玩看看，這裡就不多作說明了。

參考連結

[Arm CMSIS DSP Software Library](https://arm-software.github.io/CMSIS_5/DSP/html/index.html)  
[Arduino-Reference-Libraries-Arduino_CMSIS-DSP](https://www.arduino.cc/reference/en/libraries/arduino_cmsis-dsp/) （註：官方說明文件已停用）
