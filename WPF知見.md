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
