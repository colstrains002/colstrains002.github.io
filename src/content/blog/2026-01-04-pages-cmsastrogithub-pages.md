---
title: Pagesログ 2：テンプレ改造
description: "Description: Astroテンプレを自分用の構成に調整する（2026/01/03）"
pubDate: 2026-01-04
draft: false
---
AstroでGitHub Pagesに載せているブログを、できるだけnoteみたいな感覚に寄せたいと思っていた。ブラウザで記事を書いて、そのまま保存して、公開まで持っていく。裏側でGitが動くのは分かっているけど、日常の手触りとして「ファイルを編集してコミットする」を前面に出したくない、という話。

今回やりたかった条件はこのあたり。

*   記事作成はブラウザ内で完結（github.dev / Codespaces も使えるが、できればCMSのUI）
    
*   draft運用は維持（直リンクはOK、一覧とRSSには出さない）
    
*   既存のサイト整備の状態は崩さない（ここが一番重要）
    

試した結果、Pages CMSでかなり近づけられそうだった。やることは大きく2つで、repo側に `.pages.yml` を置いて、Pages CMS側でGitHub Appをそのrepoだけに入れる。手順としてはそれだけで、投稿画面として使えるUIが出てくる。

やったことのメモ。

1.  repo側に .pages.yml を追加  
    ローカルで `.pages.yml` を作ってmainに入れた（作成自体はZedでもgithub.devでも良い）。中身は最小で、mediaの保存先と、blogコレクション（`src/content/blog`）と、frontmatterの項目だけに絞った。

```yml
media:
  input: public/media
  output: /media

content:
  - name: blog
    label: Blog
    type: collection
    path: src/content/blog
    format: yaml-frontmatter
    filename: "{year}-{month}-{day}-{primary}.md"
    fields:
      - name: title
        label: Title
        type: string
        required: true
      - name: description
        label: Description
        type: text
      - name: pubDate
        label: PubDate
        type: date
        required: true
      - name: draft
        label: Draft
        type: boolean
        default: true
      - name: body
        label: Body
        type: rich-text
```

この段階ではサイトの挙動は特に変わらないはずで、実際、見た目の崩れなどは起きなかった（ファイルを追加しただけなので当然かもしれない）。まず壊れにくいところから入れる、という意味でも気が楽だった。

2.  Pages CMS側でGitHub連携  
    ここが初回だけ少し緊張した。GitHub Appのインストール画面で、対象リポジトリを選ぶところがある。Only select repositoriesにして、`colstrains002/colstrains002.github.io` だけを指定した。関係ないrepoまで権限を渡したくないので、ここは慎重にやる。
    
3.  Pages CMSでBlogが見えるか確認  
    Openしてみると「Blog」が出た。つまり `.pages.yml` が読めている状態になった、ということだと思う。ここまで来ると、あとは投稿UIとして使える。
    
4.  テスト記事を作って、運用条件を確認  
    まずは下書きで1本作った。GitHub側には `src/content/blog/2026-01-04-cms.md` が増えていて、frontmatterも狙いどおりの形だった。
    

```md
---
title: CMSテスト_下書き
description: Pages CMSから作成した下書きテスト
pubDate: 2026-01-04
draft: true
---
本文テスト。
```

ここから、下書きの扱いが成立するかを確認した。

*   直リンクは表示できた
    
*   一覧（/blog/）には出ていない
    
*   RSS（/rss.xml）にも出ていない
    

この3つが揃ったので、「draftは直リンクだけOK」を続けられそうだ、と判断できた。

ただし、ひとつだけ最初に引っかかった点がある。URLのslugは `cms` ではなかった。ファイル名全体がslugになるので、正しいURLは `/blog/2026-01-04-cms/`。最初に `/blog/cms/` を開いて404になって、そこで「slugの規則はこうだ」と分かった。ここは今後の運用でも迷いやすいので、覚えておく。

5.  公開切替も確認  
    最後に draft をオフにして保存してみた。すると一覧にもRSSにも出るようになった。操作としては「下書き→公開」の切り替えで完結するので、体感としてはかなりnoteに近い。

ここまでで、ブラウザ完結の投稿導線はだいぶ固まったと思う。普段はPages CMSで記事を書いて保存、公開するときだけdraftを切り替える。裏側はコミットだが、日々の操作としては「投稿画面」が中心になる。

残りで気になるのは画像まわり。本文（rich-text）で画像をアップして、`public/media` に置かれて、表示やパス（/media/〜）が破綻しないか。ここを一回だけ確認しておけば、詰まりやすいところは先に潰せそう。

今日はここまで。次は画像を1枚入れて、表示が崩れないかだけ見ておく。