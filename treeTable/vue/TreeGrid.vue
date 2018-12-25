<template>
  <el-table
    :data="data"
    border
    style="width: 100%"
    :row-style="showTr">

    <el-table-column label="操作" v-if="treeType === 'normal'" min-width="60">
      <template slot-scope="scope">
        <!-- <el-button :disabled ="!!scope.row.children"
          size="small"
          type="blue"
          @click="set(scope.row)">
          {{scope.row.isDefault==1?'首页':'设为首页'}}
        </el-button> -->
        <el-button type="text" size="small"
        @click="edit(scope.row)" v-has="'/Categorys/edit'">【编辑】</el-button>
        <el-button type="text" size="small" class="txt_col_hong" 
        @click.stop="handleDelete(scope.row)" v-has="'/Categorys/delete'">【删除】</el-button>
      </template>
    </el-table-column>
    <!-- <el-table-column label="序号">
      <template slot-scope="scope">
        {{scope.$index+1}}
      </template>
    </el-table-column> -->
    <el-table-column v-for="(column, index) in columns" :key="column.dataIndex"
      :label="column.text" :min-width="index==0?100:60">
      <template slot-scope="scope">
        <span v-if="spaceIconShow(index)" v-for="(space, levelIndex) in scope.row._level" :key='levelIndex' class="ms-tree-space"></span>
        <div class="button is-outlined is-primary is-small" v-if="toggleIconShow(index,scope.row)" @click="toggle(scope.$index)">
          <i v-if="!scope.row._expanded" class="el-icon-caret-right" aria-hidden="true"></i>
          <i v-if="scope.row._expanded" class="el-icon-caret-bottom" aria-hidden="true"></i>
        </div>
        <span v-else-if="index===0" class="ms-tree-space"></span>
        <!-- <b v-if="index===0 " :class="scope.row.icon"></b> -->
        {{scope.row[column.dataIndex]}}
      </template>
    </el-table-column>
    <el-table-column label="新窗口打开" min-width="50">
      <template slot-scope="scope">
        <!-- elementUI有bug，改用原生 -->
        <!-- <el-checkbox :checked="scope.row.is_blank=='1'?true:false" @change="changeChecked(scope.row)"
        ></el-checkbox> -->
        <input type="checkbox" :checked="scope.row.is_blank=='1'?true:false" @change="changeChecked(scope.row)">
      </template>
    </el-table-column>
    <el-table-column label="排序" min-width="50">
      <template slot-scope="scope">
        <el-input 
        size='mini' 
        :value="scope.row.sort_value"
        @change='changeInput'
        @focus="focusInput(scope.row)"
        ></el-input>
      </template>
    </el-table-column>

  </el-table>
</template>
<script>
  import Utils from '../utils/index.js'
//  import Vue from 'vue'
  export default {
    name: 'tree-grid',
    props: {
      treeStructure: {
        type: Boolean,
        default: function () {
          return false
        }
      },
      // 这是相应的字段展示
      columns: {
        type: Array,
        default: function () {
          return []
        }
      },
      // 这是数据源
      dataSource: {
        type: Array,
        default: function () {
          return []
        }
      },
      // 这个作用是根据自己需求来的，比如在操作中涉及相关按钮编辑，删除等，需要向服务端发送请求，则可以把url传过来
      requestUrl: {
        type: String,
        default: function () {
          return ''
        }
      },
      // 这个是是否展示操作列
      treeType: {
        type: String,
        default: function () {
          return 'normal'
        }
      },
      // 是否默认展开所有树
      defaultExpandAll: {
        type: Boolean,
        default: function () {
          return false
        }
      }
    },
    data () {
      return {
        editUSER:false,
        row:{}
      }
    },
    computed: {
    // 格式化数据源
      data: function () {
        let me = this
        if (me.treeStructure) {
          let data = Utils.MSDataTransfer.treeToArray(me.dataSource, null, null, me.defaultExpandAll)
          return data
        }
        return me.dataSource
      }
    },
    methods: {
      
    // 显示行
      showTr: function (row, index) {
        // console.log(row.row)
        let show = (row.row._parent ? (row.row._parent._expanded && row.row._parent._show) : true)
        row.row._show = show
        return show ? '' : 'display:none;'
      },
    // 展开下级
      toggle: function (trIndex) {
        let me = this
        let record = me.data[trIndex]
        record._expanded = !record._expanded
      },
    // 显示层级关系的空格和图标
      spaceIconShow (index) {
        let me = this
        if (me.treeStructure && index === 0) {
          return true
        }
        return false
      },
    // 点击展开和关闭的时候，图标的切换
      toggleIconShow (index, record) {
        let me = this
        if (me.treeStructure && index === 0 && record.children && record.children.length > 0) {
          return true
        }
        return false
      },
      edit: function(index) {
       this.editUSER = true;
       let data = {
            a:this.editUSER,
            b:index
        };
       this.$emit('editP',data);
      },
      handleDelete (index) {
        /*请大哥别改这一块，这里的index并不是索引，是传过来的一条数据*/
        this.$confirm('此操作将永久删除该记录, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          // this.dataSource.splice(index, 1)
          this.$emit('delP',index);
          this.$message({
            type: 'success',
            message: '删除成功!'
          })
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          })
        })
      },
      // 新窗口打开
      changeChecked(row){
        this.$emit('blank',row);
      },
      // 排序
      changeInput(value) {
        this.row.sort_value = value
        // 这里changeInput事件只有在值改变时触发，所以此处不用判断是否请求接口
        this.$emit('sort',this.data);
      },
      // 过度排序，用于修改当前项
      focusInput(row) {
        console.log(row)
        this.row = row
      }
    }
  }
</script>
<style scoped lang="less">
  .ms-tree-space{position: relative;
    top: 1px;
    display: inline-block;
    font-family: 'Glyphicons Halflings';
    font-style: normal;
    font-weight: 400;
    line-height: 1;
    width: 18px;
    height: 14px;}
  .ms-tree-space::before{content: ""}
</style>
