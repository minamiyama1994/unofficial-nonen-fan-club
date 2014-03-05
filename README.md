unofficial-nonen-fan-club
=========================

[非公式能年玲奈ちゃんファンクラブ](http://b-world.org/nounen)について色々会議


# ファンクラブの概要

* 基本的にはSNS形式
* ユーザ間にはフォロー/フォロワー関係が成立する
* ユーザはファンクラブに対しURLを投稿できる(と言うより「URLだけ」投稿できる)
* 投稿されたURLにはタグやコメントを自由に付与できる
* ファンクラブはAPIを提供し、公式サイトや関連アプリはAPIを経由してデータを取得したりアカウント作成を行ったりする
 * APIはおそらくは[http://b-world.org/nounen/api](http://b-world.org/nounen/api)以下になる

# DB設計

* ユーザーアカウント
 * ユーザー名
 * 名前
 * 自己紹介
 * パスワード
 * メールアドレス
 * 公式ファンクラブの会員番号
* URL投稿
 * 投稿者のユーザー名
 * URL
 * 日時 ( UNIXタイムスタンプ )
 * URL投稿を管理するユニークなID
* URL投稿に対するコメント
 * コメントするユーザー名
 * コメント
 * URL投稿ID
* URL投稿に対するタグ付け
 * タグ付けするユーザー名
 * タグ
 * URL投稿ID
* フォロー関係
 * フォローする側のユーザー名
 * フォローされる側のユーザー名

# API設計

以下のAPIは http://b-world.org/nounen/api (将来的にはSSLを導入し https://b-world.org/nounen/api )以下にあるものとする  
create/user/#id/#email/#password以外のAPIはセッションなどでコネクションが張られていない場合は自動的に認証画面に飛ばすようになるはず  
create/user/#id/#email/#password以外のAPIはセッションから勝手にユーザIDを引っ張ってくる  
Twitter連携やFacebook連携やTumblr連携は後回し、後回し

* create/#id/#email/#password
 * メールアドレスがemailのIDがnameでパスワードがpasswordなユーザを仮作成し、emailに認証用の自動生成したURLを送信する
* auth/#authid/
 * メールに自動送信する認証用URL　再度idとemailとpasswordを入力する画面を表示する
* auth/#authid/#email/#password
 * メールに自動送信した認証用URLに対してメールアドレスとパスワードを付与し、認証を行う　これでパスワードが一致した場合IDがnameなユーザが作成される
* set/name/#name
 * ユーザ名をnameにする
* set/email/#email
 * メールアドレスをemailに仮変更し、auth/user/#id/#authid/を元のメールアドレスに送信し認証する
* set/password/#password
 * パスワードをpasswordに仮変更し、auth/user/#id/#authid/をメールアドレスに送信し認証する
* set/info/#info
 * ユーザの自己紹介欄をinfoにする(URLの長さなどの制約の関係で将来的にこれはPOSTを利用したAPIになるかもしれない)
* set/fcn/#FCN
 * ユーザの公式ファンクラブの会員番号をFCNにする
* get/name
 * 自分のユーザ名を取得する
* get/name/#id
 * idのユーザ名を取得する
* get/email
 * 自分のメールアドレスを取得する
* get/info
 * 自分の自己紹介を取得する
* get/info/#id
 * idの自己紹介を取得する
* get/fcn
 * 自分の公式ファンクラブの会員番号をFCNにする
* get/fcn/#id
 * idの公式ファンクラブIDを取得する
* get/url/#urlid
 * urlidのurlを取得する
* get/urlid/#url
 * urlのurlidを取得する
* get/post/#id/#url
 * idのユーザがurlをポストしたかどうかを取得する
* get/post/#url
 * 自分がurlをポストしたかどうかを取得する
* get/comment/#commentid
 * commentidのcommentのURLを取得する
* get/posts/#time/#time
 * 最初のtimeから2番目のtimeまでの間に投稿されたURL群を取得する
* get/tag/#tag/#time/#time
 * tagの付与された最初のtimeから2番目のtimeまでの間に投稿されたURL群を取得する
* get/follow
 * フォローリストを取得する
* get/follow/#id
 * idのユーザのフォローリストを取得する
* get/follower
 * フォロワーリストを取得する
* get/follower/#id
 * idのユーザのフォロワーリストを取得する
* get/followerq/#id
 * idのユーザからフォローされているかどうかを取得する
* get/followq/#id/#id
 * 最初のidのユーザが2番目のidのユーザをフォローしているかどうかを取得する
* post/url/#url
 * urlを投稿する
* post/comment/#urlid/#comment
 * urlidの表すurlに対しコメントを付与する
* post/tag/#urlid/#tag
 * urlidの表すurlに対しタグを付与する
* follow/#id
 * idの表すユーザをフォローする
* unfollow/#id
 * idの表すユーザをアンフォローする