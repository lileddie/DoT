# DoT to secure DNS
Tunnel DNS requests using TLS to thwart DNS snooping

## DNS allows others to view the websites you are visiting
DNS requests are sent in plain text over the Internet

<p align="center">
  <img src="https://github.com/lileddie/fuck-comcast/blob/master/images/dns.jpg" alt="screenshot">
</p>

This allows your ISP, among others, to snoop on your web browsing.  This can allow companies to sell your browsing data to 3rd parties, targeted ads, dead kittens, etc.

## But I use HTTPS, so it's all encrypted, right??
Nope... DNS happens before any browser magic.  DNS request/response traffic exposes the domain (site) you are visiting, but not the specific page.

## Help is on the Way
Browser vendors are working to enable DNS over TLS (DoT) which will encrypt the DNS requests between you and the DNS servers enabled on your browser.  Although this protects you against snooping by your ISP or others between you and the DNS server, it doesn't protect you from browsing by the owners of the DNS server - namely Google or Microsoft.  Firefox has a great [privacy policy.](https://support.mozilla.org/en-US/kb/dns-over-https-doh-faqs#w_what-is-the-privacy-policy-for-dns-over-https)  I could not find one for Safari, Edge, or Chrome - yet.

## DNs over TLS providers
There are several DNS over TLS providers out there, in our example we will use Cloudflare, due primarily to the fact that they have a good privacy policy published [here](https://developers.cloudflare.com/1.1.1.1/commitment-to-privacy/).
## Encrypt all the things
Rather than leaving it up to the browser, you can build a DNS forwarder that encrypts all your DNS traffic.  For our example we will be using an Ubuntu 18.04 virtual machine.

* Install unbound
`sudo apt update`  
`sudo apt install unbound -y`
* disable resolved
`sudo systemctl stop systemd-resolved`  
`sudo systemctl disable systemd-resolved`
* configure unbound
`vi /etc/unbound/unbound.conf`  
```
server:
    interface: 0.0.0.0
        access-control: 192.168.0.0/16 allow
        access-control: 127.0.0.0/8 allow
forward-zone:
        name: "."
        forward-ssl-upstream: yes
## Cloudflare DNS
        forward-addr: 1.1.1.1@853 #cloudflare server 1
        forward-addr: 1.0.0.1@853 #cloudflare server 2

```
* Make sure you have a [static IP on your server.](https://askubuntu.com/a/1057594)
* Modify your router/browsers/computers
  * The DNS server should now be the static IP address of your Ubuntu server.

## Verify!!!
From the Ubuntu server, verify using tcpdump, change ens3 to your local interface:
`tcpdump -nni ens3 -s0 -vvv port 53 or port 853 -w /var/tmp/dnscrypt.pcap`
<p align="center">
  <img src="https://github.com/lileddie/fuck-comcast/blob/master/images/dnscrypt.jpg" alt="screenshot">
</p>
As you can see, the clear text DNS request is proxied from the Ubuntu server using TLS to Cloudflare's DNS servers.

That's it!  Your requests are now encrypted between your local Ubuntu server and Cloudflare's DNS servers.

## Should I use a VPN?
This is a personal choice, if you are using HTTPS and DNS over TLS, all of your data will be encrypted and safe from snooping.  The ISP (or any other intermediary) may still be able to determine which domain/site you are visiting using reverse DNS (PTR records map IP addresses to domain names).  If this is a concern, a VPN might be a better solution.
