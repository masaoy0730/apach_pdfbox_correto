# Amazon CorrettoをDockerで動かす方法
## 前提条件
1.Docker Desktop for Macがインストールされていること。
2.Amazon CorrettoのDockerイメージを使用するための基本的なコマンドライン操作が可能であること。

Finderから「Docker.app」を起動し、Dockerが正常に動作していることを確認します。
ターミナルで以下のコマンドを実行し、Dockerがインストールされているか確認します：
```
docker --version
```
## Amazon Correttoの公式イメージを取得
•	Docker Hubには、Amazon Correttoの公式イメージが提供されています。
•	以下のコマンドで必要なバージョンのCorrettoイメージを取得します：
Amazon Corretto 11の場合：
```
docker pull amazoncorretto:11
```
• ダウンロードしたイメージを使ってコンテナを起動します。
```
docker run -it --name corretto-container amazoncorretto:11 /bin/bash
```
このコマンドで、`/bin/bash`シェルに入ります。コンテナ内でJava環境が利用可能です。

コンテナ内で以下のコマンドを実行し、Java環境が正しく設定されているか確認します：
```
java -version
```

2. sudoのインストール方法
もし`sudo`がインストールされていない場合、以下の手順でインストールできます。
Amazon Linux 2の場合
1.	`yum`を使用してインストール：
```
yum install -y sudo
```
```
sudo --version
```
 


### Apach Mavenのインストール
Bashを使ってLinux環境にMavenをインストールする手順。

OSの名前やバージョン情報を確認
```
cat /etc/os-release
```

Linuxディストリビューションの詳細確認
```
lsb_release -a
```

Javaのインストールを確認。
```
java -version
echo $JAVA_HOME
```
/optに公式サイトからMavenをダウンロード。
```
curl -OL https://downloads.apache.org/maven/maven-3/3.9.9/binaries/apache-maven-3.9.9-bin.tar.gz
```
sudoが使えない場合、インストールする。

NAME="Amazon Linux"<br>
VERSION="2"<br>
ID_LIKE="centos rhel fedora"<br>
なので、Red Hat系ディストリビューションのコマンドを利用。

```
yum update #パッケージリスト更新
yum install sudo #sudoのインストール
```
usermodがインストールされていなければ、インストール。
```
sudo yum install util-linux-user
```
`usermod`コマンドは管理者権限が必要。一般ユーザーで実行するとエラー発生。
rootユーザーに切り替え:
```
su -
```
動作確認
```
usermod --help
```
```
usermod -aG wheel <ユーザー名> #ユーザー名の追加
```
tarをインストール
```
sudo yum install tar
```
ダウンロードしたアーカイブを解凍し、`/opt`に配置。シンボリックリンク `/opt/maven` を作成しておくと、将来バージョンを切り替える際に便利です。
```
sudo tar -xzvf apache-maven-3.9.9-bin.tar.gz -C /opt/
sudo ln -s /opt/apache-maven-3.9.9 /opt/maven
```
Mavenプロジェクトの作成
```
mvn archetype:generate -DgroupId=com.example -DartifactId=my-app -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

BashでDockerコンテナを終了し、インストールしたものを保存するには以下の手順を実行します。
### **1. コンテナを終了する**
Dockerコンテナを終了するには、まずコンテナから抜けます。

#### **方法**
1. **コンテナ内で`exit`コマンドを実行**:
   ```bash
   exit
   ```
   - これにより、コンテナのセッションから抜け出し、ホスト環境に戻ります。
   - コンテナは停止状態になります。

2. **コンテナを明示的に停止**（必要に応じて）:
   ```bash
   docker stop <CONTAINER_IDまたはNAME>
   ```
   - 起動中のコンテナIDや名前は以下のコマンドで確認できます。
     ```bash
     docker ps
     ```
### **2. コンテナの状態を保存する**
変更した内容やインストールしたパッケージを保存するために、現在のコンテナ状態をDockerイメージとして保存します。

#### **手順**
1. **現在のコンテナIDまたは名前を確認**:
   ```bash
   docker ps -a
   ```
2. **`docker commit`でイメージとして保存**:
   ```bash
   docker commit <CONTAINER_ID> <REPOSITORY_NAME>:<TAG>
   ```
   - 例:
     ```bash
     docker commit abc123def456 my_saved_image:latest
     ```
   - `my_saved_image`という名前で、現在のコンテナ状態がイメージとして保存されます。

3. **保存したイメージを確認**:
   ```bash
   docker images
   ```
   - 出力例:
     ```
     REPOSITORY       TAG       IMAGE ID       CREATED          SIZE
     my_saved_image   latest    789abc123def   10 seconds ago   200MB
     ```
### **3. 保存したイメージから再利用**
保存したイメージを使って新しいコンテナを作成できます。

#### **手順**
1. 新しいコンテナを作成して起動:
   ```bash
   docker run -it --name <NEW_CONTAINER_NAME> <REPOSITORY_NAME>:<TAG>
   ```
2. コンテナ内で変更が反映されていることを確認します。

### **4. 別環境への移行（オプション）**
保存したイメージを別の環境で利用する場合は、以下の手順で進めます。

#### **手順**
1. イメージをファイルとしてエクスポート:
   ```bash
   docker save -o <FILE_NAME>.tar <REPOSITORY_NAME>:<TAG>
   ```
2. ファイルを別のサーバに転送（例: `scp`コマンド）:
   ```bash
   scp my_saved_image.tar user@remote-server:/path/to/destination/
   ```
3. 別環境でイメージをインポート:
   ```bash
   docker load -i /path/to/destination/my_saved_image.tar
   ```
4. インポートしたイメージから新しいコンテナを作成して起動:
   ```bash
   docker run -it --name new-container my_saved_image:latest
   ```
### **5. 注意点**
- **ディスクスペース**: 保存したイメージが大きくなる場合があるため、不要なファイルやキャッシュは削除しておくと良いです。
- **バージョン管理**: イメージにタグ（例: `v1`, `v2`）を付けることでバージョン管理が容易になります。
  ```bash
  docker tag my_saved_image:latest my_saved_image:v1
  ```
これらの手順で、Dockerコンテナの状態を安全に保存しつつ終了し、後から再利用できるようになります。


Dockerコンテナを再起動するには、以下の方法を使用します。
### **1. 再起動コマンド**
Dockerコンテナを再起動するには、`docker restart`または`docker container restart`コマンドを使用します。

#### **コマンド例**
```bash
docker restart <コンテナ名またはコンテナID>
```
または
```bash
docker container restart <コンテナ名またはコンテナID>
```

#### **使用例**
- コンテナ名が`my-container`の場合:
  ```bash
  docker restart my-container
  ```

- コンテナIDが`abc123def456`の場合:
  ```bash
  docker restart abc123def456
  ```

---

### **2. 複数のコンテナを再起動**
複数のコンテナを同時に再起動することも可能です。コンテナ名やIDをスペースで区切って指定します。

#### **コマンド例**
```bash
docker restart container1 container2 container3
```

---

### **3. 再起動時のオプション**
- `--time` オプションを使用して、停止時に待機する秒数を指定できます（デフォルトは10秒）。
  
#### **コマンド例**
```bash
docker restart --time=5 my-container
```
- この例では、停止時に5秒待機してから再起動します。

---

### **4. コンテナの状態確認**
再起動前後にコンテナの状態を確認するには、以下のコマンドを使用します。

#### **実行中のコンテナ一覧**
```bash
docker ps
```

#### **すべてのコンテナ一覧（停止中も含む）**
```bash
docker ps -a
```

---

### **5. 注意点**
1. **保存したいデータがある場合**:
   - 再起動しても、通常はコンテナ内でインストールしたものや変更した内容は保持されます。
   - ただし、**一時的なコンテナ**（`--rm`オプション付きで起動したもの）は停止時に削除されるため、保存できません。

2. **変更内容をイメージとして保存**:
   - 再起動後も変更内容を確実に残したい場合は、以下のコマンドでイメージとして保存してください。
     ```bash
     docker commit <コンテナ名またはID> <新しいイメージ名>:<タグ>
     ```
     - 例: 
       ```bash
       docker commit my-container my-saved-image:latest
       ```

3. **再起動ポリシー**:
   - コンテナが自動的に再起動するよう設定するには、`--restart`オプションを使用して起動時にポリシーを指定します（例: `always`, `on-failure`）。
     ```bash
     docker run --restart=always <イメージ名>
     ```

Dockerコンテナを再起動した後、ターミナルから`bash`を使うには、以下の手順を実行します。
### **1. コンテナを再起動**
まず、停止中または実行中のコンテナを再起動します。
```bash
docker restart <コンテナ名またはコンテナID>
```
### **2. コンテナ内で`bash`を使用する**
再起動したコンテナ内で`bash`を使用するには、`docker exec`コマンドを使います。
```bash
docker exec -it <コンテナ名またはコンテナID> /bin/bash
```

#### **例**
```bash
docker exec -it my-container /bin/bash
```

- **`-i`**: 標準入力を有効化（対話的操作）。
- **`-t`**: 仮想端末（TTY）を割り当て。
- **`/bin/bash`**: コンテナ内で実行するシェルコマンド。

---

### **3. コンテナ内での操作**
上記コマンドを実行すると、コンテナ内に入ってBashプロンプトが表示されます。

#### **例**
```bash
root@<container_id>:/#
```

ここから通常のLinuxシェル操作が可能になります。

---

### **4. 注意点**
1. **Bashがインストールされていない場合**:
   - 一部の軽量なベースイメージ（例: Alpine Linux）ではBashがインストールされていない場合があります。その場合は、以下のようにBashをインストールしてください。
     ```bash
     apk add --no-cache bash
     ```
   - または、代わりに`sh`などの軽量シェルを使用します。
     ```bash
     docker exec -it my-container /bin/sh
     ```

2. **再起動前に保存したデータ**:
   - 再起動しても、通常はデータやインストール済みのアプリケーションは保持されます。ただし、一時的なコンテナ（`--rm`オプション付きで起動したもの）は再利用できません。

3. **コンテナが終了している場合**:
   - 再起動前にコンテナが停止しているか確認します。
     ```bash
     docker ps -a
     ```
   - 停止中の場合は、まず以下のコマンドで開始してください。
     ```bash
     docker start <コンテナ名またはコンテナID>
     ```
### **まとめ**
1. コンテナを再起動：
   ```bash
   docker restart <コンテナ名>
   ```
2. 再起動後にBashで操作：
   ```bash
   docker exec -it <コンテナ名> /bin/bash
   ```
これでターミナルから再びコンテナ内でBashを使った操作が可能になります。
