1.使用案列
```
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
		methods: {
			click(e){
				console.log(e.item,e.index)
			},
		}
	}
</script>
```
2.注意事项
```
	（1）插槽为每块内容，使用时添加 #default="{item}"，item为数组的每项内容；
	（2）由于内部使用flex布局，宽度根据父级元素宽度而定，使用样式参数时不要设置宽度；
	（3）动画时长设置的是0.5s
```
参数|类型|说明|默认值|是否必须
-|-|-|-|
v-model|Array|将要循环的数组，数组的每一项为对象形式|无|是
lable|String|循环的key值|name|否
num|Number|设置每一行有多少个元素|4|否
height|String|每个元素的高度(注意带单位)|150rpx|否
dragName|Array|每个元素添加类名|空|否
dragStyle|Object|修改每个元素样式|空|否
disabled|Boolean|是否禁止拖动排序|false|否
touchTime|Number|长按时长多少才可以排序|500|否
change|function|单次点击事件,返回一个对象(item,index)|空|否