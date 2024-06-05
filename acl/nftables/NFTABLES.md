
1. [***chains/filtering***](https://translated.turbopages.org/proxy_u/en-ru.ru.a166f5ea-6659bcf8-7b323903-74722d776562/https/wiki.nftables.org/wiki-nftables/index.php/Configuring_chains)
2. [***hooks***](https://wiki.nftables.org/wiki-nftables/index.php/Netfilter_hooks)
3. ***[about payload check](https://serverfault.com/questions/988309/filter-on-bytes-in-udp-payload-using-nftables)***
4. ***[raw documentation]()***


***


**About ingress hook**

	Функция ingress hook была добавлена в ядре Linux 4.2. В отличие от других функций netfilter, функция ingress hook привязана к определенному сетевому интерфейсу.

	Вы можете использовать nftables с помощью функции ingress hook для применения политик фильтрации на самых ранних этапах, которые вступают в силу еще до предварительной маршрутизации. Обратите внимание, что на этом самом раннем этапе фрагментированные дейтаграммы еще не были собраны заново. Так, например, сопоставление ip-адресов saddr и daddr работает для всех ip-пакетов, но сопоставление заголовков L4, таких как udp dport, работает только для нефрагментированных пакетов или первого фрагмента.

	Перехватчик входных данных предоставляет альтернативу фильтрации входных данных tc. Вам по-прежнему нужен tc для формирования трафика/управления очередями.


***