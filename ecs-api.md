## 云服务器API接口文档

### 云服务器基本功能

| 接口                                              | 描述                       | url                                 |
| ------------------------------------------------- | -------------------------- | ----------------------------------- |
| [List Servers](#list-servers)                     | 查询云服务器列表           | GET /v1/servers/{pageNo}/{pageSize} |
| [Show Server Details](#show-server-details)       | 查询云服务器详情           | GET /v1/servers/{server_id}         |
| [Start Server](#start-server)                     | 启动单个云服务器           | POST /v1/servers/{server_id}/action |
| [Start Multiple Server](#start-multiple-server)   | 批量启动云服务器           | POST /v1/servers/action             |
| [Stop Server](#stop-server)                       | 关闭单个云服务器           | POST /v1/servers/{server_id}/action |
| [Stop Multiple Server](#stop-multiple-server)     | 批量关闭云服务器           | POST /v1/servers/action             |
| [Reboot Server](#reboot-server)                   | 重启单个云服务器           | POST /v1/servers/{server_id}/action |
| [Reboot Multiple Server](#reboot-multiple-server) | 批量重启云服务器           | POST /v1/servers/action             |
| [Get VNC Console URL](#get-novnc-console-url)     | 远程登录的时候获取远程链接 | POST /v1/servers/{server_id}/action |
| [List Server Tags](#list-server-tags)             | 获取云服务器tag数据        | GET    /v1/servers/{server_id}/tags |
| [Update Server Tags](#update-server-tags)         | 更新云服务器tags           | POST /v1/servers/{server_id}/tags   |

### 镜像

| 接口                                                | 描述                 | url                           |
| --------------------------------------------------- | -------------------- | ----------------------------- |
| [List Images](#list-images)                         | 查询当前用户镜像列表 | GET /v1/images                |
| [List All Private Images](#list-all-private-images) | 查询私有镜像列表     | GET /v1/images/private        |
| [Show Image Details](#show-image-details)           | 查询镜像详情         | GET /v1/images/{imageId}      |
| [Delete Image](#delete-image)                       | 删除镜像             | DELETE /v1/images/{image_id}  |
| [Delete Datch Image](#delete-datch-image)           | 批量删除镜像         | POST /v1/images/action/delete |
| [Update Image](#update-image)                       | 修改镜像             | PATCH /v1/images/{image_id}   |



## 公共返回值

每次请求的返回值必须包含下面三个字段。所有请求的数据都要放在data字段中，之后的接口返回参数说明和示例中不再包含此信息
，只介绍data字段中的内容。

|  名称   | 必填 |  类型  |                     说明                      |
| :-----: | :--: | :----: | :-------------------------------------------: |
|  code   |  Y   |  int   | 返回码（正确时返回200，错误时返回相应错误码） |
| message |  Y   | string |         返回信息（简要介绍错误详情）          |
|  data   |  N   |   T    |       返回数据主体（错误时，可返回空）        |

## 公共错误码

> 云服务器操作类常见公共错误码
>
> 注意：因为云服务器OSS错误码没有标准定义， 所以前3位暂时为：10x

| 错误码 | 错误信息                                   | Http状态码 | 说明                             |
| ------ | ------------------------------------------ | ---------- | -------------------------------- |
| 001012 | 存在未知状态的云服务器，请重新选择云服务器 | 400        | 云服务器不满足操作的前置状态要求 |
| 001002 | 参数xxx不符合规范                          | 400        | 请求参数不满足相关要求           |
| 001033 | 云服务器xxx不存在                          | 400        | 平台查询不到该云服务器           |
| 001040 | 操作失败                                   | 400        | 平台异常，操作失败               |

## 云服务器状态字段转义对照图

1.状态字段的转义如下

（1）取status字段，不区分大小写

| status       | translation   |
| :----------- | :------------ |
| ACTIVE       | 运行中        |
| DELETED      | 已删除        |
| ERROR        | 错误/创建失败 |
| PAUSED       | 暂停          |
| SOFT_DELETED | 软删除        |
| STOPPED      | 停止          |
| SHUTOFF      | 已关机        |
| BUILD        | 创建中        |
| REBOOT       | 软重启中      |
| HARD_REBOOT  | 硬重启中      |
| RESIZE       | 变更规格中    |
| REBUILD      | 重置系统中    |
| PASSWORD     | 重置密码中    |

（2）取task_state字段，不区分大小写

| task_state   | translation |
| :----------- | :---------- |
| powering-on  | 开机中      |
| powering-off | 关机中      |
| deleting     | 删除中      |

## 云服务器基本功能

### List Servers

描述信息：查询云服务器列表的各个参数信息。

URL："/v1/servers/{pageNo}/{pageSize}"

请求方式：HTTP.GET,

返回格式：json,

> 请求参数说明

| 名称        | 必填 |  类型  | 说明                                                         |
| :---------- | :--: | :----: | :----------------------------------------------------------- |
| searchValue |  N   | string | 搜索参数，按名称/ID/内网IP/状态四个条件进行筛选。为空时不进行过滤 。注：支持根据状态查询，支持查询全部和异常状态 |
| searchField |  N   | string | 搜索key                                                      |
| sortField   |  N   |  int   | 按哪个字段进行排序，为空时使用默认排序                       |
| sortOrder   |  N   |  int   | descend/ascend,升序排序/降序排序                             |
| page        |  N   |  int   | 查询页数                                                     |
| pageSize    |  N   |  int   | 每页显示条数                                                 |

*查询条件示例*：

| 查询目的               | 对应sortField值 | 对应的searchValue     |
| ---------------------- | --------------- | --------------------- |
| 查询异常状态服务器     | status          | unusual    （特定值） |
| 查询特定ID的服务器     | uuid            | xxx   (目标值)        |
| 查询特定名称的服务器   | name            | xxx   (目标值)        |
| 查询特定内网IP的服务器 | ip              | xxx   (目标值)        |

> 请求示例

```json
/v1/servers/2/10?sortField=createTime&sortOrder=descend&searchValue=abc
```

> 返回参数说明

| 名称              |  类型  |                说明                |
| :---------------- | :----: | :--------------------------------: |
| regionId          | string |              所在区域              |
| availability_zone | string |               所属AZ               |
| name              | string |            云服务器名称            |
| id                | string |             云服务器ID             |
| status            | string |            云服务器状态            |
| task_state        | string | 管理员用户使用，实例功能的当前状态 |
| image             | object |              镜像信息              |
| id                | string |               镜像ID               |
| system            | string |          镜像操作系统名称          |
| systemType        | string |          镜像操作系统类型          |
| tags              |  list  |              镜像标签              |
| root_volume       | object |             系统盘信息             |
| volumetype        | string |               卷类型               |
| size              |  int   |             系统盘大小             |
| data_volumes      |  list  |             数据盘信息             |
| volumetype        | string |               卷类型               |
| volumeid          | string |                卷ID                |
| volumename        | string |               卷名称               |
| size              |  int   |             数据盘大小             |
| flavor            | object |              规格信息              |
| disk              |  int   |              规格大小              |
| extra_specs       | object |             规格元数据             |
| original_name     | string |                名称                |
| ram               |  int   |                内存                |
| swap              |  int   |              交换信息              |
| vcpus             |  int   |              cpu核数               |
| security_groups   |  list  |             安全组信息             |
| vpcid             | string |              vpu的ID               |
| vpcname           | string |              vpc名称               |
| nics              |  list  |            网络相关信息            |
| subnet_id         | string |               子网id               |
| subnet_name       | string |              子网名称              |
| subnet_cidr       | string |              子网cidr              |
| network_id        | string |              网络号id              |
| mac_address       | string |              mac地址               |
| fixed_ip          | string |               内网ip               |
| portid            | string |              port id               |
| eip               | object |              eip信息               |
| eipid             | string |               eip ID               |
| iptype            | string |                类型                |
| bandwidth         |  int   |                带宽                |
| chargemode        | string |                模式                |
| ip_addr           | string |               外网IP               |
| key_name          | string |              秘钥名称              |
| feeData           | object |              计费信息              |
| chargetype        | string |              计费方式              |
| startFeeTime      | string |            开始计费时间            |
| nextFeeTime       | string |              到期时间              |
| created           | string |            虚机创建时间            |
| hostId            | string |          云服务器宿主机Id          |
| host              | string |             宿主机名称             |

> 返回json实例

```json
{
    "code": "200",
    "message": "ECS application ECS module message:OK",
    "data": {
    "totalCount":1,
	"servers": [{
		"regionID": "cn-north-3",
		"availability_zone": "nova",
		"name": "newserver",
		"id": "9168b536-cd40-4630-b43f-b259807c6e87",
		"status": "active",
		"task_state": "xxx",
		"image": {
			"id": "1189efbf-d48b-46ad-a823-94b942e2a000",
			"name": "centos70X64",
			"systemType": "Centos",
			"systemType": "Linux",
			"tags": [
				"xxx",
				"xxx"
			]
		},
		"root_volume": {
			"volumetype": "SATA",
			"size": 50
		},
		"data_volumes": [{
				"volumetype": "SATA",
				"volumeid": "zzz",
				"volumename": "aaa",
				"size": 100
			},
			{
				"volumetype": "SATA",
				"volumeid": "bbb",
				"volumename": "ccc",
				"size": 100
			}
		],
		"flavor": {
			"disk": 50,
			"ephemeral": 0,
			"extra_specs": {
				"hw:cpu_policy": "dedicated",
				"hw:mem_page_size": "2048"
			},
			"original_name": "s1.xlarge.2",
			"ram": 1,
			"swap": 0,
			"vcpus": 1
		},
		"security_groups": [{
			"name": "sg_aa1"
		}],
		"vpcid": "0dae26c9-9a70-4392-93f3-87d53115d171",
		"vpcname": "myvpc",
        "hostId": "12efd5567jklu",
		"nics": [{
			"subnet_id": "157ee789-03ea-45b1-a698-76c92660dd83",
			"subnet_name": "xxxx",
			"subnet_cidr": "xxxx",
			network_id: "5d0180c1-ddaa-4582-9111-6332e0b091e1",
			mac_address: "fa:16:3e:dc:d1:aa",
			"fixed_ip": "10.20.30.137",
			"portid": "xxx"
		}],
		"eip": {
		    "eipid": "xxx",
			"iptype": "5 _bgp",
			"bandwidth": 10,
			"chargemode": "Bandwidth",
			"ip_addr": "100.100.100.100"
		},
		"key_name": "sshkey-123",
		"feeData": {
			"chargetype": "monthly",
			"startFeeTime": "2012-08-20 21:11:09",
			"nextFeeTime": "2012-08-20 21:11:09"
		},
		"created": "2012-08-20T21:11:09Z"
	}]
}
```

### Show Server Details

描述：查询某一个虚机的详细信息

接口路径："/v1/servers/{server_id}"

请求方式：HTTP.GET,

返回格式：json,

备注：

> 请求参数说明

> *无请求参数*

> 返回参数

| 名称              |  类型  |                说明                |
| :---------------- | :----: | :--------------------------------: |
| regionId          | string |              所在区域              |
| availability_zone | string |               所属AZ               |
| name              | string |            云服务器名称            |
| id                | string |             云服务器ID             |
| status            | string |            云服务器状态            |
| task_state        | string | 管理员用户使用，实例功能的当前状态 |
| image             | object |              镜像信息              |
| id                | string |               镜像ID               |
| system            | string |          镜像操作系统名称          |
| systemType        | string |          镜像操作系统类型          |
| tags              |  list  |              镜像标签              |
| root_volume       | object |             系统盘信息             |
| volumetype        | string |               卷类型               |
| size              |  int   |             系统盘大小             |
| data_volumes      |  list  |             数据盘信息             |
| volumetype        | string |               卷类型               |
| volumeid          | string |                卷ID                |
| volumename        | string |               卷名称               |
| size              |  int   |             数据盘大小             |
| flavor            | object |              规格信息              |
| disk              |  int   |              规格大小              |
| extra_specs       | object |             规格元数据             |
| original_name     | string |                名称                |
| ram               |  int   |                内存                |
| swap              |  int   |              交换信息              |
| vcpus             |  int   |              cpu核数               |
| security_groups   |  list  |             安全组信息             |
| vpcid             | string |              vpu的ID               |
| vpcname           | string |              vpc名称               |
| nics              |  list  |            网络相关信息            |
| subnet_id         | string |               子网ID               |
| subnet_name       | string |              子网名称              |
| subnet_cidr       | string |              子网cidr              |
| network_id        | string |              网络号ID              |
| mac_address       | string |              MAC地址               |
| fixed_ip          | string |               内网IP               |
| portid            | string |              port ID               |
| eip               | object |              eip信息               |
| eipid             | string |               eip ID               |
| iptype            | string |                类型                |
| bandwidth         |  int   |                带宽                |
| chargemode        | string |                模式                |
| ip_addr           | string |               外网IP               |
| key_name          | string |              秘钥名称              |
| feeData           | object |              计费信息              |
| chargetype        | string |              计费方式              |
| startFeeTime      | string |            开始计费时间            |
| nextFeeTime       | string |              到期时间              |
| created           | string |            虚机创建时间            |
| hostId            | string |              宿主机Id              |
| host              | string |             宿主机名称             |
| keyCloakUserId    | string |            控制台用户Id            |
| keyCloakUserName  | string |           控制台用户名称           |

> 返回json实例

```json
{
    "code": 200,
    "message": "OK",
    "data": {
        "server": {
            "regionID": "cn-north-3",
            "availability_zone": "nova",
            "name": "newserver",
            "id": "9168b536-cd40-4630-b43f-b259807c6e87",
            "status": "active",
            "task_state": "xxx",
            "image": {
                "id": "1189efbf-d48b-46ad-a823-94b942e2a000",
                "name": "centos70X64",
                "systemType": "Centos",
                "tags": [
                    "xxx",
                    "yyy"
                ]
            },
            "root_volume": {
                "volumetype": "SATA",
                "size": 50
            },
            "data_volumes": [
                {
                    "volumetype": "SATA",
                    "volumeid": "zzz",
                    "volumename": "aaa",
                    "size": 100
                },
                {
                    "volumetype": "SATA",
                    "volumeid": "bbb",
                    "volumename": "ccc",
                    "size": 100
                }
            ],
            "flavor": {
                "disk": 50,
                "ephemeral": 0,
                "extra_specs": {
                    "hw:cpu_policy": "dedicated",
                    "hw:mem_page_size": "2048"
                },
                "original_name": "s1.xlarge.2",
                "ram": 1,
                "swap": 0,
                "vcpus": 1
            },
            "security_groups": [
                {
                    "name": "sg_aa1"
                }
            ],
            "vpcid": "0dae26c9-9a70-4392-93f3-87d53115d171",
            "vpcname": "myvpc",
            "nics": [
                {
                    "subnet_id": "157ee789-03ea-45b1-a698-76c92660dd83",
                    "subnet_name": "xxxx",
                    "subnet_cidr": "xxxx",
                    "fixed_ip": "10.20.30.137",
                    "portid": "xxx"
                }
            ],
            "eip": {
                "eipid": "xxx",
                "iptype": "5 _bgp",
                "bandwidth": 10,
                "chargemode": "Bandwidth",
                "ip_addr": "100.100.100.100"
            },
            "key_name": "sshkey-123",
            "feeData": {
                "chargetype": "prePaid",
                "purchasetime": "1",
                "startFeeTime": "2012-08-20 21:11:09",
                "nextFeeTime": "2012-08-20 21:11:09"
            },
            "created": "2012-08-20T21:11:09Z"
        }
    }
}
```

### Start Server

描述信息：启动单个云服务器，前置状态为【已关机】

URL： "/v1/servers/{server_id}/action"

请求方式：HTTP.POST

备注：启动云服务器单个云服务器

> 请求参数说明

| 名称      | 位置 | 必填 |  类型  | 说明                                 |
| :-------- | ---- | :--: | :----: | :----------------------------------- |
| server_id | path |  Y   | string | 云服务器ID                           |
| os-start  | body |  Y   | object | 标记为启动云服务器操作，数据结构为空 |

> 请求示例

```json
{
    "os-start": {}
}
```

> 返回参数说明

请参考[公共返回值](#公共返回值)

> 错误码

请参考[公共错误码-云服务操作常见错误码](#公共错误码)

### Start multiple Server

描述信息：批量开启云服务器，所有云服务器前置都得为【已关机】

URL： "/v1/servers/action"

请求方式：HTTP.POST

备注：批量启动云服务器，1<=云服务器数量<=100

> 请求参数说明

| 名称     | 路径 | 必填 |  类型  | 说明                             |
| :------- | ---- | :--: | :----: | :------------------------------- |
| os-start | body |  Y   | object | 特定值，标记为启动云服务器操作。 |
| servers  | body |  Y   | array  | 批量操作服务器ID字符组           |

> 请求示例

```json
{
    "os-start": {
        "servers": [
            {
                "id": "616fb98f-46ca-475e-917e-2563e5a8cd19"
            },
            {
                "id": "726fb98f-46ca-475e-917e-2563e5a8cd20"
            }
        ]
    }
}
```

> 返回参数

请参考[公共返回值](#公共返回值)

> 错误码

请参考[公共错误码-云服务操作常见错误码](#公共错误码)

### Stop Server

描述信息：关闭单个云服务器，前置状态为【运行中】

URL： "/v1/servers/{server_id}/action"

请求方式：HTTP.POST

备注：关闭单个云服务器

> 请求参数说明

| 名称      | 路径 | 必填 |  类型  | 说明                                 |
| :-------- | ---- | :--: | :----: | :----------------------------------- |
| server_id | path |  Y   | string | 云服务器ID                           |
| os-stop   | body |  Y   | object | 标记为关闭云服务器操作，数据结构为空 |

> 请求示例

```json
{
    "os-stop": {}
}


```

> 返回参数

请参考[公共返回值](#公共返回值)

> 错误码

| 错误码     | 错误信息                                     | Http状态码 | 说明                                       |
| ---------- | -------------------------------------------- | ---------- | ------------------------------------------ |
| 101.001012 | 存在未知状态的云服务器，请重新选择云服务器   | 400        | 云服务器前置状态不满足，应该处于运行中状态 |
| 101.001013 | 存在未知所属的云服务器，请重新选择云服务器   | 400        | 云服务器不属于操作用户，或者不存在         |
| 101.001014 | 存在未付费类型的云服务器，请重新选择云服务器 | 400        | 云服务器无计费信息                         |

### Stop multiple Server

描述信息：批量关闭云服务器，全部服务器前置状态都得为【运行中】

URL： "/v1/servers/action"

请求方式：HTTP.POST

备注：批量关闭云服务器， 1<=云服务器数量<=100

> 请求参数说明

| 名称    | 路径 | 必填 |  类型  | 说明                     |
| :------ | ---- | :--: | :----: | :----------------------- |
| os-stop | body |  Y   | object | 标记为关闭云服务器操作。 |
| servers | body |  Y   | array  | 批量操作服务器ID字符组   |

> 请求示例

```json
{
    "os-stop": {
        "type":"HARD",
        "servers": [
            {
                "id": "616fb98f-46ca-475e-917e-2563e5a8cd19"
            },
            {
                "id": "726fb98f-46ca-475e-917e-2563e5a8cd20"
            }

        ]
    }
}

```

> 返回参数

*请参考[公共返回值](#公共返回值)*

> 错误码

请参考[公共错误码-云服务操作常见错误码](#公共错误码)

### Reboot Server

描述信息：重启单个云服务器，前置状态为【运行中】

URL： "/v1/servers/{server_id}/action"

请求方式：HTTP.POST

备注：重启单个云服务器

> 请求参数说明

| 名称      |      | 必填 |  类型  | 说明                                 |
| :-------- | ---- | :--: | :----: | :----------------------------------- |
| server_id | path |  Y   | string | 云服务器ID                           |
| reboot    | body |  Y   | object | 标记为重启云服务器操作，数据结构为空 |

> reboot字段说明

| 名称 |      | 必填 |  类型  | 说明                                                         |
| :--- | ---- | :--: | :----: | :----------------------------------------------------------- |
| type | body |  N   | string | 重启类型，默认为SOFT，SOFT：普通重启（默认）。HARD：强制重启。 |

注意：SOFT类型的重启，云服务器的状态必须为ACTIVE, 想要支持其他状态，建议用HARD。

> 请求示例

```json
{
    "reboot": {
        "type": "SOFT"
    }
}

```

> 返回参数

请参考[公共返回值](#公共返回值)

> 错误码

请参考[公共错误码-云服务操作常见错误码](#公共错误码)

### Reboot Multiple Server

描述信息：批量重启云服务器，全部云服务器前置状态都为【运行中】

URL： "/v1/servers/action"

请求方式：HTTP.POST

备注：批量重启云服务器

> 请求参数说明

| 名称   | 路径 | 必填 |  类型  | 说明                     |
| :----- | ---- | :--: | :----: | :----------------------- |
| reboot | body |  Y   | object | 标记为重启云服务器操作。 |

> reboot字段说明

| 名称    | 路径 | 必填 |  类型  | 说明                                                         |
| :------ | ---- | :--: | :----: | :----------------------------------------------------------- |
| type    | body |  N   | string | 重启类型，默认为SOFT，SOFT：普通重启（默认）。HARD：强制重启。 |
| servers | body |  Y   | array  | 批量操作服务器ID字符组                                       |

注意：SOFT类型的重启，云服务器的状态必须为ACTIVE, 想要支持其他状态，建议用HARD。

> 请求示例

```json
{
    "reboot": {
        "type":"SOFT",
        "servers": [
            {
                "id": "616fb98f-46ca-475e-917e-2563e5a8cd19"
            },
            {
                "id": "726fb98f-46ca-475e-917e-2563e5a8cd20"
            }

        ]
    }
}

```

> 返回参数

请参考[公共返回值](#公共返回值)

> 错误码

请参考[公共错误码-云服务操作常见错误码](#公共错误码)

### Get noVNC Console URL

描述信息：获取远程登录的链接信息

URL： "/v1/servers/{server_id}/action"

请求方式：HTTP.POST

备注：远程登录的时候获取远程链接

> 请求参数说明

| 名称             | 路径 | 必填 |  类型  | 说明                                     |
| :--------------- | ---- | :--: | :----: | :--------------------------------------- |
| server_id        | path |  Y   | string | server ID                                |
| os-getVNCConsole | body |  Y   | object | 操作标识符，特定值                       |
| type             | body |  Y   | String | VNC控制台的类型。有效值为novnc和xvpvnc。 |

> 请求示例

```json
{
    "os-getVNCConsole": {
        "type": "novnc"
    }
}



```

> 返回参数data说明：其他参考公共返回值

| 名称    | 路径 | 必填 | 类型   | 说明                                     |
| ------- | ---- | ---- | ------ | ---------------------------------------- |
| console | body | -    | object | 返回值标识                               |
| url     | body | -    | string | 返回的远程登录链接地址                   |
| type    | body | -    | string | VNC控制台的类型。有效值为novnc和xvpvnc。 |

> 返回参数示例

```json
{
	"code": 200,
	"message": "Successful operation!",
	"data": {
		"console": {
			"url": "http://console.staging.inspur.com/ecs/vnc/vnc_auto.html?token=b1b939af-279e-4151-befd-bed3792595ed",
			"type": "novnc"
		}
	}
}


```

> 错误码

请参考[公共错误码-云服务操作常见错误码](#公共错误码)

### List Server tags

描述信息：列出云服务器的tag

URL： "/v1/servers/{server_id}/tags"

请求方式：HTTP.GET

备注：列出云服务器的tag,用于更新是显示云服务器已有tag

> 请求参数说明

| 名称      |      | 必填 |  类型  | 说明       |
| :-------- | ---- | :--: | :----: | :--------- |
| server_id | path |  Y   | string | 云服务器ID |

> 请求示例

无参数，直接请求

> 返回参数说明

| 名称      | 类型         | 说明           |
| --------- | ------------ | -------------- |
| code      | int          | 请求结果的状态 |
| message   | string       | 请求返回的信息 |
| data      | object       | 请求返回的数据 |
| data.tags | list<Stirng> | list tag数据   |

> 返回参数示例

```json
{
    "code": 200,
    "message": "success",
    "data": {
        "tags": [
            "ECS"
        ]
    }
}
```

> 错误码

请参考[公共错误码-云服务操作常见错误码](#公共错误码)

### Update Server tags

描述信息：更新云服务器的tag

URL： "/v1/servers/{server_id}/tags"

请求方式：HTTP.POST

备注：更新云服务器的tag, 增加和删除

> 请求参数说明

| 名称      | 路径   | 必填 |  类型  | 说明                        |
| :-------- | ------ | :--: | :----: | :-------------------------- |
| server_id | path   |  Y   | string | 云服务器ID                  |
| tags      | params |  Y   | object | 更新之后目标服务器的所有tag |

> tags字段说明

| 名称 | 路径   | 必填 |  类型  | 说明                         |
| :--- | ------ | :--: | :----: | :--------------------------- |
| tags | params |  N   | string | 支持多个，中间用英文逗号隔开 |

> 请求示例

| params.key | params.value          |
| ---------- | --------------------- |
| tags       | tag1,tag2,xxx,...,yyy |

> 返回参数

请参考[公共返回值](#公共返回值)

> 错误码

请参考[公共错误码-云服务操作常见错误码](#公共错误码)

## 镜像

### Create private image

描述：通过虚机创建自定义镜像。

接口路径："/v1/servers/{server_id}/action",

请求方式：HTTP.POST,

返回格式：json,

> 请求参数说明

| 名称        | 必填 |  类型  | 说明         |
| :---------- | :--: | :----: | :----------- |
| tenant_id   |  Y   | string | 项目ID       |
| server_id   |  Y   | string | 云服务器的ID |
| name        |  Y   | string | 镜像名称     |
| description |  Y   | string | 镜像描述     |

> 请求参数说明

```json
{
     "createImage" : {
         "name" : "云服务器名称",
         "description":"填写描述信息",
         "metadata": {    // metadata可不填，键值也不传
            "meta_var": "meta_val"
        }
     }
 }
```

> 返回参数说明

| 名称     |  类型  | 说明               |
| :------- | :----: | :----------------- |
| image_id | string | 创建成功后生成的ID |

> 返回参数

```json
{
  "message": "Created",
  "code": 201,
  "data": {
    "image_id": "xxx"
  }
}
```

> 错误码

| 错误码     | 错误信息                       | Http状态码 |
| ---------- | ------------------------------ | ---------- |
| 101.003007 | 创建自定义镜像失败             | 400        |
| 101.003011 | 虚机不能创建自定义镜像,状态:%s | 400        |

### List images

描述：查询当前用户镜像列表。

接口路径："/v1/images",

请求方式：HTTP.GET,

返回格式：json,

备注：1.私有镜像状态字段转义

| status                         | translation   |
| :----------------------------- | :------------ |
| queued(排队中)、saving(保存中) | 创建中        |
| killed                         | 创建失败/错误 |
| active                         | 可用          |
| pending_delete                 | 删除中        |

> 请求参数说明

| 名称        | 必填 |  类型  | 说明                                                  |
| :---------- | :--: | :----: | :---------------------------------------------------- |
| searchValue |  N   | string | 搜索参数，按名称/ID两个条件进行筛选。为空时不进行过滤 |
| sortField   |  N   |  int   | 按哪个字段进行排序，为空时使用默认排序                |
| sortOrder   |  N   |  int   | descend/ascend,升序排序/降序排序                      |
| page        |  N   |  int   | 查询页数                                              |
| pageSize    |  N   |  int   | 每页显示条数                                          |
| visibility  |  N   | string | private：私有镜像,固定为private                       |

> 请求示例

> ```json
> {
> "searchValue":"abc",
> "sortField":"createTime",
> "sortOrder":"descend",
> "page":1,
> "pageSize":10,
> "visibility": "private"
> }
> ```

> 返回参数说明

| 名称             |  类型  | 说明             |
| :--------------- | :----: | :--------------- |
| id               | string | 镜像ID           |
| name             | string | 镜像名称         |
| tags             | string | 镜像标签         |
| status           | string | 镜像状态         |
| visibility       | string | 用户可见性       |
| description      | string | 描述信息         |
| system           | string | 镜像操作系统名称 |
| systemType       | string | 镜像操作系统类型 |
| container_format | string | 镜像种类         |
| disk_format      | string | 镜像格式         |
| created_at       | string | 创建时间         |
| updated_at       | string | 更新时间         |
| protected        | string | 权限             |

> 返回参数

> ```json
> {
>  "code": 200,
>  "message": "OK",
>  "data": {
>      "totalCount": 16,
>      "images": [
>          {
>              "id": "46cb2a76-785d-4bb7-9d97-18fe86b4129a",
>              "name": "centos71x86_64.qcow2",
>              "tags": [
>                  "inhost",
>                  "centos"
>              ],
>              "status": "active",
>              "checksum": "e01867770828b02a63ebb037cbd2d319",
>              "owner": "140785795de64945b02363661eb9e769",
>              "visibility": "public",
>              "size": 1257963520,
>              "locations": [
>                  {
>                      "url": "rbd://ff140a2b-9bca-4fa4-926a-866b439138a4/images/46cb2a76-785d-4bb7-9d97-18fe86b4129a/snap",
>                      "metadata": {}
>                  }
>              ],
>              "self": "/v2/images/46cb2a76-785d-4bb7-9d97-18fe86b4129a",
>              "file": "/v2/images/46cb2a76-785d-4bb7-9d97-18fe86b4129a/file",
>              "schema": "/v2/schemas/image",
>              "description": "inhost windows",
>              "system": "centos",
>              "systemType": "linux",
>              "container_format": "bare",
>              "disk_format": "qcow2",
>              "created_at": "2018-09-07T12:14:25.000+0000",
>              "updated_at": "2018-09-07T13:40:22.000+0000",
>              "min_disk": 0,
>              "min_ram": 0,
>              "protected": false,
>              "direct_url": "rbd://ff140a2b-9bca-4fa4-926a-866b439138a4/images/46cb2a76-785d-4bb7-9d97-18fe86b4129a/snap"
>          }
>      ]
>  }
> }
> ```





### List all private images

描述：查询私有镜像列表。

接口路径："/v1/images/private",

请求方式：HTTP.GET,

返回格式：json,

备注：创建页面获取用户所有私有镜像，返回全部值，此接口不进行分页

> 请求参数说明

| 名称 | 必填 |  类型  | 说明 |
| :--- | :--: | :----: | :--- |
| tag  |  Y   | string | ECS  |

> 返回参数说明

| 名称             |  类型   | 说明             |
| :--------------- | :-----: | :--------------- |
| id               | string  | 镜像ID           |
| name             | string  | 镜像名称         |
| tags             | string  | 镜像标签         |
| status           | string  | 镜像状态         |
| visibility       | string  | 用户可见性       |
| description      | string  | 描述信息         |
| system           | string  | 镜像操作系统名称 |
| systemType       | string  | 镜像操作系统类型 |
| container_format | string  | 镜像种类         |
| disk_format      | string  | 镜像格式         |
| created_at       | string  | 创建时间         |
| updated_at       | string  | 更新时间         |
| protected        | boolean | 权限             |

> 返回参数
>
> ```json
> {
>  "code": 200,
>  "message": "OK",
>  "data": {
>      "totalCount": 16,
>      "images": [
>          {
>              "id": "46cb2a76-785d-4bb7-9d97-18fe86b4129a",
>              "name": "centos71x86_64.qcow2",
>              "tags": [
>                  "inhost",
>                  "centos"
>              ],
>              "status": "active",
>              "checksum": "e01867770828b02a63ebb037cbd2d319",
>              "owner": "140785795de64945b02363661eb9e769",
>              "visibility": "public",
>              "size": 1257963520,
>              "locations": [
>                  {
>                      "url": "rbd://ff140a2b-9bca-4fa4-926a-866b439138a4/images/46cb2a76-785d-4bb7-9d97-18fe86b4129a/snap",
>                      "metadata": {}
>                  }
>              ],
>              "self": "/v2/images/46cb2a76-785d-4bb7-9d97-18fe86b4129a",
>              "file": "/v2/images/46cb2a76-785d-4bb7-9d97-18fe86b4129a/file",
>              "schema": "/v2/schemas/image",
>              "description": "inhost windows",
>              "system": "centos",
>              "systemType": "linux",
>              "container_format": "bare",
>              "disk_format": "qcow2",
>              "created_at": "2018-09-07T12:14:25.000+0000",
>              "updated_at": "2018-09-07T13:40:22.000+0000",
>              "min_disk": 0,
>              "min_ram": 0,
>              "protected": false,
>              "direct_url": "rbd://ff140a2b-9bca-4fa4-926a-866b439138a4/images/46cb2a76-785d-4bb7-9d97-18fe86b4129a/snap"
>          }
>      ]
>  }
> }
> ```

### Show Image Details

接口路径："/v1/images/{imageId}",

请求方式：HTTP.GET,

返回格式：json,

> 请求参数说明

> 返回参数
>
> ```json
> {
>  "code": 200,
>  "message": "OK",
>  "data": {
>      "id": "253a26bb-f2f8-4443-8152-5c505a4ad7cd",
>      "name": "dr0919ws",
>      "tags": [
>          "SYSTEMDISK_1",
>          "centos7.1",
>          "ECS"
>      ],
>      "status": "active",
>      "checksum": "e9ad8c8558ab399b02b0cdd69c9bbdb6",
>      "owner": "35ee734f83814f039084c9b55228e671",
>      "visibility": "private",
>      "size": 1073741824,
>      "locations": [
>          {
>              "url": "rbd://ff140a2b-9bca-4fa4-926a-866b439138a4/images/253a26bb-f2f8-4443-8152-5c505a4ad7cd/snap",
>              "metadata": {}
>          }
>      ],
>      "self": "/v2/images/253a26bb-f2f8-4443-8152-5c505a4ad7cd",
>      "file": "/v2/images/253a26bb-f2f8-4443-8152-5c505a4ad7cd/file",
>      "schema": "/v2/schemas/image",
>      "system": "centos7.1",
>      "systemType": "linux",
>      "systemDisk": "1",
>      "container_format": "bare",
>      "disk_format": "raw",
>      "created_at": "2018-09-27T08:45:55.000+0000",
>      "updated_at": "2018-09-27T08:47:02.000+0000",
>      "min_disk": 1,
>      "min_ram": 0,
>      "protected": false,
>      "direct_url": "rbd://ff140a2b-9bca-4fa4-926a-866b439138a4/images/253a26bb-f2f8-4443-8152-5c505a4ad7cd/snap",
>      "instance_uuid": "e119c583-78fe-4abb-b1ff-e88533200278"
>  }
> }
> ```



### Delete image

接口路径："/v1/images/{image_id}",

请求方式：HTTP.DELETE,

返回格式：json,

> 请求参数

| 名称     | 必填 |  类型  | 说明   |
| :------- | :--: | :----: | :----- |
| image_id |  Y   | string | 镜像ID |

> 返回示例

```json
{
   "code": 200,
   "data": ""
}
```

> 错误码

| 错误码     | 错误信息                             | Http状态码 |
| ---------- | ------------------------------------ | ---------- |
| 101.003006 | 删除镜像失败                         | 500        |
| 101.003015 | 当前镜像无法删除，已关联云服务器"XX" | 400        |

### Delete Datch image

接口路径："/v1/images/delete",

请求方式：HTTP.POST,

返回格式：json,

备注：同步接口

> 请求参数

| 名称 | 必填 |  类型  | 说明   |
| :--- | :--: | :----: | :----- |
| id   |  Y   | string | 镜像ID |

> 请求示例

> ```json
> {
>  "images": [
>      {
>          "id": "ac476316-caa1-4561-bbe4-7dcde2ff703a"
>      },
>      {
>          "id": "ac476316-caa1-4561-bbe4-7dcde2ff703a"
>      },
>      {
>          "id": "ac476316-caa1-4561-bbe4-7dcde2ff703a"
>      }
>  ]
> }
> ```

> 返回参数

| 名称   |  类型  | 说明         |
| :----- | :----: | :----------- |
| status | string | 返回状态信息 |

> 返回示例

```json
{
   "code": 200,
   "message": OK,
   "data": {
     "status": "删除成功2台,删除失败1台"
   }
}
```

错误码

| 错误码     | 错误信息     | Http状态码 | 说明         |
| ---------- | ------------ | ---------- | ------------ |
| 101.003006 | 删除镜像失败 | 500        | 删除镜像失败 |

### Update image

接口路径："/v1/images/{image_id}",

请求方式：HTTP.PATCH,

返回格式：json,

> 请求参数说明

| 名称  | 必填 |  类型  | 说明     |
| :---- | :--: | :----: | :------- |
| op    |  Y   | string | 修改方式 |
| path  |  Y   | string | 修改字段 |
| value |  Y   | string | 修改内容 |

> 请求示例

> ```json
> [
>  {
>      "op": "add",
>      "path": "/name",
>      "value": "填写修改名称的值"
>  },
>  {
>      "op": "add",
>      "path": "/description",
>      "value": "填写修改描述的值"
>  }
> ]
> ```

> 返回参数

```json
{
   "code": 200,
   "data":"",
   "message": "update image success"
}
```

> 错误码

| 错误码     | 错误信息                 | Http状态码 |
| ---------- | ------------------------ | ---------- |
| 101.003005 | 更新镜像失败             | 500        |
| 101.003016 | 创建中的镜像不可进行修改 | 400        |
| 101.003017 | 非私有镜像不可进行修改   | 400        |

> 

```json
{
	"consoleOrderFlowId": "9ee4612f-82a4-4239-9149-e6d05b0f216b",
	"orderId": "178549870045495296",
	"orderStatus": "paySuccess",
	"statusTime": "2019-05-08 16:53:27",
	"token": "eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJsY2hRX2ZrNFdHN0hCZFpmdkdRLUxxWTUwTWxVQVUwb1ZYUU1KcVF0UjNzIn0.eyJqdGkiOiI3Y2NjYmNlMi01OTkzLTRiMzAtYjAyZC1iMzkxMTYxYTg2ZTAiLCJleHAiOjE1NTczMTA4NzMsIm5iZiI6MCwiaWF0IjoxNTU3MzA1NDczLCJpc3MiOiJodHRwczovL2lvcGRldi4xMC4xMTAuMjUuMTIzLnhpcC5pby9hdXRoL3JlYWxtcy9waWNwIiwiYXVkIjpbImluc2lnaHQiLCJyZWFsbS1tYW5hZ2VtZW50IiwiaW90LWh1YiIsImNsaWVudC1nYW9zcyIsImRiLXNlcnZpY2UiLCJhY2NvdW50IiwicmRzLW15c3FsLWFwaSJdLCJzdWIiOiJmNjc5ZWIyMy1mNzRkLTQ3N2EtYTMxZi1hNjU2ZDcxNjA0ZWQiLCJ0eXAiOiJCZWFyZXIiLCJhenAiOiJjb25zb2xlIiwibm9uY2UiOiI2MjkzNDM4Mi1hODhiLTRkMDktOGU2Yy0yZTQzOWM4ODI3ZmEiLCJhdXRoX3RpbWUiOjE1NTczMDU0MTYsInNlc3Npb25fc3RhdGUiOiIyZjA4NmExNi1kZjY4LTRjNzAtOTg0Zi0zMjFiNDhhYmFmMmEiLCJhY3IiOiIxIiwiYWxsb3dlZC1vcmlnaW5zIjpbIioiXSwicmVhbG1fYWNjZXNzIjp7InJvbGVzIjpbIkFDQ09VTlRfQURNSU4iLCJvZmZsaW5lX2FjY2VzcyIsIk9QRVJBVEVfQURNSU4iLCJ1bWFfYXV0aG9yaXphdGlvbiIsIkFDQ09VTlRfVVNFUiJdfSwicmVzb3VyY2VfYWNjZXNzIjp7Imluc2lnaHQiOnsicm9sZXMiOlsiYWRtaW4iXX0sInJlYWxtLW1hbmFnZW1lbnQiOnsicm9sZXMiOlsidmlldy11c2VycyIsInF1ZXJ5LWdyb3VwcyIsInF1ZXJ5LXVzZXJzIl19LCJpb3QtaHViIjp7InJvbGVzIjpbImFkbWluIl19LCJjbGllbnQtZ2Fvc3MiOnsicm9sZXMiOlsiaWFhc19tcSIsImFkbWluIiwidXNlciJdfSwiZGItc2VydmljZSI6eyJyb2xlcyI6WyJhZG1pbiJdfSwiYWNjb3VudCI6eyJyb2xlcyI6WyJtYW5hZ2UtYWNjb3VudCIsIm1hbmFnZS1hY2NvdW50LWxpbmtzIiwidmlldy1wcm9maWxlIl19LCJyZHMtbXlzcWwtYXBpIjp7InJvbGVzIjpbInVzZXIiXX19LCJzY29wZSI6Im9wZW5pZCIsImludml0ZWRfcmVnaW9uIjoiW1wiY24tc291dGgtMVwiLFwiY24tc291dGgtMlwiXSIsInN2YyI6IltcIklPVFwiLFwiQ0tTXCIsXCJDSUNEXCJdIiwicGhvbmUiOiIxMzk2OTY0MzA3NSIsInByb2plY3QiOiJnYW9zcyIsImdyb3VwcyI6WyIvZ3JvdXAtZ2Fvc3MiXSwicHJlZmVycmVkX3VzZXJuYW1lIjoiZ2Fvc3MiLCJlbWFpbCI6Imdhb3NzQGluc3B1ci5jb20ifQ.jXXeExFEGk4QFpDRzp_WKgXtaMjRtNpCroOez3dMw7yMfRTDO29x8lG9OLVQEW2tOo8O7FdkYvmqwg5zqwbQu00DwR_br1YNsdRlyHaObP4BIu7Mr5Ab0iw9OGFNRpfSNZUSP5qzv7Ly-ZFtAhNLkwCbDTgrItexL0NnuJAglcjXeOc2QTqHVo4rqe8oYEylaj67B33pyOTSaF67ITUaLh_buW7tRIyC9oGZVTl8s8PFUnDkEAO6pLjQIvrjUMerv3Wi8Ff7PGpn0CISIOWrc1rvL67yCDB_1jQKwffwPCIzV_sLo_WTZbPFZ6-MQ5JGiLymLaEBpXdTig7UeqULjg",
	"orderRoute": "ECS",
	"consoleCustomization": {},
	"userId": "f679eb23-f74d-477a-a31f-a656d71604ed",
	"setCount": "1",
	"billType": "hourlySettlement",
	"orderType": "changeConfigure ",
	"duration": "1",
	"durationUnit": "H",
	"productList": [{
		"region": "cn-north-3",
		"availableZone": "",
		"productLineCode": "ECS",
		"productTypeCode": "ECS_std",
		"instanceCount": "1",
		"instanceId": "f750c817-4ab8-49d0-ad33-398cf605a6db",
		"itemList": [{
			"code": "std_2C4G",
			"value": "S1.medium.2",
			"name": "S1.medium.2"
		}, {
			"code": "CPU",
			"value": "2",
			"name": "CPU"
		}, {
			"code": "memory",
			"value": "4",
			"name": "内存"
		}, {
			"code": "image_Windows",
			"value": "da099db5-e180-49c0-a74d-54bbc18fafd4",
			"name": "Windows"
		}, {
			"code": "sys_EBS_SATA",
			"value": "60",
			"name": "云盘系统盘标准型"
		}, {
			"code": "sys_EBS_SSD",
			"value": "0",
			"name": "云盘系统盘SSD型"
		}]
	}]
}
```

> 返回参数

请参考[公共返回值](#公共返回值)

> 错误码

请参考[公共错误码-云服务操作常见错误码](#公共错误码)



