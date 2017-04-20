DNS Interface 定义
=======================

    http header:
        * context-type: application/json
        * accept: application/json
    http code:
        * 200
        * 404
        * 204
        * 400
    tips:
        全部接口添加一个operator字段, 除了Task, Apply

        <!-- 'task': -->
            <!-- task_type: secondary -->
            <!-- action: add_zone -->
        <!-- 表结构设计的时候, task, subtask,  -->
        <!-- 对于子任务： -->
            <!-- task_id: abc, -->
            <!-- subtask: NULL, -->
        <!-- 对于父认为: -->
            <!-- task_id: 123, -->
            <!-- subtask: abc -->
        <!-- 行为类接口返回的都是子任务，通过把子任务进行编排，需要调用apply接口，
             返回一个新的任务 -->
        <!-- 这个任务认为是父任务, (及时只有一个子任务，也需要调用apply接口)
            所有的任务都可以通过task接口去查询 -->

#### Author: wei.zheng
#### E-mail: monkeyerKong@gmail.com

# 目录
* [DnsRegion](#DnsRegion)
* [RegionSummary](#RegionSummary)
* [NameServerHost](#NameServerHost)
* [NameServer](#NameServer)
* [NameServerACL](#NameServerACL)
* [View](#View)
* [Zone](#Zone)
* [Record](#Record)
* [ACL](#ACL)
* [AclRule](#AclRule)
* [Apply](#Apply)
* [Task](#Task)

## DnsRegion (基础信息类)
    note:
            该接口只是从逻辑层面进行操作，不进行底层的操作。上线一个站点，
        从该接口把这个站点录入到smart中，下线一个站点，逻辑上删除，站点进行
        维护，设置站点为maintenance模式
    url: http://endpoint:8774/dnsregion 接口
    key field:
        region_id:
        site_code: 站点code
        site_name: 站点名称
        software: dns 软件
        version: dns 软件版本
        supportor: 运维负责人 
        is_maintenance: 是否进入维护模式 [True, Fasle]
        create_at
        update_at
        delete_at
        
    method:
        list: 列出所有dns站点
        add: 添加一个站点DNS服务
        delete: 删除一个站点DNS服务
        update: 更新DNS站点的信息

    add:
        url: POST http://endpoint:8774/dnsregion        
        body:
            {
                'site_code': xxx,
                'site_name': xxx,
                'software': xxx,
                'version': xxx,
                'supportor': xxx,
                'is_maintenance': xxx,
            }
        response data:
            {
                'region_id': xxx,
            }
        http code:
            200

    delete:
        note: 删除知识逻辑的删除，需要通过实施以后，才能调用，
              删除也需要把下面的资源清除，在调用该接口
        url: DELETE http://endpoint:8774/dnsregion/{id}
        response data:
            None
        http code:
            204
    update:
        url: PUT http://endpoint:8774/dnsregion/{id}
        body:
            {
                'is_maintenance': xxx,
                'supportor': xxx,
            }
        response data:
            None
        http code:
            204
    list:
        url: GET http://endpoint:8774/dnsregion
        response data:
            {
                regions: [
                    {
                        'site_code': xxx,
                        'site_name': xxx,
                        'software': xxx,
                        'version': xxx,
                        'supportor': xxx,
                        'is_maintenance': xxx,
                        'create_at': xxx,
                        'delete_at': xxx,
                        'update_at': xxx,
                        'nameservers': [
                            {'label': xxx, 'id': 'nameserver_id01'},
                            {...}
                        ]
                    },
                    {...}
                ]
            }


### RegionSummary(基础信息类)
    url: http://endpoint:8774/regionsummary/{site_code}
    key field:
        nameserver_id: 
        state: maintenance(维护), online(在线运行), offline(下线)
        host_numbers: 部署dns server用几台服务器
        inuse_acl: 该区域用了多少个acl
        view_numbers: 该区域有多少view
        zone_numbers: 该区域有多少zone

    method:
        get

    get:
        note: 对nameservers返回是一个列表，用于以后扩展，现在的情况是只有1个
        url: GET http://endpoint:8774/dnsregion/0001234
        response data:
            {
                'nameservers': [
                    {
                        'nameserver_id': xxx,
                        'state': 'online'
                        'host_numbers': 3,
                        'view_numbers': 3,
                        'zone_numbers': 15,
                        'inuse_acl': 2,    
                    },
                    {...},
                ]
            }
        http code:
            200
            

## NameServerHost(基础信息类)
    url: http://endpoint:8774/nameserverhost
    key field:
        position: 该服务器位于哪个数据中心，比如：兆维，马驹桥，东京
        role: 当前服务器在dns 服务中充当什么角色，master,或者其他
        os_type: 操作系统类型
        os_version: 操作系统版本号
        os_bit: 操作系统位数
        ip_address: 服务器的ip地址
        mask: 掩码地址
        vlan: 使用的哪个vlan
        nameserver_id: 属于哪个nameserver
        create_at:
        update_at:
        delete_at:

    method:
        list: GET
        add: POST
        delete: DELETE
        update: PUT

    list:
        url: GET http://endpoint:8774/nameserverhost
        response data:
            {
                hosts:[
                    {
                        'id': '42dedd03-7feb-4835-ad14-0d64f8497204'
                        'position': '兆维'
                        'role': 'master'
                        'os_type': 'centos'
                        'os_version': '7.0'
                        'os_bit': 64
                        'ip_address': 192.168.101.123
                        'mask': 255.255.255.0
                        'vlan': 1989
                        'nameserver_id': xxx,
                        'create_at':xxx
                        'update_at': xxxx
                        'delete_at': xxx
                    },
                    {
                        ...
                    }
                ]
            }
    add:
        url: POST http://endpoint:8774/nameserverhost
        body:
            {
                'position': '兆维',
                'role': 'master',
                'os_type': 'centos',
                'os_version': '7.0',
                'os_bit': 64,
                'ip_address': xxx,
                'mask': xxx,
                'vlan': xxx
            }
        response:
            {'id': xxx}
        http code: 
            200

    update:
        url: PUT http://endpoint:8774/nameserverhost/{id}
        eg: PUT http://endpoint:8774/nameserverhost/42dedd03-7feb-4835-ad14-0d64f8497204
        body:
            {
                'role': other,
                ....
            }
        response:
            http code: 204

    delete:
        url: DELETE http://endpoint:8774/nameserverhost/{id}
        eg: PUT http://endpoint:8774/nameserverhost/42dedd03-7feb-4835-ad14-0d64f8497204
        response data:
            None
        http code: 
            200

## NameServer(应用类) 
    note: bind9的options不在接口暴露了，由默认的模板定义
    url: http://endpoint:8774/nameserver 接口
    key field:
        id:
        label:
        views:
        zones:
        inuse_acl:

    method:
        get:
        add:
        delete:

    get:
        note: 只显示到zone的级别，不对record显示，因为record记录可能会很多，想要对record
             了解的，需要访问zone接口
        url: GET http://endpoint:8774/nameserver/{id}
        description: 获取name-server 的概要信息，包括views, zone(zone 只是zone_id), 
                  查询zone的信息，调用zone的接口
        response data:
            {
                id: '';
                views: [
                    'view-1': {
                        'id': 'xxx',
                        'acl': [{'acl_id01':xxx, 'acl_name': xxx, add_time: 'xxx'}, 
                                {'acl_id02':xxx, 'acl_name': xxx, add_time: 'xxx'},...]
                        'zones': [id1, id2, ... idn] 
                    },
                    'view-2': {
                        'id': 'xxx',
                        'acl': [{'acl_id05':xxx, 'acl_name': xxx, add_time: 'xxx'}, 
                                {'acl_id06':xxx, 'acl_name': xxx, add_time: 'xxx'},...]
                        'zones': [id1, id2, ... idn] 
                    },
                ]
            }
        http code:
            200
    add:
        note: 只是添加一个nameserver抽象
        url: POST http://endpoint:8774/{region_id}/nameserver
        body:
            {
                'label': 'xxx'
            }
        response data:
            {
                'id': 'xxxxx'
            }
        http code:
            200

    delete:
        note: 只是逻辑的删除一个nameserver抽象 
        url: DELETE http://endpoint:8774/nameserver/{id}
        response data:
            None
        http code:
            204

### NameServerACL (行为类)
    note: 在使用这个接口之前，需要先定义出ACL, 定义出view
    key field:
        nameserver_id:
        task_id:
        acl_id

    method:
        add_acl
        delete_acl

    add_acl:
        note: 在为nameserver添加acl之前，先申请acl
        url: POST http://endpoint:8774/nameserver/{nameserver_id}/action
        body:
            {
                'add_acl': ['acl_id01', 'acl_id02', ...]
            }
        response data:
            {
                'task_id': 'xxx',
            }
        http code:
            200

    delete_acl:
        url: POST http://endpoint:8774/nameserver/{id}/action
        body:
            {
                'delete_acl': ['acl_id01', 'acl_id02', ...]
            }
        response data:
            {
                'task_id': 'xxx',
            }
        http code:
            200
    
### View (行为类)
    note: acl 应用于在view上面，做智能dns使用, acl可以在NameServer接口中使用add_acl, 
          delete_acl,也可以在创建view的时候,设定acl
    url: http://endpoint:8774/view
    key field:
        view_id:
        view_name:
        acl:
        zones:
        zone_id:
        zone_name:
        task_id: xxxx 
        create_at
        update_at
        delete_at

    method:
        add
        delete
        get
        list

    get:
        url: GET http://endpoint:8774/view/{id}
        response data:
            {
                view_name: xxx,
                zones: [
                    {'zone_id': xxx, 'zone_name': 'xxx'},
                    {...}
                ],
                create_at:xxx,
                update_at:xxx,
                delete_at:xxx

            }
        http code:
            200

    list:
        url: GET http://endpoint:8774/view
            {
                views: [
                        {
                            view_name: xxx,
                            zones: [
                                    {'zone_id': xxx, 'zone_name': 'xxx'},
                                    {...}
                            ],
                            create_at:xxx,
                            update_at:xxx,
                            delete_at:xxx
                        },
                        {...}
                ]
            }
        http code: 
            200

    delete:
        url: DELETE http://endpoint:8774/view/{id}
        eg: DELETE http://endpoint:8774/view/82af4760-c4dc-4ce4-a06b-732fa7830eb5
        response data:
            {'task_id': 'xxxx'}
        http code:
            200
        <!-- tips: -->
            <!-- task_status: deleting->none(成功), deleting->failed(失败) -->
            <!-- task的状态通过task_id 进行查询 -->

    add:
        note: 这个过程可以接到添加view的指令，对文件操作，也可以暂时放到数据库，
        待在这个view下面添加zone的时候在一起操作
        url: POST http://endpoint:8774/view
        eg: POST http://endpoint:8774/view
        body:
            {
                view_name: xxxx
                acl: ['acl_id01', 'acl_id02'] (options) 
            }
        response data:
            {
                'view_id': xxxx,
                'task_id': 20f32371-a733-4bb7-a48e-acc635ce8c02,
            }
        http code:
            200
        <!-- tips: -->
            <!-- view status: processing -->
            <!-- task_status: adding->none(成功), adding->failed(失败) -->


### Zone (行为类)
    url: http://endpoint:8774/zone
    key field:
        zone_id: zone的逻辑id， 唯一
        zone_name: zone的名字, 唯一
        role:  zone的角色，master, slave, other, 应该都是master
        status: error, ok, processing 
        task_id:
        task_status:
        filename: zone的文件路径
        records: 用于记录当前zone下面record的集合，表示为records: [{'record_id': xxx, 
                                                                    'record_host': xxx,
                                                                    'record_type': xxx,
                                                                    'record_value': xxx},
                                                                    {...},
                                                                ]
        record_id: record_id的逻辑标示
        record_host: 可以是host，domain, 根据不同的record_type，record_host的意义也不一样
        record_type: 是dns record的类型，可以是A,AAAA, CNAME, NS等值
        record_value: 根据不同的record_type，value也不一样, 可以是ip地址，可以是其他domain
        create_at:
        update_at:
        delete_at:

    method:
        get:
        list:
        add:
        delete:

    get:
        url: GET http://endpoint:8774/zone/{id}
        response data:
            {
                'zone_name': xxx;
                'role': xxx;
                'status': xxx;
                'task_id': xxx;
                'filename': xxxx;
                'records': [
                    {
                        'record_id': xx, 
                        'record_host': xxx, 
                        'record_type': xx, 
                        'record_value': xxx
                    },
                    {...},
                ],
                'create_at': xxx,
                'delete_at': xxx,
                'update_at': xxx,
            }
        http code:
            200
    list:
        url: GET http://endpoint:8774/zone
        note: 对zone进行分页处理, page-size在配置文件可以定义, links中的id为当前zones列表
              最后一条记录的zone id
        response data:
            {
               'zones': [
                     {
                        'zone_name': xxx;
                        'role': xxx;
                        'status': xxx;
                        'task_id': xxx;
                        'filename': xxxx;
                        'records': [
                            {
                                'record_id': xx, 
                                'record_host': xxx, 
                                'record_type': xx, 
                                'record_value': xxx
                            },
                            {...},
                        ],
                        'create_at': xxx,
                        'delete_at': xxx,
                        'update_at': xxx,
                    },       
                    {...},
               ],
               'links':{
                    'href': http://endpoint:8774/zone?marker={id},
                    'rel': next
               }
            }
        http code:
            200

    add:
        url: POST http://endpoint:8774/zone
        body:
            {
                'zone_name': xxx,
                'records': ['record_id01', 'record_id02',...]
            }
        response data:
            {
                'zone_id': xxxx,
                'task_id': xxx
            }
        http code:
            200

    delete:
        url: DELETE http://endpoint:8774/zone/{id}
        response data:
            {
                'task_id': xxx;
            }
        http code:
            200

### Record(行为类)
    url: http://endpoint:8774/record
    key field:
        record_id:
        task_id:
        task_status: adding, deleting, updating, failed, none
        record_host:
        record_type:
        record_value:
        zone_id:
        zone_name:
        create_at:
        delete_at:
        update_at:
        status: error, ok, processing

    method:
        get:
        list:
        add:
        delete:
        update:

    get:
        url: GET http://endpoint:8774/record/{id}
        response data:
            {
                task_id: xxx,
                record_host: xxx,
                record_type: xxx,
                record_value: xxx,
                zone_id: xxx,
                zone_name: xxx,
                create_at:xxx,
                delete_at: xxx,
                update_at: xxx,
                status: xxx,
            }
        http code:
            200
    list:
        url: GET http://endpoint:8774/record
        response data:
            {
                'records': [
                    {
                        task_id: xxx,
                        record_host: xxx,
                        record_type: xxx,
                        record_value: xxx,
                        zone_id: xxx,
                        zone_name: xxx,
                        create_at:xxx,
                        delete_at: xxx,
                        update_at: xxx,
                        status: xxx,
                    },
                    {...}
                ],
               'links':{
                    'href': http://endpoint:8774/record?marker={id},
                    'rel': next
               }
            }
    add:
        url: POST http://endpoint:8774/record
        body:
            {
                zone_id: xxxx,
                record_host: xxx,
                record_type: xxx,
                record_value: xxx
            }
        response data:
            {
                'record_id': xxxx,
                'task_id': 9f90a028-a4c1-478a-8d9f-2db965a2e18c
            }
        http code:
            200
        tips:
            record status: processing
            task status: adding->none(成功), adding->failed(失败)

    delete:
        url: DELETE http://endpoint:8774/record/{id}
        response data:
            {
                task_id: 9f90a028-a4c1-478a-8d9f-2db965a2e18c
            }
        http code:
            200
        tips:
            record status: processing
            task status: deleting->none(成功), deleting->failed(失败)

    update:
        url: PUT http://endpoint:8774/record/{id}
        response data:
            {
                task_id: 9f90a028-a4c1-478a-8d9f-2db965a2e18c
            }
        http code:
            200
        tips:
            record status: processing
            task status: updating->none(成功), updating->failed(失败)
        
## ACL
    url: http://endpoint:8774/acl
    key field:
        acl_name:
        acl_id:
        rules: rule_id, address_prefix 
        inuse_views: 应用了当前acl的views
        operator:
        create_at:
        update_at:
        delete_at:

    method:
        get:
        list:
        add:
        delete:

    get:
        url: GET http://endpoint:8774/acl/{id}
        response data:
            {
                'acl_name': xxx;
                'rules': [
                    {
                        'rule_id': xxx, 
                        'address_prefix': xxx, 
                        'mode': xxx,
                        'operator': xxx
                    },
                    {...}
                ]
                'inuse_views': ['view_name01', 'view_name02', ...],
                'create_at': xxx,
                'delete_at': xxx,
                'operator': 'lisa'
            }
        http code:
            200

    list:
        url: GET http://endpoint:8774/acl
        response data:
            {
                acls: [
                    {
                        'acl_name': xxx,
                        'acl_id': xxx,   
                        'create_at': xxx,
                        'delete_at': xxx,
                        'operator': 'lisa'
                    }
                ]

            }
        http code:
            200
    add:
        url: POST http://endpoint:8774/acl
        body:
            {
                'acl_name': xxxx;
                'operator': xxx;
            }
        response data:
            {
                'acl_id': xxx,
                'task_id': xxxx,
            }
        http code:
            200

    delete:
        url: DELETE http://endpoint:8774/acl/{id}
        response data:
            {'task_id': 'd8ee861a-655a-4da0-857a-0eaa7a62231c'}
        http code:
            200
        tips:
            删除acl，会把下面的rule 也都删除掉。
            status: processing
            task status: deleting->none(成功), deleting-failed(失败)
        
### AclRule
    url: http://endpoint:8774/{acl_id}/rule
    key field
        rule_id:
        address_prefix: 可以是ip地址，以可以是网段用prefix方式表达， eg：
                        61.101.5.0/27
        mode: ['accept', 'drop'] accept: 表示允许通过, 
        acl_id:
        status:
        operator:
        create_at:
        delete_at:
        update_at:

    method:
        add:
        delete:
        update:

    add:
        url: POST http://endpoint:8774/{acl_id}/rule
        body:
            {
               'address_prefix': 192.168.89.0/24,
               'mode': accept
            }
        response data:
            {
                'rule_id': xxxx,
                'task_id': 'ab42fadc-1b7a-4d13-ae89-9e4d1dd1304a';
            }
        http code:
            200
        tips:
            这个接口只是逻辑上添加，并没有直接发指令到底层, 所有的rule添加以后，可以调用
            http://endpoint:8774/applyacl/{acl_id}

    delete:
        url: delete http://endpoint:8774/{acl_id}/rule
        response data:
            {
                'task_id': xxxx,
            }
        http code:
            204
        tips:
            这个接口只是逻辑上删除，并没有直接发指令到底层, 所有的rule删除以后，可以调用
            http://endpoint:8774/applyacl/{acl_id}
            

### Apply(行为类)
    url: http://endpoint:8774/apply
    key field:
        acl_id:
        task_id:
        create_at:
        delete_at:
        update_at:
    method:
        apply:
    apply:
        url: POST http://endpoint:8774/applyacl
        body:
            {
                'tasks': ['task_id01', 'task_id02', ...]
            }
        response data:
            {
                'task_id': '63dd9c06-8340-4f02-8467-1d5384286240'
            }
        http code:
            200
        tips:
            返回的task_id,为其他task的总任

## Task
    url: http://endpoint:8774/task
    key field:
        task_id
        task_status
        progress: 百分比
        create_at:
        complete_at:
        resource_id:
        resource_type:
        type:
        description:
        retry:
    method:
        list:
        get:
    list:
        url: GET http://endpoint:8774/task?latest={number}
        note: 默认为20条，最小1条，最多100条 配置文件可配置
        response:
            {
                tasks: [
                    {
                        'task_id': xxx,
                        'task_status':xxx,
                        'progress': xxx,
                        'create_at': xxx,
                        'complete_at': xxx,
                        'resource_id': xxx,
                        'resource_type': xxx,
                        'type': xx,
                        'description': xxx,
                        'retry'
                    },
                    {....}
                ]
            }
        http code:
            200

    get:
        url: GET http://endpoint:8774/task/{id}
        response data:
            {
                'task_id': xxx,
                'task_status':xxx,
                'progress': xxx,
                'create_at': xxx,
                'complete_at': xxx,
                'resource_id': xxx,
                'resource_type': xxx,
                'type': xx,
                'description': xxx,
                'retry'
            }
        http code:
            200
