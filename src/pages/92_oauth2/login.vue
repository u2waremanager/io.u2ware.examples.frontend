<template>

  <v-app id="inspire">
    <v-app-bar app>
      <router-link to="/">
        <U2wareAvatar></U2wareAvatar>
      </router-link>

      <v-toolbar-title> {{ $t("oauth2.index.title") }}</v-toolbar-title>

      <v-spacer></v-spacer>

      <v-btn text variant="elevated" color="primary" to="/">
        <v-icon>mdi-home</v-icon>
      </v-btn>
    </v-app-bar>

    <U2wareFooter></U2wareFooter>

    <v-main>

      <v-container max-width="600px">
        <v-form validate-on="eager" @update:model-value="dialogValidate">
          <h2 class="ms-12 mt-12">Please sign in</h2>

          <v-text-field
            v-model="username"
            :rules="[$rules.requried]"
            label="Username"
            placeholder="Username"
            prepend-icon="mdi-account"
          ></v-text-field>


          <v-text-field
            v-model="password"
            :rules="[$rules.requried]"
            :type="showPassword ? 'text' : 'password'"
            label="Password"
            prepend-icon="mdi-lock"
            :append-inner-icon="showPassword ? 'mdi-eye' : 'mdi-eye-off'"
            @click:append-inner="showPassword = !showPassword"
          ></v-text-field>

            <v-btn
              class="ms-10"
              variant="elevated"
              color="primary"
              text="Login"
              :disabled="!validate"
              @click="login()"
            ></v-btn>

        </v-form>
      </v-container>
    <form ref="loginForm" method="post" action="/login">
      <input ref="usernameForm" type="hidden" name="username" />
      <input ref="passwordForm" type="hidden" name="password" />
    </form>
    </v-main>
  </v-app>






</template>

<script>
const x = "[/login]";

export default {
  data: () => ({
    username: undefined,
    password: undefined,
    validate: false,

    showPassword : false
  }),

  methods: {
    dialogValidate(r){
      this.validate = r;
    },

    login() {
      console.log("login");
      let loginForm = this.$refs["loginForm"];
      let usernameForm = this.$refs["usernameForm"];
      let passwordForm = this.$refs["passwordForm"];
      usernameForm.value = this.username;
      passwordForm.value = this.password;
      loginForm.submit();
    },
  },


  mounted(){
    console.log(this.$route.query);
  }
};
</script>
<style>
</style>