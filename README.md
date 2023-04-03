# Advanced ZTP examples for Dell Enterprise SONIC


# Prerequsites
1. ISC-DHCP server
2. HTTP server
3. ZTP JSON file
4. config_db.json file for each switch

## ISC-DHCP server

You can use `docker-compose` file and configuration files from [Advanced ONIE SONIC](https://github.com/zhouska/advanced-onie-sonic).

## HTTP server

You can use the `docker-compose` file bundled with this repository to create a docker image.

## ZTP JSON file

The [file](web-server/ztp/ztp.json) bundled with the repository is the minimal ZTP json file needed for SONIC ZTP:

```
{
  "ztp": {
          "configdb-json": {
                 "dynamic-url": {
                   "source": {
                     "prefix": "http://192.168.50.1/",
                     "identifier": "hostname",
                     "suffix": "-config_db.json"
                   },
                   "destination": "/etc/sonic/config_db.json"
                 },
                 "clear-config": false,
                 "save-config": true
          }
  }
}

```
The file uses the `dynamic-url` [object](https://github.com/sonic-net/SONiC/blob/master/doc/ztp/ztp.md#332-dynamic-url-object) to generate filenames on the fly. In this example, `hostname` is identifier supplied by ONIE. The hostname is different for each switch and its configuration is driven by the ISC-DHCP host config [file](https://github.com/zhouska/advanced-onie-sonic/dhcp-server/SONIC-hosts):

```
group {
  option ztp_json_url "http://192.168.50.1/ztp.json";

  host leaf1 {
    hardware ethernet 0c:16:90:0e:00:00;
    fixed-address 192.168.50.8;
    option host-name "leaf1";
  }
  host leaf2 {
    hardware ethernet 0c:ae:4a:46:00:00;
    fixed-address 192.168.50.9;
    option host-name "leaf2";
  }
  host leaf3 {
    hardware ethernet 0c:16:d5:94:00:00;
    fixed-address 192.168.50.10;
    option host-name "leaf3";
  }
  host leaf4 {
    hardware ethernet 0c:92:71:f2:00:00;
    fixed-address 192.168.50.11;
    option host-name "leaf4";
  }
  host spine1 {
    hardware ethernet 0c:0f:75:27:00:00;
    fixed-address 192.168.50.12;
    option host-name "spine1";
  }
  host spine2 {
    hardware ethernet 0c:9e:e3:80:00:00;
    fixed-address 192.168.50.13;
    option host-name "spine2";
  }
}

```

Thus, each switch will be looking for a unique `config_db.json` file, based on its hostname.


1. Option `"clear-config": false` instructs switch to merge supplied `config_db.json` file with the one already present on the switch
2. Option `"save-config": true` instructs the switch to save resulting cnfiguration, so it can survive a reboot

### config_db.json file

Each switch will receive a unique file. It can contain a partial switch configuration, such as hostnames, management VRF, or a full fledged switch configuration. The examples located in [web-server/ztp](web-server/ztp) are full fledged switch configurations.

```
web-server/ztp/
├── leaf1-config_db.json
├── leaf2-config_db.json
├── leaf3-config_db.json
├── leaf4-config_db.json
├── spine1-config_db.json
├── spine2-config_db.json

```
