<template>
  <div id="app">
    <el-container style="height: 500px; border: 1px solid #eee">
      <el-aside width="210px">
        <el-menu :default-openeds="['1', '3']" style="width: 200px;">
          <el-menu-item v-for="item in threadItems"
                        :key="item.index"
                        class="ellipsis"
                        :title="item.text"
                        @click="handleMenuItemClick(item)">
            <span>{{ item.text }}</span>
          </el-menu-item>
        </el-menu>
      </el-aside>
      <el-main>
        <template v-for="(item, index) in replyItems">
        <span :key="item.index" v-html="item.text"/>
        <el-divider v-if="index !== replyItems.length - 1"
                    :key="item.index"
                    content-position="left">{{ item.rid }}</el-divider>
        </template>
      </el-main>
    </el-container>
  </div>
</template>

<script>
import axios from "axios"
export default {
  data() {
    return {
      threadItems: [
      ],
      selectedItem: null,
      replyItems:[
      ]
    };
  },
  methods: {
    handleMenuItemClick(item) {
      this.selectedItem = item
      this.replyItems.splice(0)

      axios.get('http://127.0.0.1:11451/api/thread?id=' + item.tid + '&page=1')
          .then(response => {
            let replys = response.data.Replies;
            console.log(replys.length)
            // 从api获取数据
            this.replyItems.push(
                {
                  rid : item.tid,
                  index : 0,
                  text : item.text,
                }
            )
            for(let i = 0; i < replys.length; i++) {
              this.replyItems.push(
                  {
                    rid : replys[i].id,
                    index : i + 1,
                    text : replys[i].content
                  }
              )
            }
          })
          .catch(error => {
            // 获取数据失败后的操作
            console.log(error)
            this.$message('获取串数据失败');
          })

    }
  },
  created() {
    axios.get('http://127.0.0.1:11451/api/showf?id=4&page=1')
        .then(response => {
          let forumThreads = response.data;
          console.log(forumThreads.length)
          // 从api获取数据
          for(let i = 0; i < forumThreads.length; i++) {
            this.threadItems.push(
                {
                  tid : forumThreads[i].id,
                  index : i,
                  text : forumThreads[i].content
                }
            )
          }
        })
        .catch(error => {
          // 获取数据失败后的操作
          console.log(error)
          this.$message('获取串数据失败');
        })
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.el-header {
  background-color: #B3C0D1;
  color: #333;
  line-height: 60px;
}

.el-aside {
  color: #333;
}

.ellipsis {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

</style>
