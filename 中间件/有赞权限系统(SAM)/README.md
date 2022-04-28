## SAM权限系统模型设计

依据RBAC思想，SAM权限系统业务模型设计为员工管理和权限管理两部分，员工管理主要指管理员工以及为员工指派角色，权限管理主要指管理菜单、页面、按钮、API等资源，通过定义最基本的业务功能点作为权限点，实现管理角色对资源主体的请求，构成“用户-角色-权限-资源”的授权模型。

![sam](https://img.yzcdn.cn/public_files/2018/03/02/d02bd76ecd033c3d5c2838d152514fce.jpeg)

下面是SAM权限系统模型中的一些通用语言：

- 员工：角色的载体，权限的实行者；
- 角色：角色是权限集的进一步映射。业务系统可以动态管理角色，各业务为方便用户使用可提供默认角色列表，满足不同的员工权限；
- 权限点：全局唯一用来表示某一功能点对应的权限的状态；
- 功能点：逻辑上定义的用来描述系统资源的最小单位，单个功能点都对应唯一一个权限点；
- 功能集（权限集）：即功能点的集合，有一组功能点按照特定格式进行组合；
- API：请求系统资源的通道和动作，拥有功能集属性；
- 菜单：将系统资源组织后展示给请求者的入口，拥有功能集属性；
- 页面：被当做一种特殊的菜单，拥有URL属性；
- 按钮：页面中更细粒度的入口，被当作一种特殊的菜单

## SAM权限系统模型的实现

在传统的RBAC模型中，通常通过一张关系表来保存角色与权限集的对应关系，实现权限与角色相关联。可以预见的是，随着零售业务的不断发展会积累下不计其数的功能点，导致关联表的数据极难维护和使用。

SAM权限系统利用进制转换的策略解决了这个问题 ，同时提高了存储效率以及权限判定效率。

- 一个基本类型为Long的十进制数字，它也可以看做是由64位0或1组成的二进制
- 在SAM系统模型设计中，每个功能点定义为一个权限点，该权限点由idx和pos两个属性确保是全局唯一的权限点；
- idx表示第几个Long型空间，pos表示Long型对应的二进制数中所在的位置，64位长度即可代表64个不同功能点；
- 当64位无法再容放更多功能点，这个idx属性就会自增，重新申请一个Long型
- 如此一个64位的Long数字，通过0或1组合，即可表示最多64个不同的功能点所拥有权限的状态描述

例如，权限集{1}表示拥有idx=0, pos=0对应功能点的权限，权限集{-1, 1}表示拥有idx=0, pos∈{0,1,2...63}与idx=1,pos=0对应功能的权限。

SAM权限系统将资源与所代表的功能点的关联关系通过进制的方式管理起来，采用计算机进制的思想，抽象出功能集换算公式来完成资源与二进制之间的映射，以及角色与二进制的映射。权限集换算公式如下：
$$
{(idx0,pos0),(idx0,pos1)…(idxN,posM)} => {Long0,Long1…LongN}
$$
SAM权限系统同样通过进制思想实现“Who对What进行了How的操作”，角色请求某个资源（菜单/API）时，通过权限校验计算公式——进制按位“与”运算操作的思想（见下）得出该角色是否拥有访问资源的权限。采用进制来实现运算，权限判定的效率会变得更加的高效。例如，一个仓管在点击一个商品库存菜单时，背后的权限校验计算公式，其实是将角色的权限集与资源的权限集进行按位与计算，任意一对序号为idx的Long算得不为0，即两集合有公共的功能集，认为该角色拥有对资源访问的权限。权限校验计算公式
$$
(Long0,Long1…LongN) \&(Long0,Long1…LongM)
$$
SAM权限系统模型的实现遵循RBAC模型的以下原则：

- 最小权限原则。通过最小权限原则可以将角色配置成其完成任务所需要的最小的功能集
- 责任分离原则。有了责任分离原则可以通过调用相互独立互斥的角色来共同完成敏感的任务而体现，比如要求一个仓管和商品管理员共同参与一个商品
- 数据抽象原则。数据抽象则可以通过权限的抽象来体现，如仓管操作商品发货，库存管理等抽象权限，而不用操作系统提供的典型的读、写、执行权限。

### 菜单渲染

SAM通过客户端的方式进行接入，菜单渲染在客户端一侧进行。目前SAM已经提供了php/node js两套客户端，供web层进行接入和渲染。

菜单渲染的过程可以分为三点：

- 结点定位。按照系统功能的划分，菜单通常以一棵树的形式进行展现。以零售PC后台为例，所有在页面中展示的元素，都认为是一种菜单，这样的菜单元素包括：菜单、页面、按钮。在后台访问时，用户停留的菜单通常是页面，页面有一个全局唯一的属性：URL，往上：可以通过父菜单找到根结点，往下，页面下可能包含一些子菜单——按钮。因此SAM只需要根据当前请求的URL，即可在后台菜单树中定位到唯一的页面菜单，同时获得该菜单的结点路径以及拥有的按钮。
- 权限计算。我们已经获得了用户的用户权限和完整的菜单树，根据每个菜单结点的权限集，可以计算出当前用户对结点的访问权。根据计算结果，客户端对菜单可以进行区分渲染，比如：用户通过拼URL访问一个无权限页面时会提示非法，无权限访问的菜单和按钮会自动置灰不可点击。
- 属性传递。默认菜单不具备URL属性。菜单的URL属性通过子菜单的URL传递生成，SAM会选择第一个有权限的子菜单的URL作为父结点的属性，并逐级传递到一级菜单。

### API权限校验

零售系统中除了菜单外，API是另一种被请求的资源类型。API校验是除了菜单渲染外另一道权限控制的保障。通过API网关的API请求转发到具体业务系统时，嵌入在业务系统的SAM API校验客户端会首先通过上面的权限校验公式对该角色是否具有权限访问这个API作判定，若权限校验则执行后面业务逻辑，具体流程如下图所示：

![sam](https://img.yzcdn.cn/public_files/2018/03/02/99443ea1c95caee396ac2473bf35f7e4.jpeg)

API权限校验的伪代码实现：

```python 
#权限不通过错误码提示信息
AUTHPERM_ERROR(231000401,"您没有权限执行该操作!")  
# 织入点
@Before("@annotation(com.youzan.sam.common.Auth)")
# 切面处理方法
def handle(JoinPoint pjp):  
    # 可以启动时或者运行时控制该开关是否对API进行权限校验
    if(!enable):
         return
    # 权限校验结果包装对象
    def pass=checkPermission()
    # 权限校验执行成功
    if (pass.isSuccess()):
       # 权限校验通过
       if(pass.getData().get("isSuccess")):
         return
       # 权限校验不通过
       else:
         throw new BusinessException(AUTHPERM_ERROR.getCode,AUTHPERM_ERROR.getMessage());
    # 权限校验执行失败
    else:
       throw BusinessException(pass.getCode(), pass.getMessage())

# 权限校验方法
def checkPermission():  
    # 判断是否需要走权限校验，对于某些内部调用可以直接跳过
    {...}
    # 获取卡门（API网关）隐式参数,运用了dubbo的隐式传参的能力
    def kdt_id=RpcContext.getContext().getAttachment(Constants.KDT_ID_KEY)
    def admin_id=RpcContext.getContext().getAttachment(Constants.ADMIN_ID_KEY)
    def service = RpcContext.getContext().getAttachment(Constants.SERVICE_KEY)
    def method = RpcContext.getContext().getAttachment(Constants.METHOD_KEY)
    def version = RpcContext.getContext().getAttachment(Constants.VERSION_KEY)
    # 上述参数的校验
    {...}
    # 通过StaffPermServiceProxy获取角色的权限集
    def staffPerm=StaffPermServiceProxy.getStaffPerms(adminId, kdtId)
    # 通过APIPermServiceProxy获取API的权限集
    def apiPerm=APIPermServiceProxy.getServicePerms(service, version, method)
    # 运用权限校验计算公式判定该角色是否可以访问此API
    {...}
    # 返回结果
    return pass
```

API权限校验流程可以总结如下：

1. 业务方在对应需要校验权限的API上标注@Auth注解，Spring框架会在初始化创建业务bean的时候，扫描该bean是否具有@Auth注解标注的方法，对于有@Auth注解标注的，会创建代理类，然后会将该权限切面织入到代理类中；
2. 业务调用有@Auth注解标注的方法时，会执行该权限校验切面逻辑，首先检查权限校验开关，判断是否需要权限校验，该开关可以在运行时动态设置；
3. 如果需要，再调用AuthService的权限校验方法，AuthService会根据店铺id与用户id从SAM服务端获取员工角色权限信息，根据卡门（API网关）隐式参数中service，method，version去SAM服务端获取对应API权限(相对于在对应API上直接标注权限点，这种方式更加的灵活，而且可以随着业务API版本的升级，进行很方便的升级，同时结合API网关可以对API进行分流，不同的商家可以对应不同API的权限校验);
4. 在获取到角色权限集和API权限集后，基于上面的角色与权限校验逻辑进行权限校验。校验通过，则正式发起API请求。校验不过，则提示无权限。