# source-separation
[本家](https://source-separation.github.io/tutorial/landing.html)が英語だったので，  
とりあえず日本語でコメントをつけつつチュートリアルをしている．  
ゴリゴリDeepLやGoogle translateを使用しております．．．  
B3への共有も兼ねて．．．  

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
