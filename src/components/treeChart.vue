<!--
 * @Description:
 * @Version: 2.0
 * @Author: yichuanhao
 * @Date: 2023-04-23 11:49:25
 * @LastEditors: yichuanhao 1274816963@qq.com
 * @LastEditTime: 2023-06-17 09:10:17
-->
<template>
  <div class="treeCharts">
    <div id="chart" :style="{ width: '100vw', height: '700px' }"></div>
    <div class="goBack" @click="changeGoBack" v-if="isShowOtherTree">
      <i class="el-icon-back"></i>
    </div>
    <img src="/assets/leftTop.png" class="leftTop-png" />
    <div class="fixed" @click="drawer = true">
      <i class="el-icon-s-operation"></i>
    </div>
    <el-drawer :visible.sync="drawer" direction="rtl" custom-class="treeD" :modal="false">
      <!-- 帮助 -->
      <div style="width: 220px; margin: 0 auto"><i class="el-icon-question question_icon" @click="$refs.dialog.showDialog = true"></i></div>
      <div class="switch">
        <span>颜色名称：</span>
        <el-select v-model="colorName" placeholder="请选择或搜索" size="small" collapse-tags filterable clearable @change="queryData">
          <el-option v-for="(item, index) in thirdLevelList" :key="index" :label="item.color" :value="item.color"> </el-option>
        </el-select>
      </div>
      <div class="switch">
        <span>类名字号：</span>
        <el-input-number v-model="fontSize" :step="1" :min="1" :max="13" size="small" style="width: 192px"></el-input-number>
      </div>
      <div class="switch">
        <span>色名字号：</span>
        <el-input-number v-model="otherFontSize" :step="1" :min="1" :max="13" size="small" style="width: 192px"></el-input-number>
      </div>
      <div class="switch">
        <span>圆球大小：</span>
        <el-input-number v-model="symbolSize" :step="1" :min="1" :max="13" size="small" style="width: 192px"></el-input-number>
      </div>
      <div class="switch">
        <span>环形树图：</span>
        <el-switch
          v-model="status"
          active-color="#010101"
          inactive-color="#D9D9D9"
          :active-text="status ? '是' : '否'"
          @change="statusChange"
        ></el-switch>
      </div>
      <div class="switch">
        <span>线条颜色：</span>
        <el-color-picker v-model="lineColor"></el-color-picker>
      </div>
      <div class="switch">
        <span>高亮颜色：</span>
        <el-color-picker v-model="eColor"></el-color-picker>
      </div>
      <div class="switch">
        <span>文字颜色：</span>
        <el-color-picker v-model="fontColor"></el-color-picker>
      </div>
    </el-drawer>
    <commonDialog dialogTitle="使用帮助" dialogWidth="600px" :isShowFooterBtn="false" :modal="true" ref="dialog">
      <p>
        ·图例(左上方)：<br />用于解释色名的层级关系。一二级为交叉关系，二三级为从属关系。<br />
        <br />
        ·树图（视觉中心）：<br />以树图形式展示了色名的层级关系。其中二级类别中重复色名为同一类别，<br />二级一共只有正色“青｜赤｜黄｜白｜黑”与<br />间色“绿｜碧｜红｜紫｜流黄”共10个类别。
        <br />
        <br />交互操作：<br />一级类别节点点击可收起，再次点击展开。<br />二级类别节点点击可跳转到新的子页面，展示该类别下的所有三级色名<br />（由于显示限制，默认展示可以几何体形式可视化的三级色名）
        <br />点击三级色名，高亮展示该色名的从属关系。
        <br />
        <br />·颜色名称：<br />可搜索或选择色名，结果将高亮显示。
        <br />
        <br />·环形树图：<br />默认展示环形树图，可切换展示为一般树图。
      </p>
    </commonDialog>
  </div>
</template>

<script>
import * as echarts from 'echarts';
import { colord, extend } from 'colord';
import labPlugin from 'colord/plugins/lab';
import commonDialog from './commonDialog.vue';
import Papa from 'papaparse';
extend([labPlugin]);
export default {
  name: 'threeDPage', // 层级树
  components: { commonDialog },
  data() {
    return {
      isShowOtherTree: false,
      drawer: false,
      colorName: '',
      symbolSize: 2,
      fontSize: 10,
      otherFontSize: 5,
      lineColor: '#bebebe',
      fontColor: '#fff',
      eColor: '#42cccc',
      currentDataIndexs: [], // 高亮节点index
      status: true,
      firstLevelList: [], // 第一层级列表
      secondLevelList: [], // 第二层级列表
      thirdLevelList: [], // 第三层级列表
      secondLevel: [],
      listLength: 0,
      curstomDom: '', //柱状图dom
      otherTreeData: [],
      option: {
        tooltip: {
          trigger: 'item',
          triggerOn: 'mousemove',
        },
        series: [
          {
            type: 'tree',
            data: [],
            left: '2%',
            right: '2%',
            lineStyle: {
              color: '#bebebe',
            },
            layout: 'radial', // 设置为 radial 布局方式
            symbol: 'emptyCircle',
            symbolSize: 2,
            initialTreeDepth: 3,
            animationDurationUpdate: 750,
            emphasis: {
              lineStyle: {
                color: '#42cccc',
              },
            },
            label: {
              fontSize: 10,
              color: '#fff',
            },
            leaves: {
              label: {
                fontSize: 5,
              },
            },
          },
        ],
      },
    };
  },
  methods: {
    changeGoBack() {
      this.curstomDom.clear();
      this.isShowOtherTree = !this.isShowOtherTree;
      this.option.series[0].data = [this.firstLevelList];
      this.renderBar();
    },
    queryData() {
      this.curstomDom.dispatchAction({
        type: 'downplay',
        dataIndex: this.currentDataIndexs,
      });
      if (!this.colorName) return;
      let data = this.findPath(this.colorName, this.firstLevelList);
      if (data && data.length > 0) {
        let arr = data.map((item) => item.index);
        this.currentDataIndexs = [0, ...arr];
        // 高亮点击已保存的相关节点的连线，防止上一步取消了已保存节点的高亮
        this.curstomDom.dispatchAction({
          type: 'highlight',
          dataIndex: this.currentDataIndexs,
        });
      }
    },
    findPath(targetValue, tree, path = []) {
      if (tree.name === targetValue) {
        return path.concat([tree]);
      }
      if (tree.children) {
        for (let i = 0; i < tree.children.length; i++) {
          const child = tree.children[i];
          const result = this.findPath(targetValue, child, path.concat([tree]));
          if (result) {
            return result;
          }
        }
      }
      return null;
    },
    getBarParams() {
      this.option.series[0].data = [this.firstLevelList];
      this.renderBar();
    },
    //渲染
    renderBar() {
      this.curstomDom.setOption(this.option); //设置配置项
      this.$nextTick(() => {
        this.initOtherEvent();
      });
    },
    parseCsvData(url, key) {
      let that = this;
      Papa.parse(url, {
        download: true,
        header: true,
        complete: (results) => {
          this[key] = results.data;
          that.listLength++;
        },
      });
    },
    levelListHandel() {
      let index = 1;
      this.firstLevelList.forEach((item) => {
        // item 表示一级， 取出对应的name
        index += 1;
        item.index = index;
        item.name = item.color;
        item.leaves = 1;
        item.children = [];
        item.itemStyle = {
          color: colord({ l: item.l, a: item.a, b: item.b }).toHex(),
        };
        item.label = {
          color: colord({ l: item.l, a: item.a, b: item.b }).toHex(),
        };
        this.secondLevelList.forEach((e) => {
          // val 表示二级
          let val = { ...e };
          val.name = val.color;
          e.itemStyle = {
            color: colord({ l: e.l, a: e.a, b: e.b }).toHex(),
          };
          e.label = {
            color: colord({ l: e.l, a: e.a, b: e.b }).toHex(),
          };
          if (val.parent == item.color) {
            index += 1;
            val.index = index;
            val.leaves = 2;
            item.children.push(val);
            val.children = this.thirdLevelList.filter((o) => {
              o.name = o.color;
              o.leaves = 3;
              o.itemStyle = {
                color: colord({ l: o.l, a: o.a, b: o.b }).toHex(),
              };
              o.label = {
                color: colord({ l: o.l, a: o.a, b: o.b }).toHex(),
              };
              return o.parent === item.color && o.subparent === val.color;
            });
          }
        });
      });
      this.firstLevelList = {
        index: 1,
        name: '中国传统色',
        children: this.firstLevelList,
        label: {
          rotate: 360,
        },
      };
      this.getBarParams();
    },
    initOtherEvent() {
      let that = this;
      this.curstomDom.on('click', function (params) {
        that.otherTreeData = [];
        // 先取消已高亮的节点连线
        that.curstomDom.dispatchAction({
          type: 'downplay',
          dataIndex: that.currentDataIndexs,
        });
        if (params.data.leaves == 2) {
          let otherArr = that.secondLevel.filter((val) => {
            return val.parent === params.name;
          });
          if (otherArr.length > 0 && !that.isShowOtherTree) {
            that.isShowOtherTree = true;
            that.otherTreeData = {
              index: 0,
              name: params.name,
              children: otherArr.map((i, idx) => {
                i.name = i.color;
                i.index = idx + 1;
                i.itemStyle = {
                  color: colord({ l: i.l, a: i.a, b: i.b }).toHex(),
                };
                i.label = {
                  color: colord({ l: i.l, a: i.a, b: i.b }).toHex(),
                };
                return i;
              }),
              label: {
                rotate: 360,
              },
            };
            that.curstomDom.clear();
            that.option.series[0].data = [that.otherTreeData];
            that.renderBar();
          }
        }
        const treeAncestors = params.treeAncestors;
        const dataIndexs = treeAncestors.map((item) => item.dataIndex);
        // 针对非最尾节点点击收缩展开后已高亮线路失效
        if (
          dataIndexs.length > 0 &&
          that.currentDataIndexs.length > 0 &&
          dataIndexs[dataIndexs.length - 1] === that.currentDataIndexs[that.currentDataIndexs.length - 1]
        ) {
          that.currentDataIndexs = [];
          return;
        }
        if (params.data.children) {
          that.currentDataIndexs = [];
          return;
        }
        // 重新保存当前高亮的节点
        that.currentDataIndexs = dataIndexs;
        // 高亮相关节点连线
        that.curstomDom.dispatchAction({
          type: 'highlight',
          dataIndex: dataIndexs,
        });
      });
      // 节点鼠标移入事件
      this.curstomDom.on('mouseover', function (params) {
        // 取消当前节点的高点，顶替默认事件
        that.curstomDom.dispatchAction({
          type: 'downplay',
          dataIndex: params.dataIndex,
        });

        // 高亮点击已保存的相关节点的连线，防止上一步取消了已保存节点的高亮
        that.curstomDom.dispatchAction({
          type: 'highlight',
          dataIndex: that.currentDataIndexs,
        });
      });
    },
    //改变chart大小
    changeChartSize() {
      this.curstomDom.resize();
    },
    statusChange() {
      if (this.status) {
        this.option.series = [
          {
            type: 'tree',
            data: [this.isShowOtherTree ? this.otherTreeData : this.firstLevelList],
            left: '2%',
            right: '2%',
            lineStyle: {
              color: this.lineColor,
            },
            layout: 'radial', // 设置为 radial 布局方式
            symbol: 'emptyCircle',
            symbolSize: this.symbolSize,
            initialTreeDepth: 3,
            animationDurationUpdate: 750,
            emphasis: {
              lineStyle: {
                color: this.eColor,
              },
            },
            label: {
              fontSize: this.fontSize,
              color: this.fontColor,
            },
            leaves: {
              label: {
                fontSize: this.otherFontSize,
              },
            },
          },
        ];
      } else {
        this.option.series = [
          {
            type: 'tree',
            data: [this.isShowOtherTree ? this.otherTreeData : this.firstLevelList],
            left: '2%',
            right: '2%',
            lineStyle: {
              color: this.lineColor,
            },
            layout: '',
            symbol: 'emptyCircle',
            orient: 'vertical',
            symbolSize: this.symbolSize,
            initialTreeDepth: 3,
            animationDurationUpdate: 750,
            emphasis: {
              lineStyle: {
                color: this.eColor,
              },
            },
            label: {
              fontSize: this.fontSize,
              color: this.fontColor,
              position: 'left',
              verticalAlign: 'middle',
              align: 'right',
            },
            leaves: {
              label: {
                position: 'bottom',
                rotate: -90,
                verticalAlign: 'middle',
                align: 'left',
                fontSize: this.otherFontSize,
                color: this.fontColor,
              },
            },
          },
        ];
      }
      this.curstomDom.clear();
      this.curstomDom.setOption(this.option); //设置配置项
    },
  },
  created() {
    this.index = 0;
    this.parseCsvData('/assets/color/firstLevel.csv', 'firstLevelList');
    this.parseCsvData('/assets/color/secondOhterLevel.csv', 'secondLevelList');
    this.parseCsvData('/assets/color/other.csv', 'thirdLevelList');
    this.parseCsvData('/assets/color/secondLevel.csv', 'secondLevel');
  },
  mounted() {
    // 柱状图实例化
    this.curstomDom = echarts.init(document.querySelector('#chart'));
    // 监听resize
    window.addEventListener('resize', this.changeChartSize);
  },
  watch: {
    otherTreeData: {
      handler: function (val) {
        console.log(val);
      },
      deep: true,
    },
    listLength: function (val) {
      if (val === 3) {
        this.levelListHandel();
      }
    },
    lineColor: function (val) {
      this.$nextTick(() => {
        this.option.series[0].lineStyle.color = val ? val : '#fff';
        this.curstomDom.setOption(this.option); //设置配置项
      });
    },
    fontColor: function (val) {
      this.$nextTick(() => {
        this.option.series[0].label.color = val ? val : '#fff';
        if (!this.status) this.option.series[0].leaves.label.color = val ? val : '#fff';
        this.curstomDom.setOption(this.option); //设置配置项
      });
    },
    eColor: function (val) {
      this.$nextTick(() => {
        this.option.series[0].emphasis.lineStyle.color = val ? val : '#42cccc';
        this.curstomDom.setOption(this.option); //设置配置项
      });
    },
    fontSize: function (val) {
      this.$nextTick(() => {
        this.option.series[0].label.fontSize = val;
        this.curstomDom.setOption(this.option); //设置配置项
      });
    },
    symbolSize: function (val) {
      this.$nextTick(() => {
        this.option.series[0].symbolSize = val;
        this.curstomDom.setOption(this.option); //设置配置项
      });
    },
    otherFontSize: function (val) {
      this.$nextTick(() => {
        this.option.series[0].leaves.label.fontSize = val;
        this.curstomDom.setOption(this.option); //设置配置项
      });
    },
  },
};
</script>

<style lang="scss">
.treeCharts {
  .fixed {
    position: absolute;
    right: 12px;
    top: 10px;
    i {
      font-size: 24px;
      cursor: pointer;
      text-shadow: 0px 0px 2px #000;
      color: #fff;
    }
  }
  .leftTop-png {
    position: absolute;
    top: 20px;
    left: 20px;
    width: 380px;
  }
}
.treeD {
  padding: 0 10px;
  width: 240px !important;
  .switch {
    .el-icon-search {
      line-height: 32px;
    }
    color: #fff;
    text-align: left;
    display: flex;
    align-items: center;
    margin-bottom: 10px;
    span {
      color: #000;
      white-space: nowrap;
      font-size: 14px;
    }
  }
  .el-color-picker {
    height: 25px;
  }
  .el-color-picker__trigger {
    width: 25px;
    height: 25px;
  }
  .el-drawer__header {
    padding: 10px 6px 0;
  }
}
.goBack {
  position: absolute;
  left: 20px;
  top: 120px;
  i {
    font-size: 24px;
    cursor: pointer;
    text-shadow: 0px 0px 2px #000;
    color: #fff;
  }
}
</style>
