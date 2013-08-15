	
					           根据商品ID取相应的信息
	

1.method init.
2.param {String|HTMLElement} selector 选择器.
3.HTML结构如下.

	<ul class="J_Ul">
	    <li>
	       <a href="#" class="J_Items" data-shopIds = "">
	           <img src=""/>
	       </a>
	       <div class="">
	           <div class=""><em class="J_ItemNewPrice"></em></div>
		   <div class=""><em class="J_ItemOldPrice"></em></div>
	       </div>
	       <div class="">
	           <div class="">最近成交<em class="J_ItemSales"></em> | 已有<em class="J_ItemReviews"></em>条评论</div>
	       </div>
	       <div class="">
	           <i class="J_New">新品</i>
		   <i class="J_Hot">热卖</i>
		   <i class="J_Sales">促销</i>
		   <i class="J_Edm">包邮</i>
	       </div>
	   </li>
      </ul>
      <ul class="J_Ul"></ul>
	
     1.必须提供一个板块容器 如上所示 class="J_Ul". 
     2.HTML结构可以随便写 但是要保证 板块里面有J_Items类名 对应的属性 data-shopIds(商品id),J_ItemNewPrice是必须的
     3. 板块容器支持class  
     4. .J_ItemNewPrice->价格(有促销就是促销价), J_ItemOldPrice->市场价, .J_ItemSales->最近成交, J_ItemReviews->评论 
       J_New -> 新品icon，J_Hot -> 热卖icon,J_Sales -> 促销icon, J_Edm -> 包邮icon
     5  其他的类名根据需求而定(可有可无，但是有的话 类名一定要和上面定于的一致)。
     6 以对象的方式传参数
     7 这样做的好处是：1) 添加callback回调函数
	    调用方式如下：
	    
	    var classes = D.query(".J_Kaixin"); 
	    KISSY.use('fp/price',function(S){
		S.price.init({classes:classes},function(){
		    console.log(0);
		});
	    });

    KISSY.add('mod/price',function(S) {
	var D = S.DOM,
	   E = S.Event;
	var API = '';
	var price = {
		init: function(ids, callbackFunction) {
			var me = this;
			this.classes = ids.classes;
			var allId = this.classes;
			if (this.classes.length > 1) {
				S.each(this.classes,
				function(elemClass, curIndex) {
					me.query(elemClass, callbackFunction, curIndex, allId);
				});
			} else {
				me.query(this.classes, callbackFunction);
			}
		},
		idx: 0,
		// kissy1.1.6 在IE下有bug 不能同一个callback名 用此方法解决下
		getCallback: function(fn) {
			var callback = 'jsonp_hitao_' + this.idx++;
			window[callback] = function(data) {
				fn(data);
				try {
					delete window[jsonpCallback];
				} catch(e) {}
			};
			return callback;
		},
		// 请求
		query: function(id, callbackFunction, curIndex, allId) {
			var items, itemArr = [],
			me = this,
			elemClass = D.get(id),
			items = D.query(".J_Items", elemClass);
			S.each(items,
			function(item) {
				var shopId = D.attr(item, 'data-shopIds');
				itemArr.push(shopId);
			});
		
			KISSY.ajax({
				url: API + "?nid=" + itemArr + "&n=20" + "&timestamp=" + S.now(),
				dataType: 'jsonp',
				jsonpCallback: this.getCallback(function(data) {
					var count = data.count;
					if (count <= 0) {
						return;
					}
					var callbackIds = data.items;
					if (allId) {
						me.render(id, itemArr, callbackIds, callbackFunction, curIndex, allId);
					} else {
						me.render(id, itemArr, callbackIds, callbackFunction);
					}
				})
			});
		},
		// 商品id排序 商品以什么的顺序发送到服务器 服务器以什么样的顺序返回
		shopIdsSort: function(itemArr, callbackIds) {
			var obj = {};
			for (var i = 0,
			length = itemArr.length; i < length; i += 1) {
				obj[itemArr[i]] = i;
			}
			for (var j = 0,
			length2 = callbackIds.length; j < length2; j += 1) {
				var item = callbackIds[j];
				item._id = obj[item.itemId];
			}
			var compareId = callbackIds.sort(function(a, b) {
				return a._id - b._id;
			});
			return compareId;
		},
		// 渲染
		render: function(id, itemArr, callbackIds, callbackFunction, curIndex, allIds) {
			var me = this,
			shopIdsSorts = me.shopIdsSort(itemArr, callbackIds),
			obj = {},
			itemIdArr = []; // 获取所有商品id 存入数组itemIdArr
			for (var i = 0,
			shopLength = shopIdsSorts.length; i < shopLength; i += 1) {
				var itemId = shopIdsSorts[i].itemId;
				itemIdArr.push(itemId);
			} // 用object 存储 目的是 一一对应 当单个商品下架时候 单个当前商品获取不到价格 不影响其他商品 
			for (var ii = 0,
			shopL = itemArr.length; ii < shopL; ii += 1) {
				obj[itemArr[ii]] = ii;
			}
			var itemSales = D.query('.J_ItemSales', D.get(id)),
			itemReviews = D.query(".J_ItemReviews", D.get(id)),
			itemNewPrice = D.query(".J_ItemNewPrice", D.get(id)),
			itemOldPrice = D.query(".J_ItemOldPrice", D.get(id)),
			newIcon = D.query(".J_New", D.get(id)),
			hotIcon = D.query(".J_Hot", D.get(id)),
			saleIcon = D.query(".J_Sales", D.get(id)),
			edmIcon = D.query(".J_Edm", D.get(id)),
			items = D.query(".J_Items", D.get(id)); // 遍历商品id 
			for (var k = 0,
			klength = itemIdArr.length; k < klength; k += 1) {
				if (obj.hasOwnProperty(itemIdArr[k])) { // 下面的除了icon显示以外的逻辑 都是根据徐康取价格的逻辑改的
					if (shopIdsSorts[k].soldQuantity === 0 && shopIdsSorts[k].peopleNum === 0 && itemSales[obj[itemIdArr[k]]]) {
						var parnet = D.parent(itemSales[obj[itemIdArr[k]]], 1);
						D.html(parnet, "新品上架，立即抢购");
					} else {
						itemSales[obj[itemIdArr[k]]] && D.html(itemSales[obj[itemIdArr[k]]], shopIdsSorts[k].soldQuantity);
						itemReviews[obj[itemIdArr[k]]] && D.html(itemReviews[obj[itemIdArr[k]]], shopIdsSorts[k].peopleNum);
					}
					itemNewPrice[obj[itemIdArr[k]]] && D.html(itemNewPrice[obj[itemIdArr[k]]], '<i>&yen;</i>' + Math.round(shopIdsSorts[k].newPrice)); // 促销价
					if (shopIdsSorts[k].marketPrice && shopIdsSorts[k].marketPrice !== shopIdsSorts[k].newPrice) {
						itemOldPrice[obj[itemIdArr[k]]] && D.html(itemOldPrice[obj[itemIdArr[k]]], '<i>&yen;</i>' + Math.round(shopIdsSorts[k].marketPrice));
					} // 下面是icon显示问题 如果有那个显示那个 默认情况下 页面icon都是隐藏的(css控制)
					if (shopIdsSorts[k].newItem && newIcon[obj[itemIdArr[k]]]) {
						D.css(newIcon[obj[itemIdArr[k]]], {
							"display": "inline-block"
						});
					}
					if (shopIdsSorts[k].hot && hotIcon[obj[itemIdArr[k]]]) {
						D.css(hotIcon[obj[itemIdArr[k]]], {
							"display": "inline-block"
						});
					}
					if (shopIdsSorts[k].promotion && saleIcon[obj[itemIdArr[k]]]) {
						D.css(saleIcon[obj[itemIdArr[k]]], {
							"display": "inline-block"
						});
					}
					if (shopIdsSorts[k].postFee && edmIcon[obj[itemIdArr[k]]]) {
						D.css(edmIcon[obj[itemIdArr[k]]], {
							"display": "inline-block"
						});
					}
					if (S.UA.ie === 6) {
						E.on(items[obj[itemIdArr[k]]], 'mouseenter',
						function() {
							D.addClass(this, 'box-imgwrap-hover');
						});
						E.on(items[obj[itemIdArr[k]]], 'mouseleave',
						function() {
							D.removeClass(this, 'box-imgwrap-hover');
						});
					}
				}
			} // 当加载完最后一个板块时候 执行回调函数
			if (allIds && (curIndex == (allIds.length - 1))) {
				if (callbackFunction && S.isFunction(callbackFunction)) {
					callbackFunction();
				}
			}
			if (!allIds) {
				if (callbackFunction && S.isFunction(callbackFunction)) {
					callbackFunction();
				}
			}
		}
	};
	S.price = price;
	return price;
    });
