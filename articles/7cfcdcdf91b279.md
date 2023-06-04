---
title: "SwiftのSendableプロトコルとは何か"
emoji: "🔀"
type: "tech"
topics: ["Swift", "Concurrency", "Sendable", "Multithreading", "Programming"]
published: true
---

# SwiftのSendableプロトコル

Swiftの`Sendable`プロトコルは、特定の型が同時実行（Concurrency）環境で安全に送信（共有）できることを示すためのものです。この「送信」という概念には、データの所有権の移転が含まれています。所有権の移転により、データが一度「送信」されると、受け取った側がそのデータを独占的に利用できることを意味します。これにより、同時に複数の場所からのアクセスや変更が防がれます。`Sendable`プロトコルはデータの安全な送信と同時アクセスを助け、データ競合やレースコンディションといった多スレッドプログラミングの一般的な問題を防ぐ役割があります。

`Sendable`は基本的にマーカープロトコルで、その実装自体には要求されるメソッドやプロパティはありません。型が`Sendable`プロトコルを採用することで、その型のインスタンスが安全に送信（共有）できることを示します。

# Sendableプロトコルの要件

`struct`が`Sendable`になるためには、そのすべてのプロパティが`Sendable`プロトコルを遵守しているか、またはそのすべてのプロパティが不変（`let`）である必要があります。

変更可能な（`var`）プロパティの型が`Sendable`を遵守していれば、その`struct`も`Sendable`になります。これは、そのプロパティの型が値型（例えば`Int`など）の場合、1つのインスタンスを変更しても、それが別のインスタンスに変更を引き起こすことがないためです。つまり、各インスタンスは互いに独立しており、お互いに影響を及ぼし合わない性質を持っています。

しかし、参照型（例えばクラス）や`Sendable`を遵守していない他の型の場合、その型のインスタンスが他のインスタンスに影響を及ぼす可能性があるため、そのプロパティは不変（`let`）である必要があります。

## Sendableとスレッドセーフ

`Sendable`は、あるデータを安全に異なるスレッドやアクターに送信できることを示すものです。しかし、`Sendable`に準拠しているだけでは、そのデータのすべての操作がスレッドセーフであるとは言えません。例えば、あるスレッドでデータを読み取る途中に、別のスレッドがそのデータを変更すると、不具合や予期しない動作が発生する可能性があります。このような競合を避けるためには、排他制御の仕組みを利用するなど、別の手段を取る必要があります。

# Sendableプロトコルの使用例

以下は`Sendable`プロトコルを使用した例です。

````swift
struct Point: Sendable {
    let x: Int
    let y: Int
}
````

また、可変のプロパティを持つ`struct`の例を示します。

````swift
struct Person: Sendable {
    var name: String
    let age: Int
}
````

## Sendableに適合できない例

しかし、全ての型が`Sendable`に適合できるわけではありません。参照型（クラス）はその性質上、同じインスタンスへの複数の参照を持つことが可能で、そのプロパティが可変（`var`）である場合、そのクラスは`Sendable`プロトコルに適合できません。これは、異なるスレッドから同時にそのプロパティにアクセス・変更が行われた場合、データ競合やレースコンディションを引き起こす可能性があるためです。

以下に、`Sendable`プロトコルに適合できないクラスの例を示します。

````swift
class NonSendableClass {
    var mutableState: Int = 0
}
````

この例では、`NonSendableClass`は`mutableState`という可変のプロパティを持っています。そのため、このクラスは`Sendable`プロトコルに適合できません。

また、自分で定義した型が`Sendable`プロトコルを遵守していない場合、その型を持つ構造体も`Sendable`になることはできません。

````swift
struct NonSendableType {
    var state: Int = 0
}

struct SomeStruct {
    var nonSendable: NonSendableType
}
````

この例では、`NonSendableType`は`Sendable`プロトコルを遵守していません。そのため、この型をプロパティとして持つ`SomeStruct`も`Sendable`プロトコルに適合できません。

## Sendableプロトコルに遵守していない型をletでもつstructがSendableに遵守可能な例

しかし、型が`Sendable`プロトコルを遵守していない場合でも、その型のプロパティが不変(`let`)であれば、その構造体は`Sendable`プロトコルに遵守することが可能です。以下にその例を示します。

````swift
struct NonSendableType {
    var state: Int = 0
}

struct SomeStruct: Sendable {
    let nonSendable: NonSendableType
}
````

この例では、`NonSendableType`は`Sendable`プロトコルを遵守していません。しかし、`SomeStruct`は`nonSendable`プロパティを不変(`let`)として定義しているため、`SomeStruct`は`Sendable`プロトコルに遵守することが可能です。

この不変性が重要な理由は、不変プロパティは作成後に変更することができないため、同時実行（Concurrency）環境においてもデータ競合やレースコンディションを心配する必要がないからです。

## `Sendable`に遵守可能なクラスの例

Swiftの`Sendable`プロトコルをクラスが遵守するための要件は以下の通りです:

1. クラスは`final`である必要があります。
2. 保存プロパティはすべて不変で、かつ`Sendable`である必要があります。
3. スーパークラスがない、またはスーパークラスが`NSObject`である必要があります。

これらの条件を満たしたクラスの例を以下に示します:

```swift
final class SendableClass: NSObject, Sendable {
    let intValue: Int
    let stringValue: String

    init(intValue: Int, stringValue: String) {
        self.intValue = intValue
        self.stringValue = stringValue
    }
}
```

このクラス`SendableClass`は、不変のプロパティ`intValue`と`stringValue`を持っており、これらのプロパティはどちらも`Sendable`プロトコルに遵守しています（`Int`と`String`は共に`Sendable`）。また、このクラスは`final`であり、`NSObject`を継承しています。これらの要件を満たすため、このクラスは`Sendable`プロトコルに遵守していると言えます。

### なぜNSObjectがスーパークラスとして許可されているのか？

`Swift`の`Sendable`プロトコルに適合するクラスの要件について述べると、そのクラスは「スーパークラスがないか、スーパークラスが`NSObject`である」と指定されています。これは`Swift`の同時実行（concurrency）モデルが、クラスの継承と同時実行の複雑さを扱うことを防ぐための措置です。

`NSObject`は`Objective-C`の根幹をなすクラスで、`Objective-C`の多くの機能と特性を`Swift`に提供しています。`NSObject`は基本的にスレッドセーフではありませんが、`Sendable`としてマークされるためにはそのすべてのプロパティが不変でなければならず、これにより同時アクセスによる競合状態が発生する可能性が大幅に低減されます。

しかし、注意すべき点として、`NSObject`を継承したクラスを`Sendable`として扱う場合でも、そのクラスが必要とするすべての操作がスレッドセーフであることを確認する必要があります。さらに、独自のスーパークラスを持つクラスが`Sendable`に適合するためには、そのスーパークラスが`Sendable`の要件を満たすように設計されていること、つまり、すべての公開された状態が不変であること、を保証しなければなりません。

これらを考慮に入れると、`Swift`の`Sendable`プロトコルは、データの安全性と一貫性を維持しつつ、マルチスレッドや非同期処理に対応するための強力なツールと言えます。コードの中で安全にデータを送信する際には、このプロトコルがどのように動作するかを理解することが非常に重要です。

## まとめ

Swiftの`Sendable`プロトコルは、マルチスレッド環境や非同期処理でデータを安全に送信するための重要なツールです。このプロトコルは、データの一貫性と安全性を確保しながら、同時実行の問題を解決する手段を提供しています。

この記事では、`Sendable`プロトコルの基本的な概念と、それがどのように型安全性を提供するかを紹介しました。また、どのようにして自分の型を`Sendable`にするか、そのためにどのような条件を満たす必要があるかについても説明しました。

Swiftの`Sendable`プロトコルについての理解は、安全で効率的な並行プログラミングを行うために非常に重要です。