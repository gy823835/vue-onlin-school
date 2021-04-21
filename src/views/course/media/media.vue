<template>
  <div>
    <el-row class="media-head">
      <el-col :span="16">
        <el-button
          type="primary"
          icon="el-icon-edit"
          @click="dialogFormVisible = true"
          >新增图文</el-button
        >
        <el-dialog title="新增" :visible.sync="dialogFormVisible" fullscreen>
          <el-form
            style="width: 620px"
            :model="ruleForm"
            :rules="rules"
            ref="ruleForm"
          >
            <el-form-item label="标题" prop="title" label-width="100px">
              <el-input v-model="ruleForm.title"></el-input>
            </el-form-item>
            <el-form-item label="封面" label-width="100px">
              <el-upload
                action="#"
                list-type="picture-card"
                :auto-upload="false"
              >
                <i slot="default" class="el-icon-plus"></i>
                <div slot="file" slot-scope="{ file }">
                  <img
                    class="el-upload-list__item-thumbnail"
                    :src="file.url"
                    alt=""
                  />
                  <span class="el-upload-list__item-actions">
                    <span
                      class="el-upload-list__item-preview"
                      @click="handlePictureCardPreview(file)"
                    >
                      <i class="el-icon-zoom-in"></i>
                    </span>
                    <span
                      v-if="!disabled"
                      class="el-upload-list__item-delete"
                      @click="handleDownload(file)"
                    >
                      <i class="el-icon-download"></i>
                    </span>
                    <span
                      v-if="!disabled"
                      class="el-upload-list__item-delete"
                      @click="handleRemove(file)"
                    >
                      <i class="el-icon-delete"></i>
                    </span>
                  </span>
                </div>
              </el-upload>
              <el-dialog :visible.sync="dialogVisible">
                <img width="100%" :src="dialogImageUrl" alt="" />
              </el-dialog>
            </el-form-item>
            <el-form-item label="试看内容" prop="trySee" label-width="100px">
              <Tinymce v-model="ruleForm.trySee"></Tinymce>
            </el-form-item>
            <el-form-item
              label="课程内容"
              prop="courseContent"
              label-width="100px"
            >
              <Tinymce v-model="ruleForm.courseContent"></Tinymce>
            </el-form-item>
            <el-form-item label="课程价格" prop="price" label-width="100px">
              <el-input-number
                v-model="num"
                @change="handleChange"
                :min="1"
              ></el-input-number>
            </el-form-item>
            <el-form-item label="状态" prop="status" label-width="100px">
              <el-radio v-model="radio" label="1">下架</el-radio>
              <el-radio v-model="radio" label="2">上架</el-radio>
            </el-form-item>
          </el-form>
          <div slot="footer" class="dialog-footer">
            <el-button @click="dialogFormVisible = false">取 消</el-button>
            <el-button type="primary" @click="dialogFormVisible = false"
              >提交</el-button
            >
          </div>
        </el-dialog>
      </el-col>
      <el-col :span="2" class="grid-content bg-purple-light">
        <el-select placeholder="商品状态" v-model="select">
          <el-option label="已下架" value="1"></el-option>
          <el-option label="已上架" value="2"></el-option>
        </el-select>
      </el-col>
      <el-col :span="4">
        <el-input
          v-model="listQuery.title"
          @keyup.enter.native="handleFilter"
          placeholder="标题"
          class="input-with-select"
        ></el-input>
      </el-col>
      <el-col :span="2">
        <el-button type="primary" icon="el-icon-search" @click="handleFilter">搜索</el-button>
      </el-col>
    </el-row>
    <el-table
      border
      style="width: 100%"
      v-loading="listLoading"
      @sort-change="sortChange"
      :data="list"
    >
      <el-table-column
        label="ID"
        prop="id"
        sortable="custom"
        align="center"
        width="80"
      >
        <template slot-scope="{ row }">
          <span>{{ row.id }}</span>
        </template>
      </el-table-column>
      <el-table-column label="图文内容" min-width="150px">
        <template slot-scope="{ row }">
          <img
            :src="row.cover"
            alt=""
            style="width: 100px; float: left; margin-right: 10px"
          />
          <span>
            {{ row.title }}
          </span>
          <br />
          <span style="color: red">￥{{ row.price }}</span>
        </template>
      </el-table-column>
      <el-table-column label="阅读量" align="center" width="95">
        <template slot-scope="row">
          <span v-if="row.sub_count">{{ row.sub_count }}</span>
          <span v-else>0</span>
        </template>
      </el-table-column>
      <el-table-column label="状态" width="100">
        <template slot-scope="{ row }">
          <el-tag :type="row.status | statusFilter">
            {{ row.status ? "已下架" : "已上架" }}
          </el-tag>
        </template>
      </el-table-column>
      <el-table-column label="时间" width="150px" align="center">
        <template slot-scope="{ row }">
          <span>{{ row.created_time | parseTime("{y}-{m}-{d} {h}:{i}") }}</span>
        </template>
      </el-table-column>
      <el-table-column
        label="操作"
        width="230"
        class-name="small-padding fixed-width"
      >
        <template slot-scope="{ row, $index }">
          <el-button
            type="primary"
            size="mini"
            @click="dialogFormVisible = true"
          >
            编辑
          </el-button>
          <el-dialog title="修改" :visible.sync="dialogFormVisible" fullscreen>
            <el-form
              style="width: 620px"
              :model="ruleForm"
              :rules="rules"
              ref="ruleForm"
            >
              <el-form-item label="标题" prop="title" label-width="100px">
                <el-input v-model="ruleForm.title"></el-input>
              </el-form-item>
              <el-form-item label="封面" label-width="100px">
                <el-upload
                  action="#"
                  list-type="picture-card"
                  :auto-upload="false"
                >
                  <i slot="default" class="el-icon-plus"></i>
                  <div slot="file" slot-scope="{ file }">
                    <img
                      class="el-upload-list__item-thumbnail"
                      :src="file.url"
                      alt=""
                    />
                    <span class="el-upload-list__item-actions">
                      <span
                        class="el-upload-list__item-preview"
                        @click="handlePictureCardPreview(file)"
                      >
                        <i class="el-icon-zoom-in"></i>
                      </span>
                      <span
                        v-if="!disabled"
                        class="el-upload-list__item-delete"
                        @click="handleDownload(file)"
                      >
                        <i class="el-icon-download"></i>
                      </span>
                      <span
                        v-if="!disabled"
                        class="el-upload-list__item-delete"
                        @click="handleRemove(file)"
                      >
                        <i class="el-icon-delete"></i>
                      </span>
                    </span>
                  </div>
                </el-upload>
                <el-dialog :visible.sync="dialogVisible">
                  <img width="100%" :src="dialogImageUrl" alt="" />
                </el-dialog>
              </el-form-item>
              <el-form-item label="试看内容" prop="trySee" label-width="100px">
                <Tinymce v-model="ruleForm.trySee"></Tinymce>
              </el-form-item>
              <el-form-item
                label="课程内容"
                prop="courseContent"
                label-width="100px"
              >
                <Tinymce v-model="ruleForm.courseContent"></Tinymce>
              </el-form-item>
              <el-form-item label="课程价格" prop="price" label-width="100px">
                <el-input-number
                  v-model="num"
                  @change="handChange"
                  :min="1"
                ></el-input-number>
              </el-form-item>
              <el-form-item label="状态" prop="status" label-width="100px">
                <el-radio v-model="radio" label="1">下架</el-radio>
                <el-radio v-model="radio" label="2">上架</el-radio>
              </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
              <el-button @click="dialogFormVisible = false">取 消</el-button>
              <el-button type="primary" @click="dialogFormVisible = false"
                >提交</el-button
              >
            </div>
          </el-dialog>
          <el-button
            v-if="row.status != '0'"
            size="mini"
            type="success"
            @click="handleModifyStatus(row, 0)"
          >
            上架
          </el-button>
          <el-button
            v-if="row.status != '1'"
            size="mini"
            @click="handleModifyStatus(row, 1)"
          >
            下架
          </el-button>
          <el-button
            v-if="row.status != 'deleted'"
            size="mini"
            type="danger"
            @click="handleDelete(row, $index)"
          >
            删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <pagination
      v-show="total > 0"
      :total="total"
      :page.sync="listQuery.page"
      :limit.sync="listQuery.limit"
      @pagination="getList"
    />
  </div>
</template>
<script>
import { fetchList, createMedia, updateMedia, deleteMedia } from "@/api/media";
import Tinymce from "@/components/Tinymce";
import Pagination from "@/components/Pagination";
export default {
  name: "Media",
  components: { Tinymce, Pagination },
  filters: {
    statusFilter(status) {
      const statusMap = {
        0: "success",
        1: "danger",
      };
      return statusMap[status];
    },
  },
  data() {
    return {
      radio: "1",
      num: 1,
      total: 0,
      listQuery: {
        page: 1,
        limit: 20,
        importance: undefined,
        title: undefined,
        type: undefined,
        sort: "+id",
      },
      dialogImageUrl: "",
      dialogVisible: false,
      disabled: false,
      listLoading: true,
      tableKey: 0,
      list: null,
      select: "",
      dialogFormVisible: false,
      ruleForm: {
        title: "",
        trySee: "",
        courseContent: "",
      },
      rules: {
        title: { required: true, message: "标题不能为空", trigger: "blur" },
        trySee: { required: true, message: "内容不能为空", trigger: "blur" },
        courseContent: {
          required: true,
          message: "内容不能为空",
          trigger: "blur",
        },
      },
    };
  },
  created() {
    this.getList();
  },
  methods: {
    getList() {
      this.listLoading = true;
      fetchList(this.listQuery).then((response) => {
        console.log(response);
        this.list = response.data.items;
        this.total = response.data.total;

        // Just to simulate the time of the request
        setTimeout(() => {
          this.listLoading = false;
        }, 1.5 * 1000);
      });
    },
    handleFilter() {
      this.listQuery.page = 1
      this.getList()
    },
    handleModifyStatus(row, status) {
      this.$message({
        message: "操作Success",
        type: "success",
      });
      row.status = status;
    },
    sortChange(data) {
      const { prop, order } = data;
      if (prop === "id") {
        this.sortByID(order);
      }
    },
    sortByID(order) {
      if (order === 'ascending') {
        this.listQuery.sort = '+id'
      } else {
        this.listQuery.sort = '-id'
      }
      this.handleFilter()
    },
    handleDelete(row, index) {
      this.$notify({
        title: "Success",
        message: "删除成功",
        type: "success",
        duration: 2000,
      });
      this.list.splice(index, 1);
    },
    handleRemove(file) {
      console.log(file);
    },
    handlePictureCardPreview(file) {
      this.dialogImageUrl = file.url;
      this.dialogVisible = true;
    },
    handleDownload(file) {
      console.log(file);
    },
    handleChange(value) {
      console.log(value);
    },
    handChange(value) {
      console.log(value);
    },
  },
};
</script>
<style>
.el-select {
  width: 110px;
}
.media-head {
  margin: 20px;
}
.el-table {
  margin: 20px;
}
</style>
