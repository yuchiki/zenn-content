---
title: "C#10 preview 版ではインターフェースを用いて表現できる型クラス的な制約が広がった"
emoji: "👏"
type: "tech"
topics: ["dotnet", "csharp", "型クラス", "インターフェース", "preview" ,]
published: false
---

# Todo

- [x] タイトルが強すぎやしないか
  - 普通の C# 10 でも表現できないか？
  - 弱めたのでOK


# 概要

C#10/.NET 6 の preview版では interface が static abstract member を持つことができます。この機能により、複雑でない型に対して型クラス的な制約が書けるようになりました。
以下の順で解説していきます。

1. ここでいう型クラス的とは何を言っているのかについて説明
2. C# 10 preview の static abstracts in interface 機能を紹介
3. 型クラスの表現についての実例を提示
4. C# 10 preview 版では表現できないような型-型クラス制約の例を確認
5. まとめ

今回のコードをまとめたサンプルレポジトリは[こちら](https://github.com/yuchiki/csharp-10-preview-typeclass)です。

# ここでいう型クラス的とは？

一般に型クラスとは型が備えているべき機能についての制約を書くための言語機能です。著名な言語だと Haskell などに導入されています。インターフェースと用法が一部被るところもあれば、そうでない部分もあります。
C#のインターフェースの使い方のういち、型クラスの使い方と重なる部分がどのようなものかを説明するために、まずは C# のインターフェース と Haskell の型クラスについて確認します。

## C# のインターフェース機能についての確認

いま、長さとみなしうる性質を備えている Personレコード型とDeskレコード型があり、どちらも IMeasurable インターフェースを継承しているとします。

```cs
/// 長さを計れるもののインターフェース
interface IMeasurable {
    int Measure();
}

record Person(string Name, int Height) : IMeasurable{
     public int Measure() => Height;
}

record Desk(string Material, int Size) : IMeasurable{
    public int Measure() => Size;
}
```
例えば、インターフェースを用いると「「長さ」を性質として備えている二つのものを比較して、長い方を返す関数」を書くことができます。
以下は、IMeasurable 型の値を２つとって、長い方の値を返す GetLongerOne関数の例です。

```cs
IMeasurable GetLongerOne(IMeasurable value1, IMeasurable value2) =>
    value1.Measure() >= value2.Measure() ? value1 : value2;

Person person1 = new Person("Tom", 168);
Desk desk1 = new Desk("Wood", 90);
IMeasurable longerOne = Longer(person1, desk1);

System.Console.WriteLine(longerOne); // Person { Name = Tom, Height = 168}
```


では、「「長さ」を性質として備えている二つの**同種**のものを比較して、長い方を返す関数」はどうでしょうか？一見上の関数をそのまま使っても問題ないように思えます。しかし、返り値を呼び出し元で使用する際に、IMeasurable インターフェースを備える型に共通の性質ではなく、比較される値の型に特有のメソッドを使用したい場合に困ります。

```cs
IMeasurable GetLongerOne(IMeasurable value1, IMeasurable value2) =>
    value1.Measure() >= value2.Measure() ? value1 : value2;

Person person1 = new Person("Tom", 168);
Person person2 = new Person("Bob", 172);
IMeasurable longerOne = Longer(person1, person2);

// IMeasurable インターフェースは Name を備えていないので以下の行が書けない
System.Console.WriteLine($"{longerOne.Name} is the taller person.");
```

このようなときに、C#ではジェネリック型制約という言語機能を使うことができます。
ジェネリック型制約とは、「この制約を満たす任意の型について～～～」という制約を書ける機能です。
この機能を用いて、 GetLongerOne関数を、「IMeasurable インターフェースを満たす任意の型Tについて、 T型の値を２つ受け取って、長い方のT型の値を返す関数」に書き換えます。

```cs
T GetLongerOne<T>(T value1, T value2)
    where T: IMeasurable
    => value1.Measure() >= value2.Measure() ? value1 : value2;

Person person1 = new Person("Tom", 168);
Person person2 = new Person("Bob", 172);
Person longerOne = GetLongerOne(person1, person2);

System.Console.WriteLine($"{longerOne.Name} is the taller person.");
```


## Haskell の 型クラス

上で紹介した C# コードのうち、後者のケースを Haskell の型クラス機能を用いて実装してみます。

まず、「その型の値に対してmeasureという操作ができなければならない」という制約を、型クラスを用いて書きます。

```haskell
class Measurable a where
    measure :: a -> Integer
```

「measure ができる任意の型について、その型の値を２つ受け取り measure 結果が大きい方の値を返す」関数 getLongerOne は以下のように書けます。

```haskell
getLongerOne :: Measurable a => a -> a -> a
getLongerOne x y
    | measure x >= measure y = x
    | otherwise              = y
```

次に、 Measurable の制約を満たす Person 型を以下のように定義します。

上の制約を満たす Person型を下のように定義します。

```haskell
data Person = Person { name :: String, height :: Integer }
instance Measurable Person where
    measure = height
```

下のように、getLongerOne 関数は Person 型の値を受け付けることができます。

```haskell
-- "Bob is the taller person." と表示される。
main = putStrLn $ name longerOne ++ " is the taller person."
    where
        person1 = Person "Tom" 168
        person2 = Person "Bob" 172
        longerOne = getLongerOne person1 person2
```

## C# のインターフェースと Haskell の型クラスの比較

上で示した例を見ると、C# においてジェネリクス制約を用いて書いた方のコード例と、 Haskell において型クラスを用いて書いたコード例の類似性に気づいてもらえるかと思います。
構文の類似性もさることながら、このケースにおいてはどちらも同様の制約に基づき getLongerOne 関数を実装することに成功しています。
インターフェース機能と型クラス機能には重ならない部分も大きいのですが、少なくともHaskell において型クラスを用いてできることの一部は C# のインターフェースでも実現できることはご理解いただけたかと思います。



# C# 10 preview の static abstracts in interface 機能

[static abstracts in interfaces 機能](https://devblogs.microsoft.com/dotnet/preview-features-in-net-6-generic-math/#static-abstracts-in-interfaces) は、interfaceがstatic abstractsなメンバーを持つことができる機能です。

C# においては、 static なメンバーとはインスタンスではなくクラスに紐づくメンバーのことを指し、abstract なメンバーとは、実体を持たず、継承したクラスで実装されなければならないメンバーのことを指します。static abstract を指定することで、継承先のクラスがクラス自身に紐づく特定のメソッドを備えていることを強制することができます。
下のコードは、IntroduceTypeItselfというstaic メソッドを要求するISelfIntroducerインターフェースの例です。

```cs
public interface ISelfIntroducer
{
    public abstract static string IntroduceTypeItself();
}

public class HappyClass : ISelfIntroducer
{
    pub static string IntroduceTypeItself() => "I'm a happy lucky class!";
}
```

ISelfIntroducer インターフェースを満たす型一般に関して、以下のようにIntroduceTypeItselfメソッドを呼び出す処理を書くことができます。

```cs
string AskTheirIdentity<T>()
    where T : ISelfIntroducer
{
    return T.IntroduceTypeItself();
}

var expected = "I'm a happy lucky class!";
Assert.Equal(AskTheirIdentity<HappyClass>(), expected);
```




# 型クラスの表現の実例
## 以前から書けていた型クラス的制約の例


## 今回書けるようになった型クラス的制約の例


## 例: Parseable
## 例: Monoid

# C# 10 preview 版では表現できない型クラス的制約の例

## 例: Monad

# まとめ
