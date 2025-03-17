SDK升级PS管理端实现方案
1. 涉及功能点
1.1 提供开发者平台增、删、改、查接口
1.1.1 应用自升级模块
开发者平台，创建自升级应用
应用列表
搜索功能
应用修改功能
应用查看功能
应用启/停用功能
新增页支撑接口

根据包名查询应用版本接口（增加cpUser过滤）
查询国家接口：https://test-manager.palmplaystore.com/PalmplayManager/configure/selectConfiguration?typeName=CLIENT_COUNTRY
查询品牌接口：https://test-manager.palmplaystore.com/PalmplayManager/configure/selectBrandType
查询机型接口：https://hub.shalltry.com/infoHub/api/model?projectName=PalmStore&groupName=palms_model
客户端版本接口：https://test-manager.palmplaystore.com/PalmplayManager/api/productControl/clientVersions
安卓版本：前端写死的4-15安卓版本
https://helpdesk.feishu.cn/saipan/v2/api/image/img_v3_02kf_7a1953ff-4dd4-4486-9c20-dc22d9c3678g?x-oss-process=image/resize,m_fixed,m_lfit,w_300https://helpdesk.feishu.cn/saipan/v2/api/image/img_v3_02kf_fe3d9a18-e349-43b0-ade2-7a21ef2bd91g?x-oss-process=image/resize,m_fixed,m_lfit,w_300

1.1.2 数据报表功能
开发者透视表：底表：点击图片可查看完整电子表格
1.2 PS管理端功能点
1.2.1 升级任务列表
搜索功能
查看功能
https://helpdesk.feishu.cn/saipan/v2/api/image/img_v3_02kf_bf57edab-9c59-47de-b27a-b0e5ee4e780g?x-oss-process=image/resize,m_fixed,m_lfit,w_300

1.2.2 升级应用管理
新增应用管理配置
查看应用管理配置
停用应用管理配置
启用复制应用管理配置
修改应用管理配置
https://helpdesk.feishu.cn/saipan/v2/api/image/img_v3_02kf_954b0db9-05c6-48c3-850c-996cce50c55g?x-oss-process=image/resize,m_fixed,m_lfit,w_300https://helpdesk.feishu.cn/saipan/v2/api/image/img_v3_02kf_80158b89-4273-4c1e-bbfb-23a9a628c12g?x-oss-process=image/resize,m_fixed,m_lfit,w_300

1.3 提供给客户端接口
1.3.1 拉取配置接口
2. 功能点技术实现方案
2.1 提供开发者平台增、删、改、查接口
2.1.1 应用自升级模块
2.1.1.1 开发者平台和PS管理端交互时序图
https://helpdesk.feishu.cn/saipan/v2/api/image/img_v3_02kf_34e2250d-9c4f-4707-9677-144be204f2bg?x-oss-process=image/resize,m_fixed,m_lfit,w_300

2.1.1.2 PS管理端提供的接口
域名：测试：test-manager.palmplaystore.com，生产域名：manager.palmplaystore.com
开发者平台页面依赖接口
开发者平台自升级应用允许多个其实区间
2.1.2 数据报表
2.2 自升级SDK PS管理端
域名：测试：test-manager.palmplaystore.com，生产域名：manager.palmplaystore.com
2.2.1 升级任务列表（开发者平台创建的数据）
2.2.2 升级应用管理
2.3 PS与客户端交互
2.3.1 客户端和PS交互流程图
2.3.1.1 最终选择的技术方案
方案：服务端配置缓存至CDN，服务端配置增加配置时PUSH到CDN。

客户端先去CDN拉取配置，然后在客户端本地进行下发判断（只是初次过滤，有些过滤条件客户端控制不了，如：灰度量级、设备激活时间）；
满足下发条件时，服务端再次进行下发条件判断，满足条件即下发
方案流程步骤

开发者app启动，传参至客户端自升级SDK
SDK判断本地是否安装PS、否：则结束自升级，是：则进入步骤3
本地是否存在升级SDK配置，否：则进入步骤5，是：则进入步骤4
判断APP是否已是最新版本（客户端从遍历sdk配置，判断是否已是目标版本），否：则进入步骤4.1
4.1：判断弹窗次数是否达到上限（客户端本地需要记录task对应的弹窗次数，格式如：taskId：3。目的：防止弹窗超限后还给用户弹窗，这个弹窗记录是升级任务级别的）。否：则进入步骤4.2，是：则将本地SDK配置置为失效（目的：客户端可以及时的获取新的升级任务），并结束升级。
4.2：弹窗提示用户升级，用户选择是否升级。否：则客户端记录Task级别弹窗次数；是：则进入步骤4.3
4.3：客户端跳转详情页，根据网络偏好 + SDK升级策略进行APP升级
客户端SDK去CDN拉取包维度的SDK升级配置（这个配置文件，会包含这个包维度的所有已启用的升级任务，JSON数组形式）
客户端根据CDN配置，进行初次过滤（过滤条件包括：国家、品牌、机型、版本、生效时间等），判断是否满足升级条件，否：则结束，是：则进入步骤7
客户端SDK发起服务端请求，将常见请求参数及升级接口定义的参数，传到服务端。服务端根据客户端请求参数返回下发配置
不满足升级条件，则：返回
满足升级条件，则：返回
https://helpdesk.feishu.cn/saipan/v2/api/image/img_v3_02kf_12a26e61-b772-4600-8a18-99d743e1fb1g?x-oss-process=image/resize,m_fixed,m_lfit,w_300

2.3.1.2 CDN更新及客户端拉取逻辑
CDN域名：

测试：https://test-cdn2.palmplaystore.com
生产：https://cdn2.palmplaystore.com
CDN上面的配置，以json文件形式存储。（路径定义：https://cdn2.palmplaystore.com/appUpgrade/包名.json ，示例：https://cdn2.palmplaystore.com/appUpgrade/org.telegram.messenger.web.json）

服务端增改CDN逻辑https://helpdesk.feishu.cn/saipan/v2/api/image/img_v3_02kf_03b01139-7308-4da7-8c31-246af6cdff8g?x-oss-process=image/resize,m_fixed,m_lfit,w_300

客户端拉取CDN配置逻辑https://helpdesk.feishu.cn/saipan/v2/api/image/img_v3_02kf_cd9c17a7-82cd-4971-9620-213e3441c23g?x-oss-process=image/resize,m_fixed,m_lfit,w_300

2.3.1.3 所有待选技术方案
如何保证客户端及时感知服务端最新SDK升级任务，同时避免客户端频繁请求服务端

方案一：应用配置增加客户端SDK配置刷新时间，客户端在配置失效前不会去服务端拉取配置，失效后会请求服务端拉取最新配置https://helpdesk.feishu.cn/saipan/v2/api/image/img_v3_02kf_e2f5153a-6972-4744-80e8-9144cd6af51g?x-oss-process=image/resize,m_fixed,m_lfit,w_300

方案二：服务端将配置PUSH到CDN，客户端每次启动都会去CDN获取配置。由服务端定时往CDN PUSH配置数据（防止CDN失效，避免回源） or CDN失效时，客户端回源服务端获取配置。（客户端更倾向此方案）

约定一下哪些过滤是在SDK的本地做？

国家过滤
安卓版本过滤
版本号过滤
机型过滤
品牌过滤
约定一下哪些过滤是在服务端做？

设备激活时间判断（客户端说取不到，必须放在服务端）
灰度量级（客户端是单个节点，没有办法控制到每天的灰度总量，所以只能放在服务端做）
大体流程

客户端开启定时任务，定时获取CDN上存储的多你好！我已经接收到了"SDK升级PS管理端实现方案.docx"文档。我会将它转换为Markdown格式。你好！我已经将文档转换为Markdown格式，下面是转换后的内容：
SDK升级PS管理端实现方案
1. 涉及功能点
1.1 提供开发者平台增、删、改、查接口
1.1.1 应用自升级模块
开发者平台，创建自升级应用
应用列表
搜索功能
应用修改功能
应用查看功能
应用启/停用功能
新增页支撑接口

根据包名查询应用版本接口（增加cpUser过滤）
查询国家接口：https://test-manager.palmplaystore.com/PalmplayManager/configure/selectConfiguration?typeName=CLIENT_COUNTRY
查询品牌接口：https://test-manager.palmplaystore.com/PalmplayManager/configure/selectBrandType
查询机型接口：https://hub.shalltry.com/infoHub/api/model?projectName=PalmStore&groupName=palms_model
客户端版本接口：https://test-manager.palmplaystore.com/PalmplayManager/api/productControl/clientVersions
安卓版本：前端写死的4-15安卓版本
https://helpdesk.feishu.cn/saipan/v2/api/image/img_v3_02kf_7a1953ff-4dd4-4486-9c20-dc22d9c3678g?x-oss-process=image/resize,m_fixed,m_lfit,w_300https://helpdesk.feishu.cn/saipan/v2/api/image/img_v3_02kf_fe3d9a18-e349-43b0-ade2-7a21ef2bd91g?x-oss-process=image/resize,m_fixed,m_lfit,w_300

1.1.2 数据报表功能
开发者透视表：底表：点击图片可查看完整电子表格
1.2 PS管理端功能点
1.2.1 升级任务列表
搜索功能
查看功能
https://helpdesk.feishu.cn/saipan/v2/api/image/img_v3_02kf_bf57edab-9c59-47de-b27a-b0e5ee4e780g?x-oss-process=image/resize,m_fixed,m_lfit,w_300

1.2.2 升级应用管理
新增应用管理配置
查看应用管理配置
停用应用管理配置
启用复制应用管理配置
修改应用管理配置
https://helpdesk.feishu.cn/saipan/v2/api/image/img_v3_02kf_954b0db9-05c6-48c3-850c-996cce50c55g?x-oss-process=image/resize,m_fixed,m_lfit,w_300https://helpdesk.feishu.cn/saipan/v2/api/image/img_v3_02kf_80158b89-4273-4c1e-bbfb-23a9a628c12g?x-oss-process=image/resize,m_fixed,m_lfit,w_300

1.3 提供给客户端接口
1.3.1 拉取配置接口
2. 功能点技术实现方案
2.1 提供开发者平台增、删、改、查接口
2.1.1 应用自升级模块
2.1.1.1 开发者平台和PS管理端交互时序图
https://helpdesk.feishu.cn/saipan/v2/api/image/img_v3_02kf_34e2250d-9c4f-4707-9677-144be204f2bg?x-oss-process=image/resize,m_fixed,m_lfit,w_300

2.1.1.2 PS管理端提供的接口
域名：测试：test-manager.palmplaystore.com，生产域名：manager.palmplaystore.com
开发者平台页面依赖接口
开发者平台自升级应用允许多个其实区间
2.1.2 数据报表
2.2 自升级SDK PS管理端
域名：测试：test-manager.palmplaystore.com，生产域名：manager.palmplaystore.com
2.2.1 升级任务列表（开发者平台创建的数据）
2.2.2 升级应用管理
2.3 PS与客户端交互
2.3.1 客户端和PS交互流程图
2.3.1.1 最终选择的技术方案
方案：服务端配置缓存至CDN，服务端配置增加配置时PUSH到CDN。

客户端先去CDN拉取配置，然后在客户端本地进行下发判断（只是初次过滤，有些过滤条件客户端控制不了，如：灰度量级、设备激活时间）；
满足下发条件时，服务端再次进行下发条件判断，满足条件即下发
方案流程步骤

开发者app启动，传参至客户端自升级SDK
SDK判断本地是否安装PS、否：则结束自升级，是：则进入步骤3
本地是否存在升级SDK配置，否：则进入步骤5，是：则进入步骤4
判断APP是否已是最新版本（客户端从遍历sdk配置，判断是否已是目标版本），否：则进入步骤4.1
4.1：判断弹窗次数是否达到上限（客户端本地需要记录task对应的弹窗次数，格式如：taskId：3。目的：防止弹窗超限后还给用户弹窗，这个弹窗记录是升级任务级别的）。否：则进入步骤4.2，是：则将本地SDK配置置为失效（目的：客户端可以及时的获取新的升级任务），并结束升级。
4.2：弹窗提示用户升级，用户选择是否升级。否：则客户端记录Task级别弹窗次数；是：则进入步骤4.3
4.3：客户端跳转详情页，根据网络偏好 + SDK升级策略进行APP升级
客户端SDK去CDN拉取包维度的SDK升级配置（这个配置文件，会包含这个包维度的所有已启用的升级任务，JSON数组形式）
客户端根据CDN配置，进行初次过滤（过滤条件包括：国家、品牌、机型、版本、生效时间等），判断是否满足升级条件，否：则结束，是：则进入步骤7
客户端SDK发起服务端请求，将常见请求参数及升级接口定义的参数，传到服务端。服务端根据客户端请求参数返回下发配置
不满足升级条件，则：返回
满足升级条件，则：返回
https://helpdesk.feishu.cn/saipan/v2/api/image/img_v3_02kf_12a26e61-b772-4600-8a18-99d743e1fb1g?x-oss-process=image/resize,m_fixed,m_lfit,w_300

2.3.1.2 CDN更新及客户端拉取逻辑
CDN域名：

测试：https://test-cdn2.palmplaystore.com
生产：https://cdn2.palmplaystore.com
CDN上面的配置，以json文件形式存储。（路径定义：https://cdn2.palmplaystore.com/appUpgrade/包名.json ，示例：https://cdn2.palmplaystore.com/appUpgrade/org.telegram.messenger.web.json）

服务端增改CDN逻辑https://helpdesk.feishu.cn/saipan/v2/api/image/img_v3_02kf_03b01139-7308-4da7-8c31-246af6cdff8g?x-oss-process=image/resize,m_fixed,m_lfit,w_300

客户端拉取CDN配置逻辑https://helpdesk.feishu.cn/saipan/v2/api/image/img_v3_02kf_cd9c17a7-82cd-4971-9620-213e3441c23g?x-oss-process=image/resize,m_fixed,m_lfit,w_300

2.3.1.3 所有待选技术方案
如何保证客户端及时感知服务端最新SDK升级任务，同时避免客户端频繁请求服务端

方案一：应用配置增加客户端SDK配置刷新时间，客户端在配置失效前不会去服务端拉取配置，失效后会请求服务端拉取最新配置https://helpdesk.feishu.cn/saipan/v2/api/image/img_v3_02kf_e2f5153a-6972-4744-80e8-9144cd6af51g?x-oss-process=image/resize,m_fixed,m_lfit,w_300

方案二：服务端将配置PUSH到CDN，客户端每次启动都会去CDN获取配置。由服务端定时往CDN PUSH配置数据（防止CDN失效，避免回源） or CDN失效时，客户端回源服务端获取配置。（客户端更倾向此方案）

约定一下哪些过滤是在SDK的本地做？

国家过滤
安卓版本过滤
版本号过滤
机型过滤
品牌过滤
约定一下哪些过滤是在服务端做？

设备激活时间判断（客户端说取不到，必须放在服务端）
灰度量级（客户端是单个节点，没有办法控制到每天的灰度总量，所以只能放在服务端做）
大体流程

客户端开启定时任务，定时获取CDN上存储的多个下发配置，定时时间间隔这个配置根据上一次请求，根据CDN上获取的下发配置为准，该配置最长不超过15天，超过15天的配置，客户端认为就是15天
客户端根据多个下发配置进行本地过滤
如果在客户端本地过滤后，判断符合条件，那么就再请求服务端判断是否要升级
当服务端接受到升级请求后，服务端需要实现同样的过滤逻辑，然后根据激活时间和灰度量级，来判断升级
补充：CDN作为第一层过滤；服务器访问作为第二层过滤，下发多语言

如何控制流量成本 目前需要接入SDK升级的应用体量比较大，比如MovieBox千万级别，如果完成应该升级：可能需要很大的流量成本。方案之间可以相互结合

方案一：选择定向用户进行SDK升级，非定向用户（如：用户价值不高等）不做SDK升级。定向用户可以在大麦进行圈选

方案二：指定某个版本的灰度总量，防止成本无法控制。（附言：现有交互中有每天灰度量级，类比一个滑动窗口，虽然控制了每天的成本，但缺乏总的窗口大小。很容易出现成本无法控制问题）

方案三：服务端增加丢弃策略，按比例丢弃服务端请求，不做请求处理。（附言：大型电商公司大促时，都会有丢弃算法，不可能处理客户端来的每个请求，那样服务端扛不住）【衡磊砢觉得这条是业界方法】

方案四：服务端配置缓存至CDN，服务端配置增加配置时PUSH到CDN。1）客户端先去CDN拉取配置，然后在客户端本地进行下发判断；2）满足下发条件时，服务端再次进行下发条件判断，满足条件即下发【本次首推的核心方案】

如何解决性能压力

方案一：服务端增加丢弃策略，同上。

方案二：服务端配置缓存至CDN，减少客户端请求，同上。

方案三：客户端离散请求，按照离散算法（离散算法可以由服务端PUSH到CDN，客户端获取获取CDN配置，按照离散算法进行离散请求）【如果使用CDN缓存，客户端做边缘计算的方式，这一条可以不用做了】

方案四：其他常规技术方案：限流熔断、本地缓存等

如何让客户端及时获取最新的升级任务配置（exg. 用户APP升级前，对应的升级任务已经过期或者停用，那么客户端SDK需要实时感知配置的变动）

方案一：客户端每次启动时，都请求CDN，客户端拉取CDN上配置的下发策略，在SDK进行逻辑判断 注：如沟通，使用CDN刷新时间间隔的选项，比如1天请求一次cdn还是3天请求一次cdn

方案二：客户端每次启动时，请求服务端进行下发逻辑判断【不推荐，服务器压力太大了】

最后思考点：是否可以依赖服务端布隆过滤（or 布谷鸟），过滤客户端请求。以应用自升级任务（国家、品牌、机型、旧版本区间等）作为一个Key，过滤器判断是否存在。按照产品的诉求：同一个包 + 版本拉过配置，就无需再次拉取配置。当应用自升级请求命中过滤器，则直接返回。使用此方案，则应用SDK升级配置无需增加失效时间，完全由服务端控制。

2.3.2 接口定义
域名：测试：test-upgrade-api.palmplaystore.com，生产：app-manage-api.shalltry
【来源于 传音智库 发现更多精彩 点击进入体验】
