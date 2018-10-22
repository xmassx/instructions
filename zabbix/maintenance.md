# exapmle for 3.2 api

## update maintenance period

### python example

``` python
import pyzabbix
import calendar
import time

zapi = pyzabbix.ZabbixAPI("http://zabbix.example.org")
zapi.login("zabbix", "zabbix")

# use current time
since = calendar.timegm(time.localtime())

# all fields required
zabbix_dict={
              "maintenanceid": 3,               #exampled id
              "active_since": since,
              "active_till": since+300,
              "groupids": [14],                 #list of groupids if some
              "name": "Production deployment",
              "hostids": []                     #list of hostids if some
              "timeperiods":
                [
                  {
                    "period": 300,
                    "start_date": since,
                    "timeperiod_type": 0
                  }
                ],
            }

zapi.maintenance.update(zabbix_dict)
```

### principle scheme

Step 1: login to host, 

```
POST http://company.com/zabbix/api_jsonrpc.php HTTP/1.1
Content-Type: application/json-rpc


Sending json data:
{
    "params": {
        "password": "zabbix",
        "user": "Admin"
    },
    "jsonrpc": "2.0",
    "method": "user.login",
    "id": 1
}
Response Code: 200
Response Body: {
    "jsonrpc": "2.0",
    "result": "0424bd59b807674191e7d77572075f33",
    "id": 1
}
```
save field "result" for the future.

Step 2: bump request id and get parts. Example:
```
POST http://company.com/zabbix/api_jsonrpc.php HTTP/1.1
Content-Type: application/json-rpc

Sending json data:
{
    "params": {},
    "jsonrpc": "2.0",
    "method": "maintenance.get",
    "auth": "0424bd59b807674191e7d77572075f33",
    "id": 2
}
Response Code: 200
Response Body: {
    "jsonrpc": "2.0",
    "result": [
        {
            "maintenance_type": "1",
            "name": "Metrics free disk space ignore",
            "active_till": "1536400140",
            "active_since": "1536278400",
            "maintenanceid": "2",
            "description": ""
        },
        {
            "maintenance_type": "0",
            "name": "Production deployment",
            "active_till": "1540211700",
            "active_since": "1540211400",
            "maintenanceid": "3",
            "description": "deploy"
        }
    ],
    "id": 2
}
```

Step 3: use what you find for updating maintenance
```
POST http://company.com/zabbix/api_jsonrpc.php HTTP/1.1
Content-Type: application/json-rpc

Sending json data:
{
    "params": [
        {
            "name": "Production deployment",
            "timeperiods": [
                {
                    "timeperiod_type": 0,
                    "period": 300,
                    "start_date": 1540211423
                }
            ],
            "active_till": 1540211723,
            "active_since": 1540211423,
            "maintenanceid": 3,
            "groupids": [
                14
            ],
            "hostids": []
        }
    ],
    "jsonrpc": "2.0",
    "method": "maintenance.update",
    "auth": "29f2b0ced85c14481a0d5de625ca66f8",
    "id": 37
}
Response Code: 200
Response Body: {
    "jsonrpc": "2.0",
    "result": {
        "maintenanceids": [
            3
        ]
    },
    "id": 37
}
```
