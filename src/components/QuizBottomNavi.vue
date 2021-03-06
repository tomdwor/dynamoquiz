<template>
  <v-footer padless fixed class="grey lighten-3">
    <v-container style="max-width: 1200px;">
      <v-layout>
        <v-row
          v-if="quizState === 'start'"
          class="mx-3"
          justify="center"
          no-gutters
        >
          <v-col class="py-4 text-left" cols="6">
            <v-btn class="navi-btn" large v-on:click="exitQuiz">Cancel</v-btn>
          </v-col>
          <v-col class="py-4 text-right" cols="6">
            <v-btn
              id="start-quiz-btn"
              class="navi-btn"
              large
              color="primary"
              v-on:click="startQuiz"
              >Start quiz</v-btn
            >
          </v-col>
        </v-row>
        <v-row
          v-if="quizState === 'ask' || quizState === 'check'"
          class="mx-3"
          justify="center"
          no-gutters
        >
          <v-col class="pt-3 text-center" cols="6">
            <div style="width: calc(100% - 12px);">
              <span style="font-weight: bold;"
                >{{ quizStatistics.correctAnswersNumber }} /
                {{ quizStatistics.answersNumber }} of
                {{ quizStatistics.questionsNumber }}</span
              >
              <v-progress-linear
                v-model="quizStatistics.progress"
                color="blue-grey"
                height="25"
                reactive
                style="width: 100%;"
                value="25"
              >
                <template v-slot="{ value }">
                  <strong>{{ Math.ceil(value) }}%</strong>
                </template>
              </v-progress-linear>
            </div>
          </v-col>
          <v-col class="py-4 text-right" cols="6">
            <v-btn
              v-if="quizState === 'ask'"
              id="check-answer-btn"
              class="navi-btn"
              large
              color="primary"
              v-on:click="checkAnswer"
              :disabled="isDisabledCheckBtn"
              >CHECK</v-btn
            >
            <v-btn
              v-if="quizState === 'check'"
              id="next-question-btn"
              class="navi-btn"
              large
              color="primary"
              v-on:click="nextQuestion"
              >NEXT</v-btn
            >
          </v-col>
        </v-row>
        <v-row
          v-if="quizState === 'review'"
          class="mx-3"
          justify="center"
          no-gutters
        >
          <v-col class="py-4 text-left" cols="6">
            <v-btn class="navi-btn" large v-on:click="repeatQuiz"
              >Repeat quiz</v-btn
            >
          </v-col>
          <v-col class="py-4 text-right" cols="6">
            <v-btn class="navi-btn" large color="primary" v-on:click="exitQuiz"
              >Finish quiz</v-btn
            >
          </v-col>
        </v-row>
      </v-layout>
    </v-container>
  </v-footer>
</template>

<script>
import store from "@/store/index";
import QuizHandler from "@/domain/quiz/QuizHandler.js";

export default {
  name: "QuizBottomNavi",
  store,
  data() {
    let quizHandler = new QuizHandler(store);
    return {
      quizHandler
    };
  },
  computed: {
    quizState: function() {
      return this.quizHandler.getQuizState();
    },
    quizStatistics: function() {
      return this.quizHandler.getQuizStatistics();
    },
    isDisabledCheckBtn: function() {
      let currentQuestion = this.quizHandler.getCurrentQuestion();
      let isDisabled = false;

      if ("single" === currentQuestion.type) {
        isDisabled = this.$store.state.userSingleChoice === null;
      }
      if ("multi" === currentQuestion.type) {
        let userMultiChoice = this.$store.state.userMultiChoice;
        let selectedCount = 0;
        for (let i = 0; i < userMultiChoice.length; i++) {
          if (userMultiChoice[i].value === true) {
            selectedCount++;
          }
        }
        isDisabled = selectedCount !== currentQuestion.correct.length;
      }
      if ("text" === currentQuestion.type) {
        isDisabled = this.$store.state.userTextAnswer.trim() === "";
      }
      return isDisabled;
    }
  },
  methods: {
    startQuiz: function() {
      this.quizHandler.startQuiz();
    },
    checkAnswer: function() {
      this.quizHandler.checkAnswer();
    },
    nextQuestion: function() {
      this.quizHandler.nextQuestion();
    },
    repeatQuiz: function() {
      this.quizHandler.repeatQuiz();
    },
    exitQuiz: function() {
      this.quizHandler.resetStoreValues();
      window.location.href = "/";
    }
  }
};
</script>

<style>
.navi-btn {
  width: 50%;
}

@media (max-width: 959px) {
  .navi-btn {
    width: calc(75% - 12px);
  }
}

@media (max-width: 599px) {
  .navi-btn {
    width: calc(100% - 12px);
  }
}
</style>
