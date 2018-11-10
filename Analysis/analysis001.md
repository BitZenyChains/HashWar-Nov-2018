
# analysis001

WO(A member of the Koto Developers)
https://github.com/WO01

## language

Japanese

## analysis 

どのチェーンを優先するかはブロック高ではなく，ブロックのchainwork値を利用するので（最近見直して知った）．

ちょうど，その時刻に正規チェーンのブロック
UpdateTip: new best=00000001d60df1ec9a7190b63c3df2f937fdcfd32027df944a06d9e810755f61 height=1389765 version=0x20000000 log2_work=49.070768 tx=2761116 date='2018-11-06 21:33:58' 
が不正チェーンの
UpdateTip: new best=0000001a345c04e31d6230d8fc204ee38a575f1e4211c8c97926e5f9d03e8801 height=1389866 version=0x20000000 log2_work=49.070766 tx=2762108 date='2018-11-06 21:32:48' 
のchainworkを追い越したので巻き戻しかかってます．

chainworkはbits(difficulty)の積み重ねなので，ハッシュレートが高いほど高い値になっていくようです．
ここの表示ではlog2_workがchainwork値です（logとってますが）

## Related information

https://bitcoin.stackexchange.com/questions/26869/what-is-chainwork

