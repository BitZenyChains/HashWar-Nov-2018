
# analysis002 

あのん＠セキュリティVTuber

(https://twitter.com/anon_secchan)

## language

Japanese

## analysis 

コードベースで追っていくと、停止の直接の原因はvalidation.cpp の

```
ApplyTxInUndo()内 alternate.IsSpent() == true
```

になることによります.これは初めて不正トランザクションのspentチェックをここで行うからです.

それを呼び出しているコードを追うと、不正ブロックチェーンノードdebug.logからもわかるように

```
ApplyTxInUndo() 
<= DisconnectBlock()
<= DisconnectTip()
<= ActivateBestChain()
<= ProcessNewBlock()
```

となっています。

ここで不正ブロックチェーンノードの最後のブロックを見てみましょう

2018-11-06 21:32:56 UpdateTip: new best=0000001a345c04e31d6230d8fc204ee38a575f1e4211c8c97926e5f9d03e8801 height=1389866 version=0x20000000 log2_work=49.070766 tx=2762108 date='2018-11-06 21:32:48' progress=1.000000 cache=1.8MiB(6589txo)

不正ブロックチェーンはここでブロックが先端になっているわけです。
次に正規ブロックチェーンノードだった私が持っているdebug.logを見てみます

2018-11-06 21:34:05 UpdateTip: new best=00000001d60df1ec9a7190b63c3df2f937fdcfd32027df944a06d9e810755f61 height=1389765 version=0x20000000 log2_work=49.070768 tx=2761116 date='2018-11-06 21:33:58' progress=1.000000 cache=2.2MiB(14110txo)

よく見てみると、log2_workの値が正規ブロックチェーンの方が大きいんですね.
log2_workはchainworkの値であり、chainworkはそのブロックを作るまでに必要とした今までのブロックチェーンの仕事量を示しています

chainworkが上回るとReceivedBlockTransactions()にて正規ブロックがベストチェーンのブロック候補として選ばれ、正規チェーンのブロックが受け入れられるようになるのです。

正規チェーンのブロックを受け入れると、いよいよ自分のブロックチェーンの巻き戻りが始まります。FindMostWorkChain()にて自分がどこまで戻ればいいかの確認を行い、チェーンの分岐点である 1387408 まで戻ることを決めます。そして ActivateBestChainStep() にてひたすら自分のブロックチェーンの先端からの切り離し作業が行われます。

その過程で、最後に混入した不正ブロック1387624のトランザクションのチェックが行われ(先端から遡ってチェックが行われるため最初の不正ブロックではない)、

```
ApplyTxInUndo()内 alternate.IsSpent() == true 
```

となって、再構成が失敗します。

この時点でchainActive.Tip()が1387625になり、もうどの不正ブロックチェーンノードも最新ブロックを手に入れることができなくなるため、マイニングも停止し、不正ブロックチェーンが崩壊したということです

以上から、原因は、

1. 正規ブロックチェーンののchainworkが自分の持っているブロックチェーンのchainworkより大きかったためブロック再構成が走り、ブロック再構成の過程で不正トランザクションをチェックすることによって再構成に失敗した

1. ブロックチェーンの先端を手に入れることができなくなったので、不正ブロックチェーンノードのマイニングが止まり、不正ブロックチェーンが崩壊した

という結果でした。そのブロックを受け入れるかどうかが、実はチェーンの長さじゃなくてchainworkを見てるなんてコード見るまで知りませんでした。

巻き戻りが起きるだけならまだしも、脆弱性突かれたことで不正トランザクションが混ざり、それが原因でノードが一気に停止するのは今回くらいでしょうね。

## Related information

https://anon-secchan.hatenablog.com/entry/2018/11/28/035208
