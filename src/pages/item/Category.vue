<template>
  <v-card>
      <v-flex xs12 sm10>
        <v-tree ref="tree" url="/api/category/List"
                :isEdit="isEdit"
                @handleAdd="handleAdd"
                @handleEdit="handleEdit"
                @handleDelete="handleDelete"
                @handleClick="handleClick"
        />
      </v-flex>
  </v-card>
</template>

<script>
  export default {
    name: "category",
    data() {
      return {
        isEdit:true,
      }
    },
    methods: {
      handleAdd(node) {
        console.log("add .... ",node);
          this.$http({
            method:'post',
            url:'/api/category/add',
            data:node
          }).then(res => {
            console.log("1111111111111",res);
            if(res.data.code == "0000"){
              this.$message.success("添加成功！");
              // this.$router.go(0);
            }else{
              this.$message.error("添加失败！");
            }
            this.$refs.tree.getData(); //节点添加成功，调用子组件的getData()重新获取tree数据
          }).catch(() => {
            this.$message.error("添加失败！");
          });
      },

      handleEdit(id,name) {
        console.log("handleEdit",id);
        const node = {
          id: id,
          name: name
        };
        console.log("update .... ",node);
        this.$http({
          method: 'post',
          url: '/api/category/update',
          data: node
        }).then(data => {
          if(data.data.code =="0000"){
            this.$message.success("修改成功！");
          }
        }).catch(() => {
          this.$message.error("修改失败！");
        });
      },

      handleDelete(id) {
        console.log("delete ... " + id);
        this.$http.post("/api/category/delete/"+id).then(() =>{
          this.$message.success("删除成功！");
        }).catch(() =>{
          this.message.error("删除失败！")
        })
      },
      handleClick(node) {
        console.log(node)
      },
    }
  };
</script>

<style scoped>

</style>
