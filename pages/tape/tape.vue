<template>
	<view>
		<view class="uni-padding-wrap uni-common-mt">
			<view class="uni-title uni-common-mt">
				<p>请将手机靠近嘴巴，按下绿色按钮，独处以下句子。</p>
				<p>读完后松开按钮</p>
			</view>

			<h1>{{cur_content}}</h1>
			<br>
			<uni-list>
				<uni-list-item v-bind:title="item" v-for="item in list" showArrow="false"></uni-list-item>
			</uni-list>

			<view class="uni-btn-v">
				<button @tap="startRecord" type="primary">开始录音</button>
				<button @tap="endRecord">停止录音</button>
				<button @tap="playVoice">播放录音</button>
			</view>
		</view>
	</view>
</template>

<script>
	import uniList from '../../components/uni-list/uni-list.vue'
	import uniListItem from '@/components/uni-list-item/uni-list-item.vue'
	import SparkMD5 from 'spark-md5'
	
	const recorderManager = uni.getRecorderManager();
	const innerAudioContext = uni.createInnerAudioContext();
	innerAudioContext.autoplay = true;
	export default {
		components: {
			uniList,
			uniListItem
		},
		data() {
			return {
				cur_content: '',
				list: [],
				text: 'uni-app',
				voicePath: '',
				sessionId:'',
				sentence_id:null
			}
		},
		onLoad() {
			try {
				console.log(uni.getStorageSync('sessionId'))
				uni.request({
					url: 'http://172.27.1.207:8009/rmserver/get-sentence',
					method: 'POST',
					data: {
						"count": 10
					},
					header: {
						"Gowild-SessionId": uni.getStorageSync('sessionId')
					},
					success: res => {
						let d = res.data.data[0]
						console.log(d)
						this.cur_content = d.cur_content
						this.list = d.history_contents
						this.sentence_id = d.sentence_id
					},
					fail: () => {},
					complete: () => {}
				});
			} catch (e) {
			    console.log('get session id err')
			    console.log(e)
			}

			let self = this;
			recorderManager.onStop(function(res) {
				console.log('recorder stop' + JSON.stringify(res));
				self.voicePath = res.tempFilePath;
				self.upload2Qiniu(self.voicePath)
			})
		},
		methods: {
			startRecord() {
				console.log('开始录音');
				recorderManager.start();
			},
			endRecord() {
				console.log('录音结束');
				recorderManager.stop();
			},
			playVoice() {
				console.log('播放录音');
				if (this.voicePath) {
					innerAudioContext.src = this.voicePath;
					innerAudioContext.play();
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
						
					},
					fail: () => {},
					complete: () => {}
				});
			}
		}
	}
</script>

<style>

</style>
