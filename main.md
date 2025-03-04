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

