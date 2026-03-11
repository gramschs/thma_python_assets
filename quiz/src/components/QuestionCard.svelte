<script>
  let { question, selectedOption, answered, onAnswer } = $props();

  const DIFFICULTY_STARS = ['', '★', '★★', '★★★'];
  const TOPIC_COLORS = {
    'Datentypen':       '#58a6ff',
    'Variablen':        '#bc8cff',
    'Operatoren':       '#f78166',
    'EVA-Prinzip':      '#3fb950',
    'input()':          '#e8a317',
    'Typumwandlung':    '#ffa657',
    'Zuweisungsoperator': '#79c0ff',
    'print()':          '#56d364',
  };

  function optionState(index) {
    if (!answered) return 'default';
    if (index === question.answer) return 'correct';
    if (index === selectedOption) return 'wrong';
    return 'dimmed';
  }
</script>

<div class="card">
  <div class="card-meta">
    <span class="topic-badge" style="color: {TOPIC_COLORS[question.topic] ?? 'var(--text-muted)'}">
      {question.topic}
    </span>
    <span class="difficulty" title="Schwierigkeit">
      {DIFFICULTY_STARS[question.difficulty] ?? ''}
    </span>
  </div>

  <p class="question-text">{@html question.question}</p>

  {#if question.type === 'output_prediction'}
    <pre class="code-block"><code>{question.code}</code></pre>
  {/if}

  <div class="options">
    {#each question.options as option, i}
      <button
        class="option"
        class:option--correct={answered && i === question.answer}
        class:option--wrong={answered && i === selectedOption && i !== question.answer}
        class:option--dimmed={answered && i !== question.answer && i !== selectedOption}
        class:option--selected={!answered && i === selectedOption}
        disabled={answered}
        onclick={() => onAnswer(i)}
      >
        <span class="option-letter">{String.fromCharCode(65 + i)}</span>
        <span class="option-text">{@html option}</span>
        {#if answered && i === question.answer}
          <span class="option-check">✓</span>
        {:else if answered && i === selectedOption && i !== question.answer}
          <span class="option-check">✗</span>
        {/if}
      </button>
    {/each}
  </div>
</div>

<style>
  .card {
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: var(--radius-lg);
    padding: 28px;
    display: flex;
    flex-direction: column;
    gap: 20px;
    animation: slideIn 0.25s ease;
  }

  .card-meta {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 12px;
  }

  .topic-badge {
    font-family: var(--font-code);
    font-size: 0.72rem;
    letter-spacing: 0.08em;
    text-transform: uppercase;
    font-weight: 500;
  }

  .difficulty {
    font-size: 0.65rem;
    color: var(--accent);
    letter-spacing: 0.05em;
  }

  .question-text {
    font-size: 1.05rem;
    color: var(--text);
    line-height: 1.65;
    font-weight: 400;
  }

  .code-block {
    background: var(--bg);
    border: 1px solid var(--border);
    border-left: 3px solid var(--accent);
    border-radius: var(--radius);
    padding: 16px 18px;
    overflow-x: auto;
  }

  .code-block code {
    font-family: var(--font-code);
    font-size: 0.9rem;
    color: var(--text);
    background: none;
    border: none;
    padding: 0;
    white-space: pre;
    line-height: 1.7;
  }

  .options {
    display: flex;
    flex-direction: column;
    gap: 10px;
  }

  .option {
    display: flex;
    align-items: flex-start;
    gap: 14px;
    background: var(--surface-2);
    border: 1px solid var(--border);
    border-radius: var(--radius);
    padding: 13px 16px;
    text-align: left;
    transition: border-color 0.15s, background 0.15s;
    width: 100%;
  }

  .option:not(:disabled):hover {
    border-color: var(--accent);
    background: var(--accent-dim);
  }

  .option--correct {
    border-color: var(--success) !important;
    background: var(--success-dim) !important;
  }

  .option--wrong {
    border-color: var(--error) !important;
    background: var(--error-dim) !important;
  }

  .option--dimmed {
    opacity: 0.4;
  }

  .option-letter {
    font-family: var(--font-code);
    font-size: 0.75rem;
    color: var(--text-muted);
    background: var(--border);
    width: 22px;
    height: 22px;
    border-radius: 4px;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-shrink: 0;
    font-weight: 500;
    margin-top: 1px;
  }

  .option--correct .option-letter {
    background: var(--success);
    color: #0d1117;
  }

  .option--wrong .option-letter {
    background: var(--error);
    color: #fff;
  }

  .option-text {
    font-size: 0.92rem;
    color: var(--text);
    line-height: 1.5;
    flex: 1;
  }

  .option-check {
    font-size: 0.9rem;
    flex-shrink: 0;
    margin-top: 1px;
  }

  .option--correct .option-check { color: var(--success); }
  .option--wrong .option-check   { color: var(--error); }

  @keyframes slideIn {
    from { opacity: 0; transform: translateY(10px); }
    to   { opacity: 1; transform: translateY(0); }
  }
</style>
