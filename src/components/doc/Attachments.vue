<template>
  <div class="attachments">
    <!-- 面包导航 -->
    <el-breadcrumb separator="/" style="padding-left:10px;padding-bottom:10px;font-size:12px;">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>系统管理</el-breadcrumb-item>
      <el-breadcrumb-item>附件管理</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 图片列表部分 -->
    <el-card>
      <!-- 搜索部分 -->
      <el-form :inline="true" :model="queryMap" class="demo-form-inline">
        <el-form-item label="图片路径">
          <el-input clearable @clear="search" v-model="queryMap.path" placeholder="输入图片路径查询"></el-input>
        </el-form-item>
        <el-form-item label="图片后缀">
          <el-select clearable @clear="search" v-model="queryMap.suffix" placeholder="请选择">
            <el-option label="jpg/JPG" value=".jpg"></el-option>
            <el-option label="png/PNG" value=".png"></el-option>
            <el-option label="gif/GIF" value=".gif"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" icon="el-icon-search" @click="search">查询</el-button>
          <el-button type="primary">上传<i class="el-icon-upload el-icon--right"></i></el-button>
        </el-form-item>
      </el-form>
      <!-- 图片展示部分 -->
      <el-row :gutter="20">
        <el-col style="margin-top:10px;" v-for="image in this.list" :key="image.id" :span="6">
          <div class="grid-content bg-purple">
            <el-image
              :alt="image.path"
              :fit="fits"
              :preview-src-list="srcList"
              style="width:200px;height:170px"
              :src="'http://www.zykhome.club/'+image.path"
            >
              <div slot="error" class="image-slot">
                <i class="el-icon-picture-outline"></i>
              </div>
            </el-image>
            <div>
              <el-tag
                size="mini"
                effect="dark"
                type="success"
                style="margin-left:50px;"
              >{{image.width}}px X {{image.height}}px</el-tag>
              <el-popconfirm title="这是一段内容确定删除吗？" @onConfirm="del(image.id)">
                <el-button
                  style="margin-left:30px;"
                  icon="el-icon-delete"
                  size="mini"
                  type="text"
                   slot="reference"
                >删除</el-button>
              </el-popconfirm>
            </div>
          </div>
        </el-col>
      </el-row>

      <!-- 分页 -->
      <el-pagination
        style="margin-top:30px;"
        background
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryMap.pageNum"
        :page-sizes="[8, 20, 30, 40]"
        :page-size="queryMap.pageSize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>
    </el-card>
  </div>
</template>


<script>
export default {
  data() {
    return {
      total: 0,
      fits: "contain",
      queryMap: {},
      list: [],
      srcList: []
    };
  },
  methods: {
    /**
     * 删除图片
     */
    async del(id){
     const { data: res } = await this.$http.delete(
          "upload/delete/" + id
        );
        if (res.code == 200) {
          this.$message.success("删除图片成功");
          this.getImgeList();
        } else {
          this.$message.error(res.msg);
        }
    },
    /**
     * 加载物资列表
     */
    async getImgeList() {
      const { data: res } = await this.$http.get("upload/findImageList", {
        params: this.queryMap
      });
      if (res.code !== 200) {
        return this.$message.error("获取附件列表失败");
      } else {
        var $this = this;
        this.total = res.data.total;
        this.list = res.data.list;
        this.srcList = [];
        this.list.forEach(function(item) {
          $this.srcList.push("http://www.zykhome.club/" + item.path);
        });
      }
    },
    //改变页码
    handleSizeChange(newSize) {
      this.queryMap.pageSize = newSize;
      this.getImgeList();
    },
    //翻页
    handleCurrentChange(current) {
      this.queryMap.pageNum = current;
      this.getImgeList();
    },
    /**
     * 搜索
     */
    search() {
      this.queryMap.pageNum = 1;
      this.getImgeList();
    }
  },
  created() {
    this.getImgeList();
  }
};
</script>