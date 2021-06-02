<template>
    <div>
        <el-row>
            <el-col :span="12">
                <div class="grid-content">
                    <span>剑网3安装目录：</span><span>{{ jx3path }}</span>
                </div>
            </el-col>
            <el-col :span="12">
                <div class="grid-content grid-right">
                    <el-button-group>
                        <el-button size="medium" v-on:click="selectFolder">手动选择</el-button>
                        <el-button size="medium" v-on:click="autoGet">自动获取</el-button>
                    </el-button-group>
                </div>
            </el-col>
        </el-row>
        <el-row>
            <el-col :span="24" class="margin-top">
                <span class="user-list-title">账号列表</span>
                <el-button class="grid-right" size="mini" round v-on:click="reload">刷新</el-button>
            </el-col>
        </el-row>
        <el-row>
            <el-col :span="24" class="divScroll margin-top">
                <el-collapse v-model="activeName" accordion>
                    <el-collapse-item  v-for="(item, index) in folderJson.children" :title="item.name" :name="index">
                        <template v-for="item2 in item.children">
                            <template v-for="item3 in item2.children">
                                <div class="character-list" v-for="item4 in item3.children">
                                    <span class="character-name">{{ item2.name + ' - ' + item3.name + ' - ' + item4.name}}</span>
                                    <el-button class="grid-right" size="mini" v-on:click="copyClick(item.name, item2.name, item3.name, item4.name)" round>{{ buttonText }}</el-button>
                                </div>
                            </template>
                        </template>
                        <el-button class="delete-button grid-right" size="mini" type="danger" v-on:click="deleteUser(item.name, item.path)" round>删除</el-button>
                    </el-collapse-item>
                </el-collapse>
            </el-col>
        </el-row>
    </div>
</template>

<script>
  const { dialog } = require('electron').remote
  const path = window.require('path')
  const fs = require('fs-extra')
  const { treeJson } = require('kigi')

  export default {
    name: 'Index',
    data () {
      return {
        jx3path: localStorage.getItem('jx3path') === null ? '' : localStorage.getItem('jx3path'),
        jx3userDataPath: localStorage.getItem('jx3path') === null ? '' : path.join(localStorage.getItem('jx3path'), 'Game', 'JX3', 'bin', 'zhcn_hd', 'userdata'),
        activeName: '',
        folderJson: [],
        copy: 0,
        buttonText: '复制',
        userPath1: '',
        userPath2: '',
        userName1: ''
      }
    },
    methods: {
      getFolderJson: async function (dir) {
        const options = {
          level: 5,
          onlyDir: true
        }
        this.folderJson = await treeJson(dir, options)
        this.$message({
          showClose: true,
          duration: 1000,
          message: '加载账号列表成功',
          type: 'success'
        })
      },
      autoGet: function () {
        alert('此功能未完成，请手动选择')
      },
      copyClick: function (user, server, server2, name) {
        if (this.buttonText === '粘贴') {
          this.userPath2 = path.join(this.jx3path, 'Game', 'JX3', 'bin', 'zhcn_hd', 'userdata', user, server, server2, name)
          this.$confirm('是否将&nbsp<span class="red-text">' + this.userName1 + '</span>&nbsp覆盖到&nbsp<span class="red-text">' + name + '</span>&nbsp?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning',
            dangerouslyUseHTMLString: true
          }).then(() => {
            fs.copy(this.userPath1, this.userPath2).then(() => {
              console.log('success!')
            }).catch(err => {
              console.error(err)
            })
            this.$message({
              type: 'success',
              message: '复制成功!'
            })
          }).catch(() => {
            this.$message({
              type: 'info',
              message: '已取消复制'
            })
          })
          this.buttonText = '复制'
        } else {
          this.buttonText = '粘贴'
          this.userName1 = name
          this.userPath1 = path.join(this.jx3path, 'Game', 'JX3', 'bin', 'zhcn_hd', 'userdata', user, server, server2, name)
        }
      },
      selectFolder: function () {
        dialog.showOpenDialog({
          properties: ['openFile', 'openDirectory']
        }).then(result => {
          if (result.canceled === false) {
            this.jx3path = result.filePaths[0]
            this.jx3userDataPath = path.join(this.jx3path, 'Game', 'JX3', 'bin', 'zhcn_hd', 'userdata')
            if (fs.existsSync(this.jx3userDataPath) === false) {
              alert(this.jx3userDataPath + '不存在，请重新选择')
              this.jx3path = localStorage.getItem('jx3path')
            } else {
              this.getFolderJson(this.jx3userDataPath)
              localStorage.setItem('jx3path', this.jx3path)
            }
          }
        }).catch(err => {
          console.log(err)
        })
      },
      reload: function () {
        if (fs.existsSync(this.jx3userDataPath) === true) {
          this.getFolderJson(this.jx3userDataPath)
        }
      },
      deleteUser: function (user, userPath) {
        this.$prompt('-' + user + '<br>' + '此操作将永久删除该帐号文件，是否继续？', '删除', {
          dangerouslyUseHTMLString: true,
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          inputPlaceholder: '请输入1防止误删',
          inputPattern: /1/,
          inputErrorMessage: '请输入1防止误删',
          type: 'warning'
        }).then(() => {
          fs.remove(userPath)
          this.$message({
            type: 'success',
            message: '删除成功'
          })
          this.reload()
          this.activeName = ''
        })
      }
    },
    mounted: function () {
      if (fs.existsSync(this.jx3userDataPath) === true) {
        this.getFolderJson(this.jx3userDataPath)
      }
    }
  }
</script>

<style lang="scss">
    .grid-content {
        min-height: 36px;
        line-height: 36px;
        font-size: 18px;
        font-family: "Helvetica Neue",Helvetica,"PingFang SC","Hiragino Sans GB","Microsoft YaHei","微软雅黑",Arial,sans-serif;
    }
    .grid-right {
        float: right;
    }
    .user-list-title {
        min-height: 36px;
        line-height: 36px;
        font-size: 18px;
        font-family: "Helvetica Neue",Helvetica,"PingFang SC","Hiragino Sans GB","Microsoft YaHei","微软雅黑",Arial,sans-serif;
    }
    .divScroll {
        overflow-y: scroll;
        max-height: 479px;
    }
    .divScroll::-webkit-scrollbar{
        display: none;
    }
    .character-name {
        line-height: 28px;
        font-size: 14px;
    }
    .character-list {
        margin-top: 3px;
        margin-bottom: 3px;
    }
    .delete-button {
        margin-bottom: 5px;
    }
    .red-text {
        color: red;
    }
    .margin-top {
        margin-top: 6px;
    }
</style>