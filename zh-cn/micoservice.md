##微服务


###MailHub

客户端通过网关的透明代理，与之交互完成。它为客户端提供了如下服务:
1. 消息转发代理（发送IM消息）；
2. 同步接收消息队列消息（接收IM消息）；

```csharp


/*
 * 由IM服务器实现
 */
contract UserResolverListener {
    //由业务系统通知，群组配置变更
    void GroupChanged(string groupID);
    //由业务系统通知，群解散
    void GroupRemoved(string groupID);
    //由业务系统通知，转发规则变化
    void BlockRuleChanged(string userID);
}

table Sync struct {
    uint32      Offset; //同步起始SQID
    uint32      Counter; //同步数量
}

contract MailHub{
    /**
     * 获取发送消息SQID
     */
    uint32 PutSync();
    /**
     * 发送消息
     * 如果目标不存在或者，SQID不正确将抛出异常
     */
    uint64 Put(Mail mail) throws(UserNotFound,UnexpectSQID);

    /**
     *  发起一个新的Sync事务，服务器将返回协商后的最终同步信息{@see Sync}
     *  @offset 同步起始位置
     *  @count  同步条数
     *  如果用户不存在或者，区域不正确或者已经有一个Sync事务正在运行该方法将抛出异常
     */
    uint32 Sync(uint32 offset,uint32 count) throws(UserNotFound,ResourceNotFound,ResourceBusy);

    /**
     * 关闭当前Sync事务
     */
    void Fin(uint32 offset);

    /**
     * 通过消息SQID，拉取一批消息。
     */
    Mail[] Fetch(uint32[] mailIDs) throws(ResourceNotFound);
}
```

###网关

网关提供两类服务：

1. Auth 认证代理

```csharp
contract Auth{
    /*
     * 通过用户名登陆，properties为登陆附加属性列表：例如密码或者令牌信息等
     * @return 返回登陆成功token
     */
    KV[] Login(string username,KV[] properties)  throws(UserNotFound,UserAuthFailed,ResourceNotFound);

    /*
     * 通过token下线
     */
    void Logoff(KV[] properties);
}
```
2. Push 推送代理

```csharp
contract Push {
    /**
     * 注册推送token
     */
    void Register(byte[] pushToken);
    /**
     * 反注册当前推送token
     */
    void Unregister();
}

```

###客户端

客户端也可以被看做一个微服务供网关/MailHub等调用
```csharp
contract Client{
    /**
     * 异步消息推送接口，与MailHub#Sync/MailHub#Fin接口配合使用
     */
    @gslang.Async
    void Push(Mail mail);

    /**
     * 异步消息通知，MailHub会定时调用该接口通知客户端最新的接收消息SQID
     */
    @gslang.Async
    void Notify(uint32 SQID);

    /**
     * 异步设备上下线通知，当同以用户其它设备上下线时调用该接口
     */
     @gslang.Async
    void DeviceStateChanged(Device device,bool online);
}
```

###DHKeyResolver

设备与网关加密通道参数信息查询服务，网关与客户端之间通过DH算法交换密钥，并加密保护；
```csharp
table DHKey {
    string P;
    string G;
}

contract DHKeyResolver {
    // 通过设备ID，获取DH参数
    DHKey DHKeyResolve(Device device) throws(ResourceNotFound);
}
```

###UserResolver

MailHub通过调用该接口获得用户/群组配置信息，以及转发规则

```csharp

//由业务系统实现
contract UserResolver {
    // 获取群组通知
    UserGroup QueryGroup(string groupID) throws(ResourceNotFound);
    // 获取用户的转发规则
    BlockRules QueryBlockRules(string userID);
}


```

###PushServiceProvider

离线消息推送服务，整合iOS/小米等厂商推送通道：
```csharp
contract PushServiceProvider {
    // 设备上下线通知，由网关调用
    void DeviceStatusChanged(Device device,bool online);
    // 设备上下线通知，由登陆服务器/网关调用
    void UserStatusChanged(string userID,Device device, bool online);
    // 设备渠道推送token注册，由网关调用
    void DeviceRegister(Device device,byte[] token);
    // 设备渠道推送token反注册，由网关调用
    void DeviceUnregister(Device device);
}
```
