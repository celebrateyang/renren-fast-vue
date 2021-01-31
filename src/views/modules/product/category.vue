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

          <el-button type="text" size="mini" @click="() => edit(data)">
            Edit
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

    <el-dialog :title="title" :visible.sync="dialogVisible" width="30%" :close-on-click-modal="false">
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位">
          <el-input
            v-model="category.productUnit"
            autocomplete="off"
          ></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitData">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      title: "",
      dialogType: "", //edit,add
      category: {
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        catId: null,
        icon: "",
        productUnit: "",
      },
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
      this.title = "添加分类";
      this.dialogType = "add";
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
    },
    edit(data) {
      console.log("要修改的数据=>", data);
      this.dialogVisible = true;
      this.title = "修改分类";
      this.dialogType = "edit";
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: "get",
      }).then(({ data }) => {
        console.log("要回显的数据=>", data);
        this.category.name = data.data.name;
        this.category.catId = data.data.catId;
        this.category.icon = data.data.icon;
        this.category.productUnit = data.data.productUnit;
        this.category.parentCid = data.data.parentCid;
      });
    },
    submitData(data) {
      if (this.dialogType == "edit") {
        this.editCategory();
      }
      if (this.dialogType == "add") {
        this.addCategory();
      }
    },
    // 添加分类
    addCategory() {
      console.log("提交的分类数据=>", this.category);
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false),
      }).then(({ data }) => {
        this.$message({
          message: "保存成功!",
          type: "success",
        });
        //关闭对话框
        this.dialogVisible = false;
        //刷新菜单
        this.getMenus();
        //设置默认展开的菜单
        this.defaulstexKey = [this.category.parentCid];
      });
    },
    editCategory() {
      var {name,catId,icon,productUnit} = this.category;//解构
      //var data = {name,catId,icon,productUnit}; key:value一致,可以省略 key. 这一步还可以省略,直接写在adornData里面
      this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "post",
        data: this.$http.adornData({name,catId,icon,productUnit}, false),
      }).then(({ data }) => {
        this.$message({
          message: "修改成功!",
          type: "success",
        });
        //关闭对话框
        this.dialogVisible = false;
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