# ローカルでAmazonLinux2023+Laravelの環境構築する用のコンテナ設定

※あくまでAmazonLinux2からAmazonLinux2023へ移行する際の動作検証環境を手っ取り早く構築するための設定です。

2024/9時点でAmazonLinux2023にはPHP8.3.10がインストールされます。

PHPのバージョンで他のものが必要な場合はコンテナ内で調整するか、「docker/amazonlinu2023/Dockerfile」を編集してください。


## 1. docker-compose.yamlを作成し、apache、nignxの設定を行う
docker-compose.yaml.sampleからdocker-compose.yamlを作成します。

apacheを利用する場合は、「web-nignx」のコンテナ設定を削除

nignxを利用する場合は、「web-httpd」のコンテナ設定を削除


## 2. Laravelソースの設置
1. ./src/applicationにソースをおく
2. ./src/application/.envを作成
3. データベースはmysqlの初期設定の値を入れる
```
DB_CONNECTION=mysql
DB_HOST=mysql
DB_PORT=3306
DB_DATABASE=laravel-app
DB_USERNAME=laravel-app
DB_PASSWORD=laravel-app
```

4. 後の設定は必要に応じて設定


## 3. 証明書の設置
ホスト側での作業。

初回起動や証明書の有効期限が切れた場合、docker起動前に設置しておく。

(ドメインは「localhost」が前提なので、証明書のドメインを「localhost」以外にする場合は「nignx/conf/default.conf」のserver_nameも変更すること。)

1.  キーファイルの作成
```
openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -aes-256-cbc -out ./ssl-certs/localhost-key.pem
```

2. crtファイルの作成
```
openssl req -new -key ./ssl-certs/localhost-key.pem -out ./ssl-certs/localhost-crt.pem
```
パスワードと必要な情報を入力

3.　証明書の発行
```
openssl x509 -req -in ./ssl-certs/localhost-crt.pem -signkey ./ssl-certs/localhost-key.pem -days 90 -out ./ssl-certs/localhost.pem
```

4.　パスワードなしのキーファイルを作成
```
openssl rsa -in ssl-certs/localhost-key.pem -out ssl-certs/localhost-key-nopass.pem 
```

## 4. コンテナの起動
```
docker-compose up -d 
```
でコンテナをビルド・起動させる


## 5. システム（Laravel）の設定
「laravel-app」コンテナに入って、「./src/application」に設置したLaravelのインストールをします。

設定はシステムごとに違うと思うので、「./src/application」内のReadmeなどに従ってください。

## 6. ブラウザでシステムにアクセス
http://localhost

と

https://localhost

にアクセスできればOK


### SSLの環境が不要な場合
Nignxの場合
nginx/conf/default.confにある下の部分を削除してNignxのコンテナを再起動
```
    listen 443 ssl;
    listen [::]:443 ssl;
    server_name localhost;
    ssl_certificate /etc/nginx/certs/localhost.pem;
    ssl_certificate_key /etc/nginx/certs/localhost-key-nopass.pem;
```

Apacheの場合
httpd/conf/localhost.confにある下の部分を削除してApacheのコンテナを再起動
```
<VirtualHost *:443>
    DocumentRoot "/var/www/html/src/public"
    ServerName localhost:443
    ServerAdmin you@example.com
    ErrorLog /proc/self/fd/2
    TransferLog /proc/self/fd/1

    SSLEngine on
    SSLCertificateFile "/etc/httpd/certs/localhost.pem"
    SSLCertificateKeyFile "/etc/httpd/certs/localhost-key-nopass.pem"
    
    BrowserMatch "MSIE [2-5]" \
             nokeepalive ssl-unclean-shutdown \
             downgrade-1.0 force-response-1.0
    
    CustomLog /proc/self/fd/1 \
              "%t %h %{SSL_PROTOCOL}x %{SSL_CIPHER}x \"%r\" %b"
    
</VirtualHost>
```
