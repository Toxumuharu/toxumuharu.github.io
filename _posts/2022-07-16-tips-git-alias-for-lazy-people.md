---
title: "[小ネタ] git add から git push までを打つのが面倒なので今更 alias を作成する"
author_name: "Toxumuharu"
tags:
    - git
    - shellscript
    - zshrc
    - bashrc
---

<div align="center">
<img src="https://img.icons8.com/color/144/000000/git.png"/>
</div>
<br>

# このドキュメントの内容
git にてソースコードを管理している際、開発段階などにおいて CI パイプラインを組んでおり、リモートリポジトリに高頻度で push を行い変更を反映させたい時があります。

そんな時にわざわざ git add から git push までを丁寧に打つのが面倒だという私は、zshrc にて `gitauto` という自動化のコマンドを今更 alias を用いて作成してみました。

# alias の中身
```
function gitauto() {
    git add .;
    COMMIT_MESSAGE=`git status | grep -e "modified" -e "new file" -e "deleted"`;
    git commit -m $COMMIT_MESSAGE;
    git push origin main; 
}
alias gitauto=gitauto
```

次の動作を自動で行う alias です。
1. `git add .` を行いワーク ディレクトリ全体をステージング
2. `git status` の内容のうち、"modified" や "new file" を含む内容を `COMMIT_MESSAGE` へ格納
3. `git commit` のコミット メッセージに `COMMIT_MESSAGE` を指定
4. `git push` で `main` ブランチに push

あれこれと書いていたものが、下記のように `gitauto` とだけ書けばワーク ディレクトリ以下の変更を丸ごと push してくれるので重宝しています。

また、`gitauto` コマンドの直後に `git log` を打つときちんと変更されたファイル名が見れるようになってます。

<br>

![2022-07-16-tips-git-alias-for-lazy-people](/media/20220716/1.png)

<br>

個人向けの開発用 alias ですので、運用ブランチやチーム開発の場ではご利用にならないようお気をつけ下さいませ。ご利用はくれぐれも自己責任で！

<br>
<br>

本投稿が何かの役に立てば幸いです。

Toxumuharu

<br>
<br>

<a target="_blank" href="https://icons8.com/icon/20906/git">Git icon by Icons8</a>