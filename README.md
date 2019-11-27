# page-memory
element ui分页多选，翻页记忆

html
```html
   <el-table
    ref="multipleTable"
    :data="gridData"
    border
    @selection-change="handleSelectionChange"
    header-cell-class-name="add_table_header project_table">
    <el-table-column
      type="selection"
      width="55"  align="center">
    </el-table-column>
    <el-table-column
      type="index"
      label="序号"
      width="50"
      align="center"
      :index="indexMethod">
    </el-table-column>
    <el-table-column
      prop="account"
      label="登录名">
    </el-table-column>
    <el-table-column
      prop="name"
      label="个人姓名/企业名称">
      <template slot-scope="scope">
        <div class="statusbigbox" v-if="scope.row.type === 1">
          <span class="statusbox">{{scope.row.name}}</span>
        </div>
        <div class="statusbigbox" v-if="scope.row.type === 2">
          <span class="statusbox">{{scope.row.enterpriseName}}</span>
        </div>
      </template>
    </el-table-column>
    <el-table-column
      prop="account"
      label="身份证号码/统一社会信用代码">
      <template slot-scope="scope">
        <div class="statusbigbox" v-if="scope.row.type === 1">
          <span class="statusbox">{{scope.row.identifyId}}</span>
        </div>
        <div class="statusbigbox" v-if="scope.row.type === 2">
          <span class="statusbox">{{scope.row.uniformSocialCreditCode}}</span>
        </div>
      </template>
    </el-table-column>
    <el-table-column
      label="操作" width="80" align="center">
      <template slot-scope="scope">
        <el-button type="text" size="small" class="handle_btn" @click="detailBtn(scope)">
          详情
        </el-button>
      </template>
    </el-table-column>
  </el-table>
  <!--分页-->
  <div class="page_box">
    <el-pagination
      class="pagebox"
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page.sync="currentPage"
      :total="total"
      :page-size='pageSize'
      layout="prev, pager, next, jumper">
    </el-pagination>
  </div>
  <!--分页-->
```
```js
gridData: [],
pageSize: 8,
pageNo: 0,
total: 0, // 总条数
currentPage: 1,
multipleSelectionAll: [], // 所有选中的数据包含跨页数据
multipleSelection: [], // 当前页选中的数据
idKey: 'objectId' // 标识列表数据中每一行的唯一键的名称
methods: {
   // 记忆选择核心方法
    changePageCoreRecordData () {
      // 标识当前行的唯一键的名称
      let idKey = this.idKey
      let that = this
      // 如果总记忆中还没有选择的数据，那么就直接取当前页选中的数据，不需要后面一系列计算
      if (this.multipleSelectionAll.length <= 0) {
        this.multipleSelectionAll = this.multipleSelection
        return
      }
      // 总选择里面的key集合
      let selectAllIds = []
      this.multipleSelectionAll.forEach(row => {
        selectAllIds.push(row[idKey])
      })
      let selectIds = []
      // 获取当前页选中的id
      this.multipleSelection.forEach(row => {
        selectIds.push(row[idKey])
        // 如果总选择里面不包含当前页选中的数据，那么就加入到总选择集合里
        if (selectAllIds.indexOf(row[idKey]) < 0) {
          that.multipleSelectionAll.push(row)
        }
      })
      let noSelectIds = []
      // 得到当前页没有选中的id
      this.gridData.forEach((row) => {
        if (selectIds.indexOf(row[idKey]) < 0) {
          noSelectIds.push(row[idKey])
        }
      })
      noSelectIds.forEach((id) => {
        if (selectAllIds.indexOf(id) >= 0) {
          for (let i = 0; i < that.multipleSelectionAll.length; i++) {
            if (that.multipleSelectionAll[i][idKey] === id) {
              // 如果总选择中有未被选中的，那么就删除这条
              that.multipleSelectionAll.splice(i, 1)
              break
            }
          }
        }
      })
    },
    // 选中
    handleSelectionChange (selection) {
      this.multipleSelection = selection
      // 实时记录选中的数据
      setTimeout(() => {
        this.changePageCoreRecordData()
      }, 50)
    },
    // 页面选中展示
    setSelectRow () {
      if (!this.multipleSelectionAll || this.multipleSelectionAll.length <= 0) {
        return
      }
      // 标识当前行的唯一键的名称
      let idKey = this.idKey
      let selectAllIds = []
      this.multipleSelectionAll.forEach((row) => {
        selectAllIds.push(row[idKey])
      })
      this.$refs.multipleTable.clearSelection()
      for (var i = 0; i < this.gridData.length; i++) {
        if (selectAllIds.indexOf(this.gridData[i][idKey]) >= 0) {
          // 设置选中，记住table组件需要使用ref="table"
          this.$refs.multipleTable.toggleRowSelection(this.gridData[i], true)
        }
      }
    },
    handleSizeChange (nowNum) {
      this.changePageCoreRecordData()
      this.pageNo = (nowNum - 1) * this.pageSize
      this.userList()
    },
    handleCurrentChange (nowNum) {
      this.changePageCoreRecordData()
      this.pageNo = (nowNum - 1) * this.pageSize
      this.userList()
    },
//请求接口部分
  getGoodsList(){
    let data = {};
    data['page'] = this.page.currentPage;
    data['pnum'] = this.page.pnum;
    this.$ajax({
     url:'goods/list',
     data:data
    }).then(val => {
     this.tableData = val.data.rows ? val.data.rows : [];
     this.page = {
     pnum:10,
     pageTotal:val.data.total,
     currentPage:val.data.page,
     };
     setTimeout(()=>{
         this.setSelectRow();
     }, 50)
    })
  }
}
```
