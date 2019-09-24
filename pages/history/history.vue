<template>
	<view>
		<scroll-view class="nav" scroll-x="true" scroll-left="120">
			<view id="demo1" class="nav-item" v-for="(item,index) in tabs">{{item.title}}</view>
		</scroll-view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				tabs:[
					{title:'全部',type:'all',pageIndex:1},
					{title:'已通过',type:'pass',pageIndex:1},
					{title:'待审阅',type:'pending',pageIndex:1}
				],
			}
		},
		methods: {
			
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
</style>
