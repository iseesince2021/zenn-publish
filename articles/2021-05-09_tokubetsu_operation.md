---
title: "特別定額給付金の業務オペレーションを調べてみたら思ったよりも〇〇だった" # 記事のタイトル
emoji: "" # アイキャッチとして使われる絵文字（1文字だけ）
type: "idea" # tech: 技術記事 / idea: アイデア記事
topics: ["covid19"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

# 特別定額給付金の業務オペレーションを調べてみたら思ったよりも〇〇だった


特別定額給付金の給付、病院の重症者病床の確保、ワクチンの摂取計画など、コロナ禍によって急遽発生した業務オペレーションがいろんな面でうまくいっていない、という話をよく耳にします。「なんでこんなにうまくいってないの？○○ではこんな風になっているらしいのに」という感想を持つ人も多いと思います。筆者もその1人です。

本稿は「〇〇ではこんな風になっているらしい」という話から当該事例がどういう仕組みになっているのか、ということをご紹介するとともに、日本において実現可能かどうか？を記載したいと思います。

予めお断りすると、筆者自身、検討していてベストプラクティスと、日本の置かれている状況のギャップにかなり厳しさを感じましたが、本サイトの趣旨である「マニアックな雑談」として、お許しいただきたいと思います。

手っ取り早く結論から書いてしまうと、下記のとおりとなります。

- ベストプラクティスは業務オペレーションを担う組織が国全体で1つに集約されている
- 特別定額給付金では業務オペレーションが約1700個の市区町村と国に分断されている
- コンウェイの法則に見られるように、人もITも組織を超えられない
- 今回の課題の解消には地方自治体と国の役割分担の考え方も見直しが必要


## 日本の特別定額給付金の場合

そもそも、国が個人にお金を配るには、以下の内容をなんらかの方法で明らかにしなければなりません。

- 渡す相手は誰か？・・・課題1
- どのようにお金を渡すか？・・・課題2

今回、日本の特別定額給付金は基準日時点で住民基本台帳に載っている人1人当たり10万円と定めました。このデータを持っているのは市区町村になります。これでまず課題1はクリアです。

他方、政府は日銀を通じて [^1]、市区町村は（都道府県の）指定金融機関にあたる金融機関を通じて公金の出納処理を行うことがそれぞれ定められています [^2] [^3]。また銀行振り込みのほうが現金を取り扱うより、大量の出納処理を正確に安価に安全に素早く行うことができ、合理的です。そうすると課題2は次のように形を変えます。

- 渡す相手は誰か？・・・住民基本台帳の全員
- ~~どのようにお金を渡すか？・・・課題2~~
- 渡す相手の銀行口座はどこか？・・・課題3
- 子供など銀行口座を持っていない人にどう渡すか？・・・課題4

今回は住民基本台帳における世帯主で取りまとめることにしました（家族に1人は銀行口座をもっているだろう、ということでしょうか）。その結果、課題1の解決策は「全員」から「世帯主」に、課題4は課題5に形を変えます。

- 渡す相手は誰か？・・・住民基本台帳の~~全員~~世帯主
- 渡す相手の銀行口座はどこか？・・・課題3
- ~~子供など銀行口座を持っていない人にどう渡すか？・・・課題4~~
- 相手にいくら渡せばよいのか？・・・課題5

しかしながら以下の2つの課題を解消する情報は直接対象者に確認して情報を集めなければなりません。

- 渡す相手の銀行口座はどこか？・・・課題3
- 相手にいくら渡せばよいのか？・・・課題5

そしてなぜか全国で同じ作業をするはずのこの作業を国は市区町村にゆだねてしまいました。実際に市区町村でどのような業務オペレーションが必要になるか、ということは東修平（四條畷市長）さんの[「なぜ10万円給付に時間がかかるのか」](https://note.com/politic/n/n445ebc670a87)という記事をご覧いただくと良いと思います。全国約1700ある市区町村で全部バラバラにこれを実行するって、普通に考えて非効率ですよね・・・。

[^1]: [”国庫金事務の電子化とは何ですか？”、日本銀行](https://www.boj.or.jp/announcements/education/oshiete/op/f03.htm)（参照 2021-03-25）
[^2]: [”第２章 会計管理局の事務事業”、東京都会計管理局](https://www.kaikeikanri.metro.tokyo.lg.jp/02jigyougaiyou2.pdf)（参照 2021-03-25）
[^3]: ["久万高原町公金出納事務取扱要領"](https://www.kumakogen.jp/reiki/reiki_honbun/r032RG00000608.html)（参照 2021-03-25）



## イギリスは政府が一括して業務オペレーションを行っていた

それではベストプラクティス（？）のイギリスはどうだったのでしょうか。日経電子版には下記のような記載がありました。

> コロナ禍の支援策をみると、海外は一歩先を進む。英国は納税状況などを加味し、低所得者に給付金を自動的に振り込んだ。
https://www.nikkei.com/article/DGXZQODF253RJ0V20C21A2000000/


ここからGoogle先生に教えをこうたところ、以下のことが分かりました。

- イギリスは生活保護に相当する既存の仕組みに上乗せした（そもそも全員給付ではない） [^4] [^6]
- イギリスは税金と社会保険に関わる業務オペレーションが歳入関税庁（HMRC）に一元化されている（厳密には統合中） [^4] [^5]
- イギリスの生活保護の仕組みは歳入関税庁が業務オペレーションを行っている [^5]
- イギリスでも新設された自営業者向け給付金は申請が必要だが、税務情報をもとに歳入関税庁が業務オペレーションを実施する [^7]

給付の制度は違いますが、生活保護にあたる仕組みにしぼって考えても、日本では市区町村が行うのに対し、イギリスでは国が直接行っているところに、業務オペレーションの違いがありそうです。

[^4]: ["新型ウイルス経済支援、最も手厚い国はどこ？"、BBC](https://www.bbc.com/japanese/features-and-analysis-52586299)（参照 2021-03-25）
[^5]: [”勤労者タックスクレジット”、Wikipedia](https://ja.wikipedia.org/wiki/%E5%8B%A4%E5%8A%B4%E8%80%85%E3%82%BF%E3%83%83%E3%82%AF%E3%82%B9%E3%82%AF%E3%83%AC%E3%82%B8%E3%83%83%E3%83%88)（参照 2021-03-25）
[^6]: [”コロナウイルス危機下の生活支援：セーフティネットの不足を補う”、OECD](http://www.oecd.org/coronavirus/policy-responses/supporting-livelihoods-during-the-covid-19-crisis-closing-the-gaps-in-safety-nets-264771d3/#section-d1e1442)（参照 2021-03-25）
[^7]: [ケヴィン・ピーチー、”イギリスの自営業者に給付金、その内容は？　新型ウイルス”、BBC](https://www.bbc.com/japanese/52059296)（参照 2021-03-25）


## 日本でも政府が直接サービスを行う例はある

日本では政府が国民に直接サービスを行う業務オペレーションは存在しないのでしょうか。実はあります。しかも今回のコロナ対応で行われています。

厚生労働省が主幹する[休業支援金・給付金](https://www.mhlw.go.jp/stf/kyugyoshienkin.html)は書類は厚生労働省に直接提出し、審査後、厚生労働省の各都道府県支部（都道府県労働局）が支給することになっていて、政府が国民に直接サービス提供を行っています [^8] [^9] [^10]。

これが技術的に可能なのは、もともと労働基準監督署など、国が全国で実務オペレーションを行う機能を持っているからと推測されます。また、1つの組織で完結しているため、1つの業務オペレーションになっています。やればできるんですね。

[^8]: [”新型コロナウイルス感染症対応休業支援金・給付金”、厚生労働省](https://www.mhlw.go.jp/stf/kyugyoshienkin.html)（参照 2021-03-25）
[^9]: ["新型コロナウイルス感染症対応休業支援金・給付金支給要領"、厚生労働省](https://www.mhlw.go.jp/content/11600000/000647089.pdf)（参照 2021-03-25）
[^10]: ["都道府県労働局"、Wikipedia](https://ja.wikipedia.org/wiki/%E9%83%BD%E9%81%93%E5%BA%9C%E7%9C%8C%E5%8A%B4%E5%83%8D%E5%B1%80)（参照 2021-03-25）


## なぜ特別定額給付金はバラバラだったのか

こうなると、さらに疑問が深まります。なぜ厚生労働省（の旧労働省の）主幹事業では一元化できて、総務省主幹の特別定額給付金はできなかったのでしょうか。

その理由は今回の特別定額給付金事業の立て付け、スキームにあります。実は今回、特別定額給付金給付というのは、市区町村それぞれの自主事業という扱いになっています [^11] [^12]。

- 市区町村：自主事業の自治事務を行う、事業主体（！？）
- 国：事業主体に対し100％の負担率で補助金を交付する

正直、わけがわかりませんよね。政府が閣議決定して補正予算通しましたよね。政府の政策だと思いますよね。これには地方自治法と地方分権改革が背景にあります。

地方分権改革では、自治体が行う事務は下記のように決められました。

- 法定受託事務：国から委託される事務で、法律または政令による定めが必要
- 自治事務：法定受託事務以外全部

特別定額給付金の根拠になる法律は存在しません（これもびっくりですが、日本の場合、法律に支出の定めがなくても、予算案が議会を通れば支出可能なようです。予算法律説、などでググると専門家の説明がてきます・・・）。なので、書類の様式を含め、国から事細かに指示を行う法定受託事務にはできません。

あくまでも市区町村が「自主的に」行った自治事務ということらしいです。そうするとあとは約1700のバラバラの業務オペレーションにまっしぐらです。ちなみに地方自治体が勝手に交付金だしていいのか？というと、どうも地方自治法第232条の2が根拠っぽいです？ [^14]。


[^11]: [大森彌、"「特別定額給付金給付」はどういう事務か"、全国市区町村会](https://www.zck.or.jp/site/column-article/20345.html)（参照 2021-03-25）
[^12]: ["特別定額給付金(新型コロナウイルス感染症緊急経済対策関連)"、総務省](https://www.soumu.go.jp/menu_seisaku/gyoumukanri_sonota/covid-19/kyufukin.html)（参照 2021-03-25）
[^13]: [大阪市立中央図書館、"定額給付金の根拠法令が何か知りたい。"、レファレンス協同データベース](https://crd.ndl.go.jp/reference/detail?page=ref_view&id=1000109170)（参照 2021-03-27）（今回の給付金はリーマンショック時の麻生内閣の前例を踏襲）
[^14]: ["平成30年度前期 都市経営研究科 都市行政ワークショップⅠ 議事録 地方公共団体による補助金の交付と法的規制"、大阪市立大学](https://www.ug.gsum.osaka-cu.ac.jp/wp-content/uploads/2018/06/20180413.pdf)（参照 2021-03-25）

## その事務は本当にきめ細やかさが必要な箇所なのか？

ここまでで、特別定額給付金は地方分権に基づき、1700個のバラバラの業務オペレーションが発生すること、ベストプラクティスに見えるケースはいずれも1組織で業務オペレーションを実施していることが分かりました。

地方分権改革の1つの柱としてあげられていることに、権限をより住民に近い行政機能に移すことで、きめ細かい行政サービスを行うことが挙げられてます [^15] [^16]。でも今回の給付金や生活保護など、国で決めていることまで、地方自治体に業務オペレーションを任せる、という考え方は本当に正しいのでしょうか？

これ1つだけで地方分権改革がどうの、という話にはならないのですが、この国と地方自治体の役割分担の在り方に踏み込まないと、いつまでも「なんでこんなにうまくいってないの？○○ではこんな風になっているらしいのに」という現状は変わらないなと思った次第。

正直、最初は単なる国の丸投げじゃないの、と思っていたのですが、思ったよりも~~闇が深い~~根が深かったですね。

[^15]: ["地方自治制度の概要"、総務省](https://www.soumu.go.jp/main_sosiki/jichi_gyousei/bunken/gaiyou.html)（参照 2021-03-27）
[^16]: ["「国と地方の役割分担」について"、総務省](https://www.soumu.go.jp/main_content/000467822.pdf)（参照 2021-03-27）
[^17]: [地方分権改革事例集、内閣府地方分権改革推進室](https://www.cao.go.jp/bunken-suishin/doc/jirei30_h27_all_part1.pdf)（参照 2021-03-30）