<!-- App.vue -->
<template>
  <div id="fswaitlayer" class="fa fa-spinner fa-spin"></div>
  <div class="pt-page pt-page-current pt-page-controller search-pager">
    <PageHeader ref="pageHeader" :labels="labels" pid="vftq001" version="1.0.0" showLanguage="true" @language-changed="changeLanguage" />
    <SearchForm ref="searchForm" :labels="labels" />
  </div>
</template>
<script>
import { ref } from 'vue';
import { PageHeader } from '@willsofts/will-control';
import SearchForm from '@/components/SearchForm.vue';
import { startApplication, getLabelModel, getDefaultLanguage, setDefaultLanguage } from "@willsofts/will-app";

export default {
  components: {
    PageHeader, SearchForm
  },
  setup() {
    let labels = ref(getLabelModel());
    return { labels };
  },
  mounted() {
    console.log("App: mounted ...");
    this.$nextTick(() => {
      //ensure ui completed then invoke startApplication 
      startApplication("vftq001",(data) => {
        this.messagingHandler(data);
        this.$refs.pageHeader.changeLanguage(getDefaultLanguage());
      });
      //try to find out parameters from url
      const searchParams = new URLSearchParams(window.location.href);
      console.log("param: authtoken=",searchParams.get("authtoken"),", language=",searchParams.get("language"));      
    });
  },
  methods: {
    messagingHandler(data) {
      console.log("messagingHandler: data",data); 
    },
    changeLanguage(lang) {
      setDefaultLanguage(lang);
      let labelModel = getLabelModel(lang);
      this.labels = labelModel;
    },
  }
};
</script>
