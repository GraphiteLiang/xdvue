<template>
  <div id="app1">
    <el-header style="height: 10vh;">Header</el-header>
    <el-container style="height: 90vh; border: 1px solid #eee">
      <el-aside width="400px">
        <div ref="forumPage">
          <el-menu :default-openeds="['1', '3']"
                   v-loading="this.loading"
                   style="width: 380px;">
            <el-menu-item v-for="item in threadItems"
                          :key="item.index"
                          class="ellipsis"
                          :title="item.text"
                          @click="handleMenuItemClick(item)">
              <span>{{ item.text }}</span>
            </el-menu-item>
            <el-pagination
                :pager-count="2"
                @current-change="getForumPageData"
                layout="prev, pager, next"
                :total="this.maxThreadCount">
            </el-pagination>
          </el-menu>
        </div>
      </el-aside>
      <el-main v-loading="this.mainLoading">
        <div ref="mainDataPage">
          <template v-for="(item, index) in replyItems">
            <span :key="item.index" v-html="item.text"/>
            <el-divider v-if="index !== replyItems.length - 1"
                        :key="item.index"
                        content-position="left">{{ item.rid }}</el-divider>
          </template>
          <el-pagination
              @current-change="getThreadPageData"
              layout="prev, pager, next"
              :page-size="20"
              :total="this.replyCount">
          </el-pagination>
        </div>
      </el-main>
    </el-container>
  </div>
</template>

<script>
import axios from "axios"
export default {
  data() {
    return {
      mainLoading: false,
      loading: false,
      maxThreadCount: 0,
      threadItems: [
      ],
      selectedItem: null,
      replyCount: 0,
      replyItems:[
      ]
    };
  },
  methods: {
    handleMenuItemClick(item) {
      this.loading = true;
      this.selectedItem = item
      this.replyItems.splice(0)

      axios.get('http://127.0.0.1:11451/api/thread?id=' + item.tid + '&page=1')
          .then(response => {
            let replys = response.data.Replies;
            this.replyCount = response.data.ReplyCount;
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
          }).finally( () => this.loading = false)
    },

    getForumPageData(page) {
      this.loading = true;
      axios.get('http://127.0.0.1:11451/api/showf?id=4&' + 'page=' + page)
          .then(response => {
            this.threadItems.splice(0)
            let forumThreads = response.data;
            let curMaxCount = page * forumThreads.length;
            if (this.maxThreadCount <= curMaxCount) {
              this.maxThreadCount = curMaxCount + 1
            }
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
            this.$refs.forumPage.scrollIntoView();
          })
          .catch(error => {
            // 获取数据失败后的操作
            console.log(error)
            this.$message('获取串数据失败');
          }).finally( () => this.loading = false)
    },

    getThreadPageData(page) {
      this.mainLoading = true;
      axios.get('http://127.0.0.1:11451/api/thread?id=' + this.selectedItem.tid + '&page=' + page)
          .then(response => {
            this.replyItems.splice(0)
            let replys = response.data.Replies;
            this.replyCount = response.data.ReplyCount;
            for(let i = 0; i < replys.length; i++) {
              this.replyItems.push(
                  {
                    rid : replys[i].id,
                    index : i + 1,
                    text : replys[i].content
                  }
              )
            }
            this.$refs.mainDataPage.scrollIntoView();
          })
          .catch(error => {
            // 获取数据失败后的操作
            console.log(error)
            this.$message('获取串数据失败');
          }).finally( () => this.mainLoading = false)
    },
  },
  created() {
    this.loading = true;
    axios.get('http://127.0.0.1:11451/api/showf?id=4&page=1')
        .then(response => {
          let forumThreads = response.data;
          this.maxThreadCount = forumThreads.length + 1
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
    this.loading = false;
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
body {
  margin: 0;
}
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
