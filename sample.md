# Q1 社内SNSを１から作りたい。サンプルのソースコードのあるプロジェクトを８つ示してください。
A1 by ChatGPT
はい。**「社内SNSを1から作るための参考実装」**という観点なら、単なるSNSではなく、**ユーザー認証・プロフィール・投稿・コメント・いいね・グループ・通知・権限管理**などが含まれるプロジェクトを選ぶのがよいです。

2026年8月時点で、特に参考になりそうな8プロジェクトを挙げます。

| # | プロジェクト                    | 主な技術                       | 社内SNSへの適性 | 特徴                            |
| - | ------------------------- | -------------------------- | --------- | ----------------------------- |
| 1 | **HumHub**                | PHP / Yii2                 | ★★★★★     | Enterprise Social Networkそのもの |
| 2 | **Elgg**                  | PHP / MySQL                | ★★★★★     | SNSエンジンとして非常に拡張しやすい           |
| 3 | **Mattermost**            | Go / React / PostgreSQL    | ★★★★☆     | Slack型の社内コミュニケーション            |
| 4 | **Liberu Social Network** | Laravel / PHP / Livewire   | ★★★★☆     | 現代的なSNS実装                     |
| 5 | **Spruce**                | Node.js / MongoDB          | ★★★★☆     | 小規模でコードを読みやすい                 |
| 6 | **SocialPulse**           | React / Node.js / MySQL    | ★★★★☆     | React＋REST API型の教材として良い       |
| 7 | **OpenVK**                | PHP / MySQL                | ★★★☆☆     | 大規模SNSの機能構成を研究できる             |
| 8 | **FamNet**                | Ruby on Rails / PostgreSQL | ★★★☆☆     | 閉じたコミュニティ型SNS                 |

### 1. HumHub — 最有力

HumHub

[GitHub: humhub/humhub](https://github.com/humhub/humhub?utm_source=chatgpt.com)

これは今回の目的には**一番ストレートな教材**です。

HumHub自身が「Open Source Enterprise Social Network」を名乗っており、ユーザー、プロフィール、Space（部署・プロジェクトなど）、投稿、コメント、通知、権限、検索などを持っています。さらにモジュールによる拡張を前提とした設計です。([GitHub][1])

特に、

```text
User
  ├── Profile
  ├── Follow
  └── Permission

Space
  ├── Department
  ├── Project
  └── Event

Content
  ├── Post
  ├── Comment
  ├── File
  ├── Wiki
  └── Task

Notification
Authentication
Search
```

という構造は、**社内SNSの要求分析・UMLモデルを作る際の参考**にもかなり使えます。

---

### 2. Elgg

Elgg

[GitHub: Elgg/Elgg](https://github.com/Elgg/Elgg?utm_source=chatgpt.com)

Elggは「SNSそのもの」よりも、**SNSを構築するためのフレームワーク／エンジン**として見ると面白いです。

PHP/MySQLをベースに、

* User
* Profile
* Groups
* Posts
* Comments
* Friends/Followers
* Activity
* Notifications
* Plugins
* Authentication
* Access control

などを拡張可能な形で持っています。公式にも、組織内部のコラボレーションプラットフォームとして利用できると説明されています。([GitHub][2])

**「自社独自のSNSを作る」なら、HumHubとElggはまず比較する価値があります。**

---

### 3. Mattermost

Mattermost

[GitHub: mattermost/mattermost](https://github.com/mattermost/mattermost?utm_source=chatgpt.com)

これはFacebook型SNSというより、

```text
Slack
    ↓
社内コミュニケーション
    ↓
Mattermost
```

という位置付けです。

Go + React + PostgreSQLで構成され、チャット、ワークフロー、音声通話、画面共有などを備えています。([GitHub][3])

社内SNSを、

> 「社員が自由に投稿するSNS」

ではなく、

> **「会社の情報共有基盤」**

として作るなら、非常に参考になります。

---

### 4. Liberu Social Network

[GitHub: liberusoftware/social-network-laravel](https://github.com/liberusoftware/social-network-laravel?utm_source=chatgpt.com)

比較的新しい技術スタックを見るならこれです。

* Laravel 12
* PHP 8.5
* Livewire 4
* Filament 5
* Role Based Access Control
* Privacy settings
* Modular architecture

という構成です。([GitHub][4])

特に**管理画面＋権限管理**の実装を見る教材としてよいでしょう。

---

### 5. Spruce

[GitHub: dan-divy/spruce](https://github.com/dan-divy/spruce?utm_source=chatgpt.com)

これはかなり「教材向き」です。

```text
Node.js
Express
EJS
MongoDB
```

という比較的シンプルな構成で、

* Login
* User
* Profile
* Follow
* Feed
* Post
* Comment
* Like
* Chat
* Notification
* API

などが実装されています。([GitHub][5])

**「巨大なHumHubのコードは難しいので、まずSNSの基本構造を理解したい」**場合はこちらがよいです。

---

### 6. SocialPulse

[GitHub: SocialPulse](https://github.com/xloyb/SocialPulse-React-Node.js-MySQL-Social-Media-App-Full-Stack-Social-Network-App-Open-Source?utm_source=chatgpt.com)

React + Node.js + Express + MySQLという、かなり一般的なWebアプリ構成です。

```text
React
   ↓
React Query
   ↓
Express / Node.js
   ↓
MySQL
```

という構成なので、**フロントエンドとバックエンドを分離した社内SNS**を考えるときに参考になります。投稿、プロフィール、設定、Storiesなども実装されています。([GitHub][6])

---

### 7. OpenVK

[GitHub: OpenVK/openvk](https://github.com/OpenVK/openvk?utm_source=chatgpt.com)

VK風のSNSをオープンソースで実装したプロジェクトです。

PHP 8.2+、MySQL互換DB、Redisなどを使います。([GitHub][7])

社内SNSとしてそのまま使うというより、

**「SNSにはどういうデータモデル・画面・機能が必要なのか」**

を調べる教材として面白いです。

---

### 8. FamNet

[GitHub: lassiter/FamNet](https://github.com/lassiter/FamNet?utm_source=chatgpt.com)

これは「家族・小規模コミュニティ向けの閉じたSNS」です。

Ruby on Rails + PostgreSQLで、

* User
* Group
* Post
* Comment
* Event
* Authorization
* Notification

などを実装しています。ユーザー、moderator、admin、ownerというロールもあります。([GitHub][8])

**「一般公開SNSではなく、閉じた組織」という点で社内SNSとの類似性があります。**

---

## 私なら、この3つから研究を始めます

今回の「**社内SNSを1から作る**」という目的なら、8個全部を読む必要はありません。

### A. HumHub

**完成形を見る**

```text
社内SNS
   ↓
User
Space
Post
Comment
File
Wiki
Task
Notification
Permission
Search
```

という全体像を把握する。

### B. Spruce

**小さい実装を見る**

```text
Browser
   ↓
Express
   ↓
MongoDB
```

SNSの基本的なCRUD、認証、Feed、Followなどを理解する。

### C. SocialPulse

**現代的なWeb構成を見る**

```text
React
   ↕
REST API
   ↕
Node.js
   ↕
MySQL
```

これをベースに、自社SNSを設計する。

---

## そして、今回の用途なら「SNS」より少し違う設計をおすすめします

例えば私は、社内SNSを単なる

```text
社員 → 投稿 → コメント
```

にはしません。

むしろ、

```text
                    ┌── 部門
                    ├── プロジェクト
社員 ──── 所属 ────┤
  │                 └── 委員会
  │
  ├── 投稿 ── コメント
  │     │
  │     ├── 添付ファイル
  │     ├── タグ
  │     └── リンク
  │
  ├── フォロー
  │
  ├── メッセージ
  │
  └── 通知

管理者
  ├── ユーザー管理
  ├── 組織管理
  ├── 権限管理
  ├── 投稿管理
  └── 監査ログ
```

のようにして、**「社内情報共有プラットフォーム」**として設計します。

特に社内利用では、一般SNSにはない

* LDAP / Active Directory
* SSO
* 部署・役職
* RBAC
* 退職者の無効化
* 監査ログ
* ファイルアクセス権
* 機密情報の公開範囲
* 投稿の保存期間
* 管理者権限
* バックアップ
* AIによる社内検索/RAG

が重要になります。

**HumHubの「User / Space / Content / Module」という構造は、この設計の出発点としてかなり優秀です。** ([GitHub][1])

もし「1から作る」が**学習目的**なら、次の段階では、上の8プロジェクトを参考にして、**「最小構成の社内SNS」を Python + FastAPI + PostgreSQL + React で、Docker Compose付きで実際に作る**のがかなり面白いと思います。

[1]: https://github.com/humhub/humhub?utm_source=chatgpt.com "GitHub - humhub/humhub: HumHub is an Open Source Enterprise Social Network. Easy to install, intuitive to use and extendable with countless freely available modules. · GitHub"
[2]: https://github.com/elgg/elgg?utm_source=chatgpt.com "GitHub - Elgg/Elgg: A social networking engine in PHP/MySQL · GitHub"
[3]: https://github.com/mattermost/mattermost?utm_source=chatgpt.com "GitHub - mattermost/mattermost: Mattermost is an open source platform for secure collaboration across the entire software development lifecycle.. · GitHub"
[4]: https://github.com/liberusoftware/social-network-laravel?utm_source=chatgpt.com "GitHub - liberusoftware/social-network-laravel: Social network built on the latest technologies, including Laravel 12, PHP 8.5, Filament 5 and Livewire 4 · GitHub"
[5]: https://github.com/dan-divy/spruce?utm_source=chatgpt.com "GitHub - dan-divy/spruce: A social networking platform made using Node.js and MongoDB · GitHub"
[6]: https://github.com/xloyb/SocialPulse-React-Node.js-MySQL-Social-Media-App-Full-Stack-Social-Network-App-Open-Source?utm_source=chatgpt.com "GitHub - xloyb/SocialPulse-React-Node.js-MySQL-Social-Media-App-Full-Stack-Social-Network-App-Open-Source: SocialPulse is an open-source social media app built with React, Node.js, and MySQL. It enables users to connect, share posts, and engage with friends. It's a fantastic opportunity for developers to learn full-stack development and contribute to an exciting project. · GitHub"
[7]: https://github.com/openvk/openvk?utm_source=chatgpt.com "GitHub - OpenVK/openvk: Not-yet-federated open source social network inspired by VK. · GitHub"
[8]: https://github.com/lassiter/FamNet?utm_source=chatgpt.com "GitHub - lassiter/FamNet: A closed social network for families and close community groups. · GitHub"
