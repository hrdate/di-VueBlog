<template>
  <div>
    <!-- banner -->
    <div class="about-banner banner">
      <h1 class="banner-title">关于我</h1>
    </div>
    <!-- 关于我内容 -->
    <v-card class="blog-container">
      <div class="my-wrapper">
        <v-avatar size="110">
          <img
            class="author-avatar"
            src="../../assets/img/my.jpg"
          />
        </v-avatar>
      </div>
      <div class="about-content markdown-body" v-html="websiteIntro" />
    </v-card>
  </div>
</template>

<script>
export default {
  created() {
    this.getAboutContent();
  },
  data: function() {
    return {
      websiteIntro: '',
    };
  },
  methods: {
    getAboutContent() {
      const MarkdownIt = require("markdown-it");
      const md = new MarkdownIt();
      this.websiteIntro = md.render(this.$store.state.websiteConfigForm.websiteIntro);
    }
  }
};
</script>

<style scoped>
.about-banner {
  background: url(../../assets/img/bokeyuan_2.jpg) center center / cover
    no-repeat;
  background-color: #49b1f5;
}
.about-content {
  word-break: break-word;
  line-height: 1.8;
  font-size: 14px;
}
.my-wrapper {
  text-align: center;
}
.author-avatar {
  transition: all 0.5s;
}
.author-avatar:hover {
  transform: rotate(360deg);
}
</style>
