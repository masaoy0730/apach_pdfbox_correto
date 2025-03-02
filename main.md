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
```
yum update #パッケージリスト更新
yum install sudo #sudoのインストール
usermod -aG wheel <ユーザー名> #ユーザー名の追加
```

ダウンロードしたアーカイブを解凍し、`/opt`に配置。シンボリックリンク `/opt/maven` を作成しておくと、将来バージョンを切り替える際に便利です。
```
sudo tar -xzvf apache-maven-3.9.9-bin.tar.gz -C /opt/
sudo ln -s /opt/apache-maven-3.9.9 /opt/maven

```
