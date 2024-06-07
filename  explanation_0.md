# 課題 0 のサンプルコード

> [!WARNING]
> これ以降は解答となる問題に対するサンプルコードと解説です。最初はこれを見ずに自分なりに頑張ってみてください。

```rust
use std::collections::HashMap;
use std::fmt;

#[derive(Debug)]
struct Person {
    name: String,
    age: u32,
}

impl fmt::Display for Person {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(f, "Name: {}, Age: {}", self.name, self.age)
    }
}

#[derive(Debug, PartialEq, Eq, Hash)]
enum PersonType {
    Student,
    Teacher,
}

struct PersonManager {
    people: HashMap<PersonType, Vec<Person>>,
}

impl PersonManager {
    fn new() -> Self {
        Self {
            people: HashMap::new(),
        }
    }

    fn add_person(&mut self, person_type: PersonType, person: Person) {
        self.people.entry(person_type).or_insert(Vec::new()).push(person);
    }

    fn remove_person(&mut self, person_type: &PersonType, name: &str) -> bool {
        if let Some(people) = self.people.get_mut(person_type) {
            if let Some(pos) = people.iter().position(|p| p.name == name) {
                people.remove(pos);
                return true;
            }
        }
        false
    }

    fn list_people(&self, person_type: &PersonType) {
        if let Some(people) = self.people.get(person_type) {
            for person in people {
                println!("{}", person);
            }
        } else {
            println!("No people found for {:?}", person_type);
        }
    }

    fn list_all_people(&self) {
        for (person_type, people) in &self.people {
            println!("{:?}:", person_type);
            for person in people {
                println!("{}", person);
            }
        }
    }
}

fn main() {
    let mut manager = PersonManager::new();

    manager.add_person(PersonType::Student, Person { name: String::from("Alice"), age: 20 });
    manager.add_person(PersonType::Student, Person { name: String::from("Bob"), age: 22 });
    manager.add_person(PersonType::Teacher, Person { name: String::from("Charlie"), age: 45 });

    println!("Listing all students:");
    manager.list_people(&PersonType::Student);

    println!("\nListing all teachers:");
    manager.list_people(&PersonType::Teacher);

    println!("\nListing all people:");
    manager.list_all_people();

    println!("\nRemoving Alice from students...");
    manager.remove_person(&PersonType::Student, "Alice");

    println!("\nListing all students after removal:");
    manager.list_people(&PersonType::Student);
}
```

## 課題解説

### 構造体と列挙型の定義

1. 構造体 Person：名前と年齢を持つシンプルな構造体です。

```rust
#[derive(Debug)]
struct Person {
    name: String,
    age: u32,
}
```

2. 列挙型 PersonType：Student と Teacher の 2 つのバリアントを持つ列挙型です。

```rust
#[derive(Debug, PartialEq, Eq, Hash)]
enum PersonType {
    Student,
    Teacher,
}
```

3. ハッシュマップの使用

`PersonType`ごとに`Person`を管理するハッシュマップを定義します。ハッシュマップのキーは`PersonType`、値は`Person`のベクタです。

### 関数の実装

1. `add_person`：`PersonType`ごとに`Person`を追加する関数です。

```rust
fn add_person(&mut self, person_type: PersonType, person: Person) {
    self.people.entry(person_type).or_insert(Vec::new()).push(person);
}
```

2. `remove_person`：`PersonType`ごとに`Person`を削除する関数です。

```rust
fn remove_person(&mut self, person_type: &PersonType, name: &str) -> bool {
    if let Some(people) = self.people.get_mut(person_type) {
        if let Some(pos) = people.iter().position(|p| p.name == name) {
            people.remove(pos);
            return true;
        }
    }
    false
}
```

3. `list_people`：指定した`PersonType`の全ての Person を表示する関数です。

```rust
fn list_people(&self, person_type: &PersonType) {
    if let Some(people) = self.people.get(person_type) {
        for person in people {
            println!("{}", person);
        }
    } else {
        println!("No people found for {:?}", person_type);
    }
}
```

4. `list_all_people`：ハッシュマップ内の全ての`Person`を表示する関数です。

```rust
fn list_all_people(&self) {
    for (person_type, people) in &self.people {
        println!("{:?}:", person_type);
        for person in people {
            println!("{}", person);
        }
    }
}
```

この課題を通じて、Rust のコンテナ、構造体、列挙型、およびイテレータの使用方法を実践的に学ぶことができます。

### コラム：なぜ&self.people なのか？

`self.people`は`HashMap<PersonType, Vec<Person>>`型のフィールドです。このフィールドにアクセスする際に、`&self.people`と書く必要があるのは、以下の理由によります。

#### 借用による所有権の問題を回避

Rust の所有権システムは、同時に複数のミュータブル参照を許さず、所有権の移動も制限しています。`self.people`を直接イテレートしようとすると、所有権の移動が発生し、以降の操作で`self.people`を使用できなくなります。

しかし、`&self.people`と書くことで、`self.people`を不変借用（immutable borrow）するだけになり、所有権の移動が発生しません。これにより、以下の利点があります。

1. 安全なアクセス：`self.people`を不変借用することで、他の部分での同時ミュータブル参照が防止され、安全にアクセスできます。
2. 再利用可能：`self.people`の所有権は`self`が保持したままなので、関数の他の部分でも`self.people`を使用できます。

#### イテレータの生成

`&self.people`とすることで、ハッシュマップに対するイテレータが生成されます。このイテレータは、`self.people`を不変借用した状態で生成されるため、ハッシュマップの内容を変更することなく、内容を走査できます。

### コードの具体例と解説

以下に、具体例とその解説を示します。

#### list_all_people 関数

```rust
struct PersonManager {
    people: HashMap<PersonType, Vec<Person>>,
}

impl PersonManager {
    // 他の関数は省略

    fn list_all_people(&self) {
        for (person_type, people) in &self.people {
            println!("{:?}:", person_type);
            for person in people {
                println!("{}", person);
            }
        }
    }
}
```

#### 具体例の動作

1. 不変参照を取得：`&self.people`により、`self.people`の不変参照を取得します。
2. イテレータを生成：この不変参照を使って、ハッシュマップの各エントリに対するイテレータを生成します。
3. 各エントリを走査：生成されたイテレータを使って、ハッシュマップの各エントリ（`person_type`と`people`）を走査します。

## まとめ

`for (person_type, people) in &self.people`とすることで、`self.people`を不変借用し、所有権を維持しつつ、ハッシュマップの各エントリに対するイテレータを生成します。これにより、安全に`self.people`の内容を走査し、他の部分でも`self.people`を使用することが可能になります。Rust の所有権システムと借用のルールを理解することで、このような書き方が必要な理由が明確になります。
