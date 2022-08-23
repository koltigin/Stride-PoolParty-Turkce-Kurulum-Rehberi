# Stride Testnet 4 - PoolParty - V2 Yazılım Güncellemesi 
![Stride-1](https://user-images.githubusercontent.com/102043225/180435619-36ea3c1e-410d-4abb-b7c0-ec21152bbcfb.png)

## Uyarı
Güncellemeyi 70500. blooğa gelmeden yapmayınız.  Aşağıdaki kodu girerek kontrol ediniz.
```shell
strided status 2>&1 | jq .SyncInfo
```
Yukarıdaki kodun çıktısı aşağıdaki gibiyse güncellemeyi yapabilirsiniz. 
![stride-4-v2](https://user-images.githubusercontent.com/102043225/186232420-ba50b3c1-7d11-4945-9e8b-4a1cd3e58b4a.JPG)


## Stride Güncelleme
```shell
sudo systemctl stop strided
cd $HOME && rm -rf stride
git clone https://github.com/Stride-Labs/stride.git
cd stride
git checkout 90859d68d39b53333c303809ee0765add2e59dab
make build
sudo cp $HOME/stride/build/strided /usr/local/bin
sudo systemctl restart strided && journalctl -fu strided -o cat
```
