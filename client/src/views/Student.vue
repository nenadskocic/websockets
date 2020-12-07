<template>
  <div class="student">
    <h1>Student</h1>
    <p>Answer questions posed by the teacher</p>

    <form>
      <div class="row">
        <div class="col-6">
          <h4>Status</h4>
          <div>{{ statusText }}</div>
        </div>
      </div>

      <div class="row">
        <div class="col-6">
          <h4>Leaderboard</h4>
          <div v-if="leaderboardScores.length == 0">No scores recorded yet</div>
          <div v-else>
            <div v-for="score in leaderboardScores" :key="score.name">
              <div class="row">
                <div class="col-10">{{ score.name }}</div>
                <div class="col-2">{{ score.score }}</div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <div class="dialog" v-if="!isReady">
        <div class="content">
          <h4>Please enter your name</h4>
          <input type="text" v-model="studentName" placeholder="John Smith" />
          <button @click.prevent="startSession">Start</button>
        </div>
      </div>

      <div class="question" v-if="this.activeQuestion">
        <div class="content">
          <div class="multiple" v-if="(questionType = 'multiple')">
            <div class="qContent">
              Time remaining
              <p id="time">{{ time }}</p>
            </div>
            <h3>Multiple Choice</h3>
            <p id="score">{{ questionScore }} Points</p>
            <div
              class="answers"
              v-for="element in activeQuestion"
              :key="element.page"
            >
              <h2>{{ element.question }}</h2>
              <div v-for="anws in element.answers" :key="anws.page">
                <ul class="checkbox-grid">
                  <li>
                    <input type="checkbox" v-model="anws.correct" />
                    <label :for="anws.text">{{ anws.text }}</label>
                  </li>
                </ul>
              </div>
            </div>
            <div class="submitAnswer">
              <button @click.prevent="answerToTeacher" class="submitBtn">
                Submit answer
              </button>
            </div>
          </div>
          <div v-else>
            <h4>Matching Pair</h4>
          </div>
        </div>
      </div>
    </form>

    <div class="result" v-if="studentAnswered">
      <div class="content">
        <div class="qContent">
          Time remaining
          <p id="time">{{ time }}</p>
        </div>
        <h3>Results</h3>
        <div v-for="result in studentAnswer" :key="result.page">
          <div
            v-if="
              JSON.stringify(result.answerContent.answerContent) ===
              JSON.stringify(originalQuestion.questionContent.answers)
            "
          >
            <h2>Correct answer!</h2>
            <div class="scoreContent">
              <p>Your current score</p>
              <h2>{{ parseInt(score + questionScore) }}</h2>
            </div>
          </div>
          <div v-else>
            <h2 id="incorrect">Incorrect answer!</h2>
            <h3>Correct answer was:</h3>
            <div
              v-for="originalContent in originalQuestion.questionContent"
              :key="originalContent.page"
            >
              <div v-for="incorrect in originalContent" :key="incorrect.page">
                <div v-if="incorrect.correct === true" id="incorrectList">
                  <li>{{ incorrect.text }}</li>
                </div>
              </div>
            </div>
            <div class="scoreContent">
              <p>Your current score</p>
              <h4>{{ parseInt(score + questionScore) }}</h4>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import _ from "lodash";

export default {
  sockets: {
    connect() {
      console.log("Student.vue: socket connected");

      this.sockets.subscribe("student-answer", (data) => {
        this.studentAnswer.push(data);
      });
    },
  },
  data() {
    return {
      ready: false,
      studentName: null,
      statusText: "Waiting for a question...",
      leaderboardScores: [],
      activeQuestion: null,
      time: 30,
      studentAnswered: false,
      studentAnswer: [],
      originalQuestion: [],
      score: 0,
    };
  },
  computed: {
    isReady() {
      return this.ready;
    },
  },
  methods: {
    startSession() {
      this.ready = true;
      this.$socket.emit("student-registered", {
        studentName: this.studentName,
      });

      this.sockets.subscribe("teacher-new-question", (data) => {
        (this.activeQuestion = data.question),
          (this.questionTime = data.question.questionTime),
          (this.questionScore = data.question.questionScore),
          (this.questionType = data.question.questionType),
          (this.originalQuestion = _.cloneDeep(this.activeQuestion));
      });
      // You want to also "unsubscribe" when the user
      // disconnects, or the component is destroyed.
    },
    answerToTeacher() {
      this.$socket.emit("student-answer", {
        answerContent: this.activeQuestion.questionContent.answers,
      });
      this.studentAnswered = true;
    },
    increaseScore() {
      this.score = parseInt(this.score + this.questionScore);
    },
    decreaseScore() {
      this.score = parseInt(this.score - this.questionScore);
    },
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
  destroyed() {
    //this.sockets.unsubscribe("teacher-new-question");
  },
};
</script>

<style scoped>
.question,
.dialog,
.result {
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

.question .content,
.dialog .content,
.result .content {
  padding: 30px;
  width: 70%;
  height: 50%;
  max-width: 800px;
  max-height: 1080px;
  border: 1px solid rgba(0, 0, 0, 0.4);
  border-radius: 10px;
  box-shadow: -10px 10px 20px -10px rgba(0, 0, 0, 0.2);
  background: rgb(206, 206, 206);
  text-align: center;
}

.dialog .content {
  padding: 30px;
  width: 70%;
  height: 20%;
  border: 1px solid rgba(0, 0, 0, 0.4);
  border-radius: 10px;
  box-shadow: -10px 10px 20px -10px rgba(0, 0, 0, 0.2);
  background: rgb(206, 206, 206);
  text-align: center;
}

.dialog h4 {
  margin-top: 0;
  margin-bottom: 10px;
}

.dialog input {
  margin: 10px 0;
}

.multiple h2,
h3,
.answers {
  text-align: left;
}

.result h2,
h3 {
  text-align: left;
}
.multiple label {
  font-size: 18px;
  padding-left: 0.5em;
  text-align: left;
}

.multiple li {
  float: left;
  width: 25%;
}
.multiple ul {
  list-style-type: none;
}

.submitBtn {
  margin: 60px 0 0 0;
}

.submitAnswer {
  position: relatve;
}

.qContent {
  padding: 5px;
  width: 20%;
  border: 1px solid rgba(0, 0, 0, 0.4);
  border-radius: 10px;
  box-shadow: -10px 10px 20px -10px rgba(0, 0, 0, 0.2);
  background: rgb(214, 246, 255);
  position: static;
  float: right;
  overflow: hidden;
}
.scoreContent {
  padding: 5px;
  width: 20%;
  border: 1px solid rgba(0, 0, 0, 0.4);
  border-radius: 10px;
  box-shadow: -10px 10px 20px -10px rgba(0, 0, 0, 0.2);
  background: rgb(214, 246, 255);
  position: static;
  float: right;
  top: 0;
  overflow: hidden;
}

#score {
  font-size: 20px;
  text-align: left;
  color: red;
  font-weight: bold;
}
#time {
  font-size: 20px;
  color: red;
  font-weight: bold;
}

#incorrect {
  color: red;
}

#incorrectList li {
  list-style-type: none;
  text-align: left;
}
</style>