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
				<icon type="success" size="40"/>
				<view>点饭</view>
			</view>
			<view @click="onAccounts">
				<icon type="info" size="40"/>
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
				console.log(2)
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
		width: 600rpx;
		margin: 60rpx auto 0;
	}

	.swiper {
		height: 300rpx;
	}

	.swiper-item{
		width: 600rpx;
		height: 300rpx;
	}

	.text-area {
		width: 300rpx;
		margin: 30rpx 0;
		display: flex;
		text-align: center;
		justify-content: space-between;
	}

	.title {
		font-size: 36rpx;
		color: #8f8f94;
	}
</style>
