
# analysis002 

あのん＠セキュリティVTuber

(https://twitter.com/anon_secchan)

## language

Japanese

## analysis 

コードベースで追っていくと、停止の直接の原因は

validation.cpp のApplyTxInUndo()内 alternate.IsSpent() == true

になることによります.これは初めて不正トランザクションのspentチェックをここで行うからです.

それを呼び出しているコードを追うと、不正ブロックチェーンノードdebug.logからもわかるように

ApplyTxInUndo() 
<= DisconnectBlock()
<= DisconnectTip()
<= ActivateBestChain()
<= ProcessNewBlock()

となっています。

ここで不正ブロックチェーンノードの最後のブロックを見てみましょう

2018-11-06 21:32:56 UpdateTip: new best=0000001a345c04e31d6230d8fc204ee38a575f1e4211c8c97926e5f9d03e8801 height=1389866 version=0x20000000 log2_work=49.070766 tx=2762108 date='2018-11-06 21:32:48' progress=1.000000 cache=1.8MiB(6589txo)

不正ブロックチェーンはここでブロックが先端になっているわけです。

次に正規ブロックチェーンノードだった私が持っているdebug.logを見てみます

2018-11-06 21:34:05 UpdateTip: new best=00000001d60df1ec9a7190b63c3df2f937fdcfd32027df944a06d9e810755f61 height=1389765 version=0x20000000 log2_work=49.070768 tx=2761116 date='2018-11-06 21:33:58' progress=1.000000 cache=2.2MiB(14110txo)

よく見てみると、log2_workの値が正規ブロックチェーンの方が大きいんですね.
log2_workはchainworkの値であり、chainworkはそのブロックを作るまでに必要とした今までのブロックチェーンの仕事量を示しています

ProcessNewBlock() の処理中のAcceptBlock()を見てみましょう

```
bool fHasMoreOrSameWork = (chainActive.Tip() ? pindex->nChainWork >= chainActive.Tip()->nChainWork : true);
```

chainActive.Tip() は trueとして、そのtrueのところの値

pindex->nChainWork >= chainActive.Tip()->nChainWork が false になるわけです
(pindexは受け取ったブロック、chainActive.Tip()は現在自分が信じているブロックチェーンの先端を示します)

さらに読み進めていくと、以下が見つかります

```
if (!fHasMoreOrSameWork) return true; // Don't process less-work chains
```

つまり、受け取ったブロックのchainworkが自分のブロックチェーンのchainworkを逆転すると、AcceptBlock()がtrueになるわけです
（ちなみにAcceptBlock()がfalseだとAcceptBlock FAILEDというエラーが起きて、閾値を超えるとBANします=>BAN祭りの原因）

AcceptBlock()がtrueだといよいよ自分のブロックチェーンの巻き戻りが始まります。

ProcessNewBlock()の 「if (!ActivateBestChain(state, chainparams, pblock))」の処理にあるとおり、ActivateBestChainの処理内でひたすら正規ブロックの受け取りと自分のブロックチェーンの切り離し作業が行われます。

その過程で不正ブロックの起点1387624のトランザクションのチェックが行われ、ApplyTxInUndo()内 alternate.IsSpent() == true となって、再構成が失敗します。

この時点でchainActive.Tip()が1387625になり、もうどの不正ブロックチェーンノードも最新ブロックを手に入れることができなくなるため、マイニングも停止し、不正ブロックチェーンが崩壊したということです

以上から、原因は、

1. 正規ブロックチェーンののchainworkが自分の持っているブロックチェーンのchainworkより大きかったためブロック再構成が走り、ブロック再構成の過程で不正トランザクションをチェックすることによって再構成に失敗した」

1. ブロックチェーンの先端を手に入れることができなくなったので、不正ブロックチェーンノードのマイニングが止まり、不正ブロックチェーンが崩壊した

という結果でした。そのブロックを受け入れるかどうかが、実はチェーンの長さじゃなくてchainworkを見てるなんてコード見るまで知りませんでした。

巻き戻りが起きるだけならまだしも、脆弱性突かれたことで不正トランザクションが混ざり、それが原因でノードが一気に停止するのは今回くらいでしょうね。

