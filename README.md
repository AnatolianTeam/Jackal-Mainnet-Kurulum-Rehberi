# Jackal Mainnet Kurulum Rehberi

![Jackal-GitHub](https://github.com/AnatolianTeam/Jackal-Mainnet-Kurulum-Rehberi/assets/102043225/a461c20b-bfc5-44cc-b2ee-ff8a2b1ceadd)

## Bağlantılar
 ✔️ [Website](https://jackalprotocol.com/)<br>
 ✔️ [Blockchain Explorer](https://ping.pub/jackal)<br>
 ✔️ [Doküman](https://docs.jackalprotocol.com/)<br>
 ✔️ [GitHub](https://github.com/JackalLabs/)<br>
 ✔️ [Discord](https://discord.com/invite/5GKym3p6rj)

## Gereksinimler 
| Bileşenler | Minimum Gereksinimler | **Tavsiye Edilen Gereksinimler** | 
| ------------ | ------------ | ------------ |
| CPU |	4 | 4 |
| RAM	| 16 GB | 64 GB |
| Storage	| 500 GB SSD | 1 TB SSD |

## Sistemi Güncelleme
```shell
apt update && apt upgrade -y
```

## Gerekli Kütüphanelerin Kurulması
```shell
apt install make clang pkg-config libssl-dev libclang-dev build-essential git curl ntp jq llvm tmux htop screen gcc lz4 -y < "/dev/null"
```

## Go Kurulumu
```shell
ver="1.20"
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
aşağıda değiştirmeniz gereken yerleri yazıyorum.
* `$JACKAL_NODENAME` validator adınız
* `$JACKAL_WALLET` cüzdan adınız
*  Eğer portu başka bir node kullanıyorsa aşağıdan değiştirebilirsiniz.
```shell
echo "export JACKAL_NODENAME=$JACKAL_NODENAME"  >> $HOME/.bash_profile
echo "export JACKAL_WALLET=$JACKAL_WALLET" >> $HOME/.bash_profile
echo "export JACKAL_PORT=11" >> $HOME/.bash_profile
echo "export JACKAL_CHAIN_ID=jackal-1" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Örnek
Node ve Cüzdan adımızın `Mehmet` olduğunu varsayalım. Kod aşağıdaki şekilde düzenlenecektir. 
```shell
echo "export JACKAL_NODENAME=Mehmet"  >> $HOME/.bash_profile
echo "export JACKAL_WALLET=Mehmet" >> $HOME/.bash_profile
echo "export JACKAL_PORT=11" >> $HOME/.bash_profile
echo "export JACKAL_CHAIN_ID=babajaga-1" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Jackal'ın Kurulması
```shell
cd || return
rm -rf canine-chain
git clone https://github.com/JackalLabs/canine-chain.git
cd canine-chain || return
git checkout v2.0.0
make install
canined version # 2.0.0
```
Versiyon çıktısı `2.0.0` olacak.

## Uygulamayı Yapılandırma ve Başlatma
```shell
canined config chain-id $JACKAL_CHAIN_ID
canined init --chain-id $JACKAL_CHAIN_ID $JACKAL_NODENAME
```

## Genesis ve addrbook Dosyasının Kopyalanması
```shell
curl -s https://raw.githubusercontent.com/JackalLabs/canine-mainnet-genesis/main/genesis/genesis.json > $HOME/.canine/config/genesis.json
curl -s https://snapshots1.nodejumper.io/jackal/addrbook.json > $HOME/.canine/config/addrbook.json

```

## Minimum GAS Ücretinin Ayarlanması
```shell
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.025ujkl"|g' $HOME/.canine/config/app.toml
```

## Indexer'i Kapatma (Opsiyonel)
```shell
indexer="null"
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.canine/config/config.toml
```

## SEED ve PEERS Ayarlanması
```shell
SEEDS=""
PEERS=""
sed -i 's|^seeds *=.*|seeds = "'$SEEDS'"|; s|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.canine/config/config.toml
```

## Prometheus'u Aktif Etme
```shell
sed -i 's|^prometheus *=.*|prometheus = true|' $HOME/.canine/config/config.toml
```

## Pruning'i Ayarlama
```shell
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
snapshot_interval="0"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.canine/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.canine/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.canine/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.canine/config/app.toml
sed -i -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" $HOME/.canine/config/app.toml

```

## Portları Ayarlama
```shell
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${JACKAL_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${JACKAL_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${JACKAL_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${JACKAL_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${JACKAL_PORT}660\"%" $HOME/.canine/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${JACKAL_PORT}317\"%; s%^address = \":8080\"%address = \":${JACKAL_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${JACKAL_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${JACKAL_PORT}091\"%" $HOME/.canine/config/app.toml
sed -i.bak -e "s%^node = \"tcp://localhost:26657\"%node = \"tcp://localhost:${JACKAL_PORT}657\"%" $HOME/.canine/config/client.toml
```

## Servis Dosyası Oluşturma
```shell
tee /etc/systemd/system/canined.service > /dev/null << EOF
[Unit]
Description=Jackal Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which canined) start
Restart=on-failure
RestartSec=10
LimitNOFILE=10000
[Install]
WantedBy=multi-user.target
EOF
```

## Snapshot Yükleme (NodeJumper)
```shell
canined tendermint unsafe-reset-all --home $HOME/.canine --keep-addr-book
SNAP_NAME=$(curl -s https://snapshots1.nodejumper.io/jackal/info.json | jq -r .fileName)
curl "https://snapshots1.nodejumper.io/jackal/${SNAP_NAME}" | lz4 -dc - | tar -xf - -C "$HOME/.canine"
```


## Servisi Başlatma ve Logları Kontrol Etme
```shell
systemctl daemon-reload
systemctl enable canined
systemctl start canined
journalctl -u canined -f -o cat
```  

## Cüzdan Oluşturma

### Yeni Cüzdan Oluşturma
`$JACKAL_WALLET` bölümünü değiştirmiyoruz kurulumun başında cüzdanımıza değişkenler ile isim belirledik.
```shell 
canined keys add $JACKAL_WALLET
```  

### Var Olan Cüzdanı İçeri Aktarma
```shell
canined keys add $JACKAL_WALLET --recover
```

## Cüzdan ve Valoper Bilgileri
Burada cüzdan ve valoper bilgilerimizi değişkene ekliyoruz.

```shell
JACKAL_WALLET_ADDRESS=$(canined keys show $JACKAL_WALLET -a)
JACKAL_VALOPER_ADDRESS=$(canined keys show $JACKAL_WALLET --bech val -a)
```

```shell
echo 'export JACKAL_WALLET_ADDRESS='${JACKAL_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export JACKAL_VALOPER_ADDRESS='${JACKAL_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```

🔴 **BU AŞAMADAN SONRA NODE'UMUZUN EŞLEŞMESİNİ BEKLİYORUZ.**

## Senkronizasyonu Kontrol Etme
`false` çıktısı almadıkça bir sonraki yani validator oluşturma adımına geçmiyoruz.
```shell
canined status 2>&1 | jq .SyncInfo
```

🔴 **Eşleşme tamamlandıysa aşağıdaki adıma geçiyoruz.**


## Validator Oluşturma
 Aşağıdaki komutta aşağıda berlirttiğim yerler dışında bir değişiklik yapmanız gerekmez;
   - `identity`  burada `XXXX1111XXXX1111` yazan yere [keybase](https://keybase.io/) sitesine üye olarak size verilen kimlik numaranızı yazıyorsunuz.
   - `details` `Always forward with the Anatolian Team 🚀` yazan yere kendiniz hakkında bilgiler yazabilirsiniz.
   - `website`  `https://anatolianteam.com` yazan yere varsa bir siteniz ya da twitter vb. adresinizi yazabilirsiniz.
   - `security-contact`  E-posta adresiniz.
 ```shell 
canined tx staking create-validator \
--amount=1000000ujkl \
--pubkey=$(canined tendermint show-validator) \
--moniker=$JACKAL_NODENAME \
--chain-id=$JACKAL_CHAIN_ID \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.05 \
--min-self-delegation="1" \
--gas-prices=0.1ujkl \
--gas-adjustment=1.5 \
--gas=auto \
--from=$JACKAL_WALLET \
--details="Always forward with the Anatolian Team 🚀" \
--security-contact="xxxxxxx@gmail.com" \
--website="https://anatolianteam.com" \
--identity="XXXX1111XXXX1111" \
--yes
```

## YARARLI KOMUTLAR

## Logları Kontrol Etme 
```
journalctl -fu canined -o cat
```

### Sistemi Başlatma

```
systemctl start canined
```

### Sistemi Durdurma
```
systemctl stop canined
```

### Sistemi Yeniden Başlatma
```
systemctl restart canined
```

### Node Senkronizasyon Durumu
```
canined status 2>&1 | jq .SyncInfo
```
```
curl -s localhost:26657/status | jq .result.sync_info
```

### Validator Bilgileri
```
canined status 2>&1 | jq .ValidatorInfo
```

### Node Bilgileri

```
canined status 2>&1 | jq .NodeInfo
```

### Node ID Öğrenme

```
canined tendermint show-node-id
```


### Node IP Adresini Öğrenme

```
curl icanhazip.com
```

### Cüzdanların Listesine Bakma

```
canined keys list
```

### Cüzdan Adresini Görme

```
canined keys show $JACKAL_WALLET --bech val -a
```

### Cüzdanı İçeri Aktarma

```
canined keys add $JACKAL_WALLET --recover
```

### Cüzdanı Silme

```
canined keys delete $JACKAL_WALLET
```

### Cüzdan Bakiyesine Bakma
```
canined query bank balances $JACKAL_WALLET_ADDRESS
```

### Bir Cüzdandan Diğer Bir Cüzdana Transfer Yapma

```
canined tx bank send $JACKAL_WALLET_ADDRESS GONDERILECEK_CUZDAN_ADRESI 100000000ujkl
```

### Proposal Oylamasına Katılma
```
canined tx gov vote 1 yes --from $JACKAL_WALLET --chain-id=$JACKAL_CHAIN_ID
```


### Validatore Stake Etme / Delegate Etme

```
canined tx staking delegate $VALOPER_ADDRESS 100000000ujkl --from=$JACKAL_WALLET --chain-id=$JACKAL_CHAIN_ID --gas=auto --fees 5000ujkl
```

### Mevcut Validatorden Diğer Validatore Stake Etme / Redelegate Etme
<srcValidatorAddress>: Mevcut Stake edilen validatorün adresi
<destValidatorAddress>: Yeni stake edilecek validatorün adresi 
```
canined tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 100000000ujkl --from=$JACKAL_WALLET --chain-id=$JACKAL_CHAIN_ID --gas=auto
```

### Ödülleri Çekme

```
canined tx distribution withdraw-all-rewards --from=$JACKAL_WALLET --chain-id=$JACKAL_CHAIN_ID --gas=auto
```

### Komisyon Ödüllerini Çekme

```
canined tx distribution withdraw-rewards $VALOPER_ADDRESS --from=$JACKAL_WALLET --commission --chain-id=$JACKAL_CHAIN_ID
```

### Validator İsmini Değiştirme
NEWNODENAME yazan yere yeni validator/moniker isminizi yazınız. TR karakçer içermemelidir.

```
canined tx staking edit-validator \
--moniker=NEWNODENAME \
--chain-id=$JACKAL_CHAIN_ID \
--from=$JACKAL_WALLET
```

### Validator Komisyon Oranını Degiştirme
```
canined tx staking edit-validator --commission-rate "0.02" --moniker=$JACKAL_NODENAME --chain-id=$JACKAL_CHAIN_ID --from=$JACKAL_WALLET
```

### Validator Bilgilerinizi Düzenleme
Bu bilgileri değiştirmeden önce https://keybase.io/ adresine kayıt olarak aşağıdaki kodda görüldüğü gibi 16 haneli (XXXX0000XXXX0000) kodunuzu almalısınız. Ayrıca profil resmi vs. ayarları da yapabilirsiniz. 
$NODENAME: Yeni node adınız (moniker)
$JACKAL_WALLET: Cüzdan adınız, değiştirmeniz gerekmez. Değişkenlere ekledik çünkü.
```
canined tx staking edit-validator \
--moniker=$NODENAME \
--identity=XXXX0000XXXX0000 \
--website="VARSA WEBSITENIZI YAZABILIRSINIZ" \
--details="BU BOLUME KENDINIZI TANITAN BIR CUMLE YAZABILIRSINIZ" \
--chain-id=$JACKAL_CHAIN_ID \
--from=$JACKAL_WALLET
```

### Validatoru Jail Durumundan Kurtarma 

```
canined tx slashing unjail \
  --broadcast-mode=block \
  --from=$JACKAL_WALLET \
  --chain-id=$JACKAL_CHAIN_ID \
  --gas=auto
  --gas-adjustment=1.4
```

### Node'u Tamamen Silme 

```
systemctl stop canined && \
systemctl disable canined && \
rm /etc/systemd/system/canined.service && \
systemctl daemon-reload && \
cd $HOME && \
rm -rf .canine canine-chain && \
rm -rf $(which canined) \
sed -i '/JACKAL_/d' ~/.bash_profile
```


# Hesaplar:

[Anatolian Team](https://anatolianteam.com)

[Twitter](https://twitter.com/anatolianteam)

[YouTube](https://www.youtube.com/@anatolianteam)

[Medium](https://medium.com/@anatolianteam)

[Telegram](https://t.me/anatolianteam)
