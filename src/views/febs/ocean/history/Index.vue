<template>
  <div class="main">
    <el-row :gutter="10">
      <el-col :xs="24" :sm="24">
        <div class="app-container" style="height: 100px">
          <div class="filter-container">
            <el-select v-model="queryParams.deviceId" placeholder="请选择设备" class="filter-item search-item" >
              <el-option v-for="item in options" :key="item.id" :label="item.deviceName" :value="item.deviceId" >
                <span style="float: left">{{item.deviceName}}---{{item.deviceId}}</span>
                <span style="float: right">{{item.id}}</span>
              </el-option>
            </el-select>
            <el-date-picker
              v-model="queryParams.timeRange"
              :default-time="['12:00:00', '08:00:00']"
              :picker-options="pickerOptions"
              :start-placeholder="$t('table.startTime')"
              :end-placeholder="$t('table.endTime')"
              value-format="yyyy-MM-dd HH:mm:ss"
              class="filter-item search-item date-time-range-item"
              type="datetimerange"
            />
            <el-button class="filter-item" type="primary" plain @click="search">
              {{ $t('table.search') }}
            </el-button>
            <el-button class="filter-item" type="warning" plain @click="reset">
              {{ $t('table.reset') }}
            </el-button>
          </div>
        </div>
      </el-col>
    </el-row>
    <el-row :gutter="10">
      <el-col :xs="24" :sm="24">
        <div class="app-container">
          <div id="visit-count-chart" style="width: 100%;height: 500px" />
        </div>
      </el-col>
    </el-row>
  </div>
</template>
<script>
  import echarts from 'echarts'
  import { parseTime } from '@/utils'
  import countTo from 'vue-count-to'
  import resize from '@/components/Charts/mixins/resize'

  export default {
    name: 'Dashboard',
    components: { countTo },
    mixins: [resize],
    data() {
      return {
        options: [],
        chart: null,
        queryParams: {},
        pickerOptions: {
          shortcuts: [{
            text: '最近一周',
            onClick: function (picker) {
              var end = new Date();
              var start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 7);
              picker.$emit('pick', [start, end]);
            }
          }, {
            text: '最近一个月',
            onClick: function (picker) {
              var end = new Date();
              var start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 30);
              picker.$emit('pick', [start, end]);
            }
          }, {
            text: '最近三个月',
            onClick: function (picker) {
              var end = new Date();
              var start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 90);
              picker.$emit('pick', [start, end]);
            }
          }]
        },
      }
    },
    computed: {
      user() {
        return this.$store.state.account.user
      },
      avatar() {
        return require(`@/assets/avatar/${this.user.avatar}`)
      }
    },
    mounted() {
      this.getDeviceOptions();
      this.chart = echarts.init(document.getElementById('visit-count-chart'))
      // this.initIndexData('112132324324324524')
    },
    methods: {
      search() {
        this.fetch({
          ...this.queryParams,
        })
      },
      reset() {
        this.queryParams = {}
        this.search()
      },
        getDeviceOptions: function () {
        this.$get('cable/device/list').then((r) => {
          this.options = r.data.data;
        });
      },
      fetch(params = {}) {

        if (this.queryParams.timeRange) {
          params.startTime = this.queryParams.timeRange[0]
          params.endTime = this.queryParams.timeRange[1]
        }
        // params.deviceId = '112132324324324524';
        debugger
        // this.$get('cable/device', {
        //   ...params
        // }).then((r) => {
        //   const data = r.data.data
        // })
      // },

      // initIndexData: function() {
        this.$get('cable/gasConcentration/getGasConcentrationTrafficByParam',{...params}).then((r) => {
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
                  width: 2
                }
              },
              axisTick: {
                show: true,
                alignWithLabel: true,
                lineStyle: {
                  width: 2
                }
              },
              axisLabel: {
                rotate: 45,
                fontWeight: 'bold',
                interval: data.count,
                formatter:function(value){
                  var result = "";//拼接加\n返回的类目项
                  var maxLength = 11;//每项显示文字个数
                  var valLength = value.length;//X轴类目项的文字个数
                  var rowNumber = Math.ceil(valLength / maxLength); //类目项需要换行的行数
                  if (rowNumber > 1)//如果文字大于2,
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
                  width: 2
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
                  width: 2,
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
                  width: 2,
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
                  width: 2,
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
                  width: 2,
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
  }
</style>
<style lang="scss" scoped>
  .main {
    padding: 10px;
    .app-container {
      margin: 0 0 10px 0;
    }
    .user-container {
      padding: 15px;
    }
    .project-wrapper {
      padding: 0;
      .project-header {
        padding: 18px;
        margin-bottom: 16px;
      }
      table {
        width: 100%;
        border-collapse: collapse;
        td {
          width: 50%;
          border-top: 1px solid #f1f1f1;
          border-bottom: 1px solid #f1f1f1;
          padding: .7rem;
          .project-avatar-wrapper {
            display:inline-block;
            float:left;
            margin-right:.7rem;
            .project-avatar {
              color: #42b983;
              background-color: #d6f8b8;
            }
          }
          &:nth-child(odd) {
            border-right: 1px solid #f1f1f1;
          }
        }
      }
    }
  }
</style>
