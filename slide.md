### 表参道.rb #52
- - -

### Ruby 2.7.0-preview2 の
### リリースノートを読もう

---

#### 自己紹介
- - -

* なまえ  : おしょー
* Twitter : [@pink_bangbi](https://twitter.com/pink_bangbi)
* github  : [osyo-manga](https://github.com/osyo-manga)
* ブログ  : [Secret Garden(Instrumental)](http://secret-garden.hatenablog.com)
* Rails 歴 1年半
* 趣味で Ruby にパッチを投げたりしてます
* 最近のトレンド
  * `binding_of_caller`
  * `solargraph`
  * `vim`

---

## 今日話すこと

---

### Ruby 2.7.0-preview2 の
### リリースノートを読もう

---

### [Ruby 2.7.0-preview2 リリースノート](https://www.ruby-lang.org/ja/news/2019/10/22/ruby-2-7-0-preview2-released/)

---

### ここからはリリースノートに
### 載ってない便利機能

---

#### [1行 in 演算子](https://bugs.ruby-lang.org/issues/15865)
- - -

* パターンマッチの in を1行でかける
* `case <expr>; in <pattern>; ... end`

```ruby
user = { name: "Mami", age: 15 }
if user in { name: name, age: (..20) }
  p name # => "Mami"
end

users = [
    { id: 1, name: "Homu", age: 13 },
    { id: 2, name: "Mami", age: 21 },
    { id: 3, name: "Mado", age: 14 },
]
p users.select { _1 in { name: /m/, age: (..14) } }
# => [{:id=>1, :name=>"Homu", :age=>13}]

# 右辺への代入…？🤔
(1..).take(10).select(&:even?).map { _1 + _1 } in result
p result # => [4, 8, 12, 16, 20]
```

---

#### [Enumerable#filter_map](https://bugs.ruby-lang.org/issues/15323)
- - -

* filter + map を行うメソッド
* ブロックの戻り値が `nil` または `false` である要素を取り除き、戻り値に変換した配列を返す

```ruby
# Ruby 2.6
pp (1..10).filter(&:even?).map { |it| it + it }
# => [4, 8, 12, 16, 20]


# Ruby 2.7
pp (1..10).filter_map { _1 + _1 if _1.even? }
# => [4, 8, 12, 16, 20]
```

---

#### `.` の間にコメントがかけるようになった
- - -

* メソッドチェーン等で行頭に `.` がある場合でもコメントがかけるようになった

```ruby
(1..)
  # ここに
  .take(10)
  # コメントが
  .select(&:even?)
  # かけるようになった！！
  .map { _1 + _1 }
```

---

#### [Module#const_source_location](https://bugs.ruby-lang.org/issues/10771)
- - -

* レシーバに対して引数の定数定義位置を返す

```ruby
module M
  VALUE = 42
  class X
    VALUE = 42
  end
end

p M.const_source_location(:VALUE)
# => ["/home/test.rb", 2]

p M.const_source_location(:X)
# => ["/home/test.rb", 3]

p M::X.const_source_location(:VALUE)
# => ["/home/test.rb", 4]

p Object.const_source_location(:M)
# => ["/home/test.rb", 1]
```

---

#### [Time#inspect がミリ秒を含むようになった](https://bugs.ruby-lang.org/issues/15958)
- - -

```ruby
time = Time.utc(2010,3,30, 5,43,"25.123456789".to_r)

# Ruby 2.6
puts time.to_s    # => 2010-03-30 05:43:25 UTC
puts time.inspect # => 2010-03-30 05:43:25 UTC
puts time         # => 2010-03-30 05:43:25 UTC
p time            # => 2010-03-30 05:43:25 UTC

# Ruby 2.7
puts time.to_s    # => 2010-03-30 05:43:25 UTC
puts time.inspect # => 2010-03-30 05:43:25.123456789 UTC
puts time         # => 2010-03-30 05:43:25 UTC
p time            # => 2010-03-30 05:43:25.123456789 UTC
```

---

#### [Time#floor](https://bugs.ruby-lang.org/issues/15653) / [Time#ceil](https://bugs.ruby-lang.org/issues/15772)
- - -

* ミリ秒を切り捨てる #floor の追加
* ミリ秒を切り上げる #ceil の追加

```ruby
require "time"
time = Time.utc(2010,3,30, 5,43,"25.123456789".to_r)


# 切り捨て
p time.floor(3) # => "2010-03-30 05:43:25.123 UTC"
p time.floor(7) # => "2010-03-30 05:43:25.1234567 UTC"


# 切り上げ
p time.ceil(3)  # => "2010-03-30 05:43:25.124 UTC"
p time.ceil(7)  # => "2010-03-30 05:43:25.1234568 UTC"
```


---

#### [引数の譲渡](https://bugs.ruby-lang.org/issues/16253)
- - -

* `def hoge(...)` で引数を受け取ると `foo(...)` のようなメソッドを呼びだしで引数を譲渡できる

```ruby
def foo(a, b, c, &block)
  [a, b, c, block.call(a)]
end

def hoge(...)
  # 引数を foo に譲渡
  foo(...)
end

p hoge(42, { a: 1 }, "bar"){ _1 + _1 }
# => [42, {:a=>1}, "bar", 84]
```

---

#### [UnboundMethod#bind_call が追加](https://bugs.ruby-lang.org/issues/15955)
- - -

* `bind` + `call` を行うメソッドが追加

```ruby
class Foo
  def add_1(x)
    x + 1
  end
end
class Bar < Foo
  def add_1(x) # override
    x + 2
  end
end

obj = Bar.new
p obj.add_1(1) #=> 3
p Foo.instance_method(:add_1).bind(obj).call(1) #=> 2
p Foo.instance_method(:add_1).bind_call(obj, 1) #=> 2
```

---

#### [methodとinstance_methodがRefinementで有効](https://bugs.ruby-lang.org/issues/15373)
- - -

```ruby
module Ex
  refine Numeric do
    def plus a
      self + a
    end
  end
end
using Ex

p 42.method(:plus).call(2)
# => 44
p Numeric.instance_method(:plus).bind_call(3.14, 5)
# => 8.14
```


---

# 宣伝

---

#### [Ruby Hack Challenge Holiday](https://rhc.connpass.com/event/151557/)
- - -

* Ruby 本体を自分でビルドしたりハックしたりするイベント           <!-- .element: class="fragment" -->
* Ruby のコミッタの方が主催しているので Ruby について聞けるチャンス       <!-- .element: class="fragment" -->
* steep を開発して @soutaro さんをお呼びして「Ruby の型をどう書くのか」「実際に組込クラスの型をつけて貢献する方法」をレクチャーがある        <!-- .element: class="fragment" -->
* 次回は 11/10(日) 開催予定          <!-- .element: class="fragment" -->

---

## ご清聴
## ありがとうございました
