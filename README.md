#WP REST APIをいろいろ触ってみるハンズオン@WordBench東京のメモ書き
##お約束
- JavaScriptはすぐに壊れるものと考える
- 動かなくなっても慌てない
- まずは「全角スペース」「行末のセミコロン（;）」「カッコの閉じ忘れ」を疑う
- 回線が混雑している場合もあるので焦らない
- APIクエリはWP REST API Consoleやcurl + jqで事前にチェック
- 質はともかく完成品がある（＝家でも復習可能）ので事故っても泣かない


##アジェンダ
- WP REST APIのセットアップ
- APIコンソールを使ってリクエストを投げてみる
- jQueryでWP REST APIのデータを処理してみる
- 「もっと手の込んだことやらせろ！」という人への<s>嫌がらせ</s>お題
 
##まずはWP REST APIをセットアップ
###似たプラグインがあるので要注意
|正否|URL|プラグイン名|
|:--|:--|:--|
|正解|https://ja.wordpress.org/plugins/rest-api/|WordPress REST API (Version 2)|
|バージョン違い|https://ja.wordpress.org/plugins/json-rest-api/|WP REST API (WP API)|
|別物|https://ja.wordpress.org/plugins/wp-api/|WP API|

- WP REST API (WP API)が有効化されていると「WordPress REST API (Version 2)」は動作しません
- WP APIは２年以上更新されてないので使わないこと

##APIコンソールを使ってリクエストを投げてみる
zipファイルURL:https://github.com/hideokamoto/rest-api-console/archive/master.zip
*アップデートしないので、使い続ける場合は[本家リポジトリ](https://github.com/WP-API/rest-api-console)か[公式ディレクトリ](https://wordpress.org/plugins/rest-api-console/)のを修正して使うことをお勧めします。

WP REST APIプラグインの入れたサイトにインストールして有効化すれば使えます。

GET以外は認証が必要なものが多いですので要注意です。

###よく使いそうなAPIクエリ早見表(posts)
|Query|Description|
|:--|:--|:--|
|search=XXX|XXXというキーワードを含む記事|
|per_page=10|最大10記事取得|
|page=5|5ページ目を表示|
|orderby=id|id順に整列|
|filter[cat]=category|[]の中にWP_Queryのパラメータが使える。<br/>（例：categoryというスラッグの記事を取得）|
|_embed|コメント・著者・サムネイル・カテゴリ・タグなどのデータも取得する|

- [APIリファレンス（日本語）](http://ja.wp-api.org/reference/)
- [API Reference(English)](http://v2.wp-api.org/reference/)

##jQueryでWP REST APIのデータを処理してみる
[ベースにするページはこちら（CodePen）](http://codepen.io/hideokamoto/pen/VapPaq?editors=1010 )
「Fork」をクリックして自分用のコピーを作ってから作業開始。
###jQueryでAjax通信するテンプレート
Ajaxを使ったことがないという方向けに、とりあえずのコードサンプルです。

```
var url = 'http://example.com/wp/v2/';//ここにAPIのURLを書きましょう
$.ajax({
  url: url,
  type:'GET', //POSTの時はここを'POST'にします
  dataType: 'json',
  data : '', // POSTの時に使います
  timeout:10000,
}).done(function(data) {
	/**
	 * dataという変数に取得したデータが入っているので、
	 * この中で取得したデータを処理する
	 **/
	 
  /**
   * 複数件のデータを取得した場合はforなどでループ処理する
   *
   *for (var i = data.length - 1; i >= 0; i--) {
   *  この中に書いたコードが１データ毎に実行されます
   *}
   **/
}).fail(function(data) {
   console.log(data);
  /**
   * ここにAPI通信時のエラー（404や500エラーなど）処理を書きます。
   * とりあえずconsole.logなどでエラーログを確認できるようにしておけばいいと思います。
   **/
   
});
```
ここのコード内容にマサカリ投げれる人は、そもそも自力でコードを書けばいいと思います。

###１：ブログカードを表示する
「oembed」のAPIにGETリクエストを投げます。
###oEmbedとは？
WordPressのコアに組み込まれている数少ないREST API。
記事URLを貼り付けるとブログカードが表示されるoEmbed機能を動かすためのAPI

###アドレスの作り方
http://example.com/oembed/1.0/embed?http://example.com/embed-blog-posts
embed?以降に埋め込みたい記事のURLを入れる

##戻り値サンプル
```json
{
  "version": "1.0",
  "provider_name": "WP-kyoto",
  "provider_url": "http://wp-kyoto.net",
  "author_name": "Okamoto Hidetaka",
  "author_url": "http://wp-kyoto.net/author/sitemaster_motchi0214/",
  "title": "WordPress4.4コアのWP REST APIを叩いてみる（oEmbed編）",
  "type": "rich",
  "width": 600,
  "height": 338,
  "html": "<blockquote class=\"wp-embedded-content\"><a href=\"http://wp-kyoto.net/wp4-4-call-oembed-api/\">WordPress4.4コアのWP REST APIを叩いてみる（oEmbed編）</a></blockquote>\n<script type='text/javascript'>\n<!--//--><![CDATA[//><!--\n\t\t!function(a,b){\"use strict\";function c(){if(!e){e=!0;var a,c,d,f,g=-1!==navigator.appVersion.indexOf(\"MSIE 10\"),h=!!navigator.userAgent.match(/Trident.*rv:11\\./),i=b.querySelectorAll(\"iframe.wp-embedded-content\"),j=b.querySelectorAll(\"blockquote.wp-embedded-content\");for(c=0;c<j.length;c++)j[c].style.display=\"none\";for(c=0;c<i.length;c++)if(d=i[c],d.style.display=\"\",!d.getAttribute(\"data-secret\")){if(f=Math.random().toString(36).substr(2,10),d.src+=\"#?secret=\"+f,d.setAttribute(\"data-secret\",f),g||h)a=d.cloneNode(!0),a.removeAttribute(\"security\"),d.parentNode.replaceChild(a,d)}else;}}var d=!1,e=!1;if(b.querySelector)if(a.addEventListener)d=!0;if(a.wp=a.wp||{},!a.wp.receiveEmbedMessage)if(a.wp.receiveEmbedMessage=function(c){var d=c.data;if(d.secret||d.message||d.value)if(!/[^a-zA-Z0-9]/.test(d.secret)){var e,f,g,h,i,j=b.querySelectorAll('iframe[data-secret=\"'+d.secret+'\"]'),k=b.querySelectorAll('blockquote[data-secret=\"'+d.secret+'\"]');for(e=0;e<k.length;e++)k[e].style.display=\"none\";for(e=0;e<j.length;e++)if(f=j[e],c.source===f.contentWindow){if(f.style.display=\"\",\"height\"===d.message){if(g=parseInt(d.value,10),g>1e3)g=1e3;else if(200>~~g)g=200;f.height=g}if(\"link\"===d.message)if(h=b.createElement(\"a\"),i=b.createElement(\"a\"),h.href=f.getAttribute(\"src\"),i.href=d.value,i.host===h.host)if(b.activeElement===f)a.top.location.href=d.value}else;}},d)a.addEventListener(\"message\",a.wp.receiveEmbedMessage,!1),b.addEventListener(\"DOMContentLoaded\",c,!1),a.addEventListener(\"load\",c,!1)}(window,document);\n//--><!]]>\n</script><iframe sandbox=\"allow-scripts\" security=\"restricted\" src=\"http://wp-kyoto.net/wp4-4-call-oembed-api/embed/\" width=\"600\" height=\"338\" title=\"埋め込まれた WordPress の投稿\" frameborder=\"0\" marginwidth=\"0\" marginheight=\"0\" scrolling=\"no\" class=\"wp-embedded-content\"></iframe>"
}
```


※発展系：http://codepen.io/hideokamoto/pen/QyydBB

###２：ギャラリーを作る
mediaのAPIに対してAjaxでGETリクエストを投げて、取得したデータを表示させます。

###アドレスの作り方
http://example.com/wp/v2/media

##戻り値サンプル
```json
[
  {
    "id": 1692,
    "date": "2014-01-05T11:45:36",
    "date_gmt": "2014-01-05T18:45:36",
    "guid": {
      "rendered": "http://rest-api.dev/wp-content/uploads/2014/01/spectacles-1.gif"
    },
    "modified": "2014-01-05T11:45:36",
    "modified_gmt": "2014-01-05T18:45:36",
    "slug": "spectacles-2",
    "type": "attachment",
    "link": "http://rest-api.dev/about/clearing-floats/spectacles-2/",
    "title": {
      "rendered": "spectacles"
    },
    "author": 5,
    "comment_status": "open",
    "ping_status": "closed",
    "alt_text": "",
    "caption": "",
    "description": "",
    "media_type": "image",
    "mime_type": "image/gif",
    "media_details": {
      "width": 165,
      "height": 210,
      "file": "2014/01/spectacles-1.gif",
      "sizes": {
        "thumbnail": {
          "file": "spectacles-1-150x150.gif",
          "width": 150,
          "height": 150,
          "mime_type": "image/gif",
          "source_url": "http://rest-api.dev/wp-content/uploads/2014/01/spectacles-1-150x150.gif"
        },
        "full": {
          "file": "spectacles-1.gif",
          "width": 165,
          "height": 210,
          "mime_type": "image/gif",
          "source_url": "http://rest-api.dev/wp-content/uploads/2014/01/spectacles-1.gif"
        }
      },
      "image_meta": {
        "aperture": 0,
        "credit": "",
        "camera": "",
        "caption": "",
        "created_timestamp": 0,
        "copyright": "",
        "focal_length": 0,
        "iso": 0,
        "shutter_speed": 0,
        "title": "",
        "orientation": 0,
        "keywords": []
      }
    },
    "post": 501,
    "source_url": "http://rest-api.dev/wp-content/uploads/2014/01/spectacles-1.gif",
    "_links": {
      "self": [
        {
          "href": "http://rest-api.dev/wp-json/wp/v2/media/1692"
        }
      ],
      "collection": [
        {
          "href": "http://rest-api.dev/wp-json/wp/v2/media"
        }
      ],
      "about": [
        {
          "href": "http://rest-api.dev/wp-json/wp/v2/types/attachment"
        }
      ],
      "author": [
        {
          "embeddable": true,
          "href": "http://rest-api.dev/wp-json/wp/v2/users/5"
        }
      ],
      "replies": [
        {
          "embeddable": true,
          "href": "http://rest-api.dev/wp-json/wp/v2/comments?post=1692"
        }
      ]
    }
  }
]
```

*完成サンプル：http://codepen.io/hideokamoto/pen/ZWeLBJ?editors=1010


###３：コメントフォームを作って投稿する

「http://example.com/wp/v2/comments 」に対してPOSTリクエストを送る

[WP REST API + Ajaxでコメントフォームを作る](http://wp-kyoto.net/wp-rest-api-with-ajax-comment-form/)という記事があるので、スクリプトの書き方などはそこを見ながらやってください。

*完成サンプル：http://codepen.io/hideokamoto/pen/JXWErm?editors=1010
####発展系：[投稿一覧をselectで選べるようにしたバージョン](http://codepen.io/hideokamoto/pen/ZWeLrv)
**やってること**

- postsのAPIから記事を取得
- コメント欄が解放されている記事（comment_status == 'open'）かを判定
- コメントできる記事だけselectタグにoptionとして追加

##「物足りねぇぞ！」という方はこちら
https://github.com/hideokamoto/rest-api-handson/blob/master/advanced.md
