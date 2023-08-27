<template>
  <div>
    <el-input placeholder="输入关键字进行过滤" v-model="filterText"></el-input>
    <!-- filter-node-method:对树节点进行筛选时执行的方法，返回 true 表示这个节点可以显示，返回 false 则表示这个节点会被隐藏 -->
    <!-- highlight-current:是否高亮当前选中节点，默认值是 false -->
    <el-tree :data="menus" :props="defaultProps" node-key="catId" ref="menuTree" @node-click="nodeClick"
      :filter-node-method="filterNode" :highlight-current="true">
    </el-tree>
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
      filterText: "",
    };
  },
  created() {
    this.getMenus();
  },
  watch: {
    filterText(val) {
      if(!val){
        this.getMenus()
      }
      this.$refs.menuTree.filter(val);
    },
  },
  methods: {
    // 树节点过滤
    filterNode(value, data) {
      // 如果过滤条件值 value 为空，所有节点都被保留
      if (!value) return true;
      // 如果过滤条件值不为空，只有那些节点的名称包含了过滤条件值才会被保留
      return data.name.indexOf(value) !== -1;
    },
    getMenus() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
      }).then(({ data }) => {
        this.menus = data.data;
      });
    },
    // 节点被点击时向父组件发送事件
    // data：该节点所对应的对象，node：节点对应的 Node，component：节点组件本身
    nodeClick(data, node, component) {
      // 触发自定义事件后，子组件向父组件传递信息或触发特定的行为
      // this.$emit(eventName, ...args) eventName：要触发的事件名称，args：传递给事件的可选参数
      this.$emit("tree-node-click", data, node, component);
    },
  },
};
</script>
