<template>
  <div>
    用户组
    <div>
      <el-button type="primary" size="small" @click="add_group">添加用户组</el-button>
    </div>
    <div class="group-list" style="padding: 5px 0;">
      <el-table :data="this.$store.state.group.groups" style="width: 100%" border>
        <el-table-column label="用户组名称" prop="name" align="center"></el-table-column>
        <el-table-column label="用户组等级" prop="level" align="center"></el-table-column>
        <el-table-column align="center" label="用户组操作">
          <template slot-scope="scope">
            <el-button size="mini" type="primary" @click="editGroup(scope.$index)">编辑用户组</el-button>

            <el-popconfirm title="确定删除吗？" @onConfirm="deleteGroup(scope.row._id)">
              <el-button size="mini" type="danger" slot="reference">删除用户组</el-button>
            </el-popconfirm>

          </template>
        </el-table-column>
      </el-table>
    </div>所有权限列表:
    <el-table :data="this.$store.state.stack" style="width: 100%" :default-sort="{prop:'method'}" stripe border size="small">
      <el-table-column type="index" :index="indexValue" label="序号" width="150" align="center"></el-table-column>
      <el-table-column prop="method" label="Method" width="250" align="center"></el-table-column>
      <el-table-column prop="url" label="URL"></el-table-column>
    </el-table>

    <div class="div-bg" v-show="addShow || editShow"></div>

    <!-- 创建用户组 -->

    <div class="div-editor" v-show="addShow">
      <el-button @click="addShow=false" class="btn-x" size="small">
        <i class="el-icon-close"></i>
      </el-button>

      <el-form ref="form" :model="form" label-width="80px" :inline="true">
        <el-col :span="12">
          <el-form-item label="用户组名:">
            <el-input v-model="form.name"></el-input>
          </el-form-item>
        </el-col>
      </el-form>

      <el-select v-model="form.level" placeholder="用户等级">
        <el-option v-for="item in select_opt" :key="item.value" :label="item.name" :value="item.value"></el-option>
      </el-select>

      <div class="editor-table">
        <el-table ref="multipleTable" :data="editorStack" tooltip-effect="dark" style="width: 100%" stripe
          :default-sort="{prop:'method'}">
          <el-table-column type="selection" width="55"></el-table-column>
          <el-table-column prop="method" label="Method" align="center" width="150"></el-table-column>
          <el-table-column prop="url" label="URL"></el-table-column>
        </el-table>
      </div>
      <div style="float:right;margin:10px 0px">
        <el-button size="small" type="danger" @click="addShow=false">取消</el-button>
        <el-button size="small" type="primary" @click="createGroup">确定</el-button>
      </div>
    </div>

    <!-- 编辑用户组 -->
    <div class="div-editor1" v-show="editShow">
      <el-button @click="editShow=false" class="btn-x" size="small">
        <i class="el-icon-close"></i>
      </el-button>

      <el-form ref="form" :model="groupEditor" label-width="80px" :inline="true">
        <el-col :span="12">
          <el-form-item label="用户组名:">
            <el-input v-model="groupEditor.name" :disabled="true"></el-input>
          </el-form-item>
        </el-col>
      </el-form>

      <el-select v-model="groupEditor.level" placeholder="用户等级">
        <el-option v-for="item in select_opt" :key="item.value" :label="item.name" :value="item.value"></el-option>
      </el-select>

      <div class="editor-table">
        <el-table ref="multipleTable1" :data="stack" tooltip-effect="dark" style="width: 100%" stripe
          :default-sort="{prop:'method'}">
          <el-table-column type="selection" width="55"></el-table-column>
          <el-table-column prop="method" label="Method" align="center" width="150"></el-table-column>
          <el-table-column prop="url" label="URL"></el-table-column>
        </el-table>
      </div>
      <div style="float:right;margin:10px 0px">
        <el-button size="small" type="danger" @click="editShow=false">取消</el-button>
        <el-button size="small" type="primary" @click="updateGroup">确定</el-button>
      </div>
    </div>
  </div>
</template>
<script>
  export default {
    data() {
      return {
        stack: this.$store.state.stack,
        editShow: false,
        addShow: false,
        editorStack: [],
        groups: [],
        form: {
          name: "",
          level: 0,
          stack: []
        },
        select_opt: [
          { name: "Level-0", value: 0 },
          { name: "Level-1", value: 1 },
          { name: "Level-2", value: 2 },
          { name: "Level-3", value: 3 },
          { name: "Level-4", value: 4 },
          { name: "Level-5", value: 5 }
        ],
        groupEditor: {}
      };
    },
    created() {
      this.$store.dispatch('group/getGroups', this)
      this.$store.dispatch('getStack', this)
    },
    mounted() {
    },
    methods: {
      indexValue(index) {
        return index + 1;
      },
      add_group() {
        this.stack=this.$store.state.stack
        this.editorStack = this.stack
        this.$refs.multipleTable.clearSelection()
        this.addShow = true;
      },
      editGroup(index) {
        this.groupEditor = this.$store.state.group.groups[index]
        this.stack=this.$store.state.stack
        let val=this.stack.filter(item=>{
          return this.groupEditor.stack.some(item1=>{
            if(item1.method==item.method && item1.url == item.url){
              return true
            }
          })
        })
        this.$refs.multipleTable1.clearSelection()
        if (val) {
          val.forEach(row => {
            this.$refs.multipleTable1.toggleRowSelection(row)
          })
        }
        this.editShow = true
      },
      createGroup() {
        if (!this.form.name) {
          return this.$message("名称不能为空");
        }
        this.form.stack = this.$refs.multipleTable.selection
        this.$axios
          .post("/group/group", this.form)
          .then(res => {
            if (res.data.code === 200) {
              this.addShow = false;
              this.$message({
                message: "创建成功",
                type: "success"
              });
              this.$store.dispatch('group/getGroups',this)
            } else {
              this.$message({
                message: `创建失败\n ${res.msg}`,
                type: "warning"
              })
            }
          })
          .catch(err => {
            this.$message({
              message: err,
              type: "warning"
            });
          });
      },
      deleteGroup(id) {
        this.$store.dispatch('group/deleteGroup', {
          app: this,
          id
        })
      },
      updateGroup() {
        let data = this.groupEditor;
        data.stack = this.$refs.multipleTable1.selection
        this.$store.dispatch('group/updateGroup',{
          app:this,
          data
        })
        this.editShow=false
      },
    }
  };
</script>
<style scoped lang="scss">
  .div-bg {
    position: absolute;
    z-index: 999;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    opacity: 0.5;
    background: #888;
  }

  .div-editor,
  .div-editor1 {
    padding: 10px 20px;
    position: absolute;
    z-index: 1000;
    top: 50px;
    left: 50%;
    transform: translateX(-50%);
    background: #fff;
    border-radius: 3px;
    width: 600px;
    min-height: 500px;
    border: 1px solid rgba(0, 0, 0, 0.4);
    box-shadow: 5px 5px 10px rgba(0, 0, 0, 0.4);

    .btn-x {
      position: absolute;
      top: 5px;
      right: 5px;
      z-index: 1001;
    }
  }
</style>