<script setup>
  import { storeToRefs } from "pinia";
  import { useLessonsStore } from "../stores/lessons";
  import { computed, onBeforeMount, ref } from "vue";
  import xlsx from "json-as-xlsx"
  import { groupByStudent } from "../util/group";

  const lessonsStore = useLessonsStore();
  const { reports } = storeToRefs(lessonsStore);
  const { schools } = storeToRefs(lessonsStore);

  const reportsStored = computed(() => {
  return _(reports.value)
    .orderBy((x) => x[sortFiled.value])
    .value();
  });

  const reportByStudent = computed(() => {
    return reports.value
      ? groupByStudent(reports.value["school-journal"] || [])
      : [];
  });

  let sortFiled = ref("name");

  onBeforeMount(() => {
    // lessonsStore.fetchReports("1");
    lessonsStore.fetchSchools();
  });
</script>

<script>
  import { useLessonsStore } from "../stores/lessons";
  import ReportRow from "@/components/ReportRow.vue";
  import _ from "lodash";
  function proceeded(etuds,e0,e1){
    for (var i = 0; i < etuds.length; i++){
      if (etuds[i][0] == e0 && etuds[i][1] == e1){
        console.log(etuds[i][0] == e0 && etuds[i][1] == e1)
        return true;
      }        
      }
      console.log(false)
    return false;
  }
  function resEtudName(data,etud,j){
    for (var i = 0; i < data.length; i++){
      if (etud[j][0] == data[i]['students'] ){
        return data[i]['students_name'];
      }
    }
    return '';
  }
  function resDate(data,header,etuds,x,y){
    let date = header[y + 2]
    let etud = etuds[x][0]
    let suj = etuds[x][1]
    for (let i = 0; i < data.length; i++){
      if (data[i]['date'] == date && data[i]['students'] == etud && data[i]['lessons_name'] == suj)
        return i;
    }
    return -1;
  }
  async function fetchReports(subjectId){
    let url1 = `/api/journals/${subjectId}/school/`;
    let data1 = new Object();
    fetch(url1).then(response => 
        response.json().then(data => ({
            data: data,
            status: response.status
        })
    ).then(res => {
      data1 = res.data['school-journal']
      console.log(res.status, data1)
      let header = new Array(); // { label: "User", value: "user" }, // Top level data
      let headeralias = new Array();
      let headerfields = new Array();
      header.push('ФИО'); headeralias.push('user'); headerfields.push('students_name');
      header.push('Предмет'); headeralias.push('suj'); headerfields.push('lessons_name');
      for (var i = 0; i < data1.length ; i++){
        if (!header.includes(data1[i]['date'])){
          header.push(data1[i]['date']);
          headeralias.push('d' + i.toString());
          headerfields.push('mark');
        }
      }
      //forming a header
      var jsonData = "[ { \"sheet\": \"Etuds\", \n \"columns\": [\n"; 
      for (var i = 0; i < header.length; i++){        
        jsonData = jsonData + '{ \"label\": \"'+ header[i] +'\", \"value\": \"' + headeralias[i] + '\"},'
      }
      //forming etud list
      let etuds = new Array();
      let etudsrecs = new Array();
      for (var i = 0; i < data1.length; i++){
        if (!proceeded(etuds,data1[i]['students'], data1[i]['lessons_name'])){
          etuds.push([data1[i]['students'],data1[i]['lessons_name']]);
          etudsrecs.push([]);
          for (var j = 0; j < data1.length; j++){
            if (data1[j]['students'] == etuds[etuds.length - 1][0] && data1[j]['lessons_name'] == etuds[etuds.length - 1][1]){              
              etudsrecs[etuds.length - 1].push(j)
            }
          }
        }
      }
      //forming content
      jsonData = jsonData.slice(0,jsonData.lastIndexOf(',')) + '],\n\"content\": [';
      for (var i = 0; i < etuds.length; i++){
        var name = resEtudName(data1,etuds,i);
        jsonData = jsonData + '{ \"' + headeralias[0] + '\" : \"' + name + '\",'
        jsonData = jsonData + '  \"' + headeralias[1] + '\" : \"' + etuds[i][1]+ '\",'
        for (var j = 0; j < etudsrecs[i].length; j++){ 
          jsonData = jsonData + ' \"' + headeralias[j + 2] + '\" : \"'
          //система разрешения даты и оценки
          let h = resDate(data1,header,etuds,i,j);
          if (h != -1){
            jsonData = jsonData + data1[h]['mark'] + '\" ';
          }
          else {
            jsonData = jsonData + '\"';
          }
          if (j < etudsrecs[i].length - 1) jsonData = jsonData + ',';
        }
        jsonData = jsonData + '},'
      }
      jsonData = jsonData.slice(0,jsonData.lastIndexOf(',')) + ']}]';
      console.log(jsonData);
      let ectitle = document.getElementById("selececol").options[document.getElementById("selececol").selectedIndex].text;
      let settings = {
        fileName: "Отчёт для " + ectitle, // Name of the resulting spreadsheet
        extraLength: 3, // A bigger number means that columns will be wider
        writeMode: 'writeFile', // The available parameters are 'WriteFile' and 'write'. This setting is optional. Useful in such cases https://docs.sheetjs.com/docs/solutions/output#example-remote-file
        writeOptions: {}, // Style options from https://github.com/SheetJS/sheetjs#writing-options
        RTL: false, // Display the columns from right-to-left (the default value is false)
    }
    let data = JSON.parse(jsonData);
    xlsx(data, settings)
    }));
  }

  function downloadXLSX(){
    let ec = document.getElementById("selececol").options[document.getElementById("selececol").selectedIndex].value.toString(10);
    
    fetchReports(ec);
  }
  
  export default {
    data() {
      return {
        selectedSchool: "",
      };
    },
    methods: {
      onSelectClick(id) {
        console.log(id);
        console.log(this.reportsStored);
        console.log(this.reportsStored[0]);
        console.log(this.reportByStudent);
        this.lessonsStore.fetchReports(id);
      },
      onClick() {
      
      },
    },
  };
</script>

<template>
  <h2>Отчёт по школам</h2>

  <select v-model="selectedSchool" id = 'selececol'>
    <option disabled value="">Выберите один из вариантов</option>
    <option v-for="s in schools" :value="s.id">
      {{ s.title }}
    </option>
  </select>
  <button class = "btn btn-info ms-2" @click="onSelectClick(selectedSchool)">Показать</button>
  <button class = "btn btn-info ms-2" @click="onClick(downloadXLSX())">Скачать отчёт</button>
  <div id = "jour">
    <div id = 'head' class="journal_header">
      <span id = 'head0'>ФИО</span>
      <span id = 'head1'>Предмет</span>
      <span id = 'head3'>Оценки</span>
    </div>
      <ReportRow
        v-for="(r, index) in reportByStudent"
        :number = "index"
        :students_name = "r.students_name"
        :lessons_name = "r.lessons_name"
        :date = "r.date"
        :mark = "r.marks.join(' ')"
      />
    </div>
</template>

<style scoped>
.journal_header {
  display: flex;
  justify-content: flex-start;
  margin-top: 20px;
}

.journal_header > span {
  width: calc(100% / 3);
  border: 1px solid black;
  border-right: none;
  padding: 5px;
}

.journal_header > span:last-child {
  border-right: 1px solid black;
}
</style>
