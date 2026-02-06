<template>
  <v-container class="pa-4" fluid>
    <v-card>
      <v-card-title class="d-flex align-center pe-2">
        <v-icon icon="mdi-video-input-component"></v-icon> &nbsp;
        {{ $t("oauth2.contents.tokens.title") }}&nbsp;
        <!-- 
        //////////////////////////
        // Search Field Start
        //////////////////////////
        -->
        <v-text-field
          class="ms-10"
          v-model="searchForm.name"
          density="compact"
          label="Search"
          prepend-inner-icon="mdi-magnify"
          variant="solo-filled"
          flat
          hide-details
          single-line
        ></v-text-field>
        <!-- 
        //////////////////////////
        // Search Field End
        //////////////////////////
        -->
      </v-card-title>


      <v-divider></v-divider>

      <v-data-table-server
        fixed-header
        density="compact"
        :loading="loading"
        :search="config.searchBy"
        :page="1"
        :items-per-page="20"
        :sort-by="config.sortBy"
        :items-per-page-options="config.itemsPerPageOptions"
        :headers="config.headers"
        :item-value="config.itemValue"
        :items-length="entitiesTotal"
        :items="entities"
        @update:options="searchAction"
      >
        <!-- 
        //////////////////////////
        # Table Cell Template Start
        //////////////////////////
        -->
        <template v-slot:item.id="{ item }">
          <v-btn
            variant="plain"
            color="primary"
            :text="item.id"
            style="text-transform: none"
            @click="readAction(item)"
          ></v-btn>
        </template>

        <template v-slot:item.timestamp="{ item }">
          {{ $moment.format(item.timestamp) }}
        </template>
<!--         
        <template v-slot:item.tokenValue="{ item }">
            <JsonViewer class="m-0 p-0" :value="item.tokenValue" theme="dark"></JsonViewer>

        </template>
 -->
        <!-- 
        //////////////////////////
        # Table Cell Template End
        //////////////////////////
        -->

        <template v-slot:footer.prepend>
          <v-btn class="ms-1" text variant="elevated" @click="refreshAction">
            <v-icon>mdi-refresh</v-icon>
          </v-btn>

          <v-btn
            class="ms-1"
            text
            variant="elevated"
            color="primary"
            @click="newAction"
          >
            <v-icon>mdi-plus</v-icon>
          </v-btn>

          <v-spacer></v-spacer>
        </template>
      </v-data-table-server>

      <v-dialog v-model="dialog" persistent width="800">
        <v-card
          prepend-icon="mdi-update"
          :title="$t('oauth2.contents.tokens.title')"
          :subtitle="editForm.userId"
        >
          <v-card-text>
            <v-form validate-on="eager" @update:model-value="dialogValidate">
              <!-- 
              //////////////////////////
              # Edit Form Start
              //////////////////////////
              -->
              <v-text-field
                v-model="editForm.provider"
                :rules="[$rules.requried]"
                label="provider"
                placeholder="provider"
                :disabled="!isNew"
                hint="......."
                variant="outlined"
              ></v-text-field>

              <v-text-field
                v-model="editForm.name"
                :rules="[$rules.requried]"
                label="name"
                placeholder="name"
                :disabled="!isNew"
                hint="......."
                variant="outlined"
              ></v-text-field>
             
              <v-text-field
                v-model="editForm.timestamp"
                v-if="! isNew"
                :rules="[$rules.requried]"
                label="timestamp"
                placeholder="timestamp"
                :disabled="!isNew"
                hint="......."
                variant="outlined"
              ></v-text-field>

              <v-textarea
                v-model="editForm.tokenValue"
                v-if="! isNew"
                :rules="[$rules.requried]"
                :append-icon="copyTokenIcon"
                @click:append="copyToken(editForm.tokenValue)"
                label="tokenValue"
                placeholder="tokenValue"
                readonly
                hint="......."
                variant="outlined"
              ></v-textarea>


              <!-- 
              //////////////////////////
              # Edit Form End
              //////////////////////////
              -->
            </v-form>
          </v-card-text>

          <v-card-actions>
            <v-btn
              v-if="isNew"
              class="ms-5"
              variant="elevated"
              color="primary"
              text="Save"
              :disabled="!validate"
              @click="isNew ? createAction(e) : updateAction(e)"
            ></v-btn>
            <v-btn text="Cancel" @click="cancelAction"></v-btn>

            <v-spacer></v-spacer>
            <v-btn
              v-if="!isNew"
              color="error"
              text="Delete"
              variant="text"
              @click="deleteAction"
            ></v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </v-card>
  </v-container>
</template>
<script>
const x = "[/contents/tokens]";
import $oauth2Server from "@/assets/backend/oauth2-server";
import $common from "@/assets/stores/common.js";

export default {
  data: () => ({

    copyTokenIcon : "mdi-content-copy",
    
    loading: false,
    dialog: false,
    isNew: false,
    validate: false,

    searchForm: {},
    editForm: {},

    entities: [],
    entitiesTotal: 0,
    
    config: {
      searchBy: "",
      itemsPerPageOptions: [
        { value: 10, title: "10" },
        { value: 20, title: "20" },
        { value: 50, title: "50" },
        { value: 100, title: "100" },
      ],

      ///////////////////////////////
      // Config Start
      ///////////////////////////////
      itemValue: "username",

      headers: [
        { key: "id", title: "id", align: "start" },
        { key: "timestamp", title: "timestamp", align: "start" },
        { key: "provider", title: "provider", align: "start" },
        { key: "name", title: "name", align: "start" },
      ],
      sortBy: [
        {key: "timestamp", order: "desc" }
      ],

      initForm: {

      }
      /////////////////////////////////
      // Config End
      /////////////////////////////////
    },
  }),

  watch: {
    searchForm: {
      handler(e) {
        this.refreshAction();
      },
      deep: true,
    },
  },

  computed: {
    subtitle: $common.computed.subtitle,
    userinfo : $common.computed.userinfo,
  },


  methods: {

    copyToken(v){

      navigator.clipboard.writeText(v)
      .then((r)=>{
        console.log(x, "copyToken" , 2, r);
        this.copyTokenIcon = "mdi-content-paste";
      })
      .catch((r)=>{
        console.log(x, "copyToken" , 3, r);
        this.copyTokenIcon = "mdi-content-copy";
        this.$dialog.confirm(r.message);
      })
      ;
    },


    ////////////////////////////////////////
    // handle....
    ////////////////////////////////////////
    handleCreate(){
      return $oauth2Server.tokens.create(this.editForm);
    },
    handleRead(entity){
      return $oauth2Server.tokens.read(entity);
    },
    handleUpdate(){
      return $oauth2Server.tokens.update(this.editForm);
    },
    handleDelete(){
      return $oauth2Server.tokens.delete(this.editForm);
    },
    handleSearch(query){
      return $oauth2Server.tokens.search(this.searchForm, query);
    },
    handleEntities(res){
      this.entitiesTotal = res.page.totalElements;
      this.entities = res._embedded.tokens;
      return res;
    },
    handleEntity(res){
      this.editForm = res ? res : Object.assign({}, this.config.initForm);
      return res;
    },

    ////////////////////////////////////////
    //
    ////////////////////////////////////////
    dialogOpen(isNew) {
      this.dialog = true;
      this.isNew = isNew;
      return "opened";
    },
    dialogClose() {
      this.dialog = false;
      this.isNew = false;
      return "closed";
    },
    dialogValidate(r){
      this.validate = r;
    },

    ////////////////////////////////////////
    //
    ////////////////////////////////////////
    confirmBefore(code) {
      let msg = `$dialog.before.${code}`;
      return this.$dialog.confirm(msg);
    },
    confirmAfter(code) {
      let msg = `$dialog.after.${code}`;
      return this.$dialog.alert(msg);
    },
    confirmError(code) {
      let msg = `$dialog.error.${code}`;
      return this.$dialog.alert(msg, code);
    },

    ////////////////////////////////////////
    //
    ////////////////////////////////////////    
    actionStart(loading) {
      this.loading = true == loading ? true : false;
      return Promise.resolve();
    },
    actionEnd(refresh) {
      this.loading = false;
      if (true == refresh) {
        this.dialog = false;
        this.isNew = false;
        this.refreshAction();
      }
    },

    ////////////////////////////////////////
    //
    ////////////////////////////////////////
    refreshAction() {
      this.config.searchBy = String(Date.now());
    },

    searchAction(params) {
      this.actionStart(true)
        .then((r) => {
          return this.handleSearch(params);
        })
        .then((r) => {
          return this.handleEntities(r);
        })
        .then((r) => {
          return this.actionEnd(false);
        })
        .catch((e) => {
          return this.confirmError(e);
        })
        .catch((e) => {
          this.$router.push("/");
        });
    },

    newAction() {
      this.actionStart(true)
        .then((r) => {
          return this.handleEntity();
        })
        .then((r) => {
          return this.dialogOpen(true);
        })
        .then((r) => {
          return this.actionEnd(false);
        });
    },

    cancelAction() {
      this.actionStart(true)
        .then((r) => {
          return this.dialogClose();
        })
        .then((r) => {
          return this.handleEntity();
        })
        .then((r) => {
          return this.actionEnd(false);
        });
    },

    createAction() {
      this.confirmBefore("create")
        .then((r) => {
          return this.actionStart(true);
        })
        .then((r) => {
          return this.handleCreate();
        })
        .then((r) => {
          return this.confirmAfter("create");
        })
        .then((r) => {
          return this.handleEntity();
        })
        .then((r) => {
          return this.actionEnd(true);
        })
        .catch((r) => {
          return this.confirmError(r);
        })
        .catch((r) => {
          return this.actionEnd(409 != r.error);
        });
    },

    readAction(entity) {
      this.actionStart(true)
        .then((r) => {
          return this.handleRead(entity);
        })
        .then((r) => {
          return this.handleEntity(r);
        })
        .then((e) => {
          return this.dialogOpen(false);
        })
        .then((e) => {
          return this.actionEnd(false);
        })
        .catch((e) => {
          return this.confirmError(e);
        })
        .catch((e) => {
          return this.actionEnd(true);
        });
    },

    updateAction() {
      this.confirmBefore("update")
        .then((r) => {
          return this.actionStart(true);
        })
        .then((r) => {
          return this.handleUpdate();
        })
        .then((r) => {
          console.log(1);
          return this.confirmAfter("update");
        })
        .then((r) => {
          console.log(2);
          return this.handleEntity();
        })
        .then((r) => {
          console.log(3);
          return this.actionEnd(true);
        })
        .catch((r) => {
          console.log(4);
          return this.confirmError(r);
        })
        .catch((r) => {
          return this.actionEnd(true);
        });
    },

    deleteAction() {
      this.confirmBefore("delete")
        .then((r) => {
          return this.actionStart(true);
        })
        .then((r) => {
          return this.handleDelete();
        })
        .then((r) => {
          return this.confirmAfter("delete");
        })
        .then((r) => {
          return this.handleEntity();
        })
        .then((r) => {
          return this.actionEnd(true);
        })
        .catch((r) => {
          return this.confirmError(r);
        })
        .catch((r) => {
          return this.actionEnd(true);
        });
    },
  },

  mounted() {
    this.subtitle = x;
  },

};
</script>
