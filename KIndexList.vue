<template>
	<view class="KIndexList-container">
		<scroll-view :scroll-y="true" style="height: 100%;" :scroll-into-view="cIndex" @scroll="scroll">
			<view v-for="(item, index) in KIndexData" :key="index">
				<view :class="{'item-label': true, 'item-label-active': activeIndex === item.label}" :id="item.label">
					{{item.label}}
				</view>
				<view class="item-name" v-for="(item2, index2) in item.children" :key="index2" @click="itemClick(item2)">
					{{item2.city}}
				</view>
			</view>
		</scroll-view>
		<view class="tbr-index-box" @touchstart="touchStart" @touchmove="touchMove" @touchend="touchEnd" @touchmove.stop.prevent="">
			<view :class="{'index-item': true, 'index-item-active': activeIndex === item}" 
				v-for="(item, index) in barTitle"  
				:key="index" 
				:id="item" 
				@click="indexedClick(item)">
				{{item}}
			</view>
		</view>
		<view class="bar-target-index" v-if="targetIndexShow">
			{{activeIndex}}
		</view>
	</view>
</template>

<script>
	export default {
		props: {
			KIndexData: {
				type: Array,
				default: () => []
			},
			KIndexBar: {
				type: Array,
				default: () => []
			}
		},
		data() {
			return {
				targetIndexShow: false,
				barTitle: [], // 索引列表
				cityTopList: [], // 列表top值
				indexTopList: [], // 索引top值
				activeIndex: "", // 索引位置
				cIndex: "" // 滚动条锚点位置
			};
		},
		mounted() {
			let _this = this
			if (_this.KIndexBar && _this.KIndexBar.length > 0) {
				_this.barTitle = _this.KIndexBar
			} else {
				_this.KIndexData.forEach(item => {
					_this.barTitle.push(item.label)
				})
			}
			_this.activeIndex = _this.KIndexData[0].label
			_this.$nextTick(() => {
				_this.getListNodesInfo()
				_this.getIndexNodesInfo()
			})
		},
		methods: {
			// 列取市
			itemClick(data) {
				this.$emit('itemClick', data)
			},
			// 列表锚点位置
			getListNodesInfo () {
				let _this = this
				const query = uni.createSelectorQuery().in(this);
				query.selectAll('.item-label').boundingClientRect().exec(res=>{
					if (res[0] && res[0].length > 0) {
						let oneTop = res[0][0].top
						_this.cityTopList =  res[0].map(item => {
							item.top = item.top - oneTop
							return item
						})
					}
				})
			},
			// 索引锚点位置
			getIndexNodesInfo() {
				let _this = this
				const query = uni.createSelectorQuery().in(this);
				query.selectAll('.index-item').boundingClientRect().exec(res=>{
					if (res[0] && res[0].length > 0) {
						let oneTop = res[0][0].top
						_this.indexTopList =  res[0]
					}
				})
			},
			// 滚动监听
			scroll(e) {
				let scrollTop = e.target.scrollTop
				for(let i = 0;i < this.cityTopList.length; i++){
					let h1 = this.cityTopList[i].top;
					let h2 = this.cityTopList[i + 1] ? this.cityTopList[i + 1].top : 9999999;
					if(scrollTop > h1 && scrollTop < h2) {
						this.activeIndex = this.cityTopList[i].id
					}
				}
			},
			// 点击索引
			indexedClick (data) {
				this.cIndex = data
				this.activeIndex = data
			},
			// 触摸开始
			touchStart(e) {
				this.targetIndexShow = true
				let scrollTop = e.touches[0].pageY
				for(let i = 0;i < this.indexTopList.length; i++){
					let h1 = this.indexTopList[i].top;
					let h2 = this.indexTopList[i + 1] ? this.indexTopList[i + 1].top : 9999999;
					if(scrollTop > h1 && scrollTop < h2) {
						this.activeIndex = this.indexTopList[i].id
						this.cIndex = this.indexTopList[i].id
					}
				}
			},
			// 触摸中
			touchMove(e) {
				let scrollTop = e.touches[0].pageY
				for(let i = 0;i < this.indexTopList.length; i++){
					let h1 = this.indexTopList[i].top;
					let h2 = this.indexTopList[i + 1] ? this.indexTopList[i + 1].top : 9999999;
					if(scrollTop > h1 && scrollTop < h2) {
						this.activeIndex = this.indexTopList[i].id
						this.cIndex = this.indexTopList[i].id
					}
				}
			},
			// 触摸结束
			touchEnd(e) {
				this.targetIndexShow = false
			}
		}
	}
</script>

<style scoped>
	.KIndexList-container{
		position: relative;
		height: 100%;
	}
	.KIndexList-container .item-label{
		color: #000000;
		padding: 15rpx 30rpx;
		font-size: 28rpx;
		background-color: #f7f8fa;
	}
	.KIndexList-container .item-label-active{
		color: #FF0000;
	}
	.KIndexList-container .item-name{
		color: #000000;
		padding: 24rpx 0rpx;
		margin: 0rpx 30rpx;
		font-size: 28rpx;
		border-bottom: 1rpx solid #ebedf0;
	}
	.KIndexList-container .item-name:last-child{
		border-color: transparent;
	}
	.KIndexList-container .tbr-index-box{
		position: absolute;
		top: 50%;
		right: 0rpx;
		transform: translateY(-50%);
	}
	.KIndexList-container .tbr-index-box .index-item{
		text-align: center;
		padding: 0rpx 20rpx;
		font-size: 22rpx;
		color: #000000;
	}
	.KIndexList-container .tbr-index-box .index-item-active{
		color: #FF0000;
	}
	.KIndexList-container .bar-target-index{
		position: absolute;
		top: 50%;
		left: 50%;
		transform: translate(-50%, -50%);
		line-height: 130rpx;
		width: 130rpx;
		border-radius: 50%;
		background-color: rgba(0,0,0,0.2);
		font-size: 50rpx;;
		text-align: center;
	}
</style>
