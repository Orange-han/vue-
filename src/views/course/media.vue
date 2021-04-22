<template>
  <div class="app-container">
    <div class="filter-container">
      <!-- 表格头部 -->
      <el-row :gutter="24">
        <el-col :span="4">
          <el-button
            class="filter-item"
            style="margin-left: 10px"
            type="primary"
            icon="el-icon-edit"
            @click="handleCreate"
          >
            新增图文
          </el-button>
        </el-col>
        <el-col :span="10" :offset="10">
          <el-select
           v-model="listQuery.importance"
            placeholder="商品状态"
            clearable
            style="width: 90px"
            class="filter-item"
          >
            <el-option
              v-for="item in importanceOptions"
              :key="item"
              :label="item"
              :value="item"
            />
          </el-select>
          <el-input
            placeholder="标题"
            style="width: 120px"
            class="filter-item"
          ></el-input>
          <el-button
            v-waves
            class="filter-item"
            type="primary"
            icon="el-icon-search"
            @click="handleFilter"
          >
            搜索
          </el-button>
        </el-col>
      </el-row>
    </div>
    <!-- 表格部分 -->
    <el-table
      :key="tableKey"
      :data="list"
      border
      fit
      highlight-current-row
      style="width: 100%"
      @sort-change="sortChange"
    >
      <el-table-column
        label="ID"
        prop="id"
        sortable="custom"
        align="center"
        width="80"
        :class-name="getSortClass('id')"
      >
        <template slot-scope="{ row }">
          <span>{{ row.id }}</span>
        </template>
      </el-table-column>
      <el-table-column label="图文内容" width="360px" align="center">
        <template slot-scope="{ row }">
          <div style="display:flex; flex-dirction:row">
            <img :src="row.cover" style="height:50px" />
          
          <div style="margin-left:10px" text-align="left">
            <span >{{ row.title }}</span> <br/>
            <span style="color:red">￥{{ row.price }}</span>
          </div>
          </div>
        </template>
      </el-table-column>
      <el-table-column label="订阅量" align="center" width="95">
        <template slot-scope="{ row }">
          <span>{{ row.sub_count }}</span
          >
        </template>
      </el-table-column>
      <el-table-column label="状态" class-name="status-col" width="110px" align="center">
        <template slot-scope="{ row }">
          <el-tag :type="row.status | statusFilter" >
            {{ row.status ===1 ? "已上架": "已下架"}}
          </el-tag>
        </template>
      </el-table-column>
      <el-table-column
        v-if="showReviewer"
        label="Reviewer"
        width="110px"
        align="center"
      >
        <template slot-scope="{ row }">
          <span style="color: red">{{ row.reviewer }}</span>
        </template>
      </el-table-column>

     
      <el-table-column label="创建时间" class-name="status-col" width="100">
        <template slot-scope="{ row }">
          <span>
            {{ row.created_time}}
          </span>
        </template>
      </el-table-column>
      <el-table-column
        label="操作"
        align="center"
        width="230"
        class-name="small-padding fixed-width"
      >
        <template slot-scope="{ row }">
          <el-button type="primary" size="mini" @click="handleUpdate(row)">
            编辑
          </el-button>
          <el-button
            v-if="row.status != '1'"
            size="mini"
            type="success"
            @click="handleModifyStatus(row, 1)"
          >
           上架
          </el-button>
          <el-button
            v-if="row.status != '0'"
            size="mini"
            @click="handleModifyStatus(row, 0)"
          >
            下架
          </el-button>
          <el-popconfirm
           title="确定要删除该记录吗？"
           @onConfirm="delData(row)"
            >
           <el-button
            v-if="row.status != 'deleted'"
            size="mini"
            type="danger"
            slot="reference"
          >
           删除
          </el-button>
          </el-popconfirm>
          
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

    <el-dialog :title="textMap[dialogStatus]" :visible.sync="dialogFormVisible" fullscreen>
      <el-form
        ref="dataForm"
        :rules="rules"
        :model="temp"
        label-position="left"
        label-width="70px"
        style="width: 400px; margin-left: 50px"
      >
        <el-form-item label="标题" prop="title">
          <el-input v-model="temp.title" />
        </el-form-item>
        <el-form-item label="封面">
          <div class="editor-container">
          <dropzone v-model="temp.cover" style="width:200px;" id="myVueDropzone" url="https://httpbin.org/post" @dropzone-removedFile="dropzoneR" @dropzone-success="dropzoneS" />
          </div>
        </el-form-item>
        <el-form-item label="试看内容" prop="try">
          <tinymce v-model="temp.try" :width="700"/>
        </el-form-item>
        <el-form-item label="课程内容" prop="">
          <tinymce v-model="temp.content" :width="700"/>
        </el-form-item>
        <el-form-item label="课程价格">
          <el-input-number v-model="num"></el-input-number>
        </el-form-item>
        <el-form-item label="状态">
           <el-radio v-model="radio" label="1">下架</el-radio>
          <el-radio v-model="radio" label="2">上架</el-radio>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false"> 取消 </el-button>
        <el-button
          type="primary"
          @click="dialogStatus === 'create' ? createData() : updateData()"
        >
          确定
        </el-button>
      </div>
    </el-dialog>

    <el-dialog :visible.sync="dialogPvVisible" title="Reading statistics">
      <el-table
        :data="pvData"
        border
        fit
        highlight-current-row
        style="width: 100%"
      >
        <el-table-column prop="key" label="Channel" />
        <el-table-column prop="pv" label="Pv" />
      </el-table>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="dialogPvVisible = false"
          >Confirm</el-button
        >
      </span>
    </el-dialog>
  </div>
</template>

<script>
import { fetchList, createMedia, updateMedia, deleteMedia } from "@/api/media";
// 上传图片
import Dropzone from "@/components/Dropzone";
// 富文本编辑器
import Tinymce from "@/components/Tinymce";

import waves from "@/directive/waves"; // waves directive
import { parseTime } from "@/utils";
import Pagination from "@/components/Pagination"; // secondary package based on el-pagination
export default {
  name: "ComplexTable",
  components: { Pagination, Dropzone, Tinymce },
  directives: { waves },
  filters: {
    statusFilter(status) {
      const statusMap = {
        1: "success",
        0: "info",
        deleted: "danger",
      };
      return statusMap[status];
    },
    typeFilter(type) {
      return calendarTypeKeyValue[type];
    },
  },
  data() {
    return {
      tableKey: 0,
      visible: false,
      list: null,
      total: 0,
      // 富文本编辑器
      content: "",
      // 计数器
      num: 0,
      // 单选框
      radio: "1",
      listLoading: true,
      listQuery: {
        page: 1,
        limit: 20,
        importance: undefined,
        title: undefined,
        type: undefined,
        sort: "+id",
      },
      importanceOptions: ["已上架", "已下架"],
      sortOptions: [
        { label: "ID Ascending", key: "+id" },
        { label: "ID Descending", key: "-id" },
      ],
      statusOptions: ["1", "0", "deleted"],
      showReviewer: false,
      temp: {
        id:"",
        price:"",
        sub_count:"",
        created_time:"",
        title: "",
        cover: "http://dummyimage.com/200x100",
        try: "",
        content: "",
      },
      dialogFormVisible: false,
      dialogStatus: "",
      textMap: {
        update: "编辑",
        create: "新增",
      },
      dialogPvVisible: false,
      pvData: [],
      rules: {
        title: [{ required: true, message: "标题必须填写", trigger: "blur" }],
      },
      downloadLoading: false,
    };
  },
  created() {
    this.getList();
  },
  methods: {
    // 得到列表数据
    getList() {
      // console.log(fetchList);
      fetchList(this.listQuery).then((response) => {
        // console.log(response);
        this.list = response.data.items;
        this.total = response.data.total;
      });
    },
    // 新增图文
    handleCreate() {
      this.resetTemp();
      this.dialogStatus = "create";
      this.dialogFormVisible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].clearValidate();
      });
    },
    // 添加数据
    createData() {
       this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          this.temp.id = parseInt(Math.random() * 100) + 1024 // mock a id
          createMedia(this.temp).then(() => {
            // console.log(this.temp);
            this.list.unshift(this.temp)
            this.dialogFormVisible = false
            this.$notify({
              title: '提示',
              message: '新增数据成功',
              type: 'success',
              duration: 2000
            })
          })
        }
      })
    },
    // 编辑数据
    handleUpdate(row) {
      this.temp = Object.assign({}, row);
      console.log(this.temp);
       this.dialogStatus = 'update'
      this.dialogFormVisible = true
       this.$nextTick(() => {
        this.$refs['dataForm'].clearValidate()
      })
    },
    // 更新数据
    updateData() {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          const tempData = Object.assign({}, this.temp)
          tempData.timestamp = +new Date(tempData.timestamp) // change Thu Nov 30 2017 16:41:05 GMT+0800 (CST) to 1512031311464
          updateMedia(tempData).then(() => {
            console.log(tempData);
            const index = this.list.findIndex(v => v.id === this.temp.id)
            // console.log(index);
            this.list.splice(index, 1, this.temp)
            this.dialogFormVisible = false
            this.$notify({
              title: '提示',
              message: '编辑成功',
              type: 'success',
              duration: 2000
            })
          })
        }
      })
    },
    // 删除数据
    delData(row){
      // console.log(row);
      deleteMedia(this.temp).then(()=>{
const index = this.list.findIndex(v => v.id === this.temp.id)
       this.list.splice(index, 1)
            this.dialogFormVisible = false
            this.$notify({
              title: '提示',
              message: '编辑成功',
              type: 'success',
              duration: 2000
            })
      })
       
    },
    dropzoneS(file) {
      console.log(file);
      this.$message({ message: "Upload success", type: "success" });
    },
    dropzoneR(file) {
      console.log(file);
      this.$message({ message: "Delete success", type: "success" });
    },
    handleFilter() {
      this.listQuery.page = 1;
      this.getList();
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
      if (order === "ascending") {
        this.listQuery.sort = "+id";
      } else {
        this.listQuery.sort = "-id";
      }
      this.handleFilter();
    },
    resetTemp() {
      this.temp = {
        title: "",
        try: "",
        content: "",
      };
    },

    handleDelete(row, index) {
      this.$notify({
        title: "提示",
        message: "删除成功",
        type: "success",
        duration: 2000,
      });
      this.list.splice(index, 1);
    },
    handleFetchPv(pv) {
      fetchPv(pv).then((response) => {
        this.pvData = response.data.pvData;
        this.dialogPvVisible = true;
      });
    },
    handleDownload() {
      this.downloadLoading = true;
      import("@/vendor/Export2Excel").then((excel) => {
        const tHeader = ["timestamp", "title", "type", "importance", "status"];
        const filterVal = [
          "timestamp",
          "title",
          "type",
          "importance",
          "status",
        ];
        const data = this.formatJson(filterVal);
        excel.export_json_to_excel({
          header: tHeader,
          data,
          filename: "table-list",
        });
        this.downloadLoading = false;
      });
    },
    formatJson(filterVal) {
      return this.list.map((v) =>
        filterVal.map((j) => {
          if (j === "timestamp") {
            return parseTime(v[j]);
          } else {
            return v[j];
          }
        })
      );
    },
    getSortClass: function (key) {
      const sort = this.listQuery.sort;
      return sort === `+${key}` ? "ascending" : "descending";
    },
  },
};
</script>
