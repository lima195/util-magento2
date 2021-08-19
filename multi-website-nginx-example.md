## Multi website magento 2
2 frontend websites:

#### *Default* 
url http://mage.com

#### *B2B*
url http://b2b.mage.com


### Db config exemplo:

```sql
SELECT * FROM `core_config_data` WHERE `path` LIKE '%secure/base_%'
```
| config_id | scope    | scope_id | path                         | value                | updated_at          |
| - | - | - | - | - | - |
|         7 | default  |        0 | web/unsecure/base_url        | http://mage.com/     | 2021-08-19 13:21:55 |
|         8 | default  |        0 | web/secure/base_url          | http://mage.com/     | 2021-08-19 13:21:55 |
|        58 | default  |        0 | web/unsecure/base_link_url   | http://mage.com/     | 2021-08-19 13:21:55 |
|        59 | default  |        0 | web/unsecure/base_static_url | NULL                 | 2020-11-18 17:00:02 |
|        60 | default  |        0 | web/unsecure/base_media_url  | NULL                 | 2020-11-18 17:00:02 |
|        61 | default  |        0 | web/secure/base_static_url   | NULL                 | 2020-11-18 17:00:02 |
|        62 | default  |        0 | web/secure/base_media_url    | NULL                 | 2020-11-18 17:00:02 |
|       839 | default  |        0 | web/secure/base_link_url     | http://mage.com/     | 2021-08-19 13:21:55 |
|       840 | websites |       17 | web/unsecure/base_url        | http://b2b.mage.com/ | 2021-08-19 13:30:48 |
|       841 | websites |       17 | web/unsecure/base_link_url   | http://b2b.mage.com/ | 2021-08-19 13:30:49 |
|       842 | websites |       17 | web/secure/base_url          | http://b2b.mage.com/ | 2021-08-19 13:30:49 |
|       843 | websites |       17 | web/secure/base_link_url     | http://b2b.mage.com/ | 2021-08-19 13:30:49 |

```sql
SELECT * FROM `store`
```
| store_id | code      | website_id | group_id | name     | sort_order | is_active |
| - | - | - | - | - | - | - |
|        0 | admin     |          0 |        0 | Admin    |          0 |         1 |
|        1 | default   |          1 |        1 | LojaX     |          0 |         1 |
|       11 | store_b2b |         17 |        3 | LojaX B2B |          0 |         1 |

```sql
SELECT * FROM `store_group`
```
| group_id | website_id | name               | root_category_id | default_store_id | code               |
| - | - | - | - | - | - |
|        0 |          0 | Default            |                0 |                0 | default            |
|        1 |          1 | Main Website Store |                2 |                1 | main_website_store |
|        3 |         17 | LojaX B2B           |              108 |               11 | group_b2b          |

```sql
SELECT * FROM `store_website`
```
| website_id | code        | name         | sort_order | default_group_id | is_default |
| - | - | - | - | - | - |
|          0 | admin       | Admin        |          0 |                0 |          0 |
|          1 | base        | Main Website |          0 |                1 |          1 |
|         17 | website_b2b | LojaX B2B     |          0 |                3 |          0 |


### Nginx config:

Note que talvez precisará alterar vários itens, mas o importante é:
 - include no nginx.conf.sample
 - map $http_host $MAGE_RUN_TYPE
 - map $http_host $MAGE_RUN_CODE
 - server_name dos 2 websites precisam estar de acordo com o $MAGE_RUN_CODE


```
upstream fastcgi_backend {
  server app_php:9000;
}

map $http_host $MAGE_RUN_TYPE {
  "mage.com" website;
  "b2b.mage.com" website;
}

map $http_host $MAGE_RUN_CODE {
  "mage.com" base;
  "b2b.mage.com" website_b2b;
}

## mage.com
server {
  listen 80;
  server_name mage.com;
  
  set $MAGE_MODE developer;
  set $MAGE_DEBUG_SHOW_ARGS 1;
  set $MAGE_ROOT /usr/share/nginx/www;

  include /usr/share/nginx/www/nginx.conf.sample; 
}

## b2b.mage.com
server {
  listen 80;
  server_name b2b.mage.com;

  set $MAGE_MODE developer;
  set $MAGE_DEBUG_SHOW_ARGS 1;
  set $MAGE_ROOT /usr/share/nginx/www;

  include /usr/share/nginx/www/nginx.conf.sample;
}
```

