## [Day 16] TFLM + BLE Sense + MP34DT05 就成了迷你智慧音箱(上)

學了半個月終於要端出「爆漿瀨尿蝦牛丸」了嗎？要開始讓大家體會一下牛肉(MCU)的鮮、瀨尿蝦(AI)的甜，及摻在一起比老鼠斑有過之而無不及的味道了嗎？各位施主，這又是一個介於是和不是的問題了，只有耐心看完才能感受這其中的酸甜苦辣。先說結論，這個章節確實要幫大家介紹如何整合TensorFlow Lite Microcontroller（以下簡稱**TFLM**）加Arduino IDE加Arduino Nano 33 BLE Sense開發板（以下簡稱**BLE Sense**）及板上微機電麥克風**MP34DT05-A**來運行一個智能音箱常見的基本語音命令辨識及控制的**micro_speech**範例。但看完這個章節猜想你有極大可能會想要直接棄番不再追下去。不過請不要衝動，不經一番寒徹骨焉得梅花撲鼻香，後面章節還有更好的解決方案，今天就當先學個概念，後面就會操作的更得心應手了。

在[[Day 11] 讓tinyML聽見你的呼喚](https://ithelp.ithome.com.tw/articles/10271661)已初步介紹過如何透過**BLE Sense**收集到板子上的微機電麥克風**MP34DT05-A**所接收到的聲音訊號。在[[Day 12]](https://ithelp.ithome.com.tw/articles/10272320)和[[Day 13]](https://ithelp.ithome.com.tw/articles/10273062)「tinyML開發框架(一)：TensorFlow Lite Micro初體驗」中，已分別幫大家介紹如何在**TFLM**上建立、訓練、優化模型、準備好MCU所需格式模型參數檔案，及如何使用**Arduino IDE**和**BLE Sense**將備妥的模型完整運行並推論起來。沒看過前面章節的朋友，麻煩先補一下課並把Arduino IDE工作環境建好，以免後面看的一頭霧水。接下來我們就順著**TFLM**官網介紹的[微語音(**micro_speech**)範例](https://github.com/tensorflow/tflite-micro/tree/main/tensorflow/lite/micro/examples/micro_speech)來說明如何把這兩樣素材結合在一起，完成一個類似智能音箱的語言命令的動作，說「YES」及「NO」來控制綠色和紅色LED點亮的功能。（如圖Fig. 16-1所示）

![TensorFlow Lite Microcontroller提供之micro_speech聲音辨識範例運行結果](https://1.bp.blogspot.com/-Z_lbMlnS3Uc/YVXdC5w0bKI/AAAAAAAAEyI/iwexn-dc-dEplkVsyTGW3QrOWZNTxCoTgCLcBGAsYHQ/s480/iThome_Day_16_Fig_01.gif)  
Fig. 16-1 TensorFlow Lite Microcontroller提供之micro_speech聲音辨識範例運行結果。（[圖片來源](https://github.com/tensorflow/tflite-micro/blob/main/tensorflow/lite/micro/examples/micro_speech/images/animation_on_arduino.gif)）

要運行的這個範例不難，假設你已照著[[Day 13] tinyML開發框架(一)：TensorFlow Lite Micro初體驗(下) ](https://ithelp.ithome.com.tw/articles/10273062)圖13-2把Arduino_TensorFlowLite函式庫安裝好了，此時只要新增一個micro_speech範例（如下程式所示），經過編譯、上傳，接著開啟序列埠監控視窗，對著BLE Sense板子說出「YES」或「NO」，就能看到對應的綠色、紅色LED點亮，同時也會於監控視窗上看到對應字串。如果無法順利辨識就把板子靠近嘴巴一點，或者多說幾次來改善辨識結果。

```
/* 
  Arduino_TensorFlowLite micro_speech.ino 範例程式  
  在Arduino IDE安裝好Arduino_TensorFlowLite函式庫後，
  點擊主選單[檔案]-[範例]-[Arduino_TensorFlowLite]-[micro_speech]產生。
*/

// 導入相關函式庫頭文件
#include <TensorFlowLite.h> // 導入TensorFlowLite函式庫頭文件
#include "main_functions.h" 
#include "audio_provider.h"
#include "command_responder.h"
#include "feature_provider.h"
#include "micro_features_micro_model_settings.h"
#include "micro_features_model.h"
#include "recognize_commands.h"
#include "tensorflow/lite/micro/micro_error_reporter.h"
#include "tensorflow/lite/micro/micro_interpreter.h"
#include "tensorflow/lite/micro/micro_mutable_op_resolver.h"
#include "tensorflow/lite/schema/schema_generated.h"
#include "tensorflow/lite/version.h"

// 宣告全域變數及命名空間，初始化相關指標及變數，方便後面其它子程式引用
namespace {
tflite::ErrorReporter* error_reporter = nullptr;
const tflite::Model* model = nullptr;
tflite::MicroInterpreter* interpreter = nullptr;
TfLiteTensor* model_input = nullptr;
FeatureProvider* feature_provider = nullptr;
RecognizeCommands* recognizer = nullptr;
int32_t previous_time = 0;

// 配置記憶體空間，包含聲音輸入、模型參數等
constexpr int kTensorArenaSize = 10 * 1024;
uint8_t tensor_arena[kTensorArenaSize];
int8_t feature_buffer[kFeatureElementCount];
int8_t* model_input_buffer = nullptr;
}  

// 設定腳位用途及模組初始化（只在電源啟動或重置時執行一次）
void setup() {
  // 建立除錯日誌
  static tflite::MicroErrorReporter micro_error_reporter;
  error_reporter = &micro_error_reporter;

  // 載入模型
  model = tflite::GetModel(g_model);  
  if (model->version() != TFLITE_SCHEMA_VERSION) {
    TF_LITE_REPORT_ERROR(error_reporter,
                         "Model provided is schema version %d not equal "
                         "to supported version %d.",
                         model->version(), TFLITE_SCHEMA_VERSION);
    return;
  }

  // 不使用全操作解析器，改使用微操作解析器只加入模型有用到的元素
  // 目前只加入DepthwiseConv2D, FullyConnected, Softmax, Reshape
  // tflite::AllOpsResolver resolver;
  // NOLINTNEXTLINE(runtime-global-variables)
  static tflite::MicroMutableOpResolver<4> micro_op_resolver(error_reporter);
  if (micro_op_resolver.AddDepthwiseConv2D() != kTfLiteOk) {
    return;
  }
  
  if (micro_op_resolver.AddFullyConnected() != kTfLiteOk) {
    return;
  }
  
  if (micro_op_resolver.AddSoftmax() != kTfLiteOk) {
    return;
  }
  
  if (micro_op_resolver.AddReshape() != kTfLiteOk) {
    return;
  }

  // 建立直譯器以執行模型相關動作
  static tflite::MicroInterpreter static_interpreter(
      model, micro_op_resolver, tensor_arena, kTensorArenaSize, error_reporter);
  interpreter = &static_interpreter;

  // 配置張量所需記憶體
  TfLiteStatus allocate_status = interpreter->AllocateTensors();
  if (allocate_status != kTfLiteOk) {
    TF_LITE_REPORT_ERROR(error_reporter, "AllocateTensors() failed");
    return;
  }

  // 建立輸出入
  model_input = interpreter->input(0);
  if ((model_input->dims->size != 2) || (model_input->dims->data[0] != 1) ||
      (model_input->dims->data[1] !=
       (kFeatureSliceCount * kFeatureSliceSize)) ||
      (model_input->type != kTfLiteInt8)) {
    TF_LITE_REPORT_ERROR(error_reporter,
                         "Bad input tensor parameters in model");
    return;
  }
  
  model_input_buffer = model_input->data.int8; // 宣告輸入緩衝區為INT8資料型態

  // 建立特徵提供器及相關參數
  // NOLINTNEXTLINE(runtime-global-variables)
  static FeatureProvider static_feature_provider(kFeatureElementCount,
                                                 feature_buffer);
  feature_provider = &static_feature_provider;

  static RecognizeCommands static_recognizer(error_reporter);
  recognizer = &static_recognizer;

  previous_time = 0; // 清除前次計時
}

// 設定無窮循環程式 （會一直依序重覆執行）
void loop() {
  // 擷取目前時間的聲音圖譜
  const int32_t current_time = LatestAudioTimestamp(); 
  int how_many_new_slices = 0;
  TfLiteStatus feature_status = feature_provider->PopulateFeatureData(
      error_reporter, previous_time, current_time, &how_many_new_slices);
  if (feature_status != kTfLiteOk) {
    TF_LITE_REPORT_ERROR(error_reporter, "Feature generation failed");
    return;
  }
  previous_time = current_time;

  // 假設沒有取得任何聲音就結束此次動作
  if (how_many_new_slices == 0) {
    return;
  }

  // 複製聲音資料到模型輸入端點
  for (int i = 0; i < kFeatureElementCount; i++) {
    model_input_buffer[i] = feature_buffer[i];
  }

  // 運行推理
  TfLiteStatus invoke_status = interpreter->Invoke();
  if (invoke_status != kTfLiteOk) {
    TF_LITE_REPORT_ERROR(error_reporter, "Invoke failed");
    return;
  }

  // 取得輸出值的指標
  TfLiteTensor* output = interpreter->output(0);

  // 經由辨識結果決定輸出命令字串及LED亮燈變化
  const char* found_command = nullptr;
  uint8_t score = 0;
  bool is_new_command = false;
  TfLiteStatus process_status = recognizer->ProcessLatestResults(
      output, current_time, &found_command, &score, &is_new_command);      
  if (process_status != kTfLiteOk) {
    TF_LITE_REPORT_ERROR(error_reporter,
                         "RecognizeCommands::ProcessLatestResults() failed");
    return;
  }

  // 透過序列埠回報到電腦，包含目前時間、辨識命令結果、置信分數及是否為新命令等
  RespondToCommand(error_reporter, current_time, found_command, score,
                   is_new_command);
}
```

但是…，事情真的有這麼簡單和順利嗎？為什麼板子老是聽錯或聽不懂「YES」、「NO」？我要怎麼才能把「YES」、「NO」改成中文的「開」、「關」或「紅」、「綠」甚至是「電燈亮」「電燈滅」這樣的短詞呢？我不只要改變LED顏色，可以不可改成推動電氣設備開關呢？這可能就要從頭說起了，包括下列幾大步驟。
* 範例程式組成
* 產生資料集
* 訓練及優化模型
* 佈署、推理及輸出

不想深入了解的朋友，想讓生活多點樂趣的朋友，可以先跳過下面的部份，等後面章節再來學習相同的功能但更輕鬆的作法。

## 範例程式組成

從圖Fig. 16-2就可看出，這個範例不是一個ino檔（主程式）就能運作的，還要有另外的22個檔案才能將這個範例完整執行起來，雖然每個程式的份量不是很多，但若想深人了解的朋友可能也要花上不少時間。不過我猜想大家不一定想全盤了解，而只是關心訓練好的模型怎麼放進MCU及如何控制實體家電而不只是板子上的LED。所以這裡僅先點出除了主程式外另二個重點程式，arduino_command_responder.cpp及micro_features_model.cpp。其它的就留給大家自行研究了。

![Arduino TensorFlowLite micro_speech範例程式組成](https://1.bp.blogspot.com/-PCvf468ggx4/YVYAPeoTZRI/AAAAAAAAEyQ/LPIYesGs_II9jLQZJDk4c05szMIOa5CygCLcBGAsYHQ/s1658/iThome_Day_16_Fig_02.jpg)
Fig. 16-2 Arduino TensorFlowLite micro_speech範例程式組成。(OmniXRI整理製作, 2021/9/30)

首先介紹arduino_command_responder.cpp，這段程式負責回應語音辨識的結果，即輸出結果字串和控制LED點滅。在BLE Sense板子上有一個橘色LED(LED_BUILDIN)和一個RGB三色LED(LEDR, LEDG, LEDB)。但其點亮方式剛好相反，前者是給HIGH點亮，而後者則是給LOW點亮，控制時要特別注意。目前模型推論後共會產生四種結果及燈號。
* 沒有聲音(Silence)或背景音。三色LED全滅，橘色LED交替閃爍。
* 不明(unknow)，對應'u'命令，點亮藍色LED(LEDB)。
* Yes，對應'y'命令，點亮綠色LED(LEDG)。
* No，對應'n'命令，點亮紅色LED(LEDR)。

```
/* 
  arduino_command_responder.cpp
  語音（命令）辨識結果輸出回應（控制）程式
  將辨識結果字串從序列埠傳回電腦。
  若沒有聲音則會反轉橘色LED(LED_BUILDIN)，若非「YES」「NO」則亮藍色LED(LEDB)，
  若有偵測到「YES」「NO」則點亮對應的綠色(LEDG)和紅色LED(LEDR)。
*/
#include "Arduino.h"

void RespondToCommand(tflite::ErrorReporter* error_reporter,
                      int32_t current_time, const char* found_command,
                      uint8_t score, bool is_new_command) {
  static bool is_initialized = false; // 是否已初始化旗標
  
  // 若尚未初始化
  if (!is_initialized) { 
    // 設定四個LED腳位為輸出
    pinMode(LED_BUILTIN, OUTPUT);
    pinMode(LEDR, OUTPUT);
    pinMode(LEDG, OUTPUT);
    pinMode(LEDB, OUTPUT);

    // 使其初始皆為不亮狀態
    digitalWrite(LEDR, HIGH);
    digitalWrite(LEDG, HIGH);
    digitalWrite(LEDB, HIGH);
    
    // 設初始化旗標為真
    is_initialized = true; 
  }
  
  static int32_t last_command_time = 0; // 最後一次命令時間
  static int count = 0; // 計數器
  static int certainty = 220; // 確定值

  // 若為新命令
  if (is_new_command) {
    // 輸出結果字串到序列埠，包含命令結果、置信分數和目前時間。
    TF_LITE_REPORT_ERROR(error_reporter, "Heard %s (%d) @%dms", found_command,
                         score, current_time);
                         
    // 若為Yes命令則點亮綠色LED
    if (found_command[0] == 'y') {
      last_command_time = current_time; // 記錄最後時間
      digitalWrite(LEDG, LOW); // 點亮綠色LED
    }
    
    // 若為No命令則點亮紅色LED
    if (found_command[0] == 'n') {
      last_command_time = current_time; // 記錄最後時間
      digitalWrite(LEDR, LOW); // 點亮紅色LED
    }

    // 若為不明(unknow)命令則點亮藍色LED
    if (found_command[0] == 'u') {
      last_command_time = current_time; // 記錄最後時間
      digitalWrite(LEDB, LOW); // 點亮藍色LED
    }
  }

  // 若最後時間不為零且超過3秒，則熄滅所有LED
  if (last_command_time != 0) {
    if (last_command_time < (current_time - 3000)) {
      last_command_time = 0;
      digitalWrite(LED_BUILTIN, LOW);
      digitalWrite(LEDR, HIGH);
      digitalWrite(LEDG, HIGH);
      digitalWrite(LEDB, HIGH);
    }
    
    // 若時間未超過3秒則返回
    return;
  }

  // 若為沒有聲音（或背景音）則計數器加1，橘色LED反轉。
  ++count; // 計數器加1
  
  if (count & 1) {
    digitalWrite(LED_BUILTIN, HIGH); // 點亮橘色LED
  } else {
    digitalWrite(LED_BUILTIN, LOW); // 熄滅橘色LED
  }
}
```

解釋完如何將辨識結果和LED輸出結果的關連後，相信大家很容易就可聯想到，只要使用板子上其它通用輸出入接點(GPIO)連接到繼電器，再接上一般110V的家電（只有開和關的那種），就可以控制其電源開啟和關閉了。

在這裡補充一個使用狀況，由於板上的麥克風還滿靈敏的，所以當環境中有像電風扇或背景雜音較大的情況下，會發現藍色LED一直亮著，表示有一直有收到一段聲音，但聽不懂是「YES」或「NO」，此時建議到比較安靜的環境再測試，效果會好些。

另外關於容易聽錯、想換其它語音控制命令，這和資料集產生方式及模型結構、訓練、調參、轉換、優化很有關聯，當然完成後如何佈署更是我們關心的重點，這些就留待下回分解。

參考連結

[TensorFlow Lite Microcontroller - Micro Speech 微語音辨識](https://github.com/tensorflow/tflite-micro/tree/main/tensorflow/lite/micro/examples/micro_speech)
