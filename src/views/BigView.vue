<template>
  <div class="conter">
    <dv-full-screen-container>
      <dv-border-box-11 title="大数据系统">
        <div class="load"
             v-if="!this.loadinge">
          <dv-loading>Loading...</dv-loading>
        </div>
        <!-- 遮罩层 -->
        <div class="shade"
             v-if="!this.loadinge">1</div>

        <div class="baidu-map">
          <div class="left">
            <!-- <div class="one">
              <dv-border-box-13></dv-border-box-13>
            </div> -->
            <div class="one">
              <dv-border-box-1 class="border-box">
                <dv-capsule-chart :config="categoryCapsule"
                                  class="charts" />
              </dv-border-box-1>
            </div>
            <div class="two">
              <dv-border-box-10 :color="['#5378a2', '#FFF']">
                <dv-scroll-board :config="config2" />
              </dv-border-box-10>
            </div>
            <div class="three">
              <dv-border-box-8 :color="['#5378a2', '#FFF']">
                <dv-scroll-ranking-board :color="['#5378a2', '#FFF']"
                                         :config="categoryConical" />
              </dv-border-box-8>
            </div>
          </div>
          <div class="right">
            <div class="fore">
              <dv-border-box-1 class="border-box">
                <dv-active-ring-chart :config="categoryChart"
                                      style="width:300px;height:300px;margin: -20px auto;" />
                <!-- <dv-active-ring-chart :config="categoryChart" /> -->
              </dv-border-box-1>
            </div>
            <div class="four">
              <dv-border-box-10 :color="['#5378a2', '#FFF']"></dv-border-box-10>
            </div>
            <div class="six">
              <dv-border-box-8 :reverse="true"
                               :color="['#5378a2', '#FFF']">
                <dv-conical-column-chart :config="categoryConical" />
              </dv-border-box-8>
            </div>
          </div>
          <!-- 监管时间 -->
          <div class="monitor">
            <div class="txt">系统监测时间</div>
            <div class="reader">
              <div>
                <i>{{ day }}</i>天
              </div>
              <div>
                <i>{{ hour }}</i>时
              </div>
              <div>
                <i>{{ minute }}</i>分
              </div>
              <div>
                <i>{{ second }}</i>秒
              </div>
            </div>
            <div style="width: 100%;height: 100%;">
              <dv-flyline-chart-enhanced :config="config1"
                                         :dev="true"
                                         style="width:100%;height:100%;" />
            </div>
          </div>
        </div>

        <!-- 当前日期+当前星期 -->
        <div class="datae">
          <router-link to="/dashboard/dashboard"
                       style="color: #0674be">返回首页</router-link>
          {{ nowDate + ' ' + nowWeek }}
        </div>
        <!-- 当前时间 -->
        <div class="timer">
          {{ nowTime }}
        </div>
      </dv-border-box-11>
    </dv-full-screen-container>
  </div>
</template>
<script>
// import { getTokenTime } from '@/utils/auth'
import { config1, config2, config3, config4, config5, config6, config7 } from '../util/scrollDatas'
import echarts from 'echarts'
import 'echarts/map/js/china'
import { getDashboard } from "@/api/dashboard";
export default {
  name: 'iot',
  data () {
    return {
      categoryCapsule: {},
      categoryChart: {},
      categoryConical: {},
      config3: config3,
      config2: config2,
      config1: config1,
      categoryEcharts: null,
      categoryOption: {
        title: {
          text: '分类下文章数量',
          x: 'center'
        },
        tooltip: {
          trigger: 'item',
          formatter: '{a} <br/>{b} : {c} ({d}%)'
        },
        legend: {
          left: 'center',
          top: 'bottom',
          data: []
        },
        series: [
          {
            name: '文章数量',
            type: 'pie',
            radius: [30, 110],
            roseType: 'area',
            data: []
          }
        ]
      },
      center: { lng: 118.614312, lat: 24.890586 },
      zoom: 18,
      config: config3,
      config7: config7,
      // 动画开光
      loadinge: true,
      nowDate: '', // 当前日期
      nowTime: '', // 当前时间
      nowWeek: '', // 当前星期

      day: '0', //当前天数
      hour: '0', //当前小时
      minute: '0', //当前分钟
      second: '0', //当前秒数
      dialogTableVisible: false
    }
  },
  mounted () {
    this.currentTime()

  },
  created () {
    this.getData()
  },
  // 销毁定时器
  beforeDestroy () {
    console.log("销毁定时器");
    clearInterval(this.getDate) // 在Vue实例销毁前，清除时间定时器
    clearInterval(this.monitortime)
  },
  methods: {
    getData () {
      getDashboard().then(res => {
        // 渲染分类数据 one
        this.categoryCapsule = {
          data: res.data.category.series,
          colors: ['#e062ae', '#fb7293', '#e690d1', '#32c5e9', '#96bfff'],
          unit: '篇',
          showValue: true
        }
        this.categoryChart = {
          data: res.data.category.series,
          lineWidth: 20,
          activeRadius: 110,
          radius: 100
        }
        this.pv = res.data.pv
        this.uv = res.data.uv
        this.blogCount = res.data.blogCount
        this.commentCount = res.data.commentCount
        // 渲染标签数据 fore
        this.categoryConical = {
          data: res.data.tag.series,
          img: [
            'http://datav.jiaminghi.com/img/conicalColumnChart/1st.png',
            'http://datav.jiaminghi.com/img/conicalColumnChart/2st.png',
            'http://datav.jiaminghi.com/img/conicalColumnChart/3st.png',
            'http://datav.jiaminghi.com/img/conicalColumnChart/4st.png',
            'http://datav.jiaminghi.com/img/conicalColumnChart/5st.png',
            'http://datav.jiaminghi.com/img/conicalColumnChart/6st.png',
            'http://datav.jiaminghi.com/img/conicalColumnChart/7st.png',
            'http://datav.jiaminghi.com/img/conicalColumnChart/7st.png',
            'http://datav.jiaminghi.com/img/conicalColumnChart/7st.png',
            'http://datav.jiaminghi.com/img/conicalColumnChart/7st.png',
            'http://datav.jiaminghi.com/img/conicalColumnChart/7st.png',
            'http://datav.jiaminghi.com/img/conicalColumnChart/7st.png',
            'http://datav.jiaminghi.com/img/conicalColumnChart/7st.png',
            'http://datav.jiaminghi.com/img/conicalColumnChart/7st.png'
          ]
        }
        //渲染分类数据
        // this.categoryOption.legend.data = res.data.category.legend
        // this.categoryOption.series[0].data = res.data.category.series
        // this.initCategoryEcharts()
        //渲染标签数据
        // this.tagOption.legend.data = res.data.tag.legend
        // this.tagOption.series[0].data = res.data.tag.series
        // this.initTagEcharts()
        //渲染访客地图数据
        // let mapData = this.convertData(res.data.cityVisitor)
        // this.mapOption.series[1].data = mapData
        // this.mapOption.series[2].data = mapData.splice(0, 5)
        // this.initMapEcharts()
        //渲染一周访问量数据
        // this.visitRecordOption.xAxis.data = res.data.visitRecord.date
        // this.visitRecordOption.series[0].data = res.data.visitRecord.pv
        // this.visitRecordOption.series[1].data = res.data.visitRecord.uv
        // this.initVisitRecordEcharts()
      })
    },

    handler ({ BMap, map }) {
      let that = this
      // console.log(BMap, map);
      // 经度
      this.center.lng = 118.614312
      // 维度
      this.center.lat = 24.890586
      // 地图放大等级
      this.zoom = 14
      map.enableScrollWheelZoom() // 启用滚轮放大缩小，默认禁用
      map.enableContinuousZoom() // 可拖拽
      // 地图个性化样式
      try {
        map.setMapStyleV2({
          styleId: '你在引入AK账号的百度地图发布的个性化样式'
        })
      } catch (erro) {
        return false
      }
      // 这个是遮罩层和加载动画的开关 默认一进到页面加载动画和遮罩层是开启的
      // 还有一个作用就是网络不好时，百度地图加载失败时。会出现白屏现象，加上这个遮罩层和加载动画会显得比较          流畅
      setTimeout(() => {
        this.loadinge = true
      }, 3000)
    },
    // 每0.5秒执行一次
    currentTime () {
      setInterval(this.getDate, 500)
      setInterval(this.monitortime, 1000)
    },
    // 获取当前日期时间的方法
    getDate () {
      var _this = this
      //获取年
      let yy = new Date().getFullYear()
      //获取月,注意时间是0-11，0代表1月份
      let mm = new Date().getMonth() + 1
      // 获取具体哪一天
      let dd = new Date().getDate()
      // 获取星期 0-6 0是星期天
      let week = new Date().getDay()
      //获取小时
      let hh = new Date().getHours()
      //获取分钟
      let ms =
        new Date().getSeconds() < 10
          ? '0' + new Date().getSeconds()
          : new Date().getSeconds()
      // 获取秒钟
      let mf =
        new Date().getMinutes() < 10
          ? '0' + new Date().getMinutes()
          : new Date().getMinutes()
      // 获取星期
      if (week == 1) {
        this.nowWeek = '星期一'
      } else if (week == 2) {
        this.nowWeek = '星期二'
      } else if (week == 3) {
        this.nowWeek = '星期三'
      } else if (week == 4) {
        this.nowWeek = '星期四'
      } else if (week == 5) {
        this.nowWeek = '星期五'
      } else if (week == 6) {
        this.nowWeek = '星期六'
      } else {
        this.nowWeek = '星期日'
      }
      _this.nowTime = hh + ':' + mf + ':' + ms
      _this.nowDate = yy + '/' + mm + '/' + dd
    },
    // 监测时间
    monitortime () {
      // var _that = this
      // // 当前时间
      // let x = new Date().getTime()
      // // console.log('当前时间戳', x)
      // // 点击登录存的时间
      // let y = getTokenTime()
      // // console.log('登录时间戳', y)
      // // 登录之后的监测时间
      // let z = x - y
      // // console.log('监测时间', z)
      // _that.day = parseInt(z / 1000 / 60 / 60 / 24) //日
      // _that.hour = parseInt((z / 1000 / 60 / 60) % 24) //时
      // _that.minute = parseInt((z / 1000 / 60) % 60) //分
      // _that.second = parseInt((z / 1000) % 60) //秒
    }
  }
}
</script>
<style lang="scss" scoped>
.conter {
  background-color: rgb(9, 18, 32);
  width: 100%;
  height: 100%;
  // box-sizing: border-box;
  // 最大边框的样式
  ::v-deep #dv-full-screen-container {
    height: 100% !important;
    .dv-border-box-11 .border-box-content {
      // background-color: skyblue;
      padding: 10px 9px 10px;
      overflow: hidden;
      border-radius: 55px;
      position: relative;
      .load {
        position: absolute;
        top: 40%;
        left: 50%;
        // margin-left: -37px;
        transform: translateX(-50%);
        z-index: 3;
      }
      // 遮罩层
      .shade {
        width: 100%;
        // width: 100%; 取100% margin 右边不会生效 要改成width=99%才有效果
        height: 100%;
        box-sizing: border-box !important;
        // background-color: skyblue;
        margin: 60px 8px 10px;
        background-color: rgb(9, 18, 32);
        position: absolute;
        z-index: 1;
        top: 0;
        left: 0;
      }
      // 地图
      .baidu-map {
        width: 100%;
        height: 100%;
        border-radius: 5px;
        // z-index: 999;
        display: flex;
        // justify-content: space-between;
        // 去除百度地图的图标
        position: relative;
        .anchorBL {
          display: none !important;
        }
        .left {
          width: 350px;
          height: 100%;
          position: absolute;
          top: 60px;
          left: 2%;
          // background-color: skyblue;
          // margin-left: 50px;
          .one {
            height: 280px;
            // background-color: skyblue;
          }
          .two {
            height: 250px;
            background-color: rgb(2, 52, 126);
            border-radius: 3px;
            margin: 40px 0 40px;
          }
          .three {
            height: 250px;
            background-color: rgb(2, 52, 126);
            border-radius: 3px;
          }
        }
        .right {
          width: 350px;
          height: 100%;
          position: absolute;
          top: 60px;
          right: 2%;
          // background-color: skyblue;
          // margin-right: 50px;
          .fore {
            height: 280px;
            // background-color: skyblue;
          }
          .four {
            height: 250px;
            background-color: rgb(2, 52, 126);
            border-radius: 3px;
            margin: 40px 0 40px;
          }
          .six {
            height: 250px;
            background-color: rgb(2, 52, 126);
            border-radius: 3px;
          }
        }
        // 监测时间
        .monitor {
          width: 320px;
          height: 95px;
          // background-color: skyblue;
          position: absolute;
          top: 60px;
          left: 50%;
          transform: translateX(-50%);
          .txt {
            width: 135px;
            color: #fff;
            font-size: 22px;
            margin: 2px auto;
          }
          .reader {
            color: #afafaf;
            display: flex;
            justify-content: space-between;
            margin-top: 12px;
            div {
              display: flex;
              align-items: baseline;
              i {
                font-style: normal;
                display: block;
                width: 50px;
                height: 50px;
                box-sizing: border-box;
                font-size: 20px;
                color: #fff;
                margin-right: 8px;
                background-color: rgb(5, 47, 115);
                border: 3px solid rgb(6, 116, 190);
                border-radius: 5px;
                line-height: 44px;
                text-align: center;
              }
            }
          }
        }
      }
      // 左上角日期星期
      .datae {
        // background-color: skyblue;
        font-size: 18px;
        position: absolute;
        top: 7px;
        left: 200px;
        color: #fff;
      }
      // 右上角的当前时间
      .timer {
        // background-color: skyblue;
        font-size: 18px;
        position: absolute;
        top: 7px;
        right: 200px;
        color: #fff;
      }
    }
  }
}
</style>