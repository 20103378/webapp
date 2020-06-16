<template>
  <div class="main">
    <el-row :gutter="10">
      <el-col :xs="24" :sm="24">
        <div class="app-container">
          <baidu-map class="map" :center="map.center" :zoom="map.zoom" :map-click="false" @ready="handler">
            <!--缩放-->
            <bm-navigation anchor="BMAP_ANCHOR_TOP_LEFT"></bm-navigation>
            <!--定位-->
            <bm-geolocation anchor="BMAP_ANCHOR_BOTTOM_RIGHT" :showAddressBar="true"
                            :autoLocation="true"></bm-geolocation>
          </baidu-map>
        </div>
      </el-col>
    </el-row>
  </div>
</template>
<script>
  import echarts from 'echarts'
  import countTo from 'vue-count-to'
  import resize from '@/components/Charts/mixins/resize'

  export default {
    name: 'Dashboard',
    components: { countTo },
    mixins: [resize],
    data() {
      return {
        map: {
          center: { lng: 122.15, lat: 30.13 },
          zoom: 12,
          show: true,
          dragging: true,
          chart: null
        }
      }
    },
    computed: {
      user() {
        return this.$store.state.account.user
      }
    },
    mounted() {
      // this.initIndexData('112132324324324524')
    },
    methods: {
      initIndexData: function(deviceId) {
        this.$get('cable/gasConcentration/getGasConcentrationByDeviceId?deviceId='+deviceId).then((r) => {
          const data = r.data.data
          debugger
          const color = ['#c23531', '#2f4554', '#61a0a8', '#d48265', '#91c7ae', '#749f83', '#ca8622', '#bda29a', '#6e7074', '#546570', '#c4ccd3']
          const option = {
            title: {
              text: '',
              left: 'center'
            },
            tooltip: {
              trigger: 'axis',
              padding: [2, 10],
              textStyle: {
                fontSize: 16
              }
            },
            xAxis: {
              name: '时间',
              type: 'category',
              splitLine: {
                show: false
              },
              axisLine: {
                lineStyle: {
                  width: 3
                }
              },
              axisTick: {
                show: true,
                alignWithLabel: true,
                lineStyle: {
                  width: 3
                }
              },
              axisLabel: {
                rotate: 45,
                fontWeight: 'bold',
                interval: 2,
                formatter:function(value){
                  var result = "";//拼接加\n返回的类目项
                  var maxLength = 11;//每项显示文字个数
                  var valLength = value.length;//X轴类目项的文字个数
                  var rowNumber = Math.ceil(valLength / maxLength); //类目项需要换行的行数
                  if (rowNumber > 1)//如果文字大于3,
                  {
                    for (var i = 0; i < rowNumber ; i++) {
                      var temp = "";//每次截取的字符串
                      var start = i * maxLength;//开始截取的位置
                      var end = start + maxLength;//结束截取的位置
                      temp = value.substring(start, end) + "\n";
                      result += temp; //拼接生成最终的字符串
                    }
                    return result ;
                  }
                  else {
                    return value;
                  }
                },
              },
              data: data.time
            },
            grid: {
              left: '6%',
              right: '12%',
              bottom: '4%',
              containLabel: true
            },
            yAxis: {
              type: 'value',
              name: '浓度',
              splitLine: {
                show: false
              },
              axisTick: {
                lineStyle: {
                  width: 3
                }
              },
              axisLabel: {
                fontWeight: 'bold'
              }
            },
            legend: {
              data: data.legend,
              textStyle: {
                color: '#1ff',
                fontSize: '15'
              }
            },
            series: [{
              name: 'CH4',
              type: 'line',
              symbol: 'circle',
              symbolSize: 10,
              smooth: true,
              animationDuration: 2000,
              lineStyle: {
                normal: {
                  width: 3,
                  color: color[4],
                }
              },
              connectNulls: true,
              data: data.data.CH4
            }, {
              name: 'CL2',
              type: 'line',
              symbol: 'circle',
              symbolSize: 10,
              smooth: true,
              animationDuration: 2000,
              lineStyle: {
                normal: {
                  width: 3,
                  color: color[3],
                }
              },
              connectNulls: true,
              data: data.data.CL2
            }, {
              name: 'H2S',
              type: 'line',
              symbol: 'circle',
              symbolSize: 10,
              smooth: true,
              animationDuration: 2000,
              lineStyle: {
                normal: {
                  color: color[2],
                  width: 3,
                }
              },
              connectNulls: true,
              data: data.data.H2S
            }, {
              name: 'O2',
              type: 'line',
              symbol: 'circle',
              symbolSize: 10,
              smooth: true,
              animationDuration: 2000,
              lineStyle: {
                normal: {
                  color: color[1],
                  width: 3,
                }
              },
              connectNulls: true,
              data: data.data.O2
            }
            ]
          }
          this.chart.setOption(option)
        })
      },
      handler({ BMap, map }) {
        let me = this
        console.log(BMap, map)
        // 鼠标缩放
        // map.enableScrollWheelZoom(true)
        var point = new BMap.Point(122.15, 30.13)
        map.centerAndZoom(point, 12)
        //添加地图类型控件
        map.addControl(new BMap.MapTypeControl({
          mapTypes: [
            BMAP_NORMAL_MAP,
            BMAP_HYBRID_MAP
          ]
        }))
        map.setCurrentCity('浙江')          // 设置地图显示的城市 此项是必须设置的
        var params = {}
        me.$get('cable/device/list', { ...params }).then((r) => {
          const data = r.data.data
          console.log(data)
          data.forEach(function(item) {
            // debugger
            var marker = new BMap.Marker(new BMap.Point(item.longitude, item.latitude)) // 创建点
            map.addOverlay(marker)
            var label = new BMap.Label(
              '设备名称：' + item.deviceId + '精度：' + marker.getPosition().lng + ',维度：' + marker.getPosition().lat, { offset: new BMap.Size(10, -30) })
            label.setStyle({
              border: '1px solid #FF6A00',
              borderRadius: '5px',
              padding: '5px',
              color: '#333',
              backgroundColor: 'white'
            }) //对label 样式隐藏
            // this.timer = setInterval(function () {
            me.$get('cable/gasConcentration/getRecentGasConcentrationByDeviceId?deviceId=' + item.deviceId).then((r) => {
              console.log(r)
              label.setContent('')
              label.setContent('设备名称：' + item.deviceId
                + '  （精度,纬度):' + '(' + marker.getPosition().lng + ',' + marker.getPosition().lat + ')'
                + '  最新浓度：' + JSON.stringify(r.data))
            })
            // }, item.period);

            marker.setLabel(label)
            label.addEventListener('click', function() { //点击label隐藏label
              label.setStyle({
                display: 'none'
              })
            })
            var content = '<div id="visit-count-chart" style="width:480px;height: 480px" />'
            var opts = {
              width: 480,    // 信息窗口宽度
              height: 480,    // 信息窗口高度
            }
            marker.addEventListener('mouseover', function(e) {
              var p = e.target
              var point = new BMap.Point(p.getPosition().lng, p.getPosition().lat)
              var infoWindow = new BMap.InfoWindow(content, opts)  // 创建信息窗口对象
              map.openInfoWindow(infoWindow, point) //开启信息窗口
              me.chart = echarts.init(document.getElementById('visit-count-chart'))
              me.initIndexData(item.deviceId);
            });

          })

        })
      }
    }
  }
</script>
<style lang="scss">
  .main {
    .el-card {
      border: none;
      border-radius: 2px;
    }

    .el-table.server-table {
      th {
        height: 2.7rem;
        padding: 7px 0;
        border-right: none;
      }

      td {
        padding: 7px 0;
        border-right: none;
      }
    }

    .count-header {
      padding-left: 1rem;
      height: 120px;
      line-height: 120px;
      text-align: left;
      vertical-align: center;

      img {
        width: 3.8rem;
        margin-top: 1.8rem;
        vertical-align: top;
      }

      span {
        font-size: 15px;
        color: #606266;
        font-weight: 600;

        &.des {
          margin-left: 9px;
        }
      }
    }

    .todayIp {
      color: #27c6da !important;
    }

    .todayVisit {
      color: #fc573b !important;
    }

    .totalVisit {
      color: #be63f9 !important;
    }
  }
</style>
<style lang="scss" scoped>
  .map {
    width: 100%;
    height: 600px;
    overflow: hidden;
    margin: 0;
    font-family: "Arial Black";
  }
  .main {
    padding: 10px;

    .app-container {
      margin: 0 0 10px 0;
    }
  }
</style>
