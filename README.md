# xorgxrdp-evdev-jp
evdev patched xorgxrdp for JP106/109 Japanese keyboard<BR>
based on a patch by [@mhoffmann75](https://github.com/mhoffmann75)

## これは何か？
106/109日本語キーボードのX11 scancode を送信できるようにパッチを適用したxorgxrdp です。<br>
CentOS6/7, RockyLinux8/9 のRPM/SRPM で、Xrdp 0.9.20 に対応します。

xorgxrdp version 0.9.19 は カーソルキー等一部のX11のscancode にズレが生じています。
通常のX11アプリケーションではkeysymの値を使うため実質的な問題は起きませんが、
X11 scancode を直接参照する一部のアプリケーションで想定と異なるキーが押される状態になります。<br>
このパッチを適用したxorgxrdpはX11 scancode を参照する一部のアプリケーションにおいて正しいキー入力を行えるようになります。

以下の４つの条件全てに当てはまる場合にこのパッチ適用済みxorgxrdpが意味を持ちます。

 1. 日本語Windows OS上のリモートデスクトップ接続(mstsc.exe)を使っている
 2. Xrdp をインストールしたLinuxデスクトップに接続している。
 3. XvncドライバではなくXorgドライバ `xorgxrdp` を用いている。
 4. Linux デスクトップ 上で X11 scancode を参照するアプリケーションを利用している。 
    -  例）
        - VMware vSphere HyperVisoer の WebGUI版仮想マシンコンソール
        - HPE Proliant Server に搭載された iLO4/5 HTML5 コンソール

## 具体的に何をするものか
- xorgxrdp の input driver を base から evdev に変更します。
- JP106/109日本語キーボードの以下のキーが生成するX11 scancode を変更します。

|キー名称|標準<BR>xorgxrdp<BR>X11 Scancode|パッチ適用済み<BR>xorgxrdp<BR>X11 Scancode|備考|
|:------------:|:------------:|:------------:|:------------:|
|右 Ctrl     |109|105||
|右 ALT  |113|106||
|左Windows |115|133|X11 Scancode は左右Winで同一に|
|右Windows  |116|133|X11 Scancode は左右Winで同一に|
|Menu|117|135||
| /  |112|106||
|\\(backslash)|-|97|右shiftの左側の位置「ろ」のキー|
|\\(yen)|-|125|BackSpace左側の位置「￥」キー|
|Enter|108|104||
|PrintScreen|111|107||
|Insert|106|118||
|Home|97|110||
|Delete|107|119||
|End|103|115||
|PageUp|99|112||
|PageDown|105|117||
|カーソル上↑|98|111||
|カーソル左←|100|113||

## upstreamに PR できない理由
JP106/109日本語キーボードを前提としたパッチとなっているためです。

US/UK英語など、標準的な101/104 ASCII配列キーボードレイアウトのキーボードの場合にはこのパッチはおそらく衝突しませんが、全ての言語・キーボードでのテストを実施しておりません。

具体的な問題としては、[ブラジル配列のポルトガル語キーボード](https://ja.wikipedia.org/wiki/ポルトガル語キー配列#/media/ファイル:KB_Portuguese_Brazil_text.svg)の [?] キーの既存の定義が日本語JP106/109キーボードのBakcSpace左側の位置の「￥」キー と衝突するため、このパッチでは強引に上書き処理してしまっています。

### テスト済み環境
|パッチ有用性|通常のX11アプリ使用|言語|OS|キーボードレイアウト|製品名|備考|
|:----:|:----:|:----:|:----:|:----:|:----:|:----:|
|◎|○|日本語|Windows10 Pro 21H2 | JP109|Logicool MX800|右Windowsキー、メニューキーなし|
|○|○|日本語|Windows10 Home 21H2 | US104|CHUWI Hi10pro <BR>専用キーボード|テストの範囲では支障なし|
|×|▲|日本語|macOS Monterey| JIS|MacBook Pro 2020(M1)|setxmodmap で layoutがusで認識される。<BR>[￥]キーが効かない|

## 既知の事象
  - JIS日本語キーボードを取り付けたmacOS Monterey で Microsoft RemoteDesktop Client にて接続したとき、 [￥] キーが効かない。
    - [Option] キー+[ ￥] で代用してください。

## 参考URL
- [XRDP Keyboard evdev support? #211](https://github.com/neutrinolabs/xorgxrdp/issues/211)
  - [@mhoffmann75](https://github.com/mhoffmann75)による[evdev化パッチ](https://github.com/mhoffmann75/xorgxrdp/commit/16b8fbdc3345a0caa56cb9109790ab6dbe6df892)をベースとしてこれを作成しました。
- [wikipedia キー配列](https://ja.wikipedia.org/wiki/キー配列)
- [wikipedia ポルトガル語キーボード配列](https://ja.wikipedia.org/wiki/ポルトガル語キー配列)
