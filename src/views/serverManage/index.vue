<template>
  <div class="tree">
    <div class="t-right">
        <table-model 
        @handleSelectionChange="handleSelectionChange" 
        @clickAdd="clickAdd"
        @clickEdit="clickEdit"
        @clickDel="clickDel"
        :butttonList="butttonList"
        :tableData="tableData" 
        :tableThead="tableThead"></table-model>
    </div>
    <!-- dialog -->
    <el-dialog title="新增" 
    :visible.sync="dialogFormVisible" width="500px">
      <form-model :form="form" :tableThead="tableThead"></form-model>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" v-if="isSummit === true" @click="clickSummit()">确 定</el-button>
        <el-button type="primary" v-else @click="eidt()">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import Api from '@/api/index'
import formModel from '@/components/formmodel'
import tableModel from '@/components/tablemodel'
export default {
  name: 'functionManage',
  components: {
    formModel,
    tableModel
  },
  data() {
    return {
      tableData: [],
      butttonList: ['clickAdd', 'clickEdit', 'clickDel'],
      tableThead: [
        {label: 'dbServiceId', prop: 'dbServiceId', hidden: true},
        {label: '服务器名称', prop: 'dbServiceName', tagType: 'input', type:"text"},
        {label: 'IP', prop: 'domain', tagType: 'input', type:"text"},
        {label: '账号', prop: 'account', tagType: 'input', type:"text"},
        {label: '密码', prop: 'password', tagType: 'input', type:"text", hidden: true},
        {label: '排序', prop: 'sort', tagType: 'input', type:"number"},        
        {label: '是否禁用', prop: 'isDisable', tagType: 'radio', radioInfo: [{radioText: '是', radioLabel: true}, {radioText: '否', radioLabel: false}]},
      ],
      form: {},
      dialogFormVisible: false,
      formLabelWidth: '100px',
      selectedRowInfo: '', // 选中行信息
      isSummit: true // 是否是添加菜单
    }
  },
  methods: {
    // 重置表单
    resetFields() {
      const obj = {}
      for (const item of this.tableThead) {
        if(!item.hidden) {
          if (item.tagType === 'radio') {
            obj[item.prop] = true
          } else {
            obj[item.prop] = ''
          }
        }
      }
      this.form = {...this.form, ...obj}
    },
    // 获取--获取角色列表
    async getList() {
      const {data, success, message} = await Api.getSerList()
      if (success) {
        this.tableData = data
      } else {
        this.$message({
          message,
          type: 'error',
          duration: 1000,
          isDisable: true,
          showClose: true
        })
      }
    },
    // 点击--新增
    clickAdd() {
      this.dialogFormVisible = true
      this.isSummit = true
      this.resetFields()
    },
    // 点击--编辑
    async clickEdit() {
      if (!this.selectedRowInfo || !this.selectedRowInfo.length) return this.$alert('请选中需要编辑的行', '提示', {confirmButtonText: '确定'})
      if (this.selectedRowInfo.length > 1) return this.$alert('只能选中一行进行编辑', '提示', {confirmButtonText: '确定'})
      this.dialogFormVisible = true
      this.isSummit = false
      await Api.getSerInfo(this.selectedRowInfo[0].dbServiceId)
      this.form = this.selectedRowInfo[0]
    },
    // 点击--确定编辑
    async eidt() {
      const { success } = await  Api.editSerButton(this.selectedRowInfo[0].dbServiceId, this.form)
      if (success) {
        this.dialogFormVisible = false
        this.$message.success('编辑成功')
        this.getList()
      }
    },
    // 点击--删除
    async clickDel() {
      if (!this.selectedRowInfo || !this.selectedRowInfo.length) return this.$alert('请选中需要删除的行', '提示', {confirmButtonText: '确定'})
      if (this.selectedRowInfo.length > 1) return this.$alert('只能选中一行进行删除', '提示', {confirmButtonText: '确定'})
      this.$confirm('是否删除?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async () => {
        const { dbServiceId } = this.selectedRowInfo[0]
        const { success, message } = await  Api.delSerButton(dbServiceId)
        if (success) {
          this.$message.success('删除成功')
          this.tableData.splice(this.tableData.findIndex(item => item.dbServiceId === dbServiceId), 1)
        } else {
          this.$message({
            message,
            type: 'error',
            duration: 1000,
            showClose: true
          })
        }
      })
    },
    // 点击--选中
    handleSelectionChange(val) {
      this.selectedRowInfo = val
    },
    // 点击--新增提交表达
    async clickSummit() {
      const { success, message } = await Api.AddSerButton(this.form)
      if (success) {
        this.$message.success('添加成功')
        this.getList()
      } else {
        this.$message({
          message,
          type: 'error',
          duration: 1000,
          showClose: true
        })
      }
      this.dialogFormVisible = false
    }
  },
  created() {
    this.resetFields()
    this.getList()
  }
}
</script>

<style lang="stylus" scoped>
.dialog
  display flex
  .t-left
    width 200px
    height 300px
    border 1px solid #ddd
    margin-right 20px
  .t-right
    width 400px
    height 300px
    border 1px solid #ddd
  .tr-btn
    margin-bottom 20px
    display flex
  .table .el-radio+.el-radio
    margin-left 0 !important
.btn-list
  display flex
  flex-wrap wrap
  .el-button
    margin-left 10px
    margin-top 10px
.checkColor
  border 1px solid #fff
  color rgba(0, 0,0, 0)
  border-radius 100%
  box-sizing border-box
.selectTree
  display flex
  /deep/ .el-form-item__label
    width 100px
</style>