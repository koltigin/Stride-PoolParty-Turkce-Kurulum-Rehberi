# Stride Testnet - PoolParty: Türkçe Node Kurulum Rehberi (Testnet-4'e Güncellendi)
![Stride-1](https://user-images.githubusercontent.com/102043225/180435619-36ea3c1e-410d-4abb-b7c0-ec21152bbcfb.png)

##  Sistem Gereksinimleri
* 4vCPU
* 8GB RAM
* 200GB SSD

## Stride Testnet-2'ye Katılanlar için Testnet-4 Güncellemesi
Stride'ın ikinci testnetine katılıp validator oluşturduysanız aşağıdaki linkten devam ediniz.
* https://github.com/koltigin/Stride-PoolParty-Turkce-Kurulum-Rehberi/blob/main/Stride-Testnet4-Guncelleme.md

## Stride Testnet 4 - PoolParty - V2 Yazılım Güncellemesi
Testnet 4'te 70500. blokta gerçekleşen ilk güncellemedir. Aşağıdaki linkten ulaşabilirsiniz.
* https://github.com/koltigin/Stride-PoolParty-Turkce-Kurulum-Rehberi/blob/main/Stride-Testnet4-V2-Yazilim-Guncellemesi.md

## Sistemi Güncelleme
```shell
sudo apt update && sudo apt upgrade -y
```

## Gerekli Kütüphanelerin Kurulması
```shell
sudo apt install curl make build-essential gcc tmux jq chrony htop -y < "/dev/null"
```

## Go Kurulumu
```shell
ver="1.18.4"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
rm -rf /usr/local/go
tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm -rf "go$ver.linux-amd64.tar.gz"
echo 'export GOROOT=/usr/local/go' >> $HOME/.bash_profile
echo 'export GOPATH=$HOME/go' >> $HOME/.bash_profile
echo 'export GO111MODULE=on' >> $HOME/.bash_profile
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```

## Değişkenleri Yükleme
Aşağıda değiştirmeniz gereken yerleri yazıyorum.
* `$NODENAME` validator adınız
* `$WALLET` cüzdan adınız
```shell
echo "export NODENAME=$NODENAME"  >> $HOME/.bash_profile
echo "export WALLET=$WALLET" >> $HOME/.bash_profile
echo "export STRIDE_PORT=16" >> $HOME/.bash_profile
echo "export CHAIN_ID=STRIDE-TESTNET-4" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
* Düzenleyeceğiniz kod aşağıdakine benzer olacak. Anlamanız için faydalı olacaktır. 
```shell
echo "export NODENAME=Bilge"  >> $HOME/.bash_profile
echo "export WALLET=Bilge" >> $HOME/.bash_profile
echo "export STRIDE_PORT=16" >> $HOME/.bash_profile
echo "export CHAIN_ID=STRIDE-TESTNET-4" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Stride Kurulumu
```shell
cd $HOME
git clone https://github.com/Stride-Labs/stride.git
cd stride
git checkout cf4e7f2d4ffe2002997428dbb1c530614b85df1b
make build
cp $HOME/stride/build/strided /usr/local/bin
```

## Uygulamayı Yapılandırma
```shell
strided config chain-id $CHAIN_ID
strided config keyring-backend test
strided config node tcp://localhost:${STRIDE_PORT}657
```

## Uygulamayı Başlatma
```shell
strided init $NODENAME --chain-id $CHAIN_ID
```

## Genesis ve Addrbook Dosyalarının İndirilmesi
```shell
curl https://raw.githubusercontent.com/Stride-Labs/testnet/main/poolparty/genesis.json > ~/.stride/config/genesis.json
curl https://github.com/mmc6185/node-testnets/blob/main/stride/addrbook.json?raw=true > ~/.stride/config/addrbook.json
```

## Minimum GAS Ücretinin Ayarlanması
```shell
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0ustrd\"/" $HOME/.stride/config/app.toml
```

## SEED ve PEERS Ayarlanması
```shell
SEEDS="d2ec8f968e7977311965c1dbef21647369327a29@seedv2.poolparty.stridenet.co:26656"
PEERS="2771ec2eeac9224058d8075b21ad045711fe0ef0@34.135.129.186:26656,a3afae256ad780f873f85a0c377da5c8e9c28cb2@54.219.207.30:26656,328d459d21f82c759dda88b97ad56835c949d433@78.47.222.208:26639,bf57701e5e8a19c40a5135405d6757e5f0f9e6a3@143.244.186.222:16656,f93ce5616f45d6c20d061302519a5c2420e3475d@135.125.5.31:54356"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.stride/config/config.toml
```

## Prometheus'u Aktif Etme
```shell
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.stride/config/config.toml
```

## Pruning'i Ayarlama
```shell
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.stride/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.stride/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.stride/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.stride/config/app.toml
```

## Port Ayarlarını Yapılandırma
```shell
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${STRIDE_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${STRIDE_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${STRIDE_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${STRIDE_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${STRIDE_PORT}660\"%" $HOME/.stride/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${STRIDE_PORT}317\"%; s%^address = \":8080\"%address = \":${STRIDE_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${STRIDE_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${STRIDE_PORT}091\"%" $HOME/.stride/config/app.toml
```

## Zincir Verilerini Sıfırlama
```shell
strided tendermint unsafe-reset-all --home $HOME/.stride
```

## Servis Dosyası Oluşturma
```shell
tee <<EOF >/dev/null /etc/systemd/system/strided.service 
[Unit]
Description=stride
After=network-online.target

[Service]
User=$USER
ExecStart=$(which strided) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Servisi Başlatma
```shell
sudo systemctl daemon-reload
sudo systemctl enable strided
sudo systemctl restart strided
```

## Logları Kontrol Etme
```shell
journalctl -u strided -f -o cat
```  

## Hata Alırsanız
Loglarda aşağıdaki gibi bir hata alırsanız `ctrl c` yaptıktan sonra sonraki adımı uygulayınız.
```shell
strided.service: Main process exited, code=exited, status=1/FAILURE
strided.service: Failed with result 'exit-code'.
```

## PEER Sorunu Yaşarsanız
Aşağıdaki kod verileri temizler ve state sync yüklemesi yapar. State sync, blokların belli bir kısmımı karşıdan yükleyerek snaphot alınan bloklardan başlamasıdır. Yani bloklara sıfırdan başlamazsınız.
```shell 
sudo systemctl stop strided
cd $HOME && rm -rf stride
git clone https://github.com/Stride-Labs/stride.git
cd stride
git checkout 3cb77a79f74e0b797df5611674c3fbd000dfeaa1
make build
sudo cp $HOME/stride/build/strided /usr/local/bin
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1false|" $HOME/.stride/config/config.toml
strided tendermint unsafe-reset-all --home $HOME/.stride

SEEDS=""; \
PEERS="48b1310bc81deea3eb44173c5c26873c23565d33@34.135.129.186:26656,0f45eac9af97f4b60d12fcd9e14a114f0c085491@stride-library.poolparty.stridenet.co:26656 (http://,0f45eac9af97f4b60d12fcd9e14a114f0c085491@stride-library.poolparty.stridenet.co:26656/)"; \
sed -i.bak -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.stride/config/config.toml
wget -O $HOME/.stride/config/addrbook.json "https://github.com/mmc6185/node-testnets/blob/main/stride/addrbook.json?raw=true"
SNAP_RPC=https://stride-library.poolparty.stridenet.co:443
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.stride/config/config.toml
sudo systemctl restart strided && journalctl -u strided -f -o cat
```

### Yeni Cüzdan Oluşturma
`$WALLET` bölümünü değiştirmiyoruz kurulumun başında cüzdanımıza isim belirledik.
```shell 
strided keys add $WALLET
```  

### Var Olan Cüzdanı İçeri Aktarma
```shell
strided keys add $WALLET --recover
```  

* BU AŞAMADAN SONRA NODE'UMUZUN EŞLEŞMESİNİ BEKLİYORUZ.

## Faucet  Musluk
Test token almak için Discord'da [#token-faucet](https://discord.gg/HXgZSzstTV) kanalından şu şekilde `$faucet-stride:CUZDAN_ADRESINIZ` mesaj atıyoruz.

## Cüzdan Bakiyesini Kontrol Etme
```shell
strided query bank balances CUZDAN_ADRESINIZ --chain-id $CHAIN_ID
```  

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

## Validator Linkinizi Paylaşma
Sei Discord [#role-request](https://discord.gg/HXgZSzstTV) kanalından validatorumuze ait [explorer](https://stride.explorers.guru) linkini gönderiyoruz.

## DAHA FAZLA SORUNUZ VARSA STRIDE TÜRKİYE TELEGRAM GRUBU

[Stride Türkiye Telegram Sayfası](httpst.meTeritoriTurkish)

## FAYDALI KOMUTLAR

### Logları Kontrol Etme 
```shell
journalctl -fu strided -o cat
```

### Sistemi Başlatma
```shell
systemctl start strided
```

### Sistemi Durdurma
```shell
systemctl stop strided
```

### Sistemi Yeniden Başlatma
```shell
systemctl restart strided
```

### Node Senkronizasyon Durumu
```shell
strided status 2>&1 | jq .SyncInfo
```

### Validator Bilgileri
```shell
strided status 2>&1 | jq .ValidatorInfo
```

### Node Bilgileri
```shell
strided status 2>&1 | jq .NodeInfo
```

### Node ID Öğrenme
```shell
strided tendermint show-node-id
```

### Node IP Adresini Öğrenme
```shell
curl icanhazip.com
```

### Peer Adresinizi Öğrenme
```shell
echo $(strided tendermint show-node-id)@$(curl ifconfig.me)16656
```

### Cüzdanların Listesine Bakma
```shell
strided keys list
```

### Cüzdanı İçeri Aktarma
```shell
strided keys add $WALLET --recover
```

### Cüzdanı Silme
```shell
strided keys delete CUZDAN_ADI
```

### Cüzdan Bakiyesine Bakma
```shell
strided query bank balances CUZDAN_ADRESI
```

### Bir Cüzdandan Diğer Bir Cüzdana Transfer Yapma
```shell
strided tx bank send CUZDAN_ADRESI GONDERILECEK_CUZDAN_ADRESI 100000000ustrd
```

### Proposal Oylamasına Katılma
```shell
strided tx gov vote 1 yes --from $WALLET --chain-id=CHAIN_ID 
```

### Validatore Stake Etme  Delegate Etme
```shell
strided tx staking delegate $VALOPER_ADDRESS 100000000ustrd --from=$WALLET --chain-id=$CHAIN_ID  --gas=auto
```

### Mevcut Validatorden Diğer Validatore Stake Etme  Redelegate Etme
```shell
strided tx staking redelegate MevcutValidatorAdresi StakeEdilecekYeniValidatorAdresi 100000000ustrd --from=WALLET --chain-id=CHAIN_ID  --gas=auto
```

### Ödülleri Çekme
```shell
strided tx distribution withdraw-all-rewards --from=$WALLET --chain-id=CHAIN_ID  --gas=auto
```

### Komisyon Ödüllerini Çekme
```shell
strided tx distribution withdraw-rewards VALIDATOR_ADRESI --from=$WALLET --commission --chain-id=CHAIN_ID 
```

### Validator İsmini Değiştirme
```shell
strided tx staking edit-validator 
--moniker=YENI_NODE_ADI 
--chain-id=$CHAIN_ID  
--from=$WALLET
```

### Validatoru Jail Durumundan Kurtarma 
```shell
strided tx slashing unjail 
  --broadcast-mode=block 
  --from=$WALLET 
  --chain-id=$CHAIN_ID  
  --gas=auto
```

### Node'u Tamamen Silme 
```shell
sudo systemctl stop strided && 
sudo systemctl disable strided && 
rm etc/systemd/system/stride.service && 
systemctl daemon-reload && 
cd $HOME && 
rm -rf .stride stride && 
rm -rf $(which strided)
```

# Hesaplar

[Anatolian Team](https://anatolianteam.com)

[Twitter](https://twitter.commehmetkoltigin)

[Medium](https://medium.com/@mehmetkoltigin)

[YouTube](https://www.youtube.com/channel/UCmLgaftx5e38BE0E7gpY2dA)

[Discord](https://discordapp.com/users/837933958280904737)

[Telegram](https://t.me/mehmetkoltigin)
