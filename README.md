BLOC-OFFICIAL-POOL
==================

High performance Node.js (with native C addons) mining pool for BLOC coin.

Comes with lightweight example front-end script which uses the pool's AJAX API.


#### Table of Contents
* [Basic features](#basic-features)
* [Extra features](#extra-features)
* [Support / Community](#support--community)
* [Usage](#usage)
  * [Requirements](#requirements)
  * [Easy requirements installation on Ubuntu 16 LTS](#easy-requirements-installation-on-ubuntu-16-lts)
  * [Downloading & Installing](#1-downloading--installing)
  * [Configuration](#2-configuration)
  * [Start the pool](#3-start-the-pool)
  * [Host the front-end](#4-host-the-front-end)
  * [Customize your website](#5-customize-your-website)
  * [Upgrading](#6-upgrading)
* [Monitoring Your Pool](#monitoring-your-pool)
* [Credits](#credits)
* [License](#license)


#### Basic features

* TCP (stratum-like) protocol for server-push based jobs
  * Compared to old HTTP protocol, this has a higher hash rate, lower network/CPU server load, lower orphan
    block percent, and less error prone
* IP banning to prevent low-diff share attacks
* Socket flooding detection
* Payment processing
  * Splintered transactions to deal with max transaction size
  * Minimum payment threshold before balance will be paid out
  * Minimum denomination for truncating payment amount precision to reduce size/complexity of block transactions
* Detailed logging
* Ability to configure multiple ports - each with their own difficulty
* Variable difficulty / share limiter
* Share trust algorithm to reduce share validation hashing CPU load
* Clustering for vertical scaling
* Modular components for horizontal scaling (pool server, database, stats/API, payment processing, front-end)
* Live stats API (using AJAX long polling with CORS)
  * Currency network/block difficulty
  * Current block height
  * Network hashrate
  * Pool hashrate
  * Each miners individual stats (hashrate, shares submitted, pending balance, total paid, etc)
  * Blocks found (pending, confirmed and orphaned)
* An easily extendable, responsive, light-weight front-end using API to display data

#### Extra features

* Admin panel
  * Aggregated pool statistics
  * Coin daemon & wallet RPC services stability monitoring
  * Log files data access
  * Users list with detailed statistics
* Historic charts of pool's hashrate and miners count, coin difficulty, rates and coin profitability
* Historic charts of users's hashrate and payments
* Miner login (wallet address) validation
* Five configurable CSS themes
* Universal blocks and transactions explorer
* Set fixed difficulty on miner client by passing "address" param with ".[difficulty]" postfix
* Prevent "transaction is too big" error with "payments.maxTransactionAmount" option


### Support / Community

* [BLOC official website](https://bloc.money/)
* [BLOC news](https://medium.com/@bloc.money)
* [BLOC explorer](https://bloc-explorer.com/)
* [BLOC browser mining](https://bloc-mining.com/)
* [BLOC developer](https://bloc-developer.com/)
* [BLOC US pool](https://bloc-mining.us/)
* [BLOC EU pool](https://bloc-mining.eu/)
* [BLOC bitcointalk forum](https://bitcointalk.org/index.php?topic=4108831.0)
* [BLOC Twitter page](https://twitter.com/bloc_money)
* [BLOC Telegram group](https://t.me/bloc_money)
* [BLOC Facebook page](https://www.facebook.com/Blocmoney-383098922176113)
* [BLOC Instagram page](https://www.instagram.com/bloc.money/)
* [BLOC Discord server](https://discordapp.com/invite/sMS2ZvB)

Usage
=====

#### Requirements
* Coin daemon(s) - [binaries](https://bloc.money/download) or built from [source](https://github.com/furiousteam/BLOC)
* simplewallet
* [Node.js](http://nodejs.org/) v0.10+ ([nvm will be handy](https://github.com/creationix/nvm))
* [Redis](http://redis.io/) key-value store v2.6+ ([follow these instructions](https://redis.io/topics/quickstart))

[**Redis security warning**](https://redis.io/topics/security): be sure to firewall access redis server - an easy way is to
include `bind 127.0.0.1` in your `redis.conf` file. Also it's a good idea to learn about and understand software that
you are using - a good place to start with redis is [data persistence](https://redis.io/topics/persistence).

##### Easy requirements installation on Ubuntu 16 LTS
Installing pool on different Linux distributives is different, because it depends on system default components and versions. For now, the easiest way to install pool is to use Ubuntu 16 LTS. Thus, all you had to do in order to prepare Ubunty 16 for pool installation is to run (as root):

```bash
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash
source ~/.bashrc
nvm install 0.10.48 && nvm use 0.10.48 && nvm alias default 0.10.48 && nvm use default
apt-get install redis-server nginx git build-essential cmake libboost1.58-all-dev python -y
```


#### 1) Downloading & Installing

Clone the repository and run `npm update` for all the dependencies to be installed:

```bash
git clone https://github.com/furiousteam/BLOC-OFFICIAL-POOL.git pool_destination_folder
cd pool_destination_folder
npm update
```

#### 2) Configuration

Explanation for each field:
```javascript
{
	/* Used for storage in redis so multiple coins can share the same redis instance. */
	"coin": "BLOC",

	/* Used for front-end display */
	"symbol": "BLOC",

	/* Minimum units in a single coin */
	"coinUnits": 10000,

	/* Coin network time to mine one block */
	"coinDifficultyTarget": 120,

	"logging": {

		"files": {

			/* Specifies the level of log output verbosity. This level and anything
			   more severe will be logged. Options are: info, warn, or error. */
			"level": "info",

			/* Directory where to write log files. */
			"directory": "logs",

			/* How often (in seconds) to append/flush data to the log files. */
			"flushInterval": 5
		},

		"console": {
			"level": "info",
			/* Gives console output useful colors. If you direct that output to a log file
			   then disable this feature to avoid nasty characters in the file. */
			"colors": true
		}
	},

	/* Modular Pool Server */
	"poolServer": {
		"enabled": true,

		/* Set to "auto" by default which will spawn one process/fork/worker for each CPU
		   core in your system. Each of these workers will run a separate instance of your
		   pool(s), and the kernel will load balance miners using these forks. Optionally,
		   the 'forks' field can be a number for how many forks will be spawned. */
		"clusterForks": "auto",

		/* Address where block rewards go, and miner payments come from. */
		"poolAddress": "abLoc7dSMmYXkSxcQGGYNDXXhtXDFGuxAiN8AWq5EBmk73jkRFxZ774XDnVJo2aBVoYVGmCAmgi7eUyr231UjE8UYmkTX84jzoy"

		/* Poll RPC daemons for new blocks every this many milliseconds. */
		"blockRefreshInterval": 1000,

		/* How many seconds until we consider a miner disconnected. */
		"minerTimeout": 900,

		"ports": [
			{
				"port": 4444, //Port for mining apps to connect to
				"difficulty": 2000, //Initial difficulty miners are set to
				"desc": "Mobile" //Description of port
			},
			{
				"port": 6666,
				"difficulty": 10000,
				"desc": "Computer / CPU"
			},
			{
				"port": 9999,
				"difficulty": 215000,
				"desc": "Mining Rig / GPU"
			}
		],

		/* Variable difficulty is a feature that will automatically adjust difficulty for
		   individual miners based on their hashrate in order to lower networking and CPU
		   overhead. */
		"varDiff": {
			"minDiff": 100, //Minimum difficulty
			"maxDiff": 200000, //Maximum difficulty
			"targetTime": 50, //Try to get 1 share per this many seconds
			"retargetTime": 20, //Check to see if we should retarget every this many seconds
			"variancePercent": 30, //Allow time to very this % from target without retargeting
			"maxJump": 100 //Limit diff percent increase/decrease in a single retargetting
		},

		/* Set difficulty on miner client side by passing <address> param with .<difficulty> postfix
		   minerd -u D3z2DDWygoZU4NniCNa4oMjjKi45dC2KHUWUyD1RZ1pfgnRgcHdfLVQgh5gmRv4jwEjCX5LoLERAf5PbjLS43Rkd8vFUM1m.5000 */
		"fixedDiff": {
			"enabled": true,
			"separator": ".", // character separator between <address> and <difficulty>
		},

		/* Feature to trust share difficulties from miners which can
		   significantly reduce CPU load. */
		"shareTrust": {
			"enabled": true,
			"min": 10, //Minimum percent probability for share hashing
			"stepDown": 3, //Increase trust probability % this much with each valid share
			"threshold": 10, //Amount of valid shares required before trusting begins
			"penalty": 30 //Upon breaking trust require this many valid share before trusting
		},

		/* If under low-diff share attack we can ban their IP to reduce system/network load. */
		"banning": {
			"enabled": true,
			"time": 600, //How many seconds to ban worker for
			"invalidPercent": 25, //What percent of invalid shares triggers ban
			"checkThreshold": 30 //Perform check when this many shares have been submitted
		},
		/* Slush Mining is a reward calculation technique which disincentivizes pool hopping and rewards users to mine with the pool steadily: Values of each share decrease in time – younger shares are valued higher than older shares.
		More about it here: https://slushpool.com/help/manual/rewards */
		/* There is some bugs with enabled slushMining. Use with '"enabled": false' only. */

		"slushMining": {
			"enabled": false, // 'true' enables slush mining. Recommended for pools catering to professional miners
			"weight": 120, //defines how fast value assigned to a share declines in time
			"lastBlockCheckRate": 1 //How often the pool checks for the timestamp of the last block. Lower numbers increase load for the Redis db, but make the share value more precise.
		}
	},

	/* Module that sends payments to miners according to their submitted shares. */
	"payments": {
		"enabled": true,
		"interval": 600, //how often to run in seconds
		"maxAddresses": 20, //split up payments if sending to more than this many addresses
		"mixin": 0, //number of transactions yours is indistinguishable from
		"transferFee": 1, //fee to pay for each transaction
		"minPayment": 10000, //miner balance required before sending payment
		"maxTransactionAmount": 250000000, //split transactions by this amount(to prevent "too big transaction" error)
		"denomination": 10000 //truncate to this precision and store remainder
	},

	/* Module that monitors the submitted block maturities and manages rounds. Confirmed
	   blocks mark the end of a round where workers' balances are increased in proportion
	   to their shares. */
	"blockUnlocker": {
		"enabled": true,
		"interval": 30, //how often to check block statuses in seconds

		/* Block depth required for a block to unlocked/mature. Found in daemon source as
		   the variable CRYPTONOTE_MINED_MONEY_UNLOCK_WINDOW */
		"depth": 50,
		"poolFee": 1.0, //1.0% pool fee (2% total fee total including donations)
		"devDonation": 0, //0.1% donation to send to pool dev - only works with Monero
		"coreDevDonation": 0 //0.1% donation to send to core devs - works with Bytecoin, Monero, Dashcoin, QuarazCoin, Fantoncoin, AEON and OneEvilCoin
	},

	/* AJAX API used for front-end website. */
	"api": {
		"enabled": true,
		"hashrateWindow": 600, //how many second worth of shares used to estimate hash rate
		"updateInterval": 30, //gather stats and broadcast every this many seconds
		"port": 8111,
		"blocks": 30, //amount of blocks to send at a time
		"payments": 30, //amount of payments to send at a time
		"password": "adminpoolpassword!@!" //password required for admin stats
	},

	/* Coin daemon connection details. */
	"daemon": {
		"host": "127.0.0.1",
		"port": 2086
	},

	/* Wallet daemon connection details. */
	"wallet": {
		"host": "127.0.0.1",
		"port": 2053
	},

	/* Redis connection into. */
	"redis": {
		"host": "127.0.0.1",
		"port": 6379
	},

	/* Monitoring RPC services. Statistics will be displayed in Admin panel */
	"monitoring": {
		"daemon": {
			"checkInterval": 60, //interval of sending rpcMethod request
			"rpcMethod": "getblockcount" //RPC method name
		},
		"wallet": {
			"checkInterval": 60,
			"rpcMethod": "getbalance"
		}
	},

	/* Collect pool statistics to display in frontend charts  */
	"charts": {
		"pool": {
			"hashrate": {
				"enabled": true, //enable data collection and chart displaying in frontend
				"updateInterval": 60, //how often to get current value
				"stepInterval": 1800, //chart step interval calculated as average of all updated values
				"maximumPeriod": 86400 //chart maximum periods (chart points number = maximumPeriod / stepInterval = 48)
			},
			"workers": {
				"enabled": true,
				"updateInterval": 60,
				"stepInterval": 1800, //chart step interval calculated as maximum of all updated values
				"maximumPeriod": 86400
			},
			"difficulty": {
				"enabled": true,
				"updateInterval": 1800,
				"stepInterval": 10800,
				"maximumPeriod": 604800
			},
			"price": { //USD price of one currency coin received from cryptonator.com/api
				"enabled": true,
				"updateInterval": 1800,
				"stepInterval": 10800,
				"maximumPeriod": 604800
			},
			"profit": { //Reward * Rate / Difficulty
				"enabled": true,
				"updateInterval": 1800,
				"stepInterval": 10800,
				"maximumPeriod": 604800
			}
		},
		"user": { //chart data displayed in user stats block
			"hashrate": {
				"enabled": true,
				"updateInterval": 180,
				"stepInterval": 1800,
				"maximumPeriod": 86400
			},
			"payments": { //payment chart uses all user payments data stored in DB
				"enabled": true
			}
		}
	}
}
```

#### 3) Start the pool

Go to pool folder and run:  

```bash
node init.js
```

The file `config.json` is used by default but a file can be specified using the `-config=file` command argument, for example:  

```bash
node init.js -config=config_backup.json
```

This software contains four distinct modules:
* `pool` - Which opens ports for miners to connect and processes shares
* `api` - Used by the website to display network, pool and miners' data
* `unlocker` - Processes block candidates and increases miners' balances when blocks are unlocked
* `payments` - Sends out payments to miners according to their balances stored in redis
* `chartsDataCollector` - Gathers pool's hashrate and miners count, coin difficulty, rates and coin profitability for frontend charts

By default, running the `init.js` script will start up all five modules. You can optionally have the script start
only start a specific module by using the `-module=name` command argument, for example:

```bash
node init.js -module=api
```

[Example screenshot](https://i.imgur.com/SEgrI3b.png) of running the pool in single module mode with tmux.


#### 4) Host the front-end

Simply host the contents of the `website_example` directory on nginx server.


Edit the variables in the `website_example/config.js` file to use your pool's specific configuration.
Variable explanations:

```javascript

/* Must point to the API port in your config.json file. */
var api = "http://poolhost:8111";

/* Pool server host to instruct your miners to point to.  */
var poolHost = "poolhost.com";

/* IRC Server and room used for embedded KiwiIRC chat. */
var irc = "irc.freenode.net/#forknote";

/* Contact email address. */
var email = "support@poolhost.com";

/* Market stat display params from https://www.cryptonator.com/widget */
var cryptonatorWidget = ["DSH-BTC", "DSH-USD", "DSH-EUR"];

/* Download link to cryptonote-easy-miner for Windows users. */
var easyminerDownload = "https://github.com/zone117x/cryptonote-easy-miner/releases/";

/* Used for front-end block links. */
var blockchainExplorer = "http://bloc-explorer.com/{symbol}/block/{id}";

/* Used by front-end transaction links. */
var transactionExplorer = "http://bloc-explorer.com/{symbol}/transaction/{id}";

/* Any custom CSS theme for pool frontend */
var themeCss = "themes/default-theme.css";

/* Other mining pools to display on pool frontend */
var networkStat = {
    "bloc": [
        ["bloc-mining.eu", "https://bloc-mining.eu:8111"],
        ["bloc-mining.us", "https://bloc-mining.us:8111"]
    ]
};
```

#### 5) Customize your website

The following files are included so that you can customize your pool website without having to make significant changes
to `index.html` or other front-end files, thus reducing the difficulty of merging updates with your own changes:
* `custom.css` for creating your own pool style
* `custom.js` for changing the functionality of your pool website


#### 6) Upgrading
When updating to the latest code its important to not only `git pull` the latest from this repo, but to also update
the Node.js modules, and any config files that may have been changed.
* Inside your pool directory (where the init.js script is) do `git pull` to get the latest code.
* Remove the dependencies by deleting the `node_modules` directory with `rm -r node_modules`.
* Run `npm update` to force updating/reinstalling of the dependencies.
* Compare your `config.json` to the latest example ones in this repo or the ones in the setup instructions where each config field is explained. You may need to modify or add any new changes.


### Monitoring Your Pool

* To inspect and make changes to redis I suggest using [redis-commander](https://github.com/joeferner/redis-commander)
* To monitor server load for CPU, Network, IO, etc - I suggest using [New Relic](http://newrelic.com/)
* To keep your pool node script running in background, logging to file, and automatically restarting if it crashes - I suggest using [forever](https://github.com/foreverjs/forever)


Credits
=======

* [forknote-pool](https://github.com/forknote/forknote-pool) - this pool is made from forknote-pool code.

License
-------
Released under the GNU General Public License v2

http://www.gnu.org/licenses/gpl-2.0.html
