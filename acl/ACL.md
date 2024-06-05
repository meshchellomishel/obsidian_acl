
1. [***Сети для самых маленьких***](https://linkmeup.gitbook.io/sdsm/5.-acl-i-nat/00-access-control-list)
2. [***Cisco***](https://www.cisco.com/c/en/us/support/docs/security/ios-firewall/23602-confaccesslists.html#acltypes)
3. [***PCAP filter syntax***](https://www.tcpdump.org/manpages/pcap-filter.7.html)
4. [***nftable filter syntax](https://wiki.nftables.org/wiki-nftables/index.php/Quick_reference-nftables_in_10_minutes#Tcp)
5. ***[nft families](https://wiki.nftables.org/wiki-nftables/index.php/Nftables_families)***
6. ***[About fastpath](https://thermalcircle.de/doku.php?id=blog:linux:flowtables_1_a_netfilter_nftables_fastpath)***

***

***Helpful***

1. ***[nft trace monitor](https://unix.stackexchange.com/questions/614413/how-to-properly-log-and-view-nftables-activity)***
```
chain ChainName {
        ...
        meta nftrace set 1
        ...
    }
```
2. **Chains**

		При таблице с несколькими цепочками они обрабатываются в порядке хуков. Лучше не допускать цепочек с одинаковым приоритетом и хуком.


***

***Maybe useful***

1. ***[ct states](https://serverfault.com/questions/1113387/is-m-conntrack-ctstate-new-established-necessary)***
2. 
