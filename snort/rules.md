# Snort

## When engaging in Snort, you should always use a customized copy of your production configuration

## 1. Run snort, asking for help on the various command options

```bash
snort -?
```

## 2. Determine which rules the snort.lua configuration file is loading with the'-T' option to test the configuration file

```snort
snort -T -c ./snort.lua

--------------------------------------------------
o")~   Snort++ 3.10.2.0
--------------------------------------------------
Loading ./etc/snort.lua:
Loading snort_defaults.lua:
Finished snort_defaults.lua:
        ssh
        host_cache
        pop
        so_proxy
        stream_tcp
        mms
        smtp
        gtp_inspect
        packets
        dce_http_proxy
        ips
        stream_icmp
        hosts
        normalizer
        binder
        wizard
        appid
        js_norm
        file_id
        http2_inspect
        http_inspect
        stream_udp
        ftp_data
        ftp_server
        search_engine
        port_scan
        dce_http_server
        dce_tcp
        dce_smb
        iec104
        cip
        telnet
        ssl
        sip
        rpc_decode
        netflow
        modbus
        host_tracker
        stream_user
        stream_ip
        trace
        back_orifice
        classifications
        dnp3
        active
        process
        ftp_client
        daq
        decode
        alerts
        stream
        references
        arp_spoof
        output
        network
        dns
        dce_udp
        imap
        file_policy
        s7commplus
        stream_file
Finished ./etc/snort.lua:
Loading file_id.rules_file:
Loading file_magic.rules:
Finished file_magic.rules:
Finished file_id.rules_file:
Loading /sec503/Exercises/Day3/local.rules:
Finished /sec503/Exercises/Day3/local.rules:
--------------------------------------------------
ips policies rule stats
              id  loaded  shared enabled    file
               0       1       0       1    ./etc/snort.lua
--------------------------------------------------
rule counts
       total rules loaded: 1
               text rules: 1
            option chains: 1
            chain headers: 1
--------------------------------------------------
port rule counts
             tcp     udp    icmp      ip
     src       1       0       0       0
   total       1       0       0       0
--------------------------------------------------
fast pattern groups
                      src: 2
--------------------------------------------------
search engine (ac_bnfa)
appid: MaxRss diff: 2728
appid: patterns loaded: 300
--------------------------------------------------
pcap DAQ configured to passive.

Snort successfully validated the configuration (with 0 warnings).
o")~   Snort exiting
```

Look for the line that says loading. In this specific snort.lua file the command replies 'Loading ./etc/snort.lua'

```bash
Loading ./etc/snort.lua:
```

## 2.1 Run the -T option with the snort-broken.lua configuration file, where an error is reported, but this time use the quiet option to suppress the startup messages

```bash
snort -T snort-broken.lua -q
```

## 3 Run Snort in IDS mode, starting it with a configuration file and reading from the network or readong from a pcap

```bash
snort -c snort.lua -r ../sampl.pcap -a -A alert_fast
```

### Using the built-in helper functions, Snort will list the available options for -A

```bash
snort --list-plugins | grep logger
```

### The GID is the generator ID, with a value of 1 in the output, that describes a Snort subsystem responsible for generating the output. The 1 indicates that the rules subsystem generated it. A full list of supported GIDs is availables with the following command

```bash
snort --list-gids
```

## 4. Run Snort in IDS mode with logging enabled

#### New options can be passed at the command line using --lua, followed by the requested chain. For example, changing the default false setting to true for alert_fast.file woudl look like

```bash
--lua "alert_fast = { file = true }"
```

Solution

```bash
snort -c ./etc/snort.lua -r ./sample.pcap -q -l ./logs/ -A alert_fast --lua "alert_fast = { file = true }" 
```

### Hint: The default options for the alert_fast module will need to be changed to "output to a file". The available options can be viewed with --help-config

```bash
snort --help-config alert_fast
```

#### Look at the contents of the alert_fast file

```bash
cat ./logs/alert_fast.txt
```

## To create a rule header, you must first decide what the rule action will be. This can be any of the following

- alert Generate alert
- pass Pass traffic, applying no other actions
- Log the related packets but do not generate any alerts
- drop Block the packet and log it
- reject Block the packet, log it, and generate a TCP RST to tear down the connection.

## All rule headers must specify a protocol, source, source port, direction, destionation, and destionation port. Valid protocols include

- ip
- tcp
- udp
- icmp

## General Structure of a rule header

```
alert ip $SOURCE $SPORT -> $DESTINATION $DPORT
```

### Answer

```
alert ip any any -> any any
```
