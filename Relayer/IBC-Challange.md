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

### Step 2.3: Initialise TS-Relayer

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

### Step 2.4: Initialise TS-Relayer

Inside the app.yaml you can see a mnemonic for 


