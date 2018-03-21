# rasplex-vnstat

A dirty hack to run [vnStat](https://humdi.net/vnstat/) on [RasPlex](http://www.rasplex.com).

## Installation

```
mkdir /storage/local
cd /storage/local
wget https://github.com/itspriddle/rasplex-vnstat/releases/download/v1.18/rasplex-vnstat.tar.gz
tar -vzxf rasplex-vnstat.tar.gz
rm rasplex-vnstat.tar.gz
```

Setup `PATH` so you can access `vnstat` and `vnstatd` without full paths:

```sh
echo 'export PATH="/storage/local/bin:/storage/local/sbin:$PATH"' >> /storage/.profile
```

Setup vnStat configuration file so `vnstat` and `vnstatd` work without having
to pass a `--config` option on every invocation:

```sh
ln -s /storage/local/etc/vnstat.conf /storage/.vnstatrc
```

Setup a cron to start `vnstatd`. This will preserve any existing crontab:

```sh
( crontab -l 2> /dev/null | grep -v vnstatd; echo '*/5 * * * * /usr/bin/pgrep vnstatd >/dev/null || /storage/local/sbin/vnstatd -d > /dev/null' ) | crontab -
```

## Configuration

The default `vnstat.conf` is configured to use `eth0` as the default interface
when `vnstat` is called without the `--iface` option. If you are using WiFi
instead, you can run this command to change the configuration:

```sh
sed -i 's|^Interface "eth0"$|Interface "wlan0"|' /storage/local/etc/vnstat.conf
```

## Usage

After first installing, you'll need to wait a few minutes for `vnstatd` to
start and collect usage. You can manually start it with:

```sh
vnstatd -d
```

To see a traffic report, run `vnstat`. See `vnstat --help` for vnStat usage.

## Building

This section outlines how to compile vnStat for use on RasPlex. RasPlex
doesn't include the necessary tools to compile vnStat, so this must be done on
a separate Raspberry Pi running Raspbian. I used a Raspberry Pi 2 to compile
and RasPlex is on a Raspberry Pi 3.

Run these commands from the Raspbian based Pi to download and compile vnStat,
and package it for upload to this repo:

```
sudo -i
mkdir -p /storage/local/{src,bin,sbin,etc,run,var/{lib,log}/vnstat}
cd /storage/local/src
wget https://github.com/vergoh/vnstat/releases/download/v1.18/vnstat-1.18.tar.gz
tar -vzxf vnstat-1.18.tar.gz
cd vnstat-1.18
./configure --prefix=/storage/local --sysconfdir=/storage/local/etc
make
cp vnstat /storage/local/bin
cp vnstatd /storage/local/sbin
sed 's|^DatabaseDir.*$|DatabaseDir "/storage/local/var/lib/vnstat"|;s|^LogFile.*$|LogFile "/storage/local/var/log/vnstat/vnstat.log"|;s|^PidFile.*$|PidFile "/storage/local/run/vnstat/vnstat.pid"|' cfg/vnstat.conf > /storage/local/etc/vnstat.conf
cd /storage/local
tar -vzcf rasplex-vnstat.tar.gz {bin,sbin,etc,run/vnstat,var/{log,lib}/vnstat}
```
