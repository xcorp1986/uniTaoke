<template>
	<view>
		<view style="position: absolute; right: 10px; top: 10px;">
			<view @click="favToggle" class="fav-btn" :class="{'fav-btn-active':isFav}" ></view>
		</view>
		<image class="d-img" :src="data.imgurl" mode="widthFix"></image>
		<view class="row-box">


			<view class="d-title">
				{{data.title}}
			</view>
			<view class="d-desc">
				{{data.description}}

			</view>
			<view class="flex">
				<span class="cl-money">券后价：￥{{data.price}}</span>
				<span class="flex-1 text-center">销量 {{data.sold_num}}</span>
				<view class="juan-price flex">
					<span class="juan-price-q">券</span>
					<span class="juan-price-m">￥{{data.juan_money}}</span>
				</view>
			</view>



			<a href="javascript:;" id="js-detail" class="look-text">查看图文详情</a>
			<view class="p-detail">
				{{data.content}}
			</view>

			<view class="fine_out">
				<view class="fine">
					<span class="fine_in"><span>精品</span>推荐</span>
				</view>
			</view>
			 
			<view class="tk-list">
				<view @click="setPage(item.id)" v-for="(item,index) in reclist" :key="index" class="tk-item" >
					<view class="tk-item-img-box">
						<img class="tk-item-img" :src="item.imgurl" />
						<view class="tk-juan-price">券￥{{item.juan_money}}</view>
					</view>
					<view class="tk-title">{{item.title}}</view>
					<view class="tk-row-price">
						<span class="tk-price">￥{{item.price}}(券后)</span>
						<span class="tk-sold">销量{{item.sold_num}}</span>
					</view>
				</view>
			</view>

		</view>
		<view style="height: 60px;"></view>
		<view class="bfooter">
			<view class="flex-1 flex">券后价<span class="cl-money f12">{{data.price}}</span></view>
			<view class="bfooter-yq flex">优惠：<span class="cl-money f12">￥{{data.juan_money}}</span> </view>
			<view class="bfooter-pwd  f12" @click="fixboxClass='flex-col'">口令购买</view>
			<!-- #ifdef H5 -->
			<a target="_blank" :href="lqUrl"  class="bfooter-lq  f12">立即领券</a>
			<!-- #endif --> 
			
			 
		</view>

		<view id="fixbox" :class="fixboxClass" class="fixbox">

			<view class="hd">淘口令购买
				<view @click="fixboxClass=''" class="f_close fixbox-close iconfont icon-close"></view>
			</view>
			<view class="box">
				<view class="tit">{{data.title}}</view>
				<view class="flex">券后价：<span class="cl-money">￥{{data.price}}</span></view>
				<view>复制这条信息</view>
				<span style="-webkit-user-select:text" class="num " >{{data.juan_pwd}}</span>
				<view class="flex f12">打开<span class="num f12">[手机淘宝]</span>即可领取优惠券购买</view>
			</view>
			<view class="copytext" style="margin:0 auto!important" @click="copy(data.juan_pwd)">一键复制</view>
			<view class="desc">由于部分浏览器不支持一键复制，<br>若一键复制失败，请长按文字手动复制</view>
		</view>
	</view>
</template>

<script>
	// #ifdef H5
	import Vue from 'vue' 
	import VueClipboard from  "../../common/vue-clipboard-cli.js";
	VueClipboard.config.autoSetContainer = true
	Vue.use(VueClipboard);
	// #endif 
	import taokeItem from "../../components/taoke-item.vue";
	export default {
		components: {
			taokeItem
		},
		data: function() {
			return {
				data: {},
				reclist: {},
				id: 0,
				isFav: false,
				fixboxClass:"",
				lqUrl:""
			}
		},
		onLoad: function(ops) {
			 
			this.id = ops.id;
			this.getPage();
		},
		methods: {
			setPage:function(id){
				this.id = id;
				this.getPage();
				uni.pageScrollTo({
					scrollTop:0
				})
			},
			favToggle:function(){
				var that=this;
				that.app.get({
					url:that.app.apiHost+"/index.php?m=fav&a=toggle&ajax=1",
					data:{
						tablename:"mod_taoke",
						objectid:that.id
					},
					success:function(res){
						if(res.error){
							uni.showToast({
								title:res.message
							})
							return false;
						}
						if(res.data=='add'){
							that.isFav=true;
							 
						}else{
							that.isFav=false;
						}
					}
				})
			},
			getPage: function() {
				var that = this;
				that.app.get({
					url: that.app.apiHost + "/module.php?m=taoke&a=show&id=" + that.id,
					success: function(res) {
						that.data = res.data.data;
						that.reclist = res.data.reclist;
						that.isFav = res.data.isfav;
						if(res.data.data.juan_url){
							that.lqUrl=res.data.data.juan_url;
						}else{
							that.lqUrl=res.data.data.tk_url;
						}
					}
				})
			},
			copy:function(msg){
				var that=this;
				if(msg==''){
					msg="内容为空";
				}
				//var msg="aaaa";
				// #ifdef H5
				this.$copyText(msg).then(function (e) {
				  uni.showToast({
				  	title:"复制成功"
				  })
				}, function (e) {
				  uni.showToast({
				  	title:"复制失败"
				  })
				})
				// #endif
				// #ifndef H5
				uni.setClipboardData({
					data: msg,
					success: function () {
						console.log('success');
						// #ifdef APP-PLUS
						that.goTaobao();
						// #endif
					}
				});
				// #endif
			},
			goTaobao:function (){
				// #ifdef APP-PLUS
				if (plus.os.name == "Android" ) {
					plus.runtime.launchApplication( {pname:"com.taobao.taobao"
						,extra:{url:"https://m.taobao.com/#index"}}, 
						function ( e ) {
							skyToast( "打开淘宝失败，请先安装淘宝" );
						}
					);
				}else if ( plus.os.name == "iOS" ) {
					plus.runtime.launchApplication( {action:"taobao://"}, function ( e ) {
						skyToast( "打开淘宝失败，请先安装淘宝" );
					} );
				}
				// #endif
			}
		}
	}
</script>

<style>
	@import url("../../common/taoke.css");
	.fav-btn {
		width: 36px;
		height: 36px;
		text-align: center;
		line-height: 36px;
		border-radius: 50%;
		cursor: pointer;
		background-color: #333;
		opacity: 0.9;
		color: #fff;
	}

	.fav-btn:before {
		color: #666;
		font-family: iconfont;
		content: "\e64c";
	}

	.fav-btn-active {
		background-color: #e91e63
	}

	.fav-btn-active:before {
		color: #fff;
		font-size: 22px;
	}

	.juan-price {
		float: right;
		height: 20px;
	}

	.juan-price:after {
		clear: both;
		display: block;
		content: ".";
		visibility: hidden;
	}

	.juan-price-q {
		box-sizing: border-box;
		background: #ff4d36;
		width: 25px;
		height: 25px;
		line-height: 25px;
		display: block;
		text-align: center;
		color: #fff;
		border-top-left-radius: 3px;
		border-bottom-left-radius: 3px;
		font-size: 12px;
	}

	.juan-price-m {
		border: 1px #ff4d36 solid;
		height: 25px;
		line-height: 25px;
		text-align: center;
		padding: 0 2px;
		font-size: 12px;
		border-top-right-radius: 3px;
		border-bottom-right-radius: 3px;
		padding: 0px 5px;
		box-sizing: border-box;
	}
	.look-text {
    padding: 6px 8px;
    border: 1px #ff4e36 solid;
    border-radius: 20px;
    margin:20px auto;
    display: block;
    width: 50%;
    text-align: center;
    color: #ff4e36;
    font-size: 16px;
}
.fine_out {
    margin-bottom: 4px;
    border: 1px solid white;
    background-color: white;
}
.fine_out  .fine {
    position: relative;
    width: 98%;
    margin: 20px auto 0;
    border-top: 1px solid #f5f5f5;
    text-align: center;
    color: #292929;
    font-size: 16px;
}
.fine_out  .fine .fine_in {
    background-color: white;
    top: -10px;
    position: relative;
    display: inline-block;
    padding: 0 10px;
}
.fine .fine_in span {
    font-weight: bolder;
    color: #ff4d36;
}
.bfooter{
	height: 50px;
	line-height: 50px;
	background-color: #fff;
	position: fixed;
	bottom: 0;
	left: 0;
	right: 0;
	padding-left: 5px;
	display: flex;
	flex-direction: row;
	font-size: 14px;
	color: #444;
}
.bfooter .flex-1{
	flex: 1;
	text-align: center;
}
.bfooter-yq{
	margin-right: 5px;
}
.bfooter-price{
	color: #ff4e36;
}
.bfooter-juan{
	color: #ff4e36;
	font-size: 12px;
}
.bfooter-pwd,.bfooter-lq{
	width: 21%;
	color: #fff;
    background: #ff9a50;
    cursor: pointer;
    text-align: center;
}
.bfooter-lq{
	background-color: #ff4e36;
	display: block;
}
/***fixbox**/
.fixbox{
	display: none;
	position: fixed;
	left: 50%;	 
	top: 50%;
	width: 290px;
	height: 300px;
	margin-left: -145px;
	margin-top: -150px;
	background-color: #fff;
	border-radius: 10px;
}
.fixbox .hd{
	width: 100%;
    height: 30px;
    line-height: 30px;
    font-size: 16px;
    text-align: center;
    margin-top: 5px;
    color: #ff4d36;
    overflow: hidden;
    zoom: 1;
    position: relative;
}
.fixbox .f_close{
	position: absolute;
	right: 10px;
	top: 10px;
	width: 16px;
	cursor: pointer;
}
.fixbox .box{
	width: 95%;
    min-height: 100px;
    border-radius: 5px;
    border: 1px dashed #ff4e65;
    position: relative;
    text-align: left;
    padding: 10px;
    resize: none;
    display: block;
    margin: 10px auto;
    line-height: 24px;
    color: #666;
    
    word-wrap: break-word;
    overflow-x: hidden;
    overflow-y: auto;
    font-size: 14px;
}

.fixbox .tit{
	font-size: 14px;
	overflow: hidden;
	white-space: normal;
	height: 26px;
    line-height: 26px;
}

.fixbox .num{
	color: #ff4d36;
}
.fixbox .copytext{
	font-size: 14px;
    width: 100px;
    height: 35px;
    color: #fff;
    background: #ff4d36;
    text-align: center;
    line-height: 35px;
    margin: 5px auto;
    border-radius: 5px;
    margin-bottom: 0;
    cursor: pointer;
}
.fixbox .desc{
	width: 100%;
    margin: 0 auto;
   
    color: #999;
    font-size: 12px;
    margin-top: 10px;
    text-align: center;
}

</style>
