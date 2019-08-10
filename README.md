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

Icinga2 Integration
===================

Example CheckCommand Definition:
```
/*
 * JSON Check Command
 */
object CheckCommand "check-json" {
  import "plugin-check-command"

  command = [ PluginDir + "/check_json.pl" ]

  arguments = {
    "-u" = {
      required = true
      value = "$json_url$"
    }
    
    "-a" = {
      required = true
      value = "$json_attributes$"
    }

    "-d" = "$json_divisor$"
    "-w" = "$json_warning$"
    "-c" = "$json_critical$"
    "-e" = "$json_expect$"

    "-p" = "$json_perfvars$"
    "-o" = "$json_outvars$"

    "-m" = "$json_metadata$"

    "--ignoressl" = {
      set_if = "$json_ignoressl$"
      description = "Ignore bad SSL certificates"
    }

    "-x" = {
      value = "$json_xauth_token$"
      description = "Add an X-Auth-Token header with the specified token"
    }

    "-b" = {
      value = "$json_bearer_token$"
      description = "Add an Authorization: Bearer header with the specified token"
    }
  }
}
```

Example Service Check Definition:
```
/*
 * Home Assistant API Check
 */
object Service "hass-api" {
  host_name = "hass.lan"
  check_command = "check-json"
  
  vars.json_url = "http://$address$:8123/api/"
  vars.json_attributes = "{message}"
  vars.json_expect = "API running."

  vars.json_bearer_token = "LONG_LIVED_ACCESS_TOKEN"
}
```