<template>
  <view>
    <view class="uni-padding-wrap uni-common-mt">
		<div class="logo-box">
			<image class="logo" src="../../static/gowild_logo-1.png" ></image>
		</div>
		<view class="home-title">
			<text>欢迎使用狗尾草小鹦鹉</text><br>
			<text>感谢你的时间</text>
		</view>

		<view class="uni-btn-v">
			<button type="primary" open-type="getUserInfo" @getuserinfo="onGotUserInfo" @tap="openTape">进入录音</button>
			<button type="default" @tap="openHistory">历史数据</button>
			<!-- <button type="default" open-type="getUserInfo" @getuserinfo="onGotUserInfo">提供昵称</button> -->
		</view>
    </view>
  </view>
</template>

<script>
    export default {
        data() {
            return {
            }
        },
        onLoad() {
            let self = this
            wx.login({
                success(res) {
                    if (res.code) {
                        console.log('wechat_code:'+res.code)
                        uni.request({
                            url: 'https://asr-record.gowild.info/rmserver/login-in',
                            method: 'POST',
                            data: {
                                "code": res.code
                            },
                            success: sessionRes => {
								console.log('login resp:')
                                console.log(sessionRes.data)
								try {
								    uni.setStorageSync('sessionId', sessionRes.data.data[0].sessionId)
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
            openHistory() {
                uni.navigateTo({
                    url: '../history/history'
                })
            },
			onGotUserInfo(e) {		
				uni.request({
				    url: 'https://asr-record.gowild.info/rmserver/create-or-update-userinfo',
				    method: 'POST',
				    data: {
				        "nickname": e.detail.userInfo.nickName,
						"avatar_url":e.detail.userInfo.avatarUrl
				    },
					header:{
						"Gowild-SessionId":uni.getStorageSync('sessionId')
					},
				    success: sessionRes => {
						console.log('userinfo resp:')
				        console.log(sessionRes.data)
				    }
				})
			},
			openTape() {
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
									uni.navigateTo({
									    url: '../tape/tape'
									})
								} else {
									uni.showToast({
									    title: '真棒！所有句子都念完了',
									    duration: 1000,
										icon:'none'
									})
								}
							}
						}
					});
				} catch (e) {
				    console.log('get session id err')
				    console.log(e)
				}
			},
        }
    }
</script>

<style>
	.logo-box {
		text-align: center;
		margin-top: 60upx;
		margin-bottom: 20upx;
	}
	.logo {
		width: 500upx;
		height: 500upx;
	}
	.home-title {
		font-size:15px;
		text-align:center;
	}
</style>