<template>
    <div class="charts">
        <el-row class="sel-row">
            <el-col :span="2" class="sel-label">选择城市：</el-col>
            <el-col :span="3">
                <el-select v-model="sel" filterable clearable placeholder="请选择城市" @change="getData">
                    <el-option v-for="(item,index) in options" :label="item.label" :value="item.value" :key="index">
                    </el-option>
                </el-select>
            </el-col>
        </el-row>
        <el-row :gutter="20">
            <el-col :span="4">
                <el-card class="box-card">
                    {{'实时温度：' + (cityData.wendu || 'XX') + '℃'}}
                </el-card>
            </el-col>
            <el-col :span="4">
                <el-card class="box-card">
                    {{'空气质量：' + ( cityData.aqi || '暂无' )}}
                </el-card>
            </el-col>
            <el-col :span="16">
                <el-card class="box-card">
                    {{cityData.ganmao || '暂无提示'}}
                </el-card>
            </el-col>
        </el-row>
        <div class="chart-wrap">
            <el-row>
                <el-col :span="24">
                    <el-card class="box-card">
                        <div slot="header" class="clearfix">
                            <span>近五天温度走势</span>
                            <i class="fa fa-exchange ct" @click="isChart = !isChart"></i>
                        </div>
                        <div v-show="isChart" class="animated fadeIn">
                            <!-- 这里是折线图组件 -->
                            <line-chart :lineData="lineData"></line-chart>
                        </div>
                        <div v-show="!isChart" class="animated fadeIn">
                            <!-- 这里是表格组件 -->
                            <el-table stripe :data="tableData" style="width: 100%">
                                <el-table-column prop="date" label="日期"></el-table-column>
                                <el-table-column prop="high" label="最高气温（℃）"></el-table-column>
                                <el-table-column prop="low" label="最低气温（℃）"></el-table-column>
                            </el-table>
                        </div>
                    </el-card>
                </el-col>
            </el-row>
        </div>
    </div>
</template>
<script>
// 引入城市列表
import arr from '../assets/data/citylist.json'
// 引入自己写的折线图组件
import lineChart from './common/Line'

export default {
    name: 'charts',
    components: {
        'line-chart': lineChart
    },
    data() {
        return {
            cityData: {},
            options: arr,
            sel: '101010100',
            lineData: {
                xAxis: [],
                series1: [],
                series2: []
            },
            isChart: true,
            tableData: []
        }
    },
    mounted() {
        this.getData();
    },
    methods: {
        getData() {
            this.$http.jsonp('http://wthrcdn.etouch.cn/weather_mini', {
                params: {
                    citykey: this.sel
                }
            }).then(res => {
                if (res.body.status === 1000) {
                    let data = res.body.data;
                    this.cityData = data;
                    this.renderChart(data.forecast);
                    this.renderTable(data.forecast);
                }
            }, res => {
                console.log(res)
            })
        },
        renderChart(data) {
            this.lineData.xAxis = [];
            this.lineData.series1 = [];
            this.lineData.series2 = [];
            data.forEach((v, i) => {
                this.lineData.xAxis.push(v.date);
                this.lineData.series1.push(v.high.match(/\d/g).join(''));
                this.lineData.series2.push(v.low.match(/\d/g).join(''));
            })
        },
        renderTable(data) {
            this.tableData = [];
            data.forEach((v, i) => {
                var obj = {
                    date: v.date,
                    high: v.high.match(/\d/g).join(''),
                    low: v.low.match(/\d/g).join('')
                };
                this.tableData.push(obj);
            })
        }
    }
}
</script>
<style scoped lang="less">
.sel-row {
    margin-bottom: 20px;
    .sel-label {
        line-height: 36px;
    }
}

.chart-wrap {
    margin-top: 20px;
}

.el-card__header {
    padding: 10px 20px !important;
}

.ct {
    float: right;
    line-height: 21px;
    &:hover {
        color: #1ab394;
        cursor: pointer;
    }
}
</style>
