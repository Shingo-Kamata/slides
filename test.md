# Welmo 勉強会

## 関数型言語とは

- 特定の機能をさすわけではない
- 関数とは，直積のある条件を満たす部分集合でしかない
- Pythonも関数型言語といいはれることもできる
    - Cf. https://www.slideshare.net/ksknac/120901fp-key

### python  で関数は第一級オブジェクト

-  関数を変数に代入することができる
    ```python
    f = lambda x: x+1
    >>> f(1)
    2
    ```
- 関数を関数に渡したり，関数を返り値にもつ関数をつくったりできる
    ```python
    f = lambda x: x(3)
    ```
   ``` python 
   f = lambda x: x(x)
   f(f)
   ``` 
   ``` python 
   f = lambda g,y,z: g(y,z)
   add = lambda x,y: x+y
   >>> f(add,3,4)
   7
   ``` 

### 文と式
- if文とif式
    - 文は**副作用**をもつ
    - 式は関数のように扱え、評価が成功する
- lambda 式の中で使えるのは、**式** のみ
    - if式は使えるが、if分は使えない
    - let(in)式がないので、lambdaの中で、変数に代入（束縛が正しいかな）して再利用ができない
        - 例 `lambda x: let v = x[1] in (len(v),v)` とかできない

### 関数を第一級オブジェクトとして扱うメリット
以下のように、**高階な関数（関数を引数、返り値にできる関数）** を使って、便利なことができる

- 引数に関数を渡せる：ソートの例
  ``` python
  a = (3, "りんご",'a')
  b = (0, "みかん", 'j')
  c = (2, "CPA", 'z')
  l = [a, b, c]
  sorted(l, key=lambda x:x[0])
  ```
- 返り値(return)に関数を指定できる：関数の拡張(a -> b) -> ([a] ->[b]) の例
    ```python
    def F(f):
      return lambda l: [f(x) for x in l]
    >>> F(str)([1,2,3])
    ['1', '2', '3']
    ```
- 同じようなことは，実はオブジェクト指向を使ってもできなくはない（クラスメソッドのオーバーライドなどで），関数型的記述のほうが，より軽量に実現できる事が多い

 
## 高階関数 `reduce`、`map`、`filter`
### for文をなくしたい
主に，以下の可読性向上とバグの温床を消すため

1. 行数が長くなりがち
1. forのネストの中で何をしているか忘れてしまう
1. pythonのようなオフサイドルールでは，returnの位置がネストされたfor文に入ってバグの温床になりかねない
    ``` python
    def sum_of_sum(ll):
        s = 0
        for l in ll:
            for x in l:
                s += x
            return s
    # >>> sum_of_sum([[2],[1,2,3]])
    # 8
    def sum_of_sum(ll):
    s = 0
    for l in ll:
        for x in l:
            s += x
        return s
    # >>> sum_of_sum([[2],[1,2,3]])
    # 2
    ```
1. for文は，汎用的すぎて，「何がしたい」のかわかりにくい


### forループでしたいこと
- 畳み込み
    - sum や len など
    ```python
    sum = 0
    for x in range(n):
        sum + = x
    ```
- 変換
    - str や int など
    ```python
    l_result=[]
    for x in l:
        l_result.append(x*x)
    ```

- 間引き
    ```python
    l_result=[]
    for x in l:
        if x > 5:
            l_result.append(x)
    
forループでしていること，したいことは，実は上記あるいはその組み合わせでしかない．それぞれを実現するための、**高階関数** が用意されている

### reduce
``` python
>>> from functools import reduce
>>> from operator import add
>>> reduce(add,[1,2,3,4,5])
15
>>> add(1,2)
3
```

### map
``` python
>>> list(map(lambda x: x+1, [1,2,3]))
[2, 3, 4]
```
- `list()`: mapの中身を評価する（Lazyをやめる）
- mapやfilterは **Lazy** になっている（難しい）
    - `Python3.X` から？
- 下記は、bを評価するときに実際に計算したほうが高速
    - リストを舐める回数が１回で済む。
    - そのかわり、適応する関数を `f(g(x)) = *(+(x,3),2)` としている
    ``` python
    >>> a = map(lambda x:x+3,[1,2,3])
    >>> a
    <map object at 0x10779fcf8>
    >>> b = map(lambda x:x*2,a)
    >>> b
    <map object at 0x10779fe80>
    >>> list(b)
    [8, 10, 12]
    ```
- 便利な使い方
    ```python
    >>> list(map(print,["data1","data2","data3"]))
    data1
    data2
    data3
    [None, None, None]
    >>> list(map(lambda x: print("Ans:"+x),["data1","data2","data3"]))
    Ans:data1
    Ans:data2
    Ans:data3
    [None, None, None]
    ```
- `None` とは？
    - `a = print("test")`をしてみるとわかる？ 

### filter
```python
>>> list(filter(lambda n:n%2==1, [1,2,3,4]))
>>> [1, 3]
```

### forとの対応

|作業|例|利用すべき高階関数|
|:--:|:-:|:-------------:|
|畳み込み|sum|reduce
|変換|str|map|
|間引き|?|filter

### 内包表記
- 内包表記のほうがいろいろ便利だという話も。。。
    - `map(f,l) = [f(x) for x in l]`

## 演習問題

- 畳み込み
    - リストのリストが渡されたとき，それをリストにする関数を作成せよ
    - e.g `flatten1 ([[1,2,3],[2,3],[],[9]]) = [1,2,3,2,3,9]`
- 変換
    - 文字列のリストが渡されたときに、その文字列とその長さのペアを返す関数を作成せよ
    - e.g. `str_pair(["test","hogehoge",""]) = [("test",4),("hogehoge",8),("",0)]`

- 間引き
    - 数のリストが渡されたときに、絶対値が10より小さい値を間引く関数を作成せよ
    - e.g. `filter_abs_under_10([20,3,-100,-4,0])=[20,-100]`

- 組み合わせ
    - 数を表す文字列のリストが渡されたときに、それを数に変換して、５より小さい数を間引く関数を作成せよ
    - e.g. `complex(["12","3","5","14"]) = [12,14]`

