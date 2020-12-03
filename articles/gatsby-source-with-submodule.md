---
title: "Gatsby のソースファイルとして submodule のリポジトリが使えた"
emoji: "🌝"
type: "tech"
topics: ["gatsby", "github"]
published: true
---

Gatsby でブログを作る場合、[`gatsby-source-filesystem`](https://www.gatsbyjs.com/plugins/gatsby-source-filesystem/) で Markdown ファイルなどのコンテンツを読み込んで表示するのが一般的だと思います。

[Netlify](https://www.netlify.com/) などのホスティングサービスを使用する場合、基本的には記事のコンテンツもアプリケーションと同じリポジトリに含めるのが一般的です。ですが、個人的には記事の内容とアプリケーション自体は別リポジトリにしたいと感じました。

## 記事コンテンツとアプリケーションコードを別リポジトリにする場合の選択肢

Netlify などは基本的に、1 つのリポジトリをウォッチしターゲットとするブランチに更新があった場合にデプロイが走るような構成です。そのため、アプリケーションコードとは別リポジトリで記事を管理する場合、その別レポジトリからデータを取ってくる必要があります。そこでの現実的な選択肢は 2 つです。

1. GitHub API で取得する
2. Git の submodule を使う

初めは GitHub API を使用する方向で検討していました。幸いにして、[`gatsby-source-github-api`](https://www.gatsbyjs.com/plugins/gatsby-source-github-api/)などのプラグインも存在するため、実装は容易そうです。

しかし、どうも折角 Netlify 側が GitHub と繋がっているのに、わざわざ build 時に API 経由で記事の内容を引っ張ってこないといけないのか、あまり納得がいきませんでした。

なので、よりシンプルに、Git の submodule を使って解決することにしました。

## Git の submodule を Gatsby のソースファイルとする方法

Git の submodule を理解している方であれば難なく可能なことがわかると思うのですが、以前 submodule に触れた時によくわからなかったため、今回はサンプルリポジトリを作り試してみました。

結論としては、非常に簡単にやりたいことを実現することができました。

### サンプルとその説明

こちらの["submodule-included"](https://github.com/Mizumaki-misc/netlify-submodule-sample_submodule-included)がアプリケーションのコードを含むリポジトリで、["submodule-source"](https://github.com/Mizumaki-misc/netlify-submodule-sample_submodule-source)が記事のマークダウンを含むリポジトリになります。

"submodule-included"（つまりアプリケーションのほう）の[`my-blog-starter/content`](https://github.com/Mizumaki-misc/netlify-submodule-sample_submodule-included/tree/master/my-blog-starter/content)にアクセスしてもらうとわかる通り、`my-blog-starter/content/blog`配下が submodule となっています。

![submodule image](https://storage.googleapis.com/zenn-user-upload/1hxoz2tibm8lw00oa7gj17f5g49q)

それ以外は、`gatsby-starter-blog`で始めた普通の構成になっています。

### まず submodule を追加する

記事のコンテンツを置くリポジトリを用意したら、まずはそこに対する submodule をアプリケーションのリポジトリに埋め込みます。

```sh
# git submodule add -b {branch_name} {repository_url}.git {submodule_path}
git submodule add -b master https://github.com/Mizumaki-misc/netlify-submodule-sample_submodule-source.git my-blog-starter/content/blog
```

### build コマンドに submodule を update するコマンドを仕込む

submodule としたフォルダに、対象となるリポジトリの指定したブランチの最新のコミット内容を反映させるには、`git submodule update --remote`を実行する必要があります。サイトをビルドする際にはいつも最新の情報を参照して欲しいので、これを `package.json` の `build` コマンドに追加します。

ex) [`"build": "git submodule update --remote && gatsby build"`](https://github.com/Mizumaki-misc/netlify-submodule-sample_submodule-included/blob/master/my-blog-starter/package.json#L48)

※ もし記事リポジトリに`README.md`なども含めているが、それはソースの対象外としたい場合は、[`gatsby-source-filesystem`の ignore オプション](https://github.com/Mizumaki-misc/netlify-submodule-sample_submodule-included/blob/master/my-blog-starter/gatsby-config.js#L20)を使用して`README.md`を除外してやれば良いです。

### Netlify で `npm run build` を実行するようにする

あとは、それを Netlify でのビルド時に実行するように指定するだけです。これで、アプリケーションのリポジトリと記事リポジトリを分けることができました。記事リポジトリのブランチ更新に GitHub Hooks などで Netlify の build を実行するように仕込めば、自動更新も完了です。

![success build with submodule update](https://storage.googleapis.com/zenn-user-upload/daelq56mcyyeag06bk9qgemaxcx2)

以上、Gatsby で記事コンテンツとアプリケーションコードを別リポジトリで管理する方法でした。

submodule を知っていれば当たり前の内容でしたが、あまり詳しくなかったので勉強になりました。submodule に関しては使い所ありそうだなと感じたので、今後は積極的に使っていきたいです！
