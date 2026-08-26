# Q1 humhumをgit cloneしました。インストールするにはどうしたらいいですか。
はい。ここは少し注意点があります。**`humhub/humhub` を `git clone` した場合、通常の利用者向けパッケージを展開したのとは違い、開発用ソースから起動する形**になります。

社内SNSを「勉強しながら1から作る」目的なら、私は **Dockerで起動する方法**をおすすめします。HumHub公式にも開発用Docker環境があります。MariaDB、Redis、Webサーバーなどをまとめて用意できます。([GitHub][1])

## まず確認

おそらく今は、

```bash
git clone https://github.com/humhub/humhub.git
cd humhub
```

という状態だと思います。

ここで重要なのは、**この `humhub` リポジトリ自身には、Dockerで全部を立ち上げる構成が入っているわけではない**ことです。

公式の開発用Docker環境は別リポジトリです。

### おすすめ：開発用Dockerを使う

例えばLinux/WSL2/macOSなら、

```bash
git clone https://github.com/humhub/docker-dev.git
cd docker-dev

cp humhub.env.dist humhub.env
cp .env.dist .env
```

その後 `.env` を編集して、

```text
HUMHUB_DOCKER_DOMAIN=localhost
```

などを設定します。

そして、

```bash
docker compose up -d
```

です。公式の手順もこの流れになっています。([GitHub][1])

起動後、

```text
http://localhost:8404
```

または設定したドメインからアクセスする形になります。

---

## ただし、今回の目的ならもう一段おすすめがあります

小川さんがやりたいのは単にHumHubを使うことではなく、

> **HumHubを教材にして、社内SNSを1から作る**

ということだと思います。

その場合、

```text
                    HumHub
                      │
        ┌─────────────┴─────────────┐
        │                           │
   実際に動かす                 ソースを読む
        │                           │
     Docker                    PHP / Yii2
        │                           │
    MariaDB                    データモデル
    Redis                      Module
    Web server                 Controller
                                View
                                ActiveRecord
```

という二本立てにするのがよいです。

HumHub自身も、ソフトウェアを大きく **User / Spaces / Content / Modules** の4つに分けて説明しています。([GitHub][2])

これは社内SNSの設計を考える上でかなり参考になります。

---

# もしWindowsなら

Windowsの場合は、**WSL2 + Docker Desktop**をおすすめします。

まずPowerShellで、

```powershell
wsl --install
```

Docker Desktopをインストールして、**WSL 2 integration**を有効にします。

その後UbuntuなどのWSL環境で、

```bash
git clone https://github.com/humhub/docker-dev.git
cd docker-dev

cp humhub.env.dist humhub.env
cp .env.dist .env

nano .env
```

として、

```text
HUMHUB_DOCKER_DOMAIN=localhost
```

を設定。

そして、

```bash
docker compose up -d
```

です。

確認は、

```bash
docker compose ps
```

です。

例えば、

```text
NAME              STATUS
humhub            running
db                running
redis             running
caddy             running
```

のようになればOKです。

---

## ここで一つ重要

**今 `git clone` した `humhub` は消さないでください。**

例えば、

```text
~/work/
│
├── humhub/          ← 今cloneしたもの
│
└── docker-dev/      ← これからclone
```

としておくのがいいです。

そうすると、

```text
docker-dev
   │
   └── 実際にHumHubを動かす

humhub
   │
   └── HumHub本体のソースコードを読む・改造する
```

という使い分けができます。

---

## さらに面白い進め方

今回の目的なら、私は最終的にこういう構成にするのをおすすめします。

```text
                 社内SNS
                    │
       ┌────────────┴────────────┐
       │                         │
    HumHub                    自作SNS
       │                         │
   ソース解析                 設計・実装
       │                         │
       ├─ User                  User
       ├─ Space                 Department
       ├─ Content               Project
       ├─ Module                Post
       ├─ Notification          Comment
       └─ Permission            Notification
                                 Permission
```

そして**HumHubのデータモデルをUML/PlantUMLに起こしてから、自作SNSのモデルを設計する**と、かなり良い教材になります。

HumHubはPHP/Yii2なので、例えば最初に

```text
protected/
├── models/
├── controllers/
├── components/
├── modules/
└── config/
```

あたりを追っていくと、Webアプリケーションの構造を理解しやすいです。

なお、HumHubの公式要件では現在、MariaDB 10.11+ / MySQL 8.0+が最低要件で、推奨はMariaDB 11.8 / MySQL 8.4、PHP 8.3～8.5系が対象になっています。([HumHub][3])

**もし今の環境がWindows 11 + Docker Desktopなら、次に「`git clone`したところから、localhost:8404でHumHubを表示するまで」を、コマンドを1個ずつ確認しながら進めましょう。**

[1]: https://github.com/humhub/docker-dev?utm_source=chatgpt.com "GitHub - humhub/docker-dev: HumHub - Docker Image (Dev Version) · GitHub"
[2]: https://github.com/humhub/humhub?utm_source=chatgpt.com "GitHub - humhub/humhub: HumHub is an Open Source Enterprise Social Network. Easy to install, intuitive to use and extendable with countless freely available modules. · GitHub"
[3]: https://docs.humhub.org/docs/admin/requirements/?utm_source=chatgpt.com "System Requirements | HumHub Documentation"
