
# 科学とスターウォーズロボットアームデモ


## 概要

このデモでは、ワトソンのSpeech-to-TextおよびWatson Conversationサービスを使用して、独自のロボットアームを構築し、音声コマンドで制御する方法をご紹介します。 IoTのための視覚的ツールであるNode-REDとTrodden RoboticsのPhantomX Reactor Armを活用します。 PhantomXは、エントリーレベルの研究と大学での使用を想定して設計されており、市販されている最も高性能なロボットアームを提供しています。 ファントムXアームは<a href="http://www.trossenrobotics.com/p/phantomx-ax-12-reactor-robot-arm.aspx">ここ</a>で注文できます。 このデモでは、「No Wrist Rotate」キットタイプを使用していますが、「W / Wrist Rotate」タイプは、指定されたコードを編集することでサポートできます。


## 開始する

### ArbotiX & Arduinoソフトウェアをセットアップする
アームを手に入れたら、ArbotiX Robocontroller<a href="http://learn.trossenrobotics.com/arbotix/arbotix-quick-start.html">入門ガイド</a>に従ってArbotix-M Robocontrollerをセットアップしてプログラミングしてください。


### Dynamic IDをセットする

アームを組み立てる前に、すべてのサーボにダイナミックIDを設定する必要があります。<a href="http://learn.trossenrobotics.com/index.php/getting-started-with-the-arbotix/1-using-the-tr-dynamixel-servo-tool#&panel1-1">ここで</a>IDサーボの方法を学ぶことができます。 DynaManagerに問題がある場合は、Arduinoライブラリ、<a href="https://github.com/zcshiner/Dynamixel_Serial">Dynamixal Serial</a>などの代替オプションがあります。


### あなたのロボットを組み立てる

<a href="http://learn.trossenrobotics.com/projects/165-phantomx-reactor-arm-assembly-guide.html">Trossen’s Assembly Guide</a>を使用してアームを組み立てるための詳細な指示に従ってください。 Trossen Roboticsは、ロボットが正しくプログラムされ、正しく組み立てられていることを確認するための<a href="http://learn.trossenrobotics.com/interbotix/robot-arms/17reactor-robot-arm/26-phantomx-reactor-robot-arm-build-check">テストプログラム</a>を提供しています。

 
### アームにコードをアップロードする

ロボットアームフォルダー内のarduinoコードをダウンロードまたはクローンし、ロボットアームにアップロードします。ロボットがプログラムされ、動力を与えられると、ロボットは「待機」姿勢として中心位置に移動します。

Arduinoのシリアルモニタには以下のような制御オプションが表示されます。各コントロールの番号を入力すると、あなたの腕に各アクションが表示されます。シリアルコマンド送信オプションとして「9600ボー」があることを確認してください。

![screenshot_351](https://user-images.githubusercontent.com/4265959/32201985-175e315c-bdb0-11e7-92ab-e751fb67fd90.png)



## IBM Cloudアカウントを作成する

* IBM Cloudに<a href="https://console.ng.bluemix.net/registration/?target=/catalog/%3fcategory=watson">登録</a>するか、既存のアカウントを使用してください。 アカウントには、少なくとも1アプリと1サービスの空き容量が必要です。
 
 
* 以下の前提条件が満たされていることを確認してください。

         [npm] [npm_link]パッケージマネージャを含むNode.jsランタイム

         Cloud Foundryコマンドラインクライアント

         注：Cloud Foundryのバージョンが最新のものであることを確認してくださいMake sure that you have the following prerequisites installed:
    * [npm][npm_link]パッケージマネージャーを含む、[Node.js](https://nodejs.org/#download)ランタイム
    * Cloud Foundryコマンドラインクライアント

      Note: Cloud Foundryのバージョンが最新のものであることを確認してください。


### 会話サービスの設定

会話サービスの既存のインスタンスを使用できます。 それ以外の場合は、次の手順を実行します。

1. Cloud Foundryのコマンドライン・ツールを使用してIBM Cloudに接続します。 詳細については、Watson Developer Cloudの<a href="https://console.bluemix.net/docs/cli/reference/bluemix_cli/get_started.html#getting-started">ドキュメント</a>.
を参照してください。

    ```bash
    cf login
    ```

2. IBM Cloudで会話サービスのインスタンスを作成します。 例えば：

    ```bash
    cf create-service conversation free my-conversation-service
    ```

### 会話ワークスペースの読み込み

1. ブラウザーで、IBM Cloud<a href="https://console.ng.bluemix.net/dashboard/services">コンソール</a>にナビゲートします。 

2.**[すべてのアイテム]**タブで、**[サービス]**リストで新しく作成した会話サービスをクリックします。

3.サービスの詳細ページで、**[起動ツール]**をクリックします。

4.会話サービスツールの**[ワークスペースのインポート]**アイコンをクリックします。アプリケーションプロジェクトのローカルコピーにワークスペースJSONファイルの場所を指定します：

    `<project_root>/workspace-robot-arm.json`

5.**Everything (Intents, Entities, and Dialog)**を選択し、**Import**をクリックします。ロボットアームワークスペースが作成されます。

6. `cf service-key <service_instance> <service_key>`コマンドを使用して、サービスキーから資格情報を取得します。例えば：

    ```bash
    cf service-key my-conversation-service myKey
    ```

   このコマンドの出力は、次の例のようにJSONオブジェクトです。

    ```JSON
    {
      "password": "*******",
      "url": "https://gateway.watsonplatform.net/conversation/api",
      "username": "***************************"
    }
    ```

7. Node-REDで使用するために、JSONの`password`と`username`の値（引用符は不要）をスクラッチファイルに貼り付けます。

8. Speech-to-Textサービスインスタンスを作成し、サービスにアクセスするためのサービスキーを取得します。

  ```none
  cf create-service speech_to_text standard robot-stt-service
  cf create-service-key robot-stt-service myKey
  cf service-key robot-stt-service myKey
  ```

9. Node-RED内で使用するために、資格情報の`password`と`username`の値（引用符は不要）をスクラッチファイルに保存します。


## IBM WatsonでNode-REDをセットアップする

Node-REDは、IBM Watson IoTプラットフォームでアプリケーション、デバイス、およびゲートウェイを開発するために使用できるビジュアル・プログラミング・ツールです。 Node-REDは、ハードウェア、API、およびオンラインサービスを新規かつ興味深い方法で接続するための機能を提供します。

Node-REDはNode.jsの上に構築されているため、コンピュータにNode JSとNode-REDの両方をインストールする必要があります。 この<a href="https://nodered.org/docs/getting-started/installation">インストールガイド</a>に従ってください。 完了したら、コンピュータを再起動します。

ターミナルアプリケーションを開き、 'node-red'と入力します。 これはNode-Redを起動します。

```bash
    node-red
   ```
   
「Server now running at http://127.0.0.1:1880/“」のようなサーバIPアドレスが表示されます。ノードREDが実行されると、ブラウザにIPアドレスを指定することでアクセスできます。


### 必要なノードをインストールする

ワトソンをロボットアームで使用するには、いくつかのノードをインストールする必要があります。右上のハンバーガーメニューをクリックし、 'パレットの管理'を選択します。

[インストール]をクリックし、検索バーに「node-red-node-watson」と入力します。検索画面に「node-red-node-watson」ノードが表示されます。 「インストール」ボタンをクリックしてインストールしてください。

'node-red-node-watson'ノードを正常にインストールすると、 'IBM Watson'カテゴリの下にインストールされたノードが表示されます。

同じ方法で 'node-red-contrib-browser-utils'と 'node-red-node-serialport'ノードをインストールしてください。

「入力」カテゴリに「シリアル」ノードと「マイク」ノードが追加されています。


### フローを設定する

ここで、ノードを追加してフローを作成します。 「マイク」ノードをドラッグしてフローエディタにドロップします。次に、「speech to text」ノードを挿入します。 'speech to text'ノードをダブルクリックして、サービスの資格情報を入力します。 'Speaker Labels'のチェックをはずし、 'msg.payloadに出力を置く'オプションをオンにします。

「マイク」ノードと「スピーチをテキストにする」ノードを接続します。フローエディタに「会話」ノードを挿入し、ダブルクリックして会話サービスの資格情報を追加します。

Speech-to-Textの後、既に持っている接続チェーンに会話を追加します。以下のように、「関数」ノードを挿入して関数を設定します。これにより、記録された音声コマンドとWatsonによって翻訳されたアームアクションがmsg.preloadに保存されます。

```
msg.payload = {"speech":msg.payload.input.text, "action":msg.payload.context.arm_action};
return msg;
```
アクションハンドラの後にデバッグノードを挿入します。

右端にある展開ボタンをクリックします。フローが動作するはずです。スピーチの録音を開始するには、「マイク」ノードをクリックしてから、マイクに向かって「ボディーを左に」話してください。もう一度[マイク]ノードをクリックして録音を停止します。オーディオがフロー全体で処理されると、デバッグパネルにデバッグ結果が表示されます。

![screenshot_331](https://user-images.githubusercontent.com/4265959/32201966-15f02e60-bdb0-11e7-9ac2-02df597dd253.png)

今度は、ロボットアームを制御するためにノードを追加します。 以下のように、「関数」ノードを挿入して関数を設定します。 これは、Wastonからの各アクション呼び出しを数値に変換します。

```
var action = msg.payload.context.arm_action;

if(action == "sleep") msg.payload = "0";
else if(action == "standby") msg.payload = "1";
else if(action == "body_right") msg.payload = "2";
else if(action == "body_left") msg.payload = "3";
else if(action == "body_up") msg.payload = "4";
else if(action == "body_down") msg.payload = "5";
else if(action == "arm_up") msg.payload = "6";
else if(action == "arm_down") msg.payload = "7";
else if(action == "hand_up") msg.payload = "8";
else if(action == "hand_down") msg.payload = "9";
else if(action == "gripper_open") msg.payload = "o";
else if(action == "gripper_close") msg.payload = "p";
else if(action == "light_saber") msg.payload = "a";
else msg.payload = "-1";
return msg;
```

以下のように 'スイッチ'ノードとセットアップ機能を挿入します。 このスイッチは、前の関数ノードからのアクションデータを処理します。

![screenshot_333](https://user-images.githubusercontent.com/4265959/32201964-15c0d688-bdb0-11e7-93c8-5d61bdd9a6a0.png)

「シリアル」ノードを挿入し、「編集」ボタンをクリックして、接続されたXBee-USBコンバータを登録します。 XBee-USBコンバータがコンピュータに接続されていることを確認してください。 検索をクリックすると、接続されているXBee-USBコンバータが見つかります。 リストから「/dev/cu.usbserial-XXXXXX」を選択し、次のように設定します。

![screenshot_345-2](https://user-images.githubusercontent.com/4265959/32201961-157f6c0c-bdb0-11e7-9d1c-c650ea08b031.png)

新しいノードを接続し、「Deploy」ボタンをクリックします。 接続されたXBee-USBコンバータが見つかるとすぐに、シリアルノードの下に「接続された」メッセージが表示されます。 シリアルポート経由で接続しない場合は、前の手順を確認してください。

![screenshot_344](https://user-images.githubusercontent.com/4265959/32201960-156bcd5a-bdb0-11e7-8a92-8e4cffc85351.png)


## ロボットアーム用コマンドリスト提供

![screenshot_346](https://user-images.githubusercontent.com/4265959/32201959-15562f72-bdb0-11e7-9347-e7cda661260c.png)
