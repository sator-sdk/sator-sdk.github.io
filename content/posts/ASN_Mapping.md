+++
title = "Map Enterprise ASN Infrastructure"
date = "2023-10-06"
+++


# Map Enterprise [ASN](https://www.cloudflare.com/learning/network-layer/what-is-an-autonomous-system/) Infrastructure

This post is part of a series **GO Hunt**, that involve how to perform Information Gathering or Threat Hunting with modern tools mostly written in `go` from [ProjcetDiscovery](https://github.com/projectdiscovery) and many others like [Shodan](https://www.shodan.io/), [Censys](https://search.censys.io/), [Fofa](https://en.fofa.info/) and so on, and relevant techniques used during Penetration Tests and RedTeam simulations.

Also the idea of having an ordered map it came to me from an old tool that I was using [ksubdomain](https://github.com/knownsec/ksubdomain) from **knownsec** (scary fast) that had an interesting feature to summarize the output of the scan and map all the founded domains in the ASN structures, but unfortunately the tool is not manteined anymore, there is the new version from [BoyHack](https://github.com/boy-hack/ksubdomain) that is still impressive but doesn't have some features, for example the `-summary` tag.

So I decide to implement a short Python script that let me see clearly the data as I was used to.
Organize them in a Database is obviously a MUST and something that have to be done during a PT but here im just talking about quickly visualize data to understand them better.

> Since most TTPs will be treated in depth in future articles this one is just a stand-alone post regarding ASN mapping

## ASN Infrastructure

In the Recon phase one of the first step to see the bigger picture, is to understand how the whole Network Infrastructure of a Company is dislocated, servers orign points, subdomains, and all the external services used. 

> More on how to set up a detailed objective priority planning, later-on in the series

So the ASN is one of the `root` points of this mapping since will be extreamly significant in some situations to take actions in consideration of different ASN characteristics and providers.

Even more useful is to know what Network to test on in presence of a significant number of IPs.
Since the dislocation of an Enterprise could be (and should be) accross multiple ASN providers, the Network ranges will be in very high CIDR order tipically from `/14` or '/16' or even lower in some situations,and difficult to test without making noise on the network.

## Mapping the ASN

First step to map a Company traces inside an ASN is to map its subdomains and their conresponding location.

Let's begin with the first three tools from **ProjectDiscovery** that are a very good subdomain enum, a DNS inspector and an ASN crawler:

* [Subfinder](https://github.com/projectdiscovery/subfinder)
* [dnsx](https://github.com/projectdiscovery/dnsx)
* [asnmap](https://github.com/projectdiscovery/asnmap)

Launching the tool without an enumeration list will make the tool go in automatic mode that is fine for a not refined approach, but when you need to inspect some more specific situation i suggest to use a wordlist for a more detailed and surgical enumeration. One amazing thing that all the tools from ProjectDiscovery can do is to be piped toghether to achieve a huge number of combination an different results.

Enumeration:

```sh
# Subdomains mapped across each ASN Zone and provider
subfinder -silent -d hackerone.com | dnsx -silent -a -resp-only | asnmap -json -silent | jq -n '.TARGET |= [inputs]' > target.json

# Then create a hostfile
subfinder -silent -d hackerone.com | dnsx -silent -a -resp -json | jq 'with_entries(select(.key | in({"host":1, "a":1})))' | jq -n '[inputs]' > hostsfile.json

# Parse them with the script
python ASN-recon.py target.json hostsfile.json
```

### [ASN-Recon Script](https://github.com/sator-sdk/ASN-Recon) output

<p align="center">
<img src="/posts/asnrecon_output.png">
</p>
<p align="center">
Output example of the script
</p>

This is jsut a demonstration of the capilities of those tools but as you can see with a few chained commands we were able to gather a pretty decent ammount of info and organize them toghether.

In the Next article I will show how to inspect separatly each Network with [HEDnsExtractor](https://github.com/HuntDownProject/HEDnsExtractor) from HuntDownProject to further enumerate hidden instances in the same network.