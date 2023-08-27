<template>
  <!-- template标签下只允许有一个标签 -->
  <div>
    <el-switch v-model="draggable" active-text="开启拖动" inactive-text="关闭拖动"></el-switch>
    <el-button size="mini" round style="float: right" type="danger" @click="batchDelete">批量删除</el-button>
    <el-button size="mini" round style="float: right" type="primary" v-if="draggable"
      @click="batchSave">批量保存</el-button>
    <br><br>
    <!-- expand-on-click-node:是否在点击节点的时候展开或者收缩节点， 默认值为 true，如果为 false，则只有点箭头图标的时候才会展开或者收缩节点 -->
    <!-- show-checkbox:节点是否可被选择 -->
    <!-- node-key:每个树节点用来作为唯一标识的属性，整棵树应该是唯一的 -->
    <!-- default-expanded-keys:默认展开的节点的 key 的数组 -->
    <!-- node-expand:节点被展开时触发的事件 -->
    <!-- node-collapse:节点被关闭时触发的事件 -->
    <!-- draggable:是否开启拖拽节点功能 -->
    <!-- allow-drop:拖拽时判定目标节点能否被放置。type 参数有三种情况：'prev'、'inner' 和 'next'，分别表示放置在目标节点前、插入至目标节点和放置在目标节点后 -->
    <!-- node-drop:拖拽成功完成时触发的事件 -->
    <!-- ref:用于在模板中给元素或组件添加一个引用标识，使得我们可以通过该引用标识在 JavaScript 中访问该元素或组件的实例 -->
    <el-tree :data="menus" :props="defaultProps" :expand-on-click-node="false" show-checkbox node-key="catId"
      :default-expanded-keys="expandedKeys" @node-expand="handleNodeExpand" @node-collapse="handleNodeCollapse"
      :draggable="draggable" :allow-drop="allowDrop" @node-drop="handleDrop" ref="menuTree">
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <!-- 一级和二级菜单可以添加 -->
          <el-button v-if="node.level <= 2" type="text" size="mini" @click="() => append(data)">添加</el-button>
          <el-button type="text" size="mini" @click="() => edit(data)">修改</el-button>
          <!-- 无子菜单的菜单可以删除 -->
          <el-button v-if="!data.children.length" type="text" size="mini"
            @click="() => remove(node, data)">删除</el-button>
        </span>
      </span>
    </el-tree>
    <!-- visible:是否显示 Dialog，支持 .sync 修饰符，不加.sync会使关闭按钮失效 -->
    <!-- close-on-click-modal:点击空白处是否关闭弹窗 -->
    <el-dialog :title="title" :visible.sync="dialogVisible" width="25%" :close-on-click-modal="false">
      <el-form :model="category">
        <el-form-item label="分类名称" label-width="70px">
          <!-- autocomplete:控制浏览器是否自动填充表单字段 -->
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标" label-width="70px">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位" label-width="70px">
          <el-input v-model="category.productUnit" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer">
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
      menus: [],
      defaultProps: {
        children: "children",
        label: "name",
      },
      expandedKeys: [],
      dialogVisible: false,
      category: {
        catId: null, // 后端接口修改时用到id值
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        icon: null,
        productUnit: null,
      },
      dialogType: "", // edit或add
      title: "",
      pCid: [],
      updateNodes: new Map(),
      draggable: false,
    };
  },
  created() {
    this.getMenus();
  },
  methods: {
    getMenus() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
      }).then(({ data }) => {
        // {data}表示从对象中解构出名为 data 的属性
        this.menus = data.data; //data.data即获取的三级菜单
      });
    },
    // 在展开菜单时记录其父菜单的ID到expandedKeys数组中
    handleNodeExpand(data) {
      if (data.catId && !this.expandedKeys.includes(data.catId)) {
        this.expandedKeys.push(data.catId);
      }
    },
    // 在菜单合并时从expandedKeys数组中移除相应的菜单ID
    handleNodeCollapse(data) {
      const index = this.expandedKeys.indexOf(data.catId);
      // 在expandedKeys数组中移除index位置的1个元素
      this.expandedKeys.splice(index, 1);
    },
    submitData() {
      if (this.dialogType == "add") {
        this.addCategory();
      } else if (this.dialogType == "edit") {
        this.editCategory();
      }
    },
    append(data) {
      this.dialogType = "add";
      this.title = "添加分类";
      this.dialogVisible = true;
      // 表单清空
      this.clear();
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1; //data.catLevel * 1使数据由字符串转为数字
    },
    // 添加三级分类
    addCategory() {
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false),
      }).then(({ data }) => {
        this.$message({
          type: "success",
          message: "菜单添加成功!",
        });
        this.dialogVisible = false;
        // 刷新出新的菜单
        this.getMenus();
        // 设置需要默认展开的菜单
        this.expandedKeys.push(data.catId);
      });
    },
    edit(data) {
      this.dialogType = "edit";
      this.title = "修改分类";
      this.dialogVisible = true;
      // 发送请求获取当前节点最新数据
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: "get",
      }).then(({ data }) => {
        this.category = { ...data.data };
      });
    },
    // 修改三级分类
    editCategory() {
      // 如果在edit()中只回显了catId,name,icon,productUnit四个数据，需要使用var {catId,name,icon,productUnit} = this.category;将它们抽取出来，否则其它数据会被默认值覆盖
      this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "post",
        data: this.$http.adornData(this.category, false),
      }).then(({ data }) => {
        this.$message({
          type: "success",
          message: "菜单修改成功!",
        });
        this.dialogVisible = false;
        // 刷新出新的菜单
        this.getMenus();
        // 设置需要默认展开的菜单
        this.expandedKeys.push(data.catId);
      });
    },
    // 清空表单数据
    clear() {
      this.category.catId = null;
      this.category.name = "";
      this.category.parentCid = 0;
      this.category.catLevel = 0;
      this.category.icon = null;
      this.category.productUnit = null;
    },
    remove(node, data) {
      var ids = [data.catId];
      this.$confirm(`是否删除[${data.name}]菜单？`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false),
          })
            .then(({ msg }) => {
              this.$message({
                type: "success",
                message: "菜单删除成功!",
              });
              // 刷新出新的菜单
              this.getMenus();
              // 设置需要默认展开的菜单
              this.expandedKeys.push(data.parentCid);
            })
            .catch(() => {});
        })
        .catch(() => {});
    },
    // 判断是否可以拖动，draggingNode:被拖动的当前节点，dropNode:要插入的目标节点
    allowDrop(draggingNode, dropNode, type) {
      // 被拖动的当前节点的总层数
      // const关键字表示这些变量是常量，即它们的值在声明后不可更改
      const maxLevel = this.countNodeLevel(draggingNode);
      // 拖动层数=总层数-当前节点层数+1
      const deep = Math.abs(maxLevel - draggingNode.level) + 1;
      // 拖动后的节点总层数不能大于3
      if (type == "inner") {
        return deep + dropNode.level <= 3;
      } else {
        return deep + dropNode.parent.level <= 3;
      }
    },
    // 遍历所有子节点，找总层数
    countNodeLevel(node) {
      let maxLevel = node.level;
      if (node.childNodes != null && node.childNodes.length > 0) {
        // node.childNodes是Array数组
        for (let i = 0; i < node.childNodes.length && maxLevel < 3; i++) {
          if (node.childNodes[i].level > maxLevel) {
            maxLevel = node.childNodes[i].level;
          }
          maxLevel = this.countNodeLevel(node.childNodes[i]);
        }
      }
      return maxLevel;
    },
    // draggingNode:被拖动的当前节点，dropNode:结束拖动后进入的节点，dropType:被拖动节点的放置位置（before、after、inner）、event
    handleDrop(draggingNode, dropNode, dropType, ev) {
      // 获取当前节点最新的父节点id、同级节点
      let pCid = 0;
      let siblings = [];
      if (dropType == "inner") {
        pCid = dropNode.data.catId;
        siblings = dropNode.childNodes;
      } else {
        pCid = dropNode.data.parentCid;
        siblings = dropNode.parent.childNodes;
      }
      this.pCid.push(pCid);
      // 修改被拖动的当前的最新顺序
      for (let i = 0; i < siblings.length; i++) {
        // 如果遍历到被拖动的当前节点，还需要多修改层级和父节点id
        if (siblings[i].data.catId == draggingNode.data.catId) {
          // 定义catLevel为被拖动的当前节点的原始层级
          let catLevel = draggingNode.data.catLevel;
          // 被拖动的当前节点的层级发送变化
          if (siblings[i].level != catLevel) {
            catLevel = siblings[i].level;
            // 修改被拖动的当前节点的子节级层次
            this.updateChildNodeLevel(siblings[i]);
          }
          this.updateNodes.set(siblings[i].data.catId, {
            catId: siblings[i].data.catId,
            parentCid: pCid,
            catLevel: catLevel,
            sort: i,
          });
        } else {
          this.updateNodes.set(siblings[i].data.catId, {
            catId: siblings[i].data.catId,
            sort: i,
          });
        }
      }
    },
    // 修改子节级层次
    updateChildNodeLevel(node) {
      for (let i = 0; i < node.childNodes.length; i++) {
        var cNode = node.childNodes[i];
        this.updateNodes.set(cNode.data.catId, {
          catId: cNode.data.catId,
          catLevel: cNode.level,
        });
        this.updateChildNodeLevel(cNode);
      }
    },
    batchSave() {
      const updateNodes = Array.from(this.updateNodes.values());
      this.$http({
        url: this.$http.adornUrl("/product/category/update/sort"),
        method: "post",
        data: this.$http.adornData(updateNodes, false),
      }).then(({ data }) => {
        this.$message({
          type: "success",
          message: "菜单顺序修改成功!",
        });
        // 刷新出新的菜单
        this.getMenus();
        // 设置需要默认展开的菜单
        this.expandedKeys.push(this.pCid);
        // 清空
        this.updateNodes = new Map();
        this.pCid = [];
      });
    },
    batchDelete() {
      // this.$refs:用于访问模板中使用 ref 引用标识的元素或组件实例
      // getCheckedKeys:若节点可被选择，则返回目前被选中的节点的 key 所组成的数组
      let catIds = this.$refs.menuTree.getCheckedKeys();
      // 被选中的节点的 name 所组成的数组
      let names = this.$refs.menuTree.getCheckedNodes().map((node) => node.name);
      let halfIds = this.$refs.menuTree.getHalfCheckedNodes();
      this.$confirm(`是否批量删除[${names}]菜单？`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(catIds, false),
          })
            .then(({ msg }) => {
              this.$message({
                type: "success",
                message: "菜单删除成功!",
              });
              // 刷新出新的菜单
              this.getMenus();
              // 设置需要默认展开的菜单
              this.expandedKeys.push(halfIds);
            })
            .catch(() => {});
        })
        .catch(() => {});
    },
  },
};
</script>

<style>
.custom-tree-node {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: space-between;
  font-size: 14px;
  padding-right: 8px;
}
</style>