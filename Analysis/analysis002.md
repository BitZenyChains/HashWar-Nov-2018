
# analysis002 

���̂񁗃Z�L�����e�BVTuber
(https://twitter.com/anon_secchan)

## language

Japanese

## analysis 

�R�[�h�x�[�X�Œǂ��Ă����ƁA��~�̒��ڂ̌����� validation.cpp ��ApplyTxInUndo()�� alternate.IsSpent() == true �ɂȂ邱�Ƃɂ��܂�
����͏��߂ĕs���g�����U�N�V������spent�`�F�b�N�������ōs������ł�
������Ăяo���Ă���R�[�h��ǂ��ƁA�s���u���b�N�`�F�[���m�[�hdebug.log������킩��悤��

ApplyTxInUndo() 
<= DisconnectBlock()
<= DisconnectTip()
<= ActivateBestChain()
<= ProcessNewBlock()

�ƂȂ��Ă��܂��B
�����ŕs���u���b�N�`�F�[���m�[�h�̍Ō�̃u���b�N�����Ă݂܂��傤

2018-11-06 21:32:56 UpdateTip: new best=0000001a345c04e31d6230d8fc204ee38a575f1e4211c8c97926e5f9d03e8801 height=1389866 version=0x20000000 log2_work=49.070766 tx=2762108 date='2018-11-06 21:32:48' progress=1.000000 cache=1.8MiB(6589txo)

�s���u���b�N�`�F�[���͂����Ńu���b�N����[�ɂȂ��Ă���킯�ł��B

���ɐ��K�u���b�N�`�F�[���m�[�h���������������Ă���debug.log�����Ă݂܂�

2018-11-06 21:34:05 UpdateTip: new best=00000001d60df1ec9a7190b63c3df2f937fdcfd32027df944a06d9e810755f61 height=1389765 version=0x20000000 log2_work=49.070768 tx=2761116 date='2018-11-06 21:33:58' progress=1.000000 cache=2.2MiB(14110txo)

�悭���Ă݂�ƁAlog2_work�̒l�����K�u���b�N�`�F�[���̕����傫����ł���
log2_work��chainwork�̒l�ł���Achainwork�͂��̃u���b�N�����܂łɕK�v�Ƃ������܂ł̃u���b�N�`�F�[���̎d���ʂ������Ă��܂�

ProcessNewBlock() �̏�������AcceptBlock()�����Ă݂܂��傤

    bool fHasMoreOrSameWork = (chainActive.Tip() ? pindex->nChainWork >= chainActive.Tip()->nChainWork : true);

    chainActive.Tip() �� true�Ƃ��āA����true�̂Ƃ���̒l
      pindex->nChainWork >= chainActive.Tip()->nChainWork �� false �ɂȂ�킯�ł�
      (pindex�͎󂯎�����u���b�N�AchainActive.Tip()�͌��ݎ������M���Ă���u���b�N�`�F�[���̐�[�������܂�)

����ɓǂݐi�߂Ă����ƁA�ȉ���������܂�
if (!fHasMoreOrSameWork) return true; // Don't process less-work chains

�܂�A�󂯎�����u���b�N��chainwork�������̃u���b�N�`�F�[����chainwork���t�]����ƁAAcceptBlock()��true�ɂȂ�킯�ł�
�i���Ȃ݂�AcceptBlock()��false����AcceptBlock FAILED�Ƃ����G���[���N���āA臒l�𒴂����BAN���܂�=>BAN�Ղ�̌����j

AcceptBlock()��true���Ƃ��悢�掩���̃u���b�N�`�F�[���̊����߂肪�n�܂�܂��B
ProcessNewBlock()�� �uif (!ActivateBestChain(state, chainparams, pblock))�v�̏����ɂ���Ƃ���A
ActivateBestChain�̏������łЂ����琳�K�u���b�N�̎󂯎��Ǝ����̃u���b�N�`�F�[���̐؂藣����Ƃ��s���܂��B

���̉ߒ��ŕs���u���b�N�̋N�_1387624�̃g�����U�N�V�����̃`�F�b�N���s���A
ApplyTxInUndo()�� alternate.IsSpent() == true �ƂȂ��āA�č\�������s���܂��B

���̎��_��chainActive.Tip()��1387625�ɂȂ�A�����ǂ̕s���u���b�N�`�F�[���m�[�h���ŐV�u���b�N����ɓ���邱�Ƃ��ł��Ȃ��Ȃ邽�߁A�}�C�j���O����~���A�s���u���b�N�`�F�[�������󂵂��Ƃ������Ƃł�

�ȏォ��A�����́A

1. ���K�u���b�N�`�F�[���̂�chainwork�������̎����Ă���u���b�N�`�F�[����chainwork���傫���������߃u���b�N�č\��������A�u���b�N�č\���̉ߒ��ŕs���g�����U�N�V�������`�F�b�N���邱�Ƃɂ���čč\���Ɏ��s�����v�A
2.�u���b�N�`�F�[���̐�[����ɓ���邱�Ƃ��ł��Ȃ��Ȃ����̂ŁA�s���u���b�N�`�F�[���m�[�h�̃}�C�j���O���~�܂�A�s���u���b�N�`�F�[�������󂵂�

�Ƃ������ʂł���
���̃u���b�N���󂯓���邩�ǂ������A���̓`�F�[���̒�������Ȃ���chainwork�����Ă�Ȃ�ăR�[�h����܂Œm��܂���ł����B

�����߂肪�N���邾���Ȃ�܂������A�Ǝ㐫�˂��ꂽ���Ƃŕs���g�����U�N�V������������A���ꂪ�����Ńm�[�h����C�ɒ�~����͍̂��񂭂炢�ł��傤�ˁB

