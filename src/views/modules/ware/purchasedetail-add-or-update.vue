<template>
  <el-dialog :title="!dataForm.id ? '新增' : '修改'" :close-on-click-modal="false" :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()"
      label-width="100px">
      <el-row>
        <el-col :span="12">
          <el-form-item label="采购商品" prop="skuId">
            <el-select v-model="dataForm.skuId" placeholder="请选择商品" style="width: 95%">
              <el-option v-for="sku in skus" :label="sku.skuName" :value="sku.skuId" :key="sku.skuId"></el-option>
            </el-select>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="采购数量" prop="skuNum">
            <el-input-number v-model="dataForm.skuNum" :min="1" placeholder="采购数量"></el-input-number>
          </el-form-item>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="12">
          <el-form-item label="仓库" prop="wareId">
            <el-select v-model="dataForm.wareId" placeholder="请选择仓库" style="width: 95%">
              <el-option :label="ware.name" :value="ware.id" v-for="ware in wareList" :key="ware.id"></el-option>
            </el-select>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <!-- [0新建，1已分配，2正在采购，3已完成，4采购失败] -->
          <el-form-item label="状态" prop="status" v-if="this.dataForm.id">
            <el-select v-model="dataForm.status" placeholder="请选择状态">
              <el-option label="新建" :value="0"></el-option>
              <el-option label="已分配" :value="1"></el-option>
              <el-option label="正在采购" :value="2"></el-option>
              <el-option label="已完成" :value="3"></el-option>
              <el-option label="采购失败" :value="4"></el-option>
            </el-select>
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
        purchaseId: "",
        skuId: "",
        skuNum: "",
        skuPrice: "",
        wareId: "",
        status: 0,
      },
      skus: [],
      wareList: [],
      dataRule: {
        skuId: [{ required: true, message: "采购商品不能为空", trigger: "blur" }],
        skuNum: [
          { required: true, message: "采购数量不能为空", trigger: "blur" },
        ],
        wareId: [{ required: true, message: "仓库不能为空", trigger: "blur" }],
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
      this.visible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl(
              `/ware/purchasedetail/info/${this.dataForm.id}`
            ),
            method: "get",
            params: this.$http.adornParams(),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm.purchaseId = data.purchaseDetail.purchaseId;
              this.dataForm.skuId = data.purchaseDetail.skuId;
              this.dataForm.skuNum = data.purchaseDetail.skuNum;
              this.dataForm.skuPrice = data.purchaseDetail.skuPrice;
              this.dataForm.wareId = data.purchaseDetail.wareId;
              this.dataForm.status = data.purchaseDetail.status;
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
              `/ware/purchasedetail/${!this.dataForm.id ? "save" : "update"}`
            ),
            method: "post",
            data: this.$http.adornData({
              id: this.dataForm.id || undefined,
              purchaseId: this.dataForm.purchaseId,
              skuId: this.dataForm.skuId,
              skuNum: this.dataForm.skuNum,
              skuPrice: this.skus.find(sku => sku.skuId === this.dataForm.skuId).price * this.dataForm.skuNum,
              wareId: this.dataForm.wareId,
              status: this.dataForm.status,
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
