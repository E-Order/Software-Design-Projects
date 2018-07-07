
# 软件需求规格说明书(SRS)


### 项目背景  
移动互联网发展迅速，越来越多的人乐于接受移动互联网给我们带来的便利，但是很多餐饮行业仍然采用传统人工点餐方法，既麻烦，又费时。通过调研，我们了解到许多参与者表明“自身有扫码点餐的意向，但是商家没有提供”，使得**扫码点餐**无法被人们广泛应用，因此我们希望可以解决这个困境的部分。

### 业务分析 
1 涉众人员分析（人）

点餐客户：希望能够快速自助点单，无需等待服务员。希望可以清楚地查看本店具体的菜品、及其库存和价格。希望能够清晰地看到本店的优惠信息。希望可以灵活的修改订单，添加菜品。希望可以通过点餐系统完成账单支付。

餐厅：希望能够便捷地指引未接触的客户进行扫码点餐。希望准确地记录用户点的菜，满足客户的要求。希望能够自动、快速地更新点菜信息。希望能够便捷地更新菜品信息。希望能够准确地更新菜品库存信息。

2 业务流程分析（事）

客户：

**1.** 点餐客户通过扫码成功进入主菜单，开始点餐。

**2.** 点餐客户通过上下拉浏览菜单，查看菜品。

**3.** 点餐客户选择各自想要的菜式，加入购物车。

点餐客户重复2-3步，直到选择完毕。

**4.** 点餐客户下单进入订单详情。

**5.** 点餐客户确定提交订单

**6.** 餐厅接收订单消息，开始准备菜品

### 限制条件（规）
1 运行环境约束

微信 5.3版本以上

2 质量要求

用户能简便，流畅点餐
餐厅能实时正确接收订单消息

### 产品分析

**功能性需求：**

两个系统： 用户点餐系统、商家管理系统


 ### 1 用例图
 
![image.png](https://upload-images.jianshu.io/upload_images/13021922-785afe53979283bb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 2 用例描述

### 扫码点餐Brief用例

#### 1. Use case：点餐

   Actors：客户
   
   Type：Primary
   
   Description：顾客扫码进入小程序界面，查看菜单及其详情，选择菜品及其规格，确认订单，确认后订单提交到商家。
   
#### 2. Use case：管理购物车

   Actors：客户
   
   Type：Primary
   
   Description：顾客查看自己的购物车，并增加或删除菜品。
   
#### 3. Use case：管理订单

   Actors：商家
   
   Type：Primary
   
   Description：商家查看接收自顾客的订单，根据时间顺序，准备订单上的菜品，当准备完成后将订单标记为已完成。
   
#### 4. Use case：处理付款

   Actors：客户，商家
   
   Description：客户选择付款方式，微信支付或线下支付，当支付完成后，商家接收其支付，将其订单标记为已完成，已付款。

### 扫码点餐Casual用例

#### 1.Use case:管理订单

  主成功场景：顾客提交订单，服务员/老板通过系统查看，并对顾客点的每样菜品进行处理（出菜后对商品标记已完成）
  
  交替场景：
  
  如果某样菜品暂时不能提供，则告知客户并让客户重新提交订单；
  
  如果网络出现问题，导致订单无法查看，则让服务员询问顾客，人工点单；
  
  如果系统崩溃，联络点餐系统负责人；
  
  如果误操作菜品（未出菜但点击已完成），则可以拉取最初的订单。

#### 2.Use case:查看订单

  主成功场景：顾客可以在主界面中点击“查看订单”，确认自己已经成功点餐的菜品，或者对点餐成功的菜品进行催单操作。
  
  交替场景：
  
  如果客户还未选择菜品，显示订单页面为空；
  
  如果由于网络原因提交失败，重新提交，还是不能提交可以呼叫服务员；
  
  如果显示的订单与自己所点的不一致，则可以呼叫服务员进行处理；
 
  如果系统出现问题，如卡死或者其他无法操作的情况，则需要人工呼叫服务员进行处理；  
  如果需要额外增加菜品，则需要返回到点餐界面进行操作;
  
  如果已经进行过催单操作，还想再次催单，需要联系服务员。
  
#### 3.Use case:处理付款

  主成功场景：顾客点餐后进入支付界面，选择微信支付，跳转到输入微信支付密令界面，输入正确支付密令后，完成付款。
  
  交替场景：  
  如果顾客输入的微信支付密令错误，程序会从输入微信支付密令界面跳转到支付失败界面。顾客可选择再次支付。
  
  如果顾客微信余额不足，程序会从输入支付密令界面跳转到余额不足界面。顾客可在微信充值后重新支付或直接到饭店前台  支付。
  
  如果顾客网络质量不好，程序会无法进入支付界面，或无法跳转到微信支付密令界面，显示网络质量差的信息。顾客可选择重新支付或直接到饭店前台支付。
  
  
#### 4.Use case:管理商品

  主成功场景：商家更新商品的名称、价格、图片，添加商品种类，删除商品种类，增加商品的库存数目，减少商品的库存数目。
  
  交替场景：
  
  商家登录超时，需重新登录，管理商品。
  
  当商品库存为0时，商家减少商品的库存数目时会发生错误，拒绝商家减少商品库存数目。

**功能点：**


- 点餐
 
  ![order](https://raw.githubusercontent.com/E-Order/Dashboard/master/document/Requirement_Specification/系统顺序图/order_ssd.png)
        
- 管理购物车
![管理购物车15331209](https://github.com/E-Order/Dashboard/blob/master/document/Requirement_Specification/%E7%B3%BB%E7%BB%9F%E9%A1%BA%E5%BA%8F%E5%9B%BE/15331209_%E7%AE%A1%E7%90%86%E8%B4%AD%E7%89%A9%E8%BD%A6.png)

- 提交订单
![提交订单15331236](https://github.com/E-Order/Dashboard/blob/master/document/Requirement_Specification/%E7%B3%BB%E7%BB%9F%E9%A1%BA%E5%BA%8F%E5%9B%BE/post_order_ssd.png)
- 处理付款
![处理付款15331213](https://github.com/Evene/Dashboard/blob/master/img/pay_sequence.PNG)

- 添加商品
![添加商品15331217](https://github.com/E-Order/Dashboard/blob/master/document/Requirement_Specification/%E7%B3%BB%E7%BB%9F%E9%A1%BA%E5%BA%8F%E5%9B%BE/15331217-%E6%B7%BB%E5%8A%A0%E5%95%86%E5%93%81.png)

- 更新商品
![更新商品15331217](https://github.com/E-Order/Dashboard/blob/master/document/Requirement_Specification/%E7%B3%BB%E7%BB%9F%E9%A1%BA%E5%BA%8F%E5%9B%BE/15331217-%E6%9B%B4%E6%96%B0%E5%95%86%E5%93%81.png)

- 添加种类
![添加种类15331217](https://github.com/E-Order/Dashboard/blob/master/document/Requirement_Specification/%E7%B3%BB%E7%BB%9F%E9%A1%BA%E5%BA%8F%E5%9B%BE/15331217-%E6%B7%BB%E5%8A%A0%E7%A7%8D%E7%B1%BB.png)

- 删除商品
![删除商品15331198](https://github.com/E-Order/Dashboard/blob/master/document/Requirement_Specification/%E7%B3%BB%E7%BB%9F%E9%A1%BA%E5%BA%8F%E5%9B%BE/15331198_%E5%88%A0%E9%99%A4%E5%95%86%E5%93%81.PNG)

- 删除种类
![删除种类15331198](https://github.com/E-Order/Dashboard/blob/master/document/Requirement_Specification/%E7%B3%BB%E7%BB%9F%E9%A1%BA%E5%BA%8F%E5%9B%BE/15331198_%E5%88%A0%E9%99%A4%E5%95%86%E5%93%81%E7%A7%8D%E7%B1%BB.PNG)

- 查看订单
![查看订单15331286](https://github.com/E-Order/Dashboard/blob/master/document/Requirement_Specification/%E7%B3%BB%E7%BB%9F%E9%A1%BA%E5%BA%8F%E5%9B%BE/%E6%9F%A5%E7%9C%8B%E8%AE%A2%E5%8D%95_15331286.png)

  以*查看订单*为例，绘制活动图

<img src = "https://raw.githubusercontent.com/E-Order/Dashboard/master/document/graph/activity.png">

**非功能性需求：**

1. 数据格式：
 价格精确到两位小数。（如12.00）


2. 性能：
 - 可接受至少50桌用户同时点餐
 - 商家实时接收订单消息，延迟不得超过10s

3. 安全性：
  需保障商家用户数据的安全，对账号密码进行保护。
 
4. 可靠性:
- 从远程服务失败中恢复，提供远程服务的本地实现，比如：本地产品信息数据库缓存最常用的一小部分产品信息。
- 限制服务器最大连接数，防止由于服务器负载过重而使用户掉线

5. 可适应性：
  针对第三方服务，创建一个实现了接口的资源适配器对象（主要针对支付功能）
 
 
