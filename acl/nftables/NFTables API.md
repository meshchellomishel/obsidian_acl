
***

***Subsystems***

***[netfilter subsystems](https://elixir.bootlin.com/linux/latest/source/include/uapi/linux/netfilter/nfnetlink.h#L51)***

Futher desribes only *NFNL_SUBSYS_NFTABLES* subsystem.

***

***Request***

All netlink requests goes into ***[nf_tables_api](https://elixir.bootlin.com/linux/latest/source/net/netfilter/nf_tables_api.c)***. In this file describes API policy for attributes. For request to this module you need:

For struct ***[nlmsghdr](https://elixir.bootlin.com/linux/latest/source/tools/include/uapi/linux/netlink.h#L44)***:
```
	nlmsg_type=NFNL_SUBSYS_NFTABLES<<8|__SOMEATTR__
```


***

***Netfilter header***

Netfilter have its own header ***[nfgenmsg](https://elixir.bootlin.com/linux/latest/source/include/uapi/linux/netfilter/nfnetlink.h#L34)***(maybe it header useful for other modules)

```
/* General form of address family dependent message.

*/

struct nfgenmsg {

	__u8 nfgen_family; /* AF_xxx */
	
	__u8 version; /* nfnetlink version */
	
	__be16 res_id; /* resource id */

};
```

- ***[nfgen_family](https://elixir.bootlin.com/linux/latest/source/include/uapi/linux/netfilter.h#L58)*** - its not AF_xxx family. Netfilter have own attributes for it.
- version - In all examples that i see its always ***[NFNETLINK_V0](https://elixir.bootlin.com/linux/latest/source/include/uapi/linux/netfilter/nfnetlink.h#L40)*** that equals "0".
- res_id - its useful for *NFNL_CB_BATCH* callbacks for define netfilter subsystem.

***

***Netfilter callback***


```
struct nfnl_callback {

	int (*call)(struct sk_buff *skb, const struct nfnl_info *info,
	
	const struct nlattr * const cda[]);
	
	const struct nla_policy *policy;
	
	enum nfnl_callback_type type;
	
	__u16 attr_count;

};
```

This structure describes that callback for every attribute(nlmsg_type) have itself type. Callback type defined by netlink message structure(for example batch). And if netlink message structure are not this type, there are error "Invalid arguments(EINVAL)".

In ***[nf_tables_api](https://elixir.bootlin.com/linux/latest/source/net/netfilter/nf_tables_api.c)***:
in ***[nf_tables_cb](https://elixir.bootlin.com/linux/latest/C/ident/nf_tables_cb)***:
```
[NFT_MSG_NEWTABLE] = {

	.call = nf_tables_newtable,
	
	.type = NFNL_CB_BATCH,
	
	.attr_count = NFTA_TABLE_MAX,
	
	.policy = nft_table_policy,

},

[NFT_MSG_GETTABLE] = {

	.call = nf_tables_gettable,
	
	.type = NFNL_CB_RCU,
	
	.attr_count = NFTA_TABLE_MAX,
	
	.policy = nft_table_policy,

},

[NFT_MSG_DELTABLE] = {

	.call = nf_tables_deltable,
	
	.type = NFNL_CB_BATCH,
	
	.attr_count = NFTA_TABLE_MAX,
	
	.policy = nft_table_policy,

},
```


- ***batch*** is transaction in nftables context.
	batch msg scheme:
```
	[
		{ 
			BATCH_START
		},
		{
			__SOMECMD__
		},
		...
		{
			__SOMECMD__
		},
		{
			BATCH_END
		}
	]
```

***

***Netlink payload***

***[attributes header](https://elixir.bootlin.com/linux/latest/source/include/uapi/linux/netfilter/nf_tables.h)***

***

***Chain configuration***

Chain have default verdict, by default it is  **[NF_ACCEPT](https://elixir.bootlin.com/linux/latest/source/include/uapi/linux/netfilter.h#L12)**. 

> [!All possible verdicts:]
> 
> - **[NF_DROP](https://elixir.bootlin.com/linux/latest/source/include/uapi/linux/netfilter.h#L11)**
> - **[NF_ACCEPT](https://elixir.bootlin.com/linux/latest/source/include/uapi/linux/netfilter.h#L12)**
> - **[NF_STOLEN](https://elixir.bootlin.com/linux/latest/C/ident/NF_STOLEN)**
> - **[NF_QUEUE](https://elixir.bootlin.com/linux/latest/C/ident/NF_QUEUE)**
> - **[NF_REPEAT](https://elixir.bootlin.com/linux/latest/C/ident/NF_REPEAT)**
> - **[NF_STOP](https://elixir.bootlin.com/linux/latest/C/ident/NF_STOP)**

Its verdict useful not only for chain configuration. The same constants use for *rule configuration*. 


***

***Rule configuration***

By default rule inserts in start of chain. To append it you should use *NLM_F_APPEND* flag in netlink header. 

Verdicts can be added by ***[immediate expression](https://elixir.bootlin.com/linux/latest/source/net/netfilter/nft_immediate.c)*** . Netlink policy:

```
static const struct nla_policy nft_immediate_policy[NFTA_IMMEDIATE_MAX + 1] = {
	[NFTA_IMMEDIATE_DREG] = { .type = NLA_U32 },
	[NFTA_IMMEDIATE_DATA] = { .type = NLA_NESTED },
};
```

Due to netlink policy we can put in attributes only register and data. To add verdict we should put attributes like this:

```
nftnl_expr_set_u32(e, NFTNL_EXPR_IMM_DREG, NFT_REG_VERDICT);

nftnl_expr_set(e, NFTNL_EXPR_IMM_VERDICT, data, data_len);
```

Where is verdicts:


> [!Rule verdicts] Rule verdicts
> - **[NFT_CONTINUE](https://elixir.bootlin.com/linux/latest/C/ident/NFT_CONTINUE)**
> - **[NFT_BREAK](https://elixir.bootlin.com/linux/latest/C/ident/NFT_BREAK)**
> - **[NFT_JUMP](https://elixir.bootlin.com/linux/latest/C/ident/NFT_JUMP)**
> - **[NFT_GOTO](https://elixir.bootlin.com/linux/latest/C/ident/NFT_GOTO)**
> - **[NFT_RETURN](https://elixir.bootlin.com/linux/latest/C/ident/NFT_RETURN)**
> This verdicts can be useful only for rule configuration, but there are some more common configuration verdicts, that describes in *chain configuration*  


---

***Hardware offload***

***[nf_tables_offload.c](https://elixir.bootlin.com/linux/latest/source/net/netfilter/nf_tables_offload.c)***

In *[commit](https://github.com/torvalds/linux/commit/c9626a2cbdb20e26587b3fad99960520a023432b)*:

```
netfilter: nf_tables: add hardware offload support  
  
This patch adds hardware offload support for nftables through the  
existing netdev_ops->ndo_setup_tc() interface, the TC_SETUP_CLSFLOWER  
classifier and the flow rule API. This hardware offload support is  
available for the NFPROTO_NETDEV family and the ingress hook.

Each nftables expression has a new ->offload interface
...
```

`->offload interface` should be called in ***[nft_flow_rule_create](https://elixir.bootlin.com/linux/latest/C/ident/nft_flow_rule_create)*** every time when:
1. Chain flag *[NFT_CHAIN_HW_OFFLOAD](https://elixir.bootlin.com/linux/latest/source/include/uapi/linux/netfilter/nf_tables.h#L218)* set in attribute *[NFTA_CHAIN_FLAGS](https://elixir.bootlin.com/linux/latest/source/include/uapi/linux/netfilter/nf_tables.h#L251)* 
	**AND**
2. Rule was created/deleted

CMP expression support only *NFT_CMP_EQ* op offloading: ***[code here](https://elixir.bootlin.com/linux/latest/source/net/netfilter/nft_cmp.c#L149)***

Offload process in netfilter it is transformation of nft rule to flow rule. Flowtable it is API lower then netfilter, tc, ethtool_rxnfc. Driver configuration may be done with it. 

From some document:
```
An offload device is one that has the NETIF_F_HW_TC feature enabled and implements ndo_setup_tc. 
```

ndo_setup_tc be calls from ***[tcf_block_offload_cmd](https://elixir.bootlin.com/linux/latest/C/ident/tcf_block_offload_cmd)*** 

***

***Some useful***

1. ***[offload express ACL commit](https://lwn.net/Articles/775046/)***
2. ***[about mangle](https://translated.turbopages.org/proxy_u/en-ru.ru.48954f61-6672e6e8-b5946fb7-74722d776562/https/serverfault.com/questions/467756/what-is-the-mangle-table-in-iptables)***
3. ***[Commit add ACL driver support](https://github.com/torvalds/linux/commit/002841be134e60994a34b510eebf5f091d0cd6c6)***
4. ***[about tc offload](https://netdevconf.info/2.2/papers/horman-tcflower-talk.pdf)***
5. 