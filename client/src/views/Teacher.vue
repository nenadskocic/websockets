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

    <div class="studentResults" v-if="studentFinished">
      <div class="content">
        <div class="resultsComp">
          <p>Time remaining</p>
          <p>{{ questionTime }}</p>
        </div>
        <div class="resultsComp">
          <p>% correct</p>
          <p>{{ questionTime }}</p>
        </div>
        <div class="resultsComp">
          <p>Total correct</p>
          <p>{{ questionTime }}</p>
        </div>
        <div class="resultsComp">
          <p>Total answers</p>
          <p>{{ questionTime }}</p>
        </div>

        <div class="resultsTable">
          <table>
            <thead>
              <tr>
                <th>Correct</th>
                <th>Incorrect</th>
              </tr>
            </thead>
            <tbody>
              <tr>
                <td></td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </div>
</template>
<script src="vue-grid-layout.umd.min.js"></script>
<script>
import TimeInput from "../components/TimeInput";
import QuestionEntry from "../components/QuestionEntry";
import VueGridLayout from "vue-grid-layout";

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

      this.sockets.subscribe("student-answer", (data) => {
        this.studentAnswer.push(data);

        this.studentFinished = true;
      });
    },
  },
  methods: {
    poseQuestionToStudents() {
      this.$socket.emit("teacher-new-question", {
        questionContent: this.questionContent, // Question and answers to question
        questionTime: this.questionTime, // Time to answer question
        questionScore: this.questionScore, // Score for getting question correct
        questionType: this.questionType, // Type of question (multiple, matching)
      });
      this.questionPosted = true;
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
  },
  data() {
    return {
      questionPosted: false,
      questionType: "",
      questionTime: 0,
      questionScore: 0,
      questionContent: null,
      students: [],
      studentAnswer: [],
      answerReceived: true,
      correctAnswer: "",
      studentFinished: false,
    };
  },
  watch: {
    time: {
      handler(questionTime) {
        if (questionTime > 0) {
          setTimeout(() => {
            this.time--;
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
}

.resultsTable {
  padding: 3.5%;
  width: 90%;
  height: 65%;
  border: 1px solid rgba(0, 0, 0, 0.4);
  border-radius: 10px;
  box-shadow: -10px 10px 20px -10px rgba(0, 0, 0, 0.2);
  background: rgb(206, 206, 206);
  margin: 15% 0 0 3.5%;
  padding-right: 0%;
}

table {
  border: 1px solid rgba(0, 0, 0, 0.4);
}
</style>