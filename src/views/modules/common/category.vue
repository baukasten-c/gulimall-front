<template>
  <el-tree :data="menus" :props="defaultProps" node-key="catId" ref="menuTree" @node-click="nodeClick">
  </el-tree>
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
        this.menus = data.data;
      });
    },
    //节点被点击时向父组件发送事件
    //data：该节点所对应的对象，node：节点对应的 Node，component：节点组件本身
    nodeClick(data, node, component) { 
      //触发自定义事件后，子组件向父组件传递信息或触发特定的行为
      //this.$emit(eventName, ...args) eventName：要触发的事件名称，args：传递给事件的可选参数
      this.$emit("tree-node-click", data, node, component);
    },
  },
};
</script>

<style>
</style>