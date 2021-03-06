watchclay
=========
Watches a [Claymore Ethereum mining](https://github.com/nanopool/Claymore-Dual-Miner) rig for issues, and as needed, resets the rig by power cycling an outlet on a [Ubiquiti mPower](https://www.ubnt.com/mfi/mpower/) strip. Also summarizes useful Claymore health data.

![watchclay schematic](https://raw.githubusercontent.com/llang629/watchclay/master/images/watchclay_schematic.png)

Purpose
-------
Sometimes a Claymore Ethereum mining rig can become unstable or unresponsive, particularly when optimizing GPU performance by overclocking and undervolting. If the inactivity goes unnoticed, or if a person can't get immediate physical access to cycle the power, the resulting idleness wastes potential mining capacity.

This watchclay software watches Claymore via its remote management port (default 3333), which provides updates in JSON and HTML format. It watches for several kinds of issue:

- Claymore and/or the rig become unresponsive, and return no data.
- The hash rate falls below some expected performance.
- The mining pool rejects submitted shares.
- The temperature of one or more GPUs exceeds some maximum amount.

For most of these, watchclay rechecks a configurable number of times, and if the issue persists, it power cycles the designated outlet on the mPower strip to reset the rig and return Claymore to mining. In case of GPU overheating, watchclay immediately powers down the designated outlet to prevent permanent damage.

Email updates are sent whenever an issue persists and the rig is power cycled, and again when the rig returns to normal. An email update is also sent periodically to indicate normal operation.

A high-level summary of Claymore rig health can be monitored using tail -f on a logfile of watchclay output. The summary includes overall hashrate, slowest GPU, shares accepted and rejected by the mining pool, the temperature of the hottest GPU, and total amps drawn. (Rig power consumption in watts equals current in amps multiplied by line voltage.)

![watchclay tail -f output](https://raw.githubusercontent.com/llang629/watchclay/master/images/watchclay_tailf.png)

Launch and Compatibility
--------
Typical launch command:

`python -u watchclay.py watchclay.conf >watchclay.log 2>&1 &`

The -u option prevents output buffer delay.

If no configuration file is explicitly named, watchclay looks for the default file watchclay.conf (configuration details below).

Functioning email service is expected at /usr/sbin/sendmail.

watchclay has been tested with Claymore miner versions 10.0, 9.8, and 9.7. The software is written in Python, and has been tested with Python 2.7.10 on MacOS Sierra and with Python 2.7.12 on Ubuntu 16.04. It must run somewhere besides the mining rig; otherwise the power cycle becomes suicidal. For example, a small instance on Amazon Web Services with VPN access to the rig and mPower strip works well.

Configuration
--------
Edit the watchclay.conf file to match the configuration to your environment and requriements.

### mPower

`mpower_ip` The IP address or hostname of your mPower strip.

`mpower_username` and `mpower_password` The username and password for your mPower strip. The manufacturer's defaults are **ubnt** and **ubnt**.

`mpower_outlet` The mPower outlet to be turned off and on for a power-cycle reset. If other outlets on the strip are in use, their current draw will be included in the amps reported, but they will not be power cycled.

### Claymore

`claymore_ip` The IP address of your Claymore mining rig.

`claymore_port` The TCP port for Claymore reporting. The default is **3333**.

### Limits

`hash_floor` The minimum acceptable hashrate in Mh/s for the total mining rig. A typical starting point for AMD RX570/580 is **20 Mh/s multiplied by the number of GPUs**. Catches a floundering GPU when Claymore can't remedy.

`reject_ceiling` The maximum acceptable number of shares rejected by the mining pool. Malformed shares can be an undesirable side-effect of overclocking.

`temp_ceiling` The maximum acceptable temperature for any GPU, in degrees Celsius. Unlike the other limits, exceeding the temperature ceiling results in immediate power shut down to avoid permanent damage.

### Timers

`check_time` Seconds between Claymore mining rig status checks. Suggested value is **10 seconds**.

`max_recheck` Maximum rechecks allowed when an issue arises before the rig is reset by power cycle. Too few rechecks results in hair-trigger resets, when waiting would have allowed the issue to resolve itself. Also, check_time multipled by max_recheck must be long enough to allow the rig to reboot and Claymore to restart, or else watchclay will cause an endless loop of resets. Suggested value is **12 rechecks**, which resets the rig after about two minutes of an issue persisting.

`cycle_time` Seconds to pause during a reset between power off and power on. Allows the power supply capacitors to discharge for a clean restart. Suggested value is **30 seconds**.

`wait_time` Seconds to wait before an API call to mPower or Claymore times out with no response. Suggested value is **10 seconds**.

`update_time` Seconds between email updates during normal operation. Suggested value is **3600 seconds** for hourly updates.

### Email

`sender` Email address to send updates from watchclay.

`recipients` One or more email addresses to receive updates from watchclay.

`reference` Text to include in email body for your reference. For example, you could include the URLs for your mining pool dashboard and Ethereum account balance, for easy reference. Indent subsequent lines to include multiple lines.

Feedback
--------
Feedback welcome about bugs or feature requests, via the [Issues](https://github.com/llang629/watchclay/issues) tab.

If watchclay helped you mine more efficiently, tips are always welcome! :moneybag: :beer: :smile:

Ether `0x61a7d5222cbbC4c86AF8f26954D4BA2a8983DBC9`

Happy mining!

----------
Copyright 2017 Larry Lang
