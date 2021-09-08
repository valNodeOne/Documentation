# How to setup TS-Relayer for IBC between band-laozi-testnet4 and kichain-t-4

This document describes how the set up IBC between band-laozi-testnet4 and kichain-t-4. The Relayer used in this documentation is the typescript implementation of an [IBC Relayer.](https://docs.cosmos.network/master/ibc/relayer.html#)
A documentation can be found in the [ts-relayer](https://github.com/confio/ts-relayer) repo.

## Step 1: Confirm Parameters:

First of all we will check if both chains have ibc-transfer params enabled:

```bash=
kid q ibc-transfer params
```
```
Output:
receive_enabled: true
send_enabled: true
```

```bash=
bandd q ibc-transfer params
```
```
Output:
receive_enabled: true
send_enabled: true
```

## Step 2: Install TS-Relayer

### Step 2.1: Install Nodejs

```bash=
curl -sL https://deb.nodesource.com/setup_14.x | sudo bash -

sudo apt -y install nodejs
```

Check if your nodejs version i 14.16.1 or later:

```bash=
node  -v
```

### Step 2.2: Install TS-Relayer

Consider that the following command will install the latest release. This version will most likely not work with all Cosmos SDK Releases. Therefore check the compatibility at the [ts-relayer](https://github.com/confio/ts-relayer) repo.

```bash=
sudo npm i -g @confio/relayer
```

### Step 2.3: Initialise TS-Relayer configuration

Before initialising the Relayer append the following chain configurations to ```/home/<USERNAME>/.ibc-setup/registry.yaml ```

```bash=
  band:
    chain_id: band-laozi-testnet4
    prefix: band
    gas_price: 0.005uband
    ics20_port: 'transfer'
    rpc:
      - https://rpc.laozi-testnet4.bandchain.org
  KI:
    chain_id: kichain-t-4
    prefix: tki
    gas_price: 0.025utki
    ics20_port: 'transfer'
    rpc:
      - https://rpc-challenge.blockchain.ki:443
```

After editing the registry.yaml we can initiate the configuration:

```bash=
ibc-setup init --src KI --dest band
```
This will create the app.yaml file inside the .ibc-setup directory.

### Step 2.4: Fund Relayer Wallets

Inside the app.yaml you can see a mnemonic for the relayer wallets on both chains. You can change this mnemonic if you want.
To see the addresses to these wallets you can use the command:

```bash=
ibc-setup keys list
```
To fund the wallets I sent some funds from my validator wallets:

```bash=
bandd tx bank send <fromwallet> <relayerwallet> <amount> --chain-id band-laozi-testnet4
kid tx bank send <fromwallet> <relayerwallet> <amount> --chain-id kichain-t-4
```
If you have set up a validator node for band-laozi-testnet4 you can also use their faucet according to their [documentation](https://github.com/bandprotocol/launch/blob/master/band-laozi-testnet4/README.md).


### Step 2.5: Create Channel

After funding the relayer wallets we can create the ics20 channel:

```bash=
ibc-setup ics20 -v
```
The last lines of the Output will show the created channel:

```bash=
Output:
Created channel:
  laozi-testnet4: transfer/channel-0 (connection-0)
  kichain-t-4: transfer/channel-115 (connection-114)
```
### Step 2.6: Start Relayer

To start the relayer we will create a service:

```bash=
export USERNAME=$(whoami)
sudo -E bash -c 'cat << EOF > /etc/systemd/system/relayerd.service
[Unit]
Description=Relayer Daemon
After=network-online.target

[Service]
User=$USERNAME
ExecStart=/usr/bin/ibc-relayer start
Restart=always
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF'
```

Afterwards we will enable and start the service:
```bash=
sudo systemctl enable relayerd.service
sudo systemctl start relayerd.service
```

The logs of the service can be querried by running:

```bash=
journalctl -u relayerd.service -f
```

As seen in the logs, the relayer will now query both chains every 60 seconds and will check them for ibc transactions. If you want to change the 60 seconds waiting time, you can fork the ts-relayer and change it.

## Step 3: Verify Relayer

To verify the Relayer we will send some TKI from kichain-t-4 to band-laozi-testnet4.

The balance of the receiving wallet on band-laozi-testnet4 before the transaction only contains uband:
```bash=
bandd q bank balances <adress_band-laozi-testnet4>
```
```
Output:
balances:
- amount: "958750"
  denom: uband
```
After executing the ibc-transfer
```bash=
kid tx ibc-transfer transfer transfer channel-115 <adress_band-laozi-testnet4> 1000utki --from <adress_kichain-t-4> --chain-id=kichain-t-4 --packet-timeout-timestamp 0
```
we can see the following in the relayer log (shortened):
```
info: ... waking up and checking for packets!
...
info: Relay 1 packets from kichain-t-4 => band-laozi-testnet4
...
info: Relay 1 acks from band-laozi-testnet4 => kichain-t-4
```
After the packet was sent from kichain-t-4 to band-laozi-testnet4, the acknowledgement was sent from band-laozi-testnet4 to kichain-t-4.

As a result we can now see tki in the receiving wallet:

```
Output:
balances:
- amount: "1000"
  denom: ibc/FE393788C41E65BEB325ECA8A42FDB5C1676E7BDAC8AB5DB04E34547AC53B262
- amount: "958750"
  denom: uband
```
