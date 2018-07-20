# Running a Lightning node

# this tutorial is provided as learning purpose
Please only run, this in public internet, if you know what you are doing

# requirements
* mainnet 250go
* testnet 25go
* debian stretch
* bitcoin - v0.16.1
* lightningd - v0.6

# basis

## install some tools
apt update && apt upgrade
apt-get install git build-essential htop -y

## linux
here we use debian. There is no perfect distribution. Debian as a good security focus and stable environment
when interacting with this tutoial. Your default user should not be - root.

## firewall
* blocks all ports except SSH/bitcoind(8333)/lightningd
* temporary open git/apt/80/443

Excellent tutorial are available in the web.

# first - bitcoin

## impl
Bitcoin as 3 majors implementation for server
* bitcoind maintain by bitcoin core developers and bitcoin developers
* btcd maintain by The btcsuite developers
* libbitcoin

we use bitcoind

## deps version
https://github.com/bitcoin/bitcoin/blob/master/doc/dependencies.md

## install required package
apt update
apt install autoconf libboost-all-dev libssl-dev libtool libevent-dev libminiupnpc-dev pkg-config -y

# Tell your system where to find, for example db4.8
## building libdb
wget 'http://download.oracle.com/berkeley-db/db-4.8.30.NC.tar.gz'
tar -xzvf db-4.8.30.NC.tar.gz
cd db-4.8.30.NC/build_unix/
../dist/configure --prefix=/usr/local --enable-cxx
make
sudo make install

## protocol buffer
### install via apt should be v3.0.0
apt-get install libprotobuf-dev protobuf-compiler
or
### if you want exact version
git clone https://github.com/google/protobuf.git
git checkout v2.6.1
git submodule update --init --recursive
./autogen.sh
./configure
make
make check
sudo make install
sudo ldconfig # refresh shared library cache.

## exports for bitcoind compilation
$ export BDB_INCLUDE_PATH="/usr/local/BerkeleyDB.4.8/include"
$ export BDB_LIB_PATH="/usr/local/BerkeleyDB.4.8/lib"
$ ln -s /usr/local/BerkeleyDB.4.8/lib/libdb-4.8.so /usr/local/lib/libdb-4.8.so

# compiling bitcoind
```bash
git clone https://github.com/bitcoin/bitcoin.git
git checkout v0.16.1
./autogen.sh
./configure CXXFLAGS="--param ggc-min-expand=1 --param ggc-min-heapsize=32768" --enable-cxx --disable-shared --with-pic --without-gui --prefix=/usr/local/ LDFLAGS="-L$BDB_LIB_PATH -L/usr/local/lib -L." CPPFLAGS="-I$BDB_INCLUDE_PATH -I/include/google/protobuf"
make
sudo make install
```

without => self explaining
enable-cxx => because client have c++ code
disable shared => we don't want binary use shared library, for security reason
with-pic => position independant code
prefix => where the binary is installed

## create user
adduser bitcoin

## create your datadir
target a place with sufficient space, here we just do it in /opt
```bash
mkdir -p /opt/bitcoin-data
chown bitcoin:bitcoin /opt/bitcoin-data
```

## dropin your bitcoin.conf

look at bitcoin.conf
rpc access is not setup.
make sure you have rpc/zmq inteface and txindex=1

### generate rpc access
```bash
$./share/rpcauth/rpcauth.py {{login-name}}
```
this script will gave you rpc access line to replace, the sample rpcauth line in bitcoin.conf
and give you password to be used to connect to rpc socket

### launch bitcoin daemon - bitcoind
as user bitcoin, launch bitcoin daemon

```bash
su bitcoin
bitcoind -datadir=/opt/bitcoin-data
```
or
```bash
bitcoind -datadir=/opt/bitcoin-data -testnet=1
```
for testnet


### look at logs - debug.log

```bash
tail -f /opt/bitcoin-data/debug.log
```

### interacting with bitcoin command line interface - bitcoin-cli

```bash
bitcoin-cli -datadir=/opt/bitcoin-data getblockcount
```

# second - Lightning

## new deps to install

sudo apt-get install -y \
  libgmp-dev \
  libsqlite3-dev python python3 net-tools zlib1g-dev

## first we clone the project

```bash
git clone https://github.com/ElementsProject/lightning.git
git checkout v0.6
```

```bash
make
sudo make install
```

chown -R bitcoin:bitcoin /opt/clightning/

# run lightningd in testnet

```bash
su bitcoin
/usr/local/bin/lightningd --pid-file=/run/clightningd/lightningd.pid \
           --bitcoin-cli=/usr/local/bin/bitcoin-cli --bitcoin-datadir=/opt/bitcoin-data \
           --addr="{{ip_addr}}:9735" --alias={{super_alias}} \
           --lightning-dir=/opt/clightning/ --log-level=debug --network=testnet
```

# interacting with lightning-cli

```bash
lightning-cli --lightning-dir=/opt/bitcoin-data getinfo
```



# what next ?
* systemd
* tor
* rate limiting
* lightning charge
* enjoy
