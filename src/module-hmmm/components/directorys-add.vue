<template>
  <div class="add-form">
    <el-dialog :title="titleInfo.text+titleInfo.pageTitle" :visible.sync="dialogFormVisible">
      <el-form
        :rules="ruleInline"
        ref="dataform"
        :model="formBase"
        label-position="left"
        label-width="150px"
        style="width: 80%; margin-left: 10px;"
      >
        <el-form-item label="目录名称" prop="directoryName">
          <el-input v-model="formBase.directoryName"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormH">{{$t('table.cancel')}}</el-button>
        <el-button type="primay" @click="createDate">{{$t('table.confirm')}}</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: 'DirectorysAdd',
  data() {
    return {}
  }
}
</script>

<style scoped>
</style>
