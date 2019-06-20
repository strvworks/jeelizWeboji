# jeefacetransferAPI Document

## 前書き
JEEFACETRANSFERAPIは、ビデオストリームからユーザーの感情や頭の回転情報などを検出するAPIです。
JEEFACETRANSFERAPIはニューラルネットワークを用いてリアルタイムにビデオストリームからユーザーの感情を取り出します。
これはWEBOJI APIでも利用されています。この2つのAPIは、JEEFACETRANSFERがwebojis以外の用途にも利用可能なことから分離されました。
例えば、ユーザーが満足しているかどうか、どんな精神状態であるかなどを検出することができます。JEEFACETRANSFERAPIはTHREE.jsを要求しません。

## インテグレーション
アプリケーションは以下を要求します。
- jeefacetransferNNC.jsスクリプト
- 直接起動されない、WEBOJIもしくはその他のAPIはJEEFACETRANSFERを利用すべきです。
- <canvas>DOM要素。ユーザーの検出情報を示すビデオはこのcanvasに表示可能です。

## 属性 (読み取り専用)
- JEEFACETRANSFERAPI.ready : bool値. 準備ができたかどうかを示します。

## メソッド
### 事前初期化メソッド
これらすべてのメソッドは、JEEFACETRANSFERAPI.init()よりも前に呼び出されるべきです。

- JEEFACETRANSFERAPI.set_size(<integer> widthPx, <integer> heightPx)
この関数はJEEFACETRANSFERAPI.init()を含めた他のすべてのメソッドの前に呼び出されるべきです。 これはキャンバスの縦横サイズを設定します。

- JEEFACETRANSFERAPI.set_audio(<boolean> isAudio)
マイクからオーディオストリームを取得します。デフォルトではfalseです。

- JEEFACETRANSFERAPI.onWebcamAsk(<function> callback)
Webカメラの使用を要求するとき直前にこのコールバックが起動されます。

- JEEFACETRANSFERAPI.onWebcamGet(<function> callback)
Webカメラからのビデオストリームを取得した直後にこのコールバックが起動されます。

- JEEFACETRANSFERAPI.onContextLost(<function> callback)
webglコンテキストがロストしたときに起動されるコールバックです。

### 一般メソッド
- JEEFACETRANSFERAPI.init(<object> spec)
ライブラリを初期化します。specは以下のプロパティを保持するディクショナリです。
    - <string> canvasId: <canvas>要素のID
    - <string> NNCpath: ニューラルネットワークのモデルのパス
    - <string|object> NNC: NNCpathが未定義の場合、ここに指定されたJSON文字列もしくはニューラルネットワークファイルの解析を行います。
    - <function> callbackReady(<string> errCode) : コールバック関数。 エラーがない場合、errCodeはfalseに設定されています。 エラーコードについては次章を参照してください。
    - <object> videoSettings: WebRTC MediaConstraintsをオーバーライドする辞書。 これは以下のようなプロパティを保持しています。
        - videoElement: 指定された<video>要素。デフォルトでは設定されていません。カメラ入力ではなくカスタムのビデオを設定する場合に便利です。もし指定された場合、他のビデオ設定は無効化されます。
        - deviceId: 利用デバイス(カメラなど)のIDです。デフォルトでは設定されていません。
        - facingMode: デフォルトでは’user’がセットされています。リアカメラ(外カメラ)を利用する場合は’environment’をセットしてください。
        - idealWidth: 推奨されるビデオの横サイズ default: 320,
        - idealHeight: 推奨されるビデオの縦サイズ default: 240,
        - minWidth: ビデオの最小横サイズ default: 240,
        - maxWidth: ビデオの最大横サイズ default: 1280,
        - minHeight: ビデオの最小縦サイズ  default: 240,
        - maxHeight: ビデオの最大縦サイズ  default: 1280
        
- JEEFACETRANSFERAPI.onLoad(<function> callback)
JEEFACETRANSFERAPIの準備が完了すると呼び出されるコールバックです。準備が完了するまで待機します。

- JEEFACETRANSFERAPI.switch_sleep(<bool> isSleep)
リソース節約のため、検出とレンダリングのループをストップします。fitter/viewerが画面に表示されていないときに呼び出されます。

- JEEFACETRANSFERAPI.get_cv()
canvas要素を返します。

- JEEFACETRANSFERAPI.get_videoStream()
return the WebRTC raw video stream, useful to record the audio track,
WebRTCの生ビデオストリームを返します。オーディをトラックを記録する際などに便利です。

- JEEFACETRANSFERAPI.switch_displayVideo(<boolean> isDisplayVideo)
if true, display the video of the user on the DOM canvas element, with a marker to delimit the face. Can be used before the initialization of the API.
trueの場合、canvas DOM要素にユーザーのビデオと顔の検出枠を表示します。APIの初期化前に呼び出し可能です。

- JEEFACETRANSFERAPI.on_detect(<function> callback)
Launch the callback function if the face is detected or when the detection is lost. The callback is called with 1 argument, true if the face is detected, false if the detection is lost,
顔が検出された/検出が失われた際に呼び出されるコールバックです。コールバック関数には1つの引数が渡され、顔を検出した場合はtrue,検出が失われた場合にはfalseです。

- JEEFACETRANSFERAPI.set_animateDelay(<integer> delay)
Change the delay between 2 detections. it can be helpful to free up some resources to speed up DOM transition or video encoding. The value is given in milliseconds,
検出処理同士の間隔を変更します。DOM要素のトランジションやビデオのエンコーディング等のためのリソース確保に役立つ場合があります。値はミリセカンドで渡すことができます。

- JEEFACETRANSFERAPI.set_color([<float> R,<float> G,<float>B]): set the
color of the frame (if displaying the debug view only). R,G,B are between 0 and 1. Default is [0.0, 0.5, 1.0] (light blue).
デバッグビューのみを有効化している場合のフレームの色をセットします。R,G,Bは0から1の間で指定されます。初期値は[0.0, 0.5, 1.0] (ライトブルー)です。

### 初期化時エラーコード
これらのエラーコードはcallbackReadyの第一引数として得ることができます。
- false: エラーなし
- ALREADY_INITIALIZED: APIはすでに初期化されている(多重初期化)
- NO_CANVASID: 指定されたcanvasのIDが見つからない
- INVALID_CANVASID: <canvas>要素がDOMに存在しない
- WEBCAM_UNAVAILABLE: webcamへのアクセスに失敗。(ユーザーがwebcamを持っていない。デバイスへのアクセスが許可されていない。すでにwebcamがビジー状態である。など。) 
- GL_INCOMPATIBLE: WebGL is not available, or this WebGL configuration is not enough (there is no WebGL2, or there is WebGL1 without
- GL_INCOMPATIBLE: WebGLが有効でない。もしくはWebGLの設定が十分ではない。 (動作環境がWebGL2でない。WebGL1環境でOES_TEXTURE_FLOATやOES_TEXTURE_HALF_FLOAT拡張が存在しないなど。)

### モーフと回転
これらのメソッドは、ニューラルネットワークからモーフ係数や回転などを取得するために、より高レベルなスクリプト(例えばWEBOJIなど)によって呼び出されるものです。
- JEEFACETRANSFERAPI.get_nMorphs()
モーフの数を返します。

- JEEFACETRANSFERAPI.get_morphTargetInfluences()
モーフ係数の配列を返します。

- JEEFACETRANSFERAPI.get_morphTargetInfluencesStabilized()
安定化したモーフ係数の配列を返します。

- JEEFACETRANSFERAPI.get_morphUpdateCallback()
モーフ係数の配列が更新された際に呼び出されるコールバック関数です。この関数は呼び出される際に以下の引数を持ちます。
    – <float> quality: 検出精度。0(低品質)から1(高品質)の間で与えられます。
    – <float> benchmarkCoeff: higher it is, less powerful is the computer of the user. Can be directly used as a factor of the blending coefficients for morphing amortization,
    – <float> benchmarkCoeff: この値が大きいほど、ユーザーのコンピューターが非力であることがわかります。この値を直接モーフィング償却(?)の係数と混ぜて使うことができます。
    
- JEEFACETRANSFERAPI.get_rotation()
頭の回転を示す角度を要素数3の配列でオイラー角として返します。

- JEEFACETRANSFERAPI.get_rotationStabilized(): 
頭の回転を示す角度を要素数3の配列でオイラー角として返す関数の安定化版？(原文ではsome then(同様)となっている)

- get_rotation() method but with more stabilization and automatic rotation if the head is not found,

- JEEFACETRANSFERAPI.get_positionScale()
要素数3のfloat型配列 [Px, Py, s]を返します。
PxとPyはビューポートのサイズに対する2次元の座標を示す値で、0-1の間で与えられます。
sは幅に対する相対的なスケールです。(1が横幅いっぱい) PxとPyはそれぞれ左→右(ミラー表示)、下→上を向いています。

- JEEFACETRANSFERAPI.is_detected()
顔が検出されたかどうかを返します。
