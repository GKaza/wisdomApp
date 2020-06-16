<template>
  <div id="question-container">
    <h3 class="question">
      <span v-html="question"></span>
    </h3>
    <div class="answers-container">
      <button
        v-for="answer in mixAnswers"
        :key="answer"
        class="btn answer"
        @click="answered(answer)"
      >
        <span v-html="answer"></span>
      </button>
    </div>
  </div>
</template>

<script>
export default {
  name: "Question",
  props: ["question", "correct", "incorrect"],
  methods: {
    answered(val) {
      this.$emit("answered", this.isCorrect(val));
    },
    isCorrect(ans) {
      return ans === this.correct;
    }
  },
  computed: {
    mixAnswers() {
      let random = this.incorrect.slice();
      random.splice(Math.floor(Math.random() * 4), 0, this.correct);
      return random;
    }
  }
};
</script>

<style scoped>
.answers-container {
  padding: 30px;
  display: flex;
  flex-direction: column;
}
.answer {
  min-width: fit-content;
  width: 300px;
  margin: 10px auto;
}
</style>
