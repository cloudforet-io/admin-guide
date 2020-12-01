# Upgrading SpaceONE

이 페이지에서는 운영중인 SpaceOne 의 업그레이드 방안에 대하여 안내 합니다. 업그레이드 대상 버전은 1.3.x 에서 1.5.1 입니다. 

{% hint style="danger" %}
* 이번 업데이트는 통계데이터 리셋을 필요로 합니다. 기존 수집 하였던 Statistics 서비스의 통계 데이터는 삭제 및 향후 지원되지 않을 계획이오니 주의 부탁 드립니다. 
* Inventory DB의 스키마 변경이 있습니다. 안전한 Rollback을 위하여 사전에 Inventory DB 백업을 부탁 드립니다. 
{% endhint %}

## Overall process

   전체 Upgrade 순서는 아래와 같습니다. 변경 항목은 각 버전별 업그레이드 상황에 따라 변경될 수 있습니다. 

* Checking micro service versions : 업그레이드 대상 micro service과 그 버전을 확인 합니다. 
* Checking plugin versions : 업그레이드 대상 plugin과 그 버전을 확인 합니다. 
* Modifying configuration for each micro services : micro service별 변경된 설정을 확인 합니다. 
* Updating database schema : 각 Collection별 DB 스키마 변경 내역을 확인하여 수정 합니다. 
* Modifying statistics query : 변경된 수집 쿼리를 확인하여 추가 합니다. 



## Before you begin

* spaceone 1.5.1 버전으로 정상적인 업그레이드를 위해서는 아래 사항이 사전에 준비되어야 합니다. 
* spaceone 1.3.x 가 사전에 설치되어 있어야 합니다. 
* spacectl 1.5.1이 사전에 설치되어 있어야 합니다. 미설치 환경인 경우 아래의 링크를 참조하여 spacectl을 설치해 주세요. 
* MongoDB cli 환경이 준비되어 있어야 합니다. mongo cli 설치 혹은 MongoDB Compass를 설치해 주세요\([https://docs.mongodb.com/compass/master/install](https://docs.mongodb.com/compass/master/install)\)

{% page-ref page="install-spaceone/installing-spacectl.md" %}

* Release note에 기록된 기능 업데이트 내역을 확인해 주세요. \([https://github.com/spaceone-dev/spaceone/tree/master/release\_notes](https://github.com/spaceone-dev/spaceone/tree/master/release_notes)\)
* Statistics DB와 Inventory DB의 데이터를 백업 해주세요. 



## Checking micro service versions

* SpaceONE의 각 micro service는 container image로 docker hub에 게시됩니다. 
* 이번 1.5.1 버전에서의 stable image 버전을 아래와 같이 정리 합니다. 



### Core Service

* spaceone core가 설치된 namespace는 "spaceone" 으로 가정합니다. 

| namespace | micro service | version |
| :--- | :--- | :--- |
| spaceone | config | 1.5.1 |
| spaceone | console | 1.5.1.3 |
| spaceone | console-api | 1.5.1.5 |
| spaceone | identity | 1.5.1.2 |
| spaceone | inventory | 1.5.1.3 |
| spaceone | monitoring | 1.5.1 |
| spaceone | plugin | 1.5.1 |
| spaceone | repository | 1.5.1 |
| spaceone | secret | 1.5.1 |
| spaceone | statistics | 1.5.1.2 |
| spaceone | report | 1.5.1 |
| spaceone | power-scheduler | 1.5.1 |



### Supervisor

* SpaceONE의 Supervisor가 설치된 namespace는 "supervisor"로 가정합니다. 

| namespace | micro service | version |
| :--- | :--- | :--- |
| supervisor | supervisor | 1.5.1 |



## Checking plugin versions

* SpaceONE 1.5.1에 최적화된 plugin 버전은 아래와 같습니다. 
* SpaceONE의 Official Plugin의 경우 docker hub에 게시됩니다. 

| plugin type | provider | name | version |
| :--- | :--- | :--- | :--- |
| inventory.Collector | aws | aws-ec2 | 1.4 |
| inventory.Collector | aws | aws-cloud-services | 1.2 |
| inventory.Collector | aws | aws-power-state | 1.2 |
| inventory.Collector | google cloud | google-cloud-compute | 1.1 |
| inventory.Collector | google cloud | google-cloud-services | 1.0.2 |
| inventory.Collector | google cloud | google-cloud-power-state | 1.0 |
| inventory.Collector | azure | azure-vm | 1.1 |
| monitoring.DataSource | aws | aws-cloudwatch | 1.0.2 |
| monitoring.DataSource | aws | aws-health | 1.0.2 |
| monitoring.DataSource | google cloud | google-cloud-stackdriver | 1.0.1 |
| power\_scheduler.Controller | google cloud | google-cloud-power-controller | 1.0 |

### plugin type별 버전 변경 방안

* plugin type별 버전 변경 세부 방안은 아래의 Document를 참고해주세요

#### inventory.Collector

{% page-ref page="managing-plugins/inventory.collector.md" %}



#### monitoring.DataSource 

{% page-ref page="managing-plugins/monitoring.datasource.md" %}



## Modifying configuration for each micro services

* micro service 별로 설정 변경이 필요합니다. 
* 설정 변경이 필요한 deployment는 아래와 같습니다. 

### inventory

{% tabs %}
{% tab title="inventory-worker-conf" %}
```text
# ${namespace} 항목에 core service가 설치된 namespace 명을 기록 합니다. 
    CONNECTORS:
        IdentityConnector:
          endpoint:
            v1: grpc://identity.${namespace}.svc.cluster.local:50051
        SecretConnector:
          endpoint:
            v1: grpc://secret.${namespace}.svc.cluster.local:50051
        PluginConnector:
          endpoint:
            v1: grpc://plugin.${namespace}.svc.cluster.local:50051
        ConfigConnector:
          endpoint:
            v1: grpc://config.${namespace}.svc.cluster.local:50051
```
{% endtab %}
{% endtabs %}



### statistics

{% tabs %}
{% tab title="statistics-conf" %}
```text
# ${namespace} 항목에 core service가 설치된 namespace 명을 기록 합니다. 
      CONNECTORS:
        ServiceConnector:
          statistics: grpc://localhost:50051/v1
          identity: grpc://identity.${namespace}.svc.cluster.local:50051/v1
          repository: grpc://repository.${namespace}.svc.cluster.local:50051/v1
          secret: grpc://secret.${namespace}.svc.cluster.local:50051/v1
          inventory: grpc://inventory.${namespace}.svc.cluster.local:50051/v1
          plugin: grpc://plugin.${namespace}.svc.cluster.local:50051/v1
          monitoring: grpc://monitoring.${namespace}.svc.cluster.local:50051/v1
          config: grpc://config.${namespace}.svc.cluster.local:50051/v1
          report: grpc://report.${namespace}.svc.cluster.local:50051/v1
          power_scheduler: grpc://power-scheduler.${namespace}.svc.cluster.local:50051/v1
          cost_saving: grpc://cost-saving.${namespace}.svc.cluster.local:50051/v1
```
{% endtab %}

{% tab title="statistics-scheduler-conf" %}
```text
# ${namespace} 항목에 core service가 설치된 namespace 명을 기록 합니다. 
       ServiceConnector:
          statistics: grpc://statistics.${namespace}.svc.cluster.local:50051/v1
          identity: grpc://identity.${namespace}.svc.cluster.local:50051/v1
          inventory: grpc://inventory.${namespace}.svc.cluster.local:50051/v1
          plugin: grpc://plugin.${namespace}.svc.cluster.local:50051/v1
          repository: grpc://repository.${namespace}.svc.cluster.local:50051/v1
          secret: grpc://secret.${namespace}.svc.cluster.local:50051/v1
          monitoring: grpc://monitoring.${namespace}.svc.cluster.local:50051/v1
          report: grpc://report.${namespace}.svc.cluster.local:50051/v1
          power_scheduler: grpc://power-scheduler.${namespace}.svc.cluster.local:50051/v1
```
{% endtab %}

{% tab title="statistics-worker-conf" %}
```text
# ${namespace} 항목에 core service가 설치된 namespace 명을 기록 합니다. 
        ServiceConnector:
          statistics: grpc://statistics.spaceone.svc.cluster.local:50051/v1
          identity: grpc://identity.spaceone.svc.cluster.local:50051/v1
          inventory: grpc://inventory.spaceone.svc.cluster.local:50051/v1
          plugin: grpc://plugin.spaceone.svc.cluster.local:50051/v1
          repository: grpc://repository.spaceone.svc.cluster.local:50051/v1
          secret: grpc://secret.spaceone.svc.cluster.local:50051/v1
          monitoring: grpc://monitoring.spaceone.svc.cluster.local:50051/v1
          report: grpc://report.spaceone.svc.cluster.local:50051/v1
          power_scheduler: grpc://power-scheduler.spaceone.svc.cluster.local:50051/v1
```
{% endtab %}
{% endtabs %}



### supervisor

* 아래는 설정 삭제가 필요한 항목 입니다. 

{% tabs %}
{% tab title="supervisor-conf" %}
```text
# ${namespace} 항목에 core service가 설치된 namespace 명을 기록 합니다. 
# 아래 항목을 기존 설정에서 삭제해야 합니다. 
... 중략
          backend: KubernetesConnector
... 
```
{% endtab %}
{% endtabs %}



## Modifying database schema

* SpaceONE은 MongoDB를 Main Database로 사용하고 있습니다. 
* 이번 버전에서는 Inventory DB의 Schema 변경사항이 있습니다. 
* 변경을 위해서는 각 Collection 별로 아래와 같이 command를 입력하여 Schema 변경을 수행 합니다. 



### Inventory.region

```text
use inventory;
db.region.deleteMany({state:'DELETED'})
db.region.updateMany({}, {$unset: {"state": 1}})
db.region.updateMany({}, {$unset: {"region_type": 1}})
db.region.updateMany({}, {$unset: {"region_ref": 1}})
db.region.updateMany({}, {$unset: {"deleted_at": 1}})
```



### Inventory.server

```text
use inventory;
db.server.updateMany({}, {$unset: {"region_ref": 1}})
db.server.updateMany({}, {$unset: {"region_type": 1}})
db.server.updateMany({"cloud_service_type": {$exists: false}}, {$set: {cloud_service_group:null, cloud_service_type:null}})
```

### 

### Inventory.cloud\_service

```text
use inventory;
db.cloud_service.updateMany({}, {$unset: {"region_ref": 1}})
db.cloud_service.updateMany({}, {$unset: {"region_type": 1}})
```



### Inventory.provider

```text
use inventory;
db.provider.updateMany({provider:'spaceone'}, {$set: {'tags.color': '#6638B6'}})
db.provider.updateMany({provider:'megazone'}, {$set: {'tags.color': '#004F99'}})
```





## Modifying statistics query

* statistics에서 사용하는 query중 이전 topic을 삭제후 다시 생성해야 합니다. 

### Deleting old statistics query





### Adding new statistics query

* spacectl 를 사용해서 설정을 apply 해야 합니다. 
* daily\_cloud\_service\_summary.yml 
* daily\_cloud\_service\_summary\_by\_project.yml 두 파일을 생성 후
* add\_script.sh shell을 통해 설정을 입력 합니다. 

{% tabs %}
{% tab title="daily\_cloud\_service\_summary.yml" %}
```text
---
topic: daily_cloud_service_summary
options:
  concat:
  - data_source_id: null
    extend_data:
      label: Database
    query:
      aggregate:
        group:
          fields:
          - name: value
            operator: count
      filter:
      - key: ref_cloud_service_type.is_major
        operator: eq
        value: true
      - key: ref_cloud_service_type.labels
        operator: eq
        value: Database
    resource_type: inventory.CloudService
  - data_source_id: null
    extend_data:
      label: Storage
    query:
      aggregate:
        group:
          fields:
          - key: data.size
            name: value
            operator: sum
      filter:
      - key: ref_cloud_service_type.is_major
        operator: eq
        value: true
      - key: ref_cloud_service_type.labels
        operator: eq
        value: Storage
    resource_type: inventory.CloudService
  data_source_id: null
  extend_data:
    label: Compute
  fill_na: {}
  formulas: []
  join: []
  query:
    aggregate:
      group:
        fields:
        - name: value
          operator: count
    filter:
    - key: ref_cloud_service_type.is_major
      operator: eq
      value: true
  resource_type: inventory.Server
schedule:
  hours:
  - 1
```
{% endtab %}

{% tab title="daily\_cloud\_service\_summary\_by\_project.yml" %}
```text
---
topic: daily_cloud_service_summary_by_project
options:
  concat:
  - data_source_id: null
    extend_data:
      label: Database
    query:
      aggregate:
        group:
          fields:
          - name: value
            operator: count
          keys:
          - key: project_id
            name: project_id
      filter:
      - key: ref_cloud_service_type.is_major
        operator: eq
        value: true
      - key: ref_cloud_service_type.labels
        operator: eq
        value: Database
    resource_type: inventory.CloudService
  - data_source_id: null
    extend_data:
      label: Storage
    query:
      aggregate:
        group:
          fields:
          - key: data.size
            name: value
            operator: sum
          keys:
          - key: project_id
            name: project_id
      filter:
      - key: ref_cloud_service_type.is_major
        operator: eq
        value: true
      - key: ref_cloud_service_type.labels
        operator: eq
        value: Storage
    resource_type: inventory.CloudService
  data_source_id: null
  extend_data:
    label: Compute
  fill_na: {}
  formulas: []
  join: []
  query:
    aggregate:
      group:
        fields:
        - name: value
          operator: count
        keys:
        - key: project_id
          name: project_id
    filter:
    - key: ref_cloud_service_type.is_major
      operator: eq
      value: true
  resource_type: inventory.Server
schedule:
  hours:
  - 1
```
{% endtab %}

{% tab title="add\_script.sh" %}
```text
#!/bin/bash
for domain_id in $(spacectl list domain -c domain_id -o quiet); do
    spacectl exec add statistics.Schedule -f daily_cloud_service_summary.yml -p domain_id=$domain_id
    spacectl exec add statistics.Schedule -f daily_cloud_service_summary_by_project.yml -p domain_id=$domain_id
done
```
{% endtab %}
{% endtabs %}







