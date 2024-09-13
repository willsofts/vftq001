<template>
<div id="searchpanel" class="panel-body search-panel">
    <div class="row row-height">
        <div class="col-height col-md-2">
            <label>{{ labels.userid_label }}</label>
            <input type="text" v-model="localData.userid" class="form-control input-md" maxlength="50" />
        </div>
        <div class="col-height col-md-2">
            <label>{{ labels.programid_label }}</label>
            <input type="text" v-model="localData.progid" class="form-control input-md" maxlength="50" />
        </div>
        <div class="col-height col-md-2">
            <label for="datefroms">{{ labels.datefrom_label }}</label>
            <InputDate v-model="localData.datefrom" id="datefroms" /> 
        </div>
        <div class="col-height col-md-2">
            <label for="datetos">{{ labels.dateto_label }}</label>
            <InputDate v-model="localData.dateto" id="datetos" /> 
        </div>
        <div class="col-height col-md">
            <br/>
            <button @click="searchClick" class="btn btn-dark btn-sm btn-ctrl"><i class="fa fa-search fa-btn-icon" aria-hidden="true"></i>{{ labels.search_button }}</button>
            <button @click="resetClick" class="btn btn-dark btn-sm btn-ctrl"><i class="fa fa-refresh fa-btn-icon" aria-hidden="true"></i>{{ labels.reset_button }}</button>
        </div>
    </div>
    <div id="listpanel" class="table-responsive fa-list-panel">
        <DataTable ref="dataTable" :settings="tableSettings" :labels="labels" :dataset="dataset" @data-sort="dataSorted" />
        <DataPaging ref="dataPaging" :settings="pagingSettings" @page-select="pageSelected" />
    </div>
</div>
</template>
<script>
import { ref } from 'vue';
import $ from "jquery";
import { DEFAULT_CONTENT_TYPE, getApiUrl }  from '@willsofts/will-app';
import { startWaiting, stopWaiting, submitFailure, serializeParameters }  from '@willsofts/will-app'
import { Paging, Utilities } from "@willsofts/will-app";
import { InputDate, DataTable, DataPaging } from '@willsofts/will-control';

const defaultData = {
  userid: '',
  progid: "",
  datefrom: Utilities.getDateNow(),
  dateto: Utilities.getDateNow(),
};

const tableSettings = {
    sequence: { label: "seqno_label" },
    columns: [
        {name: "curtime", type: "DATETIME", sorter: "curtime", label: "curtime_headerlabel", css: "text-center" },
        {name: "username", type: "STRING", sorter: "username", label: "userid_headerlabel", css: "text-center" },
        {name: "progid", type: "STRING", sorter: "progid", label: "programid_headerlabel" },
        {name: "progname", type: "STRING", sorter: false, label: "progname_headerlabel" },
        {name: "action", type: "STRING", sorter: "action", label: "action_headerlabel" },
        {name: "remark", type: "STRING", sorter: false, label: "remark_headerlabel" }
    ],        
};

export default {
  components: {
    InputDate, DataTable, DataPaging
  },
  props: {
    labels: Object,
    formData: Object,
    dataCategory: Object
  },
  setup(props) {
    const localData = ref({ ...defaultData, ...props.formData });
    let paging = new Paging();
    let pagingSettings = paging.setting;
    let filters = {};
    const dataset = ref({});
    return { localData, tableSettings, pagingSettings, paging, filters, dataset };
  },
  methods: {
    reset(newData) {
      if(newData) this.localData = {...newData};
    },
    getPagingOptions(settings) {
      if(!settings) settings = this.pagingSettings;
      return {page: settings.page, limit: settings.limit, rowsPerPage: settings.rowsPerPage, orderBy: settings.orderBy?settings.orderBy:"", orderDir: settings.orderDir?settings.orderDir:"" };
    },
    resetClick() {
      this.localData = {...defaultData};
    },
    searchClick() {
      this.filters = {...this.localData};
      this.resetFilters();
      this.search();
    },
    resetFilters() {
      this.pagingSettings.page = 1;
      this.pagingSettings.orderBy = "";
      this.pagingSettings.orderDir = "";
    },
    search(ensurePaging=false) {
      if(ensurePaging) {
        if(this.pagingSettings.rows <= 1 && this.pagingSettings.page > 1) {
          this.pagingSettings.page = this.pagingSettings.page - 1;
        }
      }
      console.log("search: pagingSettings",this.pagingSettings);
      let options = this.getPagingOptions();
      this.collecting(options,this.filters);
    },
    collecting(options,criterias) {
      console.log("collecting: options",options,", criteria",criterias);
      let jsondata = Object.assign({ajax: true},options);
      //Object.assign(jsondata,criterias);
      let formdata = serializeParameters(jsondata,criterias);
      startWaiting();
      $.ajax({
        url: getApiUrl()+"/api/sftq001/collect",
        data: formdata.jsondata,
        headers : formdata.headers,
        type: "POST",
        dataType: "json",
        contentType: DEFAULT_CONTENT_TYPE,
        error : function(transport,status,errorThrown) {
          console.error("error: status",status,"errorThrown",errorThrown);
          submitFailure(transport,status,errorThrown);
        },
        success: (data) => {
          stopWaiting();
          console.log("collecting: success",data);
          if(data.body) {
            this.$refs.dataTable.reset(data.body);
            this.$refs.dataPaging.reset(data.body?.offsets);
            this.pagingSettings.rows = data.body?.rows?.length;
          }
        }
      });	
    },
    pageSelected(item) {
      console.log("page selected:",item);
      this.pagingSettings.page = item.page;
      let options = this.getPagingOptions();
      this.collecting(options,this.filters);
    },
    dataSorted(sorter,direction) {
      console.log("dataSorted",sorter,"direction",direction);
      this.pagingSettings.orderBy = sorter;
      this.pagingSettings.orderDir = direction;
      let options = this.getPagingOptions();
      this.collecting(options,this.filters);
    },
  }
};
</script>
