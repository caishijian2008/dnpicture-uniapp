<template>
  <view>
    <view class="album_bg">
      <image :src="album.cover" mode="widthFix"></image>
      <view class="album_info">
        <view class="album_name">{{album.name}}</view>
        <view class="album_btn">关注专辑</view>
      </view>
    </view>
    
    <view class="album_author">
      <view class="album_author_info">
        <image :src="album.user.avatar" mode="widthFix"></image>
        <view class="author_name">{{album.user.name}}</view>
      </view>
      <view class="album_author_desc">
        <text>{{album.desc}}</text>
      </view>
    </view>
    
    <view class="album_list">
      <view class="album_item"
        v-for="(item,index) in wallpaper"
        :key="item.id">
        <go-detail :list='wallpaper'
                   :index='index'>
          <image mode="aspectFill"
                 :src="item.thumb+item.rule.replace('$<Height>',360)"></image>
        </go-detail>
      </view>
    </view>
  </view>
</template>

<script>
import goDetail from "@/components/goDetail";
export default {
  components: {
    goDetail
  },
  data() {
    return {
      params: {
        limit: 30,
        order: "new",
        skip: 0,
        // 1 返回值中有album对象和wallpaper数组，表示第一次请求数据
        // 0 返回值中只有wallpaper数组，表示不是第一次请求数据
        first: 1
      },
      id: -1,
      album: {},
      wallpaper: [],
      hasMore: true
    }
  },
  onLoad(options) {
    // console.log(options)
    this.id = options.id
    // this.id = "5e59075ae7bce73965512bd6"
    this.getList()
  },
  // 页面触底，上拉加载下一页事件, 与scroll-view组件的scrolltolower事件结果相同
  onReachBottom() {
    // console.log('sssssssssssss')
    if(this.hasMore) {
      this.params.first = 0;
      this.params.skip += this.params.limit;
      this.getList();
    } else {
      uni.showToast({
        title: "没有更多数据了",
        icon: "none"
      })
    }
  },
  methods: {
    getList() {
      this.request({
        url: `http://157.122.54.189:9088/image/v1/wallpaper/album/${this.id}/wallpaper`,
        data: this.params
      })
      .then(result => {
        console.log(result)
        if(Object.keys(this.album).length === 0) {
          this.album = result.res.album
        }
        
        if(result.res.wallpaper.length === 0) {
          this.hasMore = false;
          uni.showToast({
            title: "没有更多数据了",
            icon: "none"
          })
          return;
        }
        
        this.wallpaper = [...this.wallpaper, ...result.res.wallpaper];
      })
    }
  },
}
</script>

<style lang="scss" scoped>
.album_bg {
  position: relative;
  image {}
  .album_info {
    position: absolute;
    width: 100%;
    left: 0;
    bottom: 0;
    display: flex;
    justify-content: space-between;
    height: 80rpx;
    align-items: center;
    color: #fff;
    padding: 0 15rpx;
    .album_name {
      font-size: 40rpx;
    }
    .album_btn {
      background-color: $color;
      width: 152rpx;
      height: 60rpx;
      display: flex;
      justify-content: center;
      align-items: center;
      border-radius: 10rpx;
    }
  }
}
.album_author {
  padding: 10rpx;
  .album_author_info {
    padding: 10rpx 0;
    display: flex;
    image {
      width: 50rpx;
    }
    .author_name {
      color: #000;
      margin-left: 15rpx;
    }
  }
  .album_author_desc {}
}

.album_list {
  display: flex;
  flex-wrap: wrap;
  .album_item {
    width: 33.33%;
    border: 3rpx solid #fff;
    image {}
  }
}
</style> 
