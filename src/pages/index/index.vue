<template>
<div class="container">
	<view class='wrap' v-for="(list, wrapIndex) in dateArr" :key="wrapIndex">
		<view>
			<view class='date-show'>
        {{list.year}}年{{list.month}}月
			</view>
		</view>
		<view class='header'>
			<view v-for='(item, index) in date' :key="index" :class='(index == todayIndex) && isTodayWeek ? "weekMark" : ""'>
				{{item}}
				<view></view>
			</view>
		</view>
		<view class='date-box'>
			<view v-for='(item, index) in list.dateArr' :key="index" class="content" :class='isToday == item.isToday ?  "nowDay" : ""'>
				<view class='date-head' :class="isToday > item.isToday ? 'date-cannot' : ''">
          <view>{{item.dateNum}}</view>
				</view>
				<view class='date-weight' v-if="item.dateNum">
          <view v-if="item.weight">有号</view>
          <view v-else>无号</view>
        </view>
			</view>
		</view>
	</view>
</div>
</template>

<script>
export default {
  data () {
    return {
      year: 0,
      month: 0,
      date: ['日', '一', '二', '三', '四', '五', '六'],
      dateArr: [],
      isToday: 0,
      isTodayWeek: false,
      todayIndex: 0
    }
  },
  onLoad: function () {
    let now = new Date()
    this.year = now.getFullYear()
    this.month = now.getMonth() + 1
    this.isToday = new Date(this.year, (this.month - 1), now.getDate()).getTime()
    this.dateInit({many: true})
  },
  onPullDownRefresh () {
    this.pullDown()
    // 刷新完成后关闭
    wx.stopPullDownRefresh()
  },
  onReachBottom () {
    this.pullDown()
  },
  methods: {
    pullDown () {
      this.dateInit({setYear: this.year, setMonth: this.month + 1, many: true})
    },
    calculateDays (year, month) {
      let dateList = {
        dateArr: [], // 需要遍历的日历数组数据
        year: 0,
        month: 0
      }
      let arrLen = 0 // dateArr的数组长度
      let nextMonth = (month + 1) > 11 ? 1 : (month + 1)
      let startWeek = new Date(year + ',' + (month + 1) + ',' + 1).getDay() // 目标月1号对应的星期
      let dayNums = new Date(year, nextMonth, 0).getDate() // 获取目标月有多少天
      let obj = {}
      let num = 0
      arrLen = startWeek + dayNums
      for (let i = 0; i < arrLen; i++) {
        if (i >= startWeek) {
          num = i - startWeek + 1
          obj = {
            isToday: new Date(year, month, num).getTime(),
            dateNum: num,
            weight: num % 5 === 0 ? 1 : 0
          }
        } else {
          obj = {}
        }
        dateList.dateArr[i] = obj
        dateList.year = year
        dateList.month = month + 1
      }
      this.year = year
      this.month = month
      return dateList
    },
    async dateInit ({setYear = '', setMonth = '', num = 3, many = true}) {
      let nowDate = new Date()
      let now = setYear && setMonth ? new Date(setYear, setMonth) : nowDate
      let year = setYear && setMonth && setMonth > 11 ? setYear + 1 : now.getFullYear()
      let month = now.getMonth() // 没有+1方便后面计算当月总天数
      if (many) {
        --month // 月份减一,然后循环中加一, 获取当前月份
        for (let i = 0; i < num; i++) {
          ++month
          // 如果当前月份大于11 重置月份并且年份++
          if (month > 11) {
            month = 0
            year++
          }
          this.dateArr.push(await this.calculateDays(year, month))
        }
      } else {
        this.dateArr.push(await this.calculateDays(year, month))
      }
      let nowWeek = nowDate.getDay()
      this.isTodayWeek = true
      this.todayIndex = nowWeek
    }
  }
}
</script>

<style scoped>
.date-show{
	position: relative;
	width: 250rpx;
	font-family: PingFang-SC-Regular;
	font-size: 40rpx;
	color: #282828;
	text-align: center;
	margin: 59rpx auto 10rpx;
}
.lt-arrow,.rt-arrow{
	position: absolute;
	top: 1rpx;
	width: 60rpx;
	height: 60rpx;
}
.lt-arrow image,.rt-arrow image{
	width: 14rpx;
	height: 26rpx;
}
.lt-arrow{
	left: -110rpx;
	transform: rotate(180deg);
}
.rt-arrow{
	right: -100rpx;
}
.header{
	padding: 10rpx 20rpx;
	display: flex;
}
.header>view{
	flex: 1;
	color: #333;
	font-size: 30rpx;
	text-align: center;
	border-bottom: 1rpx solid #D0D0D0;
}
.weekMark{
	position: relative;
}
.weekMark view{
	position: absolute;
	bottom: -1rpx;
	left: 0;
	width: 100%;
	border-bottom: 2rpx solid #22A7F6;
}
.date-box{
	padding: 10rpx 20rpx;
	display: flex;
  flex-flow: row wrap;
}
.date-box .content {
	color: #020202;
	font-size: 40rpx;
	height: 100rpx;
	padding: 10rpx 0;
	box-sizing: border-box;
	text-align: center;
	flex: 0 0 calc(100% / 7);
	margin-bottom: 10rpx;
}
.date-head{
	font-size: 30rpx;
	font-weight: 600;
}
.date-head.date-cannot {
	color: #999;
}
.date-box .nowDay {
	border-radius: 50%;
	text-align: center;
	color: #fff;
	background-color: #22A7F6;
}
.date-weight{
	font-size: 22rpx;
}
.one{
	position: absolute;
	bottom: 0;
	right: 5rpx;
	width: 20rpx;
	height: 20rpx;
	border-radius: 50%;
	background-color: red;
}
.two{
	position: absolute;
	bottom: 30rpx;
	right: 5rpx;
	width: 20rpx;
	height: 20rpx;
	border-radius: 50%;
	background-color: blue;
}

</style>
