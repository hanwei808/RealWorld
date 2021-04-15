<template>
  <div class="article-page">
    <div class="banner">
      <div class="container">
        <h1>{{ article.title }}</h1>

        <article-meta
          :article="article"
          :profile="profile"
          @deal-follow="dealFollow()"
          @delete-article="deleteArticle()"
          @on-favorite="onFavorite()"
        />
      </div>
    </div>

    <div class="container page">
      <div class="row article-content">
        <div class="col-md-12" v-html="article.body"></div>
      </div>

      <hr />

      <div class="article-actions">
        <article-meta
          :article="article"
          :profile="profile"
          @deal-follow="dealFollow()"
          @delete-article="deleteArticle()"
          @on-favorite="onFavorite()"
        />
      </div>

      <div class="row">
        <div class="col-xs-12 col-md-8 offset-md-2">
          <article-comments :article="article" />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { mapState } from "vuex";
import {
  getArticle,
  deleteArticle,
  addFavorite,
  deleteFavorite,
} from "@/api/article";
import { getProfiles, follow, deleteFollow } from "@/api/user";
import MarkdownIt from "markdown-it";
import ArticleMeta from "./components/article-meta";
import ArticleComments from "./components/article-comments";

export default {
  name: "ArticleIndex",
  async asyncData({ params }) {
    const { data } = await getArticle(params.slug);
    const { article } = data;
    const { data: userData } = await getProfiles(article.author.username);
    const { profile } = userData;
    const md = new MarkdownIt();
    article.body = md.render(article.body);
    return {
      article,
      profile,
    };
  },
  components: {
    ArticleMeta,
    ArticleComments,
  },
  computed: {
    ...mapState(["user"]),
  },
  methods: {
    async dealFollow() {
      if (this.user == null) {
        this.$router.push("/login");
        return;
      }
      if (this.profile.following) {
        const { data } = await deleteFollow(this.profile.username);
        this.profile = data.profile;
      } else {
        const { data } = await follow(this.profile.username);
        this.profile = data.profile;
      }
    },
    async deleteArticle() {
      const { data } = await deleteArticle(this.article.slug);
      this.$router.go(-1);
    },
    async onFavorite() {
      if (this.user == null) {
        this.$router.push("/login");
        return;
      }
      this.article.favoriteDisabled = true;
      if (this.article.favorited) {
        // 取消点赞
        await deleteFavorite(this.article.slug);
        this.article.favorited = false;
        this.article.favoritesCount += -1;
      } else {
        // 添加点赞
        await addFavorite(this.article.slug);
        this.article.favorited = true;
        this.article.favoritesCount += 1;
      }
      this.article.favoriteDisabled = false;
    },
  },
  head() {
    return {
      title: `${this.article.title} - RealWorld`,
      meta: [
        {
          hid: "description",
          name: "description",
          content: this.article.description,
        },
      ],
    };
  },
};
</script>

<style>
</style>
