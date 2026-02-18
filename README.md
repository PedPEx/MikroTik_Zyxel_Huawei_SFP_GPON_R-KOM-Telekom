# MikroTik with Zyxel (PMG3000-D20B) or Huawei (MA5671A) SFP GPON in R-KOM/Telekom network

## English guide
### Important Links
This guide is meant to make the switch from a ISP provided GPON modem (ONT) to your own one, like a Zyxel PMG3000-D20B or a Huawei SmartAX MA5671A as easy as possible. 

A lot of infos come from differnt guides. Here is a list of all information resources and details:
- **Github repo [zyxel-gpon-sfp](https://github.com/xvzf/zyxel-gpon-sfp) by [xvzf](https://github.com/xvzf)**
- [Guide](https://jaseg.de/blog/telekom-gpon-sfp/) by [jaseg](https://github.com/jaseg/) on his blog
- [List of supported modules](https://www.computerbase.de/forum/threads/eigenes-modem-an-ftth-anschluss-via-sfp-gpon-modul.2061989/) by different ISPs in Germany, Italia, France, Portugal, Spain and Canada (Deutsche Telekom, Deutsche Glasfaser, M-Net, EON, Italien Fastweb/TIM/Vodafone, Frankreich Orange, Portugal MEO, Spanien Movistar, Canada Bell)
- Hack GPON website for [Zyxel PMG3000-D20B](https://hack-gpon.org/ont-zyxel-pmg3000-d20b/)
- Hack GPON website for [Huawei MA5671A](https://hack-gpon.org/ont-huawei-ma5671a/)
- Telekom Germany [change Modem-ID website](https://www.telekom.de/hilfe/geraete/router/zubehoer/glasfaser-modem/tausch-melden)
- Telekom Germany [fiber support](https://www.telekom.de/hilfe/internet-telefonie/glasfaser/kontakt)
- Link to Telekom shop for [Zyxel PMG3000-D20B/Telekom Digitalisierungsbox Glasfasermodem](https://geschaeftskunden.telekom.de/business/produkte/internet-festnetz/geraete-zubehoer/zubehoer/digitalisierungsbox-glasfasermodem)

### Initial state and goal

In my hometown, Regensburg (Ratisbona), two ISPs joined forces to get fiber optical cables everywhere in the city. So the R-KOM (formerly known as Glasfaser Ostbayern) and Telekom Deutschland did also install fiber in my building for the GPON. 
Because of this rather unique approach some stuff is a little bit different to the normal GPON modem exchange. 

My plan was to use a Zyxel PMG3000-D20B (Telekom sells these GPON SFP ONTs on their website) with the new [MikroTik hAP-ax-S](https://mikrotik.com/product/hap_ax_s), which has a 2.5 Gbit/s SFP slot and Wifi6 for under €70. 

### Getting access to GPON module and setting up the GPON SFP module - Zyxel PMG3000-D20B

Log in to your MikroTik device (or similar router) and with its default config applied, execute the following configuration changes:
1. Disable or remove the `sfp1` interface from the default bridge `bridge1`\
    ```/interface bridge port remove [find interface=sfp1]```
2. Deactivate auto-negotiation on `sfp1` and set it to `1GbaseX`\
    ```/interface ethernet set sfp1 auto-negotiation=no speed=1G-baseX```
3. Add static IP address to `sfp1` port within the network of the SFP GPON module (factory setting: 10.10.1.1/24)\
    Assign a address of ```10.10.1.2/24``` to the router
    ```/ip address add interface=sfp1 address=10.10.1.2/24```
4. In case the module is already plugged in, you'll now need to unplug and plug it back in or you plug it in now.\
    The initialisation of the module can take up to 2 minutes. You will know that it's ready, when the status of the `sfp1` port is `link ok`\
    ```/interface ethernet monitor sfp1 once```\
    Now you can check wether the module is reachable for the router\
    ```/ping 10.10.1.1 interface=sfp1```
5. To access the GPON module via the `LAN` network, we need to set up a `masqarade nat rule`\
    ```/ip firewall nat add chain=srcnat action=masquerade out-interface=sfp1 comment="GPON access"```
6. You can now access the GPON website and read its configured serial number\
    [10.10.1.1](http://10.10.1.1)\
    Default login is user:`admin` pw:`1234`

    ![System Overview](pics/zyxel/system_overview.png "System Overview")\
    Mine came with the firmware `V1.00(ABVJ.0)b1e`. This firmware has `ssh` deactivated by default! To enable it, you'll either need to flash the firmware `V1.00(ABVJ.0)b3v` or `V2.50(ABVJ.1)b1d` to it (thanks to [maurice-w](https://github.com/maurice-w) - [source](https://github.com/xvzf/zyxel-gpon-sfp/issues/35#issuecomment-2773403653)). You can also find these in the [`files` subfolder](files/zyxel/)\
    ![GPON Connection Information](pics/zyxel/GPON_info.png "GPON Connection Information")\
    Line Staus `O1` indicates, that no fiber link is present at the moment. You want Line Status of `O5`, which indicates a GPON link, but not necessarly a working identification at the ISP side!\
    ![SLID Configuration](pics/zyxel/SLID_config.png "SLID Configuration")\
    
    

### Getting access to GPON module and setting up the GPON SFP module - Huawei MA5671A (not tested yet)

Log in to your MikroTik device (or similar router) and with its default config applied, execute the following configuration changes:
1. Disable or remove the `sfp1` interface from the default bridge `bridge1`\
    ```/interface bridge port remove [find interface=sfp1]```
2. Deactivate auto-negotiation on `sfp1` and set it to `1GbaseX`\
    ```/interface ethernet set sfp1 auto-negotiation=no speed=1G-baseX```
3. Add static IP address to `sfp1` port within the network of the SFP GPON module (factory setting: 192.168.1.10/24)\
    Assign a address of ```192.168.1.2/24``` to the router
    ```/ip address add interface=sfp1 address=192.168.1.2/24```
4. In case the module is already plugged in, you'll now need to unplug and plug it back in or you plug it in now.\
    The initialisation of the module can take up to 2 minutes. You will know that it's ready, when the status of the `sfp1` port is `link ok`\
    ```/interface ethernet monitor sfp1 once```\
    Now you can check wether the module is reachable for the router\
    ```/ping 192.168.1.10 interface=sfp1```
5. To access the GPON module via the `LAN` network, we need to set up a `masqarade nat rule`\
    ```/ip firewall nat add chain=srcnat action=masquerade out-interface=sfp1 comment="GPON access"```
6. You can now access the GPON website and read its configured (G)PON serial number\
    [192.168.1.10](http://192.168.1.10/)\
    I bought my GPON SFP already rooted on ebay. In case you need to do that, take a look at the [hack-gpon.org](https://hack-gpon.org/ont-huawei-ma5671a/) wiki\
    Default login is user:`root` pw:`admin123`

Pictures comming soon™

### What do i need to get the setup running

As mentined before, the `SLID` is no longer used by the Telekom. Only the 12/16 digit `GPON serial number` is used to authenticate your GPON hardware with the Telekom ISP. You can either tell the Telekom/R-KOM your new `GPON SN` (ASCII string!) or clone the existing serial number (in theory at least). The cloning should work within in the Telekom network (when they built it & are actively running and maintaing it), in the R-KOM network (used by the Telekom in Regensburg) the cloning didn't work properly. The Telekom support had to open a support-ticket for the R-KOM containing the (new) `Modem-ID`. After less than 12 hours the fiber link was up and running.

After a loooong phone call with a Telekom fiber technician, a got a lot of valuable information. For example, that the `SLID` is not used anymore either by the Telekom, nor the R-KOM. The only thing left to do, is to reach out to the Telekom (or in my case the R-KOM) and notify them about your new modem. They need the so called `Modem-ID`, which is the same as the `Modem serial number` or `GPON serial number` - typically 12 or 16 digits. In case of the Zyxel ONT you have to give them the `Pon SN` in ASCII from the `SLID Configuration`-tab, the `GPON SN` number printed on the module is a weird mixture of ASCII string in textformat and decimal (ZYWN = 5A 59 57 4E). The `Modem-ID` is **not** the `PLOAM` or `ONT-Installationskennung`, these two also not used anymore by the Telekom. There is also the so called `Home-ID` used by the Telekom, but that's more or less just a hardware ID for your fibre installation. You'll need it once for the initial fibre registration but it has nothing to do with your GPON modem whatsoever.