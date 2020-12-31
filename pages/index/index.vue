<template>
	<view class="content">
		<tabs-swiper :swiperTabs="swiperTabs"
		 @change="swiperTab">
			<template v-slot:swiper="{ swiperitem }">
				{{ swiperitem.tabTitle }}
			</template>
		</tabs-swiper>
	</view>
</template>

<script>
	import TabsSwiper from '../../components/tabs-swiper/index.vue'
	export default {
		components: {
			TabsSwiper
		},
		data() {
			return {
				swiperTabs: [{
						tabTitle: 'A页面'
					},{
						tabTitle: 'B页面'
					},{
						tabTitle: 'C页面'
					},{
						tabTitle: 'D页面'
					},{
						tabTitle: 'E页面'
					},{
						tabTitle: 'F页面'
					}, {
						tabTitle: 'G页面'
					}, {
						tabTitle: 'H页面'
					}, {
						tabTitle: 'I页面'
					}
				]
			}
		},
		onLoad() {

		},
		methods: {
			swiperTab(index) {
				console.log('log---> 当前', index);
			}
		}
	}
</script>

<style scoped
 lang="scss">
	.content {
		width: 100%;
	}
</style>
