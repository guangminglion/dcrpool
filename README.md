# dcrpool

[![Build Status](https://github.com/decred/dcrpool/workflows/Build%20and%20Test/badge.svg)](https://github.com/decred/dcrpool/actions)
[![ISC License](https://img.shields.io/badge/license-ISC-blue.svg)](http://copyfree.org)
[![Go Report Card](https://goreportcard.com/badge/github.com/decred/dcrpool)](https://goreportcard.com/report/github.com/decred/dcrpool)

## Overview

dcrpool is a stratum decred mining pool. It currently supports:

* Obelisk DCR1 (supported firmware: [obelisk-sc1-v1.3.2.img](https://mining.obelisk.tech/downloads/firmware/obelisk-sc1-v1.3.2.img))
* Innosilicon D9 (supported firmware: [d9_20190521_071217.swu](http://www.innosilicon.com.cn/download/d9_20190521_071217.swu))
* Antminer DR3 (supported firmware: [Antminer-DR3-201907161805-410M.tar.gz](https://file12.bitmain.com/shop-product/firmware/Antminer%20DR3/Firmware/007201907271437364778LxDsS1k06AF/Antminer-DR3-201907161805-410M.tar.gz))
* Antminer DR5 (supported firmware: [Antminer-DR5-201907161801-600M.tar.gz](https://file12.bitmain.com/shop-product/firmware/Antminer%20DR5/Firmware/00720190727142534231Ato7d2300650/Antminer-DR5-201907161801-600M.tar.gz))
* Whatsminer D1 (supported firmware: [upgrade-whatsminer-h6-20190404.18.zip](https://github.com/decred/dcrpool/files/5651882/upgrade-whatsminer-h6-20190404.18.zip))

The default port all supported miners connect to the pool via is `:5550`. 
The pool can be configured to mine in solo pool mode or as a publicly available 
mining pool.  Solo pool mode represents a private mining pool operation where 
all connected miners to the pool are owned by the pool administrator.  For this 
reason, mining rewards are left to accumulate at the specified address for the 
mining node. There isn't a need for payment processing in solo pool mining mode, 
it is disabled as a result.

In solo pool mining mode, miners only need to identify themselves when 
connecting to the pool. The miner's username, specifically the username sent 
in a `mining.authorize` message should be a unique name identifying the client.

The pool supports Pay Per Share (`PPS`) and Pay Per Last N Shares (`PPLNS`) 
payment schemes when configured for pool mining. With pool mining, mining 
clients connect to the pool, contribute work towards solving a block and 
claim shares for participation. When a block is found by the pool, portions of 
the mining reward due participating accounts are calculated based on claimed 
shares and the payment scheme used. The pool pays out the mining reward 
portions due each participating account when it matures.

In addition to identifying itself to the pool, each connecting miner has to 
specify the address its portion of the mining reward should be sent to when a 
block is found. For this reason, the mining client's username is a combination 
of the address mining rewards are paid to and its name, formatted as: 
`address.name`. This username format for pool mining is required. The pool uses 
the address provided in the username to create an account, all other connected 
miners with the same address set will contribute work to that account.  

The user interface of the pool provides public access to statistics and pool 
account data. Users of the pool can access all payments, mined blocks by the 
account and also work contributed by clients of the account via the interface. 
The interface is only accessible via HTTPS and by default uses a self-signed 
certificate, served on port `:8080`. In production, particularly for pool 
mining, a certificate from an authority (`CA`) like 
[letsencrypt](https://letsencrypt.org/) is recommended.

## Installing and Updating

Building or updating from source requires the following build dependencies:

- **Go 1.16 or later**

  Installation instructions can be found here: <https://golang.org/doc/install>.
  It is recommended to add `$GOPATH/bin` to your `PATH` at this point.

- **Git**

  Installation instructions can be found at <https://git-scm.com> or
  <https://gitforwindows.org>.

To build and install from a checked-out repo or a copy of the latest release,
run `go install . ./cmd/...` in the root directory.
The `dcrpool` executable will be installed to `$GOPATH/bin`.  `GOPATH` defaults
to `$HOME/go` (or `%USERPROFILE%\go` on Windows) if unset.

## Database

dcrpool can run with either a [Bolt database](https://github.com/etcd-io/bbolt)
or a [Postgres database](https://www.postgresql.org/). Bolt is used by default.
[postgres.md](./docs/postgres.md) has more details about running with Postgres.

When running in Bolt mode, the pool maintains a backup of the database
(`backup.kv`), created on shutdown in the same directory as the database itself.
The user interface also provides functionality for pool administrators to backup
Bolt database when necessary.

### Example of obtaining and building from source on Ubuntu

```sh
git clone https://github.com/decred/dcrpool.git
cd dcrpool
go install
dcrpool --configfile=path/to/config.conf
```

## Configuration

dcrpool requires [dcrd](https://github.com/decred/dcrd) and [dcrwallet](https://github.com/decred/dcrwallet) when configured as a mining pool, it only requires dcrd when configured as a solo pool.
Deploying the user interface requires copying the `dcrpool/gui/assets` folder from 
source to a reachable location and updating the gui directory (`--guidir`) of 
the configuration. Currently only single instance deployments are supported, 
support for distributed deployments will be implemented in the future.

### Example of a solo pool configuration

```no-highlight
rpcuser=user
rpcpass=pass
dcrdrpchost=127.0.0.1:19556
dcrdrpccert=/home/.dcrd/rpc.cert
solopool=true
activenet=mainnet
adminpass=adminpass
guidir=/home/gui
```

The configuration above uses a [Bolt database](https://github.com/etcd-io/bbolt). 
To switch to a [Postgres database](https://www.postgresql.org/) additional config 
options will be needed, refer to [postgres.md](./docs/postgres.md). 

### Example output of a solo pool startup

```no-highlight
dcrpool --configfile=pool.conf --appdata=/tmp/dcrpool-harness/pool
2020-12-22 20:10:31.120 [INF] POOL: Maximum work submission generation time at pool difficulty is 28s.
2020-12-22 20:10:31.129 [INF] POOL: Solo pool mode active.
2020-12-22 20:10:31.149 [INF] MP: Version: 1.1.0+dev
2020-12-22 20:10:31.149 [INF] MP: Runtime: Go version go1.15.6
2020-12-22 20:10:31.149 [INF] MP: Home dir: /tmp/dcrpool-harness/pool
2020-12-22 20:10:31.149 [INF] MP: Started dcrpool.
2020-12-22 20:10:31.149 [INF] GUI: Starting GUI server on port 8080 (https)
2020-12-22 20:10:31.149 [INF] MP: Creating profiling server listening on 127.0.0.1:6060
2020-12-22 20:10:31.150 [INF] POOL: listening on :5550
```

### Example of a mining pool configuration

```no-highlight
rpcuser=user
rpcpass=pass
dcrdrpchost=127.0.0.1:19556
dcrdrpccert=/home/.dcrd/rpc.cert
walletgrpchost=127.0.0.1:19558
walletrpccert=/home/.dcrwallet/rpc.cert
maxgentime=20s
solopool=false
activenet=simnet
walletpass=walletpass
poolfeeaddrs=SsVPfV8yoMu7AvF5fGjxTGmQ57pGkaY6n8z
paymentmethod=pplns
lastnperiod=5m
adminpass=adminpass
guidir=/home/gui
```

The configuration above uses a [Bolt database](https://github.com/etcd-io/bbolt). 
To switch to a [Postgres database](https://www.postgresql.org/) additional config 
options will be needed, refer to [postgres.md](./docs/postgres.md).

### Example output of a mining pool startup

```no-highlight
dcrpool --configfile=pool.conf --appdata=/tmp/dcrpool-harness/pool
2020-12-22 19:57:45.795 [INF] POOL: Maximum work submission generation time at pool difficulty is 20s.
2020-12-22 19:57:45.816 [INF] POOL: Payment method is PPLNS.
2020-12-22 19:57:45.916 [INF] MP: Version: 1.1.0+dev
2020-12-22 19:57:45.916 [INF] MP: Runtime: Go version go1.15.6
2020-12-22 19:57:45.916 [INF] MP: Creating profiling server listening on 127.0.0.1:6060
2020-12-22 19:57:45.916 [INF] MP: Home dir: /tmp/dcrpool-harness/pool
2020-12-22 19:57:45.917 [INF] MP: Started dcrpool.
2020-12-22 19:57:45.917 [INF] GUI: Starting GUI server on port 8080 (https)
2020-12-22 19:57:45.932 [INF] POOL: listening on :5550
```

Refer to [config descriptions](config.go) for more detail. 


## Wallet accounts

In mining pool mode the ideal wallet setup is to have two wallet accounts, 
the pool account and the fee account, for the mining pool. This account structure 
separates revenue earned from pool operations from mining rewards gotten on 
behalf of participating clients. The pool account's purpose is to receive 
mining rewards of the pool. The addresses generated from it should be the mining 
addresses (`--miningaddr`) set for the mining node. It is important to set 
multiple mining addresses for the mining node in production to make it 
difficult for third-parties wanting to track coinbases mined by the pool and 
ultimately determine the cumulative value of coinbases mined. 

The fee account's purpose is to receive pool fees of the mining pool. It is 
important to set multiple pool fee addresses to the mining pool in production to 
make it difficult for third-parties wanting to track pool fees collected by 
the pool and ultimately determine the cumulative value accrued by pool operators. 
With multiple pool fee addresses set, the mining pool picks one at random for 
every payout made. The addresses generated from the fee account should be 
set as the pool fee addresses (`--poolfeeaddrs`) of the mining pool.

## Wallet Client Authentication

dcrwallet v1.6 requires client authentication certificates to be provided 
on startup via the client CA file config option (`--clientcafile`). 
Since dcrpool is expected to maintain a grpc connection to the wallet it needs 
to generate the needed certificate before the wallet is started. A config 
option (`--gencertsonly`) which allows the generation of all key pairs without 
starting the pool has been added for this purpose. Pool operators running a 
publicly available mining pool will be required to first run their pools 
with `--gencertsonly` to generate required key pairs before configuring their 
pool wallets and starting them. The test harness, [harness.sh](./harness.sh), 
provides a detailed example for reference. 

## Pool Fees

In mining pool mode pool fees collected by the pool operator are for 
maintaining a connection to the decred network for the delivery of work 
to mining clients, the submission of solved work by mining clients to the 
network and processing of block rewards based on work contributed by 
participting accounts.

## Transaction fees

Every mature group of payments plus the pool fees collected completely 
exhaust the referenced coinbases being sourced from by payments. For this 
reason payout transactions by the pool create no change. It is implicit 
that the transaction fees of the payout transaction are paid for by the 
accounts receiving payouts, since transaction fees are not collected as part 
of pool fees. Each account receiving a payout from the transaction pays a 
portion of the transaction fee based on value of the payout in comparison to 
the overall value of payouts being made by the transaction. 

## Dust payments

Dust payments generated by participating accounts of the pool are forfeited 
to the pool fees paid per each block mined. The reason for this is two-fold. 
Dust payments render payout transactions non-standard causing it to fail, also 
making participating accounts forfeit dust payments serves as a good deterrent 
to accounts that contribute intermittent, sporadic work. Participating accounts 
become compelled to commit and contribute enough resources to the pool worth 
more than dust outputs to guarantee receiving dividends whenever the pool 
mines a block. 

## Testing

The project has a configurable tmux mining harness and a CPU miner for testing
on simnet. Further documentation can be found in [harness.sh](./harness.sh).

## Should I be running dcrpool?

dcrpool is ideal for miners running medium-to-large mining operations. The 
revenue generated from mining blocks as well as not paying pool fees to a 
publicly available mining pool in the process should be enough to offset 
the cost of running a pool. It will most likely not be cost effective to run 
dcrpool for a small mining operation, the better option here would be using 
a public mining pool instead.

For people looking to setup a publicly available mining pool, dcrpool's 
well-documented configuration and simple setup process also make it a great option.

## Contact

If you have any further questions you can find us at https://decred.org/community/

## Issue Tracker

The [integrated github issue tracker](https://github.com/decred/dcrpool/issues)
is used for this project.

## License

dcrpool is licensed under the [copyfree](http://copyfree.org) ISC License.
