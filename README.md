## Lava Network - создает уровень децентрализованной инфраструктуры для удаленных вызовов процедур блокчейна - peer-to-peer RPC сеть. Данное решение позволит Web3 компаниям устранить зависимость  от централизованных RPC нод, что уменьшит уязвимость и вероятность быть скомпрометированным.


### Форма для участия в тестнете - https://docs.google.com/forms/u/0/d/e/1FAIpQLSd-uRGAInJ6hhdt-354lW2sgY5O_o3m7aigU7-1f6egE1e5Zw/formResponse

### Форма для участия в тестнете (для разработчиков) - https://docs.google.com/forms/d/e/1FAIpQLSfEo8fZndjKlPchdDmPvXCdLESYfIUe8-PLC79syX5mo2aBbA/viewform



### Эксплоер
Тестнет эксплорер -https://bd.lavanet.xyz/


### Discord
https://discord.gg/yRaKrnpgQ4


- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet-  |   4|  8GB | 160GB    |

Аренда сервера под ноду  https://pq.hosting/?from=533917

# Обновляем пакеты
```python
sudo apt update && sudo apt upgrade -y
```
### Устанавливаем инструменты разработчика и необходимые пакеты
```python
sudo apt install curl build-essential pkg-config libssl-dev git wget jq make gcc tmux chrony -y
```

#### Устанавливаем GO
```python
ver="1.19.3" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```

# Установка ноды

### Важно!! Обновлять ноду только после 22 300 блока!!!
```python
cd $HOME
git clone https://github.com/lavanet/lava
cd lava
git fetch --all
git checkout v0.4.3
make install
lavad version --long | head
sudo systemctl restart lavad && sudo journalctl -u lavad -f -o cat
```

### Скачиваем бинарные файлы
```python
cd $HOME 
curl -L https://lava-binary-upgrades.s3.amazonaws.com/testnet/v0.4.0/lavad > lavad
chmod +x lavad
sudo mv lavad /usr/local/bin/lavad
```

### Проверяем версию
```python
lavad version
#0.4.0-rc2-e2c69db
```

### Создаем переменные
```python
MONIKER_LAVA=вводим свое имя
CHAIN_ID_LAVA=lava-testnet-1
PORT_LAVA=38
```

### Сохраняем переменные, перезагружаем .bash_profile и проверяем значения переменных
```python
echo "export MONIKER_LAVA="${MONIKER_LAVA}"" >> $HOME/.bash_profile
echo "export CHAIN_ID_LAVA="${CHAIN_ID_LAVA}"" >> $HOME/.bash_profile
echo "export PORT_LAVA="${PORT_LAVA}"" >> $HOME/.bash_profile
source $HOME/.bash_profile

echo -e "\nmoniker_LAVA > ${MONIKER_LAVA}.\n"
echo -e "\nchain_id_LAVA > ${CHAIN_ID_LAVA}.\n"
echo -e "\nport_LAVA > ${PORT_LAVA}.\n"
```

### Настраиваем конфиг
```python
lavad config chain-id $CHAIN_ID_LAVA
lavad config keyring-backend test
lavad config node tcp://localhost:${PORT_LAVA}657
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0025ulava\"/" $HOME/.lava/config/app.toml
```

### Инициализируем ноду
```python
lavad init $MONIKER_LAVA --chain-id $CHAIN_ID_LAVA
```

### Загружаем генезис файл и адресбук
```python
curl -s https://raw.githubusercontent.com/K433QLtr6RA9ExEq/GHFkqmTzpdNLDd6T/main/testnet-1/genesis_json/genesis.json > $HOME/.lava/config/genesis.json
curl -s https://snapshots1-testnet.nodejumper.io/lava-testnet/addrbook.json > $HOME/.lava/config/addrbook.json
```

### Изменяем порты для возможности дальнейшего подселения других нод проектов экосистемы Космос на один сервер
```python
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${PORT_LAVA}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${PORT_LAVA}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${PORT_LAVA}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${PORT_LAVA}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${PORT_LAVA}660\"%" $HOME/.lava/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${PORT_LAVA}317\"%; s%^address = \":8080\"%address = \":${PORT_LAVA}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${PORT_LAVA}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${PORT_LAVA}091\"%" $HOME/.lava/config/app.toml
```

### Настраиваем прунинг
```python
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.lava/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.lava/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.lava/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.lava/config/app.toml
```

### Сбрасываем данные
```python
lavad tendermint unsafe-reset-all --home $HOME/.lava
```

### Создаем сервисный файл
```python
printf "[Unit]
Description=Lava Service
After=network.target

[Service]
Type=simple
User=$USER
ExecStart=$(which lavad) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target" > /etc/systemd/system/lavad.service
```

### На 800-м блоке было обновление, поэтому на текущей версии с нуля синхронизироваться не получится. Поэтому воспользуемся снэпшотом от NodeJumper

Для ускорение синхронизации можно воспользоваться снэпшотом от NodeJumper

```python
SNAP_LAVA=$(curl -s https://snapshots1-testnet.nodejumper.io/lava-testnet/ | egrep -o ">lava-testnet-1.*\.tar.lz4" | tr -d ">")
curl https://snapshots1-testnet.nodejumper.io/lava-testnet/${SNAP_LAVA} | lz4 -dc - | tar -xf - -C $HOME/.lava
```

### Запускаем сервис и проверяем логи
```python
sudo systemctl daemon-reload && \
sudo systemctl enable lavad && \
sudo systemctl restart lavad && \
sudo journalctl -u lavad -f -o cat
```

### Ждем окончания синхронизации, проверить синхронизации можно командой
```python
lavad status 2>&1 | jq .SyncInfo

#Если вывод показывает false, синхронизация завершена
```

# Создание кошелька и валидатора

### Создаем кошелек
```python
lavad keys add $MONIKER_LAVA
```

### Сохраняем мнемоник фразу в надежном месте!
```python
Если вы участвовали в предыдущих тестнетах, восстанавливаем кошелек командой и вводим мнемоник фразу

lavad keys add $MONIKER_LAVA --recover
```

### Создаем переменную с адресом кошелька и валидатора
```python
WALLET_LAVA=$(lavad keys show $MONIKER_LAVA -a)
VALOPER_LAVA=$(lavad keys show $MONIKER_LAVA --bech val -a)


echo "export WALLET_LAVA="${WALLET_LAVA}"" >> $HOME/.bash_profile
echo "export VALOPER_LAVA="${VALOPER_LAVA}"" >> $HOME/.bash_profile
source $HOME/.bash_profile
echo -e "\nwallet_LAVA > ${WALLET_LAVA}.\n"
echo -e "\nvaloper_LAVA > ${VALOPER_LAVA}.\n"
```

### Запрашиваем токены с крана командой
```python
curl -X POST -d '{"address": "$(lavad keys show $MONIKER_LAVA -a)", "coins": ["10000000ulava"]}' https://faucet-api.lavanet.xyz/faucet/
```

### Проверяем свой баланс
```python
lavad q bank balances $WALLET_LAVA
```

### После завершения синхронизации и пополнения кошелька, создаем валидатора
```python
lavad tx staking create-validator \
--amount 900000ulava \
--from $WALLET_LAVA \
--commission-rate "0.09" \
--commission-max-rate "0.20" \
--commission-max-change-rate "0.1" \
--min-self-delegation "1" \
--pubkey=$(lavad tendermint show-validator) \
--moniker $MONIKER_LAVA \
--chain-id "lava-testnet-1" \
--fees=5000ulava \
--identity="" \
--details="I" \
--website="" \
-y
```

# Удаление ноды

Для удаления ноды используйте следующие команды
```python
sudo systemctl stop lavad
sudo systemctl disable lavad
sudo rm -rf $HOME/.lava
sudo rm -rf $HOME/lavad
sudo rm -rf /etc/systemd/system/lavad.service
sudo rm -rf /usr/local/bin/lavad
sudo systemctl daemon-reload
```






