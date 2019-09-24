<template>
	<view>
		<view class="uni-padding-wrap uni-common-mt">
			<view class="page-section page-section-gap" style="text-align: center;">
				<audio style="text-align: left" :src="record.audio_url" poster="../../static/gowild_logo-1.png" :name="record.content" author="" :action="audioAction" controls></audio>
			 </view>
			<view class="uni-flex uni-column">
				<button type="default" @click="openTape(this.record.record_id)">重新录音</button>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		onLoad(p) {
			uni.request({
				url: 'http://172.27.1.207:8009/rmserver/detail-record',
				method:'POST',
				data:{
					"record_id":p.record_id
				},
				header:{
					"Gowild-SessionId":uni.getStorageSync('sessionId')
				},
				success:res=>{
					this.record = res.data.data[0]
					this.record.record_id = e.record_id
					console.log(res.data)
				},
				fail: () => {
					
				},
				complete: () => {
					
				}
			})
		},
		data() {
			return {
				audioAction: {
				    method: 'pause'
				},
				record:{
					
				}
			}
		},
		methods: {
			openTape(record_id){
				console.log(record_id)
				uni.navigateTo({
					url: '../tape/tape?record_id='+record_id
				})
			},
		}
	}
</script>

<style>

</style>
