## [Day 14] tinyML開發框架(二)：Arm CMSIS 簡介

在[[Day 10] tinyML整合開發平台介紹](https://ithelp.ithome.com.tw/articles/10269746)有提到小型AI(tinyML)應用程式開發框架(Fig. 10-3)在推論函式庫中可能會使用到**CMSIS-NN**，這是什麼？為什麼它能起到這重要的地位呢？接下來就幫大家簡單介紹一下。

Arm CMSIS全名Common Microcontroller Software Interface Standard，顧名思義就是Arm為了讓自家MCU (IP)能跨廠牌（如STM, NXP, 新唐、盛群等）、跨開發平台（MDK, Keil, IAR, eIQ, STM32CubeIDE, Arduino IDE等）、跨家族（Cortex-M0/M0+/M1/M3/M4/M7/M23/M33/M35P/M55/A5/A7/A9）所推出來的軟體開發介面，減少使用者的學習曲線。它是一種硬體抽象層(Hardware Abstraction Layer, HAL)的概念，就是不直接驅動硬體而採間接驅動的程式寫作概念，這樣就可達成上位軟體寫作統一化的好處，類似使用OpenGL, OpenCL跨GPU開發繪圖軟體一樣。

由Fig. 14-1圖可看出CMSIS的基本架構，最下方為實際硬體，包括微處理器(CPU,暫存器、旗標等)、通訊週邊（I2C, SPI, UART等）、特殊週邊（如ADC, DAC, PWM等）及一些除錯相關的硬體。中間層則負責硬體抽象描述，處理所有核心計算及週邊信號處理。上一層則為進階整合性功能，如即時作業系統、神經網路運算等，最後才是組合成完整的應用程式碼。目前CMSIS可以支援的元件非常多，有6009項，其中Cortex-M4最多，佔了39.1%，Cortex-M3次之，佔25.6%，第三為Cortex-M0+，佔14.9%，三者合計佔79.6%。而支援的開發板高達441種，相關軟體也多達682項，可見其普及性及便捷性。

![ARM CMSIS架構圖](https://1.bp.blogspot.com/-hUmF0DiGW7c/YVKPqUKGX6I/AAAAAAAAExw/HRRn5cGv9-I12ZBGFjaLDGeAsnI_HpawgCLcBGAsYHQ/s1658/iThome_Day_14_Fig_01.jpg)
Fig. 14-1 ARM CMSIS架構圖。(OmniXRI整理製作, 2021/9/28)

Arm CMSIS為[開源項目](https://github.com/ARM-software/CMSIS_5)，主要組成元件(Components)如下所示。
* **核心(Core)**：又分為Cortex-M用的Core(M)和Cortex-A用的Core(A)，提供標準API及核心運算指令。
* **驅動程式(Driver)**：用於連接實際硬體介面的中間軟體(Middleware)，以實現軟體堆疊(Stack)。
* **數位信號處理器(Digital Signal Processor, DSP)**：有超過60個以上函式，可處理定點(q7, q15, q31)、32 bit浮點及單指令多資料流(SIMD)指令應用。
* **神經網路(Neural Network, NN)**：高性能神經網路運算函式庫，執行tinyML、深度學習重要函式庫。
* **即時作業系統(Real Time Operation System, RTOS)**：又分為V1(Cortex-M0/M0+/M3/M4/M7)和V2(Cortex-M/A5/A7/A9)。
* **包裝(Pack)**：描述軟體元件、設備參數和開發板支持的交付機制。它簡化了軟件重用和產品生命週期管理 (PLM)。
* **建置(Build)**：一組可提高生產力的工具、軟體框架和工作流程，例如使用連續整合(CI)。 
* **系統視角描述（System View Description, SVD）**：可用於在除錯工具及創建外設感知設備的描述。 
* **除錯存取埠(Debug Access Port, DAP)**：提供除錯存取埠介面所需的的韌體。 
* **區域(Zone)**：定義方法來描述系統資源並將這些資源劃分為多個專案和執行區域。 

透過前面章節及上圖Fig.14-1可知，Arm Cortex-M4(F)是最適合用來開發tinyML的家族，因為它價格及性能在整個Cortex家族中間，相較M0/M0+/M3等低階MCU配置有較多的程式碼區(Flash)和隨機記憶體(SRAM)可用來儲放模型及參數，同時支援定點、浮點(FP16/FP32)及整數(INT8)運算，也支援單指令週期DSP及SIMD指令集，可讓**CMSIS-DSP**數位信號處理（含高階數學運算及機器學習常用統計算法）和**CMSIS-NN**神經網路函式庫發揮更多功效。前面章節使用的Arduino Nano 33 BLE Sense開發板的主晶片內核正是Arm Cortex-M4F。

接下來就幫大家介紹一下**CMSIS-DSP**和**CMSIS-NN**的主要元件和功能。

## CMSIS-DSP 數位信號處理函式庫

早期8/16 bit MCU並沒有浮點（少部份有定點）運算能力或硬體乘法器、除法器，基本上也不支援像矩陣計算這類高階的運算，頂多是靠很多層的迴圈、乘法及加法來完成。但遇到像數位信號處理時經常用到快速傅立葉轉換(FFT)算法，需要大量乘加演算時，此時就只能依靠專用DSP處理器來完成。當Cortex-M3問世後，開始有了多週期的乘加一體(Multiply-Accumulate, MAC, y=ax+b)指令，到了Cortex-M4上市後，開始指供單週期的乘加及SIMD指令集（又稱DSP指令集），至此，許多原來需要專用DSP處理器的工作就慢慢被取代了。但是一般程式設計師並沒有強大的數學能力，更不懂如何有效地發揮MCU加速計算的能力，所以當遇到像馬達控制、聲音信號處理等應用時，還是會選擇專用的DSP處理器。因此為了讓程式設計師能更簡單的開發DSP相關產品，於是CMSIS-DSP就此誕生。以下就簡單介紹這個函式庫提供的主要函式分類，更完整的內容可參考[官網文件說明](https://arm-software.github.io/CMSIS_5/latest/DSP/html/index.html)。對於偏屬傳統機器學習相關或信號預處理的tinyML應用，較會使用到這個部份的函式庫。

CMSIS-DSP函式庫相關函式：
* 基本數學函數（向量加法、乘法、點積及位元運算等）
* 快速數學函數（sine, cosine, 均方根、定點除法等）
* 複數數學函數（共軛複數、複數點積、乘法、絕對值等）
* 濾波函數（IIR, FIR, LMS等）
* 矩陣函數（矩陣加法、減法、乘法、轉置、反矩陣等）
* 變換函數（FFT, DCT等）
* 馬達控制函數（PID, Park, Clarke, Sine, Cosine等）
* 統計函數（絕對最大、最小，熵、平均值、開平方、次方、均方根、標準差、變異數等）
* 支持函數（向量排序、複製、填充、加權總合、整數轉換等）
* 插值函數（線性、雙線性及立方內插等）
* 支持向量機函數 (SVM)（線性、多項式、RBF、Sigmoid等）
* 貝葉斯分類器函數（f16, f32等）
* 距離函數（Float, Boolean等）
* 四元數函數（四元轉換、反轉、正規化、點積等）

## CMSIS-NN 神經網路函式庫

在[[Day 05] tinyML與卷積神經網路(CNN)](https://ithelp.ithome.com.tw/articles/10266628)時已幫大家介紹過卷積神經網路常見基本元素（如圖Fig. 5-2），而這也是CMSIS-NN的主要元素。在[[Day 10] tinyML整合開發平台介紹](https://ithelp.ithome.com.tw/articles/10269746)時（如圖Fig. 10-3）也曾提及TensorFlow Lite for Microcontroller (TFLM) 會以CMSIS-NN函式庫為基礎來產出對應的程式碼，方便MCU端可以執行。雖然CMSIS-NN也可運行在沒有浮點運算及SIMD指令的Cortex-M0/M0+/M3 CPU上，但其執行效率會和M4以上家族差上數倍到數十倍（含CPU工作時脈速度提升）。

以CMSIS-NN函式庫發展出的tinyML應用，包括有：
* 多層全連接的深度神經網路(Deep Neural Network, DNN)：如智慧感測器、手勢（動作）辨識、喚醒詞偵測等應用，
＊卷積神經網路(Convolutional Neural Network, CNN)：如影像分類、物件偵測等視覺應用。

CMSIS-NN函式庫相關函式：
* 卷積函數（1x1, 3x3，定點7位、定點15位、有號8位等）
* 激活函數（sigmoid, tanh, relu等）
* 全連接層函數（定點7位、有號8位、無號8位）
* SVDF層函數（有號8位）
* 池化函數（平均、最大，定點7位、有號8位）
* Softmax函數（定點7位、有號8位、無號8位）
* 基本數學函數（元素相加、相乘，重塑形等）

![Arm CMSIS-NN函式庫架構圖](https://1.bp.blogspot.com/-T5zNsaQsH9A/YVM9Dz68ZjI/AAAAAAAAEx4/igLgMFNeYD0e6JQA3rHEDPWWQk0ZkMJQQCLcBGAsYHQ/s1658/iThome_Day_14_Fig_02.jpg)
Fig. 14-2 Arm CMSIS-NN函式庫架構圖。(OmniXRI整理繪製, 2021/9/28)

至於如何直接使用CMSIS-NN來開發CNN，可參考下列文章介紹，或者等後面章節再補充介紹。假若你不想這麼深入了解底層運作方式，那使用TFLM或其它tinyML雲端一站式整合型開發平台即可。

參考連結

[Arm Common Microcontroller Software Interface Standard (CMSIS)文件說明](https://arm-software.github.io/CMSIS_5/latest/General/html/index.html)  
[Arm CMSIS-5 Github開源碼](https://github.com/ARM-software/CMSIS_5)  
[Arm Developer - Converting a Neural Network for Arm Cortex-M with CMSIS-NN](https://developer.arm.com/documentation/102591/0000)  
[Arm Developer - Deploying a Caffe Model on OpenMV using CMSIS-NN](https://developer.arm.com/documentation/ARM0629486814402664/0100)  
[Arm Youtube - Image recognition on Arm Cortex-M with CMSIS-NN in 5 steps](https://youtu.be/EkYp0glSenE)  
[Arm-Software ML-exsamples - CMSIS-NN CIFAR10 Example](https://github.com/ARM-software/ML-examples/tree/master/cmsisnn-cifar10)  
[Arm CMSIS-NN: Efficient Neural Network Kernels for Arm Cortex-M CPUs](https://slideplayer.com/slide/17731119/) 
