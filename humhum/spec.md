# Q1 humhumのソースコードから、１６種類の仕様書を作成してください。
A1 by chatGPT
はい。ここでは、先ほどの **16個の制約 C01～C16** を、そのままHumHubのソースコード分析用の**16種類の仕様書**に落とします。

重要なのは、今回は「HumHubにこういう機能を要求する」という要求仕様ではなく、

> **HumHubのソースコードが現在どのような制約を実現しているかを記述する仕様書**

とすることです。

HumHub公式リポジトリでは、システムを大きく **User / Spaces / Content / Modules** に分けており、YiiベースのController、Model、View、Componentなどで構成されています。([GitHub][1])
以下では、現在のmaster系ソースを基準に、実際のクラス名もできるだけ対応させています。なお、現在の変更履歴では2026年3月に1.18.2が公開されています。([GitHub][2])

---

# 仕様書01：C01 認証状態遷移制約仕様

**目的**

認証に成功していない利用者を、ログイン済みとして扱わない。

**対象**

* `humhub\modules\user\controllers\AuthController`
* `humhub\modules\user\models\User`
* `humhub\modules\user\components\User`

`AuthController::actionLogin()` は、既にログイン済みの場合はログイン処理を再実行せず、`goBack()` します。認証成功時には `login()` → `doLogin()` と進み、`Yii::$app->user->login()` が実行されます。([GitHub][3])

**制約**

```text
Login画面
   ↓
Authentication
   ↓
User status確認
   ↓
Yii::$app->user->login()
   ↓
Authenticated
```

`STATUS_ENABLED` のユーザーだけが通常のログイン処理対象になります。([GitHub][3])

**検証ポイント**

* 認証失敗時にAuthenticatedにならない
* disabled userがログインできない
* guest状態からauthenticated状態への遷移条件

---

# 仕様書02：C02 Session増殖制約仕様

**目的**

不要なSessionを増殖させない。

HumHubのUserモデルにはHTTP Sessionとの関連が定義されています。([GitHub][4])

また、ログイン時にはRemember Me設定に応じてSessionの有効期間が決定されます。User Moduleには既定のRemember Me期間も設定されています。([GitHub][5])

**制約**

```text
1 Login
   ↓
適切なAuthentication Session
```

となり、不必要なログイン状態を生成しない。

**検証ポイント**

* 多重ログイン
* Remember Me
* Session timeout
* 複数端末からのログイン

---

# 仕様書03：C03 Spaceアクセス制約仕様

HumHubではSpaceが独立した領域として扱われ、それぞれにメンバー、権限、設定、モジュールを持ちます。Spaceはプロジェクトや部署などに利用できます。([docs.humhub.org][6])

**制約**

```text
User
 ↓
Space membership / permission
 ↓
Space Content
```

というアクセス関係を維持する。

**検証ポイント**

* Space member
* Space admin
* Space moderator
* Space owner
* Guest

HumHubにはSpaceについてこれらのユーザーグループが定義されています。([docs.humhub.org][7])

---

# 仕様書04：C04 不要状態遷移制約仕様

**目的**

1つの操作から意図しない状態遷移を発生させない。

特にLogin処理では、

```text
Already Logged In
       ↓
goBack()
```

となっており、不要な再認証を避けています。([GitHub][3])

**制約**

```text
操作
 ↓
必要な状態遷移
```

だけを発生させる。

**検証対象**

* Login
* Logout
* Registration
* Space access
* Content creation

---

# 仕様書05：C05 部分情報制約仕様

**目的**

部分的に取得された情報を、完全な情報として扱わない。

Userモデルには、

```text
username
email
auth_mode
language
time_zone
last_login
authclient_id
groups
httpSessions
```

など複数の属性・関連があります。([GitHub][4])

Contentモデルも単純な本文だけではなく、Containerとの関係、Read/Write Permission、Move、Pinなどを扱います。([GitHub][8])

**制約**

```text
User
 ├─ Identity
 ├─ Group
 ├─ Permission
 └─ Session

Space
 ├─ Member
 ├─ Permission
 └─ Content
```

の必要な関係を考慮せずにアクセス判断を確定しない。

---

# 仕様書06：C06 逆方向状態遷移制約仕様

**目的**

Logout後に、旧認証状態へ意図せず戻らない。

HumHubの認証ControllerにはLogout処理があり、またSessionを利用した認証状態を管理しています。([GitHub][3])

**制約**

```text
Authenticated
      ↓
Logout
      ↓
Guest
```

に対して、

```text
Logout
  ↓
旧Authenticated Session
```

という不正な逆遷移を許さない。

---

# 仕様書07：C07 認証主体同一性制約仕様

**目的**

認証されたUserと処理対象Userを一致させる。

HumHubのUserはYiiの `IdentityInterface` を実装するUserモデルとして構成されています。([GitHub][4])

**制約**

```text
Authentication Identity
        =
Current User
```

を維持する。

**対象**

```text
humhub\modules\user\models\User
humhub\modules\user\components\User
AuthController
```

---

# 仕様書08：C08 投稿確定制約仕様

**目的**

利用者の投稿操作が確定する前に、保存済みContentとして扱わない。

HumHubのContentモデルは、ContentContainerとContentActiveRecordの関係を管理し、Read/Write Permissionなども扱います。([GitHub][8])

また、PostはHumHubのcore moduleの一つです。([docs.humhub.org][9])

**制約**

```text
Post入力
   ↓
Validation
   ↓
Permission
   ↓
Save
   ↓
Published Content
```

という確定順序を維持する。

---

# 仕様書09：C09 応答遅延制約仕様

**目的**

利用者操作に対する応答を不必要に遅延させない。

HumHubにはRedisそのものをここで直接断定するのではなく、アプリケーション側ではCache、Queue、Liveなど複数の仕組みが存在します。Core modulesには `queue`、`live`、`search` などがあります。([docs.humhub.org][9])

**制約**

```text
User action
    ↓
必要な処理
    ↓
Response
```

の間に不要な処理を挿入しない。

**分析対象**

* DB query
* Authentication
* Permission
* Content processing
* Queue
* Cache
* Frontend update

---

# 仕様書10：C10 Space確定前投稿制約仕様

**目的**

投稿対象となるContent Containerを確定する前に投稿を保存しない。

HumHubではSpaceはContentの主要なContainerであり、多くのContentが特定Spaceに関連付けられます。([docs.humhub.org][6])

Contentモデルにも `ContentContainer` との関係があります。([GitHub][8])

**制約**

```text
User
 ↓
Space
 ↓
ContentContainer
 ↓
Post
```

の順序を維持する。

これは**社内SNSでは非常に重要な制約**です。

「営業部に投稿したつもりが全社公開」という事故を防ぐ設計につながります。

---

# 仕様書11：C11 Logout後認証状態消去制約仕様

**目的**

Logout後に以前の認証状態を利用できないようにする。

HumHubのUser ModuleにはLogout URLなどの認証関連設定が存在し、AuthControllerが認証処理を担当します。([GitHub][3])

**状態モデル**

```text
Authenticated
       │
       │ Logout
       ▼
Guest
```

**制約**

```text
Guest
  ──旧Session──> Authenticated
```

という遷移を許さない。

---

# 仕様書12：C12 認証先行制約仕様

これは**シーケンス上の依存関係**を明示する仕様です。

**制約**

```text
Authentication
      ↓
Authorization
      ↓
Space Access
      ↓
Content Access
```

の順序を守る。

HumHubではUserとSpaceそれぞれにpermission contextがあり、例えばSpaceではowner/admin/moderator/member/user/guestなどのグループが定義されています。([docs.humhub.org][7])

**禁止する処理**

```text
Guest
 ↓
Space Content
```

を認証・公開設定・permission確認なしに実行すること。

---

# 仕様書13：C13 DBアクセス量制約仕様

**目的**

1つの利用者操作に対して不要なDBアクセスを増加させない。

これは性能だけでなく、**可用性・スケーラビリティ上の制約**です。

**分析対象**

```text
Controller
 ↓
Service
 ↓
Model / ActiveRecord
 ↓
Database
```

HumHubはYiiベースで、独自のActiveRecordなどを拡張しています。([docs.humhub.org][9])

**検証項目**

* 同一Userの重複取得
* 同一Spaceの重複取得
* N+1 Query
* 不要なPermission query
* Stream取得時の過剰Query

---

# 仕様書14：C14 権限情報完全性制約仕様

これは社内SNSでは非常に重要です。

**制約**

アクセス可否を決める際、

```text
User
+
User Group
+
Space Membership
+
Space Role
+
Permission
+
Content
```

の必要な情報を考慮する。

HumHubにはGlobal contextのpermissionと、Space/ProfileなどのContent Container contextのpermissionがあります。([docs.humhub.org][7])

例えば、

```text
Global Permission
       +
Space Permission
       +
Content Permission
       ↓
Access Decision
```

という構造になります。

---

# 仕様書15：C15 永続化先行制約仕様

**目的**

DBへの保存が成功する前に、利用者へ「投稿成功」と通知しない。

HumHubにはContentだけでなくNotification moduleも存在します。([docs.humhub.org][9])

**制約**

```text
Post
 ↓
Validation
 ↓
Permission
 ↓
Persistence
 ↓
Notification
```

とする。

逆に、

```text
Notification
 ↓
Persistence
```

としてはいけません。

これを守らないと、

> 「投稿したという通知は来たのに、実際の投稿が存在しない」

という不整合が発生します。

---

# 仕様書16：C16 Logout後旧Session利用禁止仕様

これはC06/C11をさらに**シーケンス・セッション依存関係**として明確にしたものです。

**制約**

```text
Authenticated Session
       ↓
Logout
       ↓
Session invalidation
       ↓
Guest
```

Logout後の古いSessionを利用して、

```text
GET Space
POST Content
GET Profile
```

などを認証済みとして実行できないこと。

HumHubのUserモデルにはHTTP Sessionとの関連が明示されています。([GitHub][4])

---

# 16仕様書の全体構造

ここまでを一覧にすると、かなりきれいに整理できます。

| No. | 仕様書          | 主対象                       | 主なUML       |
| --: | ------------ | ------------------------- | ----------- |
| C01 | 認証状態遷移       | AuthController / User     | 状態遷移        |
| C02 | Session増殖    | User / Session            | 状態・タイミング    |
| C03 | Spaceアクセス    | Space / Permission        | 状態・シーケンス    |
| C04 | 不要状態遷移       | Controller                | 状態遷移        |
| C05 | 部分情報         | User / Space / Content    | クラス・シーケンス   |
| C06 | 逆方向状態遷移      | Auth / Session            | 状態遷移        |
| C07 | 認証主体同一性      | User / Identity           | シーケンス       |
| C08 | 投稿確定         | Content / Post            | 状態・シーケンス    |
| C09 | 応答遅延         | Application / DB          | タイミング       |
| C10 | Space確定前投稿禁止 | Space / Content           | シーケンス       |
| C11 | Logout後状態消去  | Auth / Session            | 状態遷移        |
| C12 | 認証先行         | Auth / Permission         | シーケンス       |
| C13 | DBアクセス量      | ActiveRecord / DB         | シーケンス・タイミング |
| C14 | 権限情報完全性      | Permission / User / Space | シーケンス       |
| C15 | 永続化先行        | Content / Notification    | シーケンス・タイミング |
| C16 | 旧Session利用禁止 | Session / Auth            | 状態・シーケンス    |

---

## さらに重要なこと

ここで一つ、先ほどの分析から修正したほうがよい点があります。

**C01～C16は、全部がHumHubに「独立した機能」として存在するわけではありません。**

例えば、

```text
C01 認証
  ↓
C12 認証先行
  ↓
C14 権限
  ↓
C03 Spaceアクセス
  ↓
C10 Space確定
  ↓
C08 Content確定
  ↓
C15 永続化
```

という**依存グラフ**になっています。

さらにHumHubの公式ドキュメントを見ると、Spaceは独自のメンバー・権限・設定・モジュールを持ち、ContentはSpace/ProfileなどのContainerに関連付けられます。([docs.humhub.org][6])

したがって、次にやるべき分析は、

**「16仕様書 → HumHubの実際のPHPクラス → メソッド → DBテーブル → UML」**

というトレーサビリティを作ることです。

例えば、

```text
C10 Space確定前投稿禁止
        │
        ├── Space Model
        ├── Content Model
        ├── Post Module
        ├── PermissionManager
        └── Database
```

という形です。

これを**16項目すべてについてPlantUMLのクラス図・シーケンス図・状態遷移図にリンクさせる**と、HumHubを「ソースコードから逆設計する」ためのかなり本格的な仕様書体系になります。HumHub自身も、モジュール開発ではController / Model / View / Assets、Permission、Content、Notification、Migration、Securityなどを考慮する構造になっています。([docs.humhub.org][10])

[1]: https://github.com/humhub/humhub?utm_source=chatgpt.com "GitHub - humhub/humhub: HumHub is an Open Source Enterprise Social Network. Easy to install, intuitive to use and extendable with countless freely available modules. · GitHub"
[2]: https://github.com/humhub/humhub/blob/master/CHANGELOG.md?utm_source=chatgpt.com "humhub/CHANGELOG.md at master · humhub/humhub · GitHub"
[3]: https://github.com/humhub/humhub/blob/master/protected/humhub/modules/user/controllers/AuthController.php?utm_source=chatgpt.com "humhub/protected/humhub/modules/user/controllers/AuthController.php at master · humhub/humhub · GitHub"
[4]: https://github.com/humhub/humhub/blob/master/protected/humhub/modules/user/models/User.php?utm_source=chatgpt.com "humhub/protected/humhub/modules/user/models/User.php at master · humhub/humhub · GitHub"
[5]: https://github.com/humhub/humhub/blob/master/protected/humhub/modules/user/Module.php?utm_source=chatgpt.com "humhub/protected/humhub/modules/user/Module.php at master · humhub/humhub · GitHub"
[6]: https://docs.humhub.org/docs/about/humhub/?utm_source=chatgpt.com "About HumHub | HumHub Documentation"
[7]: https://docs.humhub.org/docs/admin/permissions/?utm_source=chatgpt.com "Permissions | HumHub Documentation"
[8]: https://github.com/humhub/humhub/blob/master/protected/humhub/modules/content/models/Content.php?utm_source=chatgpt.com "humhub/protected/humhub/modules/content/models/Content.php at master · humhub/humhub · GitHub"
[9]: https://docs.humhub.org/docs/develop/overview?utm_source=chatgpt.com "Overview | HumHub Documentation"
[10]: https://docs.humhub.org/docs/develop/modules/?utm_source=chatgpt.com "Getting Started | HumHub Documentation"
