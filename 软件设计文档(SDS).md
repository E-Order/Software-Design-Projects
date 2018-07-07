# 软件设计文档(SDS)

- [E-Order Frontend (微信小程序)](#1)
    - [安装](#4)
    - [技术选择理由](#5)
    - [架构设计](#6)
    - [模块划分](#7)
    - [UI设计指南](#8)
- [Seller Management System Frontend (商家)](#2)

- [E-Order Backend](#3)

<h2 id='1'> E-Order Frontend (微信小程序) </h2>

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
