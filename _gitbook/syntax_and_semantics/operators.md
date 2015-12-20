# 演算子

`+` や `-` などの演算子は、実は普通のメソッド呼び出しとなっています。例をあげます。

```crystal
a + b
```

これは以下と同じです。

```crystal
a.+(b)
```

ある型に演算子を定義したい場合は以下のようにします。

```crystal
struct Vector2
  getter x, y

  def initialize(@x, @y)
  end

  def +(other)
    Vector2.new(x + other.x, y + other.y)
  end
end

v1 = Vector2.new(1, 2)
v2 = Vector2.new(3, 4)
v1 + v2               #=> Vector2(@x=4, @y=6)
```

これから、すべての演算子を、その一般的な用途とあわせて紹介します。

## 単項演算子

```crystal
+   # 正数
-   # 負数
!   # 否定
~   # ビットの補数
```

これらは引数を持たないものとして定義されています。例をあげます。

```crystal
struct Vector2
  def -
    Vector2.new(-x, -y)
  end
end

v1 = Vector2.new(1, 2)
-v1                    #=> Vector2(@x=-1, @y=-2)
```

## 2項演算子

```crystal
+   # 加算
-   # 減算
*   # 乗算
/   # 除算
%   # 剰余
&   # ビット AND
|   # ビット OR
^   # ビット XOR
**  # べき乗
<<  # 左シフト/追加
>>  # 右シフト
==  # 等しい
!=  # 等しくない
<   # 未満 (〜より小さい)
<=  # 以下
>   # 超 (〜より大きい)
>=  # 以上
<=> # comparison
=== # case equality
```

## インデックス

```crystal
[]  # 配列のインデックス (配列長を超えると例外が発生)
[]? # 配列のインデックス (配列長を超えると nil)
[]= # 配列のインデックス指定代入
```

例をあげます。

```crystal
class MyArray
  def [](index)
    # ...
  end

  def [](index1, index2, index3)
    # ...
  end

  def []=(index, value)
    # ...
  end
end

array = MyArray.new

array[1]       # 1つ目のメソッドを実行
array[1, 2, 3] # 2つ目のメソッドを実行
array[1] = 2   # 3つ目のメソッドを実行

array.[](1)       # 1つ目のメソッドを実行
array.[](1, 2, 3) # 2つ目のメソッドを実行
array.[]=(1, 2)   # 3つ目のメソッドを実行
```

## 演算子の意味について

実際には、演算子に対して、どのような内容の処理であっても自由に定義することが可能です。しかし、上記した演算子それぞれの意味にしたがって定義することが慣習となっています。これは、複雑で読みづらいコードや、コードが想定外の動作をしてしまうことを避ける意味で重要です。

