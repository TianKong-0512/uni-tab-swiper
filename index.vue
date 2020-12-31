<template>
	<view class="tabs_swiper">
		<view class="swiper_tab">
			<scroll-view class="swiper_scroll"
			 :style="{height:`${tabHeight}rpx`}"
			 scroll-x
			 :scroll-anchoring="true"
			 :scroll-left="tabsScrollLeft"
			 :enable-flex="true"
			 @scrolltolower="scrolltolower">
				<template v-for="(item, index) in swiperTabs">
					<view class="tab"
					 :style="[tabStyle(index)]"
					 :key="index"
					 @click="tabchange(index)">
						{{item.tabTitle}}
					</view>
				</template>
				<view class="tab_actived"
				 :style="{width: `${tabWidth}rpx`,left: `${activedLeft}rpx`,height: tabActivedOpt.height,top: `calc(${tabHeight}rpx - ${tabActivedOpt.height})`}">
					<view class="actived"
					 :style="[activedStyle]"></view>
				</view>
			</scroll-view>
		</view>
		<view class="swiper_view">
			<swiper class="swiper"
			 :duration="300"
			 :easing-function="'linear'"
			 :current="current"
			 @transition="transition"
			 @animationfinish="animationfinish">
				<swiper-item v-for="(swiperItem, index) in swiperTabs"
				 :key="index">
					<slot name="swiper"
					 :swiperitem="swiperItem">
					</slot>
				</swiper-item>
			</swiper>
			<view class="swiper_mask"
			 v-if="swipermask"
			 @click.stop="swiperClick"></view>
		</view>
	</view>
</template>

<script>
	import color from './colorGradient.js';
	export default {
		props: {
			activeColor: {
				type: String,
				default: '#f70046'
			},
			titlecolor: {
				type: String,
				default: '#333333'
			},
			swiperTabs: {
				type: Array,
				default: () => {
					return [];
				}
			},
			tabHeight: {
				type: Number,
				default: 100
			},
			tabWidth: {
				type: Number,
				default: 250
			},
			tabActivedOpt: {
				type: Object,
				default: () => {
					return {
						width: '100rpx',
						height: '15rpx',
					}
				}
			},
			// 当没有修改时是否返回
			unchange: {
				type: Boolean,
				default: false
			}
		},
		data() {
			return {
				// 当前偏移量
				nowActivedLeft: 0,
				// 设备的宽度
				windowWidth: null,
				// 已经滚动的比例
				swiperRatio: 0,
				// 当前所在位置的下标
				current: 0,
				// 下一个tab的下标
				nextCurrent: 0,
				// 点击滚动前的下标
				clickCurrent: 0,
				// 当前变化类型
				changeSource: null,
				// px跟rpx单位的比例
				unitRatio: 0,
				// 解决点击tab切换时点击swiper滚动不执行问题
				swipermask: false,
				// tab渐变色的数组
				colorArr: [],
				end: false
			};
		},
		computed: {
			tabStyle() {
				return (index) => {
					let color = '';
					let next = false
					let colorindex = Math.ceil(Math.abs(this.swiperRatio * 10));
					if (colorindex >= this.colorArr.length) {
						colorindex = this.colorArr.length - 1;
					}
					if (index == this.current) {
						color = this.colorArr[colorindex]
					} else if (index == this.nextCurrent) {
						color = this.colorArr[this.colorArr.length - colorindex]
					}
					const style = {
						minWidth: `${this.tabWidth}rpx`,
						width: `${this.tabWidth}rpx`,
						paddingBottom: `${this.activedStyle.height}`,
						color: color,
					}
					return style
				}
			},
			activedStyle() {
				let style = {
					...this.tabActivedOpt,
					backgroundColor: this.activeColor
				}
				return style;
			},
			activedLeft() {
				let index = 0;
				if (this.changeSource == 'click') {
					index = this.clickCurrent;
				} else {
					index = this.current;
				}
				const changeleft = this.tabWidth * (this.swiperRatio + index);
				return changeleft;
			},
			tabsScrollLeft() {
				const left = (this.activedLeft + this.tabWidth / 2) * this.unitRatio;
				const width = this.windowWidth / 2;
				let removew = left;
				if (left > width) {
					removew = width;
				}

				return left - removew;
			}
		},
		created() {
			uni.getSystemInfo({
				success: data => {
					this.windowWidth = data.windowWidth;
					this.unitRatio = data.windowWidth / 750;
				}
			})
			this.init();
		},
		methods: {
			init() {
				this.colorArr = color.colorGradient(this.activeColor, this.titlecolor);
			},
			scrolltolower() {
				this.end = true;
			},
			tabchange(index) {
				if (this.swipermask) {
					return;
				}
				if (this.current == index) {
					return;
				}
				this.swipermask = true;
				this.changeSource = 'click';
				this.clickCurrent = this.current;
				this.current = index;
			},
			transition(event) {
				const swiperx = event.detail.dx;
				if (this.changeSource != 'click') {
					if (swiperx > 0) {
						this.nextCurrent = this.current + 1;
					} else {
						this.nextCurrent = this.current - 1;
					}
				} else {
					this.nextCurrent = this.current;
				}
				if (this.current == this.swiperTabs.length - 1 && this.nextCurrent > this.current) {
					return;
				} else if (this.current == 0 && this.nextCurrent < this.current) {
					return;
				}
				this.swiperRatio = swiperx / this.windowWidth;
				console.log('log--->swiperRatio', this.swiperRatio);
				if (Number.isInteger(this.swiperRatio)) {
					this.clickCurrent = 0;
				}
			},
			animationfinish(data) {
				let ischange = false;
				if (this.current !== data.detail.current) {
					ischange = true;
				}
				this.current = data.detail.current;
				this.clickCurrent = 0;
				this.swiperRatio = 0;
				this.changeSource = null;
				this.swipermask = false;
				if (this.unchange || ischange) {
					this.$emit('change', this.current)
				}
			},
			swiperClick() {
				this.swipermask = false;
			}
		}
	}
</script>

<style lang="scss"
 scoped>
	@import './index.scss';
</style>
