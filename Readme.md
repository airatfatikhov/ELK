## Ниже — пример docker-compose.yml для локального ELK/Elastic Stack: 3 master-only Elasticsearch node + 3 data node + Logstash + Kibana.

# Перед запуском обязательно настройте параметр ядра для Elasticsearch:

````
echo 'vm.max_map_count=262144' | sudo tee -a /etc/sysctl.conf
````

# Также отключите swap на всех серверах

# НАстройте чтобы все шарды имели копию других шардов
````
PUT _index_template/all_indices_template
{
  "index_patterns": ["*"],
  "priority": 1,
  "template": {
    "settings": {
      "number_of_replicas": 1
    }
  }
}
````

# Для всех индексов сразу (кроме системных)
````
PUT */_settings
{
  "index.number_of_replicas": 1
}
````

# Более безопасный — только ваши индексы
````
PUT logs-*,metrics-*,my-app-*/_settings
{
  "index.number_of_replicas": 1
}
````