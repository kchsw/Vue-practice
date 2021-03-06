<template>
	<scroll class="listview"
	  ref="listview"
	  :data="data"
	  @scroll="scroll"
	  :listenScroll="listenScroll"
	  :probeType="probeType"
	>
	    <ul>
	    	<li class="list-group"
	    	  v-for="group in data"
	    	  ref="listGroup"
	    	>
	    		<h2 class="list-group-title">{{group.title}}</h2>
	    		<ul>
	    			<li class="list-group-item"
	    			  v-for="item in group.items"
	    			  @click="setlectItem(item)"
	    			>
	    				<img  class="avatar" v-lazy="item.avatar">
	    				<span class="name">{{item.name}}</span>
	    			</li>
	    		</ul>
	    	</li>
	    </ul>
	    <div class="list-shortcut"
	      @touchstart.stop.prevent="onShortcutTouchStar"
	      @touchmove.stop.prevent="onShortcutTouchMove"
	    >
	    	<ul>
	    		<li class="item"
	    		  v-for="(item,index) in shortcutList"
	    		  :class="{'current': currentIndex === index}"
	    		  :data-index ="index"
	    		>
	    			{{item}}
	    		</li>
	    	</ul>
	    </div>
	    <div class="list-fixed" ref="fixed" v-show="fixedTitle">
	    	<div class="fixed-title">{{fixedTitle}}</div>
	    </div>
	    <div v-show="!data.length" class="loading-container">
	      <loading></loading>
	    </div>	
	</scroll>
</template>

<script>
    import Scroll from 'base/scroll/scroll'
    import Loading from 'base/loading/loading'
    import { getData } from 'common/js/dom'
    
    const ANCHOR_HEIGHT = 18
    const TITLE_HEIGHT = 30

	export default{
		name: 'ListView',
		data(){
			return{
				currentIndex: 0,
				scrollY: -1,
				diff: -1
			}
		},
		props:{
			data:{
				type: Array,
				default: []
			}
		},
		components:{
			Scroll,
			Loading
		},
		created(){
			this.probeType = 3
			this.listenScroll = true
			this.touch = {}
			this.listHeight = []
		},
		computed:{
			shortcutList(){
				return this.data.map((group)=>{
					return group.title.substr(0,1)
				})
			},
			fixedTitle(){
				if(this.scrollY > 0){
					return ''
				}
				return this.data[this.currentIndex] ? this.data[this.currentIndex].title : ''
			}
		},
		methods:{
			setlectItem(item){
				this.$emit('select',item)
			},
			_scrollTo(index){
				if(!index && index !== 0){
					return
				}
				if(index < 0){
					index = 0
				}else if(index > this.listHeight.length - 2){
					index = this.listHeight.length - 2
				}
				this.$refs.listview.scrollToElement(this.$refs.listGroup[index],0)
				this.scrollY = this.$refs.listview.scroll.y
			},
			onShortcutTouchStar(e){
				let anchorIndex = getData(e.target,'index')
				let firstTouch = e.touches[0]
				this.touch.y1 = firstTouch.pageY
				this.touch.anchorIndex = anchorIndex
				this._scrollTo(anchorIndex)
			},
			onShortcutTouchMove(e){
				let firstTouch = e.touches[0]
				this.touch.y2 = firstTouch.pageY
				let delta = (this.touch.y2 - this.touch.y1) / ANCHOR_HEIGHT | 0
				let anchorIndex = parseInt(this.touch.anchorIndex) + delta
				this._scrollTo(anchorIndex)	
			},
			scroll(pos){
				this.scrollY = pos.y
			},
			refresh(){
				this.$refs.listview.refresh()
			},
			_calculateHeight(){
				this.listHeight = []
				const list = this.$refs.listGroup
				let height = 0
				this.listHeight.push(height)
				for(let i = 0; i < list.length; i++){
					let item = list[i]
					height += item.clientHeight
					this.listHeight.push(height)
				}
			}
		},
		watch:{
			data(){
				setTimeout(()=>{
					this._calculateHeight()
				},20)
			},
			scrollY(newY){
				const listHeight = this.listHeight
				if(newY > 0){
					this.currentIndex = 0
					return
				}

				for(let i = 0; i < listHeight.length - 1; i++){
					let Height1 = listHeight[i]
					let Height2 = listHeight[i + 1]
					if(-newY >= Height1 && -newY < Height2){
						this.currentIndex = i
						this.diff = Height2 + newY
						return
					}
				}

				this.currentIndex = listHeight.length - 2
			},
			diff(newVal){
				let fixedTop = (newVal > 0 && newVal < TITLE_HEIGHT) ? newVal - TITLE_HEIGHT : 0
				if(this.fixedTop === fixedTop){
					return
				}
				this.fixedTop = fixedTop
				this.$refs.fixed.style.transform = `translate3d(0, ${fixedTop}px, 0)`
			}
		}

	}
</script>

<style lang='stylus' scoped>
@import '~common/stylus/variable';
@import '~common/stylus/mixin';

.listview
    position: relative
    width: 100%
    height: 100%
    overflow: hidden
    background: $color-background
    .list-group
      padding-bottom: 30px
      .list-group-title
        height: 30px
        line-height: 30px
        padding-left: 20px
        font-size: $font-size-small
        color: $color-text-l
        background: $color-list-background
      .list-group-item
        display: flex
        align-items: center
        padding: 20px 0 0 30px
        .avatar
          width: 50px
          height: 50px
          border-radius: 50%
        .name
          margin-left: 20px
          color: $color-text-l
          font-size: $font-size-medium
    .list-shortcut
      position: absolute
      z-index: 30
      right: 0
      top: 50%
      transform: translateY(-50%)
      width: 20px
      padding: 20px 0
      border-radius: 10px
      text-align: center
      background: $color-background-h
      font-family: Helvetica
      .item
        padding: 3px
        line-height: 1
        color: $color-text-l
        font-size: $font-size-small
        &.current
          color: $color-text-h
    .list-fixed
      position: absolute
      top: 0
      left: 0
      width: 100%
      .fixed-title
        height: 30px
        line-height: 30px
        padding-left: 20px
        font-size: $font-size-small
        color: $color-text-l
        background: $color-list-background
    .loading-container
      position: absolute
      width: 100%
      top: 50%
      transform: translateY(-50%)
</style>