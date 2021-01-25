<template>
  <div>
    <el-tree
      :data="menus"
      :props="defaultProps"
      :expand-on-click-node="false"
      show-checkbox
      node-key="catId"
      :default-expanded-keys="defaulstexKey"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level <= 2"
            type="text"
            size="mini"
            @click="() => append(data)"
          >
            Append
          </el-button>
          <el-button
            v-if="node.childNodes.length == 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >
            Delete
          </el-button>
        </span>
      </span>
    </el-tree>

    <el-dialog title="提示" :visible.sync="dialogVisible" width="30%">
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCategory">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      category: { name: "", parentCid: 0, catLevel: 0, showStatus: 1, sort: 0 },
      dialogVisible: false,
      menus: [],
      defaulstexKey: [],
      defaultProps: {
        children: "children",
        label: "name",
      },
    };
  },
  methods: {
    getMenus() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
      }).then(({ data }) => {
        console.log("成功获取到数据=>", data.data);
        this.menus = data.data;
      });
    },
    append(data) {
      console.log(data);
      this.dialogVisible = true;
       this.category.parentCid = data.catId;
       this.category.catLevel = data.catLevel*1+1;
    },
    // 添加分类
    addCategory() {
      console.log("提交的分类数据=>", this.category);
      this.$http({
      url: this.$http.adornUrl('/product/category/save'),
      method: 'post',
      data: this.$http.adornData(this.category, false)
      }).then(({ data }) => {
        this.$message({
              message: "保存成功!",
              type: "success",
            });
        //关闭对话框
        this.dialogVisible=false;
        //刷新菜单
        this.getMenus();
        //设置默认展开的菜单
        this.defaulstexKey = [this.category.parentCid];
       });
    },
    remove(node, data) {
      var ids = [data.catId];
      console.log("ids=>", ids);
      this.$confirm(`是否删除[${data.name}]菜单`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false),
          }).then(({ data }) => {
            this.$message({
              message: "恭喜你，删除成功了!",
              type: "success",
            });
            this.getMenus();
            this.defaulstexKey = [node.parent.data.catId];
          });
        })
        .catch(() => {});
    },
  },
  created() {
    this.getMenus();
  },
};
</script>

<style>
</style>