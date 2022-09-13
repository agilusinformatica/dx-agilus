<template>
  <div>
    <DxDataGrid 
      id="dataGrid"
      :data-source="empresas" 
      key-expr="Id" 
      :allow-column-reordering="true" 
      :column-auto-width="false" 
      @exporting="exportGrid" 
      :allow-column-resizing="true"
    >
      <DxColumnChooser :enabled="true" />
      <DxStateStoring
        :enabled="true"
        type="localStorage"
        storage-key="AgilusGrid"
      />
      <DxFilterRow :visible="true" />
      <DxSearchPanel :visible="true" />
      <DxGroupPanel :visible="true" />
      <DxSelection mode="single" />
      <DxExport :enabled="true" :formats="['xlsx', 'pdf']" />
      <DxSummary>
          <DxGroupItem
              summary-type="count"
          />
          <DxGroupItem
                column="LimiteUsuarios"
                summary-type="sum"
                :align-by-column="true"
                :show-in-group-footer="false"
            />
          <DxTotalItem
                column="LimiteUsuarios"
                summary-type="sum"
            />
      </DxSummary>
    </DxDataGrid>
  </div>
</template>

<script>
import 'devextreme/dist/css/dx.light.css';
import axios from 'axios'
import { DxDataGrid, DxFilterRow, DxSearchPanel, DxGroupPanel, DxSelection, DxSummary, DxGroupItem, DxTotalItem, DxExport, DxStateStoring, DxColumnChooser } from 'devextreme-vue/data-grid';
import { Workbook } from 'exceljs';
import saveAs from 'file-saver';
import { exportDataGrid } from 'devextreme/excel_exporter';
import { jsPDF } from 'jspdf';
import { exportDataGrid as exportDataGridToPdf} from 'devextreme/pdf_exporter';

export default {
    components: {
      DxDataGrid, DxFilterRow, DxSearchPanel, DxGroupPanel, DxSelection, DxSummary, DxGroupItem, DxTotalItem, DxExport, DxStateStoring, DxColumnChooser
    },
    data() {
        return {
          empresas: []
        }
    },
    mounted () {
      axios('http://localhost:55461/api/empresas/listar')
        .then(response => {
          this.empresas = response.data
        })
    },
    methods: {
        exportGrid(e) {
            if (e.format === 'xlsx') {
                const workbook = new Workbook(); 
                const worksheet = workbook.addWorksheet("Main sheet"); 
                exportDataGrid({ 
                    worksheet: worksheet, 
                    component: e.component,
                }).then(function() {
                    workbook.xlsx.writeBuffer().then(function(buffer) { 
                        saveAs(new Blob([buffer], { type: "application/octet-stream" }), "DataGrid.xlsx"); 
                    }); 
                }); 
                e.cancel = true;
            } 
            else if (e.format === 'pdf') {
                const doc = new jsPDF();
                exportDataGridToPdf({
                    jsPDFDocument: doc,
                    component: e.component,
                }).then(() => {
                    doc.save('DataGrid.pdf');
                });
            }
        }
    }
}
</script>

<style scoped>
</style>
