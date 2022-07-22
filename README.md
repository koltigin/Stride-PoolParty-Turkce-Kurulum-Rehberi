# Stride Testnet - PoolParty: Türkçe Node Kurulum Rehberi
![Stride-1](https://user-images.githubusercontent.com/102043225/180435619-36ea3c1e-410d-4abb-b7c0-ec21152bbcfb.png)

##  Sistem Gereksinimleri
* 4vCPU
* 8GB RAM
* 200GB SSD

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
ver="1.18.3"
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
* '$NODENAME' validator adınız
* '$WALLET' cüzdan adınız
```shell
echo "export NODENAME=$NODENAME"  >> $HOME/.bash_profile
echo "export WALLET=$WALLET" >> $HOME/.bash_profile
echo "export STRIDE_PORT=16" >> $HOME/.bash_profile
echo "export CHAIN_ID=STRIDE-1" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Stride Kurulumu
```shell
cd $HOME
git clone https://github.com/Stride-Labs/stride.git
cd stride
git checkout c53f6c562d9d3e098aab5c27303f41ee055572cb
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
```

## Minimum GAS Ücretinin Ayarlanması
```shell
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0ustrd\"/" $HOME/.stride/config/app.toml
```

## SEED ve PEERS Ayarlanması
```shell
SEEDS="baee9ccc2496c2e3bebd54d369c3b788f9473be9@seedv1.poolparty.stridenet.co:26656"
PEERS=""
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
journalctl -u strided.service -f -n 100
```  

## Cüzdan Oluşturma

`$WALLET` bölümünü değiştirmiyoruz kurulumun başında cüzdanımıza isim belirledik.
```shell 
strided keys add $WALLET
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
strided status 2&1  jq .SyncInfo
```

## Validator Oluşturma
 Aşağıdaki komutta aşağıda berlittiğim yerler dışında bir değişikli yapmanız gerekmez;
   'identity'  buraya `httpskeybase.io` sitesine üye olarak size verilen kimlik numaranızı yazıyorsunuz.
   'details'  kendiniz hakkında bilgiler verebilir ya da `Rues Community Supporter` yazabilirsiniz.
   'website'  Varsa bir siteniz yazabilirsiniz ya da `httpsforum.rues.info` olarak bırakabilirsiniz.
   'security-contact'  E-posta adresiniz.
```shell 
strided tx staking create-validator 
 --commission-max-change-rate=0.01 
 --commission-max-rate=0.2 
 --commission-rate=0.05 
 --amount 9900000usdtr 
 --pubkey=$(strided  tendermint show-validator) 
 --moniker=$NODENAME 
 --chain-id=$CHAIN_ID 
 --details=Rues Community Supporter 
 --security-contact=E-POSTANIZ 
 --website=httpsforum.rues.info 
 --identity=XXXX1111XXXX1111 
 --min-self-delegation=1000000 
 --from=$WALLET
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
strided status 2&1  jq .SyncInfo
```

### Validator Bilgileri
```shell
strided status 2&1  jq .ValidatorInfo
```

### Node Bilgileri
```shell
strided status 2&1  jq .NodeInfo
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
echo $(stride tendermint show-node-id)@$(curl ifconfig.me)26656
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
strided tx bank send CUZDAN_ADRESI GONDERILECEK_CUZDAN_ADRESI 100000000usei
```

### Proposal Oylamasına Katılma
```shell
strided tx gov vote 1 yes --from $WALLET --chain-id=CHAIN_ID 
```

### Validatore Stake Etme  Delegate Etme
```shell
strided tx staking delegate $VALOPER_ADDRESS 100000000utoi --from=$WALLET --chain-id=C$HAIN_ID  --gas=auto
```

### Mevcut Validatorden Diğer Validatore Stake Etme  Redelegate Etme
```shell
strided tx staking redelegate MevcutValidatorAdresi StakeEdilecekYeniValidatorAdresi 100000000utori --from=WALLET --chain-id=CHAIN_ID  --gas=auto
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

### Hesaplar

[Linktree](https://linktr.ee.mehmetkoltigin)

[Twitter](https://twitter.com.mehmetkoltigin)

### Komunite 
[Forum Rues](https://forum.rues.infoindex.php)

[Telegram Rues Announcement](https://t.meRuesAnnouncement)

[Telegram Rues Chat](https://t.meRuesChat)

[Telegram Rues Node](https://t.meRuesNode)

[Telegram Rues Node Chat](https://t.meRuesNodeChat)
