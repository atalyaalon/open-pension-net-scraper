# Open Pension Scraper

[![Build Status][travis-image]][travis-url]

This repo contains tools and [meta]data for the purpose of extracting publicly-available
online information to be used by Open Pension.

Open Pension is a "Hasadna" project, aiming to reveal the secrets behind the Israeli pension market.

## Pre Requirements

* Make sure you have Python `3.x` and `virtualenv` installed.
* For batch-dumping, you'll also need a [redis](http://redis.io/) server.

## Installation

* `virtualenv -p python3 venv`
* `./envrun.sh pip install -r requirements.txt`

**Note:** if you `source venv/bin/activate` in a shell,
you can skip the `./envrun.sh` in commands here
(still, it's handy if you open a shell only to run `rq worker`).

If you want `rq-dashboard` (for monitoring batch jobs via browser):

* `./envrun.sh pip install rq-dashboard`

## Running

### Dump portfolio of a single month

[this is something you don't need redis for]

For exmaple: `./envrun.sh python -m web-sources.gemelnet 101 2016 1`
would write Jan 2016 portfolio of kupa 101 to `data/gemelnet-monthly-portfolios/101-2016-01.csv`.

Pensianet allows dumping portfolios of all kranot in a single request: 
`./envrun.sh python -m web-sources.pensianet 2016 1` would write `data/pensianet/2016-01.csv`
containing portfolios of all kranot for Jan 2016.

### Batch dump reports over a period

run these on separate shells:

* [If you don't have a running redis server] `redis-server`

* `./envrun.sh rq worker` (executes the jobs `batch_gemelnet.py` queues)

#### Dumping portfolios

For example: `./envrun.sh python batch_gemelnet.py 101 1999 8 2002 4`
would queue jobs that dump portfolios of kupa 101 for all months between
Aug 1999 and April 2002 into `data/gemelnet-monthly-portfolios/101-1999-08.csv` ... `data/gemelnet-monthly-portfolios/101-2002-04.csv`

#### Dumping performance reports

For example:
`./envrun.sh python batch_gemelnet.py 101 1999 8 2002 4 --type p`
(or `./envrun.sh python batch_gemelnet.py 101 1999 8 2002 4 -t p`)
would queue a single job to fetch a performance report for kupa 101 between Aug
1999 and April 2002 and dump it
into `data/gemelnet/perf-101-1999-08-2002-04.csv`.

There's no `batch_pensianet.py` [yet?], but you can get performance for a 12 month period
by using (for example) `./envrun.sh python -m web_sources.pensianet 2016 10 --type p` (or `-t p`).
This would dump 11/2015-10/2016 performance of all kranot to `data/pensianet/perf-2015-11-2016-10.csv`.
[This doesn't require redis].

#### Monthly incremental dumping

`./dump-latest.sh [N]` queues dumps of portfolios for all kupot `N` months ago,
and performance reports for all kupot for the year between `N+11` and `N` months ago.

It also dumps portfolios and performance of all pensianet kranot for the same periods,
but these appear as single csv files (as opposed to a file per kupa in gemelnet).

Default for `N` is 2 (data for last month isn't available early in the month,
while data for 2 months ago is always available).

#### Monitoring/controlling job execution

* Monitor from console: `./envrun.sh rq info`
* Monitor via browser: `./envrun.sh rq-dashboard`

You can suspend job execution with `./envrun.sh rq suspend`
and resume work with `./envrun.sh rq resume`

### Utilities

[Don't require redis]

#### Generate CSV with totals of all Gemelnet portfolios for a given month

For example:
`./envrun.sh gemelnet_totals.py 2016 9` would generate `data/gemelnet/totals-2016-09.csv` with 
"bottom lines" of all `{kupa id}-2016-09.csv` portfolios.

#### Batch lookup stock names at quotenet

For example: `./envrun.sh python batch_quotenet_stock_names.py < sample-data/quotenet-queries.txt > something.csv`
should generate something similar to `sample-data/quotenet-output.csv`.

## Tests

Only pep8 so far ;)

## Contribute

Just fork and do a pull request (;

[travis-image]: https://api.travis-ci.org/hasadna/open-pension-net-scraper.svg?branch=master
[travis-url]: https://travis-ci.org/hasadna/open-pension-net-scraper
