<template>
  <main class="game-shell">
    <section class="board-wrap" aria-label="Игровое поле">
      <div class="board-toolbar">
        <button class="retry-button" type="button" @click="openFileDialog">Загрузить вопросы</button>
      </div>
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
            type="button"
            @click="openQuestion(category, question)"
          >
            <span v-if="question.used">-</span>
            <span v-else>{{ question.value }}</span>
          </button>
        </div>
      </div>
    </section>

    <div v-if="isFileDialogOpen" class="file-dialog-backdrop" @click.self="closeFileDialog">
      <section class="file-dialog" role="dialog" aria-modal="true" aria-labelledby="file-dialog-title">
        <h2 id="file-dialog-title">Загрузить вопросы</h2>
        <form @submit.prevent="submitFileName">
          <label for="questions-file-name">Имя файла</label>
          <input
            id="questions-file-name"
            v-model="fileNameInput"
            type="text"
            placeholder="Например, r2 или r2.json"
            autocomplete="off"
            required
          />
          <p v-if="fileNameError" class="file-dialog-error" role="alert">{{ fileNameError }}</p>
          <div class="file-dialog-actions">
            <button class="retry-button" type="button" :disabled="isLoading" @click="closeFileDialog">Отмена</button>
            <button class="retry-button" type="submit" :disabled="isLoading">
              {{ isLoading ? "Загрузка…" : "Загрузить" }}
            </button>
          </div>
        </form>
      </section>
    </div>

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
          <div class="score-adjustments">
            <button type="button" @click="adjustScore(player, 100)">+</button>
            <button type="button" @click="adjustScore(player, -100)">−</button>
          </div>
        </section>
      </div>
    </footer>

    <section v-if="activeQuestion" class="question-dialog" role="dialog" aria-modal="true">
      <button v-if="!showingAnswer" class="question-card" type="button" @click="revealAnswer">
        <div class="special" v-if="activeQuestion.special">{{ activeQuestion.special }}</div>
        <span class="question-copy" :class="{ 'is-short': isShortValue(activeQuestion.text) }" v-html="activeQuestion.text"></span>
      </button>
      <div v-else class="answer-card" @click="openQuestion(null, null)">
        <p class="question-copy" :class="{ 'is-short': isShortValue(activeQuestion.answer) }">{{ activeQuestion.answer }}</p>
      </div>
    </section>
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
      isFileDialogOpen: false,
      fileNameInput: "r1",
      fileNameError: "",
      questionsFileName: "r1",
      activeCategory: null,
      activeQuestion: null,
      showingAnswer: false,
    };
  },
  mounted() {
    this.loadQuestions("r1");
  },
  methods: {
    openFileDialog() {
      this.fileNameInput = this.questionsFileName;
      this.fileNameError = "";
      this.isFileDialogOpen = true;
    },
    closeFileDialog() {
      this.isFileDialogOpen = false;
      this.fileNameError = "";
    },
    async submitFileName() {
      const fileName = this.fileNameInput.trim().replace(/\.json$/i, "");
      if (!/^[\w-]+$/.test(fileName)) {
        this.fileNameError = "Введите имя файла без пути и специальных символов.";
        return;
      }

      this.fileNameError = "";
      this.questionsFileName = fileName;
      await this.loadQuestions(fileName);
      this.closeFileDialog();
    },
    async loadQuestions(name = this.questionsFileName) {
      this.isLoading = true;
      this.loadError = "";
      try {
        const response = await fetch(`${process.env.BASE_URL}${name}.json`);
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
      if (question) question.used = true;
      this.activeCategory = category;
      this.activeQuestion = question;
      this.showingAnswer = false;
    },
    revealAnswer() {
      this.showingAnswer = true;
    },
    adjustScore(player, amount) {
      player.score = (Number(player.score) || 0) + amount;
    },
    isShortValue(value) {
      return (
        String(value)
          .replace(/<[^>]*>/g, "")
          .trim().length < 40
      );
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
  touch-action: pan-x pan-y;
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
  min-height: 100dvh;
  margin: 0;
  overflow: hidden;
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
.board-toolbar {
  display: flex;
  justify-content: flex-end;
  margin-bottom: 8px;
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
  min-height: 90px;
}
.category-row:not(:first-child) {
  border-top: 1px solid var(--line);
}
.category-cell {
  display: flex;
  align-items: center;
  gap: 11px;
  color: #e8e8df;
  font-size: 30px;
  font-weight: 600;
}
.question-cell {
  position: relative;
  display: grid;
  min-height: 70px;
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
.is-short {
  font-size: 100px !important;
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
.retry-button:disabled {
  cursor: wait;
  opacity: 0.6;
}
.file-dialog-backdrop {
  position: fixed;
  z-index: 10;
  inset: 0;
  display: grid;
  place-items: center;
  padding: 20px;
  background: rgba(7, 9, 7, 0.82);
  backdrop-filter: blur(7px);
}
.file-dialog {
  width: min(420px, 100%);
  padding: 24px;
  border: 1px solid rgba(235, 239, 223, 0.14);
  border-radius: 14px;
  background: #191c18;
  box-shadow: 0 30px 100px rgba(0, 0, 0, 0.48);
}
.file-dialog h2 {
  margin-bottom: 18px;
}
.file-dialog form,
.file-dialog label {
  display: block;
}
.file-dialog label {
  margin-bottom: 7px;
  color: #a5a89e;
  font-size: 13px;
}
.file-dialog input {
  width: 100%;
  padding: 10px 12px;
  border: 1px solid var(--line);
  border-radius: 6px;
  outline: none;
  background: var(--surface-raised);
  color: #f3f0e9;
}
.file-dialog input:focus {
  border-color: var(--accent);
}
.file-dialog-error {
  margin: 10px 0 0;
  color: #ff9a91;
  font-size: 13px;
}
.file-dialog-actions {
  display: flex;
  justify-content: flex-end;
  gap: 8px;
  margin-top: 18px;
}
.file-dialog-actions .retry-button {
  margin: 0;
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
.score-adjustments {
  display: flex;
  justify-content: flex-end;
  gap: 16px;
  margin-top: 8px;
}
.score-adjustments button {
  width: 50px;
  height: 26px;
  padding: 0;
  border: 1px solid var(--line);
  border-radius: 5px;
  background: var(--surface-raised);
  color: var(--accent);
  cursor: pointer;
  font-size: 15px;
  line-height: 1;
}
.score-adjustments button:hover {
  border-color: var(--accent);
  background: #292d34;
}
.score-adjustments button:focus-visible {
  outline: 2px solid var(--accent);
  outline-offset: 2px;
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

.question-dialog {
  position: fixed;
  z-index: 5;
  top: 20px;
  left: 20px;
  width: calc(100% - 40px);
  overflow: hidden;
  border: 1px solid rgba(235, 239, 223, 0.14);
  border-radius: 14px;
  background: #191c18;
  box-shadow: 0 30px 100px rgba(0, 0, 0, 0.48);
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
  min-height: 80dvh;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  padding: 40px clamp(22px, 7vw, 70px) 25px;
  border: 0;
  background: transparent;
  color: #f3f0e9;
  text-align: center;
  position: relative;
}
.question-card .special {
  top: 10px;
  left: 10px;
  color: #dc5;
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
