<template>
	<!-- v7.3.8 -->
	<view class="container">
		<view class="top_title">
			<text class="iconfont icon-more" @click="clean"></text>
			<view class="top_center" :class="{ 'top_center_active': index == tabCur }" v-for="(item,index) in tabList" :key="index" @click="clickCtTab(index)">
				<text v-text="item.title"></text>
			</view>
			<text class="iconfont icon-search" @click="search"></text>
		</view>
		<!-- 内容区 -->
		<view class="content" v-show="tabCur === 0">
			<my></my>
		</view>
		<view class="content" v-show="tabCur === 1">
			<banner :swipers="bannerList"></banner>
			<redIcon></redIcon>
			<recommend :poster="recommendList"></recommend>
			<chart :chart="chartList"></chart>
			<broad :broad="broadList"></broad>
		</view>
		<musicBottom></musicBottom>
	</view>
</template>

<script>
import banner from 'components/findComponents/banner/banner.vue'
import redIcon from 'components/findComponents/redIcon/redIcon.vue'
import recommend from 'components/findComponents/recommend/recommend.vue'
import chart from 'components/findComponents/chart/chart.vue'
import broad from 'components/findComponents/broad/broad.vue'
import my from '../my/my.vue'
import musicBottom from 'pages/music/musicBottom/musicBottom'

import request from 'utils/request.js'
const appGlobalData = getApp().globalData;
	export default {
		data() {
			return {
			    tabCur:1,
			    tabList:[
					{
						title:'我的',
					},
					{
						title:'发现',
					}
				],
				bannerList:[],//轮播图
				recommendList:[],//推荐歌单
				chartList:[],//排行榜
				broadList:[],//电台
			}
		},
		onLoad(){
			this.getbannerList()
			this.getrecommendList()
			this.getchartListId()
			this.getbroadList()
		},
		methods: {
			async getbannerList(){
				let bannerListData = await request('/banner', {type: 1});//轮播图
				// type---->0: pc,1: android,2: iphone,3: ipad
				this.bannerList = bannerListData.banners
			},
			async getrecommendList(){
				let recommendListData = await request('/personalized', {limit: 10});//推荐歌单
				this.recommendList = recommendListData.result
			},
			async getchartListId(){
				let chartListIdData = await request('/toplist/detail');
				if(chartListIdData){
					chartListIdData = chartListIdData.list.sort(() => {
					        return 0.5 - Math.random();
					}).slice(0,5);
					let chartListId = chartListIdData.map(item => {
						return item.id
					})
					let arr =[]
					chartListId.map(item => this.handlechartList(item,arr))
				}
			},
			async handlechartList(item,arr){
				let chartListData = await request('/playlist/detail', {id: item});
				arr.push(chartListData.playlist)
				this.chartList = arr.map((chart,index) => {
					chart['background'] = 'chart_swiper_view' + index
					chart.tracks = [...new Set(chart.tracks)].slice(0,3)
					appGlobalData.musicIdList = chart.tracks.map(a => {
						return a.id
					})
					return chart
				})
			},
			async getbroadList(){
				let broadListData1 = await request('/dj/personalize/recommend');//推荐电台
				let broadListData2 = await request('/dj/radio/hot',{limit:6,cateId:2001});//热门电台
				this.broadList.push(broadListData1.data,[],broadListData2.djRadios.slice(0,6))
			},
		    clickCtTab(ctCur){
		        this.tabCur = ctCur
		    },
			clean(){
				plus.cache.calculate( size => { //size是多少个字节单位是b
					//你可以做下面相应的处理
					if(size<1024){
						this.xxx=size+'B'; 
					 }  
					else if(size/1024>=1 && size/1024/1024<1){
						this.xxx= Math.floor(size/1024*100)/100+'KB';
					}
					else if(size/1024/1024>=1){
						this.xxx=Math.floor(size/1024/1024*100)/100+'M';
					}
					//可以询问用户是否删除
			     	uni.showModal({
						title:'提示',
						content:'确定清除缓存吗?',
						success(res) {
							if(res.confirm){
								//使用plus.cache.clear 清除应用中的缓存数据 这里清除后还要二十几KB没有清除，达不到全部清除
								plus.cache.clear( function () {
									uni.showToast({
										title:'清除成功',
										icon:'none',
									})
								});	
							}
						}
					})
				});
			},
			search(){
				uni.navigateTo({
					url: '/pages/search/search'
				})
			}
		},
		components: {
			banner,
			redIcon,
			recommend,
			chart,
			broad,
			my,
			musicBottom
		}
	}
</script>

<style>
	.container{
		padding-left: 0.5rem;
		padding-right: 0.5rem;
	}
	.top_title{
		position: fixed;
		top: 0;
		z-index: 9999;
		width: 96%;
		display: flex;
		align-items: center;
		line-height: 2.5rem;
		background-color: #FFFFFF;
		padding-top: 66rpx;
	}
	.top_title .iconfont{
		font-size: 1.5rem;
	}
	.top_center{
		width: 100%;
	    display: flex;
	    justify-content: space-around;
		font-weight: 100;
		color: #D2D2D2;
	}
	.top_center_active{
	    color:#000000;
		font-size: 1.2rem;
		font-weight: 500;
	}
	.content{
		padding-top: 2.4rem;
	}
</style>