# GCP Cloud KMS使って、セキュリティーデータファイルを暗号化

実施手順のまとめ

- キーリングおよび暗号鍵の作成
- KMSの暗号鍵で秘密ファイルを暗号化する
- 秘密ファイルを利用する際、暗号鍵で復号化する

## キーリングおよび暗号鍵の作成
Cloud Build で SSH 認証鍵を使用するには、Cloud KMS CryptoKey を使用して鍵を暗号化、復号する必要があります。CryptoKey は KeyRing オブジェクトに格納されます。

KeyRing と CryptoKey を作成するには、gcloud kms keyrings create および gcloud kms keys create コマンドを使用します。

KeyRing を作成するには、shell またはターミナル ウィンドウで次のコマンドを実行します。このチュートリアルでは、KeyRing my-private-keyring に名前を付けます。

```sh
# Create a Cloud KMS KeyRing
gcloud kms keyrings create my-private-keyring --location=global
```

コンソール画面で作成結果確認
<img width="887" alt="gcp_kms_001.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/535707/c33ba259-e6d4-7031-7286-ac831aa1a0e6.png">

次に、my-private-key という CryptoKey を作成します。このコマンドは、my-private-keyring KeyRing に my-private-key CryptoKey を作成します。

```sh
# create a CryptoKey 
gcloud kms keys create my-private-key --location=global --keyring=my-private-keyring --purpose=encryption
```

コンソール画面で作成結果確認
<img width="1195" alt="gcp_kms_002.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/535707/aaa7a9e2-5059-47f9-300f-d2f7ffc20bab.png">


## KMSの暗号鍵で秘密ファイルを暗号化する
秘密ファイルがあります。

```sh:my_secret.txt
I'm falling in love with Aosora
```

これは見られたら大変なので暗号化をします。
暗号化するには、gcloud kms encrypt コマンドを使用します。

次のコマンドを実行します。正しいパスを指定してください。

```sh
# Encrypt a secret
gcloud kms encrypt --plaintext-file=/full/path/to/my_secret.txt \
--ciphertext-file=/full/path/to/output/my_secret.txt.enc   \
--location=global --keyring=my-private-keyring --key=my-private-key

# 秘密を暗号化しましたので、生データを削除します。
rm my_secret.txt
```

暗号化（encrypt）するため、'cloudkms.cryptoKeyVersions.useToEncrypt'権限を持つアカウントで実施する必要です。
通常ProjectのEditorでも、この権限はありません。
Ownerアカウントで実施するか、この権限を個別で追加するかという対策は必要です。


## 秘密ファイルを利用する際、暗号鍵で復号化する

```sh
# Decrypt a secret
gcloud kms decrypt --ciphertext-file=/full/path/to/my_secret.txt.enc \
--plaintext-file=/full/path/to/output/my_secret.txt \
--location=global --keyring=my-private-keyring --key=my-private-key
```

復号化（decrypt）するため、'cloudkms.cryptoKeyVersions.useToDecrypt'権限を持つアカウントで実施する必要です。
通常ProjectのEditorでも、この権限はありません。
Ownerアカウントで実施するか、この権限を個別で追加するかという対策は必要です。


<br>
ご覧して頂き、どうも有難う御座います!
DevSamurai Ben