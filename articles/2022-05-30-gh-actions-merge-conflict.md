---
title: "[GitHub Actions] フォーク元からコンフリクトマーカーをつけたままマージしてプルリクエストを作成する"
emoji: "❄️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [githubactions]
published: true
---

# はじめに

こちらの記事を見て，コンフリクトマーカーをつけたままマージしてプルリクエストを作成するワークフローを作ってみようと思い，GitHub Actionsに入門してみました．

https://zenn.dev/smikitky/articles/0d250f7367eda9#%E5%8E%9F%E6%96%87%E3%81%B8%E3%81%AE%E8%BF%BD%E5%BE%93%E3%82%92-bot-%E3%81%A7%E8%A1%8C%E3%81%86%EF%BC%88%E3%82%AA%E3%83%97%E3%82%B7%E3%83%A7%E3%83%B3%EF%BC%89

# 作ったもの

コメントに色々と書いていますが，簡単に
1. 新規ブランチを作る
2. フォーク元`upstream`をフェッチして，マージする
3. (コンフリクトがあれば)コンフリクトマーカーごとコミットも行う
4. (コミットがあれば)プッシュする．
5. (コミットがあれば)プルリクエストを作る．

の流れになります．

また，
* コンフリクトがあるかどうかを調べるために，マージした後，コミットされていないファイルを数えています．
* コミットがあるか調べるために，マージしたときの標準出力が`Already up to date.`であるか比較します．

```yml:.github/workflows/merge.yml
# upstreamリポジトリからコンフリクトマーカーをつけたままマージしてプルリクエストを作成

name: Merge upstream, Create pull request

on:
  # Actions タブから手動でこのワークフローを実行することを許可する．
  workflow_dispatch:
  
env:
  BRANCH_PREFIX: auto_merge_
  UPSTREAM_URL: https://github.com/ユーザ名/リポジトリ名.git
  USER_EMAIL: git configに設定するemailアドレス
  USER_NAME: git configに設定するユーザ名
  MERGE_COMMAND: git merge upstream/main
  
jobs:
  auto_merge:

    runs-on: ubuntu-latest
    
    steps:
    
      # checkoutしないと，ワークフロー時の環境にこのリポジトリが存在しないことになる．
      - uses: actions/checkout@v3

      # (マージ)コミットするときに必要なgit configを設定する．
      - name: Set git config for merge commit
        run: |
          git config --global user.email "$USER_EMAIL"
          git config --global user.name "$USER_NAME"
      
      # [オプション] 現在時刻を環境変数に格納する．
      # ブランチ名を作るときや，プルリクエストのタイトルなどに使う．
      # フォーマットは %Y%m%d%H%M%S (例: 20220529032610) にしてから，
      # 部分文字列を使って各値を環境変数に格納しておく．
      - name: Get current time as env var
        env:
          TZ: 'Asia/Tokyo'
        run: |
          T=$(date +'%Y%m%d%H%M%S')
          echo "CURRENT_TIME=${T}" >> $GITHUB_ENV
          echo "YEAR=${T:0:4}" >> $GITHUB_ENV
          echo "MONTH=${T:4:2}" >> $GITHUB_ENV
          echo "DAY=${T:6:2}" >> $GITHUB_ENV
          echo "HOUR=${T:8:2}" >> $GITHUB_ENV
          echo "MIN=${T:10:2}" >> $GITHUB_ENV
          echo "SEC=${T:12:2}" >> $GITHUB_ENV

      # ブランチ名を作成して環境変数に格納する．
      # ブランチを作るときと，プルリクエストのheadに指定するときに使う．
      - name: Create branch name as env var
        run: |
          BRANCH_SUFFIX="${YEAR}_${MONTH}_${DAY}_${HOUR}_${MIN}_${SEC}"
          echo "BRANCH=${BRANCH_PREFIX}${BRANCH_SUFFIX}" >> $GITHUB_ENV
          
      # 先ほどの環境変数を使用してブランチを作成してチェックアウトする．
      # 環境変数は設定したrunの次のrunから使えるようになる．
      - name: Create and Checkout new branch
        run: git checkout -b ${{ env.BRANCH }}
       
      # エイリアス upstream にupstreamリポジトリのURLを設定する．
      - name: Set upstream repo
        run: git remote add upstream ${UPSTREAM_URL}
          
      # デフォルトでfetch-depthが1(shallow clone)らしく，マージするときに，
      # fatal: refusing to merge unrelated historiesが出るのを避けるために
      # --unshallow オプションをつけてフェッチする．      
      - name: Fetch upstream repo
        run: git fetch upstream --unshallow
      
      # upstreamをマージする．
      # github-scriptsを使って，終了コード，標準出力，標準エラー出力を変数に格納する．

      # ブランチが最新のとき，プッシュやプルリクエストは必要ない．
      # ブランチが最新のとき，標準出力が "Already up to date" になるため，
      # これを使って後ほどチェックする．

      # 標準出力しか使っていないので，github-scriptを使わなくても行けると思う．
      # actions/github-script を使わない場合は，
      # コンフリクト時にエラーが出ても止まらないように，
      # continue-on-error: true を使って，
      # 標準出力を環境変数かファイルに格納する．
      - name: Merge upstream
        uses: actions/github-script@v6
        env:
          COMMAND: ${{ env.MERGE_COMMAND }}
        with:
          script: |
            const { COMMAND } = process.env
            const result = await exec.getExecOutput(COMMAND, [], {
              ignoreReturnCode: true,
            })
            console.log(result)
            core.setOutput('EXIT_CODE', result.exitCode)
            core.setOutput('STDOUT', result.stdout)
            core.setOutput('STDERR', result.stderr)
        id: merge
      
      # コンフリクトのファイル名一覧をテキストファイルに格納する．

      # マージしたときにコンフリクトのあるファイルはコミットされない．
      # git ls-files -u でマージされていないコミットとファイルが表示される．
      # cut コマンドを使ってファイル名だけ抽出して，sort コマンドで重複を削除して並び替える．
      # その結果をテキストファイルに格納する．
      # テキストファイルはGitの対象にならないように親ディテクトリに保存する．
      
      - name: Save conflict files list in text file.
        run: git ls-files --unmerged | cut --fields=2 | sort --unique > ../merge_output.txt
          
      # コンフリクト数を環境変数に格納する．
      # 先程のテキストファイルの行数を wc コマンドで数えて格納する．
      # コンフリクトがない場合は 0 になる．
      - name: Count conflicts as env var
        run: echo "CONFLICTS_NUM=$(cat ../merge_output.txt | wc --lines)" >> $GITHUB_ENV
          
      # [オプション] コンフリクトファイル一覧にURLを付けたり，タイトルを付けたりして，プルリクエストの本文を作る．
      # コンフリクトがない場合とある場合で条件分けを行う．
      
      # コンフリクトがある場合は，ファイル名一覧をマークダウンの箇条書きとリンクに装飾する．
      # sed コマンドを使って，ファイル内容を書き換えていく．
      # --in-place オプションを使わないとファイル内容が変更されない．
      # @ を区切り文字に使うことで， URL のスラッシュをエスケープする必要がない．
      # 詳しくは sed コマンドのヘルプを参照する．
      - name: Create pull request body
        run: |
          if [[ $CONFLICTS_NUM == 0 ]]; then
            echo "# Successfully merged" > ../merge_output.txt
          else
            URL_PREFIX=${GITHUB_SERVER_URL}/${GITHUB_REPOSITORY}/tree/${BRANCH}/
            sed --in-place "s@\(.*\)@- [\1]($URL_PREFIX\1)@g" ../merge_output.txt
            sed --in-place '1i# Conflict files' ../merge_output.txt            
          fi
          
          echo 'MERGE_OUTPUT=$(cat ../merge_output.txt)' >> $GITHUB_ENV
      
      # コンフリクトマーカーごとコミットする．
      # コンフリクトが存在する場合にのみ実行する．
      - name: Commit merge with conflict marker
        if: ${{ env.CONFLICTS_NUM != 0 }}
        run: git commit -am "Merge upstream with conflict marker"
      
      # リモートリポジトリにプッシュして，プルリクエストを作成する．
      
      # マージ出力が Already up to date. (リポジトリが最新)であれば実行しない．

      # upstream を設定すると，プルリクエストのマージ先がデフォルトでupstreamになるため，
      # --repo オプションで明示的にリモートリポジトリを指定する．
      # 環境変数 $GITHUB_REPOSITORY を使うと良い．
      # --base オプションには，マージを行う対象ブランチを指定する．
      # 環境変数 $GITHUB_REF_NAME を使うと良い．

      # gh コマンドでプルリクエストを作成するためには，GH_TOKENが必要になる．

      - name: Push and Create pull request
        if: startsWith(steps.merge.outputs.STDOUT, 'Already up to date.') == false
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |          
          git push origin "$BRANCH"

          PR_BODY="${{ env.MERGE_OUTPUT }}"
          TIME="${YEAR}/${MONTH}/${DAY} ${HOUR}:${MIN}:${SEC}"
          PR_TITLE="Auto merge upstream (${TIME})"
          gh pr create --head ${{ env.BRANCH }} --base $GITHUB_REF_NAME --repo "$GITHUB_REPOSITORY" --title "${PR_TITLE}" --body "${PR_BODY}"
```

他にも改善できる部分はあると思いますが，とりあえずGitHub Actionsの勉強になったので満足です．

改善点
* ブランチ名に`upstream`のコミットハッシュを使ってみる
* 定期実行

# 参考

github-scripts でコマンドを実行する

https://github.com/gh640/command-result-action

github-scripts内に環境変数を持ってくる

https://github.com/actions/github-script

`${{ 式 }}` の使い方や，`startsWith`関数について

`startsWith` の代わりに `contains` 関数でも可能ですが， `==` だとうまくいきませんでした
(改行文字が原因?)

文字列リテラルはダブルクォーテーション`"`が使えないので注意が必要でした．

https://docs.github.com/ja/actions/learn-github-actions/expressions

デフォルトの環境変数

https://docs.github.com/en/actions/learn-github-actions/environment-variables

checkoutがshallowである

https://github.com/actions/checkout/issues/125

gh pr create のデフォルトが upstream である

https://github.com/cli/cli/issues/588

似たようなことをされています．

さらに schedule の cron を使って定期実行しています．

https://qiita.com/KEINOS/items/3bcaa6cea853f6b63475
