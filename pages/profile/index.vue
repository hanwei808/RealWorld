<template>
  <div class="profile-page">
    <div class="user-info">
      <div class="container">
        <div class="row">
          <div class="col-xs-12 col-md-10 offset-md-1">
            <img src="http://i.imgur.com/Qr71crq.jpg" class="user-img" />
            <h4>{{ user.username }}</h4>
            <p>{{ user.bio }}</p>
            <button class="btn btn-sm btn-outline-secondary action-btn">
              <i class="ion-plus-round"></i>
              &nbsp; Follow {{ user.username }}
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
              <a href=""><img src="http://i.imgur.com/Qr71crq.jpg" /></a>
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
import { getProfiles } from "@/api/user";
import { getArticles } from "@/api/article";
export default {
  middleware: "authenticated",
  name: "UserProfile",
  async asyncData({ params }) {
    // const { data: userData } = await getProfiles(params.username);
    const { data } = await getArticles(
      `author:${params.username}&limit=5&offset=0`
    );
    const { articles } = data;
    const { data: data2 } = await getArticles(`favorited:${params.username}`);
    const { articles: articles2 } = data2;
    return {
      articles,
      articles2,
      isShowMyArticles: true,
    };
  },
  computed: {
    ...mapState(["user"]),
  },
  methods: {},
};
</script>

<style>
</style>
