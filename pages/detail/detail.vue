<template>
	<view>
		<view class="uni-padding-wrap uni-common-mt">
			<view class="page-section page-section-gap" style="text-align: center;">
				<audio style="text-align: left" :src="record.audio_url" poster="../../static/gowild_logo-1.png" :name="record.content" author="" :action="audioAction" controls></audio>
			 </view>
			 <text v-show="isPass">已通过</text><br>
			 <text v-show="!isPass">未通过</text>
			<view v-if="!isPass" class="uni-btn-v">
				<button @tap="startRecord" type="primary" v-show="!isRecord">重新录音</button>
				<button @tap="endRecord" type="primary" v-show="isRecord">停止录音</button>
				<button @tap="playVoice" v-show="voicePath!=''">播放录音</button>
				<button @tap="replaceVoice" v-show="voicePath!=''">替换录音</button>
			</view>
		</view>
	</view>
</template>

<script>
	const recorderManager = uni.getRecorderManager()
	const innerAudioContext = uni.createInnerAudioContext()
	innerAudioContext.autoplay = true
	
	export default {
		onLoad(p) {
			this.sentence_id = p.sentence_id
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
					this.record.record_id = p.record_id
					this.isPass = this.record.is_pass === 1
					console.log(res.data)
				}
			})
			
			let self = this;
			recorderManager.onStop(function(res) {
				console.log('recorder stop' + JSON.stringify(res));
				self.voicePath = res.tempFilePath;
			})
		},
		data() {
			return {
				audioAction: {
				    method: 'pause'
				},
				isPass:false,
				voicePath: '',
				isRecord:false,
				sentence_id:null,
				record:null
			}
		},
		methods: {
			startRecord() {
				console.log('开始录音');
				this.isRecord = !this.isRecord
				recorderManager.start();
			},
			endRecord() {
				console.log('录音结束');
				this.isRecord = !this.isRecord
				recorderManager.stop();
			},
			playVoice() {
				console.log('播放录音');
				if (this.voicePath) {
					innerAudioContext.src = this.voicePath;
					innerAudioContext.play();
				}
			},
			replaceVoice() {
				self.upload2Qiniu(self.voicePath)
				if (this.record) {
					this.record.audio_url = self.voicePath
				}
			},
			upload2Qiniu(recordFilePath){
				let self = this
				let spark = new SparkMD5()
				spark.append(recordFilePath)
				let fileMd5=spark.end()
				let fileName = fileMd5+recordFilePath.substring(recordFilePath.lastIndexOf('.'))
				console.log(fileName)
				uni.request({
					url: 'http://172.27.1.207:8009/rmserver/get-upload-token',
					method: 'POST',
					data:{
						"file_name":fileName
					},
					header:{
						"Gowild-SessionId":uni.getStorageSync('sessionId')
					},
					success: res => {
						console.log(res.data)
			
						uni.uploadFile({
						            url: 'http://up-z2.qiniup.com/',
						            filePath: recordFilePath,
						            name: 'file',
						            formData: {
						                'token': res.data.token,
										'key':fileName
						            },
						            success: (uploadFileRes) => {
										console.log(uploadFileRes.data)
										let audit_url = 'http://py9u16ymf.bkt.clouddn.com/'+JSON.parse(uploadFileRes.data).key
						                console.log(audit_url)
										
										uni.request({
											url: 'http://172.27.1.207:8009/rmserver/create-or-update-record',
											method: 'POST',
											data: {
												 "audio_url": audit_url,
												 "sentence_id": self.sentence_id
											},
											header:{
												"Gowild-SessionId":uni.getStorageSync('sessionId')
											},
											success: res => {
												console.log(res)
											},
											fail: () => {},
											complete: () => {}
										});
						            }
						        });
						
					}
				});
			}
		}
	}
</script>

<style>

</style>
