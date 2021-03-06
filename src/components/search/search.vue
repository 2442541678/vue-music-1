<template>
  <!-- 搜索 -->
  <div class="search">
    <div class="search-box-wrapper">
      <search-box ref="searchBox"
                  @query="onQueryChange"></search-box>
    </div>
    <div class="shortcut-wrapper"
         v-show="!query"
         ref="shortcutWrapper">
      <scroll class="shortcut"
              :data="shorcut"
              ref="shortcut"
              :refreshDelay="120">
        <div>
          <div class="hot-key">
            <h1 class="title">热门搜索</h1>
            <ul>
              <li class="item"
                  v-for="(item, index) in hotKey"
                  :key="index"
                  @click="addQuery(item.k)">
                <span>{{ item.k }}</span>
              </li>
            </ul>
          </div>
          <div class="search-history"
               v-show="searchHistory.length"
               ref="searchResult">
            <h1 class="title">
              <span class="text">搜索历史</span>
              <span class="clear"
                    @click="showConfirm">
                <i class="icon-clear"></i>
              </span>
            </h1>
            <!-- 历史列表 -->
            <search-list :searches="searchHistory"
                         @select="addQuery"
                         @delete="deleteOne"></search-list>
          </div>
        </div>
      </scroll>
    </div>
    <div class="search-result"
         v-show="query"
         ref="searchResult">
      <suggest :query="query"
               :num="num"
               ref="suggest"
               @listScroll="blurInput"
               @select="saveSearch"></suggest>
    </div>
    <confirm ref="confirm"
             :text="text"
             :confirmBtnText="'清空'"
             @confirm="deleteAll"></confirm>
    <router-view></router-view>
  </div>
</template>

<script>
import SearchBox from 'base/search-box/search-box'
import { getHotKey } from 'api/search'
import { ERR_OK } from 'api/config'
import Suggest from 'components/suggest/suggest'
import { mapActions } from 'vuex'
import SearchList from 'base/search-list/search-list'
import Confirm from 'base/confirm/confirm'
import Scroll from 'base/scroll/scroll'
import { playListMixin, searchMixin } from 'common/js/mixin'

export default {
  mixins: [playListMixin, searchMixin],
  data() {
    return {
      hotKey: [], // 热门关键词
      num: 20, // 一次搜索多少数据
      text: '确定要<span style="color: #ffcd32;font-whigh: 700;font-size: 20px"> 清空所有 </span>搜索记录' // 删除历史搜索框的默认文字
    }
  },
  created() {
    // 获取热门搜索的数据
    this._getHotKey()
  },
  methods: {
    // 获取热门搜索数据
    _getHotKey() {
      getHotKey().then(res => {
        if (res.code === ERR_OK) {
          this.hotKey = res.data.hotkey.slice(0, 12)
        }
      })
    },
    // 删除全部搜索记录
    deleteAll() {
      this.clearSearchHistory()
    },
    // 显示删除框
    showConfirm() {
      // 调用组件显示
      this.$refs.confirm.show()
    },
    // 迷你播放器存在的时候改变bottom
    handlePlayList(playList) {
      const bottom = playList.length > 0 ? '60px' : ''
      this.$refs.shortcutWrapper.style.bottom = bottom
      this.$refs.shortcut.refresh()
      this.$refs.searchResult.style.bottom = bottom
      this.$refs.suggest.refresh()
    },
    ...mapActions([
      'clearSearchHistory'
    ])
  },
  computed: {
    // 传给scroll计算滚动范围
    shorcut() {
      return this.hotKey.concat(this.searchHistory)
    }
  },
  components: {
    SearchBox,
    Suggest,
    SearchList,
    Confirm,
    Scroll
  },
  watch: {
    query(newQuery) {
      if (!newQuery) {
        setTimeout(() => {
          this.$refs.shortcut.refrech()
        }, 20)
      }
    }
  }
}
</script>

<style lang="stylus" rel="stylesheet/stylus">
@import '~common/stylus/variable'
@import '~common/stylus/mixin'
.search
  .search-box-wrapper
    margin 20px
  .shortcut-wrapper
    position fixed
    top 178px
    bottom 0
    width 100%
    .shortcut
      height 100%
      overflow hidden
      .hot-key
        margin 0 20px 20px 20px
        .title
          margin-bottom 20px
          font-size $font-size-medium
          color $color-text-l
        .item
          display inline-block
          padding 5px 10px
          margin 0 20px 10px 0
          border-radius 6px
          background $color-highlight-background
          font-size $font-size-medium
          color $color-text-d
      .search-history
        position relative
        margin 0 20px
        .title
          display flex
          align-items center
          height 40px
          font-size $font-size-medium
          color $color-text-l
          .text
            flex 1
          .clear
            extend-click()
            .icon-clear
              font-size $font-size-medium
              color $color-text-d
  .search-result
    position fixed
    width 100%
    top 178px
    bottom 0
</style>