# 2019CVFX_Homework3_Team5

## Generate Image with GANPaint

#### 1. 在照片中新增樹
<img src="./Images/tree.PNG" width="600px" />
我們試著在房子周圍以及天空中都畫上樹，產生出來的結果大部分看起來都很成功，也很合理。
但我們發現，若想在天空中畫樹，最好從照片邊緣開始畫，
這是因為有些照片在取景的時候會有那種周圍有樹遮蔽的效果（從樹叢中拍出去的感覺），所以這種情況下樹出現在「天空」中是合理的。<br>
<img src="./Images/tree_float.PNG" width="600px" />
若是把樹畫在天空正中間，有時候會有奇怪的「飄浮樹」產生。
此外，我們也發現樹的顏色會根據照片整體的色調而改變。
在較為明亮、夏天的照片中，樹的顏色是很飽合的綠色。
但若是叫為蕭瑟的照片，會讓產生出來的樹偏向灰或紅色。<br>

#### 2. 在照片中加上草、移除屋頂
<img src="./Images/grass.PNG" width="600px" />
草可以很完美得長在地上和建築物邊緣，效果很像物理館(？)的感覺，並且會自動避開窗戶、門、天空等地方。
屋頂和磚塊因為都是房子很重要的部分，所以GAN在生成的時候會自動修復這部分。<br>

#### 3. 移除樹、草、雲
<img src="./Images/remove.PNG" width="400px" />
因為樹、草和雲都不是照片中最主要的東西，所以大部分都可以被順利移除，但是還是會留下一點點痕跡不會完全消失。而磚塊、屋頂和門都沒有辦法被移除。 <br>

#### 4. 移除再畫
<img src="./Images/remove_paint.PNG" width="600px" />
我們先把原本照片左方的樹移除，再重新畫上新的樹，呈現出來的效果與原本的圖非常接近，幾乎看不出來哪張才是真實的照片。
<img src="./Images/remove_paint2.PNG" width="600px" />
跟上面的例子一樣，我們先把原圖中的草皮移除，再重新畫上新的草。
我們覺得很厲害的地方是，原本的草上下邊緣顏色較深，重新產生出來的也有這樣的效果。
此外在房子左方原本有個突起的小草叢，我們畫出來的草也有一個小小的隆起。 <br>

## Dissect GAN Model
我們分解了 Churchoutdoor_LSUN 這個  GAN<br>

### 1. Layer1
<img src="./Images/layer1.png" width="800px" />

#### A. 圈起來的都是小塊小塊的區域，在報告出來的結果分類上也有點不準確

#### B. 試著關掉某些 unit 對圖片的生成效果也不太明顯，應該是因為他是很前面的 layer 所以對生成的影響沒有很大

### 2. Layer4
<img src="./Images/layer4.png" width="800px" />

#### A. 圈起來的區域變大了，而且結果看起來也比較正確，雖然還是有一些類別不太對。

#### B. 關掉或開啟某些 unit 對 model 的生成會有細微的影響

### 3. Layer7
<img src="./Images/layer7.png" width="800px" />

#### A. 可以看出 layer7 在圈選區域與類別上相較前兩層的正確性高很多，其中樹跟草學得最好的感覺

#### B. 關掉 unit 對 model 生成的影響就比較大了，如圖中我們關掉了跟樹相關的 unit，則 model 生成的樹會少很多。可能是因為這個 layer 比較偏後面所以效果會比較顯著。我們認為此 layer 的效果最好。


## Compare with other method

### 1. Globally and Locally Consistent Image Completion
<img src="./Images/impainting_model.png" width="600px" />
這個方法可以應用在不歸則形狀的補全、物件移除，並且補全的東西可能不曾出現在圖片上（代表並非直接從圖片中找到相似區域去補全）。<br>
而這個方法主要分為三個架構： Completion Network、Global context discriminator、Local context discrminator。

#### A. Completion Network
Completion Network 主要的架構為 Fully Convolution network。為了讓除了需要修補的區域之外的圖片可以不會失真，產生之後會將除了修補區域以外的 RGB 值設回原值。

#### B. Global context discriminator
Global context discriminator 的 input 為整張圖片，用來分辨產生的整張圖片的真實性。

#### C. Local context discriminator
不同於 Global context discriminator，Local context discrminator 的 input 為以修補區塊為中心的局部圖片（128*128大小）。利用兩個 discrminator 的 output 結合來判斷所生成的圖片是否真實，這樣可以更加的顧及到大範圍以及小範圍圖片的合理、真實性，結果也會更加準確。
<br>
<br>

效果上在移除物的部分也蠻不錯的，如下圖<br>
<img src="./Images/inpaint-1.png" width="600px" />
<br>
下圖為挖空教堂左下角門、窗戶以及部分牆壁的結果，由於我們使用的這個pre-trained model適用於戶外自然風景，因此我們可以看到這個方法應用在修補建築物上結果看起來較為模糊，隱約可以看到窗戶的輪廓，但細節比較不清晰，在門的部分看起來也是稍微突兀。而跟GAN Paint移除門的結果比較，GAN Paint如上述在training時，會學習到自動一個建築物應該要有門這個結構，因此即使移除了門，仍會補上另一種看起來合理的門。
<br>
<img src="./Images/inpaint1.png" width="600px" />
<br>
下圖為應用到修補屋頂的部分，也會有上述的問題，因此結果看起來不太合理，若是用GAN Paint移除屋頂的話，則是跟原圖差異沒有太大。<br>
<img src="./Images/inpaint2.png" width="600px" />
<br>
而下圖我們針對樹進行移除修補的部分，則可以看到效果便相當合理。在GAN Paint移除樹的部分，畫面也算是和諧。<br>
<img src="./Images/inpaint3.png" width="600px" />
<br>
這個方法在較有挑戰性的人臉修補（face completion）也相當有突破，例如眼睛、鼻子、嘴巴等都可以達到非常逼真的修補，不過這次使用的pre-trained model比較適用於自然風景的圖片。




### 2. Image Outpainting
[這個方法](https://github.com/bendangnuksung/Image-OutPainting)是 Stanford 大學所發表的，他可以透過 input 圖片的中心，進而向左右去擴展圖片，得到更大張的圖。<br>
他的架構如下<br>
<img src="./Images/outpainting-model.png" width="600px" />
<br>
主要是參考 DCGAN 的架構，與上面的 Globally and Locally Consistent Image Completion 這個方法相似。他的 model input 為 128*128 的圖片，故 dataset 中的圖([3500 scrapped beach data](https://drive.google.com/file/d/1hKIn-Z8Uf3voESbJZVsapLHESPabjjrb/view))須先經過 preprocessing

#### Data pre-processing
先將 image down-sampling 成 128 * 128，並把他 normalize 到 [0,1]，得到 In。接著計算 In 的 mean pixel intensity 以及 Mask，利用他們與 In 計算得到 Ip。

#### Generator
Generator 的架構為 8 層的 convolution layer，並在倒數第二層中插入一層 deconvolution layer。利用 Datapreprocessing 中的 Ip 當作 input 生成最後的 output Io

#### Discrminator
Discriminator 在這篇中也運用了上一個方法中的 global+local 的概念，用來分辨 Io 以及 In。使用了 local discrminator 可以使生成後的圖片兩側比較不會有一條條的紋路，畫面看起來會比較平滑。

#### 效果
以下為我們的結果<br>
<img src="./Images/outpaint_1.jpg" width="400px" />
<img src="./Images/outpaint_2.png" width="400px" />
<br>
生成的圖片外圍的部分看起來雖然很模糊，但在色彩邊緣延續的部分看起來是合理的，海岸線的部分也有好好的連結。像是左圖那張夕陽照，在雲層，海洋的部分的顏色都有很好的延續，如果不要靠太近（？）看，整張圖看起來是合理的，不會有很突兀的感覺。
<br><br>
運用在 ganpaint 的塗上效果沒有很好，這是因為我們 training 的 dataset 是海灘的 dataset 並非 building，不過還是看得出他有照邊界的部分去做延伸<br>
<img src="./Images/outpaint_3.png" width="400px" />

## 結論
<img src="./Images/form.png" width="800px" />
