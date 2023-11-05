# Git とはなにか？

Gitは分散バージョン管理システムの一種です。
もともとはLinuxカーネルの開発のために必要になったため開発されました。
大規模な開発において、ソースコードのバージョン管理は不可欠です。
バージョン管理システムには長い歴史があり、Gitの登場以前から有償・無償を含め様々な製品が作られてきました。
その中でもGitは多人数・非同期でのソースコード管理を念頭において作られ、Linuxカーネルの開発を始めとしたさまざまなソフトウェア開発の場面で使われています。
この節ではGitを使うことでできることや主要機能の概略を眺めていくことにします。

## ロールバック

アプリケーションの開発になぜバージョン管理が不可欠なのでしょうか。
そもそもバージョン管理とはどのようなものでしょうか。
バージョン管理はソースコードに対して行われた変更を記録しておき、後から参照するためのシステムです。

Gitを使うことで可能な便利な機能の一つにロールバックがあります。
ロールバックは任意の時点の変更状態へソースコードを巻き戻す機能で、ほぼすべてのバージョンコントロールシステムに搭載されています。
例えばファイルを編集している最中に、編集前の状態に戻したくなったりしたことはないでしょうか？またはもとに戻す可能性があると感じてコピーを作成してからファイルを編集したことはないでしょうか？
Gitでは「コミット」と呼ばれる編集単位で、ファイルに対して行われた変更を復元することができます。

## 多人数での非同期編集

Gitは複数人での編集も念頭に置かれています。
Gitでの非同期な編集は次のように行われます。
まず、Gitの管理対象となっているファイルが含まれた「リポジトリ」を作成します。
Gitはこのリポジトリ上で行われた変更を追跡し、ユーザーからの指示に応じて記録します。
次に、このリポジトリを複数人でコピー（Gitの用語で「クローン」とよばれます）し、それぞれのユーザーがそれぞれ変更を加え、Gitに変更履歴として記録します。
この記録のことを「コミット」とよびます。
最後にこれらの独立したコミットを統合し、一つにまとめるコミットを作成します。このように複数のコミットをまとめるコミットを作成する操作を「マージ」と呼びます。
このように、複数人で編集を行う場合でも非同期的に（つまり同時に操作することなく）Gitを操作することができます。

## Gitの使い方

### 変更履歴の考え方

Gitでは複数のファイルの状態をまとめ、論理的な意味を持つ一連の変更をコミットとして扱います。
論理的な意味を持つ一連の変更というのは簡単に言えば変更内容に名前をつけられるような内容ということです。
例えば「〇〇という機能を追加した」「〇〇というコンポーネントのバージョンを上げた」などです。
一つのコミットには複数のファイルへの変更が含まれていることに注意してください。
ファイルの新規作成であったり、消去、名前の変更も変更の一種としてみなされます。
こうした変更には変更される「前」の状態があるはずです。
変更される前の状態もGitによって管理されているということを考えると、変更（つまりコミット）は何に対する変更なのかという情報をもっているはずです。
このような一つ前の変更（コミット）をグラフ理論から借用した用語で「親」と呼びます。
ほとんどのコミットはある一つのコミットを親に持ちます。
例外はマージした際に作成されるマージコミットとリポジトリに最初にファイルを作成した場合に作成されたコミットなどごくわずかです。

ここまでの様子を整理しましょう。
Gitの変更履歴を構成している最小単位は、複数のファイルの状態内容を保持したコミットとよばれる構造に記録されています。
ほとんどのコミットは一つの親を持ち、そのコミットと親との差分が変更内容となります。

### ステージとコミット

ここからは実際にgitのコマンドを動かしつつ、どのようにgitが動作するのか見ていきましょう。
gitのリポジトリの初期化をした直後の状態から始めます。まず現在の状態を確認します。

    $ git status
    On branch main

    No commits yet

    nothing to commit (create/copy files and use "git add" to track)

No commits yet
と言われていますね。まだ何もコミットしたものが無いのでそう言われています。
なにか適当なファイルを作ってみます。

    $ echo 'i am the first!' > hoge.txt
    $ cat hoge.txt              
    i am the first!

\"i am the first!\" と書かれた \"hoge.txt\"
という名前のファイルができました。

コミットに先立ってこのファイルを「ステージ」します。
ステージはコミットに含める変更点を指定する作業です。

    $ git add hoge.txt
    $ git status
    On branch main

    No commits yet

    Changes to be committed:
      (use "git rm --cached <file>..." to unstage)
        new file:   hoge.txt

Changes to be committed の欄に new file: hoge.txt
としてコミット予定のファイルとして hoge.txt が現れています。
これでコミットの準備ができました。
コミットには意味があるはず...（ですよね？）なのでコミットメッセージが必須です。
ここでは適当なメッセージにしておきますが、一般的には何を、なぜコミットしたのか書くようにするのが良い作法です。

    $ git commit -m 'this is first commit'
    [main (root-commit) c7e9069] this is first commit
     1 file changed, 1 insertion(+)
     create mode 100644 hoge.txt

無事にコミットできたようです。 コミットの履歴を表示してみましょう。

    $ git log
    commit c7e90697213aa28fdb0f43fbba38f417806bb0fc (HEAD -> main)
    Author: ESASHIKA Kaoru <git@pluser.dev>
    Date:   Sun Nov 5 16:21:42 2023 +0900

        this is first commit

微妙に表示内容が違うかもしれませんが、似たような内容が表示されると思います。
一番上の commit
以降に表示されている16進数の英数字ががコミットハッシュ（コミットID）です。
コミットハッシュはそのコミットにつけられた全宇宙で唯一の識別子です。
コミットを操作する場合はコミットハッシュを指定する場合があります。
その後にでてきている (HEAD -\> main) はブランチの状態を示しています。
ブランチについては後ほど説明します。

せっかくなのでもう一つコミットを積んでみましょう。
そのまえに先程作ったファイルを編集しておきます。

    $ echo 'this is second line' >> hoge.txt
    $ cat hoge.txt 
    i am the first!
    this is second line

hoge.txt に一行追加しました。 git status
して今の状態を確認してみましょう。

    $ git status
    On branch main
    Changes not staged for commit:
      (use "git add <file>..." to update what will be committed)
      (use "git restore <file>..." to discard changes in working directory)
        modified:   hoge.txt

    no changes added to commit (use "git add" and/or "git commit -a")

Changes not staged for commit の欄に hoge.txt がありますね。
変更されているけれどステージされていない状態のファイルということです。

git add hoge.txt （もしくはgit add .）してステージしましょう。

    $ git add .
    $ git status
    On branch main
    Changes to be committed:
      (use "git restore --staged <file>..." to unstage)
        modified:   hoge.txt

無事にステージされました。先程と同様にコミットしましょう。

    $ git commit -m 'this is second commit'
    [main 5788a80] this is second commit
     1 file changed, 1 insertion(+)

変更履歴を表示してみましょう。

    ❯ PAGER= git log
    commit 5788a805fb158ddfe41408d809c31cf62eddcaa4 (HEAD -> main)
    Author: ESASHIKA Kaoru <git@pluser.dev>
    Date:   Sun Nov 5 16:45:56 2023 +0900

        this is second commit

    commit c7e90697213aa28fdb0f43fbba38f417806bb0fc
    Author: ESASHIKA Kaoru <git@pluser.dev>
    Date:   Sun Nov 5 16:21:42 2023 +0900

        this is first commit

2つのコミットが表示されているのが確認できますね。

せっかくなので最後のコミットを取り消してみましょう。

    $ git reset --hard c7e9069               
    HEAD is now at c7e9069 this is first commit

    $ git log        
    commit c7e90697213aa28fdb0f43fbba38f417806bb0fc (HEAD -> main)
    Author: ESASHIKA Kaoru <git@pluser.dev>
    Date:   Sun Nov 5 16:21:42 2023 +0900

        this is first commit

    $ cat hoge.txt
    i am the first!

一番最後のコミットは忘れ去られ、なかったことにされました！
ところで、やはり気が変わったのでさっきの取り消しもなかったことにしましょう。

    $ git reset --hard 5788a80
    HEAD is now at 5788a80 this is second commit

    $ PAGER= git log                                           
    commit 5788a805fb158ddfe41408d809c31cf62eddcaa4 (HEAD -> main)
    Author: ESASHIKA Kaoru <git@pluser.dev>
    Date:   Sun Nov 5 16:45:56 2023 +0900

        this is second commit

    commit c7e90697213aa28fdb0f43fbba38f417806bb0fc
    Author: ESASHIKA Kaoru <git@pluser.dev>
    Date:   Sun Nov 5 16:21:42 2023 +0900

        this is first commit

    $ cat hoge.txt
    i am the first!
    this is second line

これで取り消しを取り消しできましたね！

# GitHubとは

## GitとGitHubとの関係

人によってはGitをGitHubの略称だと誤解している方もいらっしゃるでしょう。ここではっきりさせておきますが、両者は別のものです。
ここまでGitはバージョン管理システムだと説明してきました。ではGitHubは一体何なのでしょうか？
GitHubはGitHub社（Microsoftの子会社）が提供するWebサービスです。GitHubではGitのリポジトリをホストしてWeb上で見やすく表示してくれます。また、GitHub上ではPullRequestという機能によって、コピーされたリポジトリ間でマージなどの操作や、それに伴うレビューなどを支援してくれます。
Gitのみで複数人での開発は可能ですが、中心（つまりハブ）になるリポジトリをGitHubに置くことでより便利にGitを使うことができます。

# 用語集

-   リポジトリ:: Gitが管理対象としている一連のファイルが含まれたもの。
-   コミット::
    Gitにファイルが変更されたことを知らせ、変更履歴として記録するように指示すること。また、その変更そのもの。
-   ロールバック:: 変更を巻き戻すこと。
-   マージ:: 複数のコミットをまとめる特殊なコミットを作成すること。
-   クローン:: リポジトリをコピーすること。
-   ステージ::
    コミット前にコミットに含める変更点を指定すること。また指定された変更点。
-   コミットハッシュ（コミットID）::
    コミットを識別するための全宇宙で唯一の識別子。他の人の変更内容とは絶対に重複しない。

# 参考文献

-   \[Gitのソースコード\](<https://github.com/git/git>)
-   \[開発運用研修2018 Git/GitHub 講義編 -
    サイボウズ\](<https://speakerdeck.com/cybozuinsideout/2018-05a-git-and-github-lecture>)
-   \[開発運用研修2018 Git/GitHub 演習編 -
    サイボウズ\](<https://speakerdeck.com/cybozuinsideout/2018-05b-git-and-github-exercise>)
-   \[Pro Git\](<https://git-scm.com/book/ja/v2>)
