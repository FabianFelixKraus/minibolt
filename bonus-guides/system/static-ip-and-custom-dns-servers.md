---
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Static IP & Custom DNS servers

#### Set a static IP address and custom DNS nameservers

### Option 1: At the beginning, during the Ubuntu Server installation GUI

When you arrive at **step 5** of the [Ubuntu Server guided installation](operating-system.md#ubuntu-server-installation), you can want to choose set a static IP address and customize the DNS name server/s.

Wait for the router's DHCP server to assign the IP address to your MiniBolt node to find out what IP range the node is in, you will be able to assign the same or a different IP in the next step.

<figure><img src="../../.gitbook/assets/static-ip-dns-gif.gif" alt=""><figcaption></figcaption></figure>

> > **Subnet:** your router subnet, e.g 192.168.1.0/24
>
> > **Address**: your assigned local IP address, eg. 192.168.1.29
>
> > **Gateway:** you router IP, eg. 192.168.1.1
>
> > **Name servers:** DNS servers choosen, eg. same of gateway (192.168.1.1,192.168.1.1) (your ISP DNS) or Cloudflare DNS server (1.1.1.1,1.0.0.1) <- **(recommended)**
>
> > **Search domains:** \<left blank>

{% hint style="info" %}
This is **only** an **example** if your local network IP range is `192.168.1.0-255,` because for this case, the router assigned to the device the IP address **192.168.1.29**, but could be **192.168.0.29** (`192.168.0.0-255)`, then you will need to set this instead:

> **Subnet:** 192.168.0.0/24

> **Gateway:** 192.168.0.1
{% endhint %}

{% hint style="danger" %}
**Be careful by setting this!** If you change the router, could be the new router doesn't work in the same IP address range as the old router, the device won't be registered on the local network and will be completely out of the connection.

💡 To avoid this, you will need to **enable the "Automatic (DHCP)" mode again before changing the router**, following the [Set the DCHP (automatic) configuration](operating-system.md#set-the-automatic-dhcp-mode-configuration-by-command-line) to ensure that the DHCP server auto assigns an IP to the node in the range you are working on, and if you want, after router change, reconfigure the static IP address again following the [Option 2](operating-system.md#option-2-after-ubuntu-server-installation-by-command-line).

If you don't do this, you will have to attach the monitor screen or television and keyboard to fix this.
{% endhint %}

{% hint style="info" %}
In this step, you can set DNS name servers too, for this example, we have configured [Cloudflare DNS name servers](https://www.cloudflare.com/learning/dns/what-is-1.1.1.1/) (1.1.1.1/1.0.0.1) but you could set [others](https://www.quad9.net/service/service-addresses-and-features)
{% endhint %}

### Option 2: After Ubuntu Server installation (by command line)

After having done the [1.3 Remote access](remote-access.md) section, you could want to set a static IP address to your MiniBolt by the command line

* With user **admin**, check your current data network interface by doing

```bash
$ ip address
```

Check your own configuration, the next output is **only** an **example** of a concrete case, but in your case could be different:

<pre><code>1: lo: &#x3C;LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: <a data-footnote-ref href="#user-content-fn-1">eno1</a>: &#x3C;BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 40:a8:f0:ac:6a:37 brd ff:ff:ff:ff:ff:ff
    altname enp0s25
    inet 192.168.1.147/24 metric 100 brd 192.168.1.255 scope global dynamic eno1
       valid_lft 76855sec preferred_lft 76855sec
    inet6 fe80::42a8:f0ff:feac:6a37/64 scope link
       valid_lft forever preferred_lft forever
</code></pre>

Definitions in the case of before:

> > ```
> > <interface> = e.g eno1
> > ```
>
> > ```
> > <ipaddress> = e.g 192.168.1.147
> > ```
>
> > ```
> > <gateway> = 192.168.1.1 (this case) -> case 192.168.0.147, choose 192.168.0.1
> > ```

{% hint style="info" %}
Take note of your case data, you will need it later
{% endhint %}

* Check the current DNS server set, typing the next command

```bash
$ resolvectl status
```

**Example** of expected output:

```
Global
       Protocols: -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
resolv.conf mode: stub

Link 2 (eno1)
    Current Scopes: DNS
         Protocols: +DefaultRoute +LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
Current DNS Server: 192.168.1.1
       DNS Servers: 192.168.1.1
        DNS Domain: home
```

In the case of before:

> > ```
> > <nameserver1> = 192.168.1.1 (your router IP)
> > ```
>
> > ```
> > <nameserver2> = (secondary DNS server, not setted in this case)
> > ```

* Edit the content of the next file

<pre class="language-bash"><code class="lang-bash"><strong>$ sudo nano /etc/netplan/00-installer-config.yaml
</strong></code></pre>

* Replace the content to match this template

```
# This is the network config written by 'subiquity'
network:
  ethernets:
    <interface>:
      addresses:
      - <ipaddress>/24
      nameservers:
        addresses:
        - <nameserver1>
        - <nameserver2>
        search: []
      routes:
      - to: default
        via: <gateway>
  version: 2
```

{% hint style="info" %}
Replace **\<interface>**, **\<ipaddress>**, **\<nameserver1>**, **\<nameserver2>** (optional but recommended)**,** and **\<gateway>** to your own data.\
\
You can choose the DNS server (**\<nameserver1> +** **\<nameserver2>**) whatever you want, including the default one set in the DNS server of your router (normally of the ISP), in that case, you should type the **\<gateway>** address in **\<nameserver1>** and the same in **\<nameserver2>**. For this example, we have configured [Cloudflare DNS name servers](https://www.cloudflare.com/learning/dns/what-is-1.1.1.1/) (1.1.1.1/1.0.0.1) <-- (recommended), but you could set [others](https://www.quad9.net/service/service-addresses-and-features)
{% endhint %}

<details>

<summary>Example ⬇️</summary>

```
# This is the network config written by 'subiquity'
network:
  ethernets:
    eno1:
      addresses:
      - 192.168.1.87/24
      nameservers:
        addresses:
        - 1.1.1.1
        - 1.0.0.1
        search: []
      routes:
      - to: default
        via: 192.168.1.1
  version: 2
```

</details>

* Finally, type this command to apply the changes

```bash
$ sudo netplan apply
```

{% hint style="info" %}
If you chose a different IP address than the router assigned you, this step could break the current SSH connection, reconnect using the chosen IP address
{% endhint %}

#### **Check changes are applied correctly:**

* For the IP address change, type the next command

```bash
$ ip address
```

{% hint style="info" %}
The output of this command **may not change**, depending on whether you chose the same IP that the router originally assigned you at first, or whether you chose to change to another
{% endhint %}

* For the DNS servers change, type the next command

```bash
$ resolvectl status
```

<details>

<summary>Example ⬇️</summary>

```
Global
       Protocols: -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
resolv.conf mode: stub

Link 2 (eno1)
    Current Scopes: DNS
         Protocols: +DefaultRoute +LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
Current DNS Server: 1.1.1.1
       DNS Servers: 1.1.1.1 1.0.0.1
[...]
```

</details>

### Set the A**utomatic (DHCP) mode** configuration (by command line)

If you go to change the router you could want to enable or check if you have enabled the a**utomatic (DHCP) mode** configuration to avoid problems

* Edit the content of the next file

<pre class="language-bash"><code class="lang-bash"><strong>$ sudo nano /etc/netplan/00-installer-config.yaml
</strong></code></pre>

* Replace the content to match this template

<pre><code><strong># This is the network config written by 'subiquity'
</strong>network:
  ethernets:
    &#x3C;interface>:
      dhcp4: true
  version: 2
</code></pre>

* Finally, type this command to apply the changes

```bash
$ sudo netplan apply
```

{% hint style="info" %}
Check changes are applied correctly following the [same step](operating-system.md#check-changes-are-applied-correctly) as before
{% endhint %}

[^1]: \<interface>
