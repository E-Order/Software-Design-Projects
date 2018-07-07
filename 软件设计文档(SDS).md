# 软件设计文档(SDS)

- [E-Order Frontend (微信小程序)](#1)
    - [安装](#4)
    - [技术选择](#5)
    - [架构设计](#6)
    - [模块划分](#7)
    - [UI设计指南](#8)
- [Seller Management System Frontend (商家)](#2)
    - [技术选择](#9)
    - [架构设计](#10)
    - [模块划分](#11)
- [E-Order Backend](#3)
    - [技术选择](#12)
    - [架构设计](#13)
    - [模块划分](#14)
    - [数据库设计](#15)
    - [API设计](https://ordermeal.docs.apiary.io/#)

<h2 id='1'> 一、E-Order Frontend (微信小程序) </h2>

- 用户端设计采用微信小程序来实现。

<h3 id='4'> 1. 安装：</h3>
用户通过扫码进入小程序即可。

<h3 id='5'> 2. 选用微信小程序的理由如下(技术选型理由)：</h3>

- 该框架类似`Flux`架构。在`Flux`架构当中，`Views`查询`Stores`（而不是`Models`），并且用户交互将会触发`Actions`，`Actions`则会被提交到一个集中的`Dispatcher`当中。当`Actions`被派发之后，`Stores`将会随之更新自己并且通知`Views`进行修改。这些`Store`当中的修改会进一步促使`Views`查询新的数据。
- 微信小程序很好地实现了前后端同时开发的要求。微信提供`wx.request`这个API接口来让小程序能够因为一些操作与服务器进行交互。
- 微信用户数量庞大，使用微信小程序能够使得用户到店能够直接使用点餐系统，无需下载额外软件来完成点餐。

<h3 id='6'>  3. 架构设计 </h3>

- `Flux`架构
  改变显示数据的时候采用`wx.setData`来使得数据的同步化。若不采用`wx.setData`，改变的数据不会在`Views`上更新显示。
  
  ```javascript
  // index.js (line 155 - line 157)
  that.setData({
    foodNum: res.data,
  });
  ```
- 三层模型（表示层、业务层、持久化层）
  - 表示层
    使用wxml和wss类似html5的来显示数据，并且页面上有相应的按键来提供用户接口。
    ```
    <!-- index.wxml (line 29 - line 39) -->
    <view wx:if="{{foodNum[curIndex][index]==0}}" >
      <!--AddToCart用于实现购物车图标变为加减号，注意catch与bind的区别-->
      <image class="image-add" src="../../images/footer-icon-cart-active.png" catchtap="AddToCart" data-colindex="{{index}}"></image>
    </view>
    <view wx:if="{{foodNum[curIndex][index]!=0}}">
      <!--addFood用于实现增加数量-->
      <image class="image-add" src="../../images/+.png" catchtap="addFood" data-colindex="{{index}}"></image>
      <text class="num">{{foodNum[curIndex][index]}}</text>
      <!--subFood用于实现减少数量-->
      <image class="image-sub" src="../../images/-.png" catchtap="subFood" data-colindex="{{index}}"></image>
    </view>
    ```
    其中`catchtap`提供了用户接口。
  - 业务层
    使用JavaScript来实现相应的业务逻辑。例如在表示层显示的`AddToCart`对应的逻辑实现。
    - 用户接口的逻辑实现
    主要为`catchtap`的处理。
      ```javascript
      // index.js
      /**
       * @method AddToCart
       * @param e
       * @desc 添加购物车，购物车图片变为加减号
       */
      AddToCart:function(e) {
        var that = this;
        // 食物类别下标
        let row = this.data.curIndex;
        // 食物类别中的具体点击的食物下标
        let col = parseInt(e.target.dataset.colindex);
        console.log(row, col);
        var arr = this.data.foodNum;
        arr[row][col] = 1;
        console.log(arr);
        that.setData({
          foodNum: arr,
        });
      },
      ```
      - 非用户接口的逻辑实现
      主要为数据的传递。通过`wx.setStorage`和`wx.getStorage`来实现。
      ```javascript
      // index.js
      /**
       * @method onShow
       * @desc 当重新返回该页面的时候，调用函数setFoodNum()来赋值foodNum
       */
      onShow:function() {
        this.setFoodNum();
        console.log("foodNum", this.data.foodNum);
      },

      /**
       * @method setFoodNum
       * @desc 改变foodNum中的数据, 以便使得和购物车显示同步
       */
      setFoodNum:function() {
        var that = this;
        wx.getStorage({
          key: 'foodNum',
          success: function(res) {
            that.setData({
              foodNum: res.data,
            });
          }
        });
      },

      /**
       * @method onHide
       * @desc 进行数据传递，数据传递的方式是通过本地的存储，传递的数据是foodNum，
       * 整个菜单的data，以及当前的导航栏
       */
      onHide:function() {
        wx.setStorage({
          key: 'navLeftItems',
          data: this.data.navLeftItems,
        });
        wx.setStorage({
          key: 'foodNum',
          data: this.data.foodNum,
        });
      },
      ```
  - 持久化层
    通过`wx.request`实现与服务器访问的过程，实现查询(主要)和CRUD。
    - 当`wx.request`的`mode`为`GET`的时候，实现查询操作。
    - 当`wx.request`的`mode`为`POST`的时候，实现CRUD操作。
    
    ```javascript
    // index.js(实现查询操作)
    /**
     * @method getFood
     * @desc 通过调用服务器的API来获得相应商家的菜单信息
     */
    getFood: function() {
      var that = this;
      wx.request({
        url: config.service.getProductUrl,
        method: 'GET',
        data: {
          'sellerId': app.globalData.sellerId
        },
        header: {
          'Content-Type': 'application/json'
       },
        success: function(res) {
          console.log("foodlist",res.data);
          setTimeout(function () {
            that.setData({
              loadingHidden: true
            })
          }, 1500);
          that.initData(res);
        }
      });
    },
    
    // pay.js(实现CRUD操作)
    /**
     * @method submitOrder
     * @param {String} options 表示该订单选择哪种付款方式
     * @desc 向服务器提交订单并完成相应的后续工作
     */
    submitOrder: function(options) {
      console.log('submit_sellerId:', app.globalData.sellerId);
      console.log('submit_deskId:', app.globalData.tableNo);
      console.log('submit_openid:', app.globalData.openId);
      var that = this;
      console.log('JSON:', JSON.stringify(that.data.orderItems));
      wx.request({
        url:config.service.creatOrderUrl,
        data: {
          'deskId': app.globalData.tableNo,
          'openid': app.globalData.openId,
          'sellerId':app.globalData.sellerId,
          'amount': that.data.amount,
          'items': JSON.stringify(that.data.orderItems)
        },
        header: {
          'content-type': 'application/x-www-form-urlencoded'
        },
        method: 'POST',
        success: function(res) {
          console.log(res.data);
          if (options == 'pay_offline') {
            console.log('pay_offline');
            that.offlineOperate();
          } else if (options == 'pay_online') {
            console.log('pay_online');
            that.onlineOperate(res.data.data.orderId);
          }
        }
      });
    },
    ```

<h3 id='7'> 4. 模块划分 </h3>
模块的划分主要通过Page来实现。

- Page index  
  显示菜单，可让用户选择菜品添加到购物车或查看菜品相应的详细信息。
- Page cart  
  显示用户添加的菜品以及相应的价格，可让用户实现下单。
- Page myInfo  
  显示用户在该系统的相应信息(例如历史订单等等)。
- Page detail  
  显示菜品的详细信息。
- Page myorder  
  显示用户的订单。
- Page pay  
  提供用户“线下付款”和“线上付款”两种选择。
- Page error  
  通过扫码获取相应的商家ID和桌号，以便显示相应的菜单和给对应的商家下单。
  
<h3 id='8'> 4. 设计指南 </h3>
设计指南建立在充分尊重用户知情权与操作权的基础之上。旨在微信生态体系内，建立友好、高效、一致的用户体验，同时最大程度适应和支持不同需求，实现用户与小程序服务方的共赢。

####  a) 友好礼貌
为了避免用户在微信中使用小程序服务时，注意力被周围复杂环境干扰，小程序在设计时应该注意减少无关的设计元素对用户目标的干扰，礼貌地向用户展示程序提供的服务，友好地引导用户进行操作。

#### b) 重点突出
每个页面都应有明确的重点，以便于用户每进入一个新页面的时候都能快速地理解页面内容。在确定了重点的前提下，应尽量避免页面上出现其它与用户的决策和操作无关的干扰因素。

E-order页面示例：
![点餐界面][1]  
点餐界面简洁，左边为菜品分类栏，右方为该分类的所有菜品，上方滑动显示热销榜单。用户能迅速理解页面内容，整个页面没有与用户决策和操作无关的干扰因素

#### c) 流程明确
为了让用户顺畅地使用页面，在用户进行某一个操作流程时，应避免出现用户目标流程之外的内容而打断用户。

E-order页面示例：

![购物车][2] 
![支付][3]

从购物车跳转支付界面，符合一般操作流程。没有出现用户目标流程之外的内容而打断用户

#### d) 清晰明确
一旦用户进入E-order小程序页面，用户能明确的知道身在何处、又可以往何处去，确保用户在页面中游刃有余地穿梭而不迷路。

#### e) 导航明确，来去自如
导航是确保用户在网页中浏览跳转时不迷路的最关键因素。E-order下方导航航简洁明确的告诉用户，当前在哪，可以去哪，如何回去。所有的次级页面左上角都提供返回上一级页面操作。此外，微信iOS用户还可通过界面边缘向右滑动操作，返回上一级小程序或微信页面。安卓用户可通过物理返回键达到同样目的。

导航分页栏颜色颜色与整体界面颜色统一，图片样式符合用户生活习惯，点击时颜色改变，达到友好交互的目的。

#### f) 减少等待，反馈及时
页面的过长时间的等待会引起用户的不良情绪，使用微信小程序项目提供的技术已能很大程度缩短等待时间。即便如此，当不可避免的出现了加载和等待的时候，E-order提供加载中图片的反馈以舒缓用户等待的不良情绪。

E-order页面示例：
![loading][4]

#### g) 便捷优雅
从PC时代的物理键盘鼠标到移动端时代手指，虽然输入设备极大精简，但是手指操作的准确性却大大不如键盘鼠标精确。为了适应这个变化，需要开发者在设计过程中充分利用手机特性，让用户便捷优雅的操控界面。
E-order操作界面简单美观更便捷。点击购物车图标，购物车图标将转为添加，删除符号。符合用户操作习惯。



  [1]: https://raw.githubusercontent.com/LTimmy/markdownPhotos/master/UI1.png
  [2]: https://raw.githubusercontent.com/LTimmy/markdownPhotos/master/UI4.png
  [3]: https://raw.githubusercontent.com/LTimmy/markdownPhotos/master/UI8.png
  [4]: https://raw.githubusercontent.com/LTimmy/markdownPhotos/master/UI9.png
  
<h2 id='2'> 二、Seller Management System Frontend (商家) </h2>  

<h3 id='9'> 1. 技术选择：</h3>

- 采用Vue.js+Element UI 的框架开发商家管理的Web客户端单页面应用
- 使用nodejs开发Web Server


<h3> 2. 技术选型理由：</h3>

Vue.js:
- Vue.js是一个轻巧、高性能、可组件化的MVVM库，同时拥有非常容易上手的API

-   Vue.js是一套构建用户界面的 渐进式框架。与其他重量级框架不同的是，Vue 采用自底向上增量开发的设计。Vue的核心库只关注视图层，并且非常容易学习，非常容易与其它库或已有项目整合。另一方面，Vue完全有能力驱动采用单文件组件和 Vue 生态系统支持的库开发的复杂单页应用。数据驱动+组件化的前端开发。

- 简而言之：Vue.js是一个构建数据驱动的 web 界面的渐进式框架。Vue.js 的目标是通过尽可能简单的 API 实现响应的数据绑定和组合的视图组件。核心是一个响应的数据绑定系统。

Element UI :
- Element UI 是一套采用 Vue 2.0 作为基础框架实现的组件库，它面向企业级的后台应用，能够帮助你快速地搭建网站，极大地减少研发的人力与时间成本。它不依赖于vue,但是却是当前和vue配合做项目开发的一个比较好的ui框架。

<h3 id='10'> 3. 架构设计 </h3>

**MVVM**：

- MVVM是把MVC里的Controller和MVP里的Presenter改成了ViewModel。Model+View+ViewModel。

![image.png](https://upload-images.jianshu.io/upload_images/13021922-8babf46ec7234d26.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 把Model和View关联起来的就是ViewModel。ViewModel负责把Model的数据同步到View显示出来，还负责把View的修改同步回Model。

- View的变化会自动更新到ViewModel,ViewModel的变化也会自动同步到View上显示。
这种自动同步是因为ViewModel中的属性实现了Observer，当属性变更时都能触发对应的操作。

- **View 层**，作为视图模板存在，在 MVVM 里，整个 View 是一个动态模板。除了定义结构、布局外，它展示的是 ViewModel 层的数据和状态。View 层不负责处理状态，View 层做的是 数据绑定的声明、 指令的声明、 事件绑定的声明。
```
<el-table
        :data="tableData" //----数据绑定
        @filter-change="handleFilterChange" //----事件绑定
        style="width: 100%">
```
- ViewModel 层把 View 需要的层数据暴露，并对 View 层的 数据绑定声明、 指令声明、 事件绑定声明 负责，也就是处理 View 层的具体业务逻辑。ViewModel 底层会做好绑定属性的监听。当 ViewModel 中数据变化，View 层会得到更新；而当 View 中声明了数据的双向绑定（通常是表单元素），框架也会监听 View 层（表单）值的变化。一旦值变化，View 层绑定的 ViewModel 中的数据也会得到自动更新。

```
handleFinish(index, row) {
            this.idx = row.index;
            this.Tofinish = true;
        },
```
- **Model 层**，对应数据层的域模型，它主要做域模型的同步。通过 Ajax/fetch 等 API 完成客户端和服务端业务 Model 的同步。

```
axios.get(this.url, {params:{
                orderId: this.searchCriteria.orderId,
                deskId: this.searchCriteria.deskId,
                orderStatus: this.searchCriteria.orderStatus,
                payStatus: this.searchCriteria.payStatus,
                orderDate: this.searchCriteria.orderDate,
                page: this.cur_page-1,
                size: this.page_size
            }}).then((res) => {
                //console.log(res);
                this.tableData = res.data.data;
                this.orderNum = res.data.total;
                for (var i = 0; i < this.tableData.length; i++) {
                    Vue.set(this.tableData[i],'index',i);
                    d = this.Format(this.tableData[i].createTime*1000,"yyyy-MM-dd");
                    Vue.set(this.tableData[i],'date',d);
                    this.getDetail(i);
                }
            }).catch((error) => {
                console.log(error);
            });
```

<h3 id='11'> 4. 模块划分 </h3>

- 一个组件表示一个页面, 通过rooter路由渲染不同页面（一个js一个组件）

    - 路由： index.js
    - 用户登陆：signInShow.js
    - 用户注册：signUpShow.js
    - 商家设置：UserSetting.js
    - 订单管理:  orderManagement.js
    - 商品管理：
    - 类目管理：productManagement.js
    - 类目菜品管理：editProducts.js

<h2 id='3'> 三、E-Order Backend </h2>

<h3 id='12'> 1.技术选择 </h3>

- **Spring Boot**是有Pivotal团队提供的全新框架，其设计目的是用来简化Spring应用的初始搭建以及开发过程。该框架使用了特定的方式来进行配置，从而使开发人员不再需要定义样板化的配置，我们也可以称它为SpringMVC框架的精简版。它最大的优点就是摆脱了Spring框架中各种复杂的配置，同时继承了大量常用的第三方库配置，开发人员在使用的时候只需要少量的配置代码，可以进行快速、敏捷地开发。
    * 独立运行的Spring 项目，可以以jar包的形式独立运行
    * 内嵌Servlet 容器
    * 提供starter简化Maven 配置
    * 自动配置Spring
    * 准生产的应用监控
    * 无代码生成和xml配置

<h3 id='13'> 2.架构设计 </h3>

#### 三层架构

- 表示层（present layer）: 
    - 处理 HTTP 输入（Request），然后调用业务服务，产生输出（Response）。这里，应用开发人员只需要考虑 MVC 三个编程元素。
    
    - MVC把三层架构中的UI层再度进行了分化，分成了控制器、视图、实体三个部分，控制器完成页面逻辑，通过实体来与界面层完成通话
    
       - M（模型/Model）：业务涉及的数据对象实例。例如，你显示一定订单，它包含用户、订单、订单项、支付等业务 对象数据；
       - V（视图/View）：展示业务人机交互界面的显示模板。例如，jsp 文件等，它能将模型中的数据填入显示模板，用户可看到界面元素丰富的界面；
       - C（控制器/Controller）：用户输入处理单元。它检查输入的合法性，处理输入表单，按流程调用业务函数，生成输出需要的数据模型，选择视图模板，输出。
       - 使用MVC结构编程，使得程序业务逻辑清晰，模块结构好。对提升开发效率，增强可维护性，促进团队内部按技能分工，起到关键作用，因此几乎所有语言都有自己若干不同的 MVC 支持框架实现。
       
- 业务层（business layer）: 
    - 提供满足 ACID 要求的业务服务。这里，开发人员只需声明对外的业务服务函数，spring 等提供事务（Transaction）支持。
- 持久化层（persistence layer）：
    - 提供数据表存取机制，主要是 ORM 框架实现以对象-关系数据库的映射。这里，开发人员只需声明表和对象的映射，特殊的 SQL。

#### E-order逻辑架构

![](https://raw.githubusercontent.com/E-Order/Dashboard/master/document/graph/%E9%80%BB%E8%BE%91%E8%A7%86%E5%9B%BE.png)



<h3 id='14'> 3.模块划分 </h3>

#### 应用程序目录

![](https://github.com/E-Order/Dashboard/raw/master/document/graph/%E5%8C%85%E5%9B%BE.png)

- Controller层(用户访问后台的接口，通过用户提交的数据检查输入合法性，处理表单，进而调用service)
  - BuyerOrderController（买家端对于订单的操作）
  - BuyerProductController（买家端对于商品的操作）
  - SellerCategoryController（卖家端对于商品种类的操作）
  - SellerInfoController（卖家端用户信息操作）
  - SellerOrderController（卖家端对于订单操作）
  - SellerProductController（卖家端对于商品操作）
  - SellerUserController（卖家端用户注册登录登出）
  - WeixinController（微信授权，获取openid）
- Service层(提供可以对数据库对象进行的操作服务)
  - BuyerService（检查当前用户是否授权）
  - CategoryService （对商品种类进行操作）
  - OrderService（对订单进行操作）
  - ProduceService（对商品进行操作）
  - SellerService（对卖家信息进行操作）
- DAO层repository （数据库接口，通过DAO对数据库对象进行访问）
  - OrderDetailRepository
  - OrderMasterRepository
  - ProductCategoryRepository
  - ProductInfoRepository
  - SellerInfoRepository

<h3 id='15'> 4.数据库设计 </h3>

![](https://github.com/E-Order/Dashboard/raw/master/document/graph/%E6%95%B0%E6%8D%AE%E5%BA%93%E8%AE%BE%E8%AE%A1.png?raw=true)

- order_detail

```
CREATE TABLE `order_detail` (
  `detail_id` varchar(32) NOT NULL,
  `order_id` varchar(32) NOT NULL,
  `product_quantity` int(11) NOT NULL,
  `product_id` varchar(32) NOT NULL,
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间',
  PRIMARY KEY (`detail_id`),
  KEY `index_order_id` (`order_id`),
  KEY `index_product_id` (`product_id`),
  CONSTRAINT `order_id` FOREIGN KEY (`order_id`) REFERENCES `order_master` (`order_id`) ON DELETE NO ACTION ON UPDATE NO ACTION,
  CONSTRAINT `product_id` FOREIGN KEY (`product_id`) REFERENCES `product_info` (`product_id`) ON DELETE NO ACTION ON UPDATE NO ACTION
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4

```

- order_master


```
CREATE TABLE `order_master` (
  `order_id` varchar(32) NOT NULL,
  `desk_id` int(11) NOT NULL,
  `seller_id` varchar(32) NOT NULL,
  `buyer_openid` varchar(64) NOT NULL,
  `order_amount` decimal(8,2) NOT NULL,
  `order_status` tinyint(3) NOT NULL DEFAULT '0',
  `pay_status` tinyint(3) NOT NULL DEFAULT '0',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间',
  PRIMARY KEY (`order_id`),
  KEY `index_buyer_openid` (`buyer_openid`),
  KEY `index_seller` (`seller_id`),
  CONSTRAINT `order_seller_id` FOREIGN KEY (`seller_id`) REFERENCES `seller_info` (`seller_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
```

- product_info


```
CREATE TABLE `product_info` (
  `product_id` varchar(32) NOT NULL,
  `product_name` varchar(64) NOT NULL,
  `product_price` decimal(8,2) NOT NULL,
  `product_description` varchar(64) DEFAULT NULL,
  `product_icon` varchar(512) DEFAULT NULL,
  `product_stock` int(11) NOT NULL DEFAULT '0',
  `product_status` tinyint(3) DEFAULT '0',
  `category_type` int(11) NOT NULL,
  `seller_id` varchar(32) NOT NULL,
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间',
  PRIMARY KEY (`product_id`),
  KEY `category_type_idx` (`category_type`),
  KEY `seller_id_idx` (`seller_id`),
  CONSTRAINT `product_category_type` FOREIGN KEY (`category_type`) REFERENCES `product_category` (`category_type`) ON DELETE NO ACTION ON UPDATE NO ACTION,
  CONSTRAINT `product_seller_id` FOREIGN KEY (`seller_id`) REFERENCES `seller_info` (`seller_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
```

- product_categroy

```
CREATE TABLE `product_category` (
  `category_id` int(11) NOT NULL AUTO_INCREMENT,
  `category_name` varchar(64) NOT NULL,
  `category_type` int(11) NOT NULL,
  `seller_id` varchar(32) NOT NULL,
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间',
  PRIMARY KEY (`category_id`),
  UNIQUE KEY `category_type` (`category_type`,`seller_id`),
  KEY `index_seller_id` (`seller_id`),
  CONSTRAINT `category_seller_id` FOREIGN KEY (`seller_id`) REFERENCES `seller_info` (`seller_id`)
) ENGINE=InnoDB AUTO_INCREMENT=8 DEFAULT CHARSET=utf8mb4
```

-  seller_info

```
CREATE TABLE `seller_info` (
  `seller_id` varchar(32) NOT NULL,
  `username` varchar(32) NOT NULL,
  `password` varchar(32) NOT NULL,
  `telephone` varchar(32) NOT NULL,
  `address` varchar(64) NOT NULL,
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间',
  PRIMARY KEY (`seller_id`),
  UNIQUE KEY `username` (`username`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
```

