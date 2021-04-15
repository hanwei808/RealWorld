<template>
  <div class="profile-page">
    <div class="user-info">
      <div class="container">
        <div class="row">
          <div class="col-xs-12 col-md-10 offset-md-1">
            <img :src="profile.image" class="user-img" />
            <h4>{{ profile.username }}</h4>
            <p>{{ profile.bio }}</p>
            <button
              class="btn btn-sm btn-outline-secondary action-btn"
              @click="eiditOrFollow()"
            >
              <i class="ion-plus-round"></i>
              <span v-if="user.username == profile.username"
                >&nbsp; Edit Profile Settings</span
              >
              <span v-else
                >&nbsp;
                {{
                  (profile.following ? "Unfollow " : "Follow ") +
                  profile.username
                }}</span
              >
            </button>
          </div>
        </div>
      </div>
    </div>

    <div class="container">
      <div class="row">
        <div class="col-xs-12 col-md-10 offset-md-1">
          <div class="articles-toggle">
            <ul class="nav nav-pills outline-active">
              <li class="nav-item">
                <div
                  class="nav-link"
                  :class="{ active: isShowMyArticles }"
                  @click="() => (isShowMyArticles = true)"
                >
                  My Articles
                </div>
              </li>
              <li class="nav-item">
                <div
                  class="nav-link"
                  :class="{ active: !isShowMyArticles }"
                  @click="() => (isShowMyArticles = false)"
                >
                  Favorited Articles
                </div>
              </li>
            </ul>
          </div>

          <div
            class="article-preview"
            v-for="(article, index) in isShowMyArticles ? articles : articles2"
            :key="index"
          >
            <div class="article-meta">
              <a href=""><img :src="article.author.image" /></a>
              <div class="info">
                <a href="" class="author">{{ article.author.username }}</a>
                <span class="date">{{
                  article.createdAt | date("MMM DD, YYYY")
                }}</span>
              </div>
              <button class="btn btn-outline-primary btn-sm pull-xs-right">
                <i class="ion-heart"></i> {{ article.favoritesCount }}
              </button>
            </div>
            <nuxt-link
              :to="{
                name: 'article',
                params: {
                  slug: article.slug,
                },
              }"
              href=""
              class="preview-link"
            >
              <h1>{{ article.title }}</h1>
              <p>{{ article.description }}</p>
              <span>Read more...</span>
            </nuxt-link>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { mapState } from "vuex";
import { getProfiles, follow, deleteFollow } from "@/api/user";
import { getArticles } from "@/api/article";
export default {
  middleware: "authenticated",
  name: "UserProfile",
  async asyncData({ params }) {
    const { data: userData } = await getProfiles(params.username);
    const { profile } = userData;
    const { data } = await getArticles(
      `author=${params.username}&limit=5&offset=0`
    );
    const { articles } = data;
    const { data: data2 } = await getArticles(`favorited:${params.username}`);
    const { articles: articles2 } = data2;
    return {
      profile,
      articles,
      articles2,
      isShowMyArticles: true,
    };
  },
  computed: {
    ...mapState(["user"]),
  },
  methods: {
    async eiditOrFollow() {
      if (this.user.username == this.profile.username) {
        this.$router.push("/settings");
      } else {
        if (this.profile.following) {
          const { data } = await deleteFollow(this.profile.username);
          this.profile = data.profile;
        } else {
          const { data } = await follow(this.profile.username);
          this.profile = data.profile;
        }
      }
    },
  },
};
</script>

<style>
</style>
