<template>
  <div class="media">
    <div class="wrap">
      <div class="media-content">
        <ul>
          <li v-for="item in mediaData" :key="item.id"><img :src="item.source_url" alt="item.slug"></li>
        </ul>
        <div class="list-loading">
          <a href="javascript:;" class="weui-btn weui-btn_default" @click="getMore" v-if="loadMore">{{Tran_loadMore}}</a>
          <div class="weui-loadmore weui-loadmore_line" v-else>
            <span class="weui-loadmore__tips">{{Tran_noneMore}}</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { WPBlogSiteUrl, apiUrl } from "../utils/api.js";

export default {
  data() {
    return {
      mediaData: [],
      Tran_noneMore: this.APLang.noneMore,
      Tran_loadMore: this.APLang.loadMore,
      loadMore: false,
      pages: {
        page_count: 0,
        page: 1,
        per_page: 20
      },
    };
  },
  mounted: function() {
    this.showPGConfig();
    this.getMedia();
  },
  methods: {
    showPGConfig(data){
      this.$store.commit('newTitle', this.APLang.media);
      this.$store.commit('showFooter', false);// footer if show
    },

    // get media list
    // get -> media
    /* page:       number, default 1
       per_page:   number, default 10
       exclude:    exclude some id
       orderby:    some way by order, options: [id,include,name,slug,include_slugs,term_group,description,count]
       order:      default asc, options: [asc,desc]
    */
    getMedia() {
      this.weui.loading(this.APLang.loading);

      apiUrl.get("media",{
        params: {
          page: this.pages.page,
          per_page: this.pages.per_page
        }
      }).then(res => {
        console.log(res);
        let newRes = res.data;
        this.mediaData = this.mediaData.concat(newRes);

        if (this.pages.page_count === 0) {
          this.pages.page_count = parseInt(res.headers['x-wp-totalpages']);
        }
        if(this.pages.page_count > 1) {
          this.loadMore = true;
        }else{
          this.loadMore = false;
        }

        this.weui.loading().hide();
      }).catch(err => {
        console.log("err",err.response);
        if(err.response) {
          if (err.response.status !== 200) {
            this.weui.topTips(err.response.data.message,3000);
          }
        }else{
          this.weui.topTips(this.APLang.unknownMistake,3000);
        }
        this.weui.loading().hide();
      });
    },

    // show more data
    getMore() {
      this.pages.page += 1;
      this.pages.page_count -= 1;
      this.loadMore = true;
      this.getMedia();
    }
  }
};
</script>