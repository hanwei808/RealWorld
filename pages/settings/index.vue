<template>
  <div class="settings-page">
    <div class="container page">
      <div class="row">
        <div class="col-md-6 offset-md-3 col-xs-12">
          <h1 class="text-xs-center">Your Settings</h1>

          <form @submit.prevent="updateInfo">
            <fieldset>
              <fieldset class="form-group">
                <input
                  class="form-control"
                  type="text"
                  placeholder="URL of profile picture"
                  v-model="user.pictureUrl"
                />
              </fieldset>
              <fieldset class="form-group">
                <input
                  class="form-control form-control-lg"
                  type="text"
                  placeholder="Your Name"
                  v-model="user.username"
                />
              </fieldset>
              <fieldset class="form-group">
                <textarea
                  class="form-control form-control-lg"
                  rows="8"
                  placeholder="Short bio about you"
                  v-model="user.about"
                ></textarea>
              </fieldset>
              <fieldset class="form-group">
                <input
                  class="form-control form-control-lg"
                  type="text"
                  placeholder="Email"
                  v-model="user.email"
                />
              </fieldset>
              <fieldset class="form-group">
                <input
                  class="form-control form-control-lg"
                  type="password"
                  placeholder="Password"
                  v-model="user.pwd"
                />
              </fieldset>
              <button class="btn btn-lg btn-primary pull-xs-right">
                Update Settings
              </button>
            </fieldset>
          </form>
          <hr />
          <button @click="logout">click here to logout.</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
const Cookie = process.client ? require("js-cookie") : undefined;
import { update } from "@/api/user";
export default {
  middleware: "authenticated",
  name: "SettingsIndex",
  data() {
    return {
      user: {
        pictureUrl: "",
        about: "",
        email: "",
        pwd: "",
      },
    };
  },
  async asyncData({ params }) {
    console.log(params);
    return {
      user: { username: params.username },
    };
  },
  methods: {
    async updateInfo() {
      const { data } = await update(this.user);
      const { user } = data;
      this.$store.commit("setUser", data.user);
      Cookie.set("user", data.user);
      this.$router.push({
        name: "profile",
        params: {
          username: user.username,
        },
      });
    },
    logout() {
      Cookie.remove("user");
      this.$store.commit("setUser", null);
      this.$router.push("/");
    },
  },
};
</script>

<style>
</style>
