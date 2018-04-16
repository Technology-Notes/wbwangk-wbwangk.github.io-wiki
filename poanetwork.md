#### POA Network Whitepaper
hashguide/wiki   https://github.com/hashguide/wiki/wiki/POA-Network-Whitepaper

#### poanetwork/wiki
https://github.com/poanetwork/wiki/wiki/What-is-POA

#### Oracles Network
https://hackmd.io/s/HkV8Vw7_-

#### Introducing POA Network
https://medium.com/poa-network/introducing-oracles-network-864d1d7e37e2

#### The Issuance Model in Ethereum
https://blog.ethereum.org/2014/04/10/the-issuance-model-in-ethereum/

### 交易费分析
结论：POA网络的燃料价格是1GWEI（即0.000000001ETH），质量链可以用这个当作默认燃料价格。

#### 以太坊交易费
随便找了一个以太坊交易进行分析。[这个](https://etherscan.io/tx/0xfeda61d100b9552c33f381c4a57ac9fc7f1c838b8cf412b9a15e1aa97d50bcc1)交易从外部账户发往一个合约账户。

 - Gas Limit: 225000  
 - Gas Used By Txn: 185103  
 - Gas Price: 0.0000000022 Ether (2.2 Gwei)  
 - Actual Tx Cost/Fee: 0.0004072266 Ether ($0.21)  

计算过程：0.0004072266 = 0.0000000022 * 185103

#### POA网络交易费
随便找了一个POA网络交易进行分析。[这个](https://poaexplorer.com/tx/0x9541cb8160ba01911910733776c22a12e312db8533cfd69e056e2b93117d2c79)是个转账交易。

- Gas:	21000  
- Gas Price:	0.000000001 POA (1 GWEI)  
- Actual Tx Cost/Fee:	0.000021  

计算过程：0.000021 = 0.000000001 * 21000