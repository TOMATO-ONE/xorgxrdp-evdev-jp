# パッチを適用したカスタムrpm pkg の作成方法

カスタムパッチを適用した rpm pkg を作成する方法<BR>
対象srpmをbuildするために必要なrpm pkgはインストール済みであることを前提としています。<BR>
また、EPELリポジトリ設定済みの前提としています。

## srpms の準備
```bash
# EL8/EL9
dnf download --repo epel-source --source xorgxrdp

# EL7
yumdownloader --enablerepo=epel-source --source xorgxrdp 

```

## src rpm のインストール
```bash
rpm -Uvh xorgxrdp-0.9.19-1.el8.src.rpm

```
ホームディレクトリに、rpmbuild ディレクトリとともにSOURCE.SPECS のサブディレクトリが作成され、ソースコードやSPECファイルが展開される。

## (必要に応じて)新しいリビジョンのソースパッケージをコピー
新しいリビジョンのソースパッケージを　~/rpmbuild/SOURCE/ 以下にコピーする。


## パッチをコピー
diff -uNr または git diif にて 作成したパッチを ~/rpmbuild/SOURCE/ 以下にコピーする。

```bash
cp /mnt/xorgxrdp_fix_jp106key_scancode.patch  ~/rpmbuild/SOURCES/
```

## SPECファイルを修正
 ~/rpmbuild/SPECS 以下に展開された spec ファイルを修正する。 以下の部分を特に着目して修正を加える。

- Version:行、Revision行を必要に応じて書き換え。

- 同じバージョン、リリースのrpm の場合に新旧を判断する情報としてEpoch: がある。
プライベートビルドの場合には元のバージョンのEpoch: の値に+1 すると良い。

- Source行の直後に Patch0: ～ 始まるパッチ行を付加する。
~/rpmbuild/SOURCES/以下に置いたパッチファイル名を指定する。
元のSPECファイルに合わせて%prep セクション等を修正する。

- %changelog の直後に変更履歴を記述する。

## rpmbuild
```bash
rpmbuild -bb ~/rpmbuild/SPECS/xorgxrdp.spec
```

src rpmも作成する場合
```bash
rpmbuild -bb ~/rpmbuild/SPECS/xorgxrdp.spec
```

~/rpmbuild/RPMS/x86_64/ 　以下にrpm パッケージが、<BR>
~/rpmbuild/SRPMS/　以下に src rpmパッケージが生成される。

