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
        <button @click.prevent="poseQuestionToStudents">Post</button>
      </div>
    </div>

    <div class="dialog" v-if="questionPosted">
      <div class="content" v-if="!answerReceived">
        <h4>Please submit your answer...</h4>
        <div v-for="result in studentAnswer" :key="result.page">
          <h3>Results</h3>
          <div
            v-if="
              JSON.stringify(result.answerContent.answerContent) ===
              JSON.stringify(questionContent.answers)
            "
          >
            <h2>Correct answer!</h2>
          </div>
          <div v-else>
            <h2 id="incorrect">Incorrect answer!</h2>
            <p>Correct answer was:</p>
            <div
              v-for="originalContent in questionContent"
              :key="originalContent.page"
            >
              <div v-for="incorrect in originalContent" :key="incorrect.page">
                <div v-if="incorrect.correct === true">
                  <li>{{ incorrect.text }}</li>
                </div>
              </div>
            </div>
          </div>
          <div class="scoreContent">
            <p>Your current score</p>
            <h2>{{ questionScore }}</h2>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import TimeInput from "../components/TimeInput";
import QuestionEntry from "../components/QuestionEntry";

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
    sendStudentResults() {
      this.studentResult = true;

      this.$socket.emit("teacher-marked-quiz", {
        studentResult: this.studentResult,
      });
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
      studentResult: false,
      answerReceived: false,
      correctAnswer: "",
    };
  },
  mounted() {},
};
</script>

<style scoped>
.dialog {
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

.dialog .content {
  padding: 30px;
  width: 70%;
  max-width: 500px;
  border: 1px solid rgba(0, 0, 0, 0.4);
  border-radius: 10px;
  box-shadow: -10px 10px 20px -10px rgba(0, 0, 0, 0.2);
  background: white;
  text-align: center;
}

#incorrect {
  color: red;
}

.scoreContent {
  padding: 5px;
  width: 30%;
  border: 1px solid rgba(0, 0, 0, 0.4);
  border-radius: 10px;
  box-shadow: -10px 10px 20px -10px rgba(0, 0, 0, 0.2);
  background: rgb(214, 246, 255);
  float: none;
  overflow: hidden;
}
</style>