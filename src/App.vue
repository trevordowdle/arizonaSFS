<template>
  <div>
    <button @click="getSelectedRows()">Get Selected Rows</button>
    <br />
    <br />
    <ag-grid-vue style="width: 90%; height: 520px;"
                 class="ag-theme-balham"
                 :columnDefs="columnDefs"
                 :rowData="rowData"
                 :getRowHeight="getRH"
                 rowSelection="multiple"
                 rowMultiSelectWithClick="true"
                 @grid-ready="onGridReady"
                 >
    </ag-grid-vue>
  </div>
</template>
<script>
  import { cdata } from "./script/data";
  import {AgGridVue} from "ag-grid-vue";
  export default {
    name: "App",
    data() {
      return {
        columnDefs: null,
        rowData: null
      }
    },
    components: {
      AgGridVue
    },
    beforeMount() {
      let data = JSON.parse(cdata);

      data.map(row=>{
        row.links = [];
        row.manta ? row.links.push(row.manta) : '';
        row.bizapedia ? row.links.push(row.bizapedia) : '';
        row.buzzfile ? row.links.push(row.buzzfile) : '';
        row.facebook ? row.links.push(row.facebook) : '';
        if(row.pages){
          row.pages.map((page)=>{
            row.links.push(page.link);
          });
        }
        row.contacts = '';
        if(row.mailingInfo){
          if(row.mailingInfo.contacts.length){
            row.contacts = row.mailingInfo.contacts.reduce((list,contact)=>{
              list += contact[1] + ' ( ' + contact[0] + ' ) ';
              return list;
            },'');
            row.mailInfo = [row.mailingInfo.contacts[0][1]];
            row.mailInfo.push(row.mailingInfo.name);
            row.mailInfo.push(row.mailingInfo.address.replace('Address:',''));
          }
        }
      });
      this.columnDefs = [
          {headerName: 'Name', field: 'name', sortable: true, filter: true, /* checkboxSelection: true, */ autoHeight:true, cellClass: 'cell-wrap-text', width: 200 },
          {headerName: 'Tractors Owned', field: 'Tractors Owned', sortable: true },
          {headerName: 'Trailer Owned', field: 'Trailer Owned', sortable: true },
          {headerName: 'City', field: 'city', sortable: true, filter: true },
          {headerName: 'UCC Entries', field: 'uccEntries', sortable: true },
          {headerName: 'Mailer', field:'mailInfo', cellClass: 'cell-wrap-text', cellRenderer: function(params) {
             if(params.value){
                let thing = params.value.join('<br />');
                return thing;
             }
             return 'N/A';
          }},
          {headerName: 'Contacts', field: 'contacts', autoHeight:true, cellClass: 'cell-wrap-text'},
          {headerName: 'Links', field: 'links', width:300,
             cellRenderer: function(params) {
              let thing = params.value.map(link=>{
                return '<a href="'+link+'" target="_blank">'+link+'</a>';
              }).join('<br />');
              return thing;
            }
          },
      ];
      this.rowData = data;
    },
    methods: {
      getRH(params){
        let rh = params.node.rowHeight;
        let nh = params.data.links.length*27;
        let mi = 0; 
        if(params.data.mailInfo){
          mi = params.data.mailInfo.reduce((h,line)=>{
            if(!line){
              line = "N/A";
            }
          let ll = Math.ceil(line.length/19);
            h += (ll*27);
            return h
          },0);
        }
        nh = Math.max(nh,mi);
        
        if(nh > rh){
          return nh;
        }
        return rh;
      },
      onGridReady(params) {
        this.gridApi = params.api;
        this.columnApi = params.columnApi;
      },
      getSelectedRows() {
        const selectedNodes = this.gridApi.getSelectedNodes();
        const selectedData = selectedNodes.map( node => node.data );
        const selectedDataStringPresentation = selectedData.map( node => node.make + ' ' + node.model).join(', ');
        alert(`Selected nodes: ${selectedDataStringPresentation}`);
      }
    }
  };
</script>


<style lang="scss">
  @import "../node_modules/ag-grid-community/dist/styles/ag-grid.css";
  @import "../node_modules/ag-grid-community/dist/styles/ag-theme-balham.css";
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.cell-wrap-text {
    white-space: normal !important;
}
</style>
