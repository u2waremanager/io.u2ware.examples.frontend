<template>
  <v-container class="pa-4" fluid>
    <v-card>
      <v-card-title class="d-flex align-center pe-2">
        <v-icon icon="mdi-video-input-component"></v-icon> &nbsp; Accounts
        &nbsp;
      </v-card-title>

      <v-card-text>
        <JsonViewer class="m-0 p-0" :value="userinfo" theme="dark"></JsonViewer>
      </v-card-text>
    </v-card>
  </v-container>
</template>

<script>
const x = "[/contents/index]";
import $common from "@/assets/stores/common.js";

export default {
  data: () => ({}),

  computed: {
    subtitle: $common.computed.subtitle,
    userinfo: $common.computed.userinfo,
  },

  watch: {},

  methods: {},

  mounted() {
    this.subtitle = x


  },
};
</script>
