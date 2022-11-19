<script setup>
  import { storeToRefs } from "pinia";
  import { useLessonsStore } from "../stores/lessons";
  import { computed, onBeforeMount, ref } from "vue";

  import { groupByStudent } from "../util/group";
  import Papa from "papaparse";

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
  
  function downloadCSV(){
    var jsonData = "[ ";
    var currstr = document.getElementById("jour").outerText + '\n';
    var ind = currstr.indexOf('\n');
      currstr = currstr.slice(ind + 1);
      ind = currstr.indexOf('\n');
      currstr = currstr.slice(ind + 1);
      ind = currstr.indexOf('\n');
      currstr = currstr.slice(ind + 1);
    for (var i = 1; i < document.getElementById("jour").childElementCount; i++){      
      console.log(currstr);
      var ind = currstr.indexOf('\n');
      jsonData = jsonData + '{\n \"ФИО\" : \"' + currstr.slice(0,ind) + '\", \n';
      currstr = currstr.slice(ind + 1);
      ind = currstr.indexOf('\n');
      jsonData = jsonData + '\"Предмет\" : \"' + currstr.slice(0,ind) + '\", \n';
      currstr = currstr.slice(ind + 1);
      ind = currstr.indexOf('\n');
      jsonData = jsonData + '\"Оценки\" : \"' + currstr.slice(0,ind) + '\" },\n';
      currstr = currstr.slice(ind + 1);
    }
    jsonData = jsonData.slice(0,jsonData.lastIndexOf(',')) + ']';
    console.log(jsonData);
    var csv = Papa.unparse(jsonData, {	delimiter: ";"});
    console.log(csv);
    var blob = new Blob([csv], { type: 'charset=utf-8;data:text/csv;' });
    console.log(blob);
    var link = document.createElement("a");
    var filename = "Отчёт по школе";//  + selectedSchool;
    var url = URL.createObjectURL(blob);
    link.setAttribute("href", url);
    link.setAttribute("download", filename + '.csv');
    link.style.visibility = 'hidden';
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
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
  <button class = "btn btn-info ms-2" @click="onSelectClick(downloadCSV())">Скачать отчёт</button>
  <a href="" id="downloadlink" ref="csvfiledownload" hidden="true" download="mycsv.csv">download</a>
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
