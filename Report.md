
***

##### LXC

После деплоя ***lxc.json*** в DB не появляются записи об интерфейсах.

***

##### bridge.c

Не работают команды:

```
angtelos(config-if-eth21)# switchport
angtelos# ip igmp snooping
angtelos(config-switch)# spanning-tree
```

***linux-switch*** не заходит в функции конфигурации этих команд.

Не работает скорей всего из-за того что эксперементальный cdb-cli работает только на cdb3, нужно переписать lag_cmds.lua также как wifi_cmds и при установке эксперементального confdb удалить строчки с установкой cdb-cli и пользоваться встроенным в LXC изначально

***

##### interfaces.c

Вроде бы работают команды *shutdown*/*no shutdown*

Добавил функцию, привязанную к пути ***XPATH_ITF***:

```
{

	.schema_xpath = XPATH_ITF,
	
	.handler = iface_cdb_handler,

},
```

Если изменить `container aggregation-port` в *angtel-lag.yang* можно привязать к пути ***XPATH_ITF"/angtel-lag:aggregation-port"***.

***

##### lag.c

1. Нужно доработать проверку на конфигурацию, чтобы таймер конфигурации запускался не только в случаях перехода состояний интерфейсов, потому что при ручном отключении портов ***lag.c*** не реагирует на приходящее событие о том, что интерфейс отключен. Без этой проверки все работает.
2. В проверке на административное состояние порта нужно брать *link* не из кеша, а из ядра.
3. Добавил в создание бондинга флаг **IFF_UP** по умолчанию.
4. Нужно добавить проверку на тип в lag.c, перед обработкой портов.
5. lag_cmds.lua в static режиме lag не показывется MAC:

```
angtelos(config-if-enp29s0,eth21,eth41)# show link-aggregation  
	
	LAG 1  
         Mode         : static                    
         Oper-State   : up (for 25 seconds)       
         Load Balance : Source/Destination MAC    
         MAC Addr.    :                           
         Ports        : enp29s0,eth21,eth41

```

***

#### cdb-cli-server

При выполнении команд cdb-cli-server падает из-за ассерта:

1. Создание агрегатора
```
angtelos# int lag 4                                                                                                                      
Creating LAG-interface 'lag4' 
angtelos(config-if-lag4)# Jun 27 18:35:15 cli-client: Unexpected connection termination
```

2. Присоединение к агрегатору lacp
```
angtelos# int eth41
angtelos(config-if-eth41)# link-aggregation group 4
angtelos(config-if-eth41)# int lag4
angtelos(config-if-lag4)# link-aggregation mode lacp
angtelos(config-if-lag4)# Jun 27 18:37:00 cli-client: Unexpected connection termination
```
*Только после включения lacp на агрегаторе происходит ошибка.*

При дальнейшем использовании ничего не вылетает. Может быть как-то связано с **"create"** операцией.

**В логах:**

1. Создание агрегатора
```
tsx 16 (master): received from 'ipc-client'
[
    {
        "operation": "create",
        "target": "/ietf-interfaces:interfaces/interface[name=\"lag4\"]/angtel-lag:aggregator/oper-state",
        "value": {
            "angtel-lag:oper-state": "down"
        }
    },
    {
        "operation": "create",
        "target": "/ietf-interfaces:interfaces/interface[name=\"lag4\"]/angtel-lag:aggregator/down-reason",
        "value": {
            "angtel-lag:down-reason": "not-in-use"
        }
    },
    {
        "operation": "create",
        "target": "/ietf-interfaces:interfaces/interface[name=\"lag4\"]/angtel-lag:aggregator/time-of-last-oper-change",
        "value": {
            "angtel-lag:time-of-last-oper-change": 1095291
        }
    },
    {
        "operation": "create",
        "target": "/ietf-interfaces:interfaces/interface[name=\"lag4\"]/angtel-lag:aggregator/connection-status",
        "value": {
            "angtel-lag:connection-status": "down"
        }
    },
    {
        "operation": "create",
        "target": "/ietf-interfaces:interfaces/interface[name=\"lag4\"]/angtel-lag:aggregator/time-of-last-conn-change",
        "value": {
            "angtel-lag:time-of-last-conn-change": 1095291
        }
    }
]
Jun 27 18:35:14 cdb-lua: LY: Resolving unresolved data nodes and their constraints...
Jun 27 18:35:14 cdb-lua: LY: All data nodes and constraints resolved.
tsx 17 (slave of 16): enqueueing to 'cdb-cli-editor'
[
    {
        "operation": "create",
        "target": "/ietf-interfaces:interfaces/interface[name=\"lag4\"]/angtel-lag:aggregator/oper-state",
        "value": {
            "angtel-lag:oper-state": "down"
        }
    },
    {
        "operation": "create",
        "target": "/ietf-interfaces:interfaces/interface[name=\"lag4\"]/angtel-lag:aggregator/down-reason",
        "value": {
            "angtel-lag:down-reason": "not-in-use"
        }
    },
    {
        "operation": "create",
        "target": "/ietf-interfaces:interfaces/interface[name=\"lag4\"]/angtel-lag:aggregator/time-of-last-oper-change",
        "value": {
            "angtel-lag:time-of-last-oper-change": 1095291
        }
    },
    {
        "operation": "create",
        "target": "/ietf-interfaces:interfaces/interface[name=\"lag4\"]/angtel-lag:aggregator/connection-status",
        "value": {
            "angtel-lag:connection-status": "down"
        }
    },
    {
        "operation": "create",
        "target": "/ietf-interfaces:interfaces/interface[name=\"lag4\"]/angtel-lag:aggregator/time-of-last-conn-change",
        "value": {
            "angtel-lag:time-of-last-conn-change": 1095291
        }
    }
]
Assertion failed: !ret && cparent (/usr/src/debug/confdb3/git-r0/src/confdb.c: cdb_replica_modify: 1792)
```

2. Присоединение к агрегатору lacp
```
tsx 18 (master): received from 'ipc-client'
[
    {
        "operation": "create",
        "target": "/ietf-interfaces:interfaces/interface[name=\"eth41\"]/angtel-lag:aggregation-port/lacp/actor-oper-state",
        "value": {
            "angtel-lag:actor-oper-state": "lacp-timeout defaulted"
        }
    },
    {
        "operation": "create",
        "target": "/ietf-interfaces:interfaces/interface[name=\"eth41\"]/angtel-lag:aggregation-port/lacp/actor-port",
        "value": {
            "angtel-lag:actor-port": 0
        }
    },
    {
        "operation": "create",
        "target": "/ietf-interfaces:interfaces/interface[name=\"eth41\"]/angtel-lag:aggregation-port/lacp/debug-rx-state",
        "value": {
            "angtel-lag:debug-rx-state": "dummy"
        }
    }
]
Jun 27 18:37:00 cdb-lua: LY: Resolving unresolved data nodes and their constraints...
Jun 27 18:37:00 cdb-lua: LY: All data nodes and constraints resolved.
tsx 19 (slave of 18): enqueueing to 'cdb-cli-editor'
[
    {
        "operation": "create",
        "target": "/ietf-interfaces:interfaces/interface[name=\"eth41\"]/angtel-lag:aggregation-port/lacp/actor-oper-state",
        "value": {
            "angtel-lag:actor-oper-state": "lacp-timeout defaulted"
        }
    },
    {
        "operation": "create",
        "target": "/ietf-interfaces:interfaces/interface[name=\"eth41\"]/angtel-lag:aggregation-port/lacp/actor-port",
        "value": {
            "angtel-lag:actor-port": 0
        }
    },
    {
        "operation": "create",
        "target": "/ietf-interfaces:interfaces/interface[name=\"eth41\"]/angtel-lag:aggregation-port/lacp/debug-rx-state",
        "value": {
            "angtel-lag:debug-rx-state": "dummy"
        }
    }
]
Assertion failed: !ret && cparent (/usr/src/debug/confdb3/git-r0/src/confdb.c: cdb_replica_modify: 1792)
```