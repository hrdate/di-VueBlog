<template>
  <div>
    <div class="comment-title"><i class="iconfont iconpinglunzu" />评论</div>
    <!-- 评论框 -->
    <div class="comment-input-wrapper">
      <div style="display:flex">
        <v-avatar size="40">
          <img
            v-if="this.$store.state.avatar"
            :src="this.$store.state.avatar"
          />
          <img
            v-else
            src="https://gravatar.loli.net/avatar/d41d8cd98f00b204e9800998ecf8427e?d=mp&v=1.4.14"
          />
        </v-avatar>
        <div style="width:100%" class="ml-3">
          <div class="comment-input">
            <textarea
              class="comment-textarea"
              v-model="content"
              placeholder="留下点什么吧..."
              auto-grow
              dense
            />
          </div>
          <!-- 操作按钮 -->
          <div class="emoji-container">
            <span
              :class="chooseEmoji ? 'emoji-btn-active' : 'emoji-btn'"
              @click="chooseEmoji = !chooseEmoji"
            >
              <i class="iconfont iconbiaoqing" />
            </span>
            <button
              @click="insertComment"
              class="upload-btn v-comment-btn"
              style="margin-left:auto"
            >
              提交
            </button>
          </div>
          <!-- 表情框 -->
          <emoji @addEmoji="addEmoji" :chooseEmoji="chooseEmoji" />
        </div>
      </div>
    </div>
    <!-- 评论详情 -->
    <div v-if="count > 0 && reFresh">
      <!-- 评论数量 -->
      <div class="count">{{ count }} 评论</div>
      <!-- 评论列表 -->
      <div
        style="display:flex"
        class="pt-5"
        v-for="(item, index) of commentList"
        :key="item.id"
      >
        <!-- 头像 -->
        <v-avatar size="40" class="comment-avatar">
          <img v-if="item.avatar" :src="item.avatar" />
          <img v-else src="https://static.talkxj.com/photos/0bca52afdb2b9998132355d716390c9f.png">
        </v-avatar>
        <div class="comment-meta">
          <!-- 用户名 -->
          <div class="comment-user">
            <span v-if="!item.webSite">{{ item.userId }} {{ item.userName }}</span>
            <a v-else :href="item.webSite" target="_blank">
              {{ item.userId }}{{ item.userName }}
            </a>
            <span class="blogger-tag" v-if="item.userId == 1">博主</span>
          </div>
          <!-- 信息 -->
          <div class="comment-info">
            <!-- 楼层 -->
            <span style="margin-right:10px">{{ count - index }}楼</span>
            <!-- 发表时间 -->
            <span style="margin-right:10px">{{ item.created | date }}</span>
            <!-- 点赞 -->
            <!-- <span
              :class="isLike(item.id) + ' iconfont icondianzan'"
              @click="like(item)"
            />
            <span v-show="item.likeCount > 0"> {{ item.likeCount }}</span> -->
            <!-- 回复 -->
            <span class="reply-btn" @click="replyComment(index, item)">
              回复
            </span>
          </div>
          <!-- 评论内容 -->
          <p v-html="item.content" class="comment-content"></p>
          <!-- 回复人 -->
          <div
            style="display:flex"
            :v-model="item.replyDTOList"
            v-for="reply of item.replyDTOList"
            :key="reply.id"
          >
            <!-- 头像 -->
            <v-avatar size="36" class="comment-avatar">
              <img v-if="reply.avatar" :src="reply.avatar" />
              <img v-else src="https://static.talkxj.com/photos/0bca52afdb2b9998132355d716390c9f.png">
            </v-avatar>
            <div class="reply-meta">
              <!-- 用户名 -->
              <div class="comment-user">
                <span v-if="!reply.webSite">{{ reply.userName }}</span>
                <a v-else :href="reply.webSite" target="_blank">
                  {{ reply.userName }}
                </a>
                <span class="blogger-tag" v-if="reply.userId == 1">博主</span>
              </div>
              <!-- 信息 -->
              <div class="comment-info">
                <!-- 发表时间 -->
                <span style="margin-right:10px">
                  {{ reply.created | dateTime }}
                </span>
                <!-- 点赞 -->
                <!-- <span
                  :class="isLike(reply.id) + ' iconfont icondianzan'"
                  @click="like(reply)"
                /> -->
                <!-- <span v-show="reply.likeCount > 0"> {{ reply.likeCount }}</span> -->
                <!-- 回复 -->
                <span class="reply-btn" @click="replyComment(index, reply)">
                  回复
                </span>
              </div>
              <!-- 回复内容 -->
              <p class="comment-content">
                <!-- 回复用户名 -->
                <!-- <template v-if="reply.userId != item.userId"> -->
                <template>
                  <!-- <span  class="ml-1">
                    @{{ reply.userName }}
                  </span> -->
                  <a
                    target="_blank"
                    class="comment-userName ml-1"
                  >
                    @{{ reply.userName }}
                  </a>
                  ：
                </template>
                <span v-html="reply.content" />
              </p>
            </div>
          </div>
          <!-- 回复数量 -->
          <div
            class="mb-3"
            style="font-size:0.75rem;color:#6d757a"
            v-show="item.replyCount > 0"
            ref="check"
          >
            共
            <b>{{ item.replyCount }}</b>
            条回复，
            <span
              style="color:#00a1d6;cursor:pointer"
              @click="checkReplies(index, item)"
            >
              点击查看
            </span>
          </div>
          <!-- 回复分页 -->
          <div
            class="mb-3"
            style="font-size:0.75rem;color:#222;display:none"
            ref="paging"
          >
            <span style="padding-right:10px">
              共{{ Math.ceil(item.replyCount / 5) }}页
            </span>
            <paging
              ref="page"
              :totalPage="Math.ceil(item.replyCount / 5)"
              :index="index"
              :commentId="item.id"
              @changeReplyCurrent="changeReplyCurrent"
            />
          </div>
          <!-- 回复框 -->
          <Reply ref="reply" @reloadReply="reloadReply" />
        </div>
      </div>
      <!-- 加载按钮 -->
      <div class="load-wrapper">
        <v-btn outlined v-if="count > commentList.length" @click="listComments">
          加载更多...
        </v-btn>
      </div>
    </div>
    <!-- 没有评论提示 -->
    <div v-else style="padding:1.25rem;text-align:center">
      来发评论吧~
    </div>
  </div>
</template>

<script>
import Reply from "./Reply";
import Paging from "./Paging";
import Emoji from "./Emoji";
import EmojiList from "../assets/js/emoji";
export default {
  components: {
    Reply,
    Emoji,
    Paging
  },
  props: {
    commentList: {
      type: Array
    },
    count: {
      type: Number
    }
  },
  data: function() {
    return {
      reFresh: true,
      content: "",
      chooseEmoji: false,
      current: 1
    };
  },
  methods: {
    replyComment(index, item) {
      this.$refs.reply.forEach(item => {
        item.$el.style.display = "none";
      });
      //传值给回复框
      this.$refs.reply[index].commentContent = "";
      this.$refs.reply[index].userName = item.userName;
      this.$refs.reply[index].email = item.email;
      this.$refs.reply[index].replyUserId = item.userId;
      this.$refs.reply[index].parentId = this.commentList[index].id;
      this.$refs.reply[index].chooseEmoji = false;
      this.$refs.reply[index].index = index;
      this.$refs.reply[index].$el.style.display = "block";
    },
    addEmoji(key) {
      this.content += key;
    },
    checkReplies(index, item) {
      // console.log("checkReplies评论回复:"+this.commentList[index].id)
      this.axios.get("/comments/reply/" + item.id, {
          params: { currentPage: 1 }
        })
        .then(({ data }) => {
          // console.log(data)
          this.commentList[index].replyDTOList = data.data.records
          // console.log(this.commentList)
          this.$refs.check[index].style.display = "none"
          //超过1页才显示分页
          if (Math.ceil(item.replyCount / 5) > 1) {
            this.$refs.paging[index].style.display = "flex";
          }
        });
    },
    changeReplyCurrent(current, index, commentId) {
      //查看下一页回复
      this.axios.get("/comments/reply/" + commentId, {
          params: { currentPage: current }
        })
        .then(({ data }) => {
          this.current = data.data.current
          this.commentList[index].replyDTOList = data.data.records;
        });
    },
    listComments() {
      //查看下一页评论
      this.current++;
      const path = this.$route.path;
      if(path == "/links"){
        this.axios.get("/comments/friendLink",{
          params: { currentPage: this.current }
        }).then(({ data }) => {
            this.commentList.push(...data.data.records);
        });
      }else {
        const arr = path.split("/");
        const articleId = arr[arr.length - 1];
        this.axios.get("/comments/article/"+articleId, {
            params: { currentPage: this.current }
          })
          .then(({ data }) => {
            this.commentList.push(...data.data.records);
            // console.log("currentPage:"+this.current)
          });
      }
    },
    insertComment() {
      //判断登录
      if (!this.$store.state.userId) {
        this.$store.state.loginFlag = true;
        return false;
      }
      //判空
      if (this.content.trim() == "") {
        this.$toast({ type: "error", message: "评论不能为空" });
        return false;
      }
      //解析表情
      var reg = /\[.+?\]/g;
      this.content = this.content.replace(reg, function(str) {
        return (
          "<img src= '" +
          EmojiList[str] +
          "' width='22'height='20' style='padding: 0 1px'/>"
        );
      });
      //发送请求
      const path = this.$route.path;
      const arr = path.split("/");
      var articleId = arr[arr.length - 1];
      if(path =="/links"){
        articleId = null
      }
      var comment = {
        userId : this.$store.state.userId,
        userName: this.$store.state.userName,
        email : this.$store.state.email,
        articleId: articleId,
        content: this.content,
        isReview: 0
      };
      this.content = "";
      this.axios.post("/comments/edit", comment,{
        headers: {
              "Authorization": sessionStorage.getItem("token")
          }
      }).then(({ data }) => {
        if (data.code == 200) {
          //查询最新评论
          this.$emit("reloadComment");
          this.$toast({ type: "success", message: data.msg });
        } else {
          this.$toast({ type: "error", message: data.msg });
        }
      });
    },
    like(comment) {
      //判断登录
      if (!this.$store.state.userId) {
        this.$store.state.loginFlag = true;
        return false;
      }
      //发送请求
      let param = new URLSearchParams();
      param.append("commentId", comment.id);
      this.axios.post("/comments/like", param).then(({ data }) => {
        if (data.flag) {
          //判断是否点赞
          if (this.$store.state.commentLikeSet.indexOf(comment.id) != -1) {
            this.$set(comment, "likeCount", comment.likeCount - 1);
          } else {
            this.$set(comment, "likeCount", comment.likeCount + 1);
          }
          this.$store.commit("commentLike", comment.id);
        }
      });
    },
    reloadReply(index) {
      // console.log("reloadReply评论回复:"+this.commentList[index].id)
      this.axios.get("/comments/reply/" + this.commentList[index].id, {
          params: {
            currentPage: this.$refs.page[index].current
          }
        })
        .then(({ data }) => {
          // console.log(data);
          this.commentList[index].replyCount++;
          //回复大于5条展示分页
          if (this.commentList[index].replyCount > 5) {
            this.$refs.paging[index].style.display = "flex";
          }
          this.$refs.check[index].style.display = "none";
          this.$refs.reply[index].$el.style.display = "none";
          this.commentList[index].replyDTOList = data.data.records;
        });
    }
  },
  computed: {
    isLike() {
      return function(commentId) {
        var commentLikeSet = this.$store.state.commentLikeSet;
        return commentLikeSet.indexOf(commentId) != -1 ? "like-active" : "like";
      };
    }
  },
  watch: {
    commentList() {
      this.reFresh = false;
      this.$nextTick(() => {
        this.reFresh = true;
      });
    }
  }
};
</script>

<style scoped>
p {
  margin-bottom: 1.25rem !important;
}
.blogger-tag {
  background: #ffa51e;
  font-size: 12px;
  display: inline-block;
  border-radius: 2px;
  color: #fff;
  padding: 0 5px;
  margin-left: 6px;
}
.comment-title {
  display: flex;
  align-items: center;
  font-size: 1.25rem;
  font-weight: bold;
  line-height: 40px;
  margin-bottom: 10px;
}
.comment-title i {
  font-size: 1.5rem;
  margin-right: 5px;
}
.comment-input-wrapper {
  border: 1px solid #90939950;
  border-radius: 4px;
  padding: 10px;
  margin: 0 0 10px;
}
.count {
  padding: 5px;
  line-height: 1.75;
  font-size: 1.25rem;
  font-weight: bold;
}
.comment-meta {
  margin-left: 0.8rem;
  width: 100%;
  border-bottom: 1px dashed #f5f5f5;
}
.reply-meta {
  margin-left: 0.8rem;
  width: 100%;
}
.comment-user {
  font-size: 14px;
  line-height: 1.75;
}
.comment-user a {
  color: #1abc9c !important;
  font-weight: 500;
  transition: 0.3s all;
}
.comment-userName {
  text-decoration: none;
  color: #1abc9c !important;
  font-weight: 500;
}
.comment-info {
  line-height: 1.75;
  font-size: 0.75rem;
  color: #b3b3b3;
}
.reply-btn {
  cursor: pointer;
  float: right;
  color: #ef2f11;
}
.comment-content {
  font-size: 0.875rem;
  line-height: 1.75;
  padding-top: 0.625rem;
}
.comment-avatar {
  transition: all 0.5s;
}
.comment-avatar:hover {
  transform: rotate(360deg);
}
.like {
  cursor: pointer;
  font-size: 0.875rem;
}
.like:hover {
  color: #eb5055;
}
.like-active {
  cursor: pointer;
  font-size: 0.875rem;
  color: #eb5055;
}
.load-wrapper {
  margin-top: 10px;
  display: flex;
  justify-content: center;
  align-items: center;
}
.load-wrapper button {
  background-color: #49b1f5;
  color: #fff;
}
</style>
