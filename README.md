# xorgxrdp-evdev-jp
evdev patched xorgxrdp for JP106/109 Japanese keyboard<BR>
based on a patch by [@mhoffmann75](https://github.com/mhoffmann75)

## これは何か？
106/109日本語キーボードのX11 scancode を送信できるように[パッチ](https://github.com/TOMATO-ONE/xorgxrdp-evdev-jp/blob/main/code/xorgxrdp_fix_jp106key_scancode.patch)を適用した[xorgxrdp](https://github.com/neutrinolabs/xorgxrdp) です。<br>
CentOS6/7, RockyLinux8/9 のRPM/SRPM と Debian GNU/Linux 11 の deb で、Xrdp 0.9.20 に対応します。

xorgxrdp は カーソルキー等一部のX11のscancode にズレが生じています。
通常のX11アプリケーションではkeysymの値を使うため実質的な問題は起きませんが、
X11 scancode を直接参照する一部のアプリケーションで想定と異なるキーが押される状態になります。(具体例：カーソルキーの下 ↓ 押下で Enter が入力される)<br>
このパッチを適用したxorgxrdpはX11 scancode を参照する一部のアプリケーションにおいて正しいキー入力を行えるようになります。

以下の４つの条件全てに当てはまる場合にこのパッチ適用済みxorgxrdpが意味を持ちます。

 1. 日本語Windows OS上のリモートデスクトップ接続(mstsc.exe)を使っている
 2. Xrdp をインストールしたLinuxデスクトップに接続している。
 3. XrdpにてバックエンドにVNCではなくXorgを用いている。(`xorgxrdp`を使用している)
 4. Linux デスクトップ 上で X11 scancode を参照するアプリケーションを利用している。 
    -  例）FireFox 等で以下のWebUIに接続する場合
        - VMware vSphere HyperVisor / WebGUI版仮想マシンコンソール
        - HPE Proliant Server に搭載された iLO /  HTML5 コンソール

## 具体的に何をするものか
- xorgxrdp の input driver を base から evdev に変更します。
- 以下のキーが生成するX11 scancode を変更します。

|キー名称|標準<BR>xorgxrdp<BR>X11 Scancode|パッチ適用済み<BR>xorgxrdp<BR>X11 Scancode|備考|
|:------------:|:------------:|:------------:|:------------|
|右 Ctrl     |109|105||
|右 ALT  |113|106||
|左 Windows |115|133||
|右 Windows  |116|134||
|Menu|117|135||
| /  |112|106||
|￥ (yen)|-|132|BackSpace左側の位置「￥」キー<BR>\ (backslash)が入力される|
|\\ (backslash)|-|97|右shiftの左側の位置「ろ」のキー<BR>ブラジル配列ポルトガル語キーボードの[/ ?]と同一 keycode|
|Enter|108|104||
|PrintScreen|111|107||
|Insert|106|118||
|Home|97|110||
|Delete|107|119||
|End|103|115||
|PageUp|99|112||
|PageDown|105|117||
|カーソル上  ↑|98|111||
|カーソル左  ←|100|113||
|カーソル右  →|102|114||
|カーソル下  ↓|104|116||
|ひらがな|-|101||
|変換|-|100||
|無変換|-|102||

## upstreamに PR できない理由
このパッチは日本語環境における特定のアプリケーションの問題を解消することを目的としたWorkaroundパッチで、他言語のことを考慮していないからです。

具体的には[ブラジル配列のポルトガル語キーボード](https://ja.wikipedia.org/wiki/ポルトガル語キー配列#/media/ファイル:KB_Portuguese_Brazil_text.svg)の [/ ?] キーの[既存の定義](https://github.com/neutrinolabs/xorgxrdp/blob/devel/xrdpkeyb/rdpKeyboard.c#L458-L462)が日本語JP106/109キーボードの右shiftkey左側の位置の「\」キー と衝突するため、このパッチでは強引に上書き処理してしまっています。

US/UK英語など、標準的な101/104 ASCII配列のキーボードの場合にはこのパッチはおそらく衝突しませんが、全ての言語・キーボードでのテストを実施しておりません。

(xorgxrdp側で言語を判定して処理を変更するコードを加えるなどして将来的にはupstreamへのPRを上げてみたいと考えてはいます。)

## テスト済み環境
|パッチ有用性|通常のX11アプリ使用|言語|OS|キーボードレイアウト|製品名|備考|
|:----:|:----:|:----:|:----:|:----:|:----:|:----:|
|○|○|日本語|Windows10 Pro 21H2 | JP109|Logicool MX800|右Windowsキー、メニューキーがないため積極的なテストはできていない。|
|○|○|日本語|Windows10 Home 21H2 | US104|CHUWI Hi10pro <BR>専用キーボード|J106/109特有のキーコードは重複しない。テストの範囲では支障なし|
|×|▲|日本語|macOS Monterey| JIS|MacBook Pro 13' 2020(M1)|setxmodmap -queryで layoutがusで認識される。<BR>[￥]キーが効かない|

## 既知の事象
  - JIS日本語キーボードを取り付けたmacOS Monterey で Microsoft RemoteDesktop Client にて接続したとき、 [￥] キーが効かない。
    - ワークアラウンド： [Option] キー+[ ￥] で代用してください。
  - テンキーのNumロックが効かない。常に数字入力になる。

## 参考URL
- [XRDP Keyboard evdev support? #211](https://github.com/neutrinolabs/xorgxrdp/issues/211)
  - [@mhoffmann75](https://github.com/mhoffmann75)による[evdev化パッチ](https://github.com/mhoffmann75/xorgxrdp/commit/16b8fbdc3345a0caa56cb9109790ab6dbe6df892)をベースとしてこれを作成しました。
- [wikipedia キー配列](https://ja.wikipedia.org/wiki/キー配列)
- [wikipedia ポルトガル語キーボード配列](https://ja.wikipedia.org/wiki/ポルトガル語キー配列)
