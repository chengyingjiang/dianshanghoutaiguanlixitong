<template>
  <div class="goods">
    <bread-crumb :menuList="menuList"></bread-crumb>
    <el-card class="cardlist">
      <el-row :gutter="20">
        <el-col :span="6">
          <div class="grid-content bg-purple">
            <el-input
              placeholder="请输入搜索内容"
              v-model="paramsObj.query"
              class="input-with-select"
            >
              <el-button
                @click="getGoodsData"
                slot="append"
                icon="el-icon-search"
              ></el-button>
            </el-input>
          </div>
        </el-col>
        <el-col :span="18">
          <div class="grid-content bg-purple">
            <el-button type="primary" @click="addGoodsVisible = true"
              >添加商品</el-button
            >
          </div>
        </el-col>
      </el-row>
      <el-table class="cardlist" :data="goods" stripe border>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column prop="goods_name" label="商品名称"></el-table-column>
        <el-table-column
          prop="goods_price"
          label="商品价格（元）"
        ></el-table-column>
        <el-table-column prop="goods_weight" label="商品重量"></el-table-column>
        <el-table-column prop="add_time" label="创建时间"></el-table-column>
        <el-table-column label="操作">
          <template slot-scope="{ row }">
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="edit(row)"
            ></el-button>
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="deleteFn(row.goods_id)"
            ></el-button>
          </template>
        </el-table-column>
      </el-table>
      <pagin-ation
        :page.sync="paramsObj.pagenum"
        :limit.sync="paramsObj.pagesize"
        :total="total"
        @pagination="getGoodsData"
      ></pagin-ation>
    </el-card>
    <!-- 添加商品的对话框 -->
    <el-dialog :visible.sync="addGoodsVisible" title="添加商品">
      <el-form :model="goodsForm" :rules="goodsRules" ref="addGoodsRef">
        <el-form-item
          prop="goods_name"
          label="商品名称"
          :label-width="formLabelWidth"
        >
          <el-input v-model="goodsForm.goods_name" />
        </el-form-item>
        <el-form-item
          prop="goods_price"
          label="商品价格"
          :label-width="formLabelWidth"
        >
          <el-input type="number" v-model="goodsForm.goods_price" />
        </el-form-item>
        <el-form-item
          prop="goods_number"
          label="商品数量"
          :label-width="formLabelWidth"
        >
          <el-input type="number" v-model="goodsForm.goods_number" />
        </el-form-item>
        <el-form-item
          prop="goods_weight"
          label="商品重量"
          :label-width="formLabelWidth"
        >
          <el-input type="number" v-model="goodsForm.goods_weight" />
        </el-form-item>
        <el-form-item label="商品图片" :label-width="formLabelWidth">
          <!-- 
          action	必选参数，上传的地址	string	—	—
          headers	设置上传的请求头部	object
          on-remove	文件列表移除文件时的钩子	function(file, fileList)
          list-type	文件列表的类型	string	text/picture/picture-card	text
          on-success	文件上传成功时的钩子	function(response, file, fileList)	
         -->
          <el-upload
            :action="uploadUrl"
            :headers="headersObj"
            :on-remove="handlerRemove"
            list-type="picture"
            :on-success="handlerSuccess"
          >
            <el-button type="primary" size="small">点击上传</el-button>
          </el-upload>
        </el-form-item>
      </el-form>
      <template #footer>
        <div class="dialog-footer">
          <el-button @click="addGoodsVisible = false">取消</el-button>
          <el-button type="primary" @click="addGoods">确定</el-button>
        </div>
      </template>
    </el-dialog>
    <el-dialog title="编辑商品" :visible.sync="editDialogVisible">
      <el-form ref="editGoodsRef" :model="goodsForm" :rules="goodsRules">
        <el-form-item label="商品名称" prop="goods_name" label-width="80px">
          <el-input v-model="goodsForm.goods_name"></el-input>
        </el-form-item>
        <el-form-item label="商品价格" prop="goods_price" label-width="80px">
          <el-input v-model="goodsForm.goods_price"></el-input>
        </el-form-item>
        <el-form-item label="商品数量" prop="goods_number" label-width="80px">
          <el-input v-model="goodsForm.goods_number"></el-input>
        </el-form-item>
        <el-form-item label="商品重量" prop="goods_weight" label-width="80px">
          <el-input v-model="goodsForm.goods_weight"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editSubmit">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import { getGoodsList, addGoods, editGoods, deleteGoods } from "@/api/good";
import { getToken } from "@/utils/auth";
import BreadCrumb from "@/components/BreadCrumb.vue";
import PaginAtion from "@/components/PaginAtion.vue";
export default {
  components: { BreadCrumb, PaginAtion },
  data() {
    return {
      menuList: ["商品管理", "商品列表"],
      goods: [],
      paramsObj: {
        query: "",
        pagenum: 1,
        pagesize: 5,
      },
      total: 0,
      uploadUrl: "http://shiyansong.cn:8888/api/private/v1/upload",
      headersObj: { Authorization: getToken() },
      addGoodsVisible: false,
      editDialogVisible: false,
      goodsForm: {
        goods_name: "", //商品名称
        goods_price: 0, //商品价格
        goods_number: 0, //商品数量
        goods_weight: 0, //商品重量
        pics: [], //上传图片的临时路径
      },
      goodsRules: {
        goods_name: [
          { required: true, message: "商品名称不能为空", trigger: "blur" },
        ],
        goods_price: [
          { required: true, message: "商品价格不能为空", trigger: "blur" },
        ],
        goods_number: [
          { required: true, message: "商品数量不能为空", trigger: "blur" },
        ],
        goods_weight: [
          { required: true, message: "商品重量不能为空", trigger: "blur" },
        ],
      },
      formLabelWidth: "120px",
    };
  },
  mounted() {
    this.getGoodsData();
  },
  methods: {
    async getGoodsData() {
      const { goods, total } = await getGoodsList(this.paramsObj);
      this.goods = goods;
      this.total = total;
    },
    handleSizeChange(v) {
      this.paramsObj.pagesize = v;
      this.getGoodsData();
    },
    handleCurrentChange(v) {
      this.paramsObj.pagenum = v;
      this.getGoodsData();
    },
    // 文件上传成功触发的函数
    handlerSuccess(res) {
      let {
        meta: { status },
        data: { tmp_path },
      } = res;
      if (status === 200) {
        this.goodsForm.pics.push({ pic: tmp_path });
      }
    },
    // 删除图片时触发的函数
    handlerRemove(file) {
      // 找到删除图片的临时路径
      const tmpPath = file.response.data.tmp_path;
      // 根据路径找到要删除图片的下标
      const idx = this.goodsForm.pics.findIndex((item) => item.pic === tmpPath);
      // 使用splice实现删除
      this.goodsForm.pics.splice(idx, 1);
    },
    // 添加商品
    addGoods() {
      this.$refs.addGoodsRef.validate(async (isOk) => {
        if (isOk) {
          await addGoods({ ...this.goodsForm });
          // 重新渲染页面
          this.getGoodsData();
          // 关闭添加商品的弹框
          this.addGoodsVisible = false;
          // 提示删除成功
          this.$message.success("商品添加成功");
        } else {
          console.log("验证错误");
        }
      });
    },
    edit(row) {
      this.goodsForm = row;
      this.editDialogVisible = true;
    },
    editSubmit() {
      this.$refs.editGoodsRef.validate(async (isOk) => {
        if (isOk) {
          await editGoods(this.goodsForm.goods_id, this.goodsForm);
          this.editDialogVisible = false;
          this.$message.success("修改成功");
        } else {
          console.log("验证错误");
        }
      });
    },
    async deleteFn(id) {
      this.$confirm("确定要永久删除该条数据吗?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(async () => {
          // 点击确定 请求删除的api接口
          await deleteGoods(id);
          // 重新渲染数据
          this.getGoodsData();
          // 提示
          this.$message({
            type: "success",
            message: "删除成功!",
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除",
          });
        });
    },
  },
};
</script>

<style lang='less' scoped>
.cardlist {
  margin-top: 15px;
}
</style>