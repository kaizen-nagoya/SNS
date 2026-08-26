# Q1 humhumを分析するためにあるとよい、システムの状態遷移図、シーケンス図、タイミング図をすくなくとも１つづつ作成してください。
A1 by chatgpt
はい。HumHubを**「使う」のではなく、ソースコードを分析して自作の社内SNS設計に活かす**のであれば、まず次の3種類を作るのがよいです。

今回は対象を明確にするため、以下のシナリオに絞ります。

> **ユーザーがログインし、Spaceを開き、投稿を作成する**

この一連の処理なら、HumHubの **認証・Space・Content・DB・Webアプリケーション**の関係をかなり把握できます。

---

# 1. システム状態遷移図

まず、HumHubを「ユーザーから見たシステム」としてモデル化します。

```plantuml
@startuml
title HumHub システム状態遷移図
hide empty description

[*] --> 起動

起動 --> 待受 : Webサーバ起動
待受 --> ログイン画面 : HTTP GET

ログイン画面 --> 認証中 : login
認証中 --> ログイン画面 : 認証失敗
認証中 --> ログイン済み : 認証成功

ログイン済み --> Space表示 : Space選択
Space表示 --> 投稿作成中 : Post開始
投稿作成中 --> 投稿確認 : Submit
投稿確認 --> Space表示 : 保存成功
投稿確認 --> 投稿作成中 : 保存失敗

Space表示 --> ログイン済み : Home
ログイン済み --> ログアウト中 : Logout
ログアウト中 --> ログイン画面 : Session破棄

待受 --> 停止 : Shutdown
停止 --> [*]

@enduml
```

![state](./state.png)

### この図で見るべきところ

HumHubの分析では、特に、

```text
ログイン画面
    ↓
認証中
    ↓
ログイン済み
    ↓
Space表示
    ↓
投稿作成中
    ↓
投稿確認
    ↓
Space表示
```

という**ユーザー操作による状態変化**を追うとよいです。

さらにソースコードを調べるときは、それぞれの状態に対応する、

* Controller
* Action
* Model
* Service
* Database

を対応付けていきます。

---

# 2. ログイン→Space→投稿のシーケンス図

次は、同じシナリオを「誰と誰が通信するか」で表します。

```plantuml
@startuml
title HumHub ログイン・Space・投稿 シーケンス図

actor User

participant "Browser" as Browser
participant "HumHub\nController" as Controller
participant "Authentication" as Auth
participant "User Model" as UserModel
participant "Space Model" as Space
participant "Post/Content Model" as Content
database "MariaDB / MySQL" as DB
participant "Session / Cache" as Session

== ログイン ==

User -> Browser : ログイン画面を開く
Browser -> Controller : GET /user/auth/login
Controller --> Browser : Login Form

User -> Browser : username / password
Browser -> Controller : POST /user/auth/login

Controller -> Auth : authenticate()
Auth -> UserModel : find user
UserModel -> DB : SELECT User
DB --> UserModel : User

Auth -> Auth : パスワード検証

alt 認証成功
    Auth -> Session : create session
    Session --> Auth : session ID
    Auth --> Controller : authenticated
    Controller --> Browser : Redirect Home
else 認証失敗
    Auth --> Controller : authentication failed
    Controller --> Browser : Login Error
end

== Space表示 ==

Browser -> Controller : GET /space/xxx
Controller -> Space : findByUrl()
Space -> DB : SELECT Space
DB --> Space : Space
Space --> Controller : Space
Controller --> Browser : Space Page

== 投稿 ==

User -> Browser : 投稿入力
Browser -> Controller : POST /content/create

Controller -> Content : create()
Content -> DB : INSERT Content
DB --> Content : Content ID
Content --> Controller : saved

Controller --> Browser : Redirect Space
Browser --> User : 投稿表示

@enduml
```

![sequence](./sequence.png)

これはHumHubをソース解析するときにかなり重要です。

例えば、

```text
Browser
   ↓
Controller
   ↓
Authentication
   ↓
User Model
   ↓
Database
```

という経路を実際のPHPコードと照合します。

その後、

```text
Browser
   ↓
Controller
   ↓
Space
   ↓
Database
```

さらに、

```text
Browser
   ↓
Controller
   ↓
Content
   ↓
Database
```

を調べます。

つまり、**UMLのメッセージを実際のPHPのクラス・メソッドに対応付ける**わけです。

---

# 3. ログインから投稿表示までのタイミング図

最後に、同じ処理を「時間軸」で見ます。

```plantuml
@startuml
title HumHub ログイン～投稿表示 タイミング図

robust "User" as User
concise "Browser" as Browser
robust "HumHub" as HumHub
robust "Database" as DB
robust "Session" as Session

@0
User is "未操作"
Browser is "待機"
HumHub is "待受"
DB is "待機"
Session is "未生成"

@1
User is "ログイン画面を表示"
Browser is "GET /login"
HumHub is "ログイン画面生成"

@2
Browser is "ログインフォーム表示"
HumHub is "待受"

@3
User is "ID/PW入力"
Browser is "POST /login"
HumHub is "認証処理"

@4
HumHub is "User検索"
DB is "SELECT User"

@5
DB is "結果返却"
HumHub is "パスワード検証"

@6
HumHub is "認証成功"
Session is "Session生成"

@7
Browser is "Homeへ遷移"
HumHub is "ログイン済み"

@8
User is "Space選択"
Browser is "GET /space"
HumHub is "Space取得"
DB is "SELECT Space"

@9
DB is "結果返却"
HumHub is "Space表示"

@10
User is "投稿入力"
Browser is "POST /content"
HumHub is "Content保存"
DB is "INSERT Content"

@11
DB is "保存完了"
HumHub is "投稿表示"

@12
Browser is "投稿表示"
User is "投稿確認"

@enduml
```

---

# 3つの図の役割

この3枚をセットにすると、HumHubの分析がかなり整理できます。

| UML    | 見るもの        | HumHub分析で調べる対象                    |
| ------ | ----------- | --------------------------------- |
| 状態遷移図  | **状態**      | Login / Home / Space / Content    |
| シーケンス図 | **処理・呼出関係** | Controller / Model / Service / DB |
| タイミング図 | **時間的な変化**  | HTTP → PHP → DB → Response        |

特に今回の目的なら、私は次のように**実装との対応表**を作ることをおすすめします。

```text
UML
 │
 ├─ 状態
 │    └─ Login
 │
 ├─ シーケンス
 │    └─ Controller
 │          └─ Action
 │
 ├─ クラス
 │    └─ User
 │          └─ ActiveRecord
 │
 └─ DB
      └─ user
```

そして次の段階では、**HumHubの実際のソースコードを見ながら、この3つを「推測図」から「実装に対応した図」に書き換える**のがよいです。

特に最初の解析対象としては、**Login → User → Session**を選ぶと、HumHubの `Controller / Model / Service / Session / DB` の関係を一通り追えるので、その後の **Space → Post → Comment → Notification** の解析にもつながります。
