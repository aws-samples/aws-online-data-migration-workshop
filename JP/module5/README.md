AWS DataSyncを使ったSMBファイル共有データの移行
======================
Copyright Amazon Web Services, Inc. and its affiliates. All rights reserved.This sample code is made available under the MIT-0 license. See the LICENSE file.

何かお気付きの場合は kawabmak@amazon.com までお願い致します

-------------------------------------------------------------------------------------
**概要**
-------------------
AWS DataSyncは、オンプレミスのストレージとAmazon S3、Amazon Elastic File System、またはAmazon FSx for Windowsファイルサーバー間のデータ移行を簡単かつ迅速に行えるようにするデータ転送サービスです。 AWS DataSyncは、メタデータの保持、データの整合性の検証、並列転送とネットワークの最適化を可能にし、複雑なスクリプトの作成と管理など、データ移行に必要とされる複雑な数多くの負荷の高いタスクを自動的に処理します。

<br/><br/>

**このモジュールの流れ**
-------------------
このモジュールでは、送信元となるWindows SMB共有をWindows EC2インスタンスに作成し、送信先となるWindows SMB共有をAmazon FSx for Windowsファイルサーバーに作成します。次に、AWS DataSyncをデプロイし、DataSyncを使って送信元から送信先へデータを移行します。

<br/><br/>

**必要時間**
-------------------
この章の目安時間は約20分です。

<br/><br/>

**送信元Windows ファイル共有の作成**
-----------------------------

1. ブラウザで次のリンクを開き(https://view.highspot.com/viewer/5f1a645ba2e3a938135fa3f6)、ドキュメントを**ダウンロード**して下さい。このワークショップを通して必要な情報のメモとして使用します。   
2. ワークショップで使用しているWindows EC2インスタンスに接続し、 左下のWindowsアイコンをクリック、次に**C:\\**を入力して、Enterを押して下さい。
3. **Tools**フォルダーを右クリックし、**Properties**を選択して下さい。
4. **Sharing**タブをクリックし、**Advanced Sharing**をクリックして下さい。
5. **Share this folder**を選択し、**OK**をクリックし、ダイアログを閉じて下さい。
6. コマンドプロンプトを開き(例：左下のWindowsアイコンをクリックしてcmdと入力)、**hostname**と入力し、Enterを押して下さい。表示されたサーバーのホスト名を上記でダウンロードしたメモに記載して下さい。後ほど使用します。



<br/><br/>

**送信先Amazon FSx for Windowsファイルサーバー　ファイル共有の作成**
-----------------------------

Amazon FSx for Windowsファイルサーバーへ、送信先となるファイル共有を作成します。

1. ワークショップで使用しているWindows EC2インスタンスに接続し、 左下のWindowsアイコンをクリック、次に**fsmgmt.msc**を入力して、Enterを押して下さい。
2. **Action**をクリックし、**Connect to another computer**を選択して下さい。
3. Amazon FSx for WindowsファイルサーバーのDNS name(例： amznfsxnzvcely3.example.com)を入力し、OKを押して下さい。このDNS Nameもメモに書き留めておいて下さい。
4. **Shares**フォルダーを右クリックし、**New Share**を選択しNextを押して下さい。
5. Browseをクリックして下さい。
6. d$　を選択して下さい。
7. Make New Folderをクリックして下さい。
8. **myshare**という名前を入力して下さい。
9. OKを押して下さい。
10. Create A Shared Folderウィザードを進めます。**Shared folder permissions**の画面で**Custom permissions**を選択し、**Custom**をクリックして下さい。**Everyone**ユーザーに**Full Control**を割り当てて、その後ウィザードを終了まで進めて下さい。


<br/><br/>

**AWS DataSyncエージェントのデプロイ**
-----------------------------

AWS DataSyncエージェントをAWS VPCにEC2インスタンスとしてデプロイします。AWS DataSyncエージェントは送信元Windowsファイル共有から直接データを読み出し、送信先の**Amazon FSx for Windowsファイルサーバー上ファイル共有**へファイルを転送します。


- エージェントのアクティベーションを行うために、エージェントとなるインスタンスとAWSコンソールの双方にアクセス出来る必要があるため、このデプロイの手順はWindows EC2インスタンス上で行います。Windows EC2インスタンスに接続し、左下のWindowsアイコンをクリック、**Powershell**と入力し、表示された候補の中から**Windows Powershell**を選択して下さい。
デフォルトブラウザとなっているIEの使用を避けるため、まずはChromeのインストールを行います。

以下のスクリプトをPowerShellに貼り付けて実行して下さい。デスクトップにChromeのアイコンが現れて、プロンプトが戻ったら完了です。

    $Path = $env:TEMP; $Installer = "chrome_installer.exe"; Invoke-WebRequest "http://dl.google.com/chrome/install/375.126/chrome_installer.exe" -OutFile $Path\$Installer; Start-Process -FilePath $Path\$Installer -Args "/silent /install" -Verb RunAs -Wait; Remove-Item $Path\$Installer

2.  Chromeを開いてAWSコンソールにアクセスして下さい。コンソール上部の**Services**をクリックし、**DataSync**と入力して、現れたリンクを選択します。

    -   **Get Started**を選択して下さい。

    -   Deploy agentの部分でドロップダウンメニューから**Amazon EC2**を選択して下さい。**Learn more**のリンクから具体的なデプロイ方法のドキュメントを参照できます。

    -   [注意：このステップはAWS公式ドキュメントが更新され、使えなくなっているため、講師の支持に従って下さい]Scroll down to the table that has a list of **AWS Regions and AMI Names**, and click on
        the **Launch Instance** link corresponding to the **us-east-1** row

        ここからはEC2インスタンスの作成ウィザードの手順です。

        -   インスタンスタイプ選択のページで**m5.xlarge**を選択して下さい。

        -   **Next: Configure Instance Details**を押して次へ進みます。

        -   **Network**ドロップダウンメニューで“**mod-xyz**”のような形式のVPCを選択して下さい。

        -   **Subnet**ドロップダウンメニューで“**Public Subnet 0**”を選択して下さい。

        -   その他はデフォルト設定のままにして下さい。

        -   **Next: Add Storage**をクリックして下さい。

        -   **Next: Add Tags**をクリックして下さい。

            -   **Add Tag**をクリックして下さい。

        -   以下の値を入力して下さい。

            -   Key = **Name**

            -   Value = **DataSync-agent**

        -   **Next: Configure Security Group**をクリックして下さい。

            -   “**Select an existing security group**”チェックボックスを選択して下さい。

            -   **mod-xyz**の名前で始まるセキュリティーグループを選択して下さい。

        -   **Review and Launch**をクリックして下さい。

        -   **Launch**をクリックして下さい。

        -   キーペア選択のドロップダウンメニューで**ee-default-keypair**を選択し、チェックボックスをチェックした上、**Launch Instances**をクリックして下さい。  
            

    -  AWS consoleに戻り、**Services**をクリックし、**EC2**をクリックして下さい。

        -   左のメニューから**Instances**を選択して下さい。

        -   インスタンス一覧の中から“**DataSync-agent**”を選択して下さい。

        -   下部の窓の**Description**タブの情報の中から**private IP**アドレスを探し、メモの中の**DataSync Instance Private IP=**の所に記入して下さい。

            -   次の￥ステップに進む前にEC2インスタンスの“**Status Check**”が **“2/2 checks passed“**になっている事を確認して下さい。

        -   AWS consoleに戻り、**Services**をクリックし、**DataSync**をクリックして下さい。

            -   **Get Started**をクリックして下さい。

            -   **deploy agent**のドロップダウンメニューから**Amazon EC2**を選択し、続いて以下の値を入力して下さい。

                -   **Service endpoint:** USのPublic service endpointsを選択して下さい。
                    

                -   **Activation key:** 先ほどメモした**DataSync Instance Private IP**を入力して下さい。

                -   **Get Key**をクリックして下さい。

                    -   アクティベーションが成功すると、その旨が通知されます。

                -   次に進むため**Create Agent**をクリックして下さい。

		

	-   エージェント作成完了後、左上の**DataSync**リンクをクリックして次のタスクの作成に進みます。


<br/><br/>


**DataSyncを使ったデータの転送** 
---------------------------------

1.  前の章で表示したAWS DataSync画面で、右上の**Create task**をクリックして下さい。

    -   Source location optionsのところで**Create a new location**を選択して下さい。

    -   **Location type:** Server Message Block (SMB)を選択

    -   **Agents:** デプロイしたエージェントを選択

    -   **SMB Server:** メモしたホスト名を入力
        `例 EC2AMAZ-12ID25G`

    -   **Share name:**  **Tools**を入力

    -   **Additional settings**のところで**SMB3**を選択して下さい。
    -   User settingsのところで、以下の値を入力して下さい。
	    -   User : **admin**
	    -   Password : admin@example.comのパスワードです。Secrets Managerから入手出来ます。
	    -   domain : **example**

    -   次に進むため**Next**をクリックして下さい。
	    
2.  Destination locations optionsのところで**Create a new location**を選択して下さい。

    -   **Location type:** Amazon FSx for Windows File Serverを選択

    -   **FSx file system:** リストからMAZファイルシステムを選択

    -   **Share name:** **myshare**を入力
    -   **Additional settings**をクリックし、**Security groupsドロップダウンメニュー**から以下のセキュリティーグループを選択して下さい :
     `FileSystemSecurityGroup` と `ClientSecurityGroup`

    -   User settingsのところで、以下の値を入力して下さい。
	    -   User : **admin**
	    -   Password : admin@example.comのパスワードです。Secrets Managerから入手出来ます。
	    -   domain : **example**

    -   次に進むため**Next**をクリックして下さい。

3.  タスク名を入力して下さい。 (*例 local-smb-to-FSx-for-windows-file-server-transfer*)

    -   **Verify data:** Check integrity during transfer

    -   その他の設定はデフォルトを使用し、**Task logging**のところまで下にスクロールして下さい。

        -   **Log Level :** Select Log all transferred objects

        -   ファイル転送ログを送るCloudWatch log groupを自動生成するため**Autogenerate**をクリックして下さい。


    -   その他はデフォルト設定のままで**Next**をクリックして下さい。

    -   **Create task**をクリックして下さい。

4.  **Task status**が**Available**になるまでお待ち下さい。
    (スクリーンをリフレッシュすると状況がアップデートされます)

5.  **Start**ボタンをクリックして下さい。

6.  デフォルト設定をそのまま受け入れ、**Start**をクリックして下さい。

7.  転送状況をモニターするため、画面上部の**See execution details**ボタンをクリックして下さい。

8.  タスクはデータの転送前に、タスクセットアップのチェック、ソースとターゲットの比較等のいくつかのフェーズを経ます。

9.  **Execution status**が**Success**になれば転送は完了です。

10. タスクの**Duration**やパフォーマンス状況が表示されるので、確認してみて下さい。    

11. 右上の**View logs in CloudWatch**ボタンをクリックすると、転送結果の詳細ログを見ることが出来ます。

12. 表示された画面で、**Log streams**の中身を確認してみて下さい。何も表示されていない場合、**refresh**アイコンをクリックして最新の情報に更新して下さい。最新のログが表示されるまで数分かかることが有ります。

13. ログストリームは`task-0372cba912b113e4b-exec-xyz`のような形式で表示されます。表示されたらクリックして下さい。

14. ファイル転送の詳細が確認出来ることがわかります。



<br/><br/>

**転送データの確認**
----------------------------------------------

実際にコピーされたデータを見てみましょう。

1.  Windows EC2インスタンスのファイルエクスプローラーで、**c:\Tools**フォルダーを開き、データのサイズとファイルの数を確認して下さい。
2.  コマンドプロンプト（CMD）を開き、以下のコマンドを実行して下さい。**your-fsx-DNS-name**はメモしたFSxのDNS名に入れ替えて下さい。

    `net use t: \\your-fsx-DNS-name\myshare`

3. エクスプローラーで**T:\\** **drive**を開き、データのサイズとファイルの数を確認して下さい。C:\Tools のデータと一致しますか？

<br/><br/>



**まとめ**
-----------

このモジュールでは、WindowsのSMBファイル共有からFSx for windowsのファイル共有へ、DataSyncを使ってファイル転送するハンズオン体験を行いました。

<br/><br/>

**END OF MODULE**
-------------------




