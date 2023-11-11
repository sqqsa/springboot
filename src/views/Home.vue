<template>
  <div>
    <el-row  :gutter="10" style="margin-bottom: 40px">
      <el-col :span="6">
        <el-card>
          <div style="color:#409EFF"><i class="el-icon-user-solid" />员工总数</div>
          <div style="padding: 10px 0;text-align: center; font-weight: bold">
            <span> {{totalNum}}</span>

          </div>
        </el-card>
      </el-col>
      <el-col :span="6">
        <el-card>
          <div style="color: #67C23A"><i class="el-icon-star-on" />状态正常</div>
          <div style="padding: 10px 0;text-align: center; font-weight: bold">
            <span> {{ normalNum }}</span>
          </div>
        </el-card>
      </el-col>
      <el-col :span="6">
        <el-card>
          <div style="color: #F56C6C"><i class="el-icon-s-goods" />状态异常</div>
          <div style="padding: 10px 0;text-align: center; font-weight: bold">
            <span> {{ anomalyNum }}</span>
          </div>
        </el-card>
      </el-col>
      <el-col :span="6">
        <el-card>
          <div style="color: #8c939d"><i class="el-icon-s-cooperation" />部门总数</div>
          <div style="padding: 10px 0;text-align: center; font-weight: bold">
            <span> {{ deptNum }}</span>
          </div>
        </el-card>
      </el-col>
    </el-row>

    <div>
      <div style="width: 800px; float: right" >
        <el-calendar>
          <template
              slot="dateCell"
              slot-scope="{date, data}">
            <p :class="data.isSelected ? 'is-selected' : ''">
              {{ data.day.split('-').slice(1).join('-') }} {{ data.isSelected ? '✔️' : ''}}
            </p>
          </template>
        </el-calendar>
      </div>
      <el-row style="height: 400px;">
        <el-col :span="6">
          <div id="main" style="width: 700px; height: 400px"></div>
        </el-col>
      </el-row>

      <el-row>
        <el-col :span="6">
          <div id="pie" style="width: 700px; height: 400px"></div>
        </el-col>
      </el-row>




    </div>


  </div>




</template>

<script>

import * as echarts from 'echarts'
import request from "@/utils/request";
export default {
  name: "Home",
  data(){
    return{

      totalNum: 0,
      normalNum: 0,
      anomalyNum: 0,
      deptNum: 0

    }
  },
  created() {
    this.displayData()
  },
  methods: {
    displayData() {
      this.request.get("/echarts/countData").then(res =>{
        this.totalNum = res.data.totalNum
        this.normalNum = res.data.normalNum
        this.anomalyNum = res.data.totalNum - res.data.normalNum
        this.deptNum = res.data.deptNum
      })
    }

  },
  mounted() {
    //页面元素渲染之后再触发
    var option = {
      title: {
        text: '2023年员工入职情况',
        subtext: '趋势图',
        left: 'center'
      },
      xAxis: {
        type: 'category',
        data: ["第一季度", "第二季度", "第三季度", "第四季度"]
      },
      yAxis: {
        type: 'value'
      },
      series: [
        {
          data: [],
          type: 'line'
        },
        {
          data: [],
          type: 'bar'
        }
      ]
    };
    //饼图
    var pieOption = {
      title: {
        text: '各季度员工入职数量统计',
        subtext: '比例图',
        left: 'center'
      },
      tooltip: {
        trigger: 'item'
      },
      legend: {
        orient: 'vertical',
        left: 'left'
      },
      series: [
        {
          type: 'pie',
          radius: '60%',
          label:{            //饼图图形上的文本标签
            normal:{
              show:true,
              position:'inner', //标签的位置
              textStyle : {
                fontWeight : 300 ,
                fontSize : 14,    //文字的字体大小
                color: "#fff"
              },
              formatter:'{d}%'
            }
          },
          data: [],  // 填空
          emphasis: {
            itemStyle: {
              shadowBlur: 10,
              shadowOffsetX: 0,
              shadowColor: 'rgba(0, 0, 0, 0.5)'
            }
          }
        }
      ]
    };
    var pieDom = document.getElementById('pie');
    var pieChart = echarts.init(pieDom);

    this.request.get("/echarts/members").then(res => {

      pieOption.series[0].data = [
        {name: "第一季度", value: res.data[0]},
        {name: "第二季度", value: res.data[1]},
        {name: "第三季度", value: res.data[2]},
        {name: "第四季度", value: res.data[3]},
      ]
      pieChart.setOption(pieOption)
    })

    var chartDom = document.getElementById('main');
    var myChart = echarts.init(chartDom);

    this.request.get("/echarts/members").then(res => {
      // 填空
      // option.xAxis.data = res.data.x
      option.series[0].data = res.data
      option.series[1].data = res.data
      // 数据准备完毕之后再set
      myChart.setOption(option);

    })
  }

}
</script>

<style>

</style>