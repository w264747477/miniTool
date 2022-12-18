<template>
	<view>
		fffff
		<input class="uni-input" focus placeholder="自动获得焦点" />
		<view class="uni-list">
					<view class="uni-list-cell">
						<view class="uni-list-cell-left">
							当前选择
						</view>
						<view class="uni-list-cell-db">
							<picker @change="bindPickerChange" :value="index" :range="array">
								<view class="uni-input">{{array[index]}}</view>
							</picker>
						</view>
					</view>
				</view>
				<button type="default" @click="goto('/pages/about/about')">下载</button>
				<button type="default" @click="goto('/pages/about/about')">复制链接</button>
				</view>
				
	</view>
</template>
<script>
	export default {
		data() {
			return {
				
			};
		}
	}
</script>

<style lang="scss">

</style>
