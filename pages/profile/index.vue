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
              <i
                :class="
                  user.username == profile.username
                    ? 'ion-gear-a'
                    : 'ion-plus-round'
                "
              ></i>
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
              <li v-if="user" class="nav-item">
                <nuxt-link
                  class="nav-link"
                  :class="{
                    active: tab === 'my_articles',
                  }"
                  exact
                  :to="{
                    name: 'profile',
                  }"
                  >My Articles</nuxt-link
                >
              </li>
              <li class="nav-item">
                <nuxt-link
                  class="nav-link"
                  :class="{
                    active: tab === 'favorited_articles',
                  }"
                  exact
                  :to="{
                    name: 'profile',
                    query: {
                      tab: 'favorited_articles',
                    },
                  }"
                  >Favorited Articles</nuxt-link
                >
              </li>
            </ul>
          </div>

          <div
            class="article-preview"
            v-for="(article, index) in articles"
            :key="index"
          >
            <div class="article-meta">
              <nuxt-link
                class=""
                :to="{
                  name: 'profile',
                  params: {
                    username: article.author.username,
                  },
                  query: {
                    page: 1,
                    tab: 'my_articles',
                  },
                }"
                ><img :src="article.author.image"
              /></nuxt-link>
              <div class="info">
                <nuxt-link
                  class="author"
                  :to="{
                    name: 'profile',
                    params: {
                      username: article.author.username,
                    },
                    query: {
                      page: 1,
                      tab: 'my_articles',
                    },
                  }"
                  >{{ article.author.username }}</nuxt-link
                >
                <span class="date">{{
                  article.createdAt | date("MMM DD, YYYY")
                }}</span>
              </div>
              <button
                class="btn btn-outline-primary btn-sm pull-xs-right"
                @click="onFavorite(article)"
              >
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
              <ul class="tag-list">
                <li
                  class="tag-default tag-pill tag-outline ng-binding ng-scope"
                  ng-repeat="tag in ::$ctrl.article.tagList"
                  v-for="(tag, index) in article.tagList"
                  :key="index"
                >
                  {{ tag }}
                </li>
              </ul>
            </nuxt-link>
          </div>

          <!-- 分页列表 -->
          <nav>
            <ul class="pagination">
              <li
                class="page-item"
                :class="{
                  active: item === page,
                }"
                v-for="item in totalPage"
                :key="item"
              >
                <nuxt-link
                  class="page-link"
                  :to="{
                    name: 'profile',
                    query: {
                      page: item,
                      tab: tab,
                    },
                  }"
                  >{{ item }}</nuxt-link
                >
              </li>
            </ul>
          </nav>
          <!-- /分页列表 -->
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { mapState } from "vuex";
import { getProfiles, follow, deleteFollow } from "@/api/user";
import { getArticles, deleteFavorite, addFavorite } from "@/api/article";
export default {
  middleware: "authenticated",
  name: "UserProfile",
  async asyncData({ params, query }) {
    const { data: userData } = await getProfiles(params.username);
    const { profile } = userData;
    const page = Number.parseInt(query.page || 1);
    const limit = 5;
    const tab = query.tab || "my_articles";

    const { data } = await getArticles({
      author: tab == "my_articles" ? params.username : undefined,
      favorited: tab != "my_articles" ? params.username : undefined,
      limit,
      offset: (page - 1) * limit,
    });
    const { articles, articlesCount } = data;

    articles.forEach((article) => (article.favoriteDisabled = false));

    return {
      profile,
      articles, // 文章列表
      articlesCount, // 文章总数
      limit, // 每页大小
      page, // 页码
      tab, // 选项卡
    };
  },
  watchQuery: ["page", "tab"],
  computed: {
    ...mapState(["user"]),
    totalPage() {
      return Math.ceil(this.articlesCount / this.limit);
    },
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
    async onFavorite(article) {
      if (this.user == null) {
        this.$router.push("/login");
        return;
      }
      article.favoriteDisabled = true;
      if (article.favorited) {
        // 取消点赞
        await deleteFavorite(article.slug);
        article.favorited = false;
        article.favoritesCount += -1;
      } else {
        // 添加点赞
        await addFavorite(article.slug);
        article.favorited = true;
        article.favoritesCount += 1;
      }
      article.favoriteDisabled = false;
    },
  },
};
</script>

<style>
</style>
