Kotlin Grammer
==

* `val`で変数を定義。immutable。基本的にはこれを使う。`var`は変更する場合のみ
* `List`は中身の変更が不可、`mutableList`は変更可能。同様に`mutableMap`もある
* 参照先が同じかどうか`===`、構造の等価性(equalsメソッドによる同一性チェック)は`==` 
* if文で値を返すこともできる
* whenはswitch文
* `MutableList`から`List`にアップキャストできる
* 関数はパッケージのトップレベルで宣言できるのでクラスメソッドにする必要はない
* 関数の引数にデフォルト値を指定できる
* `vararg`で引数を可変長にできる
* コンストラクタを定義しないと自動で引数なしコンストラクタが生成される
* 1つのプライマリコンストラクタと複数のセカンダリコンストラクタが使える
* インスタンス生成に`new`は不要
* すべてのクラスは`Any`を継承する
* setter, getterの宣言は不要
* データクラスは、データを保持したいだけのクラスとして使われる。class宣言の最初に`data`をつける
* コンパニオンオブジェクトはstaticのメンバ変数を定義する
* 型チェックに`is`と`!is`が使える。チェックした後自動的にキャストされる
* `let`は、引数として関数を受け取り、元の関数がnullの場合はnullを返す。JavaのOptionalに似ている。
* Null安全。基本的にnon null typeを使う。必要な時だけ null type(`?`をつける)
    * 普通の型に`?`をつけるとnullの値を持つことができる、NPEを発生しにくくする
* 返り値でなにも返さない場合はUnit, 省略化
* `use`はJavaの`try-with-resources`
* `buildString`はJavaの`StringBuilder`