
Существуют две специальные псевдо-дисциплины – [ingress](https://tldp.org/HOWTO/Adv-Routing-HOWTO/lartc.adv-qdisc.ingress.html) и [clsact](https://qmonnet.github.io/whirl-offload/2020/04/11/tc-bpf-direct-action/#the-clsact-qdisc). «Псевдо» – потому что они не являются дисциплинами в чистом виде, т.е. не реализуют никакой логики по обработке пакетов. Их можно настраивать на сетевом интерфейсе параллельно с основной дисциплиной. Единственное их назначение – возможность вешать фильтры на входящие (ingress) или исходящие (egress) пакеты. Дисциплина clsact включает в себя ingress и позволяет вешать фильтры на оба направления.

1. ***ingress qdisc*** 
	
		Все рассмотренные до сих пор qdisc являются выходными qdisc. Однако каждый интерфейс может также иметь входящий qdisc, который не используется для отправки пакетов сетевому адаптеру. Вместо этого он позволяет вам применять фильтры tc к пакетам, поступающим через интерфейс, независимо от того, имеют ли они локальное назначение или подлежат переадресации.
	
		Поскольку фильтры tc содержат полную реализацию фильтра Token Bucket, а также могут сопоставляться с kernel flow estimator, доступно множество функциональных возможностей. Это позволяет эффективно контролировать входящий трафик еще до того, как он попадет в IP-стек.

2.  ***clsact qdisc***

		Через несколько месяцев после того, как в ядро и iproute2 был внедрен режим прямого действия, в Linux 4.5 появился новый qdisc: clsact. Это похоже на ingress qdisc, к которому мы можем подключать программы eBPF в режиме прямого действия и который не выполняет никакой постановки в очередь. Но clsact действует как надмножество ingress, в том смысле, что он также позволяет подключать программы прямого действия к выходному пути, что ранее было невозможно. Это рекомендуемый qdisc для подключения программ eBPF в режиме прямого действия.

		Более подробную информацию о qdisc class act можно найти в соответствующем журнале фиксации или в руководстве Cilium.

	 [Cilium](https://docs.cilium.io/en/latest/bpf/#tc-traffic-control).

3. ***egress hook***

	The classification-action pipeline allows the user to perform a lot of packet processing in the kernel. However, it required a classful qdisc in order to be able to execute filters and actions. Also, it was hard to configure TC for both QoS and packet processing at the same time. Therefore, Daniel Borkmann introduced the clsact qdisc[3]. Essentially, it is an ingress qdisc to which two filter vlocks can be attached. One of them is used as the ingress chain as with the ingress qdisc, the other one gets executed for egress traffic with a new hook. This way, the TC subsystem is invoked twice for an egress packet – first for the egress chain in the clsact qdisc, second for the root egress qdisc

***

##### tc examples

1. Drop all first icmp packets

ingress
```
tc qdisc add dev enp29s0 clsact

tc f add dev enp29s0 ingress  \
pref 49152                    \
protocol ip                   \
u32                           \
match u16 0x0001 0xffff at 26 \
action drop
```

egress
```
tc f add dev enp29s0 egress   \
pref 49151                    \
protocol ip                   \
u32                           \
match u16 0x0001 0xffff at 26 \
action drop
```

***

##### CLSACT offloading




***


***Some useful***

1. *[tc ingress aggregation](https://netdevconf.info/2.2/slides/salim-tc-workshop05.pdf)***
2. *[about ingress and egress filters](https://lore.kernel.org/netdev/CAFbJv-5KYtxrXwiAJmyFuKx9zVn1NaOmt-EA7eM+_StS-+dbAA@mail.gmail.com/T/)*
3. *[mb useful about ingress/egress](https://lartc.vger.kernel.narkive.com/erTXKqsd/ingress-and-egress)*
4. *[commit about egress and ingress](f6f3bac08ff9855d803081a353a1fafaa8845739)*
5. *[tc egress and ingress examples(bpf)](https://github.com/iproute2/iproute2/commit/8f9afdd531560c1534be44424669add2e19deeec)*
6. 