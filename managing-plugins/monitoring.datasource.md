# monitoring.DataSource

monitoring.DataSource 타입 플러그인 버전을 변경 합니다. 



## plugin 버전 변경

* DB에서 관련 값을 직접 변경 해야 합니다.\(mongo cli 혹은 compass와 같은 tool 필요\) 
* 각 plugin 별로 아래와 같이 입력하여 plugin 버전을 변경 합니다. 

{% tabs %}
{% tab title="aws-cloudwatch" %}
```bash
use monitoring;
db.data_source.updateMany({name:"AWS CloudWatch"}, {$set: {"plugin_info.version": "1.0.2"}});
```
{% endtab %}

{% tab title="aws-health" %}
```bash
use monitoring;
db.data_source.updateMany({name:"AWS Health"}, {$set: {"plugin_info.version": "1.0.2"}});
db.data_source.updateMany({name:"AWS Health"}, {$set: {"plugin_info.options.all_events": true}});
```
{% endtab %}

{% tab title="google-cloud-stackdriver	" %}
```bash
use monitoring;
db.data_source.updateMany({name:"Google Cloud Stackdriver"}, {$set: {"plugin_info.version": "1.0.1"}});
```
{% endtab %}
{% endtabs %}







