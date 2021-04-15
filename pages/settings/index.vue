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
                  v-model="modifyUser.pictureUrl"
                />
              </fieldset>
              <fieldset class="form-group">
                <input
                  class="form-control form-control-lg"
                  type="text"
                  placeholder="Your Name"
                  v-model="modifyUser.username"
                />
              </fieldset>
              <fieldset class="form-group">
                <textarea
                  class="form-control form-control-lg"
                  rows="8"
                  placeholder="Short bio about you"
                  v-model="modifyUser.about"
                ></textarea>
              </fieldset>
              <fieldset class="form-group">
                <input
                  class="form-control form-control-lg"
                  type="text"
                  placeholder="Email"
                  v-model="modifyUser.email"
                />
              </fieldset>
              <fieldset class="form-group">
                <input
                  class="form-control form-control-lg"
                  type="password"
                  placeholder="Password"
                  v-model="modifyUser.pwd"
                />
              </fieldset>
              <button class="btn btn-lg btn-primary pull-xs-right">
                Update Settings
              </button>
            </fieldset>
          </form>
          <hr />
          <button
            style="border: solid 1px #b85c5c; background: none; color: #b85c5c"
            @click="logout"
          >
            click here to logout.
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
const Cookie = process.client ? require("js-cookie") : undefined;
import { mapState } from "vuex";
import { update } from "@/api/user";
export default {
  middleware: "authenticated",
  name: "SettingsIndex",
  data() {
    return {
      modifyUser: {
        pictureUrl: "",
        about: "",
        email: "",
        pwd: "",
      },
    };
  },
  // async asyncData({ params }) {
  //   console.log(params);
  //   return {
  //     user: { username: params.username },
  //   };
  // },
  computed: {
    ...mapState(["user"]),
  },
  mounted() {
    this.modifyUser = JSON.parse(JSON.stringify(this.user));
  },
  methods: {
    async updateInfo() {
      const { data } = await update(this.modifyUser);
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
