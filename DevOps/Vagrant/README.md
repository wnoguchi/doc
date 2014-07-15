Vagrant
=========

```
vagrant ssh-config --host foobar | tee -a ~/.ssh/config
```

やっと余裕が出てきたので、最近戯れてなかったVagrantをバージョンアップしてみようという気になった。

インストール前
----------------

```zsh
[noguchiwataru@Macintosh] ~
% vagrant --version
Vagrant 1.3.5
```

インストール後
----------------

```zsh
[noguchiwataru@Macintosh] ~
% vagrant --version
Vagrant 1.6.3
```

いつの間にかメジャーバージョンが3も上がってた。

vagrant init してみる
----------------------

```zsh
% vagrant init
Vagrant experienced a version conflict with some installed plugins!
This usually happens if you recently upgraded Vagrant. As part of the
upgrade process, some existing plugins are no longer compatible with
this version of Vagrant. The recommended way to fix this is to remove
your existing plugins and reinstall them one-by-one. To remove all
plugins:

    rm -r ~/.vagrant.d/plugins.json ~/.vagrant.d/gems

The error message is shown below:

Bundler could not find compatible versions for gem "celluloid":
  In Gemfile:
    vagrant (= 1.6.3) ruby depends on
      listen (~> 2.7.1) ruby depends on
        celluloid (>= 0.15.2) ruby

    vagrant-berkshelf (>= 0) ruby depends on
      berkshelf (~> 2.0.7) ruby depends on
        ridley (~> 1.5.0) ruby depends on
          celluloid (0.14.1)
```

バージョンが衝突したからとりあえずプラグイン全部消して1つずつ入れなおしてくれって書いてある。
しばらくつかってなかったからもうプラグインも全部いらないかなって思って漢らしく

```zsh
[noguchiwataru@Macintosh] ~/vagrant/centos
% rm -rf ~/.vagrant.d/plugins.json ~/.vagrant.d/gems
```

デフォルトでインストールされているプラグインはこれ。

```
[noguchiwataru@Macintosh] ~
% vagrant plugin list
vagrant-login (1.0.1, system)
vagrant-share (1.1.0, system)
```

改めて `vagrant init` してみる。

なんだかVagrantの内部状態をアップグレードしてるから殺してくれるなと言ってるっぽい。  
いくらかディスク容量使うよとも言っている。  
とくに問題ないので待つ。

```
% vagrant init
Vagrant is upgrading some internal state for the latest version.
Please do not quit Vagrant at this time. While upgrading, Vagrant
will need to copy all your boxes, so it will use a considerable
amount of disk space. After it is done upgrading, the temporary disk
space will be freed.

Press ctrl-c now to exit if you want to remove some boxes or free
up some disk space.

Press any other key to continue.
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.
```

`Press any other key to continue.` と表示されたところでそのままエンターする。

これでおしまい。

Vagrant boxイメージをインポートする
-----------------------------------

いつもならHashiCorpあたりで提供されているCentOSのboxイメージを使うか、
Packer等で自作するか、より手軽にイメージが欲しければ

[A list of base boxes for Vagrant - Vagrantbox.es](http://www.vagrantbox.es/)

を使うんだけど、　Vagrantbox.es　はプルリクがあったら特に確認しないでバカスカリンクを突っ込んでるからあんまり推奨されてなかった。  
前ダウンロードしたUbuntu Serverのboxイメージのaptがやたら遅いと思ったらブラジル向いてたよ。

Getting Startedを真面目に読むと

[Getting Started - Vagrant Documentation](http://docs.vagrantup.com/v2/getting-started/)

```
vagrant init hashicorp/precise32
```

すればVagrantfileが生成される。
`vagrant up` すれば勝手にUbuntuのboxがダウンロードされてくるようだ。  
確かに動いたけど、なんでURLも指定しないでダウンロードできたのが意味がわからないし、  
僕はCentOSのboxイメージが欲しかったのでもう少し調べてみた。

[Boxes - Vagrant Documentation](http://docs.vagrantup.com/v2/boxes.html)

によると

>The easiest way to use a box is to add a box from the publicly available catalog of Vagrant boxes. You can also add and share your own customized boxes on this website.

もっとも簡単にboxを追加する方法は公開されたVagrant boxのカタログから取ってくるのが簡単だよって言ってる。

[Vagrant Cloud](https://vagrantcloud.com/)

なんだか今のバージョンではVagrant Cloudにアカウントが無くてもダウンロードできるっぽい。

今登録されているのは以下。

```
[noguchiwataru@Macintosh] ~
% vagrant box list
Berkshelf-CentOS-6.3-x86_64-minimal (virtualbox, 0)
base                                (virtualbox, 0)
chef/centos-6.5                     (virtualbox, 1.0.0)
dummy                               (aws, 0)
hashicorp/precise32                 (virtualbox, 1.0.0)
```

- [chef](https://vagrantcloud.com/chef)

とかメジャーな組織がメンテナンスしているboxが取ってくることができるので Vagrantbox.es からとってくるよりよっぽど安心できる。

References
------------

1. [Getting Started - Vagrant Documentation](http://docs.vagrantup.com/v2/getting-started/)
1. [Boxes - Vagrant Documentation](http://docs.vagrantup.com/v2/boxes.html)
1. [Vagrant Cloud](https://vagrantcloud.com/)
1. [chef](https://vagrantcloud.com/chef)
1. [vagrantのboxをvagrant cloudからもらってくる - わすれっぽいきみえ](http://kimikimi714.hatenablog.com/entry/2014/04/05/vagrant%E3%81%AEbox%E3%82%92vagrant_cloud%E3%81%8B%E3%82%89%E3%82%82%E3%82%89%E3%81%A3%E3%81%A6%E3%81%8F%E3%82%8B)
1. [A list of base boxes for Vagrant - Vagrantbox.es](http://www.vagrantbox.es/)
