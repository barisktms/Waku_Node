# Waku_Node

Donanım
Firma olarak Hetzner kullanıyorum

2 CPU 2 RAM - 40 SSD

Kurulum

# Sunucu güncellemesi ve gerekli paketler
sudo apt update && sudo apt upgrade -y
sudo apt-get install build-essential git libpq5 jq -y

# kodu girdikten sonra 1 seçelim.
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source "$HOME/.cargo/env"

sudo apt install docker.io -y
sudo curl -L "https://github.com/docker/compose/releases/download/v2.24.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Waku kurulumu

# wakuyu klonluyoruz
git clone https://github.com/waku-org/nwaku-compose
cd nwaku-compose
cp .env.example .env

# .env içine giriyoruz bu komutla
nano .env
Değiştireceğimiz kısımlar bunlar aşağıya yazıyorum:

ETH_CLIENT_ADDRESS = Infura'dan Sepolia RPC aldım bedava onu ekledim - https://www.infura.io/
ETH_TESTNET_KEY = Waku için açtığım metamaskın private keyini ekledim - Metamask hesap bilgileri kısmında
RLN_RELAY_CRED_PASSWORD = Bir şifre belirledim
CTRL X Y ENTER ile kaydedip çıkıyoruz.

# ve register edelim.
./register_rln.sh

# register etmeden önce cüzdanda sepolia eth olmalı

Waku node başlatma

# portları açma

# yes diyelim bu komutu girdikten sonra
sudo ufw enable

# bu port komutlarını toplu girebilirsiniz.
sudo ufw allow 22    
sudo ufw allow 3000   
sudo ufw allow 8545   
sudo ufw allow 8645   
sudo ufw allow 9005   
sudo ufw allow 30304  
sudo ufw allow 8645

# docker up edelim
docker-compose up -d

# bu komutlar ile kontrol edebilirsin logları
docker-compose ps
docker-compose logs nwaku

# bu komut ile içersine girelim:
nano ~/nwaku-compose/docker-compose.yml

içersine girdikten sonra ctrl w yaparak 3000:3000 yazıp aratalım

127.0.0.1:3000:3000 olanı 0.0.0.0:3000:3000 şeklinde değişelim.

CTRL X Y ENTER ile kaydedip çıkalım.

# tekrar başlatalım
docker-compose down
docker-compose up -d

Yaklaşık 1 saat içersine verileriniz güncellenecek

http://IP_ADRESİN:3000/d/yns_4vFVk/nwaku-monitoring

IP_ADRESİN kısmını değiştirip google'da aratın.

Yedekleme için keystore.json dosyasını kaydedin.

# Bu iki komutlada çıktı alıyorsanız sorun yoktur.
curl --location 'http://127.0.0.1:8645/debug/v1/version'
curl --location 'http://127.0.0.1:8645/debug/v1/info'

Güncelleme

Öncelikle indirdiğimiz nwaku-compose reposunun içerisine gidip, çalışan contianerları durduruyoruz.

cd ~/nwaku-compose
docker-compose down

Reponun son sürümünü çekiyoruz (Güncellemenin main branchte olduğunu varsayıyoruz. Versiyon 0.25.0'dan 0.26.0'a geçerken main üzerinden güncellediler.)

git pull

Dockerı yeniden ayağa kaldırıyoruz.

docker-compose up -d

Logları ve Grafana'yı kontrol etmeyi unutmayınız. Güncelledikten sonra kapatıp yeniden açmayı denerseniz hata ile karşılaşıyorsunuz. Manual olarak postgres:15.4-alpine3.18 containerı içerisine girip public.messages_backup tablosunu silmeniz gerekiyor. Bu sorun ile ilgili bağlantı.

Port değiştirme

Çalışan diğer node'lar da bazı portlar kullanıyor. Bende 8080 port sıkıntı oldu ve aşağıda anlattığım şekilde sorunu çözdüm. Sizler diğer portları da değiştirebilirsiniz.

nwaku-compose klasörüne giriyoruz

cd nwaku-compose
docker-compose.yml dosyasını açıyoruz

nano docker-compose.yml
Dosyanın içinde Ctrl+w ile

127.0.0.1
yazıp arama yapıyoruz.

Sorun yaşadığınız portu değiştirip;

tekrar başlatalım
docker-compose down
docker-compose up -d
