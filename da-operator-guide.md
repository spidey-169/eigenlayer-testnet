# Eigenlayer DA node setup

"https://docs.eigenlayer.xyz/operator-guides/avs-installation-and-registration/eigenda-operator-guide/eigenda-avs-installation-registration-and-upgrade"


# Step 1: Install Prerequisites and Operator installation

Complete the EigenLayer CLI installation and registration.

## Operator Installation

## Node Operator Checklist

### Software Requirements

1. Install Docker, following the guide from this location: https://docs.docker.com/engine/install/ubuntu/
2. Install and ensure docker compose is properly configured: https://docs.docker.com/compose/install/linux/  
I am choosing install manually option here.

If Docker is installed and running, EigenLayer can be used within a Docker container, which provides a Linux environment.

## Installation

## A. Install CLI using Binary

To download a binary for latest release, run:

```
curl -sSfL https://raw.githubusercontent.com/layr-labs/eigenlayer-cli/master/scripts/install.sh | sh -s
```

You will get the following message

```
layr-labs/eigenlayer-cli info checking GitHub for latest tag
layr-labs/eigenlayer-cli info found version: 0.5.1 for linux/amd64
layr-labs/eigenlayer-cli info installed /home/spidey/bin/eigenlayer
```

The binary will be installed inside the ``~/bin`` directory.

To add the binary to your path, run:

```
export PATH=$PATH:~/bin
```

## B. Create and List Keys

ECDSA keypair corresponds to the operator Ethereum address and key for interacting with Eigenlayer. The BLS key is used for attestation purposes within the EigenLayer protocol. Only 1 BLS key can be registered per Operator entity in EigenLayer. Only 1 BLS key should be paired with only 1 ECDSA key and vice versa.

There is a fixed 1 to 1 relationship between the ECDSA key and the BLS key. You can't re-use any kind of key with any other kind of key. Once registered you can't change the BLS key associated with ECDSA key.

## Create Keys

Generate encrypted ECDSA and BLS keys using the CLI:

```
eigenlayer operator keys create --key-type ecdsa eigenlayer-da-testnet
eigenlayer operator keys create --key-type bls eigenlayer-da-testnet
```

Here eigenlayer-testnet is the [keyname] - This will be the name of the created key file. It will be saved as eigenlayer-da-testnet.ecdsa.key.json or eigenlayer-da-testnet.bls.key.json.

This will ask you for a password to encrypt the keys. Keys will be stored in a local disk and will be shown once keys are created. It will also show the private key only once, so that you can back it up in case you lose the password or key file.

Eth address: 0x866A3529d9d6ABc8D0d73433FCF0926105727c07

## You can also import existing keys but skipping this step as I am creating fresh keys for this da operator

## Export keys

If you want to see the private key of the existing keys, you can use the below command. This will only work if your keys are in default location (~/.eigenlayer/operator_keys)

```
eigenlayer operator keys export --key-type ecdsa eigenlayer-da-testnet
```

Here `eigenlayer-da-testnet` is the name of my key

It will show a message like
```
? This will show your private key. Are you sure you want to export? Yes
? Enter password to decrypt the key ************
```
And then your private key

## Fund ECDSA Wallet

Step 1: Follow steps to get gETH and 

Step 2: Send at least 1 Goerli ETH to the “address” field referenced in your operator-config.yaml file. This ETH will be used to cover the gas cost for operator registration in the subsequent steps.

## Operator Registration

# Registration

Configuration Setup

You can create the config files needed for operator registration using the below command

```
eigenlayer operator config create
```

Fill in the relevant details as it asks

```
? Would you like to populate the operator config file? Yes
? Enter your operator address: 0x866A3529d9d6ABc8D0d73433FCF0926105727c07
? Enter your earnings address (default to your operator address): 0x866A3529d9d6ABc8D0d73433FCF0926105727c07
? Enter your ETH rpc url: https://eth-goerli.g.alchemy.com/v2/mcgWcxYDGBae2j-cWwKXfzjkP2nNLTzP
? Enter your ecdsa key path: /home/spidey/.eigenlayer/operator_keys/eigenlayer-da-testnet.ecdsa.key.json
? Enter your bls key path: /home/spidey/.eigenlayer/operator_keys/eigenlayer-da-testnet.bls.key.json
? Select your network: goerli
Created operator.yaml and metadata.json files. Please fill in the smart contract configuration details(el_slasher_address and bls_public_key_compendium_address) provided by EigenLayer team. 
Please fill in the metadata.json file and upload it to a public url. Then update the operator.yaml file with the url (metadata_url).
Once you have filled in the operator.yaml file, you can register your operator using the configuration file.
```
For RPC I am using an alchemy Goerli eth RPC Endpoint.

This will create two file: operator.yaml and metadata.json After filling the details in metadata.json, please upload this into a publicly accessible location and fill that url in operator.yaml. A valid metadata url is required for successful registration. 


Fill the 'slasher contract address' and 'bls_public_key_compendium_address' as provided by eigenlayer for goerli testnet in operator.json

el_slasher_address: "0xD11d60b669Ecf7bE10329726043B3ac07B380C22"
bls_public_key_compendium_address: "0xc81d3963087Fe09316cd1E032457989C7aC91b19"

Also add the path to the 'metadata.json' accessible on a public repo (I have added this to my github)


## Registration commands

This is the command you can use to register your operator.

```
eigenlayer operator register operator.yaml
```

Please note: Note: ECDSA and BLS keys are required for operator registration. You may choose to either create your own set of keys using the EigenLayer CLI (recommended for first time users) or import your existing keys (recommended for advanced users who already have keys created) as outlined in the previous section.

Once registered you should get a message

``2024-01-13T08:25:50.679Z        INFO    logging/zap_logger.go:89        Operator is registered and bls key added successfully ✅``

You can check registration status using 

```
eigenlayer operator status operator.yaml
```

You will get an output something like
```
Operator configuration file read successfully 0x866A3529d9d6ABc8D0d73433FCF0926105727c07 ✅
xxxxxxxx
xxxxxxxxx
```


# Step 2 Prepare Local EigenDA files

Clone this repo and execute the following commands:

```
git clone https://github.com/Layr-Labs/eigenda-operator-setup.git
cd eigenda-operator-setup
cp .env.example .env
```

## Update .env file with your environment

Manually update the .env file downloaded in the steps above. Modify the sections marked with TODO to match your environment.

In my case I kept ``NODE_HOSTNAME``, ``NODE_NGINX_CONF_HOST`` unchanged

``
NODE_HOSTNAME=localhost
``

Change the ``USER_HOME`` environment variable to your directory where eigen-da repos are present

``
USER_HOME=/home/spidey
``

FOR ECDSA and BLS File path, I am using the paths for my file. 
``
NODE_ECDSA_KEY_FILE_HOST=${EIGENLAYER_HOME}/operator_keys/ecdsa_key.json
NODE_BLS_KEY_FILE_HOST=${EIGENLAYER_HOME}/operator_keys/bls_key.json
``

NOTE: make sure these are the same in the docker_compose.yml file

Add correct password for ``NODE_ECDSA/BLS_KEY_PASSWORD``

Create local folders which are required by EigenDA:

```
mkdir -p $HOME/.eigenlayer/eigenda/logs
mkdir -p $HOME/.eigenlayer/eigenda/db
```

# Step 3: Operator Networking Security Setup (Proxmox)

## Proxmox natting setting:

Since I am using proxmox, Add NODE_RETRIEVAL_PORT (32004) and NODE_DISPERSAL port (32005) to the natting configuration of the host to allow incoming connection to these ports from outside.

## Setup firewall
For port 32005, setup firewall to only allow connections from the following IPs, to prevent excessive node traffic

```
3.221.120.68/32
52.2.226.152/32
18.214.113.214/32
```

# Step 4: Opt-in into EigenDA

Make sure to delegate to your Operator from another address with stake 1.01 times the last operator.

For opting in execute

```
sudo ./run.sh opt-in
```

# Step 5: Run EigenDA

Before running docker, ensure the paths for ECDSA/BLS key are identical and correct, otherwise it will not sign blocks

```
    volumes:
      - ${NODE_ECDSA_KEY_FILE_HOST}:/app/operator_keys/ecdsa_key.json:readonly
      - ${NODE_BLS_KEY_FILE_HOST}:/app/operator_keys/bls_key.json:readonly
```

Execute docker-compose command to start EigenDA

```
docker compose up -d
```

To view logs you should use

```
docker logs -f <container_id>
```


# Upgrade your node
Upgrade the AVS software for your EigenDA service setup by following the steps below:

## Step 1: Pull the latest repo

```
cd eigenda-operator-setup
git pull
```

Update the MAIN_SERVICE_IMAGE in your .env file with the latest EigenDA version as per the release notes.

NOTE: If there are any specific instructions that needs to be followed for any upgrade, those instructions will be given with the release notes of the specific release. Please check the latest release notes on GitHub and follow the instructions before starting the services again.

## Step 2: Pull the latest docker images

```
docker compose pull
```

## Step 3: Stop the existing services

```
docker compose down
```

## Step 4: Start your services again
Make sure your .env file still has correct values in the TODO sections before you restart your node.

```
docker compose up -d
```

