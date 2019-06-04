<template>
  <div >
    <el-table :row-class-name="rowStyle" :height="500" show-header style="width:100%" ref="table" :data="modal.list" 
      @row-click="handleTableClick" 
      :span-method="objectSpanMethod" 
      @cell-mouse-enter="cellMouseEnter" 
      @cell-mouse-leave="cellMouseLeave" 
      :cell-class-name="cellClassName" 
      @select-all="selectAll">
        <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
        <el-table-column sortable v-for="item in columns" header-align="center" align="center" :width="item.width > 0 ? item.width : 130" :key="item.field" :prop="item.field" :label="item.title">
          <template slot-scope="scope">
            <span>
              {{scope.row[item.field]}}
            </span>
          </template>
        </el-table-column>
    </el-table>
</div>
</template>

<script>

import { Notification } from 'element-ui'
import dataList from './dataList.js'

export default {
  data() {
    return {
      columns: [
        { title: "卡号", field: "card_no", width:150 },
        { title: "ICCID", field: "iccid", width:200 },
        { title: "上游渠道", field: "up_channel_name"},
        { title: "代理商", field: "agent_name"},
        { title: "代理商划拨",　field: "agent_allot_time"},
        { title: "激活日期",　field: "activation_time"},
        { title: "划拨编号", field: "agent_allot_id"},
        { title: "划拨原因", field: "allot_reason"},
        { title: "APN通道", field: "apn_channel"},
        { title: "上游卡类型",  field: "up_card_type"},
        { title: "下游卡类型",  field: "down_card_type"},
        { title: "上游套餐流量",  field: "real_flow"},
        { title: "下游套餐流量",  field: "total_flow"},
      ],

      modal: { total_count: 0, list: [] },
      // 选框
      allChecked: false,
      switchItems:{
        checkedItems: [],
        reason:''
      },

      spanArr:[], 
      styleArr:[],
      sarr:[],
      sty:'',
    }
  },
  mounted(){
    this.modal.list = dataList
    this.getSpanArr()
  },
  methods: {
    /**
     * 切换鼠标点击行的选中状态(如果是合并行那么合并的都切换状态)
     */
    handleTableClick(row, event, column){
      var arr = this.modal.list.filter( item => {
        return item.card_no == row.card_no
      }).forEach( el => {
        this.$refs.table.toggleRowSelection(el)
      })
    },
    /**
     * 全选
     */
    selectAll(items){
      var arr = items.filter(item => {
        return item.apn_channel == "apn1" || item.apn_channel == "通用apn"
      })
      this.switchItems.checkedItems = arr
    },

    /***
     * 这里会得出一个数组，用于表格合并方法objectSpanMethod进行处理
     * 比如下面的按照卡号相同与否进行判定是否合并，最后会得出[0,1,2,1,3,0...]的整型数组，
     * 既表格每行往下合并的行数: 0表示此行删除，1表示保持此行，2表示合并两行，以此类推
     */
    getSpanArr() {　
      this.spanArr = []
      for (var i = 0; i < this.modal.list.length; i++) {
        if (i === 0) {
          this.spanArr.push(1);
          this.pos = 0
        } else {
        // 判断当前元素与上一个元素是否相同
          if (this.modal.list[i].card_no === this.modal.list[i - 1].card_no) {
            this.spanArr[this.pos] += 1;
            this.spanArr.push(0);
          } else {
            this.spanArr.push(1);
            this.pos = i;
          }
        }
      }
      //给表格每行配置背景属性
      this.styleArr = []
      var i = 0
      this.spanArr.forEach((item,index) => {
        if (item != 0){
          if (i == 0 || i % 2 == 0){
            // this.styleArr.push({"background-color":"#f5f7fa"})
            this.styleArr.push('success-row')
            i++
          }else{
            // this.styleArr.push({"background-color":""})
            this.styleArr.push('')
            i++
          }
        }else{
          this.styleArr.push(this.styleArr[index-1])
        }
        this.sarr = this.styleArr
      })
    },
    //合并行
    objectSpanMethod({ row, column, rowIndex, columnIndex }){
      //确定要合并的列
      if (columnIndex <= 9 || columnIndex === 12) {
        const _row = this.spanArr[rowIndex];
        const _col = _row > 0 ? 1 : 0;
        return {
          rowspan: _row,
          colspan: _col
        }
      }
    },
    //给每行绑定样式
    rowStyle({row, rowIndex}){
      if(row.card_no == this.sty){
        // return {"background-color":"#ECF5FF"}
        return 'another-row'
      }else{
        return this.sarr[rowIndex]
      }
    },
    //鼠标移入触发的事件
    cellMouseEnter(row, column, cell, event) {
      this.sty = row.card_no
    }, 
    //鼠标移出触发的事件
    cellMouseLeave(row, column, cell, event) { 
      this.sty = ''
    },
    //设置满足条件的单元格样式
    cellClassName({row,column,rowIndex,columnIndex}){
      if(columnIndex == 16 || columnIndex == 24){
        return 'cell-style'
      }
    },
  }
}
</script>
<style>
.upload-in-tab {
    display: inline-block;
    position: relative;
}
.upload-in-tab .el-upload__input {
    display: none;
}
.el-table .success-row {
    background: #f5f7fa;
}
.el-table .another-row {
    background: #ECF5FF;
}
.cell-style {
    border-right:  1px solid #ebeef5
}
</style>

