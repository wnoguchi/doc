# Vyatta

## タイムゾーンの設定

```
set system time-zone Asia/Tokyo
```

## ログイン時のバナーを変える

```
set system login banner post-login "Welcome!"
```

## 公開鍵認証方式のセットアップ

デフォルトでは公開鍵認証は有効になっています。 

`~/.ssh/authorized_keys` は Vyatta
によって自動的に上書きされるので直接編集は禁忌です。
既に公開鍵があるものと仮定します。 事前にアップロードしておきましょう。 

```
loadkey vyatta 鍵ファイル名
```

注意として、公開鍵のフォーマットは 

```
ssh-rsa ***** username@hostname
```

のような形式になっていないといけません。
なんの変哲もないOpenSSH形式の公開鍵ですが、コメントの部分が username@hostname
になっていないといけないのです。
このコメントの部分にあまりフリーダムな内容を書いていると Vyatta
に蹴っ飛ばされます。 また、パスワードログインを無効にしましょう。 

```
set service ssh disable-password-authentication
```

最後に、 

```
commit
save
```

おしまいです。 

## 設定を破棄したい場合

```
discard
exit
```

