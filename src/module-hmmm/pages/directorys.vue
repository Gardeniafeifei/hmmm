<template>
  <div class="dashboard-container">
    <div class="app-container">
      <el-card>
        <el-button 
          type="info" 
          plain 
          size="small"
          @click="showAddDialog()"
        >新建目录</el-button>
        <el-button type="info" plain size="small" :to="{ path: '/SubjectsList' }">返回学科</el-button>
        <el-form :inline="true" :model="formInline" class="demo-form-inline" style="margin-top: 10px">
          <el-form-item label="目录名称">
            <el-input v-model="formInline.user" placeholder="请输入"></el-input>
          </el-form-item>
          <el-form-item label="状态">
            <el-select v-model="formInline.region" placeholder="请选择">
              <el-option label="启用" value="true"></el-option>
              <el-option label="禁用" value="false"></el-option>
            </el-select>
            <el-button>取消</el-button>
            <el-button type="primary" @click="search()">搜索</el-button>
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
          :default-sort="{prop: 'date', order: 'descending'}"
        >
          <el-table-column align="center" label="序号">
            <template slot-scope="scope">
              <span>{{scope.row.id}}</span>
            </template>
          </el-table-column>
          <el-table-column align="center" label="目录名称">
            <template slot-scope="scope">
              <span>{{scope.row.directoryName}}</span>
            </template>
          </el-table-column>
          <el-table-column align="center" label="创建者">
            <template slot-scope="scope">
              <span>{{scope.row.creator}}</span>
            </template>
          </el-table-column>
          <el-table-column align="center" label="创建日期" sortable>
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
              <!-- aaa{{scope.row.state}} -->
              <el-button type="primary" size="mini" @click="showEditDialog(scope.row.id)">编辑</el-button>
              <el-button type="primary" size="mini" @click="handleStatus(scope.row)">
              
                <span v-if="scope.row.state==0">启用</span>
                <span v-else>禁用</span>
              </el-button>
              <el-button
                v-if="scope.row.status!='deleted'"
                size="mini"
                type="danger"
                @click="delDirectory(scope.row.id)"
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
      </el-card>
      <!-- 添加对话框 -->
      <el-dialog width="400px" title="新建目录" :visible.sync="addDialogFormVisible">
        <el-form ref="addForm" :model="addForm" :rules="addRules" label-width="80px">
          <el-form-item label="学科ID" prop="subjectID">
            <el-input v-model="addForm.subjectID"></el-input>
          </el-form-item>
          <el-form-item label="目录名称" prop="directoryName">
            <el-input v-model="addForm.directoryName"></el-input>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="addDialogFormVisible = false">取消</el-button>
          <el-button type="primary" @click="addSubmit()">确定</el-button>
        </div>
      </el-dialog>
      <!-- 编辑对话框 -->
      <el-dialog width="400px" title="目录修改" :visible.sync="editDialogFormVisible">
        <!-- {{editForm}} -->
        <el-form ref="editForm" :model="editForm" :rules="editRules" label-width="80px">
          <el-form-item label="学科ID" prop="subjectID">
            <el-input v-model="editForm.subjectID"></el-input>
          </el-form-item>
          <el-form-item label="目录名称" prop="directoryName">
            <el-input v-model="editForm.directoryName"></el-input>
          </el-form-item>
          <el-form-item label="数据id" prop="id">
            <el-input v-model="editForm.id"></el-input>
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
import { detail, list, update, add, remove } from '@/api/hmmm/directorys'
// import Dialog from './../components/directorys-add'
export default {
  name: 'DirectorysList',
  data() {
    return {
      addDialogFormVisible: false,
      addForm: {
        subjectID:'',
        directoryName: ''
      },
      editForm: {
        subjectID: '',
        directoryName: '',
        id: ''
      },
      addRules: {
        subjectID: [
          {required: true ,message: '学科ID必填', trigger: 'blur'}
        ],
        directoryName: [
          {required: true, message: '目录名称必填', trigger: 'blur'},
          {max: 25, message: '最多输入25个字', trigger: 'blur'}
        ]
      },
      editDialogFormVisible: false,
      editRules: {
        subjectID: [
          {required: true, message: '学科ID必填', trigger: 'blur'}
        ],
        directoryName: [
          {required: true, message: '目录名称必填', trigger: 'blur'},
          {max: 25, message: '最多输入25个字', trigger: 'blur'}
        ],
        id: [
          {required: true, message: '数据id必填', trigger: 'blur'}
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
      directorysList: [],
      titleInfo: {
        pageTitle: '新建目录', // 页面标题
        text: '' // 新增, 编辑文本
      },
      formData: {
        directoryName: ''
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
        // const data = await detail ({'id':id})
        console.log(data)
        this.editForm = data
      })
    },
    // 搜索操作
    search () {
      this.reqParams.pagenum = 1
      this.getList()
    },
    // 添加操作
    addSubmit () {
      // 整个表单验证
      this.$refs.addForm.validate(async valid => {
        if (valid) {
          // 提交添加请求
          const {status} = await add ()
          console.log(status)
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
    delDirectory (id) {
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
    // 启用 禁用
    handleStatus(val) {
      var status = ''
      if (val.state === true) {
        val.state = 0
        status = '禁用'
      } else {
        val.status = 1
        status = '启用'
      }
      var data = {
        id: val.id,
        state: val.state
      }
      this.$confirm('已成功' + status + ', 是否继续?', '提示', {
        type: 'warning'
      })
      .then(async () => {
        await disabled(data)
          .then(response => {
            this.message.success('已成功' + status + '!')
            this.getList(this.requestParameters)
          })
          .catch(response => {
            this.$message.error(status + '失败!')
          })
      })
      .catch(() => {
        this.$message.info('已取消操作!')
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
    }
  }
}
</script>

<style scoped>
</style>
