<template>
	<scroll-view @scrolltolower="handleToLower" class="recommend_view" scroll-y v-if="recommends.length > 0">
    <!-- 推荐 -->
		<view class="recommend_wrap">
      <navigator 
        class="recommend_item"
        v-for="item in recommends"
        :key="item.id"
        :url="`/pages/album/index?id=${item.target}`"
       >
        <image mode="widthFix" :src="item.thumb"></image>
      </navigator>
    </view>
    
    <!-- 月份 -->
    <view class="month_wrap">
      <view class="month_title">
        <view class="month_title_info">
          <view class="month_info">
            <text>{{months.DD}}</text> / 
            {{months.MM}} 月
          </view>
          <view class="month_text"> {{months.title}}</view>
        </view>
        <view class="month_title_more">更多 > </view>
      </view>
      <view class="month_content">
        <view 
        class="month_item"
        v-for="(item, index) in months.items"
        :key="item.id">
          <go-detail :list='months.items'
                     :index='index'>
            <image mode='aspectFill'
                   :src="item.thumb+item.rule.replace('$<Height>',360)">
            </image>
          </go-detail>
        </view>
      </view>
    </view>
    
    <!-- 热门 -->
    <view class="hots_wrap">
      <view class="hots_title">
        <text> 热门 </text>
      </view>
      <view class="hots_content">
        <view class="hots_item"
          v-for="(item,index) in hots"
          :key="item.id">
          <go-detail :list='hots'
                     :index='index'>
            <image mode='widthFix'
                   :src="item.thumb">
            </image>
          </go-detail>
        </view>
      </view>
    </view>
	</scroll-view>
</template>

<script>
  import moment from 'moment'
  import goDetail from '@/components/goDetail'
	export default {
    components: {
      goDetail
    },
		data() {
			return {
        // 推荐列表
				recommends: [],
        // 月份
        months: {},
        // 热门
        hots: [],
        params: {
          // 要获取几条数据
          limit: 30,
          // 关键字
          order: 'hot',
          // 要跳过几条数据
          skip: 0
        },
        // 是否还有下一页
        hasMore: true
			}
		},
		methods: {
      // 获取接口的数据
      getList() {
        this.request({
          url: "http://157.122.54.189:9088/image/v3/homepage/vertical",
          data: this.params
        })
        .then(result => {
          console.log('recommend: ',result)
          
          // 判断还有没有下一页的数据
          if(result.res.vertical === 0) {
            this.hasMore = false
            uni.showToast({
              title: "没有更多数据了",
              icon: "none"
            })
            return;
          }
          
          // 优化
          if(this.recommends.length === 0 ) {
            // 推荐模块
            this.recommends = result.res.homepage[1].items
            // 月份模块
            this.months = result.res.homepage[2]
            this.months.MM = moment(this.months.stime).format('MM')
            this.months.DD = moment(this.months.stime).format('DD')
          } else {
            // 热门数据的列表
            // 数组拼接
            this.hots = [...this.hots, ...result.res.vertical];
            console.log('hots: ', this.hots)
          }
        })
      },
      // 滚动条触底
			handleToLower() {
        /**
         * 1.修改参数 skip+=limit
         * 2.重新发送请求 getList
         * 3.请求成功则 hots 数据叠加
         */
        if (this.hasMore) {
          this.params.skip += this.params.limit
          this.getList()
        } else {
          uni.showToast({
            title: "没有数据了",
            icon: "none"
          })
        }
      }
		},
    mounted() {
      uni.setNavigationBarTitle({
        title: "推荐"
      })
      this.getList()
    }
	}
</script>

<style lang="scss" scoped>
.recommend_view {
  height: calc( 100vh - 36px);
}
.recommend_wrap {
  display: flex;
  flex-wrap: wrap;
  .recommend_item {
    width: 50%;
    border: 5rpx solid #fff;
  }
}

.month_wrap {
  .month_title {
    display: flex;
    justify-content: space-between;
    padding: 20rpx;
    .month_title_info {
      color: $color;
      font-size: 30rpx;
      font-weight:  600;
      display: flex;
      .month_info {
        text {
          font-size: 34rpx;
        }
      }
      .month_text {
        font-size: 34rpx;
        color: #666;
        margin-left: 30rpx;
      }
    }
    .month_title_more {
      font-size: 24rpx;
      color: $color;
    }
  }
  .month_content {
    display: flex;
    flex-wrap: wrap;
    .month_item {
      width: 33.33%;
      border: 5rpx solid #fff;
    }
  }
}

.hots_wrap {
  .hots_title {
    padding: 20rpx;
    text {
      border-left: 20rpx solid $color;
      padding-left: 20rpx;
      font-size: 34rpx;
      font-weight: 600;
    }
  }
  .hots_content {
    display: flex;
    flex-wrap: wrap;
    .hots_item {
      width: 33.33%;
      border: 5rpx solid #fff;
    }
  }
}
</style>
