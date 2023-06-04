---
title: "SwiftのSendableプロトコルとは何か"
emoji: "🔀"
type: "tech"
topics: ["Swift", "Concurrency", "Sendable", "Multithreading", "Programming"]
published: true
---

# SwiftのSendableプロトコル

Swiftの`Sendable`プロトコルは、特定の型が同時実行（Concurrency）環境で安全に送信（共有）できることを示すためのものです。`Sendable`プロトコルはデータの安全な送信と同時アクセスを助け、データ競合やレースコンディションといった多スレッドプログラミングの一般的な問題を防ぐ役割があります。

`Sendable`は基本的にマーカープロトコルで、その実装自体には要求されるメソッドやプロパティはありません。型が`Sendable`プロトコルを採用することで、その型のインスタンスが安全に送信（共有）できることを示します。

# Sendableプロトコルの要件

`struct`が`Sendable`になるためには、そのすべてのプロパティが`Sendable`プロトコルを遵守しているか、またはそのすべてのプロパティが不変（`let`）である必要があります。

変更可能な（`var`）プロパティを持つ`struct`も、そのプロパティの型が`Sendable`を遵守していれば、その`struct`全体が`Sendable`になることが可能です。これは、そのプロパティの型が値型（例えば`Int`など）で、各インスタンスが独立していて他のインスタンスに影響を及ぼさないからです。

しかし、参照型（例えばクラス）や`Sendable`を遵守していない他の型の場合、その型のインスタンスが他のインスタンスに影響を及ぼす可能性があるため、そのプロパティは不変（`let`）である必要があります。

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

# Sendableに適合できない例

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

# Sendableプロトコルに遵守していない型をletでもつstructがSendableに遵守可能な例

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

# まとめ

Swiftの`Sendable`プロトコルは、データの安全な送信と同時アクセスを可能にし、データ競合やレースコンディションといった多スレッドプログラミングの一般的な問題を防ぐ重要な役割を果たします。このプロトコルは、型が`Sendable`を遵守していれば、その型のインスタンスを安全に送信（共有）できることを示します。

しかし、全ての型が`Sendable`に適合できるわけではありません。特に参照型や可変プロパティを持つ型は、特別な注意が必要です。その一方で、不変プロパティを持つ型は、そのプロパティが`Sendable`プロトコルに遵守していない場合でも、その型自体が`Sendable`になる可能性があります。

Swiftの`Sendable`プロトコルについての理解は、安全で効率的な並行プログラミングを行うために非常に重要です。