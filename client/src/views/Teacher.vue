<template>
  <div class="teacher">
    <h1>Teacher</h1>
    <p>Post a question to students</p>

    <div class="row">
      <div class="col-6">
        <select v-model="questionType">
          <option value="">Select a question type</option>
          <option value="multiple">Multiple Choice</option>
          <option value="matching">Matching Pairs</option>
        </select>
      </div>
      <div class="col-4">
        <input
          type="number"
          min="0"
          placeholder="Score"
          v-model="questionScore"
        />
      </div>
      <div class="col-2">
        <TimeInput
          min="10"
          max="90"
          step="5"
          placeholder="Time"
          v-model="questionTime"
          @input="questionTime = $event"
        />
      </div>
    </div>

    <div class="row">
      <div class="col-12">
        <QuestionEntry
          v-bind:type="questionType"
          @input="questionContent = $event"
        />
      </div>
    </div>

    <div class="row" v-if="isQuestionReady">
      <div class="col-12">
        <button @click.prevent="poseQuestionToStudents">
          Send to students
        </button>
      </div>
    </div>

      <div class="studentResults" v-if="questionPosted">
        <div class="content">
          <div class="resultsComp">
            <p>Time remaining</p>
            <p>{{ this.timer }}</p>
          </div>
          <div class="resultsComp">
            <p>% correct</p>
            <p>{{ this.percentageCorrect }}</p>
          </div>
          <div class="resultsComp">
            <p>Total correct</p>
            <p>{{ this.totalCorrect }}</p>
          </div>
          <div class="resultsComp">
            <p>Total Answers</p>
            <p>{{ this.totalAnswers }}</p>
          </div>

          <div class="row" v-for="statsL in teacherStats" :key="statsL.page">
            <div class="col-6">
              <h2>Correct</h2>
              <div v-if="isStudCorrect">
                <div class="col-6">{{ statsL.teacherStats.studentName }}</div>
                <div class="col-2">{{ statsL.teacherStats.scoreCounter }}</div>
              </div>
              <div v-else>
                <div class="col-6">{{ statsL.teacherStats.studentName }}</div>
                <div class="col-2">{{ statsL.teacherStats.scoreCounter }}</div>
              </div>
            </div>
          </div>

          <div class="submitAnswer" v-if="isReadyNext">
            <button v-on:click="askAnother">Ask another question</button>
          </div>
        </div>
      </div>
  </div>
</template>
<script>
import TimeInput from "../components/TimeInput";
import QuestionEntry from "../components/QuestionEntry";
import _ from "lodash";

export default {
  components: {
    TimeInput,
    QuestionEntry,
  },
  sockets: {
    connect() {
      console.log("Teacher.vue: socket connected");

      this.sockets.subscribe("student-registered", (data) => {
        this.students.push(data);
      });
    },
  },
  methods: {
    poseQuestionToStudents() {
      this.$socket.emit("teacher-new-question", {
        questionContent: this.questionContent,
        questionTime: this.questionTime,
        questionScore: this.questionScore,
        questionType: this.questionType,
      });
      this.questionPosted = true;

      this.sockets.subscribe("teacher-result-display", (data) => {
        this.teacherStats.push(data);
        this.studentName = data.teacherStats.studentName;
        this.scoreCounter = data.teacherStats.scoreCounter;
        this.totalAnswers = data.teacherStats.totalAnswers;
        this.totalCorrect = data.teacherStats.totalCorrect;
        this.percentageCorrect = data.teacherStats.percentageCorrect;
        this.isStudCorrect = data.teacherStats.isStudCorrect;
        this.timer = parseInt(data.teacherStats.timer);
      });
    },
    askAnother() {
      this.questionPosted = false;
      this.readyNext = false;
      this.timer = 0;
    },
  },
  computed: {
    isQuestionReady() {
      return (
        this.questionScore > 0 &&
        this.questionTime > 0 &&
        this.questionType != ""
      );
    },
    isReadyNext() {
      return this.readyNext;
    },
  },
  data() {
    return {
      questionPosted: false,
      questionType: "",
      questionTime: 0,
      questionScore: 0,
      questionContent: null,
      students: [],
      teacherStats: [],
      timer: 0,
      readyNext: false,
    };
  },
  watch: {
    timer: {
      handler(questionTime) {
        if (questionTime > 0) {
          setTimeout(() => {
            this.timer--;

            if (this.timer == 0) {
              this.readyNext = true;
            }
          }, 1000);
        }
      },
      immediate: true,
    },
  },
  mounted() {},
};
</script>

<style scoped>
.studentResults {
  position: absolute;
  left: 0;
  right: 0;
  bottom: 0;
  top: 0;
  background: rgba(255, 255, 255, 0.9);
  display: flex;
  justify-content: center;
  align-items: center;
}

.studentResults .content {
  padding: 30px;
  width: 70%;
  height: 70%;
  border: 1px solid rgba(0, 0, 0, 0.4);
  border-radius: 10px;
  box-shadow: -10px 10px 20px -10px rgba(0, 0, 0, 0.2);
  background: rgb(206, 206, 206);
  text-align: center;
}
.resultsComp {
  padding: 5px;
  width: 20%;
  border: 1px solid rgba(0, 0, 0, 0.4);
  border-radius: 10px;
  box-shadow: -10px 10px 20px -10px rgba(0, 0, 0, 0.2);
  background: rgb(214, 246, 255);
  position: static;
  float: right;
  overflow: hidden;
  text-align: center;
  margin-right: 3%;
  margin-bottom: 5%;
}

.submitAnswer {
  margin-top: 30%;
}
</style>