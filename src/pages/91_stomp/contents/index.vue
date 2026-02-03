<template>
  <v-container class="pa-4" fluid>
    <v-card>
              <v-card-title class="d-flex align-center pe-2">
                <v-icon icon="mdi-format-list-checkbox"></v-icon> &nbsp;
                Messages&nbsp;
              </v-card-title>

              <v-card-text>
                <v-table style="height: 86vh" fixed-header density="compact">
                  <thead>
                    <tr>
                      <th class="text-left" width="20%">Timestamp</th>
                      <th class="text-left" width="20%">Principal</th>
                      <th class="text-left">Payload</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="item in messages" :key="item.timestamp">
                      <td>{{ $moment.format(item.timestamp) }}</td>
                      <td>{{ item.principal }}</td>
                      <td>
                        <JsonViewer
                          class="m-0 p-0"
                          :value="item.payload"
                          theme="dark"
                        ></JsonViewer>
                      </td>
                    </tr>
                  </tbody>
                </v-table>
              </v-card-text>

</v-card>
</v-container>


</template>

<script>
const x = "[/contents/index]";
import $stompServer from "@/assets/backend/stomp-server";

export default {
  data: () => ({
    messages: [],
  }),

  computed: {},

  watch: {},

  methods: {
    messageReceived(m, p) {
      // console.log(x, "messageReceived()", m, p);
      this.messages.push(m);
    },
  },

  mounted() {

    $stompServer.stomp
      .connect()
      .then((r) => {
        console.log(x, "mounted()", 1, r);
        if(r != undefined) {
          $stompServer.stomp.subscribe(r, this.messageReceived);
        }
      })
      .catch((r) => {
        console.log(x, "mounted()", 2);
      });
  },
};
</script>
