# 名前がいったい何だというの？ #
（原文） http://www.eaipatterns.com/ramblings/39_namingchannels.html

2006年3月5日

_訳注1: 「名前がいったい何だというの？（What's in a name?）」はシェイクスピア『ロミオとジュリエット』の有名な台詞。_

## 新しい要素 (A New Element) ##

![http://www.eaipatterns.com/img/simplemessaging.png](http://www.eaipatterns.com/img/simplemessaging.png)

メッセージング・ベースの相互作用では、2つの新しい要素が導入される。チャンネルとメッセージだ。この2つの新要素そのものは単純なのだが、選択肢の幅が広がるので、新しい決断を求められる。一見簡単そうだが注意を要するのは、「チャンネルにどんな名前を付けるか？」という問題だ。メソッドが別のメソッドを直接呼ぶだけだった時代には、媒介する要素が間に何もなかったので、何も決めなくてよかった。今や、われわれは新しいレベルの間接性を手にしてしまったので、それに適切な名前を付けなければならないのだ。さて選択、選択・・・

## 単なる名前以上のもの (More Than Just a Name) ##

チャンネルに適切な名前を見つけるには、まずそのチャンネルが何を表現するものなのかを決める必要がある。チャンネルは純粋に論理的な構成物といえるようなものにも関わらず、送信者と受信者とを結びつける重要なメカニズムである。メッセージの交換を行なうためには、メッセージの送信者と受信者の両方が、共通のチャンネル名について合意しなければならない。

すぐに思い浮かぶ選択肢を、いくつか見てみよう。

  * 非常に単純なアプローチは、コンポーネント毎にチャンネルを割り当てるというものだ。たとえば、クレジットカードの検証を行なうコンポーネントを `CreditService` と呼ぶことにし、同じく `CreditService` と名付けられたチャンネルに送信されたメッセージに対して応答するものとする。クレジットに関係する何かが必要なコンポーネントは、みなこのチャンネルにメッセージを送信することになる。この単純なアプローチでも、実装レベルではある一定の間接性が実現される。チャンネル名さえ変えなければ、誰にも気付かれずにクレジットサービスの実装を別のものに置き換えることが可能だからだ。しかし、その相互作用の意味論は、それほど分離され（decoupled）てはいない。呼び出す側は、未だにどのコンポーネントが求める機能を提供しているかを知っていなくてはならないからだ。これでは、コンポーネント同士が直接、一対一の相互作用をする場合とほとんど同じである。

  * 特定のサービスへの依存性を減らすために、メソッド名にちなんだ名前をチャンネルに付けることもできる。メソッド名とは、すなわちそこで実現される操作のことだ。たとえば、サービスが顧客から提示されたクレジットカードを検証する操作を提供するならば、チャンネルには `VerifyCreditCard` という名前を付けるのだ。忘れてはならない重要なことは、これらのチャンネル操作は、オブジェクト指向アプローチの場合のようには特定のクラスに結びつけられていない、ということだ。その結果、どのコンポーネントがこのタイプの要求にサービスを提供できるかということを、呼び出し側はもはや知る必要がなくなり、抽象化のレベルがいくぶん増すことになる。サービス指向の計算処理は、一般にこのアプローチに従う。ただし、操作はたいてい特定のサービス（インタフェース）に結びつけられているのだが。

  * せっかくチャンネルを導入したのに、2つのコンポーネント間の相互作用の意味論は、まだ少しだけ呼び出しスタック・ベースのシステムと同じ臭いがする。あるコンポーネントが特定の要求（「このクレジットカードを確認せよ」）を送信し、応答（「カード良好」または「カード不良」）を期待する、というものだからだ。でもがっかりしないで欲しい、まだチャンネルの意味論の創造的可能性を、すべて検討し尽くした訳ではないからだ。上記の例はいずれも、要求側のコンポーネントが、クレジットカードを検証しなければならないことを知っているのが前提となっている。この重荷を「呼び出し側（caller）」から完全に取り払って、コンポーネント同士を真に分離することはできるだろうか？　チャンネル名（とそれに関連した意味）を `OrderReceived` に変更することで、分離の度合いをもう1ステップ進めることができる。この単純な名前の変更は、責務の相当大きな移動を意味する。チャンネル上のメッセージは、もはや命令を表すものではなくなり、むしろイベントとなる。あることが発生した、という通知だ。受信側がそのイベントにどう対応するかは、送信側からは完全に隠蔽される。また、そのイベントに対して受信者は1つである、という前提ももはや無い。イベントを1つ以上の購読者が処理してもいいのだ。たとえば、新しい注文が来たら、単にクレジットのチェックを実行するだけでなく、在庫の検証もする必要があるかもしれない。これでまた、通信し合うパーティ間の前提が減ることになった。結果として、EDAは多くの場合、SOAよりも疎結合だと考えられる。

![http://www.eaipatterns.com/img/channelnames.png](http://www.eaipatterns.com/img/channelnames.png)

## 責務の移動 (Shifting Responsibilities) ##

命令でなくイベントを通して通信を行なうということは、難解だが重要な責務の移動を示している。コンポーネント同士は分離されて、「呼び出し側」がもはや次にどの関数が実行されるかを知らなくてもよいし、どのコンポーネントがその関数を実行するかも知らなくてよい、というところまで行く。そしてもう1つ、同じく重要な呼び出し側と呼び出される側との間の責務の移動は、状態の維持に関するものである。

問い合わせ（queries）と命令に基づいたシステムにおいては、状態は通常、そのデータの「マスタ」と見なされるアプリケーションによって保持される。別のアプリケーションがそのデータを参照する必要があるときは、所有者のアプリケーションに対して問い合わせを送り、処理を中断してその応答を待つ。たとえば、ある注文管理システムが注文を捌く必要があるときは、商品発送アプリケーションに商品発送を指示するために、注文管理システムは顧客管理システムに顧客の住所を問い合わせる。

![http://www.eaipatterns.com/img/statetransfer1.png](http://www.eaipatterns.com/img/statetransfer1.png)

イベント駆動のシステムは、違う動きをする。正反対といえるほど違う。システムは他のシステムに情報を問い合わせるのではなく、代わりに必要なデータのコピーを自分自身で持ち、データが更新されるのを聞き取る。上記の例でいえば、商品発送システムが顧客の住所を複製して持っていて、注文が来たらその住所を商品発送の宛名に使えばよく、顧客管理システムに問い合わせる必要がない。このようにデータを複製して持つのは危険なように見えるが、利点もある。顧客管理システムは単にデータの変更をブロードキャストするだけでよく、誰がデータを持っているかを逐一知っている必要がない。顧客管理システムに住所データの問い合わせが送られることは決してないので、システムが成長して住所データの需要が増大したとしても、顧客管理システムがボトルネックなることは決してない。

この責務の移動の背後にある原理は、またしても結合度の概念と結びついている。疎結合の相互作用においては、データの情報源が通信相手の利便性のために、状態を保持しつづけることを要求されるべきではない。状態保持の重荷を消費側（consumer）に移動させることで、コンポーネントはデータ消費側のニーズを完全に忘れ去ることができる。これこそが、疎結合の主要素なのだ。問い合わせ－応答パターンの相互作用からの脱却は、多くのコンポーネントがイベントの集約者（Aggregator）として振る舞わなければならないことを意味する。つまり、コンポーネントは複数の情報源からイベントを聞き取り、自分に関係する状態を保持し、複数イベントからの情報をまとめて新たなイベントを生み出す。たとえば、商品発送システムなら住所変更イベントと注文イベントとを効果的にまとめ上げて、ある特定の住所への商品発送要求にする。

![http://www.eaipatterns.com/img/statetransfer2.png](http://www.eaipatterns.com/img/statetransfer2.png)

## 問題点 (The Catch) ##

イベント・ベースのシステムは、アーキテクチャの「採点表」のすべてにおいて、命令－制御型のシステムを上回っているようにも聞こえる。イベント・ベースのシステムは、より疎結合であり、個々のコンポーネントを自由に組み合わせて、さらに大きなシステムを構築できる。このように、再利用と柔軟性が高い。さらに加えて、イベント・ベースのシステムは、問題空間からシステムモデルへのマッピングも自然に行なうことができる。結局、実世界の多くの相互作用はイベントに基づいているという訳だ。それでは、非イベント・ベースのシステムを使うのは負け組なのか？　そうとも言えない。世の常どおり、タダ飯なんてありえないのだ、少なくともGoogle以外ではね。動的で疎結合なシステムというのは、本質的に設計したりデバッグしたりするのが困難だ（「[よい作曲家／構成者はきわめて稀だ](19_composing.md)」を参照）。そのため、正しいツールを手に入れて（たとえば、「[依存性の可視化](11_dependencies.md)」を参照）、かつ小規模な早い段階での失敗から、経験を積んでおくことが間違いなく必要なのだ。