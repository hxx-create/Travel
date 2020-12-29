<template>
    <div>
        <!--按钮-->
        <el-button type="primary" @click="toAddHandler">添加联系人</el-button>
        <!-- <el-button type="danger">批量删除</el-button> -->

        <!--表格-->
        
        <el-table :data="addresss.list" :cell-style="changeCellStyle">
            <el-table-column prop="id" label="编号"></el-table-column>
            <el-table-column prop="province" label="姓名"></el-table-column>
            <el-table-column prop="city" label="性别"></el-table-column>
            <el-table-column prop="area" label="身份证号"></el-table-column>
            <!-- <el-table-column prop="address" label="所属街道"></el-table-column> -->
            <el-table-column prop="telephone" label="电话"></el-table-column>
            <el-table-column  label="操作">
                <template v-slot="slot">
                    <a class="el-icon-delete" href="" @click.prevent="toDeleteHandler(slot.row.id)">删除</a>
                    <a class="el-icon-edit" href="" @click.prevent="toUpdateHandler(slot.row)">修改</a>
                </template>
            </el-table-column>
        </el-table>
        <!--表格结束-->

        <!-- 分页开始-->
    <el-pagination 
        :hide-on-single-page="true"
        layout="prev, pager, next" 
        :total="addresss.total" 
        @current-change="pageChageHandler">
        </el-pagination>
        <!--分页结束 -->

        <!--模态框-->
        <el-dialog
            :title="title"
            :visible.sync="visible"
            width="60%">
            <el-form :model="form" label-width="80px">
                <el-form-item label="姓名">
                    <el-input v-model="form.province"></el-input>
                </el-form-item>
                <el-form-item label="性别">
                    <el-input v-model="form.city"></el-input>
                </el-form-item>
                <el-form-item label="身份证号">
                    <el-input v-model="form.area"></el-input>
                </el-form-item>
                <!-- <el-form-item label="街道">
                    <el-input v-model="form.address"></el-input>
                </el-form-item> -->
                <el-form-item label="电话">
                    <el-input v-model="form.telephone"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button size="small" @click="closeModalHandler">取 消</el-button>
                <el-button size="small" type="primary" @click="submitHandler">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
import request from '@/utils/request'
import querystring from 'querystring'    //导入一个系统库，用来查询字符串
export default {
    //用于存放网页中需要调用的方法
    methods:{
            // 当分页中当前页改变的时候执行
    pageChageHandler(page){
        // 将params中当前页改为插件中的当前页
        this.params.page = page-1;
        // 加载
        this.loadData();
    },
    loadData(){
      let url = "http://39.96.84.188:6677/address/query"
      if(this.params.status === "全部"){
        delete this.params.status;
      }
      request({
          url,
          method:"post",
          headers:{
              "Content-Type":"application/x-www-form-urlencoded"
          },
          data:querystring.stringify(this.params)
      }).then((response)=>{
          // orders为一个对象，其中包含了分页信息，以及列表信息
          this.addresss = response.data;
      })
    },
        submitHandler(){
            //通过request与后台逻辑进行交互，并且携带参数
            let url="http://39.96.84.188:6677/address/saveOrUpdate"
            request({
                url,
                method:"POST",
                headers:{
                    "Content-Type":"application/x-www-form-urlencoded"
                },
                data:querystring.stringify(this.form)
            }).then((response)=>{
                //请求结束，模态框关闭
                this.closeModalHandler();
                this.$message({
                    type:"success",
                    message:response.message
                })
                this.loadData();
            })
        },
        toDeleteHandler(id){
            //先确认
          this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          //调用后台接口完成删除操作
          let url="http://39.96.84.188:6677/address/deleteById?id="+id;
          request.get(url).then((response)=>{
              //刷新数据
                this.loadData();
              //提示结果
                this.$message({
                type: 'success',
                message:response.message
                });
          })
          
        })
        },
        toUpdateHandler(row){
            //模态框的表单中显示出当前行的信息
            this.form=row;
            this.title="修改地址信息"
            this.visible=true;
        },
        closeModalHandler(){
            this.visible=false;
        },
        toAddHandler(){
            this.visible=true;
            this.form={
            }
        },
        changeCellStyle (row, column, rowIndex, columnIndex) {
   if(row.column.label === "编号"){
       return 'color: skyblue'  // 修改的样式
   }else if(row.column.label === "操作"){
       return 'color:orange'
   }
}
    },
    //用于存放要向网页中显示的数据
    data(){
        return{
            form:{
            },
            visible:false,
            title:"录入地址信息",
            addresss:{},
                  params:{
          page:0,
          pageSize:10
      },
        }
    },
    //生命周期,VUE实例完毕后要执行的操作
    created(){
       this.loadData();
    }
}
</script>

<style scoped>

</style>