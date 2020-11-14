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

      <div class="result" v-if="studentFlag">
        <h2>Please see teacher for your result!</h2>
      </div>
    </form>
  </div>
</template>

<script>
export default {
  sockets: {
    connect() {
      console.log("Student.vue: socket connected");
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
      studentFlag: false,
      studentResult: false,
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
          (this.questionType = data.question.questionType);
      });

      this.sockets.subscribe("teacher-marked-quiz", (data) => {
        this.studentResult = data.studentResult;
      });
      // You want to also "unsubscribe" when the user
      // disconnects, or the component is destroyed.
    },
    answerToTeacher() {
      this.$socket.emit("student-answer", {
        answerContent: this.activeQuestion.questionContent.answers,
      });
      this.studentFlag = true;
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
    // this.sockets.unsubscribe('teacher-new-question');
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
  height: 40%;
  max-width: 800px;
  max-height: 1000px;
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
  width: 10%;
  border: 1px solid rgba(0, 0, 0, 0.4);
  border-radius: 10px;
  box-shadow: -10px 10px 20px -10px rgba(0, 0, 0, 0.2);
  background: rgb(214, 246, 255);
  position: static;
  float: right;
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
</style>