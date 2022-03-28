<template>
  <section class="container">
    <v-container>
      <v-form ref="form" onSubmit="return false;">
        <v-row justify="space-between">
          <v-col cols="12" md="4">
            <v-text-field
              v-model="search_word"
              label="検索ワード"
              @keydown.enter.exact="searchBookInfo"
              :counter="max"
              :rules="rules"
            ></v-text-field>
          </v-col>
        </v-row>
        <v-row align="center">
          <v-col cols="6">
            <v-select
              v-model="select_sort"
              :items="select_sort_items"
              item-text="state"
              item-value="val"
              return-object
              label="並び順"
              outlined
            ></v-select>
          </v-col>
        </v-row>
        <v-row align="center">
          <v-col cols="6">
            <v-select
              v-model="select_hits"
              :items="select_hits_items"
              item-text="state"
              item-value="val"
              return-object
              label="最大表示数"
              outlined
            ></v-select>
          </v-col>
        </v-row>
        <v-row align="center">
          <v-col cols="6">
            <v-select
              v-model="select_booksGenreId"
              :items="select_booksGenreId_items"
              item-text="state"
              item-value="val"
              return-object
              label="ジャンル絞り込み"
              outlined
            ></v-select>
          </v-col>
        </v-row>
        <v-row align="center">
          <v-col cols="6">
            <v-btn color="primary" @click="searchBookInfo">検索</v-btn>
          </v-col>
        </v-row>
      </v-form>
    </v-container>
    <v-row>
      <v-col
        v-for="item of bookData"
        :key="item.title"
        class="d-flex child-flex"
        cols="4"
      >
        <v-hover v-slot="{ hover }">
          <v-card
            class="mx-auto"
            color="grey lighten-4"
            max-width="600"
            @click="externalLink(item.Item.itemUrl)"
          >
            <v-img
              contain
              :src="`${item.Item.largeImageUrl}`"
              :lazy-src="`${item.Item.smallImageUrl}`"
              aspect-ratio="1"
              class="grey lighten-2"
            >
              <v-expand-transition>
                <div
                  v-if="hover"
                  class="
                    d-flex
                    transition-fast-in-fast-out
                    orange
                    darken-2
                    v-card--reveal
                    text-h3
                    white--text
                  "
                  style="height: 100%"
                >
                  ¥{{ item.Item.itemPrice }}
                </div>
              </v-expand-transition>
            </v-img>
            <v-card-text class="pt-1" style="position: relative">
              <div class="font-weight-light black--text">
                {{ item.Item.title }}
              </div>
            </v-card-text>
          </v-card>
        </v-hover>
      </v-col>
    </v-row>
  </section>
</template>

<script>
const axios = require("axios");
export default {
  data() {
    return {
      search_word: "",
      bookData: {},
      allowSpaces: true,
      max: 50,
      model: "search_word",
      select_sort: { state: "売上順", val: "sales" },
      select_sort_items: [
        { state: "売上", val: "sales" },
        { state: "評価", val: "reviewAverage" },
        { state: "レビュー数", val: "reviewCount" },
      ],
      select_hits: { state: "5", val: "5" },
      select_hits_items: [
        { state: "5", val: "5" },
        { state: "10", val: "10" },
        { state: "30", val: "30" },
      ],
      select_booksGenreId: { state: "指定なし", val: "000" },
      select_booksGenreId_items: [
        { state: "指定なし", val: "000" },
        { state: "漫画（コミック）", val: "001001" },
        { state: "語学・学習参考書", val: "001002" },
        { state: "絵本・児童書・図鑑", val: "001003" },
        { state: "小説・エッセイ", val: "001004" },
        { state: "パソコン・システム開発", val: "001005" },
        { state: "ビジネス・経済・就職", val: "001006" },
        { state: "旅行・留学・アウトドア", val: "001007" },
        { state: "人文・思想・社会", val: "001008" },
        { state: "ホビー・スポーツ・美術", val: "001009" },
        { state: "美容・暮らし・健康・料理", val: "001010" },
        { state: "エンタメ・ゲーム", val: "001011" },
        { state: "科学・技術", val: "001012" },
        { state: "写真集・タレント", val: "001013" },
        { state: "資格・検定", val: "001016" },
        { state: "ライトノベル", val: "001017" },
        { state: "楽譜", val: "001018" },
        { state: "文庫", val: "001019" },
        { state: "新書", val: "001020" },
        { state: "ボーイズラブ（BL）", val: "001021" },
        { state: "付録付き", val: "001022" },
        { state: "バーゲン本", val: "001023" },
        { state: "セット本", val: "001025" },
        { state: "カレンダー・手帳・家計簿", val: "001026" },
        { state: "文具・雑貨", val: "001027" },
        { state: "医学・薬学・看護学・歯科学", val: "001028" },
        { state: "ティーンズラブ（TL）", val: "001029" },
      ],
    };
  },

  computed: {
    rules() {
      const rules = [];
      if (this.max) {
        const rule = (v) =>
          (v || "").length <= this.max ||
          `A maximum of ${this.max} characters is allowed`;

        rules.push(rule);
      }

      if (!this.allowSpaces) {
        const rule = (v) =>
          (v || "").indexOf(" ") < 0 || "No spaces are allowed";

        rules.push(rule);
      }

      return rules;
    },
  },

  watch: {
    match: "validateField",
    max: "validateField",
    model: "validateField",
  },

  methods: {
    validateField() {
      this.$refs.form.validate();
    },

    searchBookInfo() {
      let url =
        "https://app.rakuten.co.jp/services/api/BooksTotal/Search/20170404?format=json&page=1&applicationId=" +
        process.env.RAKUTEN_API_ID;
      if (this.search_word) {
        url += "&keyword=" + this.search_word;
      }
      if (this.select_sort.val) {
        url += "&sort=" + this.select_sort.val;
      }
      if (this.select_hits.val) {
        url += "&hits=" + this.select_hits.val;
      }
      if (this.select_booksGenreId.val) {
        url += "&booksGenreId=" + this.select_booksGenreId.val;
      }
      console.log(url);
      axios.get(encodeURI(url)).then((res) => {
        this.bookData = res.data["Items"];
      });
    },

    externalLink(url) {
      open(url, "_blank");
    },

    test() {
      console.log(1);
    },
  },
};
</script>
