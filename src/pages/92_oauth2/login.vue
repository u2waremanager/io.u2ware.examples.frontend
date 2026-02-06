<template>
  <v-app id="inspire">
    <v-main>
      <v-container class="pa-12" fluid>
        <v-form validate-on="eager" @update:model-value="dialogValidate">
          <h2>Please sign in </h2>

          <v-text-field
            v-model="username2"
            :rules="[$rules.requried]"
            label="username"
            placeholder="username"
          ></v-text-field>

          <v-text-field
            v-model="password2"
            :rules="[$rules.requried]"
            label="password"
            placeholder="password"
          ></v-text-field>

            <v-btn
              variant="elevated"
              color="primary"
              text="Login"
              :disabled="!validate"
              @click="login()"
            ></v-btn>

        </v-form>
      </v-container>
    </v-main>

    <pre>{{ query }}</pre>

    <form ref="loginForm" method="post" action="/login">
      <input ref="username" type="hidden" name="username" />
      <input ref="password" type="hidden" name="password" />
    </form>
  </v-app>
</template>

<script>
const x = "[/login]";

export default {
  data: () => ({
    username2: undefined,
    password2: undefined,
    validate: false,
    query : undefined,
  }),

  methods: {
    dialogValidate(r){
      this.validate = r;
    },

    login() {
      console.log("login");
      let loginForm = this.$refs["loginForm"];
      let username = this.$refs["username"];
      let password = this.$refs["password"];
      username.value = this.username2;
      password.value = this.password2;
      loginForm.submit();
    },
  },


  mounted(){
    console.log(this.$route.query);

    this.query = this.$route.query;
  }
};
</script>
