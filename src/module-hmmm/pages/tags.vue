<template>
  <div class="dashboard-container">
    <div class="app-container">
      <el-card>
        <el-button 
          type="info" 
          plain 
          size="small"
          @click="showAddDialog()"
        >新建标签</el-button>
        <el-button type="info" plain size="small">返回学科</el-button>
        <el-form :inline="true" :model="formInline" class="demo-form-inline" style="margin-top: 10px">
          <el-form-item label="标签名称">
            <el-input v-model="formInline.user" placeholder="请输入"></el-input>
          </el-form-item>
        </el-form>
        <el-table
          :key="tableKey"
          :data="dataList"
          v-loading="listLoading"
          element-loading-text="给我一点时间"
          fit
          highlight-current-row
          style="width: 100%"
          border
        >
          <el-table-column align="center" label="序号">
            <template slot-scope="scope">
              <span>{{scope.row.id}}</span>
            </template>
          </el-table-column>
          <el-table-column align="center" label="标签名称">
            <template slot-scope="scope">
              <span>{{scope.row.tagName}}</span>
            </template>
          </el-table-column>
          <el-table-column align="center" label="创建者">
            <template slot-scope="scope">
              <span>{{scope.row.creator}}</span>
            </template>
          </el-table-column>
          <el-table-column align="center" label="创建日期">
            <template slot-scope="scope">
              <span>{{scope.row.addDate}}</span>
            </template>
          </el-table-column>
          <el-table-column align="center" label="面试题数量">
            <template slot-scope="scope">
              <span>{{scope.row.totals}}</span>
            </template>
          </el-table-column>
          <el-table-column align="center" label="状态">
            <template slot-scope="scope">
              <span v-if="scope.row.state==1">启用</span>
              <span v-else>禁用</span>
            </template>
          </el-table-column>
          <!-- 操作按钮 -->
          <el-table-column
            align="center"
            label="操作"
            width="280"
            class-name="small-padding fixed-width"
          >
            <template slot-scope="scope">
              <el-button type="primary" size="mini" @click="showEditDialog(scope.row.id)">编辑</el-button>
              <el-button type="primary" size="mini" @click="handleStatus(scope.row)">
                <span v-if="scope.row.state==0">启用</span>
                <span v-else>禁用</span>
              </el-button>
              <el-button
                v-if="scope.row.status!='deleted'"
                size="mini"
                type="danger"
                @click="delTag(scope.row.id)"
              >删除</el-button>
            </template>
          </el-table-column>
        </el-table>
        <!-- end -->
        <!-- 分页 -->
        <div class="pagination" style="margin-top: 10px">
          <div class="pages">
            <el-pagination
              background
              @size-change="handleSizeChange"
              @current-change="handleCurrentChange"
              :current-page="Number(requestParameters.page)"
              :total="Number(total)"
              :page-size="Number(requestParameters.pagesize)"
              :page-sizes="[10,20,30, 50]"
              layout="sizes, prev, pager, next, jumper"
            ></el-pagination>
          </div>
        </div>
        <!-- end -->
        <!-- 新增标签弹层 -->
        <Dialog ref="edittag" :titleInfo="titleInfo" :formBase="formData" v-on:newDataes="getList"></Dialog>
      </el-card>
      <el-dialog width="400px" title="新建标签" :visible.sync="addDialogFormVisible">
        <el-form ref="addForm" :model="addForm" :rules="addRules" label-width="80px">
          <el-form-item label="新建标签" prop="tagName">
            <el-input v-model="addForm.tagName"></el-input>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="addDialogFormVisible = false">取消</el-button>
          <el-button type="primary" @click="addSubmit()">确定</el-button>
        </div>
      </el-dialog>
      <!-- 编辑对话框 -->
      <el-dialog width="400px" title="标签修改" :visible.sync="editDialogFormVisible">
        <el-form ref="editForm" :model="editForm" :rules="editRules" label-width="80px">
          <el-form-item label="标签名称" prop="tagName">
            <el-input v-model="editForm.tagName"></el-input>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="editDialogFormVisible = false">取消</el-button>
          <el-button type="primary" @click="editSubmit()">确定</el-button>
        </div>
      </el-dialog>
    </div>
  </div>
</template>

<script>
import { status } from '@/api/hmmm/constants'
import { list, detail, add, remove, update } from '@/api/hmmm/tags'
// import Dialog from './../components/tags-add'
export default {
  name: 'tagsList',
  data() {
    return {
      addDialogFormVisible: false,
      addForm: {
        tagName: ''
      },
      addRules: {
        tagName: [
          {required: true, message: '目录名称必填', trigger: 'blur'}
        ]
      },
      editForm: {
        subjectID: '',
        tagName: '',
        id: ''
      },
      editDialogFormVisible: false,
      editRules: {
        tagName: [
          {required: true, message: '目录名称必填', trigger: 'blur'}
        ]
      },
      tableKey: 0,
      dataList: [],
      total: null,
      listLoading: true,
      alertText: '',
      requestParameters: {
        page: 1,
        pagesize: 10
      },
      formInline: {
        user: '',
        region: ''
      },
      reqParams: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      tagsList: [],
      titleInfo: {
        pageTitle: '新建目录', // 页面标题
        text: '' // 新增, 编辑文本
      },
      formData: {
        tagName: ''
      }
    }
  },
  mounted () {
    this.initialDate()
  },
  methods: {
    // 显示添加目录的对话框
    showAddDialog () {
      this.addDialogFormVisible = true
      this.$nextTick(() => {
        this.$refs.addForm.resetFields()
      })
    },
    // 显示编辑目录的对话框
    showEditDialog (id) {
      this.editDialogFormVisible = true
      this.editForm.id = id
      console.log(id)
      this.$nextTick(async() => {
        console.log(id)
        this.$refs.editForm.resetFields()
        const {data} = await detail ({'id':id})
        console.log(data)
        this.editForm = data
      })
    },
    // 添加操作
    addSubmit () {
      // 整个表单验证
      this.$refs.addForm.validate(async valid => {
        if (valid) {
          // 提交添加请求
          const data = await add ()
          console.log(data)
          // 关闭对话框  更新列表数据
          this.addDialogFormVisible = false
          this.getList()
        }
      })
    },
    // 编辑操作
    editSubmit (id) {
      this.$refs.editForm.validate(async valid => {
        if (valid) {
          const data = await update ({'id': id})
          console.log(data)
          this.editDialogFormVisible = false
          this.getList() 
        }
      })
    },
    // 删除操作
    delTag (id) {
      this.$confirm('是否删除该目录信息', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async () => {
        const data = await remove ({'id': id})
        console.log(data)
        this.getList()
      })
    },
    // 初始数据
    initialDate() {
      // 读取数据
      this.getList()
    },
    // 获取列表数据
    async getList(params) {
      this.listLoading = true
      const { data: res } = await list(this.requestParameters)
      // debugger
      // 获取数据给 dataList
      this.dataList = res.items
      // 获取总条数
      this.total = res.counts
      this.alertText = `共 ${this.total} 条数据`
      this.listLoading = false
    },
    // 重置
    restForm() {
      this.$refs['requestParameters'].resetFields()
    },
    // 搜索信息
    handleFilter() {
      this.requestParameters.page = 1
      this.getList(this.requestParameters)
    },
    // 每页显示信息条数
    handleSizeChange(val) {
      this.requestParameters.pagesize = val
      if (this.requestParameters.page === 1) {
        this.getList(this.requestParameters)
      }
    },
    // 进入某一页
    handleCurrentChange(val) {
      this.requestParameters.page = val
      this.getList()
    },

    // 搜索项目后表单清空
    query() {
      this.formData = {}
    },
  }
}
</script>

<style scoped>
</style>
