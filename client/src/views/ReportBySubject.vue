<script setup>
import { storeToRefs } from "pinia";
import { useTeacherStore } from "../stores/teacherStore";
import { computed, onBeforeMount, ref } from "vue";

import { groupByStudent } from "../util/group";
import Papa from "papaparse";
const teacherStore = useTeacherStore();
const { subjects, lessons, subjectStudents, report } =
  storeToRefs(teacherStore);

const sortedStudents = computed(() => {
  return subjectStudents !== null
    ? // ascending sort
      subjectStudents.value.sort((s1, s2) => (s1.surname > s2.surname ? 1 : -1))
    : null;
});

const sortedLessons = computed(() => {
  return lessons !== null
    ? // ascending sort
      lessons.value.sort((l1, l2) => (l1.date > l2.date ? 1 : -1))
    : null;
});

onBeforeMount(() => {
  teacherStore.fetchTeacherSubjects(1);
});
</script>

<script>
import { useLessonsStore } from "../stores/lessons";
import ReportRow from "@/components/ReportRow.vue";
import _ from "lodash";
import { useTeacherStore } from "../stores/teacherStore";
import { formatDate } from "../util/formatDate";

function downloadCSV(){
    var jsonData = "[ ";
    var header = new Array();
    var currstr = "ФИО\n" + document.getElementById("jour").outerText + '\n';
    for (var i = 0; i < document.getElementById("head").childElementCount; i++){
      var ind = currstr.indexOf('\n');
      header.push(currstr.slice(0,ind));
      currstr = currstr.slice(ind + 1);
    }
    console.log(currstr);
    console.log(document.getElementById("jour").childElementCount);
    for (var i = 1; i < document.getElementById("jour").childElementCount; i++){
      console.log(document.getElementById("jour").childElementCount);    
      jsonData = jsonData + '{'
      for (var j = 0; j < header.length; j++){        
        var ind = currstr.indexOf('\n');
        jsonData = jsonData + '\n \"'+ header[j] +'\" : \"' + currstr.slice(0, ind) + '\"';
        if (j < header.length - 1) jsonData = jsonData + ',';
        currstr = currstr.slice(ind + 1);
        
      }
      jsonData = jsonData + '},'
    }
    jsonData = jsonData.slice(0,jsonData.lastIndexOf(',')) + ']';
    console.log(jsonData);
    var csv = Papa.unparse(jsonData, {	delimiter: ";"});
    console.log(csv);
    var blob = new Blob([csv], { type: 'charset=utf-8;data:text/csv;' });
    console.log(blob);
    var link = document.createElement("a");
    var filename = "Отчёт по предмету";//  + selectedSchool;
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
      selectedSubject: "",
    };
  },
  methods: {
    onSelectClick(id) {
      this.teacherStore.fetchReportBySubjectId(id);
      this.teacherStore.fetchStudentsBySubjectId(id);
      this.teacherStore.fetchSubjectLessons(id);
    },
  },
};
</script>

<template>
  <div class="report_wrapper">
    <h2>Отчёт по предметам</h2>
    <select v-model="selectedSubject">
      <option disabled value="">Выберите один из вариантов</option>
      <option v-for="s in subjects" :value="s.id">
        {{ `${s.name} (${s.level})` }}
      </option>
    </select>
    
    <button class = "btn btn-info ms-2" @click="onSelectClick(selectedSubject)">Показать</button>
    <button class = "btn btn-info ms-2" @click="onSelectClick(downloadCSV())">Скачать отчёт</button>
    <a href="" id="downloadlink" ref="csvfiledownload" hidden="true" download="mycsv.csv">download</a>
      <div id = "jour" v-if="!Boolean(lessons) || !Boolean(subjectStudents)">
        Выберите предмет
      </div>

      <div id = "jour" v-else>
        <div id = "head" class="journal_header">
          <span></span>
          <span v-for="lesson in sortedLessons">{{
            formatDate(lesson.date)
          }}</span>
        </div>

        <div class="journal_row" v-for="s in sortedStudents">
          <span>{{ `${s.surname} ${s.name} ${s.patr}` }}</span>
          <span v-for="lesson in sortedLessons">{{
            report[
              report.findIndex(
                (record) =>
                  record.lessons === lesson.id &&
                  record.students_name === `${s.surname} ${s.name} ${s.patr}`
              )
            ]?.mark || ""
          }}</span>
      </div>
      </div>

  </div>
</template>

<style scoped>
.report_wrapper {
  display: flex;
  flex-direction: column;
  gap: 20px;
  align-items: flex-start;
}
.journal_header {
  display: flex;
  justify-content: flex-start;
  margin-top: 20px;
}

.journal_header > span {
  border: 1px solid black;
  border-right: none;
  padding: 5px;
  width: 100px;
}
.journal_header > span:first-child {
  width: 300px;
}
.journal_header > span:last-child {
  border-right: 1px solid black;
}

.journal_row {
  display: flex;
  justify-content: flex-start;
}

.journal_row > span {
  border: 1px solid black;
  border-top: none;
  border-right: none;
  padding: 5px;
  width: 100px;

  display: flex;
  justify-content: center;
}

.journal_row > span:first-child {
  width: 300px;
  display: flex;
  justify-content: flex-start;
}

.journal_row > span:last-child {
  border-right: 1px solid black;
}
</style>
