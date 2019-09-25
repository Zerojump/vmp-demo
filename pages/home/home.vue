<template>
  <view>
    <view class="uni-padding-wrap uni-common-mt">
      <image src="../../static/gowild_logo-1.png"></image>
      
	  <view class="text-box" scroll-y="true">
		  <text>欢迎使用狗尾草小鹦鹉</text><br>
		  <text>感谢你的时间</text>
	  </view>

      <view class="uni-btn-v">
        <button type="primary" @tap="openTape">开始录音</button>
        <button type="default" @tap="openHistory">历史数据</button>
		<button type="default" @tap="openHistory2">历史数据2</button>
      </view>
    </view>
  </view>
</template>

<script>
    export default {
        data() {
            return {
				sessionId:'tourist'
            }
        },
        onLoad() {
            let self = this
            wx.login({
                success(res) {
                    if (res.code) {
                        console.log('wechat_code:'+res.code)
                        uni.request({
                            url: 'http://172.27.1.207:8009/rmserver/login-in',
                            method: 'POST',
                            data: {
                                "code": res.code
                            },
                            header: {
                                "Gowild-SessionId": "tourist"
                            },
                            success: sessionRes => {
								console.log('login resp:')
                                console.log(sessionRes.data)
								try {
								    uni.setStorageSync('sessionId', sessionRes.data.data[0].sessionId);
								} catch (e) {
									console.log('set session id err')
								    console.log(e)
								}
                            },
                            fail: () => {

                            },
                            complete: () => {

                            }
                        })
                    } else {
                        console.log('登录失败！' + res.errMsg)
                    }
                }
            })
        },
        methods: {
            openTape() {
                uni.navigateTo({
                    url: '../tape/tape'
                })
            },
            openHistory() {
                uni.navigateTo({
                    url: '../history/history'
                })
            },
            openHistory2() {
                uni.navigateTo({
                    url: '../history/history2'
                })
            }
        }
    }
</script>

<style>

</style>