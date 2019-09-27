<template>
	<view>
		<view class="uni-padding-wrap uni-common-mt" v-if="!hasSentence">
			<view class="home-title">
				<text>没有需要录音的句子啦</text>
			</view>
		</view>
		<view class="uni-padding-wrap uni-common-mt" v-if="hasSentence">
			<view class="home-title">
				<text>按住录音按钮，读出以下句子。</text><br>
				<text>读完后松开按钮</text>
			</view>

			<view class="sentence uni-bold">
				<text>{{cur_content}}</text>
			</view>
			<br>
			
			<view class="uni-center history-sentence">
				<p v-for="item in list">{{item}}</p>
				<p>...</p>
			</view>
			
			<view class="uni-btn-v">
				<button type="primary" @touchstart="startRecord" @touchend="endRecord">{{voicePath ===''? '按住说话' : '重新录音'}}</button>
				<!-- <button @tap="startRecord" type="primary" v-show="!isRecord">点击录音</button>
				<button @tap="endRecord" type="primary" v-show="isRecord">停止录音</button> -->
				<button @tap="playVoice" v-show="voicePath!=''">播放录音</button>
				<button @tap="getSentence" v-show="voicePath!=''">下一句</button>
				<button type="default" @tap="openHistory">历史数据</button>
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
				hasSentence:false,
				isRecord:false,
				cur_content: '',
				list: [],
				voicePath: '',
				sentence_id:null
			}
		},
		onLoad() {
			this.getSentence()
			let self = this;
			recorderManager.onStop(function(res) {
				console.log('recorder stop' + JSON.stringify(res));
				self.voicePath = res.tempFilePath;
				self.upload2Qiniu(self.voicePath)
			})
			
			innerAudioContext.onEnded(function(res){
				uni.hideToast();
			})
		},
		methods: {
			openHistory() {
			    uni.navigateTo({
			        url: '../history/history'
			    })
			},
			startRecord() {
				console.log('开始录音');
				this.isRecord = !this.isRecord
				uni.showToast({
				    title: '录音中...',
				    duration: 100000000,
					icon:'none',
					image:'../../static/record.png'
				})
				recorderManager.start();
			},
			endRecord() {
				console.log('录音结束');
				this.isRecord = !this.isRecord
				recorderManager.stop();
				uni.hideToast();
			},
			playVoice() {
				console.log('播放录音');
				if (this.voicePath) {
					innerAudioContext.src = this.voicePath;
					uni.showToast({
					    title: '播放...',
					    duration: 100000000,
						icon:'none',
						image:'../../static/play.png'
					})
					innerAudioContext.play();
				}
			},
			getSentence() {
				try {
					console.log(uni.getStorageSync('sessionId'))
					uni.request({
						url: 'https://asr-record.gowild.info/rmserver/get-sentence',
						method: 'POST',
						data: {
							"count": 4
						},
						header: {
							"Gowild-SessionId": uni.getStorageSync('sessionId')
						},
						success: res => {
							if (res.data && res.data.data[0]) {
								let d = res.data.data[0]
								if (d.sentence_id > 0) {
									this.hasSentence = true
									console.log(d)
									this.cur_content = d.cur_content
									this.list = d.history_contents
									this.sentence_id = d.sentence_id
									this.voicePath = ''
								} else {
									this.hasSentence = false
									uni.showToast({
									    title: '真棒！所有句子都念完了',
									    duration: 1000,
										icon:'none'
									})
									uni.redirectTo({
									    url: '../home/home'
									});
								}
							}
						}
					});
				} catch (e) {
				    console.log('get session id err')
				    console.log(e)
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
					url: 'https://asr-record.gowild.info/rmserver/get-upload-token',
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
						            url: 'https://up-z2.qiniup.com/',
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
											url: 'https://asr-record.gowild.info/rmserver/create-or-update-record',
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
											}
										});
						            },
									fail() {
										console.log('upload fail')
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
	.home-title {
		font-size:14px;
		text-align:center;
	}
	.sentence {
		margin: 10px;
		font-size:28px;
		text-align:center;
	}
	.history-sentence p {
		font-size:12px;
	}
</style>
