<template>
	<view class="content">
		<drag-sort v-model="arr" :disabled="true" @change="click" :touchTime="1000" lable="lable" :dragStyle="{background:'yellow'}" :dragName="['tianjia']" height="100rpx" :num="3">
			<template #default="{item}">
				<view style="font-size: 20rpx;color: #fff;text-align: center;line-height: 100rpx;" :style="{background:item.background}">{{item.lable}}</view>
			</template>
		</drag-sort>
	</view>
</template>

<script>
	import dragSort from '@/pages/components/dragSort/dragSort.vue'
	export default {
		components:{
			dragSort
		},
		data() {
			return {
				arr:[
					{lable:'hello1',background:'#a9dd7a'},
					{lable:'hello2',background:'#299ff3'},
					{lable:'hello3',background:'#5069e3'},
					{lable:'hello4',background:'#b34ee7'},
					{lable:'hello5',background:'#df589d'},
					{lable:'hello6',background:'#dfaf58'},
					{lable:'hello7',background:'#ff0000'},
					{lable:'hello8',background:'#ede368'},
					{lable:'hello9',background:'#9e9e9e'},
					{lable:'hello10',background:'#795548'}
				]
			}
		},
		onLoad() {
			
		},
		methods: {
			click(e){
				console.log(e)
				console.log(this.arr)
			},
		}
	}
</script>

<style>
	.content{
		/* width: 60%; */
		/* margin: 0 auto; */
	}
</style>
