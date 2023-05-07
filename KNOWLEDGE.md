## よく使用するrailsコマンド
- `generate(g)`: テンプレート機能を使用してプログラムのひな型を生成する
- `console(c)`: Railsアプリケーションの環境を読み込んだうえでirbを起動する
- `server(s)`: 開発用アプリケーションサーバを起動する
- `test(t)`: アプリケーションのテストを実行する
- `test:system`: アプリケーションのシステムテストを実行する
- `dbconsole(db)`: 設定ファイルに定義されている接続情報を使用してDBへ接続し、コンソールを起動する
- `new`: Railsアプリケーションを作成する
- `db:create`: データベースを作成する
- `db:drop`: データベースを削除する
- `db:migrate`: マイグレーションファイルの内容のテーブル定義をDBに反映させる
- `db:seed`: `db/seed.rb`の内容を実行する
- `db:setup`: データベースを作成し、スキーマの読み込みとシードデータの読み込みを行う
- `routes`: URLとControllerを紐づけているルーティング情報を出力する

---

## ActivateRecordによるモデルの役割
1. データベースと接続し、データベースのレコードとActiveRecordオブジェクトを結びつける役割
2. ビジネスロックの実装的な振る舞いに関する、バリデーションやレコード保存などに実行する様々なコールバックなどを実行する役割
---
# モデリング基礎知識
## CRUD操作との対応表
| 動詞 | 具体的な内容 | ActiveRecordの対応するメソッド |
| ---- | ---- | ---- |
| Create | レコードを作る | Book.create, Book#save |
| Read | レコードを参照する | Book.find, Book.all |
| Update | レコードを更新する | Book#save, Book#update_attributes |
| Delete | レコードを削除する | Book#delete, Book#destroy |

---

## ActiveRecordのメソッドの対応表
| ActiveRecordのメソッド | SQL | 説明 |
| ---- | ---- | ---- |
| select | SELECT句 | フィールドを絞りたいときに使用 |
| where | WHERE句 | WHERE条件を記述 |
| limit | LIMIT句 | 件数を指定 |
| offset | OFFSET, LIMIT句 | 上から数えた場合のオフセットを指定。ページングなどで使える |
| order | ORDER BY句 | 順番を指定 |
| group | GROUP BY句 | SQLレベルでグルーピングする際に使用 |
| joins | INNER JOIN句 | INNER JOINによるテーブル同士の結合を行う |
| left_outer_joins | LEFT OUTER JOIN句 | LEFT OUTER JOINによるテーブル同士の結合を行う |
| includes | - | 関係先のモデルに対して、参照元の情報を基にして先読みできるデータを取得する |
| not | WHERE句 | 否定のWHERE条件を記述する |

---

## ActiveRecord::Relationの動作
- `ActiveRecord`に対して`Query Interface`が呼ばれる → `ActiveRecord::Relation`のインスタンスが生成される
- `ActiveRecord::Relation`に対して繰り返し`Query Interface`を呼び出すことができる
- 繰り返し呼び出した`Query Interface`は`ActiveRecord::Relation`のインスタンスに蓄積され、どんなSQLを発行するのかの情報が更新されていく
- 実際にデータが必要になった時点で蓄積された情報を基にSQLを発行し、データを取得する

## scopeとは
よく利用する検索条件に名前を付けてひとまとめにしたもの。重ねて呼び出すことも可能。
### メリット
- 繰り返し利用するクエリの再利用性が上がる
- クエリに名前を付けることで可読性が上がる

### デフォルトスコープ
デフォルトで特定のscopeが適用された状態にすることが可能。`unscoped`で利用しないクエリを組み立てることも可能。
``` Ruby
default_scope -> { order("published_on desc") }
```
---
## モデル同士のリレーション
### 簡単な例: 「1対多」
例: BooksとPublishersの関係を「1対多」とする。

```mermaid
graph LR
  A(Books) ---|多:1| B(Publishers)
```

`ActiveRecord`でこれを表現するには `has_mary`と`belongs_to`というクラスメソッドを使用する。

```Ruby
class Publisher < ApplicationRecord
  has_many :books
end
```

```Ruby
class Book < ApplicationRecord
  ...
  belongs_to :publisher
end
```
このように関連を宣言することでPublisherモデルには`books`というメソッド、Bookモデルには`publisher`というメソッドが定義される。このメソッドを通して、関連するモデルの情報を引き出したりモデル同士の関連自体を追加したりなどの操作ができるようになる。  
あるいは「1対1」という関係の定義が必要になる場合には`has_many`のかわりに`has_one`というクラスメソッドを使用する。

### 「多対多」のリレーション
BooksとAuthorsの関係性を「多対多」とする。
```mermaid
graph LR
  A(Books) ---|多:多| B(Authors)
```
多対多を表現するには、`has_many`のオプション`through`で中間テーブルを指定する。
```Ruby
class Book < ApplicationRecord
  has_many :books_authors
  has_many :authors, through: :book_authors
end
```
```Ruby
class Authors < ApplicationRecord
  has_many :books_authors
  has_many :books, through: :book_authors
end
```

### バリデージョン一覧
| バリデーション名 | 内容 |
| ---- | ---- |
| `validates_associated` | 関連先のレコードがvalidであること |
| `confirmation` | emailなどのように、確認フィールドと内容が一致する事 |
| `exclusion` | ある値が含まれていない事 |
| `format` | あるフォーマットを満たす事 |
| `inclusion` | ある値がふくまれている事 |
| `length` | 最長や最短など、ある長さを満たす事 |
| `numericality` | 数値であること、オプションでゼロ以上、偶数かなどを検査する |
| `presence` | 値が空でない事 |
| `absence` | 値が空である事 |
| `uniqueness` | 値が一意である事。scopeオプションで一意性の範囲を指定する。 |

## `Active::Record::Enum`で列挙型
例
```Ruby
enum sales_status: {
        reservation: 0, # 予約受付
        now_on_sale: 1, # 販売中
        end_of_print: 2, # 販売終了
    }
```
### Enumで定義された状態の確認
ステータスごとに述語メソッドで状態を問い合わせることが可能。
```Ruby
book.now_on_sale?
book.end_of_print?
```
enumの値の末尾に「!」付きのメソッドを使うと、そのenum値へ更新する。
```Ruby
book.now_on_sale!
book.end_of_print!
```
enumで定義したカラム（`sales_status`）を直接参照すると文字列の値が取得可能。DBに保存されている実際の値を確認するには`カラム名_before_type_cast`というメソッドを呼び出す。
```Ruby
book.sales_status
=> "end_of_print"
book.sales_status_before_type_cast
=> 2
```
enumで定義していない値を保存しようとすると `ArgumentError` が発生し、保存に失敗する。

### Enumを導入すると利用できるScope
enum型の名称で検索するためのscopeも追加される。該当するenum名で検索するscope(`.enum名`)、該当enum名を含まない検索scope(`.not_enum名`)

### 補足
Rails4.1から`ActivateRecord::Enum`というEnum機能が導入。それ以前は`enumerize`が利用されていた。

(enumerize)[https://github.com/brainspec/enumerize]

これを利用するとより高機能なEnum機能を利用できる。
enum定義に対応するscopeの名称をより柔軟に定義可能であったり
値からenum名を取得するメソッドが用意されているなど。
複雑な使い方をする場合にはこちらを利用する。

# コントローラの役割
## ルーティングからコントローラとアクションの決定法
`http://localhost:3000/books/1`というURLへのアクセスを考える。
この時アクセスしたパスは「`/books/1`」である。
このようなルーティングの設定は「`config/routes.rb`」というファイルに定義される。
```Ruby:config/routes.rb
Rails.application.routes.draw do
  get "books/:id" => "books#show"
end
```
これでGETメソッドで`/books/:id`というパターンにマッチするパスへアクセスした時に、「Booksコントローラのshowアクション」を実行するという定義になる。

---
## showアクションの中身
Railsのジェネレータを使用してBooksコントローラのひな型を作成する。
```cmd:cmd
rails g controller books
```
showメソッドを`app/controllers/books_controller.rb`に作成する。
```Ruby:app/controllers/books_controller.rb
class BooksController < ApplicationController
  def show
    @book = Book.find(params[:id])
    respond_to do |format|
      format.html
      format.json
    end
  end
end
```
---
## アクションにたいしてフックで処理を差し込む
フック: アクションの前後に処理を差し込むことのできるコールバック処理のようなもの。フィルターとも呼ぶ。

---
### フック一覧
| フック名 | 処理が実行されるタイミング |
| ---- | ---- |
| `before_action` | Actionの実行前 |
| `after_action` | Actionの実行後 |
| `around_action` | Actionの前後 |

---
## 例外処理
ユーザに通知すべき例外処理などは基本的にコントローラが担当する。

## StrongParameters
### 脆弱性
StrongParameters: Mass Assignment機能を利用する際に起こり得る脆弱性へ対抗する手段の1つ。

Mass Assignment: モデルの生成や更新の際にRubyのハッシュを使って一括で属性を設定できる便利な仕組み。

意図しない属性の変更を一般ユーザに許してしまうことがある。
### 対応策
Mass Assignmentで利用しても良いHashのkeyを許可リストとして定義することで想定していないパラメータを利用しないように制限。