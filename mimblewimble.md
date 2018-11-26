ミンブルウィンブル（むにゃむにゃ）<br>
トム　エルビス　ジェデューサー(訳: tanayuk)<br>
2016年7月19日

# 導入
Bitcoinは暗号的な合理性によって、  妥当性が検証された世界で最も普及した金融システムです。
しかしながら、これはBlockchainと呼ばれるパブリックなデータベースに全てのトランザクションを書き込むことで達成されており、なおかつこれが正真正銘の正な状態である、と検証しようとする人は全てをダウンロードし、すべての過去に起こったトランザクションを再検証しなければならない。一方で、それらのトランザクションの多くは最終の状態(打ち消し合うようなトランザクションが後に発生するため)にほとんど影響を与えないのである。

これを執筆している時点において、1億5,000万件のトランザクションがブロックチェーンに登録されており、たかだか400万程度の最終的な状態を表すために、登録されているトランザクションを全て再現する必要がある。

もし監査人がチェックしなければいけないデータが彼ら自身のものだけでであれば良いのであるが、それは出力が各々サインされたものの最終的なチェーンのの状態であること、というのが絶対条件であるため困難である。言い換えれば、Blockchain全体によってのみ最終的な状態が検証されなければならない、ということである。

それに加えこれらのトランザクションは暗号的には原子的で、アウトプットは各々のトランザクションとして書き込まれ、表層化することは明らかです。この”トランザクショングラフ”の結果は多くの情報を公開するとともに、多くの企業によって、事業モデルとその発注関係が監視される対象となります。このことが非プライベートで使う人にとってとても危険なものにしています。

これらの課題に対していくつかの解決策が提案されています。Greg Maxwellは取引量を暗号化する方法を模索し、これによりトランザクショングラフは表層するものの、総合計はが正しいことは検証可能にします[1]。Maxwell博士はBitcoinユーザ向けに対話的なトランザクションと複雑なトランザクションををもったシステムであるCoinJoinも制作しました。Nocolas van Saberhagenはトランザクションエントリーを秘匿化し、さらに複雑化したトランザクショングラフのシステムを開発しました(と同時に、ユーザのインタラクションも不要にしました)[3]。その後、Shen Noetherがこれら２つのアプローチを組み合わせ、Maxwellの"秘密のトランザクション"とvan Saberhagenの暗黒化を取り入れました[4]。

これらの解決法は非常によくできており、Bitcoinをとても安全に使えるものにする可能性があります。しかしながら、データが多くなる課題に対してはむしろ悪影響を及ぼします。秘密化されたトランザクションは数kバイトの証明を各アウトプットに付与する必要があり、van Saberhagenの署名にはそのアウトプットが本当に費消されたかどうかを証明するには、全てのアウトプットが一生保存されていることを必要としているためです。

Maxwell博士のCoinJoinはユーザのインタラクションを必要とする問題もあります。Yuan Horas Muton博士はトランザクションを自由にマージ可能とすることでこの問題を解決しました[5]が、それはペアリングベースの暗号を利用する必要がありました。これは潜在的により遅く、より信用し難いものになります。かれはこれを”一方方向集約署名”と名付けました(OWAS: One-way aggregate signatures)。

OWASはブロックにトランザクションを組み合わせる良いアイディアでした。想像してください、私達は各ブロックに渡って(おそらく糊のようなデータを使って)ブロックを組み合わせることができ、それによってアウトプットが作られたり壊された際に、それはそもそも存在しないことと同じことなのです。そして、全体のチェーンを検証する際、ユーザはいつお金がシステムに投入されたか(BitcoinやMonero、もしくはsidechainにPegされたような、各ブロックの新たなお金[6])、のみを知るだけで良く、最終的に費消されなかったアウトプットと残りは削除され忘れ去ることができます。そして、私達は取引量を隠した秘匿トランザクションが得られ、OWASがトランザクションをぼかし、Bitcoinよりも完全に検証可能なブロックチェーンをより少ない領域で使うことができるのです。さらに、ペアリングベースの暗号や新たな仮説が必要なく、Bitcoinのような通常の離散対数署名で済む、ということを想像してください。これが私の提案です。

この私の創作物をMimblewimbleと名付けます。なぜならば、これは全てのユーザの情報について語ることを防ぐように使われるからです[7]。

# 秘匿トランザクションとOWAS
まず我々が最初にしなければならないことは、Bitcoinのスクリプトを削除することです。これは悲しいことですが、少し強烈すぎて、一般的なスクリプトを使ってトランザクションをマージすることが不可能だからです。我々はMaxwell博士の秘匿トランザクションがアウトプットの費消の正当性とインタラクションなしでトランザクションを組み合わせることを評価するのに十分であることを実証します。これは実際OWASにとって理想的であり、リレーノードがトランザクションフィーを取得することや、受け取り手がトランザクションフィーを変更することを可能にします。これらの追加項目はBitcoinには決してできないことですが、我々は無料で利用することができるのです。

秘匿トランザクションがどのようにして実現されるのか、を読者に思い出してもらうことから始めます。まず、取引量は下記の数式で定義されます：

    C = r*G + v*H

ここで、`C`はPedersenの誓約、`G`と`H`はバックドアのない楕円曲線の組み合わせ生成、`v`は取引量、`r`は秘密のランダムなブラインド鍵です。

このアウトプットについて、`v`は`[0,2^64]`の範囲をとるものと定義され、これによって、ユーザはオーバーフローアタック等の攻撃ができないことになります。

トランザクションを検証するには、検証者は全てのアウトプットへの誓約、さらに`f*H`(`f`は自明で与えられるトランザクションフィー)と全ての入力誓約の減算を加えます。この結果は0のはずであり、これが無取引量もしくは全ての破棄、を証明するのです。

ここで着目するのは、このようなトランザクションを発行するには、ユーザーは全ての誓約エントリについての`r`の値の合計を知っておく必要があります。したがって、`r`の値(とその合計)が秘密鍵として働くのです。もし、`r`のアウトプットの値を受け取り手のみに公開すれば、我々は認証システムを持つことになるのです！残念ながら、もし我々が全ての誓約の合計が0というルールを守るのであれば、これは不可能なのですが、送信者は彼の全てのrの値の合計値を知っているおり、したがって受信者のrの値の合計は負の値となることを知っているからです。その代りに、我々は0でない値である`k*G`を合計することをトランザクションに許可し、空の文字列の署名を鍵として必要とし、その取引量部分が0であることを証明するのです。

我々はトランザクションに必要なだけの`k*G`の値を署名と、検証時にそれらの合計を持たせます。

トランザクションを作成するために、送信者と受信者は下記の儀式を行います。

1. 受信者と送信者は取引量についての合意を行います。これを`b`とします

2. Sender creates transaction with all inputs and change output(s), and gives
   recipient the total blinding factor (r-value of change minus r-values of
   inputs) along with this transaction. So the commitments sum to r*G - b*H.

3. Recipient chooses random r-values for his outputs, and values that sum
   to b minus fee, and adds these to transaction (including range proof).
   Now the commitments sum to k*G - fee*H for some k that only recipient
   knows.

4. Recipient attaches signature with k to the transaction, and the explicit
   fee. It has done.

Now, creating transactions in this manner supports OWAS already. To show this,
suppose we have two transactions that have a surplus k1*G and k2*G, and the
attached signatures with these. Then you can combine the lists of inputs and
outputs of the two transactions, with both k1*G and k2*G to the mix, and
voilÃ¡! is again a valid transaction. From the combination, it is impossible to
say which outputs or inputs are from which original transaction.

Because of this, we change our block format from Bitcoin to this information:

  1. Explicit amounts for new money (block subsidy or sidechain peg-ins) with
     whatever else data this needs. For a sidechain peg-in maybe it references
     a Bitcoin transaction that commits to a specific excess k*G value?

  2. Inputs of all transactions

  3. Outputs of all transactions

  4. Excess k*G values for all transactions

Each of these are grouped together because it do not matter what the transaction
boundaries are originally. In addition, Lists 2 3 and 4 should be required to be
coded in alphabetical order, since it is quick to check and prevents the block
creator of leaking any information about the original transactions.

Note that the outputs are now identified by their hash, and not by their position
in a transaction that could easily change. Therefore, it should be banned to have
two unspent outputs are equal at the same time, to avoid confusion.


# 複数ブロックに渡るトランザクションのマージ
さて、我々はMaxwell博士の秘匿トランザクションを用いて、インタラクション不要なバージョンの、Maxwell博士のCoinJoinを利用してきましたが、Maxwell博士の最後の奇跡をまだ見ていません。我々はまだ彼が[8]で説明したもう一つのアイディアであるトランザクションカットスルーが必要です。もう一度強調しますが、我々はインタラクションを必要としないバージョンを作り出し、それがいくつかのブロックでどのように使われるか、を示します。


We can imagine now each block as one large transaction. To validate it, we add all the
output commitments together, then subtracts all input commitments, k*G values, and all
explicit input amounts times H. We find that we could combine transactions from two
blocks, as we combined transactions to form a single block, and the result is again
a valid transaction. Except now, some output commitments have an input commitment exactly
equal to it, where the first block's output was spent in the second block. We could
remove both commitments and still have a valid transaction. In fact, there is not even
need to check the rangeproof of the deleted output.

The extension of this idea all the way from the genesis block to the latest block, we
see that EVERY nonexplicit input is deleted along with its referenced output. What
remains are only the unspent outputs, explicit input amounts and every k*G value.
And this whole mess can be validated as if it were one transaction: add all unspent
commitments output, subtract the values k*G, validate explicit input amounts (if there
is anything to validate) then subtract them times H. If the sum is 0, the entire
chain is good.

What is this mean? When a user starts up and downloads the chain he needs the following
data from each block:

  1. Explicit amounts for new money (block subsidy or sidechain peg-ins) with
     whatever else data this needs.

  2. Unspent outputs of all transactions, along with a merkle proof that each
     output appeared in the original block.

  3. Excess k*G values for all transactions.

Bitcoin today there are about 423000 blocks, totaling 80GB or so of data on the hard
drive to validate everything. These data are about 150 million transactions and 5 million
unspent nonconfidential outputs. Estimate how much space the number of transactions
take on a Mimblewimble chain. Each unspent output is around 3Kb for rangeproof and
Merkle proof. Each transaction also adds about 100 bytes: a k*G value and a signature.
The block headers and explicit amounts are negligible. Add this together and get
30Gb -- with a confidential transaction and obscured transaction graph!


# 質問と洞察
下記が直近数週間の間の質問と、夢で私に語りかけ汗まみれで起きたものです。
実際には問題はないですが。

Q. If you delete the transaction outputs, user cannot verify the rangeproof and maybe
   a negative amount is created.

A. This is OK. For the entire transaction to validate all negative amounts must have
   been destroyed. User have SPV security only that no illegal inflation happened in
   the past, but the user knows that _at this time_ no inflation occurred.


Q. If you delete the inputs, double spending can happen.

A. In fact, this means: maybe someone claims that some unspent output was spent in the old days. But this is impossible, otherwise the sum of the combined transaction could not be zero.

An exception is that if the outputs are amount zero, it is possible to make two that
are negatives of each other, and the pair can be revived without anything breaks. So to
prevent consensus problems, outputs 0-amount should be banned. Just add H at each output,
now they all amount to at least 1.


# 今後の調査
下記が現状私がこれを執筆段階で答えられない質問です。

1. What script support is possible? We would need to translate script operations into
   some sort of discrete logarithm information.

2. We require user to check all k*G values, when in fact all that is needed is that their
   sum is of the form k*G. Instead of using signatures is there another proof of discrete
   logarithm that could be combined?

3. There is a denial-of-service option when a user downloads the chain, the peer can give
   gigabytes of data and list the wrong unspent outputs. The user will see that the result
   do not add up to 0, but cannot tell where the problem is.

   For now maybe the user should just download the blockchain from a Torrent or something
   where the data is shared between many users and is reasonably likely to be correct.


[1] https://people.xiph.org/~greg/confidential_values.txt<br>
[2] https://bitcointalk.org/index.php?topic=279249.0<br>
[3] https://cryptonote.org/whitepaper.pdf<br>
[4] https://eprint.iacr.org/2015/1098.pdf<br>
[5] https://download.wpsoftware.net/bitcoin/wizardry/horasyuanmouton-owas.pdf<br>
[6] http://blockstream.com/sidechains.pdf<br>
[7] http://fr.harrypotter.wikia.com/wiki/SortilÃ¨ge_de_Langue_de_Plomb<br>
[8] https://bitcointalk.org/index.php?topic=281848.0
