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
			<button type="primary" @tap="openTape">开始录音</button>
			<button type="default" @tap="openHistory">历史数据</button>
			<button type="default" open-type="getUserInfo" @getuserinfo="onGotUserInfo">提供昵称</button>
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
                            url: 'http://172.27.1.207:8009/rmserver/login-in',
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
			onGotUserInfo(e) {
			    console.log(e.detail.errMsg)
			    console.log(e.detail.userInfo)
			    console.log(e.detail.rawData)
				
				uni.request({
				    url: 'http://172.27.1.207:8009/rmserver/create-or-update-userinfo',
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