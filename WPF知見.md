WPF知見

WPF/.NET開発で判明した技術的な落とし穴・実装パターン。\
進め方のルールではなく技術知見なので、ガイドライン.mdとは分離する。

# GridSplitterと{Binding}の競合

GridSplitterはドラッグ中、隣接するColumnDefinition/RowDefinitionのWidth/HeightへSetValueで直接書き込む。\
このSetValueは対象プロパティに宣言済みの{Binding}を破壊する(バインドが外れて二度と反映されなくなる)。

対応 : そのプロパティにXAMLで{Binding}を直接宣言しない。\
代わりにビヘイビアがSetCurrentValueで値を反映する。\
SetCurrentValueはバインド/Expressionを保持したまま値だけ更新できる。

# 自己再帰UserControlの構築

UserControlが自分自身の型を子として持つ場合、XAMLに直接`<views:Foo>`と書くとVisibilityに関わらずInitializeComponent時点で無条件に構築され、スタックオーバーフローする。

対応 : 子インスタンスはコードビハインドで生成する。\
Dispatcher.BeginInvoke(DispatcherPriority.Loaded, ...)で1サイクル遅延させ、ContentControl等にアタッチする。

# ColumnDefinition/RowDefinitionのバインド

ColumnDefinition/RowDefinitionはビジュアルツリーに属さない。\
RelativeSource AncestorTypeは効かない(ビジュアルツリーを辿るため)。

ElementNameバインドはXAMLのNameScopeで解決されるため、これらの型でも機能する。

# カスタム添付プロパティの双方向バインド

SetCurrentValueで反映する添付プロパティは、FrameworkPropertyMetadataOptions.BindsTwoWayByDefaultで既定化しておくと、XAML側でMode=TwoWay指定なしでも双方向反映できる。

# .icoを小サイズ表示すると欠けて見える

.icoは複数解像度のフレームを内包する。\
BitmapImageにDecodePixelWidth/Heightを指定しないと、低品質なフレームが選ばれ小サイズ表示時に欠けて見えることがある。

対応 : BitmapImage.DecodePixelWidth/Heightで明示的にデコードサイズを指定する。\
Image側もRenderOptions.BitmapScalingMode=HighQualityにすると縮小表示がより綺麗になる。

# .resxのDesigner.csはdotnet build(CLI)では自動生成されない

Generator=PublicResXFileCodeGeneratorを設定していても、Visual Studioを介さないdotnet buildではDesigner.csの中身(プロパティ一覧)は更新されない。\
.resxにキーを追加しただけでは、XAML側から新しいStrings.Xxxを参照するとビルドエラーになる。

対応 : .resxを編集したら、対応するDesigner.csのプロパティも手動で追記する。

# WinRT型(Windows.*)とWPF型の名前衝突

Windows.Media.Ocr(OCR)などWinRT APIを使う際、Windows.Graphics.ImagingとSystem.Windows.Media.Imagingの両方にBitmapDecoder/BitmapFrameが存在し、両方usingしていると型名があいまいになる。

対応 : 衝突する型だけ`Windows.Graphics.Imaging.BitmapDecoder`のように完全修飾名で書く。\
usingのエイリアス(`using WinRTBitmapDecoder = Windows.Graphics.Imaging.BitmapDecoder;`)でも回避できる。

# Background未指定のPanelは空領域がヒットテスト対象外

Grid等のPanelはBackground未指定(null)だと、子要素の無い空領域が
ヒットテスト対象外になる。\
その領域をクリックしても、そのPanel自身は元よりPreviewMouseDown等の
イベントすら発生しない(子要素越しでも祖先越しでも拾えない)。

対応 : クリックを拾いたいPanelに`Background="Transparent"`を指定する。\
見た目は変わらずヒットテスト対象になる。
