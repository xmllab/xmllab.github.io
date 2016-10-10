---
layout: post
title: "RubyStyleGuideとRubocopでの設定"
date: 2016-10-10 09:18:25 +0900
comments: false
categories: 
---

いろいろな書き方ができるRuby。そのRubyを取り巻くコーディングポリシーの状況やツールについて、この回では書いていきたいと思います。

以前に某AdventCalendarに載せた内容だけどRubyStyleGuideもRubocopもその時に比べて変更されてきているので、改めて内容の反映した上でこっちに掲載。


Rubyの特徴とベストプラクティス
--------

Rubyは１つの目的を達成するために複数のアプローチが可能な言語です。例えば配列の長さを取得したいときに`array.size`と`array.length`のどちらを使っても、配列の長さを取得することができます。これはもともと他の言語を使っていた人がRubyに親しみやすくなる要因の一つとなっているのですが、ある程度大きな開発をするときにはコーディングポリシーをしっかり決めておかないと、プログラムを書いた人によっては癖が強すぎて読みづらい事態が起きたり、最悪の場合追加開発の時に元のコードを読み違えて、バグの誘発につながるリスクもあります。

それらのリスクを低減し、他のRubyプログラマーが保守できるコードを書くためのベストプラクティスとして、[The Ruby Style Guide](https://github.com/fortissimo1997/ruby-style-guide/blob/japanese/README.ja.md)というものが規定されています。

しかし定義されているポリシーは膨大で、気をつけていたつもりでも実は準拠していないコードを書いてしまうケースもあるでしょう。


そんな時に役にたつツールの一つとして、[RuboCop](https://github.com/bbatsov/rubocop)があります。


静的解析ツール：RuboCop
--------

[RuboCop](https://github.com/bbatsov/rubocop)は、あらかじめ決められたコーディングポリシーを元に、それに対して違反していないか、違反していた場合どう直すべきかの指標を示してくれるツールの１つです。

インストールも簡単で、下記のコマンドでインストールを行えます。

``` sh
$ gem install rubocop --no-ri --no-rdoc
Successfully installed rubocop-0.27.1
1 gem installed
```

実際に下記のようなrubyファイルを作成し、`rubocop`コマンドで検査してみましょう

rubyファイル（sample.rb）
``` ruby
def aaaa
    bbbb
end
```

チェックの実行
``` sh
$ rubocop sample.rb 
Inspecting 1 file
C

Offenses:

sample.rb:2:1: C: Use 2 (not 6) spaces for indentation.
      bbbb
^^^^^^

1 file inspected, 1 offense detected
```

実行してみるとエラーが出ます。
これはインデントが半角スペース2つ(RubyStyleGuide準拠)ではないために起こっているエラーです。

これを半角スペース２つに直すと、今度は無事に実行完了となるはずです。

``` sh
$ rubocop sample.rb 
Inspecting 1 file
.

1 file inspected, no offenses detected
```

RuboCop活用する
--------

さきほどは保存したファイルに対して[RuboCop](https://github.com/bbatsov/rubocop)を実行しましたが、保存する前段階にエディタに警告を表示してくれるプラグインもあったりします。

* [vim-rubocop](https://github.com/ngmy/vim-rubocop)
* [rubocop-emacs](https://github.com/bbatsov/rubocop-emacs)
* [Sublime RuboCop plugin](https://github.com/pderichs/sublime_rubocop)




また、[Jenkins](http://jenkins-ci.org/)を活用している場合、[rubocop-checkstyle_formatter](https://github.com/eitoball/rubocop-checkstyle_formatter)と[Jenkins](http://jenkins-ci.org/)の[ViolationsPlugin](https://wiki.jenkins-ci.org/display/JENKINS/Violations)を組み合わせることで、[Jenkins](http://jenkins-ci.org/)のジョブ上に警告を表示させることもできます。


RuboCopとチーム開発
--------

[RuboCop](https://github.com/bbatsov/rubocop)は上記ツール類と組み合わせることで、チームとしてのコーディングポリシーをある程度統一することができます。ただ、各々のプロジェクトチームのこれまでの積み重ねであったり、文化的な背景であったりで、「どうしてもこの記述は合わない」といったような例外事項が出てくると思います

そんなときのために[RuboCop](https://github.com/bbatsov/rubocop)には設定ファイルでポリシーを制御する機能があります。プロジェクトのルートフォルダに`.rubocop.yml`という名前で設定を記述することで、チームにとって最適なポリシーでコーディングすることができます。設定ファイルの内容は[config/default.yml](https://github.com/bbatsov/rubocop/blob/master/config/default.yml)を元に記述します。

ただ、各設定項目が[Ruby Style Guide](https://github.com/fortissimo1997/ruby-style-guide/blob/japanese/README.ja.md)のどの部分に該当するのかを日本語でまとめたところがなかったので、デフォルトでの有効無効設定と合わせてまとめてみました。(2016/10/01現在)


RubyStyleGuideでの記述 | RubocopのCop | デフォルト値が有効
:----------------------|:------------|:--------:
ソースファイルのエンコーディングにはUTF-8を用いましょう。 | Style/Encoding: | 
インデントにはスペース2つを用いましょう(別名ソフトタブ)。ハードタブを用いてはいけません。 | Style/IndentationWidth: | ◯
Unix-styleの改行にしましょう。(*BSD/Solaris/Linux/OSXユーザーはデフォルトで設定されています。Windowsユーザーは特に注意が必要です。) | Style/EndOfLine: | ◯
命令文や式の区切りに;を用いてはいけません。当然、１行につき式１つにしましょう。 | Style/Semicolon: | ◯
本文のないクラスは１行のフォーマットを用いましょう。 | #N/A | 
１行のメソッドは避けましょう。この用法は実際にはよく見かけるものではあるのですが、構文に奇妙な点があるため、使用は望ましくないです。仮に使うとしても、１行メソッドに含めるのは多くとも式１つまでにすべきです。本文が空のメソッドはこのルールの例外です。 | Style/SingleLineMethods: | ◯
演算子の前後、コンマ、コロン、セミコロンの後ろ、{の前後、}の前にはスペースを入れましょう。スペースはRubyのインタープリタには(ほとんどの場合)重要ではありませんが、スペースの適切な使用は、読みやすいコードを書くための鍵です。／演算子についてただひとつの例外は、指数演算子です:／{と}については多少の解説が必要でしょう。ブロック、ハッシュリテラル、そして文字列埋め込み式にそれぞれ使われるからです。ハッシュリテラルでは、２つのスタイルが許容できます。／１つ目の書き方は、わずかながら少し読みやすいです(そして、Rubyコミュニティでより広く使われているのはこちらかもしれません)。２つ目の書き方は、ブロックとハッシュを視覚的に差別化できるという点で有利です。どちらでも片方を採用すれば、常に同じ方式を採用しましょう。 | Style/SpaceAfterColon: | ◯
(、[の後と、]、)の前にはスペースは入れません。 | Style/SpaceInsideBrackets: | ◯
!の後にはスペースは入れません。 | Style/SpaceAfterNot: | ◯
範囲リテラルの内側にスペースは入れません。 | Style/SpaceInsideRangeLiteral: | ◯
whenはcaseと同じ深さに揃えましょう。このスタイルは"TheRubyProgrammingLanguage"、"ProgrammingRuby"双方で確立されたものです。 | Style/CaseIndentation: | ◯
条件式を変数に代入するときは、条件分岐のインデントは普段と同じ基準で揃えましょう。 | Style/MultilineAssignmentLayout: | 
メソッド定義式の間には空行をいれ、メソッド同士は論理的なかたまりに分けましょう。 | Style/EmptyLineBetweenDefs: | ◯
メソッド呼び出しの最後の引数の後ろのコンマは避けましょう。引数が複数行にわかれていない時は、特に避けましょう。 | Style/TrailingCommaInArguments: | ◯
メソッドの引数に初期値を割り当てるとき、=演算子の周りにはスペースを入れましょう。／いくつかのRuby本は最初のスタイルを提案していますが、２つ目の方が、実用的により優れています(そして、おそらくこちらのほうが読みやすいでしょう)。 | Style/SpaceAroundEqualsInParameterDefault: | ◯
\を用いた行の継続は可能であれば避けましょう。可能であればというのは、つまり、文字列連結以外のすべての場合でです。 | #N/A | 
一貫した複数行のメソッドチェーンのスタイルを採用しましょう。Rubyコミュニティには2つのよく使われるスタイル-先頭に.を付けるもの(OptionA)、末尾に.を付けるもの(OptionB)-があり、どちらも良いと考えられています。 | Style/DotPosition: | ◯
メソッド呼び出しが複数行に及ぶときは、引数は揃えましょう。１行の長さの制約のために、引数を揃えるのに適していない時は、最初の引数以降をインデント１つ分で揃えるスタイルも許容できます。 | Style/AlignParameters: | ◯
複数行に及ぶ配列は、要素を揃えましょう。 | Style/AlignArray: | ◯
可読性のため、大きな数値にはアンダースコアをつけましょう。 | Style/NumericLiterals: | ◯
APIドキュメントを書くなら、RDocとその規約に従いましょう。コメント行とdefの間に空行を入れてはいけません。 | #N/A | 
１行は80字までにしましょう。 | Metrics/LineLength: | ◯
行末のスペースは避けましょう。 | Style/TrailingWhitespace: | ◯
ファイルの終端には改行を入れましょう。 | Style/TrailingBlankLines: | ◯
ブロックコメントは使ってはいけません。前にスペースが入ると機能しませんし、通常のコメントと違い、簡単に見分けが付きません。 | Style/BlockComments: | ◯
::は、定数(クラスやモジュールも含みます)やコンストラクタ(例えばArray()やNokogiri::HTML())を参照するときにのみ使いましょう。通常のメソッド呼び出しでは::の使用は避けましょう。 | Style/ColonMethodCall: | ◯
引数があるとき、defは括弧と共に使いましょう。引数がない場合は括弧は除きましょう。 | Style/DefWithParentheses: | ◯
オプショナル引数は引数リストの最後に定義しましょう。引数リストの先頭にオプショナル引数があるメソッドを呼んだ場合、Rubyの挙動は予測不能です。 | Style/OptionalArguments: | ◯
変数を定義するために多重代入を使うのは避けましょう。多重代入を使っていいのはメソッド戻り値を変数に代入する時、splat演算子とともに使う時、変数の値を相互に入れ替えたい時に限ります。多重代入は代入をそれぞれ別に実施した場合と比べて可読性に劣ります。 | Style/ParallelAssignment: | ◯
多重代入においては不要なアンダースコア変数を後ろに並べないようにしましょう。アンダースコア変数は左辺にsplat変数を定義するときには必要です。その場合、splat変数はアンダースコアではありえないです。 | #N/A | 
forは、どうしても使わなければいけない明確な理由が明言できる人以外は、使ってはいけません。多くの場合は代わりにイテレータを使うべきです。forはeachをつかって実装されています(だから、より遠回しです)が、forは(eachと違い)新しいスコープを導入せず、そのブロック内で定義された変数は、ブロックの外からも見えます。 | Style/For: | ◯
thenは複数行にまたがるif/unlessでは使ってはいけません。 | Style/MultilineIfThen: | ◯
複数行にまたがるif/unlessでは、条件式は常にif/unlessと同じ行に置きましょう。 | Lint/ConditionPosition: | ◯
三項演算子(?:)をif/then/else/end構文よりも優先的に使いましょう。そちらの方がより一般的だし、あきらかに簡潔です。 | Style/OneLineConditional: | ◯
三項演算子の１つの分岐には１つだけ式を入れましょう。つまり、三項演算子はネストしてはいけません。そのようなケースではif/elseの方がよいです。 | Style/NestedTernaryOperator: | ◯
ifx;...を使ってはいけません。代わりに三項演算子を使いましょう。 | Style/IfWithSemicolon: | ◯
ifやcaseが式で、値を返すという事実を活用しましょう。 | #N/A | 
１行のcase文ではwhenxthen...を使いましょう。代わりの表現であるwhenx:...は、Ruby1.9で廃止されました。 | Style/WhenThen: | ◯
whenx;...を使ってはいけません。 | #N/A | 
notの代わりに!を使いましょう。 | Style/Not: | ◯
!!は避けましょう。 | Style/DoubleNegation: | ◯
andとorの使用は禁止です。使うべき理由がないです。常に、代わりに&&と｜｜を使いましょう。 | Style/AndOr: | ◯
複数行にまたがる三項演算子?:は避けましょう;代わりにif/unlessを使いましょう。 | Style/MultilineTernaryOperator: | ◯
本文が１行のときは、if/unless修飾子を優先的に使いましょう。他の良い代替案としては&&/｜｜を使った制御構文があります。 | Style/IfUnlessModifier: | ◯
複数行に渡るような些細とは言えない規模のブロックにif/unless修飾子を用いるのは避けましょう。 | #N/A | 
if/unless/while/until修飾子をネストして利用しないようにしましょう。可能であれば&&/｜｜を使いましょう。 | Style/NestedModifier: | ◯
否定形のときはifよりunlessを優先的に使いましょう。(もしくは｜｜構文を使いましょう)。 | Style/NegatedIf: | ◯
unlessをelse付きで使ってはいけません。肯定条件を先にして書き換えましょう。 | Style/UnlessElse: | ◯
if/unless/while/untilの条件式の周囲を括弧で括らないようにしましょう。／留意しなければならないのは、このルールには例外(条件式中の安全な代入)があるということです。 | Style/ParenthesesAroundCondition: | ◯
複数行のwhile/untilでは、while/untilconditiondoを使ってはいけません。 | Style/WhileUntilDo: | ◯
本文が１行のときは、while/until修飾子を利用しましょう。 | Style/WhileUntilModifier: | ◯
否定形のときは、whileよりもuntilを使いましょう。 | Style/NegatedWhile: | ◯
無限ループが必要な時は、while/untilの代わりにKernel#loopを用いましょう。 | Style/InfiniteLoop: | ◯
後判定ループの場合、begin/end/untilやbegin/end/whileより、break付きのKernel#loopを使いましょう。 | Lint/Loop: | ◯
内部DSL(例えばRake、Rails、RSpec)や、Rubyで「キーワード」と認識されているメソッド(例えばattr_readerやputs)や、アトリビュートにアクセスするメソッドでは、引数の周りの括弧を省略しましょう。それ以外のすべてのメソッドでは、メソッド呼び出しの時に括弧を付けましょう。 | #N/A | 
暗黙のオプションハッシュの外側の括弧は省略しましょう。 | #N/A | 
内部DSLの一部として使われるメソッドの引数では、外側の括弧類は省略しましょう | #N/A | 
引数のないメソッド呼び出しの括弧は省略しましょう。 | #N/A | 
ブロック内で呼び出されるメソッドがただ１つである場合、簡略化されたproc呼び出しを用いましょう。 | #N/A | 
１行のブロックではdo...endより{...}を使いましょう。複数行のブロックでは{...}は避けましょう(複数行のメソッドチェーンは常に醜いです)。制御構文(的な用法)やメソッド定義(的な用法)では常にdo...endを使いましょう(例えばRakefilesや特定のDSLなど)メソッドチェーンでのdo...endは避けましょう。／{...}を用いた複数行のメソッドチェーンをOKと主張する人もいるかもしれないが、自問してみてほしい-そのコードは本当に読みやすいだろうか？また、そのブロックの中はメソッドに切り出すことはできないのか？ | Style/BlockDelimiters: | ◯
単に他のブロックに引数を渡すだけのブロックリテラルを避けるため、ブロック引数を明示することを検討しましょう。ただしブロックがProcに変換されることでのパフォーマンスに気をつけましょう。 | #N/A | 
制御構文上不要なreturnは避けましょう。 | Style/RedundantReturn: | ◯
不要なselfは避けましょう(selfのアクセサへの書き込みでのみ必要です)。 | Style/RedundantSelf: | ◯
当然の帰結として、ローカル変数でメソッドをシャドウイングするのは、それらが等価なものでない限り避けましょう。 | #N/A | 
代入部分を括弧で囲まずに、=の返り値を条件式に用いてはいけません。これは、Rubyistの中では条件式内での安全な代入としてとても有名です。 | Lint/AssignmentInCondition: | ◯
利用できるときには省略された自己代入演算子を用いましょう。 | Style/SelfAssignment: | ◯
変数がまだ初期化されていないときにだけ初期化したいのであれば、｜｜=を使いましょう。 | #N/A | 
boolean変数には｜｜=を用いてはいけません(現在の値がfalseであったときに何が起こるか考えてみましょう)。 | #N/A | 
値が入っているかわからない変数の前処理のは&&=を用いましょう。&&=を使えば変数が存在するときのみ値を変更するので、存在確認に用いている不要なifを除去できます。 | #N/A | 
case等価演算子===の露骨な使用は避けましょう。その名が示す通り、caseの条件判定で用いられており、その外で用いられると混乱のもとになります。 | Style/CaseEquality: | ◯
==で用が足りるならeql?は使わないようにしましょう。eq?で実現されている、より厳密な等価性が必要になることは、実際には稀です。 | #N/A | 
Perlスタイルの($:や$;などのような)特別な変数の使用は避けましょう。それらは極めて暗号的なので、ワンライナー以外での利用は推奨できません。Englishライブラリから提供される人にやさしいエイリアスを用いましょう。 | Style/SpecialGlobalVars: | ◯
メソッド名と開き括弧の間にスペースを入れてはいけません。 | Style/SpaceAfterMethodName: | ◯
メソッドの最初の引数が開き括弧で始まるならば、常にメソッド呼び出しに括弧を用いましょう。例えば次のように書きますf((3+2)+1)。 | #N/A | 
Rubyインタープリタを走らせるときは、常に-wオプションを付けましょう。これまでのルールのどれかを忘れてしまった時に警告を出してくれます！ | #N/A | 
ネストしたメソッド定義は行ってはいけません-代わりにラムダを用いましょう。ネストしたメソッド定義は実際には外側のメソッドと同じスコープ(例えばclass)でメソッドを生成します。そのうえ、そのようにネストしたメソッドは、そのメソッドを含んでいるメソッドがが呼び出されるたびに再定義されるでしょう。 | Lint/NestedMethodDefinition: | ◯
１行の本文を持つラムダには新しいリテラルを持ちましょう。lambdaは複数行にまたがるときに使いましょう。 | Style/Lambda: | ◯
stabbylambdaを定義するときは、引数の周りの括弧は省略しないようにしましょう。 | Style/StabbyLambdaParentheses: | ◯
stabbylambdaに引数がないときは、引数のための括弧は省略しましょう。 | #N/A | 
Proc.newよりprocを使いましょう。 | Style/Proc: | ◯
lambdaやprocの呼び出しにはproc[]やproc.()よりproc.call()を使いましょう。 | Style/LambdaCall: | ◯
使わないブロック引数やローカル変数の先頭には_を付けましょう。単に_を用いるのも許容されます(少し説明不足ではありますが)。この記法を使うことで、RubyインタープリタやRubocopのような対応しているツールでは変数を使っていないという警告を抑制できます。 | Lint/UnusedBlockArgument: | ◯
STDOUT/STDERR/STDINの代わりに$stdout/$stderr/$stdinを用いましょう。STDOUT/STDERR/STDINは定数であり、Rubyでの定数は、実際は再代入できます(つまりリダイレクトに使えます)が、もし実行するとインタープリタからの警告が出ます。 | #N/A | 
$stderr.putsの代わりにwarnを用いましょう。簡潔さや明快さもさることながら、warnは必要であれば警告を抑制することができます(警告レベルを-W0を用いて0に設定することによって実現できます)。 | #N/A | 
あまりに暗号めいているString#%メソッドよりもsprintfやformatを使いましょう。 | Style/FormatString: | ◯
あまりに暗号めいているArray#*メソッドよりもArray#joinを使いましょう。 | Style/ArrayJoin: | ◯
配列かどうかわからない変数を配列とみなして処理したいときは、明示的にArrayかどうかチェックするよりも、[*var]やArray()を使いましょう。 | #N/A | 
ロジックを使って複雑な比較を行うよりも、可能な限りRangeやComparable#between?を用いましょう。 | #N/A | 
==を明示した比較よりも判定メソッドを用いましょう。数値の比較はOKです。 | Style/EvenOdd: | ◯
boolean値を扱わない限り、明示的なnilでないかの検査は避けましょう。 | Style/NonNilCheck: | ◯
BEGINブロックの使用は避けましょう。 | Style/BeginBlock: | ◯
ENDブロックを使ってはいけません。代わりにKernel#at_exitを使いましょう。 | Style/EndBlock: | ◯
フリップフロップの使用は避けましょう。 | Style/FlipFlop: | ◯
制御構文で条件式のネストは避けましょう。／不正なデータを弾きたいときはガード節を使いましょう。ガード節とは、できるだけ素早く関数から抜けられるようにと関数の先頭に置かれている条件文のことです。／ループ内では条件判定ブロックよりもnextを使いましょう。 | Style/GuardClause: | ◯
collectよりmap、detectよりfind、find_allよりselectinjectよりreduce、lengthよりsizeを使いましょう。これは絶対のルールではないです。別名のほうが可読性に優れているなら、そちらを使っていただいて構いません。韻を踏んでいるほうのメソッド名はSmalltalkから引き継いできたもので、他のプログラミング言語でそこまで一般的ではないです。find_allよりもselectが推奨されるのは、rejectとの相性がよいことと、メソッド名から挙動を推察することも容易だからです。 | Style/CollectionMethods: | 
sizeの代わりにcountを用いてはいけません。Array以外のEnumerableオブジェクトでは、countを使うと要素数の計算のためにコレクション全体を走査してしまいます。 | #N/A | 
mapとflattenの組み合わせの代わりに、flat_mapを用いましょう。これは深さが２以上の配列には適用できません。すなわち、users.first.songs==['a',['b','c']]のときは、flat_mapよりmap+flattenを用いましょう。flattenは配列を全て平坦にするのに対し、flat_mapは配列を１次元だけ平坦にします。 | #N/A | 
reverse.eachの代わりにreverse_eachを用いましょう。何故なら、Enumerableをincludeしているクラスの側で、より効率のよい実装が行われていることがあるからです。最悪、クラスが特殊化を行っていない場合でも、Enumerableから継承される一般的な実装は、reverse.eachを用いた場合と同じ効率です。 | #N/A | 
識別子は英語で名づけましょう。 | Style/AsciiIdentifiers: | ◯
シンボル、メソッド、変数にはsnake_caseを用いましょう。 | Style/MethodName: | ◯
クラスやモジュールにはCamelCaseを用いましょう。(HTTP、RFC、XMLのような頭字語は大文字を保ちましょう)。 | Style/ClassAndModuleCamelCase: | ◯
ファイル名にはsnake_caseを用いましょう。例えばhello_world.rbのように。 | Style/FileName: | ◯
ディレクトリ名にはsnake_caseを用いましょう。例えばlib/hello_world/hello_world.rbのように。 | #N/A | 
ソースファイル１つにつきただ１つのクラス/モジュールだけが書かれている状態を目指しましょう。そのファイルの名前はそのクラス/モジュールと同じ名前で、ただしキャメルケースをスネークケースに変換して用いましょう。 | #N/A | 
定数はSCREAMING_SNAKE_CASEを用いましょう。 | Style/ConstantName: | ◯
述語メソッド(boolean値が返るメソッド)は疑問符で終わりましょう。(すなわちArray#empty?のように)。boolean値を返さないメソッドは、疑問符で終わるべきではないです。 | Style/PredicateName: | ◯
危険な可能性のあるメソッド(引数やselfを変更するようなメソッドや、exit!(exitと違ってファイナライザが走らない)のようなもの)は、その安全なバージョンがある場合には、危険であることを明示する意味で感嘆符で終わりましょう。 | #N/A | 
危険な(感嘆符付き)メソッドがあるときは、対応する安全な(感嘆符なし)メソッドを定義できないか検討しましょう。 | #N/A | 
短いブロックと共にreduceを使うとき、引数は ｜ a,e ｜ と名づけましょう。(accumulator,element). | Style/SingleLineBlockParams: | ◯
二項演算子を定義するとき、引数名はotherを用いましょう(<<と[]は意味が違ってくるので、このルールの例外です)。 | Style/OpMethod: | ◯
自己説明的なコードを書いて、このセクションの残りのパートは無視しましょう。本当に！ | #N/A | 
コメントは英語で書きましょう。 | Style/AsciiComments: | ◯
最初の#とコメントの間にスペースを１つ入れましょう。 | Style/LeadingCommentSpace: | ◯
１語より長いコメントは頭語を大文字化してピリオドを打ちましょう。文と文の間にはスペースを一つだけ入れましょう。 | #N/A | 
過剰なコメントは避けましょう。 | #N/A | 
コメントは最新に保ちましょう。古くなったコメントは、コメントがないより悪いです。 | #N/A | 
悪いコードを説明するコメントは避けましょう。自己説明的なコードへのリファクタリングを行いましょう(やるかやらないか-"やってみる"はなしだ。--Yoda)。 | #N/A | 
注釈は、通常関連するコードのすぐ上に書きましょう。 | #N/A | 
注釈のキーワードの後ろは:を続けましょう。その後ろに問題点を書きましょう。 | Style/CommentAnnotation: | ◯
もし問題点の記述に複数行かかる場合は、後続の行は#の後ろにスペース３つでインデントしましょう。(通常の１つに加え、インデント目的に２つ) | #N/A | 
もし問題が明らかで、説明すると冗長になる時は、問題のある行の末尾に、本文なしの注釈だけ付けましょう。この用法は例外であり、ルールではありません。 | #N/A | 
あとで追加されるべき、今はない特徴や機能の注釈にはTODOを使いましょう。 | #N/A | 
直す必要がある壊れたコードの注釈にはFIXMEを使いましょう。 | #N/A | 
パフォーマンスに問題を及ぼすかもしれない遅い、または非効率なコードの注釈にはOPTIMIZEを使いましょう。 | #N/A | 
疑問の残るコードの書き方でコードの臭いを感じた箇所の注釈にはHACKを使いましょう。 | #N/A | 
意図したとおりに動くか確認する必要がある箇所の注釈にはREVIEWを使いましょう。例:REVIEW:ArewesurethisishowtheclientdoesXcurrently? | #N/A | 
適切に感じるのであれば、他の独自のキーワードを用いても構いませんが、それらのキーワードはREADMEやそれに類するものに書いておきましょう。 | #N/A | 
クラス定義の構造には一貫性をもたせましょう。 | #N/A | 
クラスの中に複数行あるようなクラスをネストしてはいけません。それぞれのクラスごとにファイルに分けて、外側のクラスの名前のついたフォルダに含めるようにしましょう。 | #N/A | 
クラスメソッドしかないクラスよりモジュールを使いましょう。クラスはインスタンスを生成することに意味がある時にのみ使われるべきです。 | #N/A | 
モジュールのインスタンスメソッドをクラスメソッドにしたいときは、extendselfよりもmodule_functionを使いましょう。 | Style/ModuleFunction: | ◯
クラス階層の設計を行うときは、リスコフの置換原則.に従いましょう。 | #N/A | 
あなたのクラスを可能な限りSOLIDに保ちましょう。 | #N/A | 
ドメインオブジェクトのクラスにおいては常に適切なto_sメソッドを提供しましょう。 | #N/A | 
単純なアクセサやミューテータの定義には、attr群を用いましょう。 | Style/TrivialAccessors: | ◯
attrの使用は避けましょう。代わりにattr_readerやattr_accessorを使いましょう。 | Style/Attr: | ◯
Struct.newの使用を考えましょう、それは、単純なアクセサ、コンストラクタや比較演算子を定義してくれます。 | #N/A | 
Struct.newで初期化されたインスタンスを拡張してはいけません。それは余分なクラスレベルをもたらし、複数回requireされた時に、奇妙なエラーの原因にもなります。 | Style/StructInheritance: | ◯
あるクラスのインスタンス生成する追加の方法を提供したいときは、ファクトリメソッドの追加を検討しましょう。 | #N/A | 
継承よりダック・タイピングを使いましょう。 | #N/A | 
継承での振る舞いが"扱いづらい"ので、クラス変数(@@)の使用は避けましょう。／このように、ひとつのクラス階層に属するすべてのクラスは、１つのクラス変数を共有してしまいます。クラス変数よりもクラスのインスタンス変数のほうを使うべきです。 | Style/ClassVars: | ◯
意図した使い方に沿って、可視性(private、protected)を設定しましょう。全てをpublic(デフォルトの設定)のままにしないようにしましょう。結局私達は今Rubyを書いているのだから。Pythonではなく。 | #N/A | 
public、protected、privateは、適用するメソッド定義と同じインデントにしましょう。そして、以降のすべてのメソッド定義に適用されることを強調するために、それらの修飾子の前１行と後１行に空行を入れましょう。 | Style/AccessModifierIndentation: | ◯
クラスメソッドを定義するときはdefself.methodを用いましょう。クラス名を繰り返さないので、簡単にリファクタリングできるようになります。 | Style/ClassMethods: | ◯
メソッドの別名をつける時はaliasを使いましょう。クラスのレキシカルスコープの中ではselfと同じように名前解決されます。またユーザーにとっても、実行時やサブクラス側で明示的にエイリアスを変更しなければ、変更されないことが読み取りやすいです。／aliasはdefと同じく予約語なので、シンボルや文字列よりも名前そのものを使いましょう。言い換えると、alias:foo:barではなく、aliasfoobarと書きましょう。／また、Rubyがエイリアスや継承をどのように扱うか注意しましょう。aliasはそれが定義された地点で解決されたメソッドを参照します。動的には解決されません。／この例では、Fugitive#given_nameは、Fugitive#first_nameではなく、オリジナルのWesterner#first_nameを呼び出します。Fugitive#given_nameもオーバーライドしたい時は、継承したクラスでも再定義しなければなりません。 | #N/A | 
モジュールやクラス、実行時のシングルトンクラス等では、aliasの挙動が予期できないので、エイリアス定義には常にalias_methodを用いましょう。 | Style/Alias: | ◯
例外はfailよりraiseを使いましょう。 | Style/SignalException: | ◯
２引数のraiseでは、RuntimeErrorを明示しないようにしましょう。 | Style/RedundantException: | ◯
raiseの引数としては例外クラスのインスタンスよりも、例外クラスとメッセージをそれぞれの引数で渡す方を使いましょう。 | Style/RaiseArgs: | ◯
ensureブロックからreturnしてはいけません。もしensureの中から明示的に値を返した場合は、returnはどの例外発生よりも優先されて、例外など発生していなかったかのように値を返してしまいます。事実上、例外は静かに捨てられます。 | Lint/EnsureReturn: | ◯
可能な場所では、暗黙のbeginブロックを用いましょう。 | Style/RedundantBegin: | ◯
不確実性のあるメソッド(AvdiGrimmによって作られた言葉です)を用いてbeginの蔓延を和らげましょう。 | #N/A | 
例外をもみ消してはいけません。 | Lint/HandleExceptions: | ◯
rescueを修飾子として利用するのは避けましょう。 | Style/RescueModifier: | ◯
制御フローに例外を使っては行けません。 | #N/A | 
Exceptionをrescueするのは避けましょう。これはexitのシグナルも捕捉するため、プロセスを殺すのにkill-9が必要になります。 | Lint/RescueException: | ◯
より詳細な例外をrescueチェーンの上に配置しましょう。そうでなければ、決してrescueされません。 | #N/A | 
プログラム内で確保した外部リソースは、ensureで開放しましょう | #N/A | 
自動的にリソースを開放してくれる機能を含むメソッドを利用可能な時は、そちらを使いましょう。 | #N/A | 
新しい例外クラスを導入するより、基本ライブラリの例外クラスを使いましょう | #N/A | 
配列やハッシュを生成する時はリテラル記法を使いましょう。(コンストラクタに引数を渡す場合を除けば、ということですが) | Style/EmptyLiteral: | ◯
単語(空でなく、スペースを含まない文字列)の配列を生成する時は%wリテラルを使いましょう。このルールは配列の要素が２つ以上の場合に限ります。 | Style/WordArray: | ◯
シンボルの配列が必要な時(かつRuby1.9との互換性を維持しなくていい時)は%iリテラルを使いましょう。このルールは配列の要素が２つ以上の場合に限ります。 | Style/SymbolArray: | 
ArrayやHashリテラルの最後の要素の後ろの,は避けましょう。複数行にわかれていない時は特に避けましょう。 | Style/TrailingCommaInLiteral: | ◯
配列に大きな隙間を作るのは避けましょう。 | #N/A | 
配列の最初や最後にアクセスしたいときは、[0]や[-1]よりfirstやlastを使いましょう。 | #N/A | 
要素が一意のものを扱うときは、Arrayの代わりにSetを用いましょう。Setは要素に重複と順序がないようなコレクションの実装です。これはArrayの直感的な二項演算子と、Hashの速さが合わさっています。 | #N/A | 
ハッシュのキーには文字列よりシンボルが好まれます。 | #N/A | 
変更のできるオブジェクトをハッシュのキーに使うのは避けましょう。 | #N/A | 
ハッシュのキーがシンボルの時は、Ruby1.9のハッシュリテラル記法を用いましょう。 | Style/HashSyntax: | ◯
Ruby1.9のハッシュ記法とロケット記法を同じハッシュリテラル内で混在させてはいけません。シンボルでないキーがある場合は、ロケット記法を使いましょう。 | #N/A | 
Hash#has_key?よりHash#key?を、Hash#has_value?よりHash#value?を用いましょう。ここでMatzが述べているように、長い記法は廃止が検討されています。 | Style/PreferredHashMethods: | ◯
存在すべきキーを扱う時は、Hash#fetchを用いましょう。 | #N/A | 
Hash#fetchのデフォルト値を使い、自力でロジックを書かないようにしましょう。 | #N/A | 
Hash#fetchのデフォルト値は評価するべき式に副作用があったり実行コストが高いときはうまくいかないので、代わりにブロックを使いましょう。 | #N/A | 
ハッシュから連続して複数の値が必要になる時は、Hash#values_atを用いましょう。 | #N/A | 
Ruby1.9以降、ハッシュは順序付けられるということを信頼しましょう。 | #N/A | 
コレクションを走査している時に変更を加えてはいけません。 | #N/A | 
コレクションにアクセスするとき、[n]の代替のリーダーメソッドが提供されている場合に直接[n]経由でアクセスすることは避けましょう。nilに対して[]を呼ぶことを避けることが出来ます。 | #N/A | 
コレクションに対するアクセサを提供するとき、コレクション内の要素にアクセスする前に、nilでアクセスするのを防ぐための代替のアクセス方法を提供しましょう。 | #N/A | 
文字列連結の代わりに文字列挿入や文字列整形を使いましょう。 | #N/A | 
文字列挿入時には、括弧の内部にスペースを入れるべきではありません。 | Style/SpaceInsideStringInterpolation: | ◯
文字列リテラルの引用符は一貫したスタイルで使いましょう。Rubyコミュニティでは、デフォルトでシングルクォートを用いるもの(OptionA)、ダブルクォートを用いるもの(OptionB)の二つのよく使われるスタイルがあって、どちらも良いと考えられています。 | Style/StringLiterals: | ◯
文字リテラル構文?xを用いてはいけません。Ruby1.9以降、この表記法を必要とする場面はないはずです-?xは'x'(１文字の文字列)と解釈されるからです。 | Style/CharacterLiteral: | ◯
文字列の中の挿入されるインスタンス変数やグローバル変数の周りの{}は省略してはいけません。 | Style/VariableInterpolation: | ◯
文字列に挿入するときにObject#to_sを使ってはいけません。自動的に呼び出されます。 | Lint/StringConversionInInterpolation: | ◯
大きなデータの塊を作る必要があるときは、String#+の使用は避けましょう。代わりに、String#<<を使いましょう。文字列連結は、文字列インスタンスを直接書き換えるため、たくさんの新しいオブジェクトを作ってしまうString#+よりも常に速いです。 | #N/A | 
利用するケースにより特化した速い代替手段がある場合、String#gsubは使わないようにしましょう。 | #N/A | 
複数行のヒアドキュメントを用いるときは、先頭のスペースも保持してしまうということを頭に入れておかなければなりません。過剰なスペースを取り除くためのマージンを採用するのはよい習慣です。 | #N/A | 
単に文字列中から文字列を探すだけの時は、正規表現を使ってはいけません:string['text']を使いましょう。 | #N/A | 
文字列の添字に直接正規表現を渡すことで、文字列の構築をシンプルにできます。 | #N/A | 
キャプチャした結果を使う必要のないときは、キャプチャしないグループを用いましょう。 | #N/A | 
最後に正規表現にマッチした値を示すPerlレガシーの暗号的な変数を用いてはいけません($1、$2など)。代わりにRegexp.last_match(n)を用いましょう。 | Style/PerlBackrefs: | ◯
どの値が入っているか追うのが困難になるので、グループ番号を使うのは避けましょう。代わりにグループに名前をつけましょう。 | #N/A | 
文字クラスの中では、特別な意味を持つ文字が少ないので注意が必要です:^、-、\、]のみが特別な意味を持つので、.や括弧を[]の中でエスケープしてはいけません。 | #N/A | 
^や$は、文字列の先頭や末尾ではなく、行頭や行末にマッチするので注意が必要です。もし文字列全体の先頭末尾にマッチさせたいときは、\A、\zを使いましょう(\n?\zと等価である\Zと混同しないようにしましょう)。 | #N/A | 
複雑な正規表現にはx識別子を用いましょう。これを用いることで、より読みやすくなり、便利なコメントを使えるようになります。スペースが無視されることに注意しましょう。 | #N/A | 
sub/gsubでの複雑な置換は、ブロックやハッシュを用いることで実現できます。 | #N/A | 
文字列挿入と"文字の双方が入る１行の文字列には、%()(%Q()の短縮形)を使いましょう。複数行の時はヒアドキュメントを使いましょう。 | Style/BarePercentLiterals: | ◯
文字列に'と"双方が含まれない限り、%qの使用は避けましょう。通常の文字列リテラルのほうがより読みやすいので、エスケープが大量に必要出ない限りは、そちらを使いましょう。 | Style/UnneededPercentQ: | ◯
/'が１つ以上の正規表現に限り、%rを使いましょう。 | Style/RegexpLiteral: | ◯
呼び出すコマンドにバッククォートが含まれる(かなり起こりえないが)ことがない限り、%xの使用は避けましょう。 | Style/CommandLiteral: | ◯
%sの使用は避けましょう。Rubyコミュニティは、スペース含むシンボルをを作る時は:"文字列"がよいと決めたようです。 | #N/A | 
パーセントリテラルの区切り文字は、%rを除いて()が好まれます。正規表現の中では、括弧は色々なシーンで使われるので、正規表現の内容によっては、より使われる機会の少ない{のほうが良い選択となることがあるかもしれません。 | Style/PercentLiteralDelimiters: | ◯
不要なメタプログラミングは避けましょう。 | #N/A | 
ライブラリを作成する時にコアクラスを汚染するのはやめましょう。(モンキーパッチを当ててはいけません)。 | #N/A | 
ブロック渡しのclass_evalのほうが、文字列挿入型よりも好ましいです。 | #N/A | 
文字列挿入型のclass_eval(または他のeval)を用いる時は、挿入されたときのコードをコメントに追加しましょう(Railsで使われているプラクティス)。 | #N/A | 
method_missingを用いたメタプログラミングは避けましょう。何故なら、バックトレースがよくわからなくなるし、#methodsのリストの中に出てこず、ミススペルしたメソッド呼び出しも無言で動いてしまいます、例えばnukes.launch_state=falseのようにです。代わりに、移譲、プロキシ、またはdefine_methodを使いましょう。もしmethod_missingを使わなければならない時は:／respond_to_missing?も実装されているか確かめましょう／既知の接頭辞、find_by_*のようなものだけを捕捉しましょう--可能な限りアサートさせましょう／最後にsuperを呼び出しましょう／アサートする、特別でないメソッドに移譲しましょう: | Style/MethodMissing: | ◯
private/protected制約を回避しないために、sendよりもpublic_sendを使いましょう。 | Style/Send: | 
sendは他の既存のメソッドと衝突するかもしれないので、__send__を使いましょう。 | #N/A | 
ruby-wで実行しても何も警告されないコードを書きましょう。 | #N/A | 
オプショナルな変数としてのハッシュの使用を避けましょう。そのメソッドはあまりにたくさんのことをやろうとしていませんか？(オブジェクトの初期化はこのルールの例外です) | #N/A | 
コードのある行が10行を超えるメソッドは避けましょう。理想を言えば、多くのメソッドは5行以内がよいです。空行は行数には含めません。 | Metrics/MethodLength: | ◯
３つや４つ以上引数を設定するのは避けましょう。 | Metrics/ParameterLists: | ◯
もし本当にグローバルなメソッドが必要な場合は、Kernelに定義し、privateに設定しましょう。 | #N/A | 
グローバル変数の代わりに、モジュールのインスタンス変数を使用しましょう | Style/GlobalVars: | ◯
複雑なコマンドラインオプションをパースするためにOptionParserを使いましょう。また、些細なオプションにはruby-sを使いましょう。 | #N/A | 
現在のシステム時間を読み出すには、Time.newよりもTime.nowを使いましょう。 | #N/A | 
破壊的変更をしなくても済むなら、できるだけ関数的プログラミング手法を使いましょう。 | #N/A | 
それがメソッドの目的でない限り、引数に破壊的変更をするのはやめましょう。 | #N/A | 
３段階を超えるブロックのネストは避けましょう。 | Metrics/BlockNesting: | ◯
一貫性を保ちましょう。理想を言えば、このガイドラインに沿いましょう。 | #N/A | 
常識を用いましょう。 | #N/A | 

もし、すでにある程度ポリシーが固まっている場合は、下記のコマンドを使って`.rubocop_todo.yml`を生成することができます。

``` sh
$ rubocop --auto-gen-config
Inspecting 1 file
C

Offenses:

sample.rb:2:1: C: Use 2 (not 4) spaces for indentation.
    bbbb
^^^^

1 file inspected, 1 offense detected
Created .rubocop_todo.yml.
Run `rubocop --config .rubocop_todo.yml`, or
add inherit_from: .rubocop_todo.yml in a .rubocop.yml file.
```

これはそのプロジェクトのコーディングポリシーが[config/default.yml](https://github.com/bbatsov/rubocop/blob/master/config/default.yml)からどれだけ違っているかが記述されているファイルです。現在のコードを規範として利用するときにはここで生成した`.rubocop_todo.yml`を`.rubocop.yml`として配布するか、実行時に下記のように指定することで、コーディングポリシーの統一を図ることができます。

``` sh
$ rubocop --config .rubocop_todo.yml
```

RuboCopによるコードの自動修正
--------

[RuboCop](https://github.com/bbatsov/rubocop)はコーディングポリシーに即していないコードについて、自動修正する機能があります。ただし、修正できる項目は上記で生成した`.rubocop_todo.yml`の内容として、`# Cop supports --auto-correct.`が付与されているものに限ります。

``` yaml
# Offense count: 1
# Cop supports --auto-correct.
# Configuration parameters: Width.
Style/IndentationWidth:
  Enabled: false
```

上記の記述が`.rubocop_todo.yml`に含まれていた場合は、[Ruby Style Guide](https://github.com/fortissimo1997/ruby-style-guide/blob/japanese/README.ja.md)の[インデントには スペース 2つ(別名ソフトタブ)。 ハードタブを用いてはいけません。](https://github.com/fortissimo1997/ruby-style-guide/blob/japanese/README.ja.md#spaces-indentation)に則していない項目があることを示しています。これに対して下記のコマンドで自動修正を行うことができます。

``` sh
$ rubocop --auto-correct
Inspecting 1 file
C

Offenses:

sample.rb:2:1: C: [Corrected] Use 2 (not 4) spaces for indentation.
    bbbb
^^^^

1 file inspected, 1 offense detected, 1 offense corrected
```

まとめ
--------

今回はRubyの記述に関するベストプラクティスである[Ruby Style Guide](https://github.com/fortissimo1997/ruby-style-guide/blob/japanese/README.ja.md)と、それを元にした静的解析ツールの[RuboCop](https://github.com/bbatsov/rubocop)、そしてその周辺ツール、コーディングポリシーのチューニング方法、コードの自動修正機能について紹介しました。

自分のコードの客観評価としても利用できるツールだと思いますし、導入も簡単なのでちょっと試してみてはいかがでしょう？




