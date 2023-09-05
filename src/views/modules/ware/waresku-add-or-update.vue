<template>
  <el-dialog :title="!dataForm.id ? '新增' : '修改'" :close-on-click-modal="false" :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()"
      label-width="80px">
      <el-row>
        <el-col :span="13">
          <el-form-item label="仓库" prop="wareId">
            <el-select v-model="dataForm.wareId" placeholder="请选择仓库" style="width: 90%">
              <el-option :label="ware.name" :value="ware.id" v-for="ware in wareList" :key="ware.id"></el-option>
            </el-select>
          </el-form-item>
        </el-col>
        <el-col :span="11">
          <el-form-item label="库存数" prop="stock">
            <el-input-number v-model="dataForm.stock" :min="1"></el-input-number>
          </el-form-item>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="13">
          <el-form-item label="商品" prop="skuId">
            <el-select v-model="dataForm.skuId" placeholder="请选择商品" style="width: 90%">
              <el-option v-for="sku in skus" :label="sku.skuName" :value="sku.skuId" :key="sku.skuId"></el-option>
            </el-select>
          </el-form-item>
        </el-col>
        <el-col :span="11">
          <el-form-item label="锁定库存" prop="stockLocked">
            <el-switch v-model="dataForm.stockLocked" active-color="#13ce66" inactive-color="#ff4949" :active-value="1"
              :inactive-value="0"></el-switch>
          </el-form-item>
        </el-col>
      </el-row>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
export default {
  data() {
    return {
      visible: false,
      dataForm: {
        id: 0,
        skuId: "",
        wareId: "",
        stock: "",
        skuName: "",
        stockLocked: "",
      },
      skus: [],
      wareList: [],
      dataRule: {
        wareId: [{ required: true, message: "仓库不能为空", trigger: "blur" }],
        stock: [{ required: true, message: "库存数不能为空", trigger: "blur" }],
        skuId: [{ required: true, message: "商品不能为空", trigger: "blur" }],
        stockLocked: [
          { required: true, message: "锁定库存不能为空", trigger: "blur" },
        ],
      },
    };
  },
  created() {
    this.getSkus();
    this.getWares();
  },
  methods: {
    getSkus() {
      this.$http({
        url: this.$http.adornUrl("/product/skuinfo/sku/list"),
        method: "get",
      }).then(({ data }) => {
        this.skus = data.data;
      });
    },
    getWares() {
      this.$http({
        url: this.$http.adornUrl("/ware/wareinfo/list"),
        method: "get",
        params: this.$http.adornParams({
          page: 1,
          limit: 500,
        }),
      }).then(({ data }) => {
        this.wareList = data.page.list;
      });
    },
    init(id) {
      this.dataForm.id = id || 0;
      this.dataForm.skuId = "";
      this.visible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl(`/ware/waresku/info/${this.dataForm.id}`),
            method: "get",
            params: this.$http.adornParams(),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm.wareId = data.wareSku.wareId;
              this.dataForm.stock = data.wareSku.stock;
              this.dataForm.stockLocked = data.wareSku.stockLocked;
              this.dataForm.skuId = data.wareSku.skuId;
              this.dataForm.skuName = data.wareSku.skuName;
            }
          });
        }
      });
    },
    // 表单提交
    dataFormSubmit() {
      this.$refs["dataForm"].validate((valid) => {
        if (valid) {
          this.$http({
            url: this.$http.adornUrl(
              `/ware/waresku/${!this.dataForm.id ? "save" : "update"}`
            ),
            method: "post",
            data: this.$http.adornData({
              id: this.dataForm.id || undefined,
              skuId: this.dataForm.skuId,
              wareId: this.dataForm.wareId,
              stock: this.dataForm.stock,
              skuName: this.skus.find(sku => sku.skuId === this.dataForm.skuId).skuName,
              stockLocked: this.dataForm.stockLocked,
            }),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.$message({
                message: "操作成功",
                type: "success",
                duration: 1500,
                onClose: () => {
                  this.visible = false;
                  this.$emit("refreshDataList");
                },
              });
            } else {
              this.$message.error(data.msg);
            }
          });
        }
      });
    },
  },
};
</script>
