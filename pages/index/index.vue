<template>
	<view class="content">
		<view class="swiper-box">
			<swiper class="swiper"
					:indicator-dots="true"
					:autoplay="true"
					:interval="2000"
					:duration="500">
				<swiper-item>
					<image class="swiper-item" src="../../static/img1.png" />
				</swiper-item>
				<swiper-item>
					<image class="swiper-item" src="../../static/img2.png" />
				</swiper-item>
				<swiper-item>
					<image class="swiper-item" src="../../static/img3.png" />
				</swiper-item>
			</swiper>
		</view>
		<view class="text-area">
			<view @click="onlucky">
				<image class="accounts-img" src="../../static/img5.png" />
				<view>点饭</view>
			</view>
			<view @click="onAccounts">
				<image class="accounts-img" src="../../static/img4.jpg" />
				<view>记账</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				title: 'Hello'
			}
		},
		onLoad() {

		},
		methods: {
			onlucky(e) {
				wx.navigateToMiniProgram({
					appId: 'wx055fd9c35ed854c5',// 要跳转的小程序的appid
					path: 'pages/v3/v3',// 跳转的目标页面
					envVersion: 'release',
					success(res) {
					// 打开成功
					}
				})
			},
			onAccounts(e) {
				uni.navigateTo({
					url: '/pages/accounts/index'
				});
			}
		}
	}
</script>

<style>
	.content {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}

	.swiper-box {
		width: 100%;
		margin: 60rpx auto 0;
	}

	.swiper {
		height: 400rpx;
	}

	.swiper-item{
		width: 750rpx;
		height: 400rpx;
	}

	.text-area {
		width: 400rpx;
		margin: 30rpx 0;
		font-size: 28rpx;
		display: flex;
		text-align: center;
		justify-content: space-between;
	}

	.title {
		font-size: 36rpx;
		color: #8f8f94;
	}

	.accounts-img {
		width: 140rpx;
		height: 140rpx;
		border-radius: 50%;
	}
</style>
