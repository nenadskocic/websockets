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
          <input type="text" v-model="studentName" placeholder="John Smith">
          <button @click.prevent="startSession">Start</button>
        </div>
      </div>

      <div class="question" v-if="this.activeQuestion">
        <div class="content">
          A question came in!
        </div>
      </div>
    </form>
  </div>
</template>

<script>
export default {
  sockets: {
    connect() {
      console.log('Student.vue: socket connected');
    }
  },
  data() {
    return {
      ready: false,
      studentName: null,
      statusText: "Waiting for a question...",
      leaderboardScores: [],
      activeQuestion: null,
    }
  },
  computed: {
    isReady() {
      return this.ready;
    }
  },
  methods: {
    startSession() {
      this.ready = true;
      this.$socket.emit('student-registered', {studentName: this.studentName});

      this.sockets.subscribe('teacher-new-question', (data) => {
        this.activeQuestion = data.question;
      });

      // You want to also "unsubscribe" when the user 
      // disconnects, or the component is destroyed.

    }
  },
  destroyed() {
    // this.sockets.unsubscribe('teacher-new-question');
  }
}
</script>

<style scoped>

.question,
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

.question .content,
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

.dialog h4 {
  margin-top: 0;
  margin-bottom: 10px;
}

.dialog input {
  margin: 10px 0;
}

</style>