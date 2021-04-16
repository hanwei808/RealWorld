<template>
  <div class="article-meta">
    <nuxt-link
      :to="{
        name: 'profile',
        params: {
          username: article.author.username,
        },
      }"
    >
      <img :src="article.author.image" />
    </nuxt-link>
    <div class="info">
      <nuxt-link
        class="author"
        :to="{
          name: 'profile',
          params: {
            username: article.author.username,
          },
        }"
      >
        {{ article.author.username }}
      </nuxt-link>
      <span class="date">{{ article.createdAt | date("MMM DD, YYYY") }}</span>
    </div>
    <button
      class="btn btn-sm btn-outline-secondary"
      :class="{
        active: article.author.following,
      }"
      @click="editOrFollow()"
    >
      <i
        :class="
          article.author.username == user.username
            ? 'ion-edit'
            : 'ion-plus-round'
        "
      ></i>
      <span v-if="user && article.author.username == user.username"
        >&nbsp; Edit Article</span
      >
      <span v-else
        >&nbsp;
        {{
          (profile.following ? "Unfollow " : "Follow ") + profile.username
        }}</span
      >
    </button>
    &nbsp;&nbsp;
    <button
      class="btn btn-sm btn-outline-primary"
      :class="{
        active: article.favorited,
      }"
      @click="deleteOrFavorite()"
    >
      <i
        :class="
          article.author.username == user.username ? 'ion-trash-a' : 'ion-heart'
        "
      ></i>
      <span v-if="user && article.author.username == user.username"
        >&nbsp; Delete Article</span
      >
      <span v-else>&nbsp; Favorite Post</span>
      <span
        class="counter"
        v-if="user && article.author.username != user.username"
        >({{ article.favoritesCount }})</span
      >
    </button>
  </div>
</template>

<script>
import { mapState } from "vuex";
export default {
  name: "ArticleMeta",
  props: {
    article: {
      type: Object,
      required: true,
    },
    profile: {
      type: Object,
      required: true,
    },
  },
  computed: {
    ...mapState(["user"]),
  },
  methods: {
    editOrFollow() {
      if (
        this.user != null &&
        this.article.author.username == this.user.username
      ) {
        this.$router.push({
          name: "editor",
          params: {
            slug: this.article.slug,
          },
        });
      } else {
        this.$emit("deal-follow");
      }
    },
    deleteOrFavorite() {
      if (
        this.user != null &&
        this.article.author.username == this.user.username
      ) {
        this.$emit("delete-article");
      } else {
        this.$emit("on-favorite");
      }
    },
  },
};
</script>

<style>
</style>
