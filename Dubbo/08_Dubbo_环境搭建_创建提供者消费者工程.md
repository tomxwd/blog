---
title: 08_Dubbo_环境搭建_创建提供者消费者工程
date: 2019-09-09 21:38:52
tags: 
 - Dubbo
categories:
 - Java
 - Dubbo
---

# 08_Dubbo\_环境搭建_创建提供者消费者工程

![1568036474886](08_Dubbo_环境搭建_创建提供者消费者工程/1568036474886.png)

创建三个项目，正常来说本来是两个，一个订单服务，一个用户服务，但是由于分模块开发，所以抽取出公共的interface模块。



## gmall-interface模块

bean.UserAddress：

```java
import java.io.Serializable;
/**
 * 用户地址
 */
public class UserAddress implements Serializable {

    private Integer id;
    private String userAddress; //用户地址
    private String userId; //用户id
    private String consignee; //收货人
    private String phoneNum; //电话号码
    private String isDefault; //是否为默认地址    Y-是     N-否

    public UserAddress() {
        super();
        // TODO Auto-generated constructor stub
    }

    public UserAddress(Integer id, String userAddress, String userId, String consignee, String phoneNum,
                       String isDefault) {
        super();
        this.id = id;
        this.userAddress = userAddress;
        this.userId = userId;
        this.consignee = consignee;
        this.phoneNum = phoneNum;
        this.isDefault = isDefault;
    }

    public Integer getId() {
        return id;
    }
    public void setId(Integer id) {
        this.id = id;
    }
    public String getUserAddress() {
        return userAddress;
    }
    public void setUserAddress(String userAddress) {
        this.userAddress = userAddress;
    }
    public String getUserId() {
        return userId;
    }
    public void setUserId(String userId) {
        this.userId = userId;
    }
    public String getConsignee() {
        return consignee;
    }
    public void setConsignee(String consignee) {
        this.consignee = consignee;
    }
    public String getPhoneNum() {
        return phoneNum;
    }
    public void setPhoneNum(String phoneNum) {
        this.phoneNum = phoneNum;
    }
    public String getIsDefault() {
        return isDefault;
    }
    public void setIsDefault(String isDefault) {
        this.isDefault = isDefault;
    }
    
    @Override
    public String toString() {
        return "UserAddress{" +
                "id=" + id +
                ", userAddress='" + userAddress + '\'' +
                ", userId='" + userId + '\'' +
                ", consignee='" + consignee + '\'' +
                ", phoneNum='" + phoneNum + '\'' +
                ", isDefault='" + isDefault + '\'' +
                '}';
    }
}
```

service.OrderService：

```java
public interface OrderService {

    /**
     * 初始化订单
     * @param userId
     */
    void initOrder(String userId);

}
```

service.UserService:

```java
public interface UserService {

    /**
     * 按照用户id返回所有的收货地址
     * @param userId
     * @return
     */
    List<UserAddress> getUserAddressList(String userId);

}
```



## order-service-consumer

service.impl.OrderServiceImpl:

```java
public class OrderServiceImpl implements OrderService {

    UserService userService;

    public void initOrder(String userId) {
        // 1、查询用户的收货地址
        List<UserAddress> addressList = userService.getUserAddressList(userId);
        System.out.println("addressList = " + addressList);
    }

}
```



## user-service-provider

service.impl.UserServiceImpl:

```java
public class UserServiceImpl implements UserService {

    public List<UserAddress> getUserAddressList(String userId) {
        System.out.println("UserServiceImpl.....old...");
        // TODO Auto-generated method stub
        UserAddress address1 = new UserAddress(1, "北京市昌平区宏福科技园综合楼3层", "1", "李老师", "010-56253825", "Y");
        UserAddress address2 = new UserAddress(2, "深圳市宝安区西部硅谷大厦B座3层（深圳分校）", "1", "王老师", "010-56253825", "N");
		/*try {
			Thread.sleep(4000);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}*/
        return Arrays.asList(address1,address2);
    }

}
```





