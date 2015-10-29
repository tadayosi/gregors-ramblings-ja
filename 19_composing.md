# よい作曲家／システム構成者はきわめて稀だ #
（原文） http://www.eaipatterns.com/ramblings/19_composing.html

2004年11月30日

## 結合すべきか、分離すべきか（To Couple or Not to Couple） ##

最近は、結合性についての議論が活発だ。実は、この話題については言いたいことが山ほどあるので、大部分は別のポストに取っておこうと思う。今回は次のことだけに留めておこう。私の考えでは、結合性とはシステム間の依存性を測る指標だ。[David Orchard](http://www.pacificspirit.com/blog/)は、究極の疎結合ソリューションを次のようなナイスな言葉で定義している。「2つのシステムを疎結合にするにはどうすればいいか？　接続しないことだ！」　私が思うに、結合性についてこんなにも議論が白熱する理由の1つは、もともと接続を必ずしも想定していなかったシステム同士を接続することが一般に多い、という現実があるからだ。自然と、議論はどこまでの依存を、したがってどこまでの結合を、システム間に導入すべきかということになる。

悲しいかな、ソフトウェアアーキテクチャにおいて1つの正しい答えがあることは稀で、結合性についても例外ではない。結合を高めれば効率的でシンプルになるが、一方で硬直的で脆弱にもなる。一般に疎結合なアーキテクチャの例として、イベント駆動システム（event-driven system）がある。イベント駆動システムは、入力イベントに応答する独立した処理要素が集まって出来ている。ある処理要素がイベントメッセージを受け取ると、指定の機能を実行して、1つ以上の出力イベントを発行する。その出力イベントは、さらに別の処理ノードが受け取ることもできる。独立した処理要素からより大きな相互接続したシステムを構成することで、完全なソリューションを作り上げるのだ（下図を参照）。

![http://www.eaipatterns.com/img/composing.gif](http://www.eaipatterns.com/img/composing.gif)

## 構成可能性（Composability） ##

構成可能であることの利点の1つは、処理要素をさまざまなシステム群の中で再利用できることだ。再利用が可能なのは、処理要素が互いに直接的な前提を一切もっていないからだ。このことは再利用を助け、将来の要求に対しても柔軟に対応することを可能にする。たとえば、2つの既存ノード間に新たな処理ノードを差し挟むのは非常に簡単だ。しかし、柔軟性にはダークサイドもある（[ダース・ベイダーの息吹を聞こう](http://www.eaipatterns.com/img/vader.wav)）。柔軟なシステムを作る方法は、システムの構成方法についての制約を緩めることだ。こうすることで、将来の必要に応じてシステムを構成し直すことが可能となる。しかし、これはシステムが提供できるミス発見の手段が、少なくなることも意味する。たとえば、疎結合アーキテクチャとしては完全に妥当なシステムを構成したとしても、それが実際には何もまともな動作をしない、ということがあるかもしれない。たとえば、決して発行されることのないイベントを、処理要素が待ち受けていることもあるかもしれない。あるいは、チャネル上のメッセージ型が、処理要素が想定しているものと一致していないこともあるかもしれない。

## 統合テスト（Integration Testing） ##

結果として、疎結合な、構成可能なシステムでは、統合テストはこれまでにないほど重要になる。我々は、多くのプロジェクトからこの教訓を学んだ。ストーリーはこんな感じだ。それぞれのコンポーネントは十分に単体テストがされていて（メッセージングと単体テストについては[「メッセージングアーキテクチャにおける依存性注入」](13_ioc.md)を参照）、誰もが望むグリーンバーを示している。しかし、実際のシステムは致命的な欠陥を露呈してしまう。システムがビクとも動かない、というよくある奴だ。ほとんどの場合、その原因は個々のコンポーネントが不適切に構成されているからだ。ほとんどのシステムでは全構成の95%は実際には何ら役に立つことをしていない、ということを思い出す必要がある。しかし、柔軟なアーキテクチャに関するかぎり、システム構成はすべてが重要なのだ。

このことは、個々の処理要素を実際に構成することだけでなく、その構成可能性を検証することについても、我々は注意深く考える必要があることを意味する。どちらも簡単なタスクではない。構成可能性とは、個々の処理要素が実際の構成についてどんな過度の前提も置いていない、ということだ。多くの場合、従来のテスト手法では構成可能性を検証することができない。なぜなら構成可能性というのは、コンポーネントを予想しないシナリオでも使えることを意味するからだ。当然、予想できないシナリオを明示的にテストするのは難しい。その代わり、構成可能性について何かを言うならば、たいていはコード分析やインスペクションをしなければならない。構成可能性に対する最大の障害の1つは、隠れた前提だ。隠れた前提はたいてい回避できないが、さらに溢れんばかりの問題を持ち込みさえする。たとえば、あるコンポーネントが中心のデータベースにデータを格納して、そのデータが別のコンポーネントから読まれるならば、その2つのコンポーネント間には「帯域外の（out of band）」依存関係が持ち込まれる。その主な問題は、この依存関係はシステムのどこにも明示的に表現されておらず、たいていは実行時になるまで発見されないということだ。

## 新しいプログラミングモデル（New Programming Model） ##

個々の処理要素を実際に構成したものに対するテストは、システムの検証におけるもう1つの重要な部分だ。この種のテストは統合テストに分類されるもので、単体テストと真の機能（ユーザ）テストとの間に位置する。この種のテストで重要なことは、ここで選択された処理要素の構成が、期待通りの機能を果たすかどうかを検証することだ。通常は、システムのチャネルにテストデータを流して、期待される結果と実際とを比較することでテストする。幸いにして、疎結合のシステムでは一般にテストデータをとても簡単に注入できるので、そのテスト技法はとてもシンプルなもので済む。しかし、まともにテストカバレッジを達成するだけでも、チャレンジングな仕事になるだろう。

こうした統合テストの作成時には、構成可能性の導入によってアプリケーションに新しいプログラミングモデルも導入してしまっている、ということを意識しておく必要がある。構成されたシステムは[Pipes-and-Filters（パイプ＆フィルタ）](http://www.eaipatterns.com/PipesAndFilters.html)アーキテクチャのルールに従うのであって、オブジェクト指向アーキテクチャではないのだ。それはつまり、構成要素もルールも異なるものを扱うということだ。テストのアプローチを、少しばかり再検討する必要がある。

## 妥当性検証ルール（Validation Rules） ##

従来のテストでは、必要な妥当性検証の一部分しか実行されないだろう。その理由は、従来のプログラミングモデル（たとえばオブジェクト指向プログラミング）では、コンパイラがそのプログラミングモデル固有のルールの多くをすでに検証してくれているからだ。たとえば、構文エラー、生成前のオブジェクトへの参照などだ。構成層（composition layer）は我々自身が決めたルールに従うので、たいてい我々自身で妥当性検証をやらなくてはいけない。たとえば、ループをもつことは妥当なのか？　決して発行されないイベントを待ち受けることは妥当なのか？　誰も待ち受けていないイベントを発行することは妥当なのか？　自分自身とだけ通信して、互いに通信し合わない複数のサブシステムをもつことは妥当なのか？

これらの妥当性検証ルールを自動実行する構成検査の手段をもつことは、とても有益だ。通常、パイプ＆フィルタ型システムは有向グラフとして表すことができるため、妥当性検証を行うのにたいてい簡単なグラフアルゴリズムを利用できる。ポイントは、イベント発行とイベント受信に関する情報を、シンプルな中心リポジトリのようなものに一元管理することだ。この作業はできれば設計時に行いたいが、実行時に行ってもよい（たとえば、[Control Bus](http://www.eaipatterns.com/ControlBus.html)を通して）。このような妥当性検証が重要であることは、多くのアーキテクトが、自動化された分析を行えることがアーキテクチャスタイルのキーとなる性質だ、と考えているという事実に強調されている（たとえば、[「アーキテクチャスタイル、デザインパターン、オブジェクト」](http://www-2.cs.cmu.edu/afs/cs/project/compose/ftp/pdf/ObjPatternsArch-ieee97.pdf)を参照）。いったんアーキテクチャスタイルをより形式的に表現できれば、望ましい制約を表現できる言語を現実に思い描くことができる。こうした言語は、大規模なシステムでは実際に役立つだろう。別の方法として、システムの構造をもっと馴染みのある構文（たとえばXML）に変換してもいい。そして、XPATHのような言語を使って望ましい妥当性検証を実行するのだ。

もっと非形式的な分析方法に、出来上がったシステムを可視化する方法がある。たとえば、[以前のブログエントリ](11_dependencies.md)で書いたものだ。この場合、妥当性検証は半分しか自動化されない。なぜなら、その構成が良いか悪いかを評価するのは人間だからだ。しかし、人間が簡単に理解できる形の情報を、システムが自動的に準備してくれる。このアプローチは、やってみると非常に理にかなった妥協案だと判明することが多い。

## まとめ（Conclusion） ##

疎結合で構成可能なシステムを構築することには、明らかな利点がある。しかし、そのようなアーキテクチャは、システムに構成層という更なる層を導入することを忘れてはならない。この層は他とは別個に把握し、テストしなければならないものだ。それは、この層がたいてい他の層とは異なるアーキテクチャスタイルや設計原理に従うものだからだ。