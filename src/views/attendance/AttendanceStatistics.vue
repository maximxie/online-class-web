<template>
<div style="width: 100%">
  <el-row :gutter="10" style="margin-bottom: 60px">
    <el-col :span="8">
      <el-card style="color: #409EFF">
        <div><i class="el-icon-user-solid"/> 历史发起次数</div>
        <div class="el-card-div">{{ num1 }}</div>
      </el-card>
    </el-col>
    <el-col :span="8">
      <el-card style="color: #F56C6C">
        <div><i class="el-icon-money"/> 最近五次平均到课率</div>
        <div class="el-card-div">{{ (parseFloat(num2)*100).toFixed(2) }}%</div>
      </el-card>
    </el-col>
    <el-col :span="8">
      <el-card style="color: #67C23A">
        <div><i class="el-icon-bank-card"/> 最近五次平均旷课率</div>
        <div class="el-card-div">{{ (parseFloat(num3)*100).toFixed(2) }}%</div>
      </el-card>
    </el-col>
  </el-row>
  <div id="main"></div>
</div>
</template>

<script>
import * as echarts from "echarts";
import {mapGetters} from "vuex";

export default {
  name: "AttendanceStatistics",
  data(){
    return{
    }
  },
  computed:{
    ...mapGetters({
      list: 'attendanceStore/statisticList',
      num1: 'attendanceStore/historyNum',
      num2: 'attendanceStore/scheduleNum',
      num3: 'attendanceStore/truancyRate'
    })
  },
  created() {

  },
  mounted() {
    this.$store.dispatch('attendanceStore/getStatisticList',{id:this.$route.params.courseId}).then(res=>{
      var option = {
        title: {
          text: '显示最近5次签到记录(%)'
        },
        tooltip: {
          trigger: 'axis'
        },
        legend: {
          data: ['到课率', '旷课率']
        },
        grid: {
          left: '3%',
          right: '4%',
          bottom: '3%',
          containLabel: true
        },
        toolbox: {
          feature: {
            saveAsImage: {}
          }
        },
        xAxis: {
          type: 'category',
          boundaryGap: false,
          data: []
        },
        yAxis: {
          type: 'value'
        },
        series: [
          {
            name: '旷课率',
            type: 'line',
            data: []
          },
          {
            name: '到课率',
            type: 'line',
            data: []
          }
        ]
      };
      this.list.forEach((item,i) => {
        option.xAxis.data[i] = item.sign_name
        option.series[0].data[i] = (parseFloat(item.truancy*100)).toFixed(2)
        option.series[1].data[i] = (parseFloat(item.schedule*100)).toFixed(2)
      })
      var chartDom = document.getElementById('main');
      var myChart = echarts.init(chartDom);
      myChart.setOption(option);
    });
  },
  methods:{
  }
}
</script>

<style scoped>

.el-card-div{
  padding: 10px 0;
  text-align: center;
  font-weight: bold;
}
#main{
  width: 1200px;
  height: 450px;
}
</style>
