# Stride Testnet 2 Güncellemesi
Güncellemeyi yapmadan önce aşağıdaki kodu çalıştırarak `155420`. bloğa gelip gelmediğinizi kontrol ediniz. `155420`. bloğa gelmeden güncelleme yapmayınız!
```shell
strided status 2>&1 | jq .SyncInfo
```
Yukarıdaki çıktı aşağıdaki resimdeki gibi olmalıdır.
![stride-guncelleme](https://user-images.githubusercontent.com/102043225/183877412-9d168200-afe6-4e10-9cdf-06efea168ae8.JPG)


```shell
sudo systemctl stop strided
cd $HOME
rm -rf stride
git clone https://github.com/Stride-Labs/stride.git && cd stride
git checkout 4ec1b0ca818561cef04f8e6df84069b14399590e 
make build
sudo cp $HOME/stride/build/strided /usr/local/bin
sudo systemctl restart strided && journalctl -fu strided -o cat
```
Kaynak:
https://github.com/Stride-Labs/testnet#upgrade
