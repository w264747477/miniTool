<template>
	<view>
		<view class="top">
			<input class="uni-input" focus placeholder="请输入链接" v-model="preUrl" />
			<button type="default" @click="toPasing">解析</button>
		</view>
		<video id="myVideo" :src=url @error="videoErrorCallback" controls></video>
		<view class="button">
			<button type="default" @click="copyUrl">复制链接</button>
			<button type="default" @click="downLoad">保存</button>

		</view>
		<view class="">
			进度:{{percentCurrent}}
		</view>

	</view>
</template>
<script>
import { fail } from "assert";
	var http = require("../../utils/http");
	export default {
		data() {
			return {
				url: '',
				preUrl: '',
				percentCurrent: ''

			};
		},
		onReady: function(res) {
			// #ifndef MP-ALIPAY
			this.videoContext = uni.createVideoContext('myVideo')
			// #endif
		},
		methods: {
			downLoad: function() {
				// uni.showLoading({
				// 	title: '下载中'
				// })
				
				//下载视频
					const fileName = new Date().valueOf()
				    const filePath = wx.env.USER_DATA_PATH + '/' + fileName + '.mp4'
				
					const downloadTask = uni.downloadFile({
						url: this.url, //视频路径
						filePath: filePath,
						success: res => {
							console.log(res)
							if (res.statusCode == 200) {
								console.log('1')
								uni.saveVideoToPhotosAlbum({
									filePath: filePath, //注意这里是res 不是red
									success: () => {
										uni.showToast({
											title: '下载成功',
											icon: 'none'
										});
									},
									fail: (err) => {
										// uni.hideLoading()
										console.log(err)
										uni.showToast({
											title: '保存失败，请稍后重试',
											icon: 'none'
										});
									}
								});
								//下载的回调 不要都用res  要换一个使用 方便区分
								downloadTask.onProgressUpdate(red => {
									let that = this;
									that.percentCurrent = red.progress;

									console.log(that.percentCurrent);
									if (red.progress === 100) {
										uni.showToast({
											title: '下载完成！',
											icon: 'success',
											duration: 2000,
											position: 'top'
										});
										//当进程全部下载完成之后 在执行保存到相册的回调
										uni.saveImageToPhotosAlbum({
											filePath: res.tempFilePath, //注意这里是res 不是red
											success: () => {
												uni.showToast({
													title: '下载成功',
													icon: 'none'
												});
											},
											fail: () => {
												uni.showToast({
													title: '保存失败，请稍后重试',
													icon: 'none'
												});
											}
										});
									}
								});
							}
						},
					    fail:(err)=> {
					    	console.log(err)
					    }
					});
			},
			copyUrl: function() {
				uni.setClipboardData({
					data: this.url,
					success: function() {
						console.log('已复制到剪贴板');
					}
				});
			},
			toPasing: function() {
				var parsingUrl = this.preUrl.trim()
				var parsingUrlList = parsingUrl.split('http')
				console.log('http' + parsingUrlList[1])
				var params = {
					url: "/video/fetch",
					method: "get",
					data: {
						// appType: 1,
						// 应用类型 1小程序 2微信公众号 3 PC 4 H5
						type: 'auto',
						url: 'http' + parsingUrlList[1]
					},
					callBack: res => {
						this.url = res
						console.log(res)
						// wx.setStorageSync('loginResult', res);
						// wx.setStorageSync('token', 'bearer' + res.access_token);
					}
				};
				http.request(params);
			},

			videoErrorCallback: function(e) {
				console.log(e)
				uni.showModal({
					content: e.target.errMsg,
					showCancel: false
				})
			},
			getRandomColor: function() {
				const rgb = []
				for (let i = 0; i < 3; ++i) {
					let color = Math.floor(Math.random() * 256).toString(16)
					color = color.length == 1 ? '0' + color : color
					rgb.push(color)
				}
				return '#' + rgb.join('')
			}
		}
	}
</script>

<style lang="scss">
	.top {
		// display: flex;

	}

	.button {
		display: flex;
	}
</style>
