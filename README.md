# Stafihub kurulum

Kurulum Adımları:

Önce yetkilendirme için hazırlık yapıyoruz ( Aşağıdaki kodları sırayla putty üzerinden sunucunuza girin)

cd $HOME
sudo apt update
sudo apt list --upgradable
sudo apt install make clang pkg-config libssl-dev build-essential git jq ncdu bsdmainutils -y < "/dev/null"




Şimdi Go kuruyoruz: ( Son komutta go versiyonu resimdeki gibi olmalı sarı ile işaretlediğim tek satır olarak girilecek)

GO: https://go.dev/dl/


cd $HOME
wget -O go1.18.2.linux-amd64.tar.gz https://golang.org/dl/go1.18.2.linux-amd64.tar.gz

1652656050393.png

rm -rf /usr/local/go && tar -C /usr/local -xzf go1.18.2.linux-amd64.tar.gz && rm go1.18.2.linux-amd64.tar.gz
echo 'export GOROOT=/usr/local/go' >> $HOME/.bashrc
echo 'export GOPATH=$HOME/go' >> $HOME/.bashrc
echo 'export GO111MODULE=on' >> $HOME/.bashrc
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bashrc && . $HOME/.bashrc
go version


Şimdi Git Deposunu Klonluyoruz: ( Güncel versiyonuna göre komutu düzenledim yeni güncelleme olursa tekrar güncelleme duyurusu iletiriz)

git clone --depth 1 --branch public-testnet-v2 https://github.com/stafihub/stafihub1652656236446.png

Şimdi Kurulum Yapıyoruz:

cd $HOME/stafihub && make install



Genesis İndiriyoruz ( Aşağıdaki YOUR_NODE_NAME yazan yere kendi Validatör isminizi girin)

stafihubd init sahstafi --chain-id stafihub-public-testnet-2

wget -O $HOME/.stafihub/config/genesis.json "https://raw.githubusercontent.com/stafihub/network/main/testnets/stafihub-public-testnet-2/genesis.json"

stafihubd tendermint unsafe-reset-all --home ~/.stafihub


Node yapılandırıyoruz :

sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.01ufis\"/" $HOME/.stafihub/config/app.toml
sed -i '/\[grpc\]/{:a;n;/enabled/s/false/true/;Ta};/\[api\]/{:a;n;/enable/s/false/true/;Ta;}' $HOME/.stafihub/config/app.toml
peers="4e2441c0a4663141bb6b2d0ea4bc3284171994b6@46.38.241.169:26656,79ffbd983ab6d47c270444f517edd37049ae4937@23.88.114.52:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.stafihub/config/config.toml

---------------------

Screen Kuruyoruz ve Screen içinde stafihub komutunu başlatacağız

screen -S stafi

Node arka planda çalıştırıyoruz:

stafihubd start

------------

Yada Node çalıştırmak için aşağıdaki hizmeti yükleyin (bunu tercih ediyorum)

echo "[Unit]
Description=StaFiHub Node
After=network.target

[Service]
User=$USER
Type=simple
ExecStart=$(which stafihubd) start
Restart=on-failure
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target" > $HOME/stafihubd.service
sudo mv $HOME/stafihubd.service /etc/systemd/system
sudo tee <<EOF >/dev/null /etc/systemd/journald.conf
Storage=persistent
EOF
sudo systemctl restart systemd-journald
sudo systemctl daemon-reload
sudo systemctl enable stafihubd
sudo systemctl restart stafihubd

