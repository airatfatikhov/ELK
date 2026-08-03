## Ниже — пример docker-compose.yml для локального ELK/Elastic Stack: 3 master-only Elasticsearch node + 3 data node + Logstash + Kibana.

# Перед запуском обязательно настройте параметр ядра для Elasticsearch:

````
echo 'vm.max_map_count=262144' | sudo tee -a /etc/sysctl.conf
````

