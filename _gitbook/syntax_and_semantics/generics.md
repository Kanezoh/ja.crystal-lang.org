# ジェネリクス

[インスタンス変数と型推論](instance_variables_type_inference.html)で説明したように、インスタンス変数の型は代入された値から推論されます:

```crystal
class MyBox
  def initialize(@value)
  end

  def value
    @value
  end
end
```

例えば、上記のコードに以下を追加したとします。

```crystal
MyBox.new(1)
```

このとき、コンパイラが型推論した結果を `crystal tool hierarchy file.cr` で確認すると以下となります。

```
+- class MyBox
       @value : Int32
```

この箱クラスのインスタンスを、別の型を与えていくつか作成してみます。

```crystal
MyBox.new(nil)
MyBox.new("hello")
MyBox.new(1)
```

すると、結果は以下となります。

```
+- class MyBox
       @value : (Nil | String | Int32)
```

こうなると、この箱を1つの決まった型として扱うことが不可能になります。

```crystal
MyBox.new(1)

box = MyBox.new("hello")
box.value.size # Error: undefined method 'size' for Int32
```

したがって、このような場合には、`@value` がインスタンスごとに固有の型を持つことが望ましいと考えることでしょう。これは特にオブジェクトのコレクションを扱う場合に必要になります。例えば、配列やハッシュが様々な型の要素を含むときに、要素がそれぞれ固有の型を持っていないと非常に扱いづらいものになります。

型変数を使うことで、クラスを汎用的なものにすることができます。例をあげます。

```crystal
class MyBox(T)
  def initialize(@value)
  end

  def value
    @value
  end
end
```

これは以下のようにしてインスタンス化します。

```crystal
MyBox(Int32).new(1)

box = MyBox(String).new("hello")
box.value.size #=> 5
```

これで動くようになりました。理由は、この場合 `MyBox` はもはや単一の型ではなく、複数の同じ種類の型の中である1つの型を示すものとなっているからです。そして、それぞれの型は `T` の型によって識別されます。つまり、`MyBox(Int32)` と `MyBox(String)` は同じ種類ですが別の型として扱われます。そして、それぞれの `@value` が共有されることはありません。`tool hierarchy` コマンドを再度実行してみましょう。その結果は以下となります。

```
+- generic class MyBox(T)
   |
   +- generic class MyBox(String)
   |      @value : String
   |
   +- generic class MyBox(Int32)
          @value : Int32
```

ただ、上記のコードにはわずかな欠点があります。それは、以下が許されてしまうということです。

```crystal
MyBox(Int32).new("hello")
```

これは、`T` の型とインスタンス変数 `@value` の型に関連性がないことが原因です。これは簡単に修正できます。[型制約](type_restrictions.html)を利用すればいいのです。

```crystal
class MyBox(T)
  def initialize(@value : T)
  end

  def value
    @value
  end
end

MyBox(Int32).new(1)       # OK
MyBox(Int32).new("hello") # エラー
```

これで欠点が解消しました。`MyBox(Int32)` としたとき、`T` の型は `Int32` になります。そして、コンストラクタを呼び出した際に、渡される値は `T` の型、つまり `Int32` に合致している必要があるからです。

ただ、それでも `T` と `@value` の間に関連性があるわけではありません。しかし、`MyBox(T)` のインスタンスを作るためには必ず `T` の値を渡す必要があり、それが `@value` の型となるため、実際には、こうしておくことですべての場合に問題なく動作します。

では、次のコードを見てください。

```crystal
class MyBox(T)
  def initialize(@value : T)
  end

  def value=(new_value)
    @value = new_value
  end

  def value
    @value
  end
end

box = MyBox(Int32).new(1) # OK
box.value = "hello"       # OK
```

`value=` メソッドに型制約が設定されていないため、このコードは完全に正当なコードです。ただ、クラスとしては「壊れ」てしまっていると言えます。対策はやはり型制約を利用することです。

```crystal
class MyBox(T)
  def initialize(@value : T)
  end

  def value=(new_value : T)
    @value = new_value
  end

  def value
    @value
  end
end

box = MyBox(Int32).new(1) # OK
box.value = "hello"       # エラー
```

型引数は複数設定することが可能です。

```crystal
class MyDictionary(K, V)
end
```

型引数は1文字の名前のみ設定することができます。

## 型変数の型推論

もし型引数が指定されなかった場合、ジェネリック型のコンストラクタに設定された型制約は自由変数として扱われ、それが型推論に利用されます。例をあげます。

```crystal
MyBox.new(1)       #:: MyBox(Int32)
MyBox.new("hello") #:: MyBox(String)
```

上記では、`MyBox` に型引数を与えていません。このとき、コンパイラは以下の流れで推論を行います。

* `MyBox.new(value)` は処理を `initialize(@value : T)` に委譲する
* `T` が存在しないため、自由変数として扱われる
* `MyBox` は実際は `MyBox(T)` であり、`T` は自由変数であり型引数でもあるため、`T` は渡された値の型になる

このようにして、ジェネリック型の扱いが冗長になってしまうことを軽減しています。

## ジェネリックな構造体とモジュール

構造体とモジュールをジェネリックにすることも可能です。ジェネリックなモジュールは以下のようにインクルードします。

```crystal
module Moo(T)
  def t
    T
  end
end

class Foo(U)
  include Moo(U)

  def initialize(@value : U)
  end
end

foo = Foo.new(1)
foo.t # Int32
```

上記で `T` は `Int32` となります。これは、`Foo.new(1)` によって `U` が `Int32` となり、そして、ジェネリックなモジュールをインクルードすることで `T` が `Int32` となるためです。

## ジェネリック型の継承

ジェネリックなクラスとモジュールを継承することも可能です。継承する際には、具体的な型を指定するか、もしくは型変数を移譲することができます。

```crystal
class Parent(T)
end

class Int32Child < Parent(Int32)
end

class GenericChild(T) < Parent(T)
end
```
