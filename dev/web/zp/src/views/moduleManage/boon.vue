/*
 * @Author: wenxt 
 * 福利管理页面
 * @Date: 2019-12-25 14:55:59 
 * @Last Modified by: wenxt
 * @Last Modified time: 2019-12-27 17:28:09
 */

<template>
  <div id="moduleBoon">
    <!-- {{welfareData}} -->
    <div class="searchDiv">
      <el-select @change="welfareChange" size="mini" v-model="welfare" clearable placeholder="全部">
        <el-option
          v-for="item in welfareName"
          :key="item"
          :label="item"
          :value="item">
        </el-option>
      </el-select>
    </div>
    <div class="tableDiv">
      <el-table
        ref="multipleTable"
        :data="welfareList"
        tooltip-effect="dark"
        style="width: 100%"
        @selection-change="changeWelfare">
        <el-table-column
          type="selection"
          >
        </el-table-column>
        <el-table-column
          prop="name"
          label="福利名称"
          width="900">
        </el-table-column>
        <el-table-column
          prop="status"
          label="状态"
          >
        </el-table-column>
        <el-table-column
          fixed="right"
          label="操作"
          width="100">
          <template slot-scope="scope">
            <el-button v-if="scope.row.status === '使用中'" @click="toBlocking(scope.row)" type="text" size="small">冻结</el-button>
            <el-button v-else type="text" size="small" @click="toRegain(scope.row)">恢复</el-button>
          </template>
        </el-table-column>
      </el-table>
    </div>
    <div class="footerDiv">
      <div class="btnDiv">
        <el-button @click="toBatchDelete" size="mini" type="danger" plain>批量删除</el-button>
      </div>
      <div class="pageDiv">
        <el-pagination
          :page-size="pageSize"
          :current-page.sync="currentPage"
          background
          @current-change="pageChange"
          layout="prev, pager, next"
          :total="welfareData.length">
        </el-pagination>
      </div>
    </div>
  </div>
</template>

<script>
import {deleteWelfareById,findAllWelfare,findWelfareById,saveOrUpdateWelfare} from '@/api/welfare.js';
import config from '@/utils/config.js';

export default {
  data() {
    return {
      //福利数据
      welfareData:[],
      //福利数据备份
      welfareDataBackup:[],
      //福利名称数组（去重）
      welfareName:[],
      welfare:'',
      currentWelfare:{},
      //当前页
      currentPage:1,
      //每页条数
      pageSize:config.pageSize,
      ids:[],
    };
  },
  computed: {
    //分页后的数据
    welfareList(){
      let temp = [...this.welfareData];
      let page = this.currentPage;
      let size = this.pageSize;
      return temp.slice((page-1)*size,page*size);
    }
  },
  methods: {
    //获取福利数据(success)
    async findWelfare(){
      try {
        //页面回到首页
        this.currentPage = 1;
        let res = await findAllWelfare();
        // console.log(res.data);
        this.welfareData = res.data;
        this.welfareDataBackup = res.data;
        let welfareNameArr = res.data.map((item)=>{
          // console.log(item.name,'-------');
          return item.name;
        })
        //去重
        this.welfareName = [...new Set(welfareNameArr)];
      } catch (error) {
        console.log(error);
      }
    },
    //冻结福利(还差在不同页面进行修改时，他会跳转回第一页，因为进行了findAll，还有查看某一个福利的时候也是)
    async toBlocking(val){
      // console.log(val);
      let temp = {...val};
      // console.log(temp.status);
      temp.status = "被冻结";
      this.currentWelfare = temp;
      try {
        let res = await saveOrUpdateWelfare(this.currentWelfare);
        if(res.status === 200){
                config.successMsg(this,'冻结成功');
                this.findWelfare();
              }else{
                config.errorMsg(this,'冻结失败');
              }
      } catch (error) {
        config.errorMsg(this,'冻结失败');
        console.log(error);
      }
    },
    //恢复福利使用(同上)
    async toRegain(val){
      // console.log(val);
      let temp = {...val};
      // console.log(temp.status);
      temp.status = "使用中";
      this.currentWelfare = temp;
      try {
        let res = await saveOrUpdateWelfare(this.currentWelfare);
        if(res.status === 200){
                config.successMsg(this,'恢复成功');
                this.findWelfare();
              }else{
                config.errorMsg(this,'恢复失败');
              }
      } catch (error) {
        config.errorMsg(this,'恢复失败');
        console.log(error);
      }
    },
    //福利名称改变(success)
    welfareChange(val){
      if(val){
        let result = this.welfareDataBackup.filter(item=>{
          return item.name === val;
        });
        this.welfareData = result;
      }else{
        this.findWelfare();
      }
    },
    //获取复选框选中的(success)
    changeWelfare(val){
      //获取到选中的所有数据数组
      let ids = val.map(item => {
        return item.id;
      })
      this.ids = ids;
    },
    //批量删除(success)
    toBatchDelete(){
      //获取要删除的id
      let ids = this.ids;
      if(ids.length > 0){
        //弹出提示框询问用户是否删除
        this.$confirm('是否删除?', '提示', {
          confirmButtonText: '删除',
          callback:action => {
            if(action === "confirm"){
              let result = [];
              ids.forEach(async id => {
                try {
                  let res = await deleteWelfareById({id:id});
                  result.push(res.status);
                } catch (error) {
                  result.push(500);
                }
              });
              setTimeout(()=>{
                //判断result的值是否都是200，如果不是则返回失败。
                let resu = result.every(item=>{
                  return item===200;
                })
                if(resu){
                  config.successMsg(this,"批量删除成功");
                }else{
                  config.errorMsg(this,"批量删除失败");
                }
                //数据已经删除所以要重新更新
                this.findWelfare();
              },2000);
            }
          }
        });
      }else{
        this.$message({
          message: "请选中要删除的数据",
          type: "warning"
        });
      }
    },
    //页面改变(success)
    pageChange(page){
      // console.log(page,'!!!!!!!!!!');
      this.currentPage = page;
      // 让当前页发生改变
    }

  },
  created() {
    this.findWelfare();
  },
  mounted() {}
};
</script>
<style lang="scss" scoped>
  .footerDiv{
    overflow: hidden;
    margin-top: 10px;
    .btnDiv{
      float: left;
    }
    .pageDiv{
      float: right;
    }
  }
</style>