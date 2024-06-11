
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


***

***Netlink payload***

***[attributes header](https://elixir.bootlin.com/linux/latest/source/include/uapi/linux/netfilter/nf_tables.h)***

***
