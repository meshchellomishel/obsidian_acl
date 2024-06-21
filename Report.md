
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
4. lag_cmds.lua в static режиме lag не показывется MAC:

```
angtelos(config-if-enp29s0,eth21,eth41)# show link-aggregation  
	
	LAG 1  
         Mode         : static                    
         Oper-State   : up (for 25 seconds)       
         Load Balance : Source/Destination MAC    
         MAC Addr.    :                           
         Ports        : enp29s0,eth21,eth41

```