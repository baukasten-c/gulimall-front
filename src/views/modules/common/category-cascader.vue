<template>
  <!--  使用说明： 1）、引入category-cascader.vue
                  2）、语法：<category-cascader :catelogPath.sync="catelogPath"></category-cascader>
        解释：catelogPath：指定的值是cascader初始化需要显示的值，应该和父组件的catelogPath绑定;
          由于有sync修饰符，所以cascader路径变化以后自动会修改父的catelogPath，这是结合子组件this.$emit("update:catelogPath",v);做的
      -->
  <div>
    <!-- options：可选项数据源，键名可通过 Props 属性配置 -->
    <el-cascader filterable clearable placeholder="试试搜索：手机" v-model="paths" :options="categorys" :props="setting"
      style="width:330px"></el-cascader>
  </div>
</template>

<script>
export default {
  // 接受父组件传来的值
  props: {
    catelogPath: {
      type: Array,
      default() {
        return [];
      },
    },
  },
  data() {
    return {
      setting: {
        value: "catId",
        label: "name",
        children: "children",
      },
      categorys: [],
      paths: this.catelogPath,
    };
  },
  watch: {
    catelogPath(v) {
      this.paths = this.catelogPath;
    },
    paths(v) {
      this.$emit("update:catelogPath", v);
      // 还可以使用pubsub-js进行传值
      PubSub.publish("catPath", v);
    },
  },
  created() {
    this.getCategorys();
  },
  methods: {
    getCategorys() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
      }).then(({ data }) => {
        this.categorys = data.data;
      });
    },
  },
};
</script>