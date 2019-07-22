<template>
  <v-form v-model="valid" ref="BrandForm">
    <v-text-field v-model="brand.name" label="请输入品牌名称" required :rules="nameRules" />
    <v-text-field v-model="brand.letter" label="请输入品牌首字母" required :rules="letterRules"/>
    <v-cascader
      url="/api/category/List"
      multiple
      required
      v-model="brand.categories"
      label="请选择商品分类"/>
    <v-layout row>
      <v-flex xs3>
        <span style="font-size: 16px; color: #444">品牌LOGO：</span>
      </v-flex>
      <v-flex>
        <v-upload v-model="brand.image" url="/api/upload/image" :multiple="false" :pic-width="250" :pic-height="90"/>
      </v-flex>
    </v-layout>
    <v-layout class="my-4" row>
      <v-spacer/>
      <v-btn @click="submit" color="primary">提交</v-btn>
      <v-btn @click="clear">重置</v-btn>
    </v-layout>
  </v-form>
</template>

<script>
  export default {
    name: "brand-form",
    props: {
      oldBrand: {type: Object},
      isEdit: {type: Boolean,default: false}
    },
    data() {
      return {
        valid: false, // 表单校验结果标记
        brand: {
          name: '', // 品牌名称
          letter: '', // 品牌首字母
          image: '',// 品牌logo
          categories: [], // 品牌所属的商品分类数组
        },
        changeClassRules:{
          'brand.name': [{required: true, message: "请输入联系人邮箱", trigger: "blur"},{type: "email", message: "请输入正确的邮箱地址", trigger: "blur"}]
        },
        // nameRules: [
        //   // console.log("12343",this.nameRules),
        //   v => !!v || '品牌名称不能为空',
        //   v => v.length > 1 || '品牌名称至少2位'
        // ],
        nameRules: [
          v => !!v || '品牌名称不能为空',
          v => (v && v.length > 1) || '品牌名称至少2位'
        ],
        letterRules: [
          v => !!v || '首字母不能为空',
          v => /^[a-zA-Z]{1}$/.test(v) || "品牌字母只能是1个字母"
        ]
      }
    },
    methods: {
      // submit(){
      //   //提交表单
      //   if(this.$refs.BrandForm.validate()){
      //     console.log("this.$refs.BrandForm.validate()",this.$refs.BrandForm);
      //     /**
      //      * 使用解构表达式获取数据，除categories以外的数据都放入rest中，然后对categories使用map进行处理，得到id后重新赋值给
      //      * rest里面的categories数组
      //      */
      //     const {categories, ... rest}=this.brand;
      //     rest.categories=categories.map(c => c.id).join(",");
      //     console.log("rest",rest);
      //     if(this.isEdit) {
      //       /*this.$http.delete("/api/brand/bid/" + this.oldBrand.id).then().catch();
      //       console.log("rest1",this.isEdit);*/
      //
      //       console.log("rest1",rest);
      //       this.$http({
      //         contentType: "application/json;charset=UTF-8",
      //         dataType: 'json',
      //         method: 'post',
      //         url: '/api/brand/update',
      //         data:this.$qs.parse(rest),
      //       }).then(data => {
      //         console.log("data",data);
      //         if(data.data.code =="0000"){
      //           this.$message.success("修改成功！");
      //           //关闭对话框
      //           this.$emit('closeWindow');
      //           this.clear();
      //         }
      //       }).catch(() => {
      //         this.$message.error("修改失败！");
      //       });
      //     }
      //     else{
      //     this.$http({
      //       method:'post',
      //       url:"/api/brand/add",
      //       data:this.$qs.parse(rest),
      //     }).then(() =>{
      //         console.log("asd",data);
      //         //关闭对话框
      //         this.$emit('closeWindow');
      //         this.$message.success("添加成功！");
      //         this.clear();
      //       }).catch(()=>{
      //         this.$message.error("添加失败！");
      //         this.clear();
      //         this.$emit("closeWindow");
      //       }
      //     );
      //   }
      //   }
      // },

      submit(){
        //提交表单
        if(this.$refs.BrandForm.validate()){
          /**
           * 使用解构表达式获取数据，除categories以外的数据都放入rest中，然后对categories使用map进行处理，得到id后重新赋值给
           * rest里面的categories数组
           */
          const {categories, ... rest}=this.brand;
          rest.categories=categories.map(c => c.id).join(",");
          console.log("12312",rest);
          if(this.isEdit) {
            // this.$http.delete("/api/brand/cid_bid/" + this.oldBrand.id).then().catch();
            console.log("rest1",this.isEdit);
            console.log("rest2",rest);
            this.$http({
              contentType: "application/json;charset=UTF-8",
              dataType: 'json',
              method: 'post',
              url: '/api/brand/update',
              data:this.$qs.parse(rest),
              }).then(data => {
              console.log("data",data);
              if(data.data.code =="0000"){
                this.$message.success("修改成功！");
                //关闭对话框
              // this.$parent.closeWindow();
              //   this.$emit("closeWindow");
              //   this.$emit("close");
              //   this.clear();
              }
            }).catch(() => {
              // this.$emit("close");
              // this.clear();
              this.$message.error("修改失败！");
            });
          }else{
            this.$http({
            contentType: "application/json;charset=UTF-8",
            dataType: 'json',
            method:'post',
            url:"/api/brand/add",
            data:this.$qs.parse(rest),
          }).then(() =>{
              //关闭对话框
              this.$message.success("添加成功！");
            }).catch(()=>{
              this.$message.success("添加失败！");
            }
          );
            // this.clear();
            // this.$emit("close");
        }
          this.$emit("close");
          this.clear();
        }
      },
      // handleEdit() {
      //   console.log("update .... ",this.rest);
      //   this.$http({
      //     method: 'post',
      //     url: '/api/category/update',
      //     data:this.$qs.stringify(this.rest),
      //   }).then(data => {
      //     if(data.data.code =="0000"){
      //       this.$message.success("修改成功！");
      //     }
      //   }).catch(() => {
      //     this.$message.error("修改失败！");
      //   });
      // },
      clear() {
       /* this.brand.name='',
        this.brand.letter='',
        this.brand.image='',
        this.brand.categories=[],
        this.$refs.image_url._data.dialogImageUrl='',*/
        // 重置表单
        this.$refs.BrandForm.reset();
        // 需要手动清空商品分类
        this.categories = [];
      }
    },
    watch: {
      oldBrand: {// 监控oldBrand的变化
        handler(val) {
          if (val) {
            // 注意不要直接复制，否则这边的修改会影响到父组件的数据，copy属性即可
            this.brand = Object.deepCopy(val)
          } else {
            // 为空，初始化brand
            this.clear();
            /*this.brand = {
              name: '',
              letter: '',
              image: '',
              categories: [],
            }*/
          }
        },
        deep: true
      }
    }
  }
</script>

<style scoped>

</style>
