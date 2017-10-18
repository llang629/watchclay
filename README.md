watchclay
=========
Watches a [Claymore Ethereum mining](https://github.com/nanopool/Claymore-Dual-Miner) rig for issues, and as needed, resets the rig by power cycling an outlet on a [Ubiquiti mPower](https://www.ubnt.com/mfi/mpower/) strip.

![watchclay schematic](https://raw.githubusercontent.com/llang629/watchclay/master/images/watchclay_schematic.png)

Purpose
-------
Sometimes a Claymore Ethereum mining rig can become unstable or unresponsive, particularly when optimizing GPU performance by overclocking and undervolting. If the inactivity goes unnoticed, or if a person can't get immediate physical access to cycle the power, the resulting idleness wastes potential mining capacity.

This watchclay software watches Claymore via its remote management port (default 3333), which provides updates in JSON and HTML format. It watches for several kinds of issue:

- Claymore and/or the rig become unresponsive, and return no data.
- The hash rate falls below some expected performance.
- The mining pool rejects submitted shares.
- The temperature of one or more GPUs exceeds some maximum amount.

For most of these issues, watchclay rechecks a configurable number of times, and if the issues persist, it power cycles the designated outlet on the mPower strip to reset the rig and return Claymore to mining. In case of GPU overheating, watchclay immediately powers down the designated outlet to prevent permanent damage.

Email updates are sent whenever an issue persists and the rig is power cycled, and again when the rig returns to normal. An email update is also sent periodically to indicate normal operation.

A high-level summary of Claymore rig health can be monitored using tail -f on a logfile of watchclay output. The summary includes overall hashrate, slowest GPU, shares accepted and rejected by the mining pool, the temperature of the hottest GPU, and total amps drawn. (Rig power consumption in watts equals current in amps multiplied by line voltage.)

![watchclay tail -f output](https://raw.githubusercontent.com/llang629/watchclay/master/images/watchclay_tailf.png)

Launch and Compatibility
--------
Typical launch command:

`python -u watchclay.py watchclay.conf >watchclay.log 2>&1 &`

The -u option prevents output buffer delay.

If no configuration file is explicitly named, watchclay looks for the default watchclay.conf (see configuration file details below).

Functioning email service is expected at /usr/sbin/sendmail.

watchclay has been tested with Claymore miner versions 10.0, 9.8, and 9.7. The software is written in Python, and has been tested with Python 2.7.10 on MacOS Sierra and with Python 2.7.12 on Ubuntu 16.04. It must run somewhere besides the mining rig, as otherwise the power cycle becomes suicidal. For instance, a small instance on Amazon Web Services with VPN access to the rig and mPower strip works well.

Configuration
--------
Edit the watchclay.conf file to match the configuration to your environment and requriements.

`mpower_ip` The IP address or hostname of your mPower strip.
`mpower_username` and `mpower_password` The username and password for your mPower strip. The manufacturer's defaults are `ubnt` and `ubnt`.
=ubnt
mpower_password=ubnt
mpower_outlet=1
[claymore]
claymore_ip=192.168.1.50
claymore_port=3333
[limits]
hash_floor=100    ; Mh/s, rig total
reject_ceiling=20 ; share rejects by mining pool
temp_ceiling=80   ; degrees Celsius, hottest GPU
[timers]
check_time=10     ; seconds between status checks
max_recheck=12    ; maximum recheck attempts before rig reset
# check_time * max_recheck must be greater than Claymore reboot time to avoid endless power cycles
cycle_time=30     ; seconds for power cycle pause
wait_time=10      ; seconds for mPower and Claymore API responses
update_time=3600  ; seconds between normal email updates
[email]
sender=watchclay@yourdomain.com
recipients=youremail@yourdomain.com
# reference text included in email body, indent for multi-line string
reference =
For Ethereum mining pool activity:
https://ethermine.org/miners/0x<youraccount>
For Ethereum paid balance:
https://etherscan.io/address/0x<youraccount>
Happy mining!


Feedback
--------
Feedback welcome about bugs or feature requests, via the [Issues tab](https://github.com/llang629/watchclay/issues).

If watchclay helped you mine more efficiently, tips are always welcome! :moneybag: :beer: :smile:

Ether `0x61a7d5222cbbC4c86AF8f26954D4BA2a8983DBC9`


----------
Copyright 2017 Larry Lang
