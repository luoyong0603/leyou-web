<template>
  <div>
  <v-card-title>
    <v-btn color="primary" @click="addBrand">
      <v-icon small>el-icon-plus</v-icon>&nbsp;新增品牌
    </v-btn>
    <v-btn color="error" @click="deleteAllBrand">
      <v-icon small>el-icon-delete</v-icon>&nbsp;删除品牌
    </v-btn>
    <!--空间隔离组件-->
    <v-spacer />
    <!--搜索框，与search属性关联-->
    <v-text-field label="输入关键字搜索" v-model="key" append-icon="search" hide-details/> <!--因为默认在文本框下面预留有错误提示空间。hide-details可以取消-->
  </v-card-title>

    <v-data-table
      v-model="selected"
      :headers="headers"
      :items="brands"
      :search="search"
      :pagination.sync="pagination"
      :total-items="totalBrands"
      :loading="loading"
      select-all
      class="elevation-1"
    >

      <template slot="items" slot-scope="props">
        <td>
          <v-checkbox
            v-model="props.selected"
            primary
            hide-details>
          </v-checkbox>
        </td>
        <td class="text-xs-center">{{ props.item.id }}</td>
        <td class="text-xs-center">{{ props.item.name }}</td>
        <td class="text-xs-center"><img :src="props.item.image"></td>
        <td class="text-xs-center">{{ props.item.letter }}</td>
        <td class="text-xs-center">
          <v-btn flat icon color="info" @click="editBrand(props.item)"><v-icon small>el-icon-edit</v-icon></v-btn>
          <v-btn flat icon color="error" @click="deleteBrand(props.item)"><v-icon small>el-icon-delete</v-icon></v-btn>
        </td>
      </template>
  </v-data-table>
    <!--弹出的对话框-->
    <v-dialog max-width="580" v-model="showForm" persistent scrollable>
      <v-card>
        <!--对话框的标题-->
        <v-toolbar dense dark color="primary">
          <v-toolbar-title>{{isEdit? '修改' : '新增' }}品牌</v-toolbar-title>
          <v-spacer/>
          <!--关闭窗口的按钮-->
          <v-btn icon @click="closeWindow"><v-icon>close</v-icon></v-btn>
        </v-toolbar>
        <!--对话框的内容，表单-->
        <v-card-text class="px-5">
          <my-brand-form @close="closeWindow" :isEdit="isEdit" :oldBrand="oldBrand"></my-brand-form>
        </v-card-text>
      </v-card>
    </v-dialog>



  </div>
</template>

<script>
  // 导入自定义的表单组件
  import MyBrandForm from './BrandForm'

  export default {
      name: "MyBrand",
      data() {
        return {
          key:"",//搜索条件key
          selected:[],//CheckBox单选复选
          showForm:false,//弹框显示隐藏
          search: '', // 搜索过滤字段
          totalBrands: 0, // 总条数
          brands: [], // 当前页品牌数据
          loading: true, // 是否在加载中
          pagination: {}, // 分页信息 包含 page// 当前页  rowsPerPage//每页大小   sortBy: '', // 排序字段  descending:false, // 是否降序
          headers: [
            {text: 'id', align: 'center',sortable: true, value: 'id'}, /*sortable: true 表示是否有上下排序的箭头*/
            {text: '名称', align: 'center', sortable: false, value: 'name'},
            {text: 'LOGO', align: 'center', sortable: false, value: 'image'},
            {text: '首字母', align: 'center', value: 'letter', sortable: true,},
            {text: '操作', align: 'center', value: 'id', sortable: false}
          ],
          oldBrand:{},//回显要修改的数据
          isEdit:false,//是否编辑
          selected:[] //选择的条目
        }
      },
      mounted(){ // 渲染后执行
        // 查询数据
        // this.getDataFromServer();
       /* this.loadBrands();*/
      },
    components:{
      MyBrandForm
    },
    created(){
        //去后台查
      this.loadBrands();
    },
    //监控 只要修改搜索框就调用调用一次接口
    watch:{
        key(){
           this.pagination.page = 1;
           this.loadBrands();
        },
        pagination:{
          deep:true,
          handler(){
            this.loadBrands();
          }
        }
    },
      methods:{
        //编辑品牌信息
        editBrand(oldBrand){
          this.$http.get("/api/category/bid/"+oldBrand.id).then(({data}) =>{
              // 修改标记
              this.isEdit = true;
              // 控制弹窗可见：
              this.showForm = true;
              // 获取要编辑的brand
              this.oldBrand = oldBrand;
              // 回显商品分类
              this.oldBrand.categories = data;
            }).catch();
        },
        deleteBrand(oldBrand){
            // if(this.selected.length === 1 && this.selected[0].id === oldBrand.id){   注释这句判断语句的目的在于【省略勾选后再删除操作】直接【点击删除图标即可进入删除操作】
              this.$message.confirm('此操作将永久删除该品牌，是否继续?').then(() =>{
                //发起删除请求
                  this.$http.delete("/api/brand/bid/" + oldBrand.id).then(() =>{
                    this.loadBrands();
                  }).catch()
              }).catch(() =>{
                  this.$message.info("删除已取消!");
              })
            // }
        },
        deleteAllBrand(){
          //拼接id数组
          // 加了{}就必须有return
          const ids = this.selected.map( s => s.id);
          if(this.selected.length>0){
            this.$message.confirm('此操作将永久删除该品牌，是否继续?').then(() =>{
              //发起删除请求
              this.$http.delete("/api/brand/bid/" + ids.join("-")).then(() =>{
                this.loadBrands();
              }).catch()
            }).catch(() =>{
              this.$message.info("删除已取消!");
            })
          }
          else{
            this.$message.warning("请先勾选需要删除的条目!");
          }
        },

        //添加品牌
        addBrand(){
          this.isEdit = false;
          this.showForm = true; // 控制弹窗可见：
          this.oldBrand = null;
        },
        closeWindow(){
          //重新加载数据
          this.loadBrands();
          // 关闭窗口
          this.showForm = false;
          // this.$refs.MyBrandForm.clear();
        },

        //获取展示内容
        loadBrands(){
          this.loading=true;
          this.$http.get("/api/brand/page",{
            params:{
              page:this.pagination.page,//当前页
              rows:this.pagination.rowsPerPage,//每页大小
              sortBy:this.pagination.sortBy,//排序字段
              desc:this.pagination.descending,//是否降序
              key:this.key//搜索条件
            }
          }).then(resp =>{
            console.log("dayin",resp);
            this.brands = resp.data.data.items;
            this.totalBrands = resp.data.data.total;
            this.loading = false;
          })
        },

      }
    }
</script>

<style scoped>

</style>
