---
title: "GitHub複数アカウントどうしてる？`includeIf`で.gitconfigを自動切り替えした話"
emoji: "🔀" # 絵文字は自由に変更OK
type: "tech" # tech or idea
topics: ["git", "github", "git config", "複数アカウント", "includeIf"]
published: false # 公開前は false、公開時は true に変更
---

## 💡 はじめに

こんにちは。**マリヤ・ヤギポワ**（Maria Yagipowa）です。
専門学校で情報系の講師をしながら、自分自身もまだまだ手を動かしながら、
日々学んでいるギーク寄りのエンジニアです。

この記事は *Zennでの初投稿* になります。  
せっかく書くなら「自分がちょっと感動した便利設定」から始めたいと思って、
今回のテーマを選びました。

テーマは、Gitの `includeIf`。  
特定のディレクトリ以下だけに、別の `.gitconfig` を読み込ませる機能です。  
複数のアカウントを使い分けたいとき、**いちいち設定を変えなくていい**あの感じ、
最高ですよね。

この記事では、私が実際にやってみた設定例をベースに、
なるべく丁寧に手順を解説していきます。

複数のGitHubアカウントを使っている方や、
学習用・仕事用で使い分けたい方に、特におすすめです。

## ⚙️ Gitのユーザー設定って何？

Gitでは、コミットした人の名前とメールアドレスを記録します。  
これは「**履歴に誰が関わったかを明確にするため**」にとっても大事な情報。

設定コマンドはおなじみ:

``` bash
git config --global user.name "Your Name"
git config --global user.email "YourEmail@example.com"
```

これは自分のマシンにインストールしたGitの設定で、
たいていの技術書でGitの最初の章で「ユーザー名とメールを設定しましょう」
って出てきますよね。  
`--global` オプションを付けることで、ホームディレクトリの .gitconfigという
ファイルに設定が保存されます。

---



## 📚 Git設定の3つのレベル

まずはおさらいです。  
Gitのユーザー情報（名前やメールアドレス）を設定する際、
`git config` には適用範囲を表す3つのオプションがあります。

```bash
--system   → OS全体（/etc/gitconfig）
--global   → ユーザー単位（~/.gitconfig）
--local    → プロジェクト単位（.git/config）
```

そして、それぞれの優先順位は以下の通り。上にいくほど“強く”効きます 👇

```bash
▲ 最優先（強い）
|  .git/config（ローカル設定）
|  ~/.gitconfig（グローバル設定）
▼ /etc/gitconfig（システム設定）
```

このあたりの内容は、だいたいどのGitの技術書にも最初のほうに載ってますよね。  
でも正直なところ、私はずっと `--global` しか使ってませんでした。

だって、GitHubアカウントは1つだけだったし、それで特に困ったこともなかったからです。

だけど今回、Zennで記事を書いたり、技術メモをGitで管理したりと、
 **仕事用（授業用）とは別のGitHubアカウントを使いたい場面が出てきた**んです。

---

そうだ！今こそ `--local` を使えばええんや！
 **これ、進研ゼミでやったやつや！（やってない）**

……って思ったんですが、**そのとき私、気づいたんです。**

## 🤯こんなお悩みはありませんか？

> 「**毎回 **`--local` **で設定変えるの、正直めんどい。」
> 「しかも設定忘れて、職場アカウントでコミットしたら最悪やで😇」**

実は、私は今後こんな感じで `git-projects/` 以下にプライベートで
学習ログや記事用のプロジェクトをたくさん作っていく予定なんです。

```
📁 C:\Users\〇〇〇\Documents\git-projects\
├─ 📁 zenn-content/           	← Zenn CLIで書いて公開する記事
├─ 📁 study-log-sc2025/      	← 情報処理試験の学習記録
├─ 📁 git-memo/              	← Gitコマンドやトラブル時の備忘録
├─ 📁 til-log/               	← 「今日学んだこと」
├─ 📁 mokumoku-log/          	← もくもく会の活動記録・成果ログ
```

だがしかし、予定は未定🫠  
もしかしたら、今後もっとディレクトリが増えるかもしれないし、  
アカウントだってもう1つ2つ増えるかもしれない。

……え、**そのたびに毎回設定するの？ 地獄やん。**  
忘れるのが仕様の私にとって、これはもう……事故の予感しかしない😇

---

## 🔮そんなときは `includeIf` の出番！

「**プログラマの三大美徳の１つは怠惰やぞ**🦥」と思いながら調べていたら、  
ありました、救世主的な機能が。

その名も `includeIf`。  
**Gitは、特定のディレクトリ以下だけに適用される設定ファイルを**
**“自動で読み込む”ことができる**んです！

やっぱりね、そりゃぁありますよね。  
ありがとう、リーナス・トーバルズ……！（心の底から）

つまり「このフォルダ以下は全部“個人用”」みたいな感じで、  
**設定を切り替えるスイッチを“仕込んでおく”ことができる**んです。かしこい。

---

### 🧩`.gitconfig` に以下を追加：

まずは通常の `~/.gitconfig` に、以下の設定を追記します：  
（※編集には VSCode やメモ帳などのテキストエディタを使えばOKです）

```ini
[includeIf "gitdir:C:/Users/your-username/Documents/git-projects/"]
  path = ~/.gitconfig-personal
```
| 項目      | 説明                                              |
| --------- | ------------------------------------------------- |
| `gitdir:` | 対象のディレクトリ（この下のGitリポジトリに適用） |
| `path =`  | 読み込む `.gitconfig` のファイルパス              |

> ✅ この例では「`git-projects/` 以下のリポジトリでは、
> `~/.gitconfig-personal` を追加で読み込む」という意味になります。  
> この設定を使えば、**ディレクトリごとに別のユーザー情報を自動で切り替えることができます。**

※ `.gitconfig` の末尾に追記しておくと見落としにくいです。

💡 `gitdir:` で指定するパスや、`path =` のファイル名は **自由に決めてOK** です。  
たとえば `~/.gitconfig-zenn` や `~/.gitconfig-github-alt` など、用途に合わせて
わかりやすくしても大丈夫！

---

### 🧾`~/.gitconfig-personal` に記述する内容はこちら：

```ini
[user]
  name = Your Name 
  email = your.name@example.com
```

この設定により、たとえば `git-projects/` 以下のリポジトリで作業する場合は、  
自動的にここで設定したユーザー名・メールアドレスが使われるようになります。

設定し忘れなし。毎回 `--local` で書かなくてもいい。  
**最高かよ🥰**

---

### 🔁 応用編：複数のアカウントに切り替える場合

💡 `includeIf` は複数書けるので、用途に応じて切り替えることができます。  
「work用」「趣味用」など、分けたい人には特におすすめ！

> 🪄 このセクションは応用編なので、  
> 「まずは1つだけ設定したい！」という人は飛ばしてもOKです！

```ini
[includeIf "gitdir:C:/Users/your-username/Documents/work/"]
  path = ~/.gitconfig-work

[includeIf "gitdir:C:/Users/your-username/Documents/hobby/"]
  path = ~/.gitconfig-hobby
```
それぞれの設定ファイルに、以下のように自分用のユーザー情報を記述しておけばOKです：

#### `~/.gitconfig-work`

```ini
[user]
  name = Your Real Name
  email = your.work.email@company.com
```

#### `~/.gitconfig-hobby`

```ini
[user]
  name = Hobby Name
  email = hobby.dev@example.com
```

こうやって分けておけば、**仕事・趣味・Zenn用・チーム開発用などで自動切り替え**できて、  
ミスの予防にもなるんです。  
リーナス・トーバルズ、ほんまにありがとう（2回目）🙌
 ……あなたが神か。

---

## ✅ よく使うコマンドまとめ

| 目的                             | コマンド                             |
| -------------------------------- | ------------------------------------ |
| 今、どの `user.name`？           | `git config user.name`               |
| どこから設定されてるの？         | `git config --show-origin --list`    |
| includeIf 設定ちゃんと効いてる？ | `.gitconfig-personal` が出てるか確認 |

------

## 🧠 おまけ：`--show-origin` ってなに？

```bash
git config --show-origin --list
```

これ、地味に便利です。

- どの設定ファイルが効いてるか丸見え
- `includeIf` の確認にも最適

> ☝️ `--show-origin` は「設定がどこから来たか」を表示するオプション。
>  名前の途中にある `origin` は「出どころ（由来）」という意味で、Gitのリモート名 `origin` とは関係ありません。たまたまです（笑）

つまり、「**今どの設定が使われていて、どこから来ているのか**」を把握するには、このコマンドが一番確実です。

---

## ✍️ 設定確認コマンドの出力例：

💡 以下は `git config --show-origin --list` の出力例です。
 （※パスや設定ファイルは、ご自身の環境によって異なることがあります）

```bash
$ git config --show-origin --list

file:/etc/gitconfig          core.editor=vim
file:/etc/gitconfig          color.ui=auto
file:/Users/your-username/.gitconfig     user.name=Your Name
file:/Users/your-username/.gitconfig     user.email=your.name@example.com
file:/Users/your-username/.gitconfig     includeif.gitdir:C:/Users/your-username/Documents/git-projects/.path=~/.gitconfig-personal
file:/Users/your-username/.gitconfig-personal     user.name=Your Alt Name
file:/Users/your-username/.gitconfig-personal     user.email=your.alt@example.com

```

💡 このコマンドは、現在有効な設定と、
それぞれ「どの設定ファイルから来ているか（出どころ）」を表示してくれます。

たとえば `~/.gitconfig-personal` の設定がちゃんと読み込まれていれば、
このように `file:~/.gitconfig-personal` の行が表示されます。

------

## 🎯 まとめ

- 複数のGitアカウントを使い分けるなら、`includeIf` は本当に便利
- 一度設定してしまえば、自動で切り替わる安心感
- 設定忘れがちな人（＝私）でも事故を防げます

---

## 📝おわりに

というわけで、今後も学習ログや技術メモをGitで管理していきたい私にとって、
この`includeIf`設定は本当に便利でした。  
この記事が、同じように「アカウント切り替え面倒くさい！」と思っている  
誰かの役に立てば嬉しいです🥰

もしこの記事が役に立ったら、いいねやスクラップでの感想もらえると励みになります✍️
💬 ご意見・質問などあれば、お気軽にScrapでどうぞ！  
👉 [Scrapはこちら](https://zenn.dev/yagipowa/scraps/9ff29a4f9bceed)

