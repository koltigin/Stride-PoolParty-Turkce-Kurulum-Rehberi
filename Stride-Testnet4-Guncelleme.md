# Stride Testnet 4 - PoolParty
![Stride-1](https://user-images.githubusercontent.com/102043225/180435619-36ea3c1e-410d-4abb-b7c0-ec21152bbcfb.png)

## Stride Kurulumu
```shell
sudo systemctl stop strided
cd $HOME && rm -rf stride
git clone https://github.com/Stride-Labs/stride.git
cd stride
git checkout cf4e7f2d4ffe2002997428dbb1c530614b85df1b
make build
sudo cp $HOME/stride/build/strided /usr/local/bin
```

## Genesis Dosyasının İndirilmesi
```shell
wget -qO $HOME/.stride/config/genesis.json "https://raw.githubusercontent.com/Stride-Labs/testnet/main/poolparty/genesis.json"
```

## SEED ve PEERS Ayarlanması
```shell
SEEDS="d2ec8f968e7977311965c1dbef21647369327a29@seedv2.poolparty.stridenet.co:26656"
PEERS="2771ec2eeac9224058d8075b21ad045711fe0ef0@34.135.129.186:26656,a3afae256ad780f873f85a0c377da5c8e9c28cb2@54.219.207.30:26656,328d459d21f82c759dda88b97ad56835c949d433@78.47.222.208:26639,bf57701e5e8a19c40a5135405d6757e5f0f9e6a3@143.244.186.222:16656,f93ce5616f45d6c20d061302519a5c2420e3475d@135.125.5.31:54356"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.stride/config/config.toml
```

## Değişkenlerin Ayarlanması
```shell
sed -i '/STRIDE_CHAIN_ID/d' ~/.bash_profile
echo "export STRIDE_CHAIN_ID=STRIDE-TESTNET-4" >> $HOME/.bash_profile
source $HOME/.bash_profile
strided config chain-id $STRIDE_CHAIN_ID
```

## Zincir Verilerini Sıfırlama
```shell
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1false|" $HOME/.stride/config/config.toml
strided tendermint unsafe-reset-all --home $HOME/.stride
```

## Node'u Başlatma
```shell
sudo systemctl start strided 
```

## Loglara Bakma
```shell
journalctl -fu strided -o cat
```

## Cüzdan Bakiyesini Kontrol Etme
```shell
strided query bank balances CUZDAN_ADRESINIZ --chain-id $CHAIN_ID
```  
Token yoksa discord'dan istiyoruz.

## Senkronizasyonu Kontrol Etme
`false` çıktısı almaldıkça bir sonraki yani validator oluşturma adımına geçmiyoruz.
```shell
strided status 2>&1 | jq .SyncInfo
```

## Validator Oluşturma
 Aşağıdaki komutta aşağıda berlittiğim yerler dışında bir değişikli yapmanız gerekmez;
   * `identity`  buraya `https://keybase.io` sitesine üye olarak size verilen kimlik numaranızı yazıyorsunuz.
   * `details` kendiniz hakkında bilgiler verebilir ya da `Rues Community Supporter` yazabilirsiniz.
   * `website`  Varsa bir siteniz yazabilirsiniz ya da `https://forum.rues.info` olarak bırakabilirsiniz.
   * `security-contact`  E-posta adresiniz.
 ```shell 
strided tx staking create-validator \
--amount=9900000ustrd \
--pubkey=$(strided tendermint show-validator) \
--moniker=$NODENAME \
--chain-id=$CHAIN_ID \
--commission-rate="0.10" \
--commission-max-rate="0.20" \
--commission-max-change-rate="0.1" \
--min-self-delegation="1" \
--fees=250ustrd \
--gas=200000 \
--from=$WALLET \
--details="Rues Community Supporter" \
--security-contact="E-POSTANIZ" \
--website="https://forum.rues.info" \
--identity="XXXX1111XXXX1111" \
-y
```

# Stride Testnet-4'e Hoş Geldiniz :)
