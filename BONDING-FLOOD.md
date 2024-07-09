
### Enslaving

##### In the kernel, when port is enslaving:

1. *[join](https://elixir.bootlin.com/linux/latest/source/drivers/net/bonding/bond_main.c#L1955)*

##### events in linux-switch:

enp29s0(first port in the lag):
1. master, linkinfo
2. slave flag setting
3. slave go up
4. mii status go up
enp41(second port in the lag):
1. address
2. slave go up
3. master, linkinfo
4. slave flag setting
5. slave go down
6. slave go up
7. mii status go up

##### at result

minimum 4 netlink events for join port to the lag. In average it is 7 events.

### Releasing

1. linkinfo(bonding default values)
2. no master

1 message is 1KB in average. Socket buffer is 4MB(4096KB).
with 22 ports we have:
22(ports count) * 8(netlink events) * 1KB = 176KB.
22 * 50 = 1100 KB
4096 / 22 ~= 186 packets per port.