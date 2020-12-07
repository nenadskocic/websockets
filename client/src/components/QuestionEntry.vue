<template>
  <div>
    <!--  
      Example:
        Which Prime Minister helped create the Canadian Charter of Rights and Freedoms?

        1. Pierre Trudeau (Correct = "true")
        2. Jean Chretien (Correct = "false")
        3. Stephen Harper (Correct = "false")
        4. Brian Mulroney (Correct = "false")
    -->
    <div v-show="type == 'multiple'">
      <h4>Multiple Choice</h4>

      <textarea
        placeholder="What is your question?"
        v-model="questionAsked"
      ></textarea>

      <h5>Answers</h5>

      <div class="row" v-for="(answer, idx) in answersAllowed" :key="idx">
        <div class="col-1">
          <input type="checkbox" v-model="answer.correct" @change="changed" />
        </div>
        <div class="col-10">
          <input
            type="text"
            placeholder="Answer"
            v-model="answer.text"
            @change="changed"
          />
        </div>
        <div class="col-1">
          <button @click.prevent="removeAnswer(idx)">Remove</button>
        </div>
      </div>

      <div class="row">
        <div class="col-12 text-right">
          <button v-on:click="addAnswer()">Add</button>
        </div>
      </div>
    </div>

    <div v-show="type == 'matching'">
      <h4>Matching Pairs</h4>

      <div class="row" v-for="(pair, index) in matchAllowed" :key="index">
        <div class="col-5">
          <input type="text" placeholder="A" min="0" v-model="pair.mQuestion" />
        </div>

        <div class="col-1">
          <div>=</div>
        </div>

        <div class="col-5">
          <input type="text" placeholder="B" min="0" v-model="pair.mAnswer" />
        </div>

        <div class="col-1">
          <button @click.prevent="removeMatch(index)">Remove</button>
        </div>
      </div>

      <div class="row">
        <div class="col-12 text-right">
          <button v-on:click="addMatchingQuestion()">Add</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "QuestionEntry",
  props: ["type"],
  methods: {
    /**
     * Multiple Choice
     * */
    addAnswer() {
      this.answersAllowed.push({ text: "", correct: false });
    },
    removeAnswer(idx) {
      this.answersAllowed.splice(idx, 1);
    },
    changed() {
      this.$emit("input", this.entry);
    },
    /**
     * Matching
     * */
    addMatchingQuestion() {
      this.matchAllowed.push({ mQuestion: "", mAnswer: "" });
    },
    removeMatch(index) {
      this.matchAllowed.splice(index, 1);
    },
  },
  computed: {
    /**
     * Multiple Choice
     * */
    entry() {
      let entryContent = {
        question: this.questionAsked,
        answers: this.answersAllowed,
      };
      return entryContent;
    },
    hasCorrectAnswer() {
      if (this.answersAllowed.length == 0) return;

      for (let x = 0; x < this.answersAllowed.length; x++) {
        if (this.answersAllowed[x].correct) return true;
      }
      return false;
    },
    ready() {
      return this.hasCorrectAnswer && this.questionAsked;
    },

    /**
     * Matching
     * */
  },
  data() {
    return {
      questionAsked: null,
      answersAllowed: [{ text: "", correct: false }],
      matchAllowed: [{ mQuestion: "", mAnswer: "" }],
    };
  },
};
</script>

<style scoped>
</style>