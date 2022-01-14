<template>
	<view class="dragSort">
		<transition-group class="tuodong" tag="uni-view">
			<view
			  v-for="(item,index) in value" 
			  :key="item[lable]" 
			  class="animation" 
			  :class="['move'+index,...dragName]"
			  :style="[dragClass[lable] == item[lable]?{opacity:0}:'',{width:`calc(100% / ${num})`,height},dragStyle]" 
			  @touchstart="e => touchstart(index,e)" 
			  @touchend="delData"
			  @touchmove.prevent="dragSort" 
			  @click="change(item,index)" 
			>	
				<slot :item="item"></slot>
			</view>
		</transition-group>
		<!-- 长按克隆元素效果 -->
		<view 
		  class="clone" 
		  v-show="cloneShow" 
		  :style="[cloneShow?cloneStyle:'',cloneToStyle,{width:`calc(100% / ${num})`,height}]"
		>
			<slot :item="dragClass"></slot>
		</view>
	</view>
</template>

<script>
	export default {
		props:{
			value:{
				type: Array,
				required:true
			},
			lable:{
				type: String,
				default:'name'
			},
			num:{
				type: Number,
				default:4
			},
			height:{
				type: String,
				default:'150rpx'
			},
			dragName:{
				type: Array,
				default: function(){
					return []
				}
			},
			dragStyle:{
				type: Object,
				default: function(){
					return {}
				}
			},
			disabled:{
				type: Boolean,
				default:false
			},
			touchTime:{
				type: Number,
				default:500
			}
		},
		data(){
			return{
				dragIndex:-1,//最开始的索引值
				cloneShow:false,//模拟的克隆元素是否显示
				cloneStyle:{},//拖动模拟的克隆元素动画
				cloneToStyle:{},//模拟的克隆元素到自身的位移
				touchOne:{},//手指第一次按住的位置
				quantity:{},//第一次按住与元素自身的差量
				dragClass:{},//获取按住的元素信息用来创建模拟的克隆元素
				dragInfo:{},//获取元素自身的宽高，来判断位移多少改变索引
				info:{},//克隆元素位移时的差量
				displace:{x:0,y:0},//用来判断位移到第几块了
				fatherInfo:{},//父级容器信息，用来判断拖出容器时不进行排序
			}
		},
		methods:{
			// 单次点击
			change(item,index){
				this.$emit('change',{item,index})
			},
			// 长按可拖动，记录点击元素信息
			touchstart(e,event) {  
				if(!this.disabled)return
				let info = event.touches[0]
				this.time = setTimeout(() => {  
					// 获取移动的元素索引
					this.dragIndex = e
					// 保存数据,方便排序
					this.dragClass = JSON.parse(JSON.stringify(this.value[e]))
					this.cloneShow = true
					// 记录点击位置
					this.touchOne.clientX = info.clientX
					this.touchOne.clientY = info.clientY
					// 获取元素信息
					const query = uni.createSelectorQuery().in(this);
					this.$nextTick(()=>{
						let oldDom = new Promise((resolve,reject)=>{query.select(`.move${e}`).boundingClientRect(data => {resolve(data)}).exec()})
						let newDom = new Promise((resolve,reject)=>{query.select(`.clone`).boundingClientRect(data => {resolve(data)}).exec()})
						let fatherDom = new Promise((resolve,reject)=>{query.select(`.tuodong`).boundingClientRect(data => {resolve(data)}).exec()})
						Promise.all([oldDom,newDom,fatherDom]).then(v=>{
							// 自身元素的信息,用来判断位移多少后进行排序
							this.dragInfo = v[0]
							this.quantity.left = this.touchOne.clientX - v[0].left
							this.quantity.right = v[0].width - (this.touchOne.clientX - v[0].left)
							// 父级容器信息保存
							this.fatherInfo = v[2]
							// 模拟克隆元素位移到自身
							setTimeout(k=>{
								this.cloneToStyle.transform = `translate(${v[0].left - v[1].left}px,${v[0].top - v[1].top}px)`
								this.cloneToStyle.transition = `transform 0s`
								this.cloneToStyle.opacity = 1
								this.$forceUpdate()
							},150)
							// 保存起来，拖动时添加差量
							this.info.X = v[0].left - v[1].left
							this.info.Y = v[0].top - v[1].top
							this.$forceUpdate()
						})
					})
				},this.touchTime)  
				return false 
			},  
			// 松开清除数据
			delData(){
				clearTimeout(this.time)
				this.dragClass = {}
				this.cloneStyle = {}
				this.cloneToStyle = {}
				this.touchOne = {}
				this.displace = {x:0,y:0}
				this.cloneShow = false
				this.dragIndex = -1
				this.$forceUpdate()
			},
			// 拖动计算图标位置
			dragSort(event){
				clearTimeout(this.time)
				let {touchOne,dragIndex,info,displace,dragInfo,fatherInfo,quantity,num} = this
				// 判断长按是否生效
				if(!this.cloneShow) return
				this.cloneToStyle = {opacity:1}
				// 拿到元素的宽高
				let {width,height} = dragInfo
				// 获取滑动信息
				let touch = event.touches[0]
				if( touch.clientX-quantity.left+width*.5>=fatherInfo.left && touch.clientX+quantity.right-width*.5<=fatherInfo.width+fatherInfo.left
					&& touch.clientY>=fatherInfo.top && touch.clientY<=fatherInfo.height+fatherInfo.top){
					// x轴平移
					let transversal = ((touch.clientX - touchOne.clientX)/width).toFixed(1)
					// y轴平移
					let vertical = ((touch.clientY - touchOne.clientY)/height).toFixed(1)
					// 左移
					if( transversal < displace.x-.5){
						this.value.splice(dragIndex-1,0,...this.value.splice(dragIndex,1))
						this.dragIndex--
						this.displace.x--
					}else if(transversal > displace.x+.5 && dragIndex+1<=this.value.length-1){
						this.value.splice(dragIndex+1,0,...this.value.splice(dragIndex,1))
						this.dragIndex++
						this.displace.x++
					}else if(vertical < displace.y - .5 && dragIndex - num >= 0){
						this.value.splice(dragIndex-num,0,...this.value.splice(dragIndex,1))
						this.dragIndex -= num
						this.displace.y--
					}else if(vertical > displace.y+.5 && dragIndex+num<=this.value.length-1){
						this.value.splice(dragIndex+num,0,...this.value.splice(dragIndex,1))
						this.dragIndex +=num
						this.displace.y++
					}
				}
				// 拖动克隆元素
				this.cloneStyle['z-index'] = 40
				this.cloneStyle.transform = `translate(${info.X + touch.clientX - touchOne.clientX}px,${info.Y + touch.clientY - touchOne.clientY}px)`
				this.cloneStyle.transition = `transform 0s`
				this.$forceUpdate()
			},
		},
		watch:{
			cloneShow(news,old){
				if(!news){
					this.$emit('input',this.value)
				}
			}
		}
	}
</script>
<style lang="scss" scoped>
	.dragSort{
		width: 100%;
		height: 100%;
	}
	.clone{
		opacity: 0;
		position: absolute !important;
		top: 0;
		left: 0;
	}
	.tuodong{
		position: relative;
		width: 100%;
		height: 100%;
		display: flex;
		flex-wrap: wrap;
	}
	.animation{
		transition: all .5s
	}
</style>
