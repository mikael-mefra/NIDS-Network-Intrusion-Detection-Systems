# Suricata

## When engaging in Suricata, you should always use a customized copy of your production configuration
## Suricata is daemon-firwst. It's suricata.yaml has a default-log-dir setting (default:/var/log/suricata/)

### Run Suricata asking for help on the various command options

```bash
suricata -h
```

## Determine which rules the suricata.yaml configuration file is loading

```bash
suricata -T -c ./etc/suricata.yaml -vvv

i: suricata: This is Suricata version 8.0.3 RELEASE running in SYSTEM mode
W: runmodes: eve module 'ikev2' has been replaced by 'ike'
i: suricata: Configuration provided was successfully loaded. Exiting.
```

## Excersice 3

### Run Suricata against sample.pcap file using the working suricata.yaml file and generate logs

```bash
suricata -c suricata.yaml -r sample.pcap -l ./logs/
```

#### Suricata output options are configured in the suricata.yaml configuration file. Using your favorite text editor, browse through suricata.yaml looking for the output section

```bash
cat ./suricata.yaml | grep "^outputs:" -A 20 | grep -v "#"
```

```bash
cat fast.log
11/24/2022-18:52:48.615635  [**] [1:1000000:2] LamanServer RCE Attempt! [**] [Classification: (null)] [Priority: 3] {TCP} 192.168.2.239:58496 -> 192.168.2.209:50503
11/24/2022-18:52:48.615635  [**] [1:1000001:1] Better LamanServer Exploit Rule [**] [Classification: (null)] [Priority: 3] {TCP} 192.168.2.239:58496 -> 192.168.2.209:50503
```

```bash
jq . ./logs/eve,json
```

#### With jq, you can parse the eve.json file and ask for only the "alerts"

```bash
cat ./logs/eve.json | jq -r -c 'select(.event_type=="alert")|' | jq .

{
  "timestamp": "2022-09-28T17:48:27.850168+0000",
  "flow_id": 1118171606996900,
  "pcap_cnt": 7,
  "event_type": "alert",
  "src_ip": "10.121.70.151",
  "src_port": 21,
  "dest_ip": "10.234.125.254",
  "dest_port": 2217,
  "proto": "TCP",
  "ip_v": 4,
  "pkt_src": "wire/pcap",
  "alert": {
    "action": "allowed",
    "gid": 1,
    "signature_id": 100000034,
    "rev": 1,
    "signature": "Failed FTP Login",
    "category": "",
    "severity": 3
  },
  "direction": "to_server",
  "flow": {
    "pkts_toserver": 1,
    "pkts_toclient": 0,
    "bytes_toserver": 76,
    "bytes_toclient": 0,
    "start": "2022-09-28T17:48:27.850168+0000",
    "src_ip": "10.121.70.151",
    "dest_ip": "10.234.125.254",
    "src_port": 21,
    "dest_port": 2217
  }
}

cat ./logs/eve.json | jq -r -c 'select(.event_type=="alert")|.src_ip'
10.121.70.151
cat ./logs/eve.json | jq -r -c 'select(.event_type=="alert")|.alert|.signature_id'
100000034
```

## Excecise 4

### Run Suricata with the working suricata.yaml file and dump the configuration options. Review the output items to form a valid --set option to add payloads to eve-log alert types.

```bash
# The --dump-config output can be a bit overwhelming. Using more or less will allow you to page through the output or, like below, you can narrow the output using grep
suricata -c suricata.yaml --dump-config | grep "^output"

# The 'outputs.1.eve-log.types.0.alert.tagged-packets = yes' option is very close. We only need to change tagged-packets to payload. The --set option would be:
--set outputs.1.eve-log.types.0.alert.payload=yes

suricata -c suricata.yaml -r ../sample.pcap -l ./logs/ --set outputs.1.eve-log.types.0.alert.payload=yes

cat ./logs/eve.json | jq -r | grep "payload"

 
echo "NTMwIExvZ2luIGluY29ycmVjdC4NCg==" | base64 -d
530 Login incorrect.
```

#### Run Suricata reading the files and replace the payload option with packet. After running Suricata extract the Base64 encoded packet from eve.json and decode it

```bash
suricata -c suricata.yaml -r ../sample.pcap -l ./logs/ --set outputs.1.eve-log.types.0.alert.packet=yes

cat ./logs/eve.json | jq -r | grep "packet"

echo "ANBZqq+AAAGWPD+oCABFAAA+XidAAC4GFJsKeUaXCup9/gAVCKlLe0eaQnO271AYwADTTgAANTMwIExvZ2luIGluY29ycmVjdC4NCg==" | base64 -d

uY29ycmVjdC4NCg==" | base64 -d
�Y����<?E>^'@.�
yF�
�}�K{G�Bs��P��N530 Login incorrect.

echo "ANBZqq+AAAGWPD+oCABFAAA+XidAAC4GFJsKeUaXCup9/gAVCKlLe0eaQnO271AYwADTTgAANTMwIExvZ2luIGluY29ycmVjdC4NCg==" | base64 -d | xxd

00000000: 00d0 59aa af80 0001 963c 3fa8 0800 4500  ..Y......<?...E.
00000010: 003e 5e27 4000 2e06 149b 0a79 4697 0aea  .>^'@......yF...
00000020: 7dfe 0015 08a9 4b7b 479a 4273 b6ef 5018  }.....K{G.Bs..P.
00000030: c000 d34e 0000 3533 3020 4c6f 6769 6e20  ...N..530 Login
00000040: 696e 636f 7272 6563 742e 0d0a            incorrect...
```

## To create a rule header, you must first decide what the rule action will be. This can be any of the following:
- alert Generate alert
- pass Pass traffic, applying no other actions
- Log the related packets but do not generate any alerts
- drop Block the packet and log it
- reject Block the packet, log it, and generate a TCP RST to tear down the connection. 


## All rule headers must specify a protocol, source, source port, direction, destionation, and destionation port. Valid protocols include:
- ip
- tcp
- udp
- icmp

## General Structure of a rule header:
```
alert ip $SOURCE $SPORT -> $DESTINATION $DPORT
```

### Answer
alert ip any any -> any any (sid:1000000;)


