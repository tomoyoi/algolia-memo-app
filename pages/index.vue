<template>
  <div>
    <b-form-row>
      <b-col cols="10">
        <b-form-input type="text" v-model="query" />
      </b-col>
      <b-col cols="2">
        <b-button block @click="searchMemo">検索</b-button>
      </b-col>
    </b-form-row>
    <b-button class="my-2" variant="primary" block @click="openRegisterModal"
      >メモを追加</b-button
    >

    <b-card class="my-2"
            v-for="memo in memoList"
            :key="memo.objectID">
      <b-card-title v-html="memo.title"></b-card-title>
      <p v-html="memo.description"></p>
      <b-button v-if="!memo.done" variant="primary" @click="makeMemoDone(memo.objectID)">完了</b-button>
      <b-button v-else disabled>完了済</b-button>
      <b-button @click="openUpdateModal(memo.objectID)">編集</b-button>
      <b-button @click="deleteMemo(memo.objectID)">削除</b-button>
    </b-card>

    <b-modal
      v-model="registerModalIsVisible"
      @ok="registerMemo"
      @calcel="clearInput"
    >
      <b-form-group label="Title" label-for="title">
        <b-form-input id="title" type="text" v-model="memoInput.title" />
      </b-form-group>
      <b-form-group label="Description" label-for="description">
        <b-form-input
          id="description"
          type="text"
          v-model="memoInput.description"
        />
      </b-form-group>
    </b-modal>

    <b-modal v-model="updateModalIsVisible"
             @ok="updateMemo"
             @calcel="clearInput">
      <b-form-group label="Title" label-for="title">
       <b-form-input id="update-title" type="text" v-model="memoInput.title" />
      </b-form-group>
      <b-form-group label="Description" label-for="description">
       <b-form-input id="update-description" type="text" v-model="memoInput.description" />
      </b-form-group>
    </b-modal>
    <b-card
      class="my-2"
      v-for="memo in memoList"
      :key="memo.objectID"
      :title="memo.title"
    >
    <p>{{ memo.description }}</p>
    </b-card>
  </div>
</template>

<script>
import * as algoliasearch from "algoliasearch";
import config from "~/algolia.config.js";

const client = algoliasearch(config.appId, config.apiKey);
const index = client.initIndex("memo");

export default {
  data() {
    return {
      memoInput: {
        title: "",
        description: "",
        done: false,
      },
      registerModalIsVisible: false,
      updateModalIsVisible: false,
      query: '',
      memoList: []
    };
  },
  methods: {
    clearInput() {
      this.memoInput.title = "";
      (this.memoInput.description = ""), (this.memoInput.done = false);
    },
    openRegisterModal() {
      this.registerModalIsVisible = true;
    },
    async registerMemo() {
      await index.saveObject(this.memoInput, {
        autoGenerateObjectIDIfNotExist: true,
      });
      this.clearInput();
    },
    async searchMemo () {
      let searchResult = await index.search(this.query)
      this.memoList = searchResult.hits
    },
    openUpdateModal (objectID) {
      const memo = this.getMemoByObjectID(objectID)
      this.memoInput.title = memo.title
      this.memoInput.description = memo.description
      this.updateTargetObjectID = objectID
      this.updateModalIsVisible = true
    },
    async updateMemo () {
      await index.saveObject({
        objectID: this.updateTargetObjectID,
        ...this.memoInput
      })
      await this.searchMemo()
      this.clearInput()
    },
    async deleteMemo (objectID) {
      await index.deleteObject(objectID)
      await this.searchMemo()
    },
    async makeMemoDone (objectID) {
      await index.partialUpdateObject({
        objectID: objectID,
        done: true
      })
      await this.searchMemo()
    },
    async searchMemo () {
      let searchResult = await index.search(this.query)
      this.memoList = searchResult.hits
    },
    getMemoByObjectID(objectID) {
      return this.memoList.filter(memo => memo.objectID === objectID)[0]
    }
  },
};
</script>