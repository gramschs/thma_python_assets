<script>
  import QuestionCard from '../components/QuestionCard.svelte';

  let { chapter, onBack } = $props();

  // Zustand
  let questions = $state([]);
  let loading = $state(true);
  let error = $state(null);

  let currentIndex = $state(0);
  let selectedOption = $state(null);
  let answered = $state(false);
  let score = $state(0);
  let finished = $state(false);

  // Fragen laden und mischen
  $effect(() => {
    loading = true;
    chapter.file()
      .then(mod => {
        questions = shuffle(mod.default);
        loading = false;
      })
      .catch(err => {
        error = 'Aufgaben konnten nicht geladen werden.';
        loading = false;
      });
  });

  const QUESTIONS_PER_QUIZ = 10;

  function shuffle(arr) {
    const a = [...arr];
    for (let i = a.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [a[i], a[j]] = [a[j], a[i]];
    }
    return a.slice(0, QUESTIONS_PER_QUIZ);
  }

  let currentQuestion = $derived(questions[currentIndex] ?? null);
  let progress = $derived(questions.length > 0 ? (currentIndex / questions.length) * 100 : 0);
  let isCorrect = $derived(answered && selectedOption === currentQuestion?.answer);

  function handleAnswer(optionIndex) {
    if (answered) return;
    selectedOption = optionIndex;
    answered = true;
    if (optionIndex === currentQuestion.answer) {
      score += 1;
    }
  }

  function nextQuestion() {
    if (currentIndex + 1 >= questions.length) {
      finished = true;
    } else {
      currentIndex += 1;
      selectedOption = null;
      answered = false;
    }
  }

  function restart() {
    questions = shuffle(questions);
    currentIndex = 0;
    selectedOption = null;
    answered = false;
    score = 0;
    finished = false;
  }

  let grade = $derived.by(() => {
    const pct = score / questions.length;
    if (pct >= 0.9) return { label: 'Ausgezeichnet!', color: 'var(--success)' };
    if (pct >= 0.7) return { label: 'Gut gemacht!', color: 'var(--accent)' };
    if (pct >= 0.5) return { label: 'Fast geschafft!', color: '#e8a317' };
    return { label: 'Weitermachen!', color: 'var(--text-muted)' };
  });
</script>

<div class="chapter-view">
  <div class="chapter-header">
    <div>
      <p class="chapter-label">Kapitel {chapter.number}</p>
      <h2 class="chapter-title">{chapter.title}</h2>
    </div>
  </div>

  {#if loading}
    <div class="status-box">
      <div class="spinner"></div>
      <p>Aufgaben werden geladen…</p>
    </div>

  {:else if error}
    <div class="status-box">
      <p class="error-text">{error}</p>
      <button class="btn-primary" onclick={onBack}>Zurück</button>
    </div>

  {:else if finished}
    <div class="results">
      <div class="results-score" style="--grade-color: {grade.color}">
        <span class="score-num">{score}</span>
        <span class="score-denom">/ {questions.length}</span>
      </div>
      <p class="results-label" style="color: {grade.color}">{grade.label}</p>
      <p class="results-sub">
        Du hast {score} von {questions.length} Aufgaben richtig beantwortet.
      </p>
      <div class="results-actions">
        <button class="btn-primary" onclick={restart}>Nochmal üben</button>
        <button class="btn-secondary" onclick={onBack}>Kapitelübersicht</button>
      </div>
    </div>

  {:else if currentQuestion}
    <div class="progress-wrap">
      <div class="progress-bar">
        <div class="progress-fill" style="width: {progress}%"></div>
      </div>
      <span class="progress-label">{currentIndex + 1} / {questions.length}</span>
    </div>

    <QuestionCard
      question={currentQuestion}
      {selectedOption}
      {answered}
      onAnswer={handleAnswer}
    />

    {#if answered}
      <div class="feedback" class:feedback--correct={isCorrect} class:feedback--wrong={!isCorrect}>
        <div class="feedback-icon">{isCorrect ? '✓' : '✗'}</div>
        <div class="feedback-body">
          <p class="feedback-title">{isCorrect ? 'Richtig!' : 'Leider falsch.'}</p>
          <p class="feedback-explanation">{@html currentQuestion.explanation}</p>
        </div>
      </div>
      <div class="next-wrap">
        <button class="btn-primary" onclick={nextQuestion}>
          {currentIndex + 1 < questions.length ? 'Nächste Aufgabe →' : 'Ergebnis anzeigen →'}
        </button>
      </div>
    {/if}
  {/if}
</div>

<style>
  .chapter-view {
    display: flex;
    flex-direction: column;
    gap: 28px;
  }

  .chapter-header {
    display: flex;
    justify-content: space-between;
    align-items: flex-start;
  }

  .chapter-label {
    font-family: var(--font-code);
    font-size: 0.75rem;
    color: var(--accent);
    letter-spacing: 0.1em;
    text-transform: uppercase;
    margin-bottom: 6px;
  }

  .chapter-title {
    font-size: 1.4rem;
    font-weight: 400;
    color: var(--text);
    letter-spacing: -0.01em;
  }

  /* Progress */
  .progress-wrap {
    display: flex;
    align-items: center;
    gap: 14px;
  }

  .progress-bar {
    flex: 1;
    height: 4px;
    background: var(--surface-2);
    border-radius: 2px;
    overflow: hidden;
  }

  .progress-fill {
    height: 100%;
    background: var(--accent);
    border-radius: 2px;
    transition: width 0.4s ease;
  }

  .progress-label {
    font-family: var(--font-code);
    font-size: 0.78rem;
    color: var(--text-muted);
    white-space: nowrap;
  }

  /* Feedback */
  .feedback {
    display: flex;
    gap: 16px;
    padding: 18px 20px;
    border-radius: var(--radius-lg);
    border: 1px solid transparent;
    animation: fadeIn 0.2s ease;
  }

  .feedback--correct {
    background: var(--success-dim);
    border-color: var(--success);
  }

  .feedback--wrong {
    background: var(--error-dim);
    border-color: var(--error);
  }

  .feedback-icon {
    font-size: 1.1rem;
    font-weight: 600;
    flex-shrink: 0;
    margin-top: 1px;
  }

  .feedback--correct .feedback-icon { color: var(--success); }
  .feedback--wrong .feedback-icon { color: var(--error); }

  .feedback-title {
    font-weight: 600;
    font-size: 0.95rem;
    margin-bottom: 4px;
  }

  .feedback--correct .feedback-title { color: var(--success); }
  .feedback--wrong .feedback-title { color: var(--error); }

  .feedback-explanation {
    font-size: 0.88rem;
    color: var(--text-muted);
    line-height: 1.6;
  }

  .next-wrap {
    display: flex;
    justify-content: flex-end;
  }

  /* Results */
  .results {
    display: flex;
    flex-direction: column;
    align-items: center;
    text-align: center;
    gap: 16px;
    padding: 48px 0;
  }

  .results-score {
    display: flex;
    align-items: baseline;
    gap: 4px;
  }

  .score-num {
    font-size: 5rem;
    font-weight: 300;
    color: var(--grade-color);
    font-family: var(--font-code);
    line-height: 1;
  }

  .score-denom {
    font-size: 2rem;
    color: var(--text-muted);
    font-family: var(--font-code);
  }

  .results-label {
    font-size: 1.4rem;
    font-weight: 600;
    letter-spacing: -0.01em;
  }

  .results-sub {
    color: var(--text-muted);
    font-size: 0.95rem;
  }

  .results-actions {
    display: flex;
    gap: 12px;
    margin-top: 16px;
    flex-wrap: wrap;
    justify-content: center;
  }

  /* Buttons */
  .btn-primary {
    background: var(--accent);
    color: #0d1117;
    border: none;
    padding: 10px 24px;
    border-radius: var(--radius);
    font-size: 0.9rem;
    font-weight: 600;
    transition: opacity 0.15s, transform 0.1s;
  }

  .btn-primary:hover {
    opacity: 0.88;
    transform: translateY(-1px);
  }

  .btn-secondary {
    background: none;
    border: 1px solid var(--border);
    color: var(--text-muted);
    padding: 10px 24px;
    border-radius: var(--radius);
    font-size: 0.9rem;
    transition: border-color 0.15s, color 0.15s;
  }

  .btn-secondary:hover {
    border-color: var(--text-muted);
    color: var(--text);
  }

  /* Loading */
  .status-box {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 16px;
    padding: 60px 0;
    color: var(--text-muted);
  }

  .spinner {
    width: 28px;
    height: 28px;
    border: 2px solid var(--border);
    border-top-color: var(--accent);
    border-radius: 50%;
    animation: spin 0.7s linear infinite;
  }

  .error-text {
    color: var(--error);
  }

  @keyframes spin {
    to { transform: rotate(360deg); }
  }

  @keyframes fadeIn {
    from { opacity: 0; transform: translateY(6px); }
    to   { opacity: 1; transform: translateY(0); }
  }
</style>
