<template>
	
	<view>
		<view class="uni-padding-wrap uni-common-mt">
			<view class="uni-flex uni-column">
				<view class="flex-item flex-item-V ">
					<scroll-view class="nav" scroll-x="true">
						<view id="demo1" class="nav-item" v-bind:class="{active:curTabIndex===index}"
						 @click="changeTab(index)"
						 v-for="(item,index) in tabs">{{item.title}}</view>
					</scroll-view>
				</view>
				<view class="flex-item flex-item-V">
					<scroll-view :scroll-top="scrollTop" scroll-y="true" class="scroll-Y" lower-threshold=20 show-scrollbar=true
					 @scroll="scroll" @scrolltoupper="upper" @scrolltolower="lower">
						<uni-list-item class="list-item" v-bind:title="item.content" v-for="item in tabs[curTabIndex].list" @click="openDetail(item.record_id)"></uni-list-item>
					</scroll-view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	import uniList from '../../components/uni-list/uni-list.vue'
	import uniListItem from '@/components/uni-list-item/uni-list-item.vue'
	export default {
	    components: {uniList,uniListItem},
		data() {
			return {
				scrollLeft:100,
				curTabIndex:0,
				page_size:15,
				tabs:[
					{title:'全部',type:'all',pageIndex:1,hasMore:true,list:[]},
					{title:'已通过',type:'pass',pageIndex:1,hasMore:true,list:[]},
					{title:'待审阅',type:'pending',pageIndex:1,hasMore:true,list:[]}
				],
				scrollTop: 0,
			}
		},
		onShow() {
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
				if (!tab.hasMore) {
					return
				}
				let page_index = tab.pageIndex
				console.log(tab.type+' '+more+' '+page_index)
				if (more) {
					page_index = tab.pageIndex+1
				}
				
				uni.request({
					url: 'http://172.27.1.207:8009/rmserver/get-history-records',
					// url: 'http://172.19.1.231:9097/rmserver/get-history-records',
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
						let list_result = res.data.data
						console.log(list_result)
						if (list_result.length<self.page_size){
							self.tabs[self.curTabIndex].hasMore = false
						}
						if (list_result[0]) {
							self.tabs[self.curTabIndex].list = self.tabs[self.curTabIndex].list.concat(list_result)
							
						} else {
							if (!more) {
								self.tabs[self.curTabIndex].list = []
							} 
						}
						
						// console.log(self.list)
						console.log(self.tabs[self.curTabIndex].list.length)
					},
					fail: () => {},
					complete: () => {}
				});
			},
			scroll: function(e) {
				// console.log(e)
				this.scrollTop = e.detail.scrollTop
			}
			,openDetail(record_id){
				uni.navigateTo({
					url: '../detail/detail?record_id='+record_id
				})
			},
			upper: function(e) {
				console.log('top')
				console.log(e)
				
				this.tabs[this.curTabIndex].pageIndex = 1
				this.getList(this.curTabIndex,false)
			},
			lower: function(e) {
				console.log('end')
				console.log(e)
				
				this.getList(this.curTabIndex,true)
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
	.page {
		background: #E5E5E5;
	}
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
	    width: 33.34%;
	    display: inline-block;
	    text-align: center;
	}
	.nav-item.active{
	    color: green;
	}
	.scroll-Y {
		margin-top:50upx ;
		height: 1000upx;
		background: #FFFFFF;
	}
</style>
