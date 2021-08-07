# source-separation
[本家](https://source-separation.github.io/tutorial/landing.html)が英語だったので，  
とりあえず日本語でコメントをつけつつチュートリアルをしている．  
ゴリゴリDeepLやGoogle translateを使用しております．．．  
B3への共有も兼ねて．．．  

以下はscrapboxから移植したので，結構見づらいので，おいおい整形します．

## BASICS OF SOURCE SEPARATION  

### Representing Audio  
source separationのためのオーディオ表現について見ていく．  

連続時間信号は，時間と振幅の両方で離散化されている．  
オーディオチャネルが１つの場合monophonic，２つの場合stereophonicという．  

sample rate 
一秒間に何回のサンプリングが行われるか．単位はHz  
サンプリング定理とナイキスト周波数についての説明．  

trainning, validation, testingでsample rateは同じにするという過程がある．  
異なる場合はダウンサンプルして同じに揃える必用がある．  

source separationは3つのアプローチによって行われる．  
1 分離しやすい表現に変換する．  
2 実際に分離する  
3 分離したものを逆変換して音源に戻す．  

したがって，オーディオ表現に求められるのは可逆性．  
変換して，何らかの処理をして，分離するなどの処理を行った後，逆変換でもとに戻せないといけない． 
これらの処理によってArtifactは生じてしまうが，それを最小化したいというお気持ちがある．  
加えて，source separateしやすいような表現が求められる．Fig7参照．  


### Input Representations
[https://scrapbox.io/files/60fc1b3f736656002204beaf.png]

Time-Frequency (TF) Representations  
y: frequency, x: time, value: energy  

意味：特定の周波数と特定の時間における，振幅の大きさ．  
カラーバーがない場合は，明るいほうが振幅が高いと考える．  
Time-frequency representations は，source separationで使用される最も一般的なアプローチである．  

Short-time Fourier Transform (STFT)  
[https://scrapbox.io/files/60fc1bb3f87214001c26e939.png]

windowを動かしてDFTする．  
STFTの各ビンが複素数であり，振幅成分と，位相成分を含んでいることに注意する．  

Here are some important parameters to consider when computing an STFT:  
 window types  
 	window typeはDFT前のセグメントの形状を決定する． 
 Window Length  
 	shor-time-windowに含まれるサンプル数を決定する．  
 	周波数軸の分解能を決める．（window-lengthが大きいほど，frequency resolutionは高くなる．）  
 Hop Length  
 	隣接するshort-time-windowの距離を決定する．  

信号の位相をモデル化することは困難であるため，位相を明示的に表現しないスペクトログラムに音源分離を適応する．  

	- Magnitude Spectrogram
		STFTの結果が複素数で表されるため，絶対値を計算したもの．  
	- Power Spectrogram  
		複素数の値の２乗を計算した値．  
	- Log Spectrogram  
		人間の聴覚は対数に近いので，SIFTの各要素の絶対値にLogを取ったもの  
	- Log Power Spectrogram  
		二乗を計算した後Logをとった値．  

スペクトログラムは明示的な位相表現を持たないTF表現を指すときに使われることが多い．  

Mel-spaced Spectrograms    
人間の聴覚は周波数に対して，対数的なので，  
高い周波数を押しつぶして，より低い周波数に焦点を当てている．  
Deepに置いて，計算量を削減するためにも用いられる．  

### Output Representations
音源分離は，一度処理を加えた後，音源に逆変換可能である必用がある．  
直接分離後の波形を返すこともあるが，多くは，マスクを返すアルゴリズムであることが多い．  
マスクはもとの混合音のスペクトログラムに適応される．  

Which is Better? Inputting a Waveform or a Time-Frequency Representation?  
 
深層学習に限って言うと  
music separationのSoTAのシステムでは，  
波形とスペクトログラムの両方を入力とする．  
speech separationの場合では，波形を入力として使用することに収束している．  

### TF Representations and Masking
混合音から，音源を近似するときにマスクは必要不可欠である．
単一のソースを分離する場合は，１つのマスクで良いが，複数のソースを対象として分離したい場合は複数個のマスクが必用．  

マスクはスペクトログラムと同じ形状であり，  
maskとspectrogramの間で，アダマール積を計算することで，source estimateを出力する．  
マスクの各要素の値は，ある音源のもとの混合音のエネルギーに占める割合を示す．0-1の間の値を取る．  

推定するべきmaskは，適応した際に良いsource estimateを出力するようなmaskである．  
maskを使って対象の音源を取り出すこともできるが，maskを反転させて，対象の音以外を取り出すという処理もできる．  
	binary-mask  
		0 or 1 の値を取るmask  
	soft-mask  
		0-1の区間の値を取るmask 混合音のエネルギーをソースの一部に割り当てることができる．  

maskを推定した後，混合音信号にてきおうして，推定ソースのmagnitude spectrogramをオーディオに戻して聞けるようにする必用がある．位相を理解する必用がある．

##　Phase
音源分離は，より良いmaskを推定することに主眼が置かれることが多いが，   
位相も重要な概念である．  
Phase - A Quick Primer  
[https://scrapbox.io/files/60fc28b030d952002232e0a6.gif]  

位相はφで表され，負の値を取るときは，遅れを正の値を取るときは進んでいることを表す．  

Why We Don’t Model Phase  
  
[https://scrapbox.io/files/60fc2ae691ff0c001c87490c.gif]  

隣り合うタイムステップで，時間間隔が同じだとしても，  
高周波成分と低周波成分によって位相の折返しは変わってしまう．  

人間が位相を正確に認識するとは限らないので，source separationの分野で，magnitudeと同程度にモデリングすることは少ない．  
位相推定手法をざっくりと  
	The Easy Way  
		混合物から位相をコピーする方法．そこそこうまく行くらしい．  
		maskとspectrogramのアダマール積で，source estimationしたときと同様に，位相成分を掛け算するだけ．  
	The Hard Way  
		Phase Estimation  
			Griffin-Lim  
				STFTと逆STFTを繰り返し行うことで，spectrogramの再構成を行う  
			modelのend-to-endに入出力を波形で行うことができる．  


Evaluation  
	source separationの評価は難しい問題であり，客観的と主観的両方の視点から行われる．  
		客観的な手法では estimate sourceとgrandtruthの間で比較を行う計算をする．  
		主観的な評価は，人間が点数をつける．  

真の値は，干渉，ノイズ，誤差を付け加えて構成されていると考えられている．  
Source-to-Artifact Ratio (SAR)  
Source-to-Interference Ratio (SIR)  
Source-to-Distortion Ratio (SDR)  
Signal-to-Noise Ratio (SNR)  

同じSDRでも実際聞いてみると，質が違っていた．．．  

## nussl
nusslを通じて，source separationを学ぶ．  

why nussl   
nusslがいいよって話が書いてある．  

nussl  
https://github.com/nussl/nussl  
document  
https://nussl.github.io/docs/  

### nussl.AudioSignal()   
main container for all things.  
オーディオデータに関連するすべてのメインコンテナであり，オーディオを簡単に操作するための多くの有用なユーティリティを提供する．  
nussl.AudioSignalオブジェクトは，nusslのすべての音源分離アルゴリズムの中核をなすものである．  

## Deep Learning
deepの方へ．

### intro
grand truthと比較する必用があるので，音源分離は教師つき機械学習である  
よって，大量の分離された音源データが必用となる．  

### Building Blocks
この辺は，ML，DLの基礎と重複してしまうので，適当にまとめています．


#### fully connenct
活性化関数無しで，入力データに線形変換を施す．  
やくわりとしては，次の層の入力の次元を拡大または縮小すること．  
source-separationにおいては，fully-connented-layerはマスクを作成するために使用され，出力の次元(スペクトログラムの周波数成分の数や、波形のサンプル数)と一致するように，  
前の層の次元を変更する．  
maskとして使用する場合は，活性化関数を持っている．

#### Deep Clustering 
スペクトログラムを入力として用いられる．
目標は同じソースに支配されるTF binが全て近くにあり，異なるソースに支配されたTF binが遠くにある，高次元の埋め込み空間を学習することである．  

#### Waveform Losses
波形ドメインの損失を計算する気暗澹な方法は，スペクトログラムの場合と同様に，実際の波形と推定波形の間でL1もしくはMSEの損失を取得することである

### Architectures

source-separationの最新の深層学習のアーキテクチャ紹介
混合スペクトログラムに適用されるマスクを作る
音源の波形を直接推定する
の２パターンに分けられる．

#### Unet
Unetは音楽の音源分離でよく使われる．
スペクトログラムを入力し，2D convolutionをして，入力のエンコーディングを作る．
同じ数の2D deconvolution layerでデコードする．
encoding layerは，対応する，decoding layerに連結される．

Unetが学習したmaskとinputの間で乗算され，損失が計算される．
U-Netは畳み込み式なので，一定の形状を持つスペクトログラムを処理する必用がある．
オーディオ信号はUnetが学習したのと同じ数の時間と周波数の次元を持つスペクトログラムに分割される必用がある．

いろいろなバリエーションが存在している．

#### Mask Inference

スペクトログラムを入力し，それぞれをRNNに突っ込んでマスクを出力する．
BLSTMを使用するのが一般的．






