# Create a vm
```sh
(
more_memory=4096
name=cardano-node

curl -o preseed.cfg https://www.debian.org/releases/bullseye/example-preseed.txt
cat <<EOF >> preseed.cfg
d-i grub-installer/bootdev string default
d-i netcfg/get_hostname string debian11-$name
d-i passwd/root-login boolean false
d-i passwd/user-fullname string whs
d-i passwd/user-password password magic
d-i passwd/user-password-again password magic
d-i passwd/username string whs
d-i preseed/late_command string apt-install openssh-server
popularity-contest popularity-contest/participate boolean false
tasksel tasksel/first multiselect standard
EOF
virt-install \
  --disk size=20 \
  --initrd-inject preseed.cfg \
  --location https://deb.debian.org/debian/dists/bullseye/main/installer-amd64/ \
  --memory ${more_memory:-2048} \
  --name debian11-$name \
  --network bridge:virbr0 \
  --os-variant debian11 \
  --vcpus 2 \
  ;
) &
```

# Config nix
```sh
sudo mkdir -p /etc/nix/
cat <<EOF | sudo tee /etc/nix/nix.conf
experimental-features = nix-command flakes
substituters = https://cache.nixos.org https://hydra.iohk.io
trusted-public-keys = iohk.cachix.org-1:DpRUyj7h7V830dp/i6Nti+NEO2/nhblbov/8MW7Rqoo= hydra.iohk.io:f/Ea+s+dFdN+3Y/G+FDgSq+a5NEWhJGzdjvKNGv0/EQ= cache.nixos.org-1:6NCHdD59X431o0gWypbMrAURkbJ16ZPMQFGspcDShjY=
EOF
```

# Build
```sh
nix build .#testnet/node -o testnet-node-local
nix build .#cardano-cli -o cardano-cli-build
```

# Start cardano node
```sh
./testnet-node-local/bin/cardano-node-testnet
```

# Exports
```sh
export CARDANO_NODE_SOCKET_PATH=$HOME/cardano-node/state-node-testnet/node.socket
export PATH=$PATH:$HOME/cardano-node/cardano-cli-build/bin/
```

# Percentage
```sh
cardano-cli query tip --testnet-magic 1097911063
```

# Wallet 1
```sh
cardano-cli address key-gen \
  --signing-key-file 1.skey \
  --verification-key-file 1.vkey \
  ;
```

```sh
cardano-cli address build \
  --out-file 1.addr \
  --payment-verification-key-file 1.vkey \
  --testnet-magic 1097911063 \
  ;
```

```sh
cardano-cli query utxo \
  --address $(cat 1.addr) \
  --testnet-magic 1097911063 \
  ;
```

# Wallet 2
```sh
cardano-cli address key-gen \
  --signing-key-file 2.skey \
  --verification-key-file 2.vkey \
  ;
```

```sh
cardano-cli address build \
  --out-file 2.addr \
  --payment-verification-key-file 2.vkey \
  --testnet-magic 1097911063 \
  ;
```

```sh
cardano-cli query utxo \
  --address $(cat 2.addr) \
  --testnet-magic 1097911063 \
  ;
```
