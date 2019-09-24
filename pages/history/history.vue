<template>
	<view>
		<scroll-view class="nav" scroll-x="true" scroll-left="120">
			<view id="demo1" class="nav-item" v-bind:class="{active:curTabIndex===index}"
			 @click="changeTab(index)"
			 v-for="(item,index) in tabs">{{item.title}}</view>
		</scroll-view>
		
		<scroll-view :scroll-top="scrollTop" scroll-y="true" class="scroll-Y" lower-threshold=1
		 @scroll="scroll">
			<uni-list-item v-bind:title="item.content" v-for="item in list" @click="openDetail(item.record_id)"></uni-list-item>
		</scroll-view>
	</view>
</template>

<script>
	import uniList from '../../components/uni-list/uni-list.vue'
	import uniListItem from '@/components/uni-list-item/uni-list-item.vue'
	export default {
	    components: {uniList,uniListItem},
		data() {
			return {
				sessionId:'',
				curTabIndex:1,
				page_size:15,
				tabs:[
					{title:'全部',type:'all',pageIndex:1},
					{title:'已通过',type:'pass',pageIndex:1},
					{title:'待审阅',type:'pending',pageIndex:1}
				],
				list:[],
				scrollTop: 0,
				old: {
					scrollTop: 0
				}
			}
		},
		onLoad() {
			let page = this.getList(this.curTabIndex,false)
		},
		methods: {
			changeTab(tabIndex){
				this.curTabIndex = tabIndex
				this.getList(tabIndex,false)
			},
			getList(tabIndex,more){	
				let self = this
				let tab = self.tabs[tabIndex]
				let page_index = tab.pageIndex
				console.log(tab.type+' '+more+' '+page_index)
				if (more) {
					page_index = tab.pageIndex+1
				}
				
				uni.request({
					// url: 'http://172.27.1.207:8009/rmserver/get-history-records',
					url: 'http://localhost:9097/rmserver/get-history-records',
					method: 'POST',
					data: {
						 "query_type": tab.type,
						 "count": self.page_size,
						 "cur_page":page_index
					},
					header:{
						"Gowild-SessionId":uni.getStorageSync('sessionId')
					},
					success: res => {
						self.curTabIndex = tabIndex
						self.tabs[self.curTabIndex].pageIndex = page_index
						if (more) {
							self.list = self.list.concat(res.data.list)
						} else {
							self.list = res.data.list
						}
						console.log(self.list.length)
					},
					fail: () => {},
					complete: () => {}
				});
			},
			scroll: function(e) {
				// console.log(e)
				this.old.scrollTop = e.detail.scrollTop
				console.log('getList')
				this.getList(this.curTabIndex,true)
			}
			,openDetail(audioid){
				uni.navigateTo({
					url: '../detail/detail?audioId='+audioid
				})
			},
			upper: function(e) {
				console.log('top')
				console.log(e)
			},
			lower: function(e) {
				console.log('end')
				console.log(e)
			},
			goTop: function(e) {
				console.log(e)
				this.scrollTop = this.old.scrollTop
				this.$nextTick(function() {
					this.scrollTop = 0
				});
				uni.showToast({
					icon:"none",
					title:"纵向滚动 scrollTop 值已被修改为 0"
				})
			}
		}
	}
</script>

<style>
	.nav {
	    height: 80rpx;
	    width: 100%;
	    box-sizing: border-box;
	    overflow: hidden;
	    line-height: 80rpx;
	    background: #f7f7f7;
	    font-size: 16px;
	    white-space: nowrap;
	    position: fixed;
	    top: 0;
	    left: 0;
	    z-index: 99;
	}
	.nav-item {
	    width: 20%;
	    display: inline-block;
	    text-align: center;
	}
	.nav-item.active{
	    color: green;
	}
	.scroll-Y {
		margin-top:80upx ;
		height: 1100upx;
	}
</style>
