# charge-lnd

This script matches your open channels against a number of customizable criteria and applies a channel policy based on the matching template.

## Installation

This script needs a moderately recent lnd (https://github.com/lightningnetwork/lnd) instance running.

You need to have admin rights to control this node.
By default this script connects to `localhost:10009`, using the macaroon file in `~/.lnd/data/chain/bitcoin/mainnet/admin.macaroon`.
If you need to change this, please have a look at the optional arguments `--grpc` and `--lnddir`.

You need to install Python. The gRPC dependencies can be installed by running:

```
$ pip install -r requirements.txt
```

On some systems using Python 3, use pip3 instead:

```
$ pip3 install -r requirements.txt
```

## Usage

charge-lnd takes only a minimal set of parameters:

```
usage: charge-lnd.py [-h] [--lnddir LNDDIR] [--grpc GRPC] [-c CONFIG]

optional arguments:
  -h, --help            show this help message and exit
  --lnddir LNDDIR       (default ~/.lnd) lnd directory
  --grpc GRPC           (default localhost:10009) lnd gRPC endpoint
  -c CONFIG, --config CONFIG
                        (default: charge.config) path to config file
```

All policies are defined using an INI style config file (defaults to `charge.config` in the current directory)

Each `[section]` defined in the config file describes a 'template'.
A single template consists of basically two components;
- a match definition
- a strategy

The match definition allows templates to match against attributes of channels and associated nodes.
The strategy then defines how to set the channel policy.

For example:
```
[friends]
match = id
channels = 634674x1877x1, 697111262856151041

strategy = static
base_fee_msat = 0
fee_ppm = 1
```

This template matches the channels using the `id` matcher. This matcher checks if the channel matches one of the channel IDs in the `channels` property.  This is a comma-separated list of short or full channel IDs.

If a channel matches the `[friends]` template, the `static` strategy is used, which just takes the `base_fee_msat` and `fee_ppm`  properties defined in the template and applies them to the channel.

There is a special `[default]` section, that will be used if none of the templates matches a channel. The `[default]` section only contains a strategy, not a match definition.

All templates are evaluated top to bottom. The first matching template is applied (except for the default template). More than one matcher can be defined for a template. In those cases, the results of the matchers must both be true to match the template (logical AND).

A more elaborate example can be found in the [charge.config](charge.config) file.

### matchers

Currently available matchers:
- **id** (match on channel IDs and node pubkeys. properties: **channels**, **nodes**)
- **initiator** (match on initiator status. properties: **initiator**)
- **balance** (match on channel balance. properties: **max_ratio**, **min_ratio**)
- **peersize** (match on peer node size. properties: **min_channels**, **max_channels**, **min_sats**, **max_sats**)
- **peerpolicy** (match on policy used by peer. properties: **min_base_fee_msat**, **max_base_fee_msat**, **min_fee_ppm**, **max_fee_ppm**)
- **private** (match on channel private flag. properties: **private**)

### strategies
- **ignore** (ignores the channel)
- **static** (sets fixed base fee and fee rate values. properties: **base_fee_msat**, **fee_ppm**)
- **match_peer** (sets the same base fee and fee rate values as the peer)

## Contributing

Contributions are highly welcome!
Feel free to submit issues and pull requests on https://github.com/accumulator/charge-lnd/

Please also consider opening a channel with my node, or sending tips via keysend:

* accumulator : `0266ad254117f16f16c3457e081e6207e91c5e414477a208cf4d9c633322799038@89.99.0.115:9735`
