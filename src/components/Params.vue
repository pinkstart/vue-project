<template>
  <div>
    <h3>分类参数</h3>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>分类参数</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 提示区域 -->
      <el-alert show-icon title="注意！只允许为第三级分类设置动态参数和静态属性" type="warning" :closeable="false"></el-alert>
      <!-- 级联选择器 -->
      <el-row>
        <el-col>
          请选择分类：
          <!--
                        expand-trigger="hover" :设置级联选择器悬停触发 
                        v-model="selectedKeys" ：绑定级联选择器选中的数据
                     :options="options" ：绑定级联选择器显示的数据来源
                     :props="cascaderProps" ： 设置级联选择器中选项的配置信息 
                     @change="cascaderChange" ：当级联选择器中的选项发生改变时触发
                     change-on-select ：级联选择器中的任何一项都可以选择
                     clearable ： 可以按 X 删除选择的内容 -->
          <el-cascader expand-trigger="hover" v-model="selectedKeys" :options="cateList" :props="cascaderProps" @change="cascaderChange" clearable></el-cascader>
        </el-col>
      </el-row>

      <!-- 动态参数和静态属性的tab页签 -->
      <el-tabs v-model="activeName" @tab-click="handleClick">
        <el-tab-pane label="动态参数" name="many">
          <el-button type="primary" size="mini" :disabled="isBtnDisabled" @click="addDialogVisible = true">添加参数</el-button>

          <!-- 展示动态参数数据的表格 -->
          <el-table :data="manyTableData" border stripe>
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag v-for="(item,i) in scope.row.attr_vals" 
                :key="i" closable @close="tagClose(i,scope.row)">{{item}}</el-tag>

                <el-input ref="saveTagInput" class="input-new-tag" 
                v-if="scope.row.inputVisible"
                 v-model="scope.row.inputValue" 
                 size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)" 
                  @blur="handleInputConfirm(scope.row)">
                </el-input>
                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">添加参数</el-button>
              </template>
            </el-table-column>
            <el-table-column type="index"></el-table-column>
            <el-table-column prop="attr_name" label="参数名称">
            </el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button size="mini" icon="el-icon-edit" type="primary" @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
                <el-button size="mini" icon="el-icon-delete" type="danger" @click="delParams(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <el-tab-pane label="静态属性" name="only">
          <el-button type="primary" size="mini" :disabled="isBtnDisabled" @click="addDialogVisible = true">添加属性</el-button>

          <!-- 展示静态属性数据的表格 -->
          <el-table :data="onlyTableData" border stripe>
            <el-table-column type="expand"></el-table-column>
            <el-table-column type="index"></el-table-column>
            <el-table-column prop="attr_name" label="属性名称">
            </el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button size="mini" icon="el-icon-edit" type="primary" @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
                <el-button size="mini" icon="el-icon-delete" type="danger" @click="delParams(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>

    <!-- 添加参数对话框 -->
    <el-dialog :title="'添加' + titleText" :visible.sync="addDialogVisible" width="50%" @close="addDialogClosed">

      <!-- 弹窗主体 -->
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px">
        <el-form-item label="参数名称" prop="attr_name">
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParams">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 添加参数对话框 -->
    <el-dialog :title="'修改' + titleText" :visible.sync="editDialogVisible" width="50%" @close="editDialogClosed">

      <!-- 弹窗主体 -->
      <el-form :model="editForm" :rules="addFormRules" ref="editFormRef" label-width="100px">
        <el-form-item label="参数名称" prop="attr_name">
          <el-input v-model="editForm.attr_name"></el-input>
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editParams">确 定</el-button>
      </span>
    </el-dialog>

  </div>
</template>
    
<script>
export default {
  data() {
    return {
      //获取用户选择的级联选择器中的数据
      selectedKeys: [],
      //用来保存分类数据的数组
      cateList: [],
      //配置级联选择器中的内容要如何展示
      cascaderProps: {
        //使用label设置级联选择器显示的内容是什么
        label: 'cat_name',
        //使用value来设置级联选择器选中项到底是什么
        value: 'cat_id',
        //设置子节点属性
        children: 'children'
      },
      //activeName保存的是选中的tabPane项的name属性值
      activeName: 'many',
      //用来保存动态参数的数据
      manyTableData: [],
      //用来保存静态属性的数据
      onlyTableData: [],
      //保存需要添加的参数信息
      addForm: {
        attr_name: '',
        attr_sel: ''
      },
      //验证规则对象
      addFormRules: {
        attr_name: [{ required: true, message: '请输入分类名称', trigger: 'blur' }]
      },
      //控制添加参数弹出窗的显示和隐藏
      addDialogVisible: false,
      //控制修改参数弹出窗的显示和隐藏
      editDialogVisible: false,
      //修改参数的信息
      editForm: {},

      inputVisible: false,
      inputValue: ''
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    async getCateList() {
      const { data: res } = await this.$http.get('categories')
      //将数据保存到data中的cateList
      this.cateList = res.data
    },
    async cascaderChange() {
      this.getParamsData()
    },
    handleClick() {
      console.log(this.activeName)
      this.getParamsData()
    },
    async getParamsData() {
      //因为只能给三级分类去添加动态参数和静态属性。
      if (this.selectedKeys.length !== 3) {
        //如果用户选择的不是三级分类，那么selectedKeys数组里面就没有三个元素
        this.selectedKeys = []
        //如果用户选择的不是三级分类，清空表格数据
        this.manyTableData = []
        this.onlyTableData = []
        return
      }

      console.log(this.selectedKeys)
      //获取三级分类的id
      let cateId = this.selectedKeys[2]
      //发送请求获取三级分类对应的动态参数
      const { data: res } = await this.$http.get(`categories/${cateId}/attributes`, { params: { sel: this.activeName } })

      if (res.meta.status !== 200) {
        this.$message.error('获取参数信息失败')
      }

      //将参数信息字符串转换成数组形式
      res.data.forEach(item => {
        if (item.attr_vals.trim() === '') {
          item.attr_vals = []
        } else {
          item.attr_vals = item.attr_vals.split(' ')
        }

        //给每一个动态参数添加inputVisible的属性
        item.inputVisible = false
        item.inputValue = ''
      })

      //   console.log(res)
      //判断当前是动态参数还是静态属性，根据情况将数据挂载到不同的data中。
      if (this.activeName === 'many') {
        this.manyTableData = res.data
      } else {
        this.onlyTableData = res.data
      }

      console.log(res.data)
    },
    addDialogClosed() {
      //对话框关闭之后要将对话框中的表单重置
      this.$refs.addFormRef.resetFields()
    },
    addParams() {
      //对表单进行预校验
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return

        //如果校验成功，需要添加参数
        const { data: res } = await this.$http.post(`categories/${this.cateId}/attributes`, {
          attr_name: this.addForm.attr_name,
          attr_sel: this.activeName
        })

        if (res.meta.status !== 201) {
          return this.$message.error('添加参数失败')
        }

        this.$message.success('添加参数成功')
        this.getParamsData()

        this.addDialogVisible = false
      })
    },
    editDialogClosed() {
      //重置表单
      this.$refs.editFormRef.resetFields()
    },
    async showEditDialog(attr_id) {
      //根据id查找到需要编辑的参数信息，并将参数放到表单中展示
      const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes/${attr_id}`, { params: { attr_sel: this.activeName } })
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数信息失败')
      }
      this.editForm = res.data
      // alert(123)
      this.editDialogVisible = true
    },
    editParams() {
      //当用户点击确定按钮的时候，预校验数据
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return

        //校验成功
        //发送请求，完成编辑
        console.log(this.activeName)
        const { data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${this.editForm.attr_id}`, {
          attr_sel: this.activeName,
          attr_name: this.editForm.attr_name
        })
        if (res.meta.status !== 200) {
          return this.$message.error('更新参数信息失败')
        }
        this.$message.success('更新参数信息成功')
        this.getParamsData()
        // alert(123)
        this.editDialogVisible = false
      })
    },
    async delParams(attr_id) {
      //弹出确定取消窗询问用户是否删除
      const result = await this.$confirm('此操作将永久删除该参数, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)

      if (result === 'cancel') {
        return this.$message.info('取消了删除操作')
      }

      //发送请求完成删除
      const { data: res } = await this.$http.delete(`categories/${this.cateId}/attributes/${attr_id}`)

      if (res.meta.status !== 200) {
        return this.$message.error('删除参数信息失败')
      }

      this.$message.success('删除参数信息成功')
      this.getParamsData()

      this.editDialogVisible = false
    },
    showInput(row) {
      row.inputVisible = true
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    async handleInputConfirm(row) {
      if(row.inputValue.trim() === ""){
        row.inputValue = "";
        return;
      }

      //假如文本框中有正常的内容，
      //要将用户输入在文本框的内容添加到attr_vals数组中
      row.attr_vals.push( row.inputValue.trim() );
      row.inputValue = "";
      row.inputVisible = false
      
      this.saveAttrVals(row)
      
    },
    tagClose(index,row){
      //删除attr_vals数组中对应的元素
      row.attr_vals.splice(index,1);
      //将数组更新到数据库中
      this.saveAttrVals(row)
    },
    async saveAttrVals(row){
      //应该要发送请求，添加参数
      const {data:res} = await this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`,{
        attr_name: row.attr_name,
        attr_sel: row.attr_sel,
        attr_vals: row.attr_vals.join(" ")
      })

      if(res.meta.status !== 200){
        return this.$message.error("更新参数信息失败")
      }

      this.$message.success("更新成功");
    }
  },
  computed: {
    //分类id其实很常用，所以可以定义成计算属性
    cateId() {
      if (this.selectedKeys.length === 3) {
        //如果是三级分类就返回分类的id
        return this.selectedKeys[2]
      }
      //否则不是三级分类，返回null
      return null
    },

    //按钮是否可用
    isBtnDisabled() {
      if (this.selectedKeys.length !== 3) {
        return true
      }

      return false
    },
    titleText() {
      if (this.activeName === 'many') {
        return '动态参数'
      } else {
        return '静态属性'
      }
    }
  }
}
</script>

<style lang="less" scoped>
.el-col {
  padding: 15px;
}

.el-tag {
  margin: 10px;
}

.el-input {
  width: 130px;
}
</style>


