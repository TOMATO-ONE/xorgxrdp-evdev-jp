# パッチを適用したカスタムdeb pkg の作成方法

カスタムパッチを適用したdebian/ubuntu deb pkg を作成する方法<BR>
対象pkgをbuildするために必要なpkgはインストール済みであることを前提としています。

### packger 名、メールアドレスを環境変数にセットしておく。
```shell
export DEBEMAIL="junker.tomato<at>example.com"
export DEBFULLNAME="TOMATO-ONE"
```

### quilt コマンドの準備とalias設定
debian/ubuntu deb pkg のパッチ管理コマンドのquilt をインストールしaliasを作る
```bash
sudo apt install quilt -y

alias dquilt="quilt --quiltrc=${HOME}/.quiltrc-dpkg"
. /usr/share/bash-completion/completions/quilt
complete -F _quilt_completion -o filenames dquilt
```

以下内容で `${HOME}/.quiltrc-dpkg` を作成する。
```
d=. ; while [ ! -d $d/debian -a `readlink -e $d` != / ]; do d=$d/..; done
if [ -d $d/debian ] && [ -z $QUILT_PATCHES ]; then
    QUILT_PATCHES="debian/patches"
    QUILT_PATCH_OPTS="--reject-format=unified"
    QUILT_DIFF_ARGS="-p ab --no-timestamps --no-index --color=auto"
    QUILT_REFRESH_ARGS="-p ab --no-timestamps --no-index"
    QUILT_COLORS="diff_hdr=1;32:diff_add=1;34:diff_rem=1;31:diff_hunk=1;33:diff_ctx=35:diff_cctx=33"
    if ! [ -d $d/debian/patches ]; then mkdir $d/debian/patches; fi
fi
```

### apt の source.list の deb-src 行を有効にする。
```bash
sudo sed -i.bak -e "s/^# deb-src http)/deb-src http/g"  /etc/apt/sources.list
sudo apt update
```
### 元となるsource pkg をダウンロードする。
```
apt source xorgxrdp
```

### 新しいリビジョンに更新する場合

新しいリビジョンのソースアーカイブを配置した上でソース展開ディレクトリに移動し、以下コマンドを実行する。
```
uupdate -v 0.9.19 ../xorgxrdp-0.9.19.tar.gz
```
新しいソースコードを元に新しいディレクトリが作られるので移動する。
```
cd ../xorgxrdp-0.9.19
```

### pkgバージョンを設定する。
`dch -v <porch>:<source version>-<debian version> -b`

```bash
dch -v 1:0.9.19-1 -b 
```
editor が起動し、changesを記載する。

### controle ファイル修正
依存pkgの記述を修正し、説明を加える
```bash
vim ./debian/control
```

### rule ファイル修正
configure オプションを変更する場合は、ruleファイルを修正する。
```bash
vim ./debian/rule
```

### patch 追加
quiltコマンドに頼らずファイルパッチを作る場合は `diff -uNr ` または `git diff` で作成し、 debian/patches ディレクトリにコピーする。

quilt コマンドでimport する

```bash
 dquilt import /mnt/xorgxrdp_fix_jp106key_scancode.diff
 dquilt series
```

手動で行う場合は、debian/patches にコピーし、./dbian/patches/series に追記
```
 cp /mnt/xorgxrdp_fix_jp106key_scancode.diff ./debian/patches/
vim ./debian/patches/series
```

### patch 適用
```bash
while dquilt push; do dquilt refresh; done
```
(必要に応じて)パッチを削除する場合は、以下のようにする。
```bash
 dquilt delete debian/patches/xorgxrdp_fix_jp106key_scancode.diff
 dquilt series
```
(必要に応じて)パッチにヘッダを追記
```
dquilt header -e
```
### バックポートバージョンの追記
バックポート番号(~bpXX)を追記する場合は、以下を実行する。
```bash
dch --bpo
```

### dpkg build
dpkg のビルド実施
```
dpkg-buildpackage -us -uc -b
```

srcpkg も作成する場合
```
dpkg-buildpackage -us -uc
```

### 生成ファイル
親ディレクトリにdebファイルが生成される。

src pkg としては
.dsc ファイル、.debian.tar.xz 、ソースアーカイブの3点が最低限現必要。
.orig.tar.gz は オリジナルアーカイブのリンクである。



