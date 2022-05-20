---
title: "C#10 preview版では表現できる型クラス的制約が広がった"
emoji: "👏"
type: "tech"
topics: ["dotnet", "csharp", "型クラス", "preview" , "staticAbstract"]
published: false
---

# Todo

- [x] タイトルが強すぎやしないか
  - 普通の C# 10 でも表現できないか？
  - 弱めたのでOK


# 概要

C#10/.NET 6 の preview版では interface が static abstract member を持つことができます。この機能により、複雑でない型に対して型クラス的な制約が書けるようになりました。
以下の順で解説していきます。

1. 型クラスとは何かについて説明
2. C# 10 preview の static abstracts in interface 機能を紹介
3. 型クラスの表現についての実例を提示
4. C# 10 preview 版では表現できないような型-型クラス制約の例を確認
5. まとめ

今回のコードをまとめたサンプルレポジトリは[こちら](https://github.com/yuchiki/csharp-10-preview-typeclass)です。

# 型クラスとは？

# C# 10 preview の static abstracts in interface 機能

[static abstracts in interfaces 機能](https://devblogs.microsoft.com/dotnet/preview-features-in-net-6-generic-math/#static-abstracts-in-interfaces) は、interfaceがstatic abstractsなメンバーを持つことができる機能です。

C# においては、 static なメンバーとはインスタンスではなくクラスに紐づくメンバーのことを指し、abstract なメンバーとは、実体を持たず、継承したクラスで実装されなければならないメンバーのことを指します。static abstract を指定することで、継承先のクラスがクラス自身に紐づく特定のメソッドを備えていることを強制することができます。

```cs
public interface ISelfIntroducer
{
    public abstract static string IntroduceTypeItself();
}


public class HappyClass : ISelfIntroducer
{
    public static string IntroduceTypeItself() => "I'm a happy lucky class!";
}

```




# 型クラスの表現の実例

## 以前から書けていた型クラス的制約の例


## 今回書けるようになった型クラス的制約の例


## 例: Parseable
## 例: Monoid

# C# 10 preview 版では表現できない型クラス的制約の例

## 例: Monad

# まとめ
