<template>
  <v-container class="pa-4" fluid>
    <v-card>
      <v-card-title class="d-flex align-center pe-2">
        <v-icon icon="mdi-video-input-component"></v-icon> &nbsp; 
        {{ $t("contents.keys.title") }}&nbsp;        
      </v-card-title>
      <v-card-text>
        <!-- <v-textarea v-model="keys" readonly rows="20"></v-textarea> -->
        <pre class="pa-10">{{ keys }}</pre>
      </v-card-text>
    </v-card>
  </v-container>
</template>

<script>
const x = "[/contents/keys]";
import $oauth2Server from "@/assets/backend/oauth2-server.js";
import $common from "@/assets/stores/common.js";

export default {
  data: () => ({

    keys : ""
  }),

  computed: {
    subtitle: $common.computed.subtitle,
    userinfo: $common.computed.userinfo,
  },

  watch: {},

  methods: {},

  mounted() {

    this.subtitle = x;
    $oauth2Server.oauth2.publicKey()
    .then((r)=>{
      console.log(x, "mounted", 1, r);
      this.keys = r;     
    })
    .catch((r)=>{
      console.log(x, "mounted", 2, r);
      this.keys = "......Error.....";
    })

  },
};
</script>