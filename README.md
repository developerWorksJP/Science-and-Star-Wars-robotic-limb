
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

     
     cf login

     IBM Cloudで会話サービスのインスタンスを作成します。 例えば：

1. Cloud Foundryのコマンドライン・ツールを使用してIBM Cloudに接続します。 詳細については、Watson Developer Cloudの<a href="https://console.bluemix.net/docs/cli/reference/bluemix_cli/get_started.html#getting-started">ドキュメント</a>.
を参照してください。

    ```bash
    cf login
    ```

2. Create an instance of the Conversation service in IBM Cloud. For example:

    ```bash
    cf create-service conversation free my-conversation-service
    ```

### 会話ワークスペースの読み込み

1. In your browser, navigate to your IBM Cloud <a href="https://console.ng.bluemix.net/dashboard/services">console</a>.

2. From the **All Items** tab, click the newly created Conversation service in the **Services** list.

3. On the Service Details page, click **Launch tool**.

4. Click the **Import workspace** icon in the Conversation service tool. Specify the location of the workspace JSON file in your local copy of the app project:

    `<project_root>/workspace-robot-arm.json`

5. Select **Everything (Intents, Entities, and Dialog)** and then click **Import**. The robot arm workspace is created.

6. Retrieve the credentials from the service key using the command `cf service-key <service_instance> <service_key>`. For example:

    ```bash
    cf service-key my-conversation-service myKey
    ```

   The output from this command is a JSON object, as in this example:

    ```JSON
    {
      "password": "*******",
      "url": "https://gateway.watsonplatform.net/conversation/api",
      "username": "***************************"
    }
    ```

7. Paste  the `password` and `username` values (without quotation marks) from the JSON into a scratch file for use within Node-RED.

8. Create a Speech-to-Text service instance and retrieve service keys to access the service:

  ```none
  cf create-service speech_to_text standard robot-stt-service
  cf create-service-key robot-stt-service myKey
  cf service-key robot-stt-service myKey
  ```

9. Save  the `password` and `username` values (without quotation marks) for your credentials into a scratch file for use within Node-RED.


## Setup Node-RED with IBM Watson

Node-RED is a visual programing tool that you can use to develop your applications, devices and gateways on IBM Watson IoT platform. Node-RED provides capabilities for connecting hardware, APIs and online services in new and interesting ways.

Node-RED is built on top of Node.js so you need to install both Node JS and Node-RED on your computer. Please follow this <a href="https://nodered.org/docs/getting-started/installation">Installation guide</a>. After done, restart your computer.

Open Terminal application and type ‘node-red’. This would launch Node-Red.
```bash
    node-red
   ```
   

You’ll see a Server IP Address like  " Server now running at http://127.0.0.1:1880/“. Once Node-RED is running, you can access it by pointing your browser at the IP Address. 

### Install Required Nodes

You need to install several Nodes to use Watson with your robot arm. Click the hamburger menu on top-right side and choose ‘Manage palette’.

Click ‘Install’ tap and type 'node-red-node-watson’ in the search bar. The search screen would show ‘ node-red-node-watson’ node. Install it by clicking a ‘Install’ button.

After installing 'node-red-node-watson’ node successfully, you’ll see installed nodes under ‘IBM Watson’ category.

Install 'node-red-contrib-browser-utils’ and  'node-red-node-serialport' node in the same way.

You’ll see a ‘Serial’ node and ‘microphone’ node are added in the ‘input’ category. 

### Set a flow

Now create a flow by adding nodes. Drag and drop an ‘microphone’ node on the flow editor. Then insert a ‘speech to text’ node. Double click the ‘speech to text’ node and set enter your credentials for the service. Uncheck ‘Speaker Labels’ and check ‘Place output on msg.payload’ option.

Connect the ‘microphone’ node and ’speech to text’ nodes. Insert a ‘conversation’ node on the flow editor, then double click and add your credentials for the Conversation service.

Add Conversation to the connected chain you already have, after Speech-to-Text. Insert a ‘function’ node and set up function as below. This would save a recorded voice command and an arm action translated by Watson into msg.preload.

```
msg.payload = {"speech":msg.payload.input.text, "action":msg.payload.context.arm_action};
return msg;
```

Insert a debug node after your Action Handler.

Click on the deploy button on the right corner. The flow should now be working. Click the ‘microphone’ node to start recording a speech, then speak ‘body left’ to your microphone. Click the ‘microphone’ node again to stop recording. Once the audio has been processed through the entire flow, you should see a debug result at the debug’ panel.

![screenshot_331](https://user-images.githubusercontent.com/4265959/32201966-15f02e60-bdb0-11e7-9ac2-02df597dd253.png)

Now it’s time to add more nodes to control the robot arm. Insert a ‘function’ node and set up function as below. This will convert each action call from Waston to a number.

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

Insert a ‘switch’ node and setup function as below. This switch will handle action data from previous function node.

![screenshot_333](https://user-images.githubusercontent.com/4265959/32201964-15c0d688-bdb0-11e7-93c8-5d61bdd9a6a0.png)

Insert a ‘serial’ node then click the ‘edit’ button to register a connected XBee-USB Converter. Make sure your XBee-USB converter is connected to your computer. Click search to find the connected XBee-USB converter.
Choose ‘/dev/cu.usbserial-XXXXXX’ from the list and setup settings as follows.

![screenshot_345-2](https://user-images.githubusercontent.com/4265959/32201961-157f6c0c-bdb0-11e7-9d1c-c650ea08b031.png)

Connect the new nodes and click the Deploy button. A ‘connected’ message would appear under the serial node as soon as a connected XBee-USB converter is found. If it doesn’t connect via Serial port, check previous steps.

![screenshot_344](https://user-images.githubusercontent.com/4265959/32201960-156bcd5a-bdb0-11e7-8a92-8e4cffc85351.png)


## Provided Command list for Robot Arm

![screenshot_346](https://user-images.githubusercontent.com/4265959/32201959-15562f72-bdb0-11e7-9347-e7cda661260c.png)
