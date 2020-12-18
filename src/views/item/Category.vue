<template>
  <div>
  <v-card>
    <v-flex xs12 sm10>
      <v-tree url="/item/category/of/parent" isEdit
              @handleAdd="handleAdd"
              @handleEdit="handleEdit"
      @handleDelete="handleDelete"></v-tree>
    </v-flex>
  </v-card>

  <v-dialog v-model="show"  width="300" height="200">
    <v-card>
      <v-card-title>
        <div>
        <h1 class="headline mb-0">所属分类：{{parentName}}</h1>
      </div>
      </v-card-title>
      <v-card-text>
        <v-text-field label="分类名称：" v-model="category.name"/>
      </v-card-text>
      <v-card-actions>
        <v-spacer/>
        <v-btn color="blue darken-1" flat @click.native="show=false">取消</v-btn>
        <v-btn color="blue darken-1" flat @click.native="save">保存</v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
  </div>
</template>

<script>
  export default {
    name: "category",
    data() {
      return {
        isEdit: false,
        show:false,
        category: {name: '',parentId:-1,sort:1},
        parentName:''
      }
    },
    methods: {
      handleAdd(node) {
        this.show = true;
        this.parentName = node.parentName;
        const {name,id,parentName,...obj} = node;
        console.info(obj);
        this.category = obj;
        console.info(this.category);
      },
      handleEdit(id, name) {
        this.isEdit = true;
        this.category = {};
        this.category.id = id;
        this.category.name = name;
        //this.save();
        this.show = true;
      },
      handleDelete(id) {
        console.log("delete ... id=" + id)
        this.$http({
          method: 'delete',
          url: '/item/category?id='+id
        }).then(resp => {
          // 关闭窗口
          this.show = false;
          this.$message.success("删除成功！");
        }).catch(err => {
          console.info(err.message);
          this.show = false;
          this.$message.error("删除失败！");
        });
      },
      handleClick(node) {
        console.log(node)
      },
      save(){
        this.$http({
          method: this.isEdit ? 'put' : 'post',
          url: '/item/category',
          data: this.category
        }).then(() => {
          // 关闭窗口
          this.show = false;
          this.$message.success("保存成功！");
        }).catch(() => {
          this.show = false;
          this.$message.error("保存失败！");
        });
      }
    }
  };
</script>

<style scoped>

</style>
