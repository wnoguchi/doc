# Chef

やるぞ！  
ここではchef-solo。  
CentOSはイバラの道らしいのでUbuntu13.04で。

## Getting Started

### インストール

```
curl -L http://www.opscode.com/chef/install.sh | sudo bash
```

### レポジトリの作成

```
git clone git://github.com/opscode/chef-repo.git
```

```
knife configure
```

### クックブック作成

```
cd chef-repo/
knife cookbook create hello -o cookbooks
** Creating cookbook hello
** Creating README for cookbook: hello
** Creating CHANGELOG for cookbook: hello
** Creating metadata for cookbook: hello
```

### レシピ編集

```ruby
# cookbooks/hello/recipes/default.rb
#
# Cookbook Name:: hello
# Recipe:: default
#
# Copyright 2013, YOUR_COMPANY_NAME
#
# All rights reserved - Do Not Redistribute
#

log "Hello, Chef!"
```

### 実行するレシピの一覧の定義

```json
// localhost.json
{
  "run_list": [
    "recipe[hello]"
  ]
}

```

### chef-soloの設定

```ruby
# solo.rb
file_cache_path "/tmp/chef-solo"
cookbook_path [ "/home/wnoguchi/chef-repo/cookbooks" ]
```

### 実行

```
sudo chef-solo -c solo.rb -j ./localhost.json

Starting Chef Client, version 11.6.0
Compiling Cookbooks...
Converging 1 resources
Recipe: hello::default
  * log[Hello, Chef!] action write

Chef Client finished, 1 resources updated

```

zshを入れてみる

```ruby
package "zsh" do
  action :install
end
```

さらにRubyの文法を活用してみる。  
ちなみにAmazon LinuxではなくてUbuntuでやってるのでパッケージ名はUbuntuのパッケージングリポジトリに載ってる名前でやらないとエラーになるみたい。

```ruby
%w{zsh gcc make libreadline-dev vim}.each do |pkg|
  package pkg do
    action :install
  end
end
```

出力は以下のとおり。

```
Starting Chef Client, version 11.6.0
Compiling Cookbooks...
Converging 6 resources
Recipe: hello::default
  * log[Hello, Chef!] action write

  * package[zsh] action install
    - install version 5.0.0-2ubuntu3 of package zsh

  * package[gcc] action install (up to date)
  * package[make] action install (up to date)
  * package[libreadline-dev] action install
    - install version 6.2-9ubuntu1 of package libreadline-dev

  * package[vim] action install
    - install version 2:7.3.547-6ubuntu5 of package vim

Chef Client finished, 4 resources updated
```

## 参考サイト

- [Amazon.co.jp： 入門Chef Solo - Infrastructure as Code eBook: 伊藤直也: Kindleストア](http://www.amazon.co.jp/%E5%85%A5%E9%96%80Chef-Solo-Infrastructure-Code-ebook/dp/B00BSPH158)  
主にこれを参考にさせてもらっています。
- [特集　DevOps時代の必須知識：インフラストラクチャ自動化フレームワーク「Chef」の基本 (1/2) - ＠IT](http://www.atmarkit.co.jp/ait/articles/1305/24/news003.html)
- [Rubyist Magazine - Chef でサーバ管理を楽チンにしよう！ (第 1 回)](http://magazine.rubyist.net/?0035-ChefInDECOLOG)
- [Chef Soloの正しい始め方 | tsuchikazu blog](http://tsuchikazu.net/chef_solo_start/)
- [Chef Soloと Knife Soloでの　ニコニコサーバー構築 (1):dwango エンジニア ブロマガ:ドワンゴ研究開発チャンネル(ドワンゴグループのエンジニア) - ニコニコチャンネル:生活](http://ch.nicovideo.jp/dwango-engineer/blomaga/ar311555)
