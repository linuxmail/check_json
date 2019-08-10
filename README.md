check_json
==========

Nagios plugin to check JSON attributes via http(s).

This Plugin is a fork of the existing JSON Plugin from https://github.com/c-kr/check_json with the enhancements of using the Monitoring::Plugin Perl Module, allowing to use thresholds and performance data collection from various json attributes, and supporting X-Auth and Bearer Token authorization methods.

Performance data is also enhanced to extract performance data compliant to Nagios and Graphite standards. One attribute is selected for thresholds check, multiple others can be added for extracting performance data. This plugin is aimed at simplifying Nagios, Icinga & Icinga2 polling of JSON status APIs.

Usage: 
```
check_json -u|--url <URL> -a|--attribute <attribute> [ -c|--critical <threshold> ] [ -w|--warning <threshold> ] [ -p|--perfvars <fields> ] [ -o|--outputvars <fields> ] [ -t|--timeout <timeout> ] [ -d|--divisor <divisor> ] [ -T|--contenttype <content-type> ] [ --ignoressl ] [ -h|--help ]
```

Example: 
```
./check_json.pl --url http://192.168.5.10:9332/local_stats --attribute '{shares}->{dead_shares}' --warning :5 --critical :10 --perfvars '{shares}->{dead_shares},{shares}->{live_shares},{clients}->{clients_connected}'
```

Result:
```
Check JSON status API OK - dead_shares: 2, live_shares: 12, clients_connected: 234 | dead_shares=2;5;10 live_shares=12 clients_connected=234
```

Home Assistant Check API Status
```
./check_json.pl --url http://hass.lan:8123/api/ --attribute '{message}' --expect 'API running.' --bearer LONG_LIVED_ACCESS_TOKEN
```

Home Assistant Check Entity Status
```
./check_json.pl --url http://hass.lan:8123/api/states/sensor.living_room_temperature --attribute '{state}' --warning 75 --critical 85 --bearer LONG_LIVED_ACCESS_TOKEN
```

Requirements
============

Perl JSON package

* Debian / Ubuntu : libjson-perl libmonitoring-plugin-perl libwww-perl
