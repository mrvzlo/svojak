<template>
  <main class="game-shell">
    <section class="board-wrap" aria-label="Игровое поле">
      <div v-if="loadError" class="load-error" role="alert">
        <strong>Не удалось загрузить игру</strong>
        <span>{{ loadError }}</span>
        <button class="retry-button" type="button" @click="loadQuestions">Попробовать снова</button>
      </div>
      <div v-else-if="isLoading" class="loading-state" role="status">Загружаем вопросы…</div>
      <div v-else class="game-board">
        <div v-for="category in categories" :key="category.name" class="category-row">
          <div class="category-cell">
            <span>{{ category.name }}</span>
          </div>
          <button
            v-for="question in category.questions"
            :key="question.value"
            class="question-cell"
            :class="{ 'is-used': question.used }"
            :disabled="question.used"
            type="button"
            @click="openQuestion(category, question)"
          >
            <span v-if="question.used"></span>
            <span v-else>{{ question.value }}</span>
          </button>
        </div>
      </div>
    </section>

    <footer class="players-section">
      <div class="players-grid">
        <section v-for="(player, index) in players" :key="index" class="player-card" :class="`player-card-${index + 1}`">
          <div class="score-row">
            <input
              :id="`player-name-${index}`"
              v-model="player.name"
              class="player-name"
              type="text"
              maxlength="24"
              :placeholder="`Игрок ${index + 1}`"
            />
            <div class="score-input-wrap">
              <input :id="`player-score-${index}`" v-model.number="player.score" type="number" step="100" />
            </div>
          </div>
        </section>
      </div>
    </footer>

    <div v-if="activeQuestion" class="question-overlay" role="dialog" aria-modal="true" @click.self="closeQuestion">
      <section class="question-dialog">
        <button v-if="!showingAnswer" class="question-card" type="button" @click="revealAnswer">
          <img v-if="activeQuestion.isImage" class="question-image" :src="activeQuestion.text" />
          <span v-else class="question-copy">{{ activeQuestion.text }}</span>
        </button>
        <div v-else class="answer-card">
          <img v-if="activeQuestion.answerImage" class="question-image" :src="activeQuestion.answer" />
          <p v-else class="question-copy">{{ activeQuestion.answer }}</p>
        </div>
      </section>
    </div>
  </main>
</template>

<script>
const defaultPlayers = () => [
  { name: "", score: 0 },
  { name: "", score: 0 },
  { name: "", score: 0 },
];

export default {
  name: "App",
  data() {
    return {
      categories: [],
      players: defaultPlayers(),
      isLoading: true,
      loadError: "",
      activeCategory: null,
      activeQuestion: null,
      showingAnswer: false,
    };
  },
  mounted() {
    this.loadQuestions();
  },
  methods: {
    async loadQuestions() {
      this.isLoading = true;
      this.loadError = "";
      try {
        const response = await fetch(`${process.env.BASE_URL}questions.json`);
        const data = await response.json();
        this.categories = data.categories.map((category) => ({
          ...category,
          questions: category.questions.map((question) => ({ ...question, used: false })),
        }));
      } catch (error) {
        this.loadError = error instanceof Error ? error.message : "Неизвестная ошибка загрузки.";
      } finally {
        this.isLoading = false;
      }
    },
    openQuestion(category, question) {
      if (question.used) return;
      question.used = true;
      this.activeCategory = category;
      this.activeQuestion = question;
      this.showingAnswer = false;
    },
    revealAnswer() {
      this.showingAnswer = true;
    },
    closeQuestion() {
      this.activeQuestion = null;
      this.activeCategory = null;
      this.showingAnswer = false;
    },
  },
};
</script>

<style>
:root {
  font-family: "DM Sans", sans-serif;
  color: #f3f0e9;
  background: #101210;
  font-synthesis: none;
  text-rendering: optimizeLegibility;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  font-optical-sizing: auto;
  font-weight: 400;
  --surface: #18181b;
  --surface-raised: #1f1f23;
  --muted: #83848c;
  --accent: #6a95f3;
  --line: rgba(235, 239, 223, 0.1);
}

* {
  box-sizing: border-box;
}

body {
  min-width: 320px;
  min-height: 100vh;
  margin: 0;
  background: radial-gradient(ellipse at 74% 0%, rgba(116, 135, 57, 0.09), transparent 31rem), #101210;
}

button,
input {
  font: inherit;
}
button {
  -webkit-tap-highlight-color: transparent;
}

.game-shell {
  width: min(1120px, calc(100% - 64px));
  margin: 0 auto;
}

h1,
h2 {
  margin: 0;
  font-family: Manrope, sans-serif;
  font-weight: 600;
  letter-spacing: -2px;
}
h1 {
  font-size: clamp(36px, 5vw, 52px);
  line-height: 1;
}
h1 span {
  color: var(--accent);
}

.board-wrap {
  min-height: 276px;
  margin-top: 8px;
}
.game-board {
  padding: 0 14px;
  border: 1px solid var(--line);
  border-radius: 12px;
  background: rgba(24, 27, 24, 0.9);
  overflow: hidden;
}
.board-heading,
.category-row {
  display: grid;
  grid-template-columns: minmax(170px, 1.35fr) repeat(5, minmax(72px, 1fr));
  gap: 8px;
}
.category-row {
  min-height: 100px;
}
.category-row:not(:first-child) {
  border-top: 1px solid var(--line);
}
.category-cell {
  display: flex;
  align-items: center;
  gap: 11px;
  color: #e8e8df;
  font-size: 32px;
  font-weight: 600;
}
.question-cell {
  position: relative;
  display: grid;
  min-height: 80px;
  place-items: center;
  align-self: center;
  padding: 0;
  border: 1px solid transparent;
  border-radius: 7px;
  background: #252922;
  color: var(--accent);
  cursor: pointer;
  font-family: Manrope, sans-serif;
  font-size: 32px;
  font-weight: 700;
  transition: background 0.16s ease, border-color 0.16s ease, transform 0.16s ease, color 0.16s ease;
}
.question-cell:not(:disabled):hover {
  transform: translateY(-2px);
  border-color: rgba(215, 243, 106, 0.38);
  background: #30362a;
}
.question-cell:focus-visible,
.close-button:focus-visible,
.question-cell.is-used {
  background: transparent;
  color: #59604b;
  cursor: default;
}

.loading-state,
.load-error {
  display: flex;
  min-height: 245px;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  gap: 10px;
  border: 1px solid var(--line);
  border-radius: 12px;
  color: #a5a89e;
  font-size: 13px;
}
.load-error strong {
  color: #f3f0e9;
  font-family: Manrope, sans-serif;
  font-size: 18px;
}
.load-error span {
  color: #a5a89e;
}
.retry-button {
  margin-top: 5px;
  padding: 9px 14px;
  border: 1px solid var(--line);
  border-radius: 6px;
  background: var(--surface-raised);
  color: var(--accent);
  cursor: pointer;
  font-size: 11px;
}

.players-section {
  padding-top: 16px;
}
h2 {
  font-size: 24px;
  letter-spacing: -0.9px;
}
h2 span {
  color: #85897e;
  font-weight: 400;
}
.players-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 12px;
}
.player-card {
  min-width: 0;
  padding: 16px 18px 14px;
  border: 1px solid var(--line);
  border-radius: 10px;
  background: var(--surface);
}
.player-card-top {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 10px;
}
.player-name {
  width: 100%;
  padding: 0 0 10px;
  border: 0;
  border-bottom: 1px solid var(--line);
  border-radius: 0;
  outline: none;
  background: transparent;
  color: #f3f0e9;
  font-family: Manrope, sans-serif;
  font-size: 17px;
  font-weight: 600;
}
.player-name::placeholder {
  color: #777b70;
}
.player-name:focus {
  border-bottom-color: var(--accent);
}
.score-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding-top: 12px;
}
.score-row > label {
  color: #777b70;
  font-size: 8px;
  font-weight: 700;
  letter-spacing: 1px;
}
.score-input-wrap {
  display: flex;
  align-items: baseline;
  gap: 5px;
}
.score-input-wrap input {
  width: 120px;
  padding: 0;
  border: 0;
  outline: none;
  background: transparent;
  color: var(--accent);
  font-family: Manrope, sans-serif;
  font-size: 32px;
  font-weight: 700;
  text-align: right;
}
.score-input-wrap input:focus {
  text-decoration: underline;
  text-decoration-color: rgba(215, 243, 106, 0.45);
  text-underline-offset: 4px;
}
.score-input-wrap input::-webkit-inner-spin-button,
.score-input-wrap input::-webkit-outer-spin-button {
  margin-left: 5px;
  opacity: 0.4;
}
.score-input-wrap span {
  color: #777b70;
  font-size: 9px;
}

.question-overlay {
  position: fixed;
  z-index: 5;
  inset: 0;
  display: grid;
  place-items: center;
  padding: 20px;
  background: rgba(7, 9, 7, 0.82);
  backdrop-filter: blur(7px);
}
.question-dialog {
  width: min(630px, 100%);
  overflow: hidden;
  border: 1px solid rgba(235, 239, 223, 0.14);
  border-radius: 14px;
  background: #191c18;
  box-shadow: 0 30px 100px rgba(0, 0, 0, 0.48);
  animation: dialog-in 0.2s ease-out;
}
@keyframes dialog-in {
  from {
    opacity: 0;
    transform: translateY(8px) scale(0.99);
  }
  to {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}
.close-button {
  display: grid;
  width: 30px;
  height: 30px;
  place-items: center;
  border: 1px solid var(--line);
  border-radius: 50%;
  background: transparent;
  color: #a5a89e;
  cursor: pointer;
  font-size: 21px;
  line-height: 1;
}
.close-button:hover {
  color: white;
  border-color: rgba(235, 239, 223, 0.3);
}
.question-card,
.answer-card {
  display: flex;
  width: 100%;
  min-height: 305px;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  padding: 40px clamp(22px, 7vw, 70px) 25px;
  border: 0;
  background: transparent;
  color: #f3f0e9;
  text-align: center;
}
.question-card {
  cursor: pointer;
}
.question-copy {
  margin: 0;
  font-family: Manrope, sans-serif;
  font-size: clamp(21px, 4vw, 30px);
  font-weight: 500;
  letter-spacing: -0.8px;
  line-height: 1.4;
}
.question-image {
  display: block;
  max-width: 100%;
  max-height: 290px;
  object-fit: contain;
  border-radius: 8px;
}

.answer-card {
  min-height: 305px;
}

@media (max-width: 720px) {
  .game-shell {
    width: min(100% - 32px, 560px);
  }
  .board-wrap {
    overflow-x: auto;
  }
  .game-board {
    min-width: 610px;
  }
  .players-section {
    padding-top: 34px;
  }
  .players-hint {
    max-width: 135px;
    text-align: right;
    line-height: 1.5;
  }
  .players-grid {
    grid-template-columns: 1fr;
    gap: 9px;
  }
  .player-card {
    padding: 13px 15px 12px;
  }
  .player-card-top {
    margin-bottom: 7px;
  }
  .player-name {
    font-size: 15px;
  }
  .score-row {
    padding-top: 9px;
  }
}

@media (max-width: 420px) {
  .game-shell {
    width: calc(100% - 24px);
  }
  .board-wrap {
    margin-right: -12px;
  }
}

@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    scroll-behavior: auto !important;
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
</style>
