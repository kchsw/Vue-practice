<template>
	<transition name="slide">
		<div class="add-song" v-show="showflag" @click.stop>
			<div class="header">
				<h1 class="title">添加歌曲到列表</h1>
				<div class="close" @click="hide">
					<i class="icon-close"></i>
				</div>
			</div>
			<div class="search-box-wrapper">
				<search-box
				  ref="searchBox"
				  @query="onQuertChange"
				  placeholder="搜索歌曲"
				></search-box>
			</div>
			<div class="shortcut" v-show="!query">
				<switches
				  :switches="switches"
				  :currentIndex="currentIndex"
				  @switch="switchItem"
				></switches>
				<div class="list-wrapper">
					<scroll class="list-scroll"
					  :refreshDelay="refreshDelay"
					  ref="songList"
					  v-if="currentIndex === 0"
					  :data="playHistory"
					>
						<div class="list-inner">
							<song-list
							  :songs="playHistory"
							  @select="selectSong"
							></song-list>
						</div>
					</scroll>
					<scroll class="list-scroll"
					  :refreshDelay="refreshDelay"
					  ref="searchList"
					  v-if="currentIndex === 1"
					  :data="searchHistory"
					>
						<div class="list-inner">
							<search-list 
							  @delete="deleteSearchHistory" 
							  @select="addQuery" 
							  :searches="searchHistory"></search-list>
						</div>
					</scroll>
				</div>
			</div>
			<div class="search-result" v-show="query">
				<Suggest 
				  :query="query"
				  @listScroll="bulrInput"
				  @select="selectSuggest"
				  :showSinger="showSinger"
				></Suggest>
			</div>
			<top-tip ref="topTip">
				<div class="tip-title">
					<i class="icon-ok"></i>
					<span class="text">1首歌曲已经添加到播放列表</span>
				</div>
			</top-tip>
		</div>
	</transition>
</template>

<script>
	import SearchBox from 'base/search-box/search-box'
  	import SongList from 'base/song-list/song-list'
  	import SearchList from 'base/search-list/search-list'
  	import Scroll from 'base/scroll/scroll'
  	import Switches from 'base/switches/switches'
  	import TopTip from 'base/top-tip/top-tip'
  	import Suggest from 'components/suggest/suggest'
  	import { searchMixin } from 'common/js/mixin'
  	import { mapGetters, mapActions } from 'vuex'
  	import Song from 'common/js/song'

	export default{
		mixins: [searchMixin],
		name: 'AddSong',
		data(){
			return{
				showflag: false,
				showSinger: false,
				currentIndex: 0,
				switches: [
					{
						name: '最近播放'
					},
					{
						name: '搜索历史'
					}
				]
			}
		},
		computed: {
			...mapGetters([
		        'playHistory'
		    ])	
		},
		methods: {
			show(){
				this.showflag = true
				this.refreshList()
			},
			hide(){
				this.showflag = false
			},
			refreshList(){
				setTimeout(()=>{
					if(this.currentIndex === 0){
						this.$refs.songList.refresh()
					}else{
						this.$refs.searchList.refresh()
					}
				}, 20)
			},
			switchItem(index) {
		        this.currentIndex = index
		    },
		    selectSong(song, index){
		    	//当前播放歌曲 index 为 0
		    	if(index !== 0){
		    		this.insertSong(new Song(song))
		    		this.$refs.topTip.show()
		    	}
		    },
		    selectSuggest(){
		    	this.$refs.topTip.show()
                this.saveSearch()
		    },
		    ...mapActions([
		        'insertSong'
		    ])
		},
		components: {
	        SearchBox,
	        SongList,
	        SearchList,
	        Scroll,
	        Switches,
	        TopTip,
	        Suggest
	    }

	}
</script>

<style lang='stylus' scoped>
@import '~common/stylus/variable';
@import '~common/stylus/mixin';

.add-song
    position: fixed
    top: 0
    bottom: 0
    width: 100%
    z-index: 200
    background: $color-background
    &.slide-enter-active, &.slide-leave-active
      transition: all 0.3s
    &.slide-enter, &.slide-leave-to
      transform: translate3d(100%, 0, 0)
    .header
      position: relative
      height: 44px
      text-align: center
      .title
        line-height: 44px
        font-size: $font-size-large
        color: $color-text
      .close
        position: absolute
        top: 0
        right: 8px
        .icon-close
          display: block
          padding: 12px
          font-size: 20px
          color: $color-theme

    .search-box-wrapper
      margin: 20px
    .shortcut
      .list-wrapper
        position: absolute
        top: 165px
        bottom: 0
        width: 100%
        .list-scroll
          height: 100%
          overflow: hidden
          .list-inner
            padding: 20px 30px
    .search-result
      position: fixed
      top: 124px
      bottom: 0
      width: 100%
    .tip-title
      text-align: center
      padding: 18px 0
      font-size: 0
      .icon-ok
        font-size: $font-size-medium
        color: $color-theme
        margin-right: 4px
      .text
        font-size: $font-size-medium
        color: $color-text
</style>