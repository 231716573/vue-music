<template>
  <div class="singer" ref='singer'>
  	<div class="loading" v-show="!singers.length">
  		<loading></loading>
  	</div>
  	<list-view @select="selectSinger" :data="singers" ref="list"></list-view>
  	<router-view></router-view>
  </div>
</template>

<script>
import {getSingerList} from 'api/singer'
import {ERR_OK} from 'api/config'
import Singer from 'common/js/singer'
import Loading from 'base/loading/loading'
import ListView from 'base/listview/listview'
import {mapMutations} from 'vuex'
import {playlistMixin} from 'common/js/mixin'

const HOT_NAME = '热门'
const HOT_SINGER_LEN = 10

export default {
	mixins: [playlistMixin],
	data() {
		return {
			singers: [] 
		}
	},
	created() {
		// setTimeout(() => {
			this._getSingerList()
			// this.selectSinger()
		// }, 2000)
	},
	methods: {
		handlePlaylist(playList) {
			const bottom = playList.length > 0 ? '60px' : ''
			this.$refs.singer.style.bottom = bottom
			this.$refs.list.refresh()
		},
		// 接收从 listview 传出来的select内容
		selectSinger(singer) {
			this.$router.push({
				path: `/singer/${singer.id}`
			})
			// console.log(singer)
			this.setSinger(singer)
		},
		// 获取歌手列表
		_getSingerList() {
			getSingerList().then((res) => {
				if (res.code === ERR_OK) {
					this.singers = this._normalizeSinger(res.data.list)
					// console.log(this.singers)
				}
			})
		},
		// 对歌手列表进行排版处理
		_normalizeSinger(list) {
			let map = {
				hot: {
					title: HOT_NAME,
					items: []
				}
			}
			list.forEach((item, index) => {
				// 在热门添加指定内容
				if (index < HOT_SINGER_LEN) {
					map.hot.items.push(new Singer({
						id: item.Fsinger_mid,
						name: item.Fsinger_name
					}))
				}

				// 新增 A-Z 栏，并添加对应内容
				const key = item.Findex
				if (!map[key]) {
					map[key] = {
						title: key,
						items: []
					}
				}
				map[key].items.push(new Singer({
						id: item.Fsinger_mid,
						name: item.Fsinger_name
					}))
			})
			// 为了得到有序列表，我们需要处理 map
			let hot = []
			let ret = []
			for (let key in map) {
				let val = map[key]
				if (val.title.match(/[a-zA-Z]/)) {
					ret.push(val)
				} else if (val.title === HOT_NAME) {
					hot.push(val)
				}
			} 
			ret.sort((a, b) => {
				return a.title.charCodeAt(0) - b.title.charCodeAt(0)
			})
			return hot.concat(ret)
		},
		...mapMutations({
			setSinger: 'SET_SINGER'
		})
	},
	components: {
		Loading, ListView
	}
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="stylus" rel="stylesheet/stylus">
  .singer
    position: fixed
    top: 88px
    bottom: 0
    width: 100%
</style>
