<template>
	<view class="box">
		<img class='image' src="./static/noVideo.jpeg" alt="" v-if="url==''">
		<video @play="videoLoad" id="myVideo" :src=url @error="videoErrorCallback" controls show-fullscreen-btn
			v-else></video>
		<view class="top">
			<input class="uni-input ipt" focus placeholder="请输入链接" v-model="preUrl" />
			<button type="default" size="mini" class="btn" @click="toPasing">解析</button>
			<button type="default" size="mini" class="btn" @click="clear">清空</button>
		</view>

		<!-- <view class="top bottom" v-if="url!=''">
			<button type="default" size="mini" class="btn" @click="copyUrl">复制链接</button>
			<button type="default" size="mini" class="btn" @click="downLoad">保存</button>

		</view> -->
		<view class="top bottom">
			<button type="default" size="mini" class="btn" @click="copyUrl">复制链接</button>
			<button type="default" size="mini" class="btn" @click="downloadE">保存</button>

		</view>
		<view class="">
			<progress :percent="pgList[0]" show-info stroke-width="3" v-show="showProgress" />
		</view>
		<view class="" style="padding: 0 5px">
			<span style="color:  #e858ff">
				支持以下平台去水印：抖音，bilibli,火山,快手,皮皮虾,tiktok,西瓜，bangumi
			</span>
			</br>
			<span style="color:  #ff0011">链接可直接复制粘贴，自动提取链接</span>
		</view>

	</view>
</template>
<script>
	import {
		fail
	} from "assert";
	var http = require("../../utils/http");
	var config = require("../../utils/config.js"); //统一的网络请求方法
	export default {
		data() {
			return {
				url: '',
				preUrl: '',
				percentCurrent: '',
				pgList: [0, 0, 0, 0],
				showProgress: false,

			};
		},
		onReady: function(res) {
			// #ifndef MP-ALIPAY
			this.videoContext = uni.createVideoContext('myVideo')
			// #endif
		},
		methods: {
			clear: function() {
				this.preUrl = ''
				this.url = ''
				this.showProgress = false
			},
			setProgress: function() {
				this.pgList = [20, 40, 60, 80]
			},
			clearProgress: function() {
				this.pgList = [0, 0, 0, 0]
			},
			videoLoad: function(e) {
				console.log(this.url)
			},
			downloadE: function() {
				let that = this
				var parsingUrl = this.preUrl.trim()

				var parsingUrlList = parsingUrl.split('http')
				var params = {
					url: "/video/download",
					method: "get",
					responseType: "arraybuffer",
					data: {
						// appType: 1,
						// 应用类型 1小程序 2微信公众号 3 PC 4 H5
						type: 'auto',
						url: 'http' + parsingUrlList[1]
					},
					callBack: rest => {
						if (rest.statusCode === 200) {
							const cd = rest.header['Content-Disposition'],
								filename = cd.match(/"(\S*)"/)[1]
							//获取全局唯一的文件管理器
							const fs = wx.getFileSystemManager()
							//wx.env.USER_DATA_PATH 指定临时文件存入的路径，后面字符串自定义
							const filePath = `${wx.env.USER_DATA_PATH}/${filename}`
							console.log('111')
							// 写文件
							fs.writeFile({
								filePath,
								data: rest.data,
								encoding: "binary", //二进制流文件必须是 binary
								success: (res) => {
									console.log('222')
									that.handleFile(filePath)
									console.log(111, 'down', res, filePath)
								}
							});
						}
					}

				};
				http.request(params);

			},
			// 对不同文件的处理
			handleFile(filePath) {
				const filetype = filePath.split('.')[1],
					typeObj = {
						gif: 'img',
						GIF: 'img',
						png: 'img',
						PNG: 'img',
						jpg: 'img',
						JPG: 'img',
						jpeg: 'img',
						mp4: 'video',
						doc: 'doc',
						docx: 'doc',
						xls: 'doc',
						xlsx: 'doc',
						ppt: 'doc',
						pptx: 'doc',
						pdf: 'doc'
					},
					result = (ok = '成功保存到相册', no = '保存失败') => {
						return {
							filePath,
							success: res => {
								uni.showToast({
									title: ok
								})
							},
							fail: err => {
								uni.showToast({
									title: no
								})
							}
						}
					}

				if (typeObj[filetype] === 'video') {
					// 保存视频到系统相册:mp4
					uni.saveVideoToPhotosAlbum(result())
				} else if (typeObj[filetype] === 'img') {
					// 保存图片到系统相册:gif,jpg,jpeg,png,GIF,JPG,PNG
					uni.saveImageToPhotosAlbum(result())
				} else {
					// 打开文件:doc,docx,xls,xlsx,ppt,pptx,pdf
					uni.openDocument({
						...result('打开文档成功', '打开文档失败'),
						showMenu: true //showMenu是否显示右上角菜单
					});
				}
			},
			downLoad: function() {
				// uni.showLoading({
				// 	title: '下载中'
				// })
				var parsingUrl = this.preUrl.trim()

				var parsingUrlList = parsingUrl.split('http')
				let url = 'http' + parsingUrlList[1]
				//下载视频
				const fileName = new Date().valueOf()
				const filePath = wx.env.USER_DATA_PATH + '/' + fileName + '.mp4'

				let that = this
				const downloadTask = uni.downloadFile({
					url: config.domain + "/video/download?type=auto&url=" + url, //视频路径
					// filePath: filePath,
					success: rese => {

						if (rese.statusCode == 200) {

							uni.saveVideoToPhotosAlbum({
								filePath: rese.tempFilePath, //注意这里是res 不是red
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
							// 	downloadTask.onProgressUpdate(red => {
							// 		let that = this;
							// 		that.percentCurrent = red.progress;

							// 		console.log(that.percentCurrent);
							// 		if (red.progress === 100) {
							// 			uni.showToast({
							// 				title: '下载完成！',
							// 				icon: 'success',
							// 				duration: 2000,
							// 				position: 'top'
							// 			});
							// 			//当进程全部下载完成之后 在执行保存到相册的回调
							// 			uni.saveImageToPhotosAlbum({
							// 				filePath: res.tempFilePath, //注意这里是res 不是red
							// 				success: () => {
							// 					uni.showToast({
							// 						title: '下载成功',
							// 						icon: 'none'
							// 					});
							// 				},
							// 				fail: () => {
							// 					uni.showToast({
							// 						title: '保存失败，请稍后重试',
							// 						icon: 'none'
							// 					});
							// 				}
							// 			});
							// 		}
							// 	});

						}
					},
					fail: (err) => {
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
				console.log(this)
				var parsingUrl = this.preUrl.trim()
				console.log(parsingUrl)
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
						this.url = res.data
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
	.box {
		width: 100%;
		background-color: rgb(240, 240, 240);
		min-height: 100%;

		.image {
			height: 200px;
		}

		#myVideo {
			width: 100%;
			height: 200px;

		}

		.btn {
			background-color: #2b82f5;
			color: #ffffff;
			// padding: 0 0;
			margin: 0;
		}

		.top {
			background-color: #ffffff;
			padding: 5px;
			margin-bottom: 10px;
			display: flex;
			justify-content: space-around;
			align-items: center;
			box-shadow: 0px 5px #eaf6f9;

			.ipt {
				width: 50%;
				height: 30px;
				font-size: 13px;
				border-radius: 5px;
				background-color: rgb(240, 240, 240);
				// border: 1px solid #0f63f0;

			}

		}

		.bottom {
			// display: flex;
			// justify-content: flex-end;
			// align-items: center;
		}
	}
</style>
