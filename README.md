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
テーブル|説明
--------|--------
user|ユーザーアカウント
url|URL投稿
comment|URL投稿に対するコメント
tag|URL投稿に対するタグ付け
follow|フォロー関係

## user
コラム|データ型|属性|その他|説明
--------|--------|--------|--------|--------
username|varchar(32)|||ユーザー名
nickname|varchar(32)|||名前
introducion|varchar(256)|||自己紹介
password|varchar(32)|||パスワード
mail|varchar(256)|||メールアドレス
fcn||||公式ファンクラブの会員番号

## url
コラム|データ型|属性|その他|説明
--------|--------|--------|--------|--------
username|varchar(32)|||投稿者のユーザー名
url|varchar(4096)|||URL
time|int(10)|UNSIGNED||日時 ( UNIXタイムスタンプ )
id|int(128)|UNSIGNED|AUTO_INCREMENT|URL投稿を管理するユニークなID

## comment
コラム|データ型|属性|その他|説明
--------|--------|--------|--------|--------
username|varchar(32)|||コメントするユーザー名
comment|varchar(128)|||コメント
id|int(128)|||URL投稿ID

## tag
コラム|データ型|属性|その他|説明
--------|--------|--------|--------|--------
username|varchar(32)|||タグ付けするユーザー名
tag|varchar(32)|||タグ
id|int(128)|||URL投稿ID

## follow
コラム|データ型|属性|その他|説明
--------|--------|--------|--------|--------
follower|varchar(32)|||フォローする側のユーザー名
followee|varchar(32)|||フォローされる側のユーザー名

# API設計

以下のAPIは http://b-world.org/nounen/api (将来的にはSSLを導入し https://b-world.org/nounen/api )以下にあるものとする  
create/user/#id/#email/#password以外のAPIはセッションなどでコネクションが張られていない場合は自動的に認証画面に飛ばすようになるはず  
create/user/#id/#email/#password以外のAPIはセッションから勝手にユーザIDを引っ張って来る  
Twitter連携やFacebook連携やTumblr連携は後回し、後回し  
セッションがつながっておらず認証済みだと確認できなかった場合、403 Forbiddenを返し、BODYには"not found user"を格納する  
GET系のAPIで存在しない各種IDを指定したなどの場合は404 Not Foundを返す  
POST系のAPIでサーバ内部でエラーが発生した場合、500 Internal Server Errorを返す  

* API
 * SUMMARY
 * METHOD
 * RESPONS
  * STATUS
  * BODY
* create/#id/#email/#password
 * メールアドレスがemailのIDがidでパスワードがpasswordなユーザを仮作成し、emailに認証用の自動生成したURLを送信する
 * POST
 * respons
  * 200 OK
  * 空
* auth/#authid/
 * メールに自動送信する認証用URL　再度idとemailとpasswordを入力する画面を表示する
 * GET
 * respons
  * 200 OK
  * idとemailとpasswordを入力する画面
* auth/#authid/#email/#password
 * メールに自動送信した認証用URLに対してメールアドレスとパスワードを付与し、認証を行う　これでパスワードが一致した場合IDがidなユーザが作成される
 * POST
 * respons
  * 201 Created
  * [トップ](http://b-world.org/nounen)へ飛ばす
* email/#email
 * メールアドレスをemailに仮変更し、auth/user/#id/#authid/を元のメールアドレスに送信し認証する
 * PUT
 * respons
  * 200 OK
  * 空
* password/#password
 * パスワードをpasswordに仮変更し、auth/user/#id/#authid/をメールアドレスに送信し認証する
 * PUT
 * respons
  * 200 OK
  * 空
* name/#name
 * ユーザ名をnameにする
 * PUT
 * respons
  * 201 Created
  * ユーザ管理画面([http://b-world.org/nounen/user](http://b-world.org/nounen/user)以下の予定)へ飛ばす
* info/#info
 * ユーザの自己紹介欄をinfoにする(URLの長さなどの制約の関係で将来的にこれはPOSTを利用したAPIになるかもしれない)
 * PUT
 * respons
  * 201 Created
  * ユーザ管理画面([http://b-world.org/nounen/user](http://b-world.org/nounen/user)以下の予定)へ飛ばす
* fcn/#FCN
 * ユーザの公式ファンクラブの会員番号をFCNにする
 * PUT
 * respons
  * 201 Created
  * ユーザ管理画面([http://b-world.org/nounen/user](http://b-world.org/nounen/user)以下の予定)へ飛ばす
* name
 * 自分のユーザ名を取得する
 * GET
 * respons
  * 200 OK
  * {"name":name}形式のJSONデータ
* name/#id
 * idのユーザ名を取得する
 * GET
 * respons
  * 200 OK
  * {"name":name}形式のJSONデータ
* email
 * 自分のメールアドレスを取得する
 * GET
 * respons
  * 200 OK
  * {"email":email}形式のJSONデータ
* info
 * 自分の自己紹介を取得する
 * GET
 * respons
  * 200 OK
  * {"info":info}形式のJSONデータ
* info/#id
 * idの自己紹介を取得する
 * GET
 * respons
  * 200 OK
  * {"info":info}形式のJSONデータ
* fcn
 * 自分の公式ファンクラブの会員番号をFCNにする
 * GET
 * respons
  * 200 OK
  * {"fcn":fcn}形式のJSONデータ
* fcn/#id
 * idの公式ファンクラブIDを取得する
 * GET
 * respons
  * 200 OK
  * {"fcn":fcn}形式のJSONデータ
* url/#urlid
 * urlidのurlを取得する
 * GET
 * respons
  * 200 OK
  * {"urlid":urlid}形式のJSONデータ
* urlid/#url
 * urlのurlidを取得する
 * GET
 * respons
  * 200 OK
  * {"url":url}形式のJSONデータ
* post/#id/#url
 * idのユーザがurlをポストしているかどうかを取得する
 * GET
 * respons
  * 200 OK
  * 空
* post/#url
 * 自分がurlをポストしたかどうかを取得する
 * GET
 * respons
  * 200 OK
  * 空
* comment/#commentid
 * commentidのcommentのURLを取得する
 * GET
 * respons
  * 201 Created
  * commentidのコメントへ飛ばす
* posts/follow/#time/#time
 * フォローしているユーザが最初のtimeから2番目のtimeまでの間に投稿されたURL群を取得する
 * GET
 * respons
  * 200 OK
  * [urlid...]形式のJSONデータ
* posts/all/#time/#time
 * 最初のtimeから2番目のtimeまでの間に投稿されたURL群を取得する
 * GET
 * respons
  * 200 OK
  * [urlid...]形式のJSONデータ
* tag/follow/#tag/#time/#time
 * フォローしている人が投稿したtagの付与された最初のtimeから2番目のtimeまでの間に投稿されたURL群を取得する
 * GET
 * respons
  * 200 OK
  * [urlid...]形式のJSONデータ
* tag/all/#tag/#time/#time
 * すべての投稿からtagの付与された最初のtimeから2番目のtimeまでの間に投稿されたURL群を取得する
 * GET
 * respons
  * 200 OK
  * [urlid...]形式のJSONデータ
* follow
 * フォローリストを取得する
 * GET
 * respons
  * 200 OK
  * {"follow":[id...]}形式のJSONデータ
* follow/#id
 * idのユーザのフォローリストを取得する
 * GET
 * respons
  * 200 OK
  * {"follow":[id...]}形式のJSONデータ
* follower
 * フォロワーリストを取得する
 * GET
 * respons
  * 200 OK
  * {"follower":[id...]}形式のJSONデータ
* follower/#id
 * idのユーザのフォロワーリストを取得する
 * GET
 * respons
  * 200 OK
  * {"follow":[id...]}形式のJSONデータ
* followerq/#id
 * idのユーザからフォローされているかどうかを取得する
 * GET
 * respons
  * 200 OK
  * 空
* followq/#id/#id
 * 最初のidのユーザが2番目のidのユーザをフォローしているかどうかを取得する
 * GET
 * respons
  * 200 OK
  * 空
* url/#url
 * urlを投稿する
 * POST
 * respons
  * 200 OK
  * 空
* comment/#urlid/#comment
 * urlidの表すurlに対しコメントを付与する
 * POST
 * respons
  * 200 OK
  * 空
* tag/#urlid/#tag
 * urlidの表すurlに対しタグを付与する
 * POST
 * respons
  * 200 OK
  * 空
* follow/#id
 * idの表すユーザをフォローする
 * PUT
 * respons
  * 200 OK
  * 空
* unfollow/#id
 * idの表すユーザをアンフォローする
 * PUT
 * respons
  * 200 OK
  * 空
