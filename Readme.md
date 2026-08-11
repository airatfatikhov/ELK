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

# Для автоматического удаления индексов используется Index Lifecycle Management (ILM). Сам по себе index template не удаляет индексы — он привязывает политику жизненного цикла к новым индексам.

* Создайте ILM-политику с мин.настройками

````
PUT _ilm/policy/jaeger-span-delete-2-days
{
  "policy": {
    "phases": {
      "delete": {
        "min_age": "2d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}
````


* Создайте Index Template с привязкой к ILM

````
PUT _template/jaeger-span-template
{
  "index_patterns": ["jaeger-span-*"],
  "order": 1,
  "settings": {
    "number_of_shards": 5,
    "number_of_replicas": 1,
    "index.lifecycle.name": "jaeger-span-delete-2-days"
  }
}
````

* Проверка шаблона

````
# Посмотреть шаблон
GET _index_template/logs_template

# Симулировать создание индекса
POST _index_template/_simulate_index/logs-test-001
````

# Проверка статуса конкретного индекса
Если вы хотите узнать, на какой стадии сейчас находится индекс и когда он перейдет к удалению:
````
GET <index_name>/_ilm/explain
````

# Проверка самой политики (шаблона)
Чтобы увидеть определение политики и убедиться, что фаза delete настроена корректно:
````
GET _ilm/policy/<policy_name>
````