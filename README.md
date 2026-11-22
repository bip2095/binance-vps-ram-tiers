# Binance VPS: How Much RAM a 24/7 Bot Needs, What AVA Hosting Charges From €2/mo, and When You Should Skip It

For one Freqtrade or ccxt bot trading a handful of pairs, AVA Hosting's **€5.00/month VPS One** (1 vCPU, 2 GB RAM, 25 GB NVMe) covers it. Move to **VPS Power at €20.00/month** (4 vCPU, 8 GB RAM, 90 GB NVMe) when a database, a dashboard and a second strategy share the same box. The €2.00 Start tier is real, and it's the wrong place to save three euros: 1 GB of RAM leaves no headroom for a backtest while the bot is live.

Ava's data centre is in Moldova, and that single fact decides who should read the rest of this page. If your edge depends on reacting to the order book within milliseconds of Binance's matching engine, none of these plans is the right buy, and the honest advice is to rent in Tokyo or Singapore instead. If your bot runs grids, DCA ladders, or signals on 15-minute and longer candles, geography costs you nothing that shows up in the P&L; what matters is the machine staying up and the IP staying identical.

The figures below come from Ava's VPS store and its VPS ordering page, which list the same ladder under two different naming schemes. Where a number isn't published, this article says so rather than filling it in.

> **Fastest decision:** one bot → VPS One at €5.00/month. Several bots plus Postgres and a dashboard → VPS Power at €20.00/month. 👉 [See AVA Hosting's current VPS plans and prices](https://my.ava.hosting/aff.php?aff=15562)

## The full plan list

Seven plans sit on the VPS store page, from a 512 MB annual box to an 8-core machine.

| Plan | vCPU / RAM (ECC) | NVMe | Price | Billing | Order |
| --- | --- | --- | --- | --- | --- |
| VPS Nano | 1 / 512 MB | 20 GB | €11.00 | Annual, 100 Mbps port | [Order](https://my.ava.hosting/aff.php?aff=15562) |
| VPS One | 1 / 2 GB | 25 GB | €5.00 | Monthly | [Order](https://my.ava.hosting/aff.php?aff=15562) |
| VPS Basic | 2 / 4 GB | 50 GB | €10.00 | Monthly | [Order](https://my.ava.hosting/aff.php?aff=15562) |
| VPS Lucky | 2 / 6 GB | 70 GB | €15.00 | Monthly | [Order](https://my.ava.hosting/aff.php?aff=15562) |
| VPS Power | 4 / 8 GB | 90 GB | €20.00 | Monthly | [Order](https://my.ava.hosting/aff.php?aff=15562) |
| VPS Extra | 6 / 12 GB | 120 GB | €30.00 | Monthly | [Order](https://my.ava.hosting/aff.php?aff=15562) |
| VPS Ultra | 8 / 16 GB | 160 GB | €40.00 | Monthly | [Order](https://my.ava.hosting/aff.php?aff=15562) |

Every plan runs on Intel Xeon E5 cores with NVMe storage, unlimited transfer, free DDoS filtering, weekly backups, full root access and a control panel, on a 1 Gbps port except Nano at 100 Mbps. The advertised SLA is 99.9% with 24/7 support. VPS Nano is listed as out of stock at the time of writing, which matters more than it looks: 512 MB wouldn't run a modern bot stack anyway, and it bills annually rather than monthly.

## The other name for the same ladder

Ava's ordering page labels the VPS line differently and shows slightly more disk at the top end.

| Plan | vCPU / RAM | NVMe | Price |
| --- | --- | --- | --- |
| Start | 1 / 1 GB | 15 GB | €2.00/mo |
| Mini | 1 / 2 GB | 25 GB | €5.00/mo |
| Basic | 2 / 4 GB | 50 GB | €10.00/mo |
| Pro | 4 / 8 GB | 100 GB | €20.00/mo |
| Business | 6 / 12 GB | 150 GB | €30.00/mo |
| Power | 8 / 16 GB | 200 GB | €40.00/mo |

Two things are worth noticing. At €20, €30 and €40 the ordering page gives 10 GB, 30 GB and 40 GB more disk than the store page does for the same money, so it's worth opening both before you pay. The €2.00 Start plan also carries a free-trial flag on that page, the only plan that does. 👉 [Open the VPS ordering page and compare both ladders](https://my.ava.hosting/aff.php?aff=15562)

## Why a bot can't just stay on your desktop

A Binance bot is a process that holds websocket subscriptions to the order book and to your user-data stream all night, fires REST orders into rate-limited endpoints, and is holding positions while you sleep. Three things break that arrangement at home: your ISP, your power supply, and Windows Update deciding 3 a.m. is a good time to reboot. Each one costs you a reconnection and a resync, and a resync during a fast move can cost more than a year of hosting.

The security argument is stronger than the uptime one. Binance lets you pin an API key to a list of IP addresses. On a home line with a rotating address that feature is dead weight; on a server with a fixed IP it becomes the control that saves you if a key leaks, because the key is worthless from any other address. Pair it with trade-only or read-only permissions and withdrawals switched off, and a leak stops being an emergency.

Rate limits are counted by weight, per IP and per account. Once several bot processes share one known address, you can budget that weight instead of guessing, and unrelated traffic from the rest of your household stops contaminating the numbers.

Two operational details decide whether the setup survives its first week. The bot has to start itself after a reboot, which means a systemd unit with a restart policy or a Docker container set to always restart. And the server clock has to stay in sync, because Binance rejects requests whose timestamps drift. Install chrony or systemd-timesyncd and forget about it.

## Which tier fits which bot

Python bots are light on CPU and heavy on RAM only when you feed them history. A single Freqtrade instance with a few pairs and SQLite idles well under a gigabyte; the memory goes to dataframes, downloaded candles and whatever database you attach.

**€2.00 Start (1 GB)**: a price-alert script, a webhook relay, a single small ccxt loop with tight logging. Not a comfortable home for Freqtrade with a dozen pairs and a candle cache.

**€5.00 VPS One (2 GB)**: the default for one bot. Freqtrade or a ccxt script, Docker if you like it, a lightweight dashboard, five to ten pairs. This is the plan most readers should buy, and the €3 you save on Start isn't worth the swap file.

**€10.00 VPS Basic (4 GB)**: two to four bots in a Compose stack, or one bot plus Postgres plus Grafana. Comfortable if you also keep months of candles on disk, since you get 50 GB.

**€15.00 VPS Lucky (6 GB)**: the odd tier in the lineup. It buys RAM and disk rather than cores, so it suits a history-hoarding single strategy more than a parallel one.

**€20.00 VPS Power (8 GB, 4 cores)**: several strategies, a real database, and hyperopt or backtests running beside live trading. The Xeon E5 cores are an older generation, so optimisation runs take longer than they would on a current desktop chip; run them overnight rather than expecting laptop speed.

**€30.00 and €40.00**: tick-level research archives, dozens of processes, or a bot stack you're sharing with other people. Most single-trader setups never touch the top of the range.

A rough rule that holds up: each containerised bot costs a few hundred megabytes at idle, and the database plus dashboard costs about as much as one more bot. If you can describe your stack in one sentence, 2 GB is enough. If you need a diagram, buy 8 GB. 👉 [Start with VPS One at €5.00/month](https://my.ava.hosting/aff.php?aff=15562)

## What the price list doesn't tell you

**No Windows image is advertised.** The VPS pages describe root access, control panels and Linux-style administration, with no mention of a Windows licence or RDP. If your bot needs a Windows desktop, ask support before you pay, and budget for a licence if one is possible at all.

**The dedicated IP is worth confirming.** A third-party review published in February 2026 states that Ava's VPS plans include a dedicated IP and full root access. The store page confirms root access on every plan but doesn't spell out IP allocation. Since IP whitelisting is probably the reason you're here, confirm the address is dedicated and static before ordering, then test it from the VPS itself.

**Moldova is the only location.** That's fine for latency-insensitive strategies and bad for anything that reads the book. Before committing, run a ping and a jitter test from the VPS to `api.binance.com` for a few minutes; averages hide spikes, and spikes are what break websocket streams.

**No money-back window is published** on the VPS store page or the ordering page. What you get instead is monthly billing, which caps your downside at one month, plus the free-trial flag on the €2.00 Start tier. Check the trial terms in the cart before you rely on them.

**Prices are in euros** and billed monthly, apart from the annual Nano. If you're paying from a dollar or pound account, your card issuer's conversion applies.

**Independent coverage is thin.** The same 2026 review praises the hardened facility and fast NVMe; a user comment on that page describes decent uptime but slow ticket replies. Neither is a lab test of a trading workload, so treat both as background rather than a verdict.

## Who shouldn't buy this

Anyone running latency arbitrage or market making against the order book should host in Asia, not Moldova. Buy there and accept the price; a €20 server on the wrong continent won't be rescued by more RAM.

Anyone already running a small always-on machine at home with a static IP and a UPS can keep it. The VPS buys you a fixed address and someone else's power bill, and if you already have both, you're paying for convenience.

Readers whose exchange access is restricted at home should read the rules rather than the plan table. Binance's global exchange isn't available in every country, US traders use Binance.US, and renting a server in another region doesn't change account eligibility. Using a VPS to route around a regional restriction is a way to lose an account, not a workaround.

## FAQ

**Will 1 GB run a single Freqtrade instance?** It will start and idle, and it will struggle the first time you run hyperopt or load a year of candles. The €5.00 plan doubles RAM for the price of a coffee, and that's the one to buy.

**How much RAM do five bots need?** Four gigabytes is the realistic floor for five containerised bots sharing a Postgres instance and a dashboard. Eight if you also want to backtest on the same machine while they trade.

**Can I use Windows?** Nothing on the VPS pages advertises a Windows licence or RDP access. Confirm with support first, and plan for Ubuntu or Debian if the answer is no.

**How do I set up the IP whitelist?** Create the key in Binance's API management, restrict it to the VPS's IP, grant trade-only or read-only permission, leave withdrawals disabled, and store the key encrypted rather than in a plain config file.

**How long do I have to get a refund?** No refund window is published. Monthly billing is your protection: run it for a month, and if the latency or the uptime doesn't hold, cancel rather than argue.

**Which plan would you put a grid bot on?** VPS One for one pair set, VPS Basic once you run several pairs and keep a database of fills. Grid bots don't need cores; they need the process to stay alive.

## Choosing

One bot, one exchange, a handful of pairs: VPS One at €5.00/month, and put the saved €3 toward a second month.

Several strategies, a database, a dashboard, and backtests on the same box: VPS Power at €20.00/month. The €10 tier works until you add the third container and start watching memory.

Two things to settle in the cart before you pay: whether you're buying from the store page or the ordering page (the disk differs at the same price), and whether your bot needs Windows, which isn't advertised anywhere on the VPS line. 👉 [Pick the plan on AVA Hosting's VPS store](https://my.ava.hosting/aff.php?aff=15562)
