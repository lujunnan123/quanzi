<template>
	<view class="blogitem">
		<view class="head">
			<view class="userinfo">
				<view class="avator">
					<image :src="giveAvatar(item)" mode="aspectFill"></image>
				</view>
				<view class="name">{{getName(item)}}</view>
				<view class="time">
					<uni-dateformat :date="item.publish_date" format="MM月dd hh:mm"
						:threshold="[60000,3600000*24*7]"></uni-dateformat>
				</view>
			</view>
			<view class="more" @click="clikMore">
				<view class="iconfont icon-elipsis">

				</view>
			</view>
		</view>
		<view class="body">
			<view class="title" @click="goDetail">{{item.title}}</view>
			<view class="text" @click="goDetail">
				<view class="t">
					{{item.description}}
				</view>
			</view>
			<view class="piclist">
				<view class="pic" :class="item.picurls.length==1?'only':''" v-for="(i,index) in item.picurls" :key="i">
					<image @click="clickPic(index)" :src="i" mode="aspectFill"></image>
				</view>
			</view>
		</view>

		<view class="info">
			<view class="box">
				<text class="iconfont icon-browse">
				</text>
				<text>{{item.view_count}}</text>
			</view>
			<view class="box" @click="goDetail">
				<text class="iconfont icon-comment">
				</text>
				<text>{{item.comment_count && item.comment_count>0 ? item.comment_count : '评论'}}</text>
			</view>
			<view class="box" :class="item.isLike?'active':''" @click="clickLike">
				<text class="iconfont icon-good">
				</text>
				<text>{{item.like_count}}</text>
			</view>
		</view>


		<!--  弹出层 -->
		<view>
			<u-action-sheet :actions="list" cancelText="取消" :closeOnClickOverlay="true" :closeOnClickAction="true"
				:show="show" @select="sheetSelect" @close="sheetClose"></u-action-sheet>
		</view>

	</view>
</template>

<script>
	import {
		getName,
		giveAvatar,
		likeFun
	} from "../../utils/tools.js"
	import{store} from'@/uni_modules/uni-id-pages/common/store.js'
	const db = uniCloud.database()
	export default {
		name: "blog-item",
		props: {
			item: {
				type: Object,
				default () {
					return {

					}
				}
			},
			isLike:Boolean,
			like_count:Number
		},
		onLoad() {
			console.log("onload:",this.isLike);
		},
		data() {
			return {
				picArr: [1, 2, 3],
				show: false,
				list: [{
						name: "修改",
						type: "edit",
						disabled: true
					},
					{
						name: "删除",
						type: "del",
						color: "#F56C6C",
						disabled: true
					}
				]
			};
		},
		methods: {
			// 点赞
			clickLike(){
				if(!store.hasLogin){
					uni.showModal({
						title:"请登录后再点赞",
						success: (res) => {
							if(res.confirm){
								uni.navigateTo({
									// 标注1
									url:"/uni_modules/uni-id-pages/pages/login/login-withpwd"
								})
							}
						}
					})
					return
				}
				
				// 防抖处理
				let time = Date.now();
				if(time-this.likeTime < 2000){
					uni.showToast({
						title:"操作太频繁了，请稍后....",
						icon:"none"
					})
					return
				}			
					
				// this.item.isLike ? this.item.like_count-- : this.item.like_count++; 
				// this.item.isLike = !this.item.isLike ;
				let like_count = this.item.like_count
				this.item.isLike?like_count-- : like_count++; 
				let isLike = !this.item.isLike ;
				console.log(isLike);
				this.$emit("update:isLike",isLike);
				this.$emit("update:like_count",like_count);
				
				this.likeTime = time;
				likeFun(this.item._id)
			},
			// 点击更多
			clikMore() {
				let uid = uniCloud.getCurrentUserInfo().uid;
				if (uid == this.item.user_id[0]._id || this.uniIDHasRole('admin') || this.uniIDHasRole('webmaster')) {
					this.list.forEach(item => {
						item.disabled = false;
					})
				}
				this.show = true
			},
			// 弹窗选择
			sheetSelect(e) {
				// console.log(e);
				this.show = false;
				let type = e.type;
				if (type == "del") {
					this.delFun()
				}
			},
			delFun() {
				uni.showLoading({
					title: "加载中。。。"
				})
				db.collection("quanzi_article").doc(this.item._id).update({
					delState: true
				}).then(res => {
					uni.hideLoading()
					uni.showToast({
						title: "删除成功",
						icon: "none"
					})
					this.$emit("delEvent",true)
				}).catch(err => {
					uni.hideLoading()
				})
			},


			// 关闭弹窗
			sheetClose() {
				this.show = false

			},
			getName,
			giveAvatar,
			clickPic(index) {
				uni.previewImage({
					urls: this.item.picurls,
					current: index
				})
			},
			// 跳转详情
			goDetail() {
				uni.navigateTo({
					url: "/pages/detail/detail?id=" + this.item._id
				})
			}
		}
	}
</script>

<style lang="scss">
	.blogitem {
		margin: 20rpx;

		.head {
			display: flex;
			justify-content: space-between;
			align-items: center;
			font-size: 32rpx;

			.userinfo {
				display: flex;
				align-items: center;

				.avator {
					width: 40rpx;
					height: 40rpx;
					overflow: hidden;
					border-radius: 50%;

					image {
						width: 100%;
						height: 100%;
						display: block;
					}
				}

				.name {
					color: #222;
					padding-left: 20rpx;
				}

				.time {
					margin-left: 20rpx;
					// padding-left: 20rpx;
					font-size: 22rpx;
					color: #bfbfbf;
				}
			}

			.more {
				padding: 5rpx;

				.iconfont {
					font-size: 50rpx;
					color: #888;
				}
			}
		}

		.body {
			padding: 15rpx 0 30rpx;

			.title {
				font-size: 44rpx;
				font-weight: 600;
				color: #000;
				text-align: justify;
			}

			.text {
				font-size: 30rpx;
				color: #888;
				text-align: justify;
				padding: 10rpx 0;

				.t {
					text-overflow: -o-ellipsis-lastline;
					overflow: hidden;
					text-overflow: ellipsis;
					display: -webkit-box;
					-webkit-line-clamp: 2;
					line-clamp: 2;
					-webkit-box-orient: vertical;
				}
			}

			.piclist {
				padding-top: 20rpx;
				display: flex;

				.pic {
					width: 225rpx;
					height: 225rpx;
					margin-right: 6rpx;
					overflow: hidden;

					image {
						width: 100%;
						height: 100%;
					}

					&:first-child {
						border-radius: 20rpx 0 0 20rpx;
					}

					&:last-child {
						border-radius: 0 20rpx 20rpx 0;
					}

					&.only {
						border-radius: 20rpx;
					}
				}
			}
		}

		.info {
			display: flex;
			justify-content: space-around;
			color: #333;
			font-size: 26rpx;
			align-items: center;

			.box {
				display: flex;
				align-items: center;
				justify-content: center;
				flex: 1;
				padding: 15rpx 0 5rpx;

				.iconfont {
					font-size: 40rpx;
					padding-right: 10rpx;
				}
				&.active{
					color: #0199fe;
				}
			}
		}
	}
</style>