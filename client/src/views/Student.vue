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
            <div v-for="score in leaderboardScores" :key="score.page">
              <div class="row">
                <div class="col-10">{{ score.studentName }}</div>
                <div class="col-2">{{ score.scoreCounter }}</div>
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

      <div class="question" v-if="isQuestion">
        <div class="content">
          <div class="multiple" v-if="(questionType = 'multiple')">
            <div class="qContent">
              Time remaining
              <p id="time">{{ timer }}</p>
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

    <div class="result" v-if="isResult">
      <div class="content">
        <div class="qContent">
          Time remaining
          <p id="time">{{ timer }}</p>
        </div>
        <h3>Results</h3>
        <div
          v-if="
            JSON.stringify(activeAnswers) ===
            JSON.stringify(originalQuestion.questionContent.answers)
          "
        >
          <h2>Correct answer!</h2>
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
        </div>

        <div class="scoreContent">
          <p>Your current score</p>
          <h2>{{ scoreCounter }}</h2>
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
    },
  },
  data() {
    return {
      ready: false,
      result: false,
      question: false,
      studentName: null,
      statusText: "Waiting for a question...",
      leaderboardScores: [],
      userScores: { studentName: null, scoreCounter: null },
      activeQuestion: null,
      originalQuestion: [],
      scoreCounter: 0,
      timer: 0,
      originalAnswers: [],
      activeAnswers: [],
    };
  },
  computed: {
    isReady() {
      return this.ready;
    },
    isQuestion() {
      return this.question;
    },
    isResult() {
      return this.result;
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
          (this.activeAnswers = this.activeQuestion.questionContent.answers),
          (this.originalQuestion = _.cloneDeep(this.activeQuestion));
        this.originalAnswers = _.cloneDeep(
          this.activeQuestion.questionContent.answers
        );

        this.timer += parseInt(this.questionTime);
        this.question = true;
      });
      // You want to also "unsubscribe" when the user
      // disconnects, or the component is destroyed.
    },
    answerToTeacher() {
      if (_.isEqual(this.activeAnswers, this.originalAnswers)) {
        this.scoreCounter += parseInt(this.questionScore);
      } else {
        this.scoreCounter -= parseInt(this.questionScore);
      }

      this.result = true;

      this.userScores.studentName = this.studentName;
      this.userScores.scoreCounter = this.scoreCounter;
      this.leaderboardScores.push(this.userScores);
    },
  },
  watch: {
    timer: {
      handler(questionTime) {
        if (questionTime > 0) {
          setTimeout(() => {
            this.timer--;
          }, 1000);
        } else {
          this.question = false;
          this.result = false;
        }
      },
      immediate: true,
    },
  },
  destroyed() {},
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
  text-align: center;
}

.scoreContent h2 {
  text-align: center;
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