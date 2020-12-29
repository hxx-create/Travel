<template>
    <div>
        <!-- 按钮 -->
        <el-button type="success" size="small" @click="toAddHandler" >添加</el-button>
        <!-- <el-button type="danger" size="small">批量删除</el-button> -->
        <!-- 按钮 -->
        <!-- 表格 -->
        <el-table :data="products.list" :cell-style="changeCellStyle">
            <el-table-column prop="id" label="编号"></el-table-column>
            <el-table-column prop="name" label="产品名称"></el-table-column>
            <el-table-column prop="price" label="价格"></el-table-column>
            <el-table-column prop="description" label="描述"></el-table-column>
            <el-table-column prop="categoryId" label="所属分类"></el-table-column>
           <el-table-column  prop="photo" label="照片">
            <template v-slot="slot">
             <img  class="img" :src="slot.row.photo" >
            </template>
           </el-table-column>
            <el-table-column prop="id" label="操作">
                <template v-slot="slot">
                    <a class="el-icon-delete" href="" @click.prevent="toDeleteHandle(slot.row.id)">删除</a>
                    <a class="el-icon-edit" href="" @click.prevent="toUpdateHandle(slot.row)">修改</a>
                </template>
            </el-table-column>
        </el-table>
        <!-- /表格 -->
    <!-- 分页开始 -->
    <el-pagination 
        :hide-on-single-page="true"
        layout="prev, pager, next" 
        :total="products.total" 
        @current-change="pageChageHandler">
        </el-pagination>
    <!-- /分页结束 -->       
        <!-- 模态框 -->
        <el-dialog
          title="录入产品信息"
          :visible.sync="visible"
          width="60%">
          <el-form :model="form" label-width="80px">
              <el-form-item label="产品名称">
                  <el-input v-model="form.name"></el-input>
              </el-form-item>
              <el-form-item label="价格">
                  <el-input v-model="form.price"></el-input>
              </el-form-item>
              <el-form-item label="所属栏目">
                  <el-select v-model="form.categoryId" placeholder="请选择">
                       <el-option
                            v-for="item in options"
                            :key="item.id"
                            :label="item.name"
                            :value="item.id">
                      </el-option>
                 </el-select>
              </el-form-item>
              <el-form-item label="介绍">
                  <el-input type="textarea" placeholder="请输入内容" row=5 v-model="form.description"></el-input>
              </el-form-item>
              <el-form-item label="图片">
                <el-upload
                class="upload-demo"
                action="http://134.175.154.93:6677/file/upload"
                :file-list="fileList"
                :on-success="uploadSuccessHandler"
                list-type="picture">
                <el-button size="small" type="primary">点击上传</el-button>
                <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
                </el-upload>
              </el-form-item>
          </el-form>
          <span slot="footer" class="dialog-footer">
          <el-button  size="small" @click="closeModalHandler">取 消</el-button>
          <el-button size="small" type="primary" @click="submitHandler">确 定</el-button>
          </span>
        </el-dialog>
         <!-- /模态框 -->   


    </div>
</template>

<script>
import request from '@/utils/request'
import querystring from 'querystring'
export default {  
    //用于存放页面中需要调用的方法
    methods:{
            // 当分页中当前页改变的时候执行
    pageChageHandler(page){
        // 将params中当前页改为插件中的当前页
        this.params.page = page-1;
        // 加载
        this.loadData();
    },
        //上传成功的事件处理函数
        uploadSuccessHandler(response){
            let photo = "http://134.175.154.93:8888/group1/"
            +response.data.id
            //将图片地址设置到form中，便于一起提交给后台
            console.log(response)
            this.form.photo =photo
        },
    loadData(){
      let url = "http://39.96.84.188:6677/product/query"
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
          this.products = response.data;
      })
    },
    loadCategoty(){
        let url="http://39.96.84.188:6677/category/findAll";
        request.get(url).then((response)=>{
            // 将查询结果设置到customers中，this只想外部函数的this
            this.options = response.data;
            this.closeModalHandler();
        })
    },
        submitHandler(){
            //this.form 对象 ---字符串--> 后台
            //json 字符串‘{“type”:"customer","age"=12}
            //request.post(url,this.form)
            //查询字符串 type=customers&age=12
            //通过request与后台进行交互，并且要携带参数
            let url = "http://39.96.84.188:6677/product/saveOrUpdate";
            request({
                url,
                method:"POST",
                headers:{
                "Content-Type":"application/x-www-form-urlencoded"
                },
                data:querystring.stringify(this.form)

            }).then((response)=>{
                //模态框关闭
                this.closeModalHandler();
                //刷新
                this.loadData();
                //提示消息
                this.$message({
                    type:"success",
                    message:response.message
                })
            })
        },
        
        toDeleteHandle(id){ 
            this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
            }).then(() => {
             //调用后台接口，完成删除操作
            let url = "http://39.96.84.188:6677/product/deleteById?id="+id;
            request.get(url).then((response)=>{
                //1.刷新数据
                this.loadData()
                //2.提示结果
            this.$message({
                type: 'success',
                message: response.message
            });
            })
            })

        },
        toUpdateHandle(row){
            //模态框的表单显示当前行信息
            this.filelist =[];
            this.form = row ;
            this.visible = true;
        },
        closeModalHandler(){
            this.visible = false;
        },  
        toAddHandler(){
            this.fileList = [];
            this.form ={ }
            this.visible = true;
            this.loadData();
            
        },
                changeCellStyle (row, column, rowIndex, columnIndex) {
                if(row.column.label === "编号"){
                    return 'color: skyblue'  // 修改的样式
                }else if(row.column.label === "操作"){
                    return 'color:orange'
                }
                }
    },
    //用于存放要向网页中存放的数据
    data(){
        return{
            visible:false,
            products:{},
            params:{
          page:1,
          pageSize:10
      },
            options:[],
            fileList:[],
            form:{},
            // src:'http://39.96.84.188/imgs/1.jpg'
            
        }
    },

    created(){
        this.loadData();
        this.loadCategoty();
    }
  
    
}
</script>

<style scoped>
.img{
    width:100px ;
    height: 100px;
    background-repeat: no-repeat;
    background-origin: content-box;
    background-size: cover;
}
</style>