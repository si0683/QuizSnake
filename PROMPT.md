# Промт: Quiz Snake — змейка с образовательным квизом

> Это готовое техническое задание для нейросети. Скопируйте весь файл и отдайте его целиком.
> Документ описывает игру **в точности такой, какая она лежит в этом репозитории**. Если игра меняется, этот файл обновляется вместе с ней (см. `AGENTS.md`).
>
> Файл состоит из двух частей:
> 1. **ТЗ** — что и почему должна делать игра. Важные места и исправленные ошибки отмечены знаком ⚠️.
> 2. **Полные исходники** в конце (раздел «Исходный код») — эталонная рабочая версия.
>
> **Нейросети:** если нужна точная копия игры, создай три файла **один в один** по разделу «Исходный код» и скачай звуки по ссылкам. Если просят изменить игру, бери исходники за основу и соблюдай ТЗ.

---

## Роль и цель

Ты — опытный разработчик образовательных браузерных игр. Напиши игру «Quiz Snake» — змейка с образовательным квизом — на **HTML, CSS и чистом JavaScript (Canvas 2D)**, без библиотек, TypeScript, React и сборки. Игру показывают детям на уроке программирования: код простой, с подробными комментариями на русском, игра красивая и работает на компьютере и телефоне.

Что в игре:
- вопрос висит над полем, на поле лежат 4 ответа-пузыря — змейка должна съесть правильный (растёт), неверный — минус жизнь;
- в режиме «Ввод» змейка ест ❓ и печатает ответ в окне вопроса;
- по полю бродят враги, каждые 3 верных ответа — новый уровень и выбор улучшения (12 улучшений), иногда босс-преследователь;
- стены, хвост и враги отнимают жизнь, есть неуязвимость после удара;
- 3 мира: Фэнтези, Космос, Эко-Панк; предметы: математика, русский, английский.

## Лаборатория (раздел 0 в начале `script.js`)

| Поле | По умолчанию | Что делает |
|---|---|---|
| `titleTop` | `'Quiz'` | Название в меню |
| `snakeHead` | `''` | Своя голова змейки |
| `startLength` | `4` | Длина на старте |
| `lives` | `3` | Жизни |
| `snakeSpeed` | `1` | Множитель скорости змейки |
| `maxEnemies` | `4` | Максимум врагов на поле |
| `growPerAnswer` | `2` | Рост за верный ответ |
| `wrapWalls` | `false` | Сквозь стены |
| `enemySpeed` | `1` | Множитель скорости врагов |
| `rainbowSnake` | `false` | Радужная змейка |

### Механика: змейка
- Поле 20 × 13 клеток по 40, сверху полоса 64 под панель (мир 800 × 584), целиком помещается в экран. Шахматка поля, рамка (пунктир, если стены проходимые).
- Змейка двигается шагами: раз в `step / LAB.snakeSpeed · бонус · 0.95^(уровень−1)` с (Легко 0.18, Нормально 0.14, Сложно 0.11; не быстрее 0.06). Очередь поворотов до 2, разворот назад запрещён.
- ⚠️ Режим «Кнопки»: 4 ответа лежат на поле пузырями (не ближе 5 клеток к голове), вопрос — в плашке над полем. Съел верный → +`growPerAnswer` длины, очки 10·уровень. Неверный → −1 жизнь, тост с правильным ответом, новый вопрос. Режим «Ввод»: на поле один ❓, съел → окно вопроса с полем ввода (пауза).
- Каждые 3 верных ответа — уровень +1, ещё враг (до `maxEnemies`), выбор улучшения; каждые 9 — босс-преследователь на 15 с (идёт к голове).
- Враги ходят по клеткам каждые 0.75/0.6/0.45 с (сложность) случайно, не наступая на ответы. Стена (без «сквозь стены»), хвост или враг → −1 жизнь, змейка возвращается в центр, 2 с неуязвимости (мигает).
- 12 улучшений: медленнее, +1 жизнь, сквозь стены, щит, подсказка 50/50 (на поле 2 ответа; во «Вводе» — первая буква), очки ×2, заморозка врагов, фрукты 🍎, гибкий хвост, охотник (можно есть врагов), долгая защита 4 с, короче хвост.

## Общее для всех пяти игр (Quiz Shooter, Quiz Tower, Quiz Snake, Quiz Jump, Quiz Breakout)

### Миры (`THEMES`)
| id | Название | Акцент | Враги | Босс |
|---|---|---|---|---|
| fantasy | 🏰 Фэнтези — рыцари и магия | `#FFD700` (тёмно-синий фон) | 👹 💀 | 🐉 Дракон |
| space | 🚀 Космос — лазеры и НЛО | `#00FFFF` (чёрный фон) | ☄️ 🛸 | 👾 Мега-Мозг |
| eco | 🌿 Эко-Панк — защита природы | `#39FF14` (тёмно-зелёный фон) | 🗑️ 🦗 | 🧟 Токсичный Монстр |

Герои миров: Фэнтези 🤺 🧙 🧝‍♂️, Космос 🧑‍🚀 🤖 👽, Эко-Панк 🧑‍🌾 🐼 🦉. Акцент мира пишется в CSS-переменную `--accent`.

### Квиз (раздел «Вопросы» одинаковый во всех играх)
- Предметы (`SUBJECTS`): 🧮 Математика (пул из 20 случайных примеров: сложение, вычитание без отрицательных ответов, умножение; числа растут со сложностью), 📝 Русский (28 словарных слов с пропущенной гласной: `М_ЛОКО`), 🇬🇧 Английский (26 слов для перевода: `SPACE → Космос`).
- Задания перемешиваются и идут по очереди (`questionPool`), пул пересобирается, когда кончился. 4 варианта: верный + 3 неверных (близкие числа / другие гласные / другие переводы).
- Способ ответа (`INPUT_MODES`): **Кнопки** — 4 большие кнопки (клавиши `1`–`4`); **Ввод** — поле `<input>` + Enter, ⚠️ автофокус при открытии, `inputMode = numeric` для математики. Сравнение `normalize()`: без пробелов, без регистра, Ё = Е.
- ⚠️ Пока открыт вопрос — **жёсткая пауза** (мир не двигается). Неверно — окно мигает красным, подсвечивается/пишется правильный ответ (1.1 с). Верно — зелёная вспышка (0.5 с) и награда своей игры.
- Улучшения (перки) — рогалик: 3 случайных карточки из пула, каждое можно взять один раз за игру.

### Экраны и интерфейс
- HTML-слои поверх `<canvas>`: меню (название из `LAB`, подпись «мир · предмет · способ ответа», рекорд, «Начать», «Настройки», подсказки по управлению), выбор сложности (3 карточки), вопрос, улучшения, настройки (мир, предмет, способ ответа — карточки; звук + громкость; сброс рекорда), пауза, GAME OVER (очки, рекорд, «Рестарт», «В меню»), отсчёт 3-2-1 (кроме Quiz Tower), всплывающие надписи.
- ⚠️ Верхняя панель — сетка `grid-template-columns: 1fr auto 1fr` (у змейки `1fr 1fr`, вопрос — плашкой по центру): центральный блок **точно по центру**, все блоки и кнопка паузы одной высоты и на одной линии. Через `space-between` центр уезжал (до 46 px), высоты были разные. Экраны — `position: fixed` на всё окно; всплывающие надписи переносятся (`max-width: 92%`).
- Стекло-панели `rgba(12,16,40,0.84)` + `backdrop-filter: blur`, объёмные кнопки (янтарная и бирюзовая), шрифты Russo One + Rubik через `@import` в CSS, адаптив `@media`.
- `Esc`/`P` — пауза, автопауза при скрытии вкладки, после показа экрана `blur()` активной кнопки. Пока печатают ответ — клавиши игры не работают.
- Сохранение в `localStorage` (`<игра>-settings`, `<игра>-best`) в `try/catch`.

### Звук
⚠️ Только Web Audio API (`fetch → decodeAudioData`, `createBufferSource → createGain`), `SOUND_START = { coin: 0.045 }`. Файлы в `sound/`: `jump.wav`, `coin.mp3`, `tick.wav`, `start.wav`, `crash.wav`. В CodePen (`codepen`/`cdpn.io` в адресе) звуки берутся с GitHub Pages игры.

### Файлы и CodePen
- `index.html` / `style.css` / `script.js` — строго раздельно, чистый JavaScript + Canvas 2D, графика — эмодзи и фигуры, без библиотек и сборки. Подключение с `?v=N`.
- ⚠️ `codepen/index.pen.html` — часть `index.html` между метками «Для CodePen», без `<head>`, `<link>`, `<script>`: CodePen сам подключает CSS и JS, иначе скрипт загрузится дважды (`Identifier ... has already been declared`).
- Код для детей: объекты и функции (без классов), подробные русские комментарии, нумерованные разделы, раздел 0 — «Лаборатория» `LAB` из 10 настроек (значения по умолчанию = обычная игра).

## Чек-лист

- [ ] Три отдельных файла, чистый JS без библиотек, комментарии на русском, `LAB` в начале.
- [ ] Нет ошибок в консоли; движение через `dt` (секунды, не больше 0.05).
- [ ] Вопрос ставит игру на жёсткую паузу; верный ответ даёт награду, неверный показывает правильный ответ.
- [ ] Работают оба способа ответа (кнопки и ввод с автофокусом) и все три предмета и мира.
- [ ] Настройки и рекорд сохраняются; `?v=` у CSS и JS.
- [ ] В CodePen работают `codepen/index.pen.html` + `style.css` + `script.js`, звуки грузятся с GitHub Pages.

---

## Звуковые файлы

Положи в папку `sound/` рядом с `index.html` (или скачай из репозитория https://github.com/si0683/QuizSnake):

| Файл | Скачать |
|---|---|
| `jump.wav` | https://si0683.github.io/QuizSnake/sound/jump.wav |
| `coin.mp3` | https://si0683.github.io/QuizSnake/sound/coin.mp3 |
| `tick.wav` | https://si0683.github.io/QuizSnake/sound/tick.wav |
| `start.wav` | https://si0683.github.io/QuizSnake/sound/start.wav |
| `crash.wav` | https://si0683.github.io/QuizSnake/sound/crash.wav |

## Исходный код

<!-- SOURCES:START -->

<!-- Этот раздел создаётся скриптом tools/build.py — не редактируйте вручную -->

### `index.html`

````html
<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover">
  <title>Quiz Snake</title>
  <!-- Подключаем файл с оформлением.
       ?v=2 — номер версии: меняем его при каждом обновлении игры,
       чтобы браузер не брал старый файл из кэша -->
  <link rel="stylesheet" href="style.css?v=2">
</head>
<body>

<!-- ↓↓↓ Для CodePen копируйте в окно HTML всё, что ниже этой строки ↓↓↓
     Только эту часть! CodePen сам подключает style.css и script.js.
     Если скопировать строки <link> или <script src>, игра загрузится дважды
     и появится ошибка «Identifier ... has already been declared». -->

<!-- «Сцена» — рамка игры. Внутри холст (canvas) и все экраны поверх него -->
<div class="stage" id="stage">

  <!-- Холст: здесь JavaScript рисует поле, змейку, ответы и врагов -->
  <canvas id="gameCanvas"></canvas>

  <!-- Панель во время игры -->
  <div class="hud hidden" id="hud">
    <div class="hud-top">
      <div class="hud-left">
        <div class="hud-box">
          <span class="hud-label">Жизни</span>
          <b class="hearts" id="lives">❤️❤️❤️</b>
        </div>
        <div class="hud-box">
          <span class="hud-label">Очки</span>
          <b id="score">0</b>
        </div>
      </div>
      <div class="hud-right">
        <div class="hud-box">
          <span class="hud-label">Длина</span>
          <b id="length">4</b>
        </div>
        <div class="hud-box">
          <span class="hud-label">Уровень</span>
          <b id="level">1</b>
        </div>
        <button class="icon-btn" id="pauseButton" type="button" aria-label="Пауза">
          <svg viewBox="0 0 24 24" width="22" height="22"><rect x="6" y="5" width="4" height="14" rx="1.5" fill="currentColor"/><rect x="14" y="5" width="4" height="14" rx="1.5" fill="currentColor"/></svg>
        </button>
      </div>
    </div>
    <!-- Текущий вопрос: съешь правильный ответ на поле -->
    <div class="question-banner" id="questionBanner">8 × 7 = ?</div>
    <!-- Улучшения, которые уже взяли -->
    <div class="weapon" id="perkChips"></div>
  </div>

  <!-- Обратный отсчёт 3 - 2 - 1 -->
  <div class="countdown hidden" id="countdown"></div>

  <!-- Всплывающая надпись -->
  <div class="toast" id="toast"></div>

  <!-- ЭКРАН 1. Стартовое меню -->
  <section class="screen" id="menuScreen">
    <div class="card">
      <h1 class="title">
        <span class="title-top">Quiz</span>
        <span class="title-bottom">Snake</span>
      </h1>
      <p class="subtitle" id="menuInfo">Съешь правильный ответ и не попадись врагам!</p>
      <div class="best">🏆 Рекорд: <b id="menuBest">0</b></div>
      <div class="buttons">
        <button class="btn" id="startButton" type="button">Начать</button>
        <button class="btn btn-secondary" id="settingsButton" type="button">Настройки</button>
      </div>
      <div class="howto">
        <div><kbd>←</kbd> <kbd>↑</kbd> <kbd>→</kbd> <kbd>↓</kbd> или <kbd>W</kbd> <kbd>A</kbd> <kbd>S</kbd> <kbd>D</kbd> — поворот</div>
        <div>📱 Свайп в нужную сторону</div>
        <div>Съешь правильный ответ — змейка растёт. Неверный — минус жизнь</div>
        <div><kbd>Esc</kbd> — пауза</div>
      </div>
    </div>
  </section>

  <!-- ЭКРАН 2. Выбор сложности -->
  <section class="screen hidden" id="difficultyScreen">
    <div class="card card-wide">
      <h2 class="screen-title">Выбери сложность</h2>
      <div class="choice-grid" id="difficultyChoices"></div>
    </div>
  </section>

  <!-- ЭКРАН 3. Вопрос (квиз) -->
  <section class="screen hidden" id="quizScreen">
    <div class="card quiz-card" id="quizCard">
      <div class="quiz-subject" id="quizSubject">🧮 Математика</div>
      <div class="quiz-question" id="quizQuestion">8 × 7 = ?</div>
      <div class="quiz-options" id="quizOptions"></div>
      <form class="quiz-form hidden" id="quizForm" autocomplete="off">
        <input class="quiz-input" id="quizInput" type="text" placeholder="Ответ" autocomplete="off" autocapitalize="characters" spellcheck="false">
        <button class="btn btn-small" type="submit">Ответить ⏎</button>
      </form>
      <div class="quiz-hint" id="quizHint">Верный ответ — змейка растёт!</div>
    </div>
  </section>

  <!-- ЭКРАН 4. Улучшения после волны -->
  <section class="screen hidden" id="perkScreen">
    <div class="card card-wide">
      <h2 class="screen-title">Новый уровень! Выбери улучшение</h2>
      <div class="choice-grid" id="perkChoices"></div>
    </div>
  </section>

  <!-- ЭКРАН 5. Настройки -->
  <section class="screen screen-settings hidden" id="settingsScreen">
    <div class="settings-panel">
      <div class="settings-header">
        <h2>Настройки</h2>
        <button class="icon-btn" id="settingsClose" type="button" aria-label="Закрыть">✕</button>
      </div>

      <div class="setting">
        <h3>Мир</h3>
        <div class="options options-cards" id="themeOptions"></div>
      </div>

      <div class="setting">
        <h3>Предмет</h3>
        <div class="options options-cards" id="subjectOptions"></div>
      </div>

      <div class="setting">
        <h3>Как отвечать</h3>
        <div class="options options-cards" id="inputOptions"></div>
      </div>

      <div class="setting">
        <h3>Звук</h3>
        <div class="row">
          <label class="switch">
            <input type="checkbox" id="soundToggle">
            <span class="switch-slider"></span>
          </label>
          <input class="volume" type="range" id="volumeRange" min="0" max="100" step="5" aria-label="Громкость">
        </div>
      </div>

      <div class="settings-footer">
        <button class="btn btn-ghost btn-small" id="resetBestButton" type="button">Сбросить рекорд</button>
        <button class="btn btn-small" id="settingsDone" type="button">Готово</button>
      </div>
    </div>
  </section>

  <!-- ЭКРАН 6. Пауза -->
  <section class="screen hidden" id="pauseScreen">
    <div class="card">
      <h2 class="screen-title">Пауза</h2>
      <div class="buttons">
        <button class="btn" id="resumeButton" type="button">Продолжить</button>
        <button class="btn btn-secondary" id="pauseMenuButton" type="button">В меню</button>
      </div>
    </div>
  </section>

  <!-- ЭКРАН 7. Игра окончена -->
  <section class="screen hidden" id="gameOverScreen">
    <div class="card">
      <h2 class="over-title">GAME OVER</h2>
      <div class="new-record hidden" id="newRecord">Новый рекорд!</div>
      <div class="stats">
        <div class="stat"><span>Очки</span><b id="finalScore">0</b></div>
        <div class="stat"><span>Длина</span><b id="finalLength">0</b></div>
        <div class="stat"><span>Ответы</span><b id="finalAnswers">0</b></div>
      </div>
      <div class="best">🏆 Рекорд: <b id="finalBest">0</b></div>
      <div class="buttons">
        <button class="btn" id="restartButton" type="button">Рестарт</button>
        <button class="btn btn-secondary" id="overMenuButton" type="button">В меню</button>
      </div>
    </div>
  </section>

</div>

<!-- ↑↑↑ Для CodePen копируйте до этой строки (строку <script> ниже — НЕ копировать) ↑↑↑ -->

<!-- Подключаем файл с логикой игры (номер версии — как у style.css) -->
<script src="script.js?v=2"></script>
</body>
</html>
````

### `style.css`

````css
/* =========================================================
   QUIZ SNAKE — оформление (цвета, шрифты, экраны, кнопки)
   ========================================================= */

/* Шрифты Google Fonts: Russo One — заголовки и цифры, Rubik — текст */
@import url('https://fonts.googleapis.com/css2?family=Russo+One&family=Rubik:wght@400;500;700&display=swap');

/* ---------- Цвета игры (переменные) ---------- */
/* --accent JavaScript меняет под выбранный мир */
:root {
  --bg: #070b1f;
  --panel: rgba(12, 16, 40, 0.84);
  --line: rgba(255, 255, 255, 0.14);
  --text: #eef1ff;
  --dim: #a7b0dd;
  --accent: #00ffff;
  --amber: #ffb84d;
  --amber-light: #ffd27a;
  --amber-dark: #d9821c;
  --cyan: #4fd6e8;
  --cyan-dark: #1f8a99;
  --good: #39d97a;
  --bad: #ff4d6d;
  --outline: #0d1030;
}

* {
  box-sizing: border-box;
}

html,
body {
  margin: 0;
  height: 100%;
}

body {
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  background:
    radial-gradient(900px 600px at 20% -10%, #1c2560 0%, transparent 60%),
    radial-gradient(800px 600px at 100% 100%, #2a1450 0%, transparent 55%),
    var(--bg);
  color: var(--text);
  font-family: 'Rubik', sans-serif;
  user-select: none;
  -webkit-user-select: none;
  -webkit-tap-highlight-color: transparent;
}

/* ---------- Сцена и холст ---------- */
/* Размер сцены задаёт JavaScript (функция resizeGame) */
.stage {
  position: relative;
  overflow: hidden;
  box-shadow: 0 0 80px rgba(0, 0, 0, 0.6);
  touch-action: none; /* чтобы касание не прокручивало страницу */
}

#gameCanvas {
  display: block;
  width: 100%;
  height: 100%;
}

.hidden {
  display: none !important;
}

/* ---------- Панель во время игры (HUD) ---------- */
.hud {
  position: absolute;
  inset: 0;
  padding: 12px;
  pointer-events: none; /* касания проходят сквозь панель */
}

.hud-top {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 8px;
}

.hud-box {
  display: flex;
  flex-direction: column;
  padding: 6px 12px;
  border: 1px solid var(--line);
  border-radius: 14px;
  background: var(--panel);
  backdrop-filter: blur(8px);
}

.hud-label {
  color: var(--dim);
  font-size: 11px;
  font-weight: 700;
  letter-spacing: 0.04em;
  text-transform: uppercase;
  white-space: nowrap;
}

.hud-box b {
  font-family: 'Russo One', sans-serif;
  font-size: 22px;
  font-weight: 400;
  line-height: 1.15;
}

.hud-hp {
  min-width: 140px;
}

.hud-hp b {
  font-size: 14px;
}

.hud-center {
  align-items: center;
}

.hud-center b {
  color: var(--accent);
  text-shadow: 0 0 12px var(--accent);
}

.hud-right {
  display: flex;
  gap: 8px;
  align-items: flex-start;
}

.bar {
  height: 10px;
  margin: 5px 0 3px;
  border-radius: 999px;
  background: rgba(255, 255, 255, 0.12);
  overflow: hidden;
}

.bar-fill {
  width: 100%;
  height: 100%;
  border-radius: 999px;
  background: linear-gradient(90deg, var(--bad), #ff9f43, var(--good));
  background-size: 300% 100%;
  transition: width 0.25s;
}

.weapon {
  position: absolute;
  left: 12px;
  bottom: 12px;
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
  max-width: calc(100% - 24px);
}

.chip {
  padding: 4px 10px;
  border: 1px solid var(--line);
  border-radius: 999px;
  background: var(--panel);
  font-size: 12px;
  font-weight: 500;
  white-space: nowrap;
}

.boss-bar {
  position: absolute;
  top: 78px;
  left: 50%;
  width: min(420px, 80%);
  transform: translateX(-50%);
  text-align: center;
  font-family: 'Russo One', sans-serif;
  font-size: 14px;
  color: var(--bad);
  text-shadow: 0 0 10px var(--bad);
}

.boss-bar .bar {
  height: 12px;
  border: 1px solid rgba(255, 77, 109, 0.6);
}

.boss-bar .bar-fill {
  background: linear-gradient(90deg, #ff2a5c, #ff7a45);
}

.icon-btn {
  pointer-events: auto;
  display: grid;
  place-items: center;
  width: 44px;
  height: 44px;
  border: 1px solid var(--line);
  border-radius: 14px;
  background: var(--panel);
  color: #fff;
  font-size: 18px;
  cursor: pointer;
  backdrop-filter: blur(8px);
}

/* ---------- Экраны ---------- */
.screen {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 14px;
  background: radial-gradient(circle at 50% 45%, rgba(7, 11, 31, 0.3), rgba(7, 11, 31, 0.75));
  animation: fadeIn 0.2s ease;
}

.card {
  width: min(400px, 100%);
  max-height: 100%;
  overflow-y: auto;
  padding: 24px 22px;
  border: 1px solid var(--line);
  border-radius: 26px;
  background: var(--panel);
  box-shadow: 0 24px 60px -20px rgba(0, 0, 0, 0.7), 0 0 0 1px rgba(255, 255, 255, 0.02);
  backdrop-filter: blur(10px);
  text-align: center;
  animation: rise 0.3s cubic-bezier(0.2, 0.9, 0.3, 1.2);
}

.card-wide {
  width: min(720px, 100%);
}

/* Заголовок игры с неоновым свечением в цвет мира */
.title {
  margin: 0 0 6px;
  font-family: 'Russo One', sans-serif;
  font-weight: 400;
  line-height: 0.95;
}

.title span {
  display: block;
  -webkit-text-stroke: 5px var(--outline);
  paint-order: stroke fill;
}

.title-top {
  font-size: 52px;
  color: #fff;
  text-shadow: 0 0 16px var(--accent), 0 6px 0 var(--outline);
}

.title-bottom {
  font-size: 58px;
  color: var(--accent);
  text-shadow: 0 0 24px var(--accent), 0 6px 0 var(--outline);
}

.subtitle {
  margin: 12px 0 14px;
  color: var(--dim);
  font-size: 15px;
}

.best {
  display: inline-block;
  margin-bottom: 18px;
  padding: 6px 14px;
  border-radius: 999px;
  background: rgba(255, 184, 77, 0.12);
  color: var(--amber);
  font-weight: 500;
}

.screen-title {
  margin: 0 0 16px;
  font-family: 'Russo One', sans-serif;
  font-size: 28px;
  font-weight: 400;
}

.buttons {
  display: flex;
  flex-direction: column;
  gap: 14px;
}

.howto {
  display: grid;
  gap: 8px;
  margin-top: 20px;
  color: var(--dim);
  font-size: 13px;
}

kbd {
  padding: 2px 7px;
  border-radius: 6px;
  background: rgba(255, 255, 255, 0.1);
  border-bottom: 2px solid rgba(255, 255, 255, 0.22);
  color: var(--text);
  font-family: inherit;
  font-size: 12px;
}

/* ---------- Кнопки (объёмные, «нажимаются» вниз) ---------- */
.btn {
  padding: 14px 28px;
  border: 0;
  border-radius: 16px;
  background: linear-gradient(180deg, var(--amber-light), var(--amber));
  box-shadow: 0 6px 0 var(--amber-dark), 0 12px 24px -6px rgba(0, 0, 0, 0.5);
  color: #3b2300;
  font-family: 'Russo One', sans-serif;
  font-size: 20px;
  cursor: pointer;
  transition: transform 0.1s, box-shadow 0.1s, filter 0.15s;
}

.btn:hover {
  filter: brightness(1.07);
}

.btn:active {
  transform: translateY(4px);
  box-shadow: 0 2px 0 var(--amber-dark);
}

.btn-secondary {
  background: linear-gradient(180deg, #7ae9f6, var(--cyan));
  box-shadow: 0 6px 0 var(--cyan-dark), 0 12px 24px -6px rgba(0, 0, 0, 0.5);
  color: #05303a;
}

.btn-secondary:active {
  box-shadow: 0 2px 0 var(--cyan-dark);
}

.btn-ghost {
  background: rgba(255, 255, 255, 0.08);
  box-shadow: none;
  color: var(--dim);
}

.btn-ghost:active {
  box-shadow: none;
}

.btn-small {
  padding: 11px 18px;
  font-size: 15px;
}

/* ---------- Карточки выбора: герои, сложность, улучшения ---------- */
.choice-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(170px, 1fr));
  gap: 12px;
}

.choice {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 6px;
  padding: 16px 12px;
  border: 2px solid var(--line);
  border-radius: 20px;
  background: rgba(255, 255, 255, 0.05);
  color: var(--text);
  font-family: inherit;
  cursor: pointer;
  transition: transform 0.15s, border-color 0.15s, box-shadow 0.15s;
}

.choice:hover,
.choice:focus-visible {
  transform: translateY(-4px);
  border-color: var(--accent);
  box-shadow: 0 0 22px -4px var(--accent);
  outline: none;
}

.choice-icon {
  font-size: 46px;
  line-height: 1.1;
}

.choice-title {
  font-family: 'Russo One', sans-serif;
  font-size: 18px;
}

.choice-desc {
  color: var(--dim);
  font-size: 13px;
  line-height: 1.35;
}

.choice-stats {
  display: grid;
  gap: 3px;
  width: 100%;
  margin-top: 4px;
  font-size: 12px;
  text-align: left;
}

.stat-row {
  display: flex;
  align-items: center;
  gap: 6px;
}

.stat-row span {
  width: 64px;
  color: var(--dim);
}

.stat-dots {
  letter-spacing: 1px;
  color: var(--accent);
}

/* ---------- Окно вопроса ---------- */
.quiz-card {
  width: min(460px, 100%);
  border-color: var(--accent);
  box-shadow: 0 0 40px -10px var(--accent), 0 24px 60px -20px rgba(0, 0, 0, 0.7);
}

.quiz-subject {
  color: var(--dim);
  font-size: 14px;
  font-weight: 700;
  letter-spacing: 0.06em;
  text-transform: uppercase;
}

.quiz-question {
  margin: 14px 0 18px;
  font-family: 'Russo One', sans-serif;
  font-size: 40px;
  letter-spacing: 0.04em;
  word-break: break-word;
}

.quiz-question small {
  display: block;
  margin-top: 6px;
  color: var(--dim);
  font-family: 'Rubik', sans-serif;
  font-size: 15px;
  letter-spacing: 0;
}

.quiz-options {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
}

.quiz-option {
  min-height: 64px;
  padding: 12px;
  border: 2px solid var(--line);
  border-radius: 18px;
  background: rgba(255, 255, 255, 0.07);
  color: #fff;
  font-family: 'Russo One', sans-serif;
  font-size: 24px;
  cursor: pointer;
  transition: transform 0.12s, border-color 0.12s, background 0.12s;
}

.quiz-option:hover {
  transform: translateY(-3px);
  border-color: var(--accent);
  background: rgba(255, 255, 255, 0.12);
}

.quiz-option.right {
  border-color: var(--good);
  background: rgba(57, 217, 122, 0.3);
}

.quiz-option.wrong {
  border-color: var(--bad);
  background: rgba(255, 77, 109, 0.3);
}

.quiz-form {
  display: flex;
  gap: 10px;
}

.quiz-input {
  flex: 1;
  min-width: 0;
  padding: 14px 16px;
  border: 2px solid var(--accent);
  border-radius: 16px;
  outline: none;
  background: rgba(0, 0, 0, 0.35);
  color: #fff;
  font-family: 'Russo One', sans-serif;
  font-size: 24px;
  text-align: center;
  text-transform: uppercase;
}

.quiz-hint {
  margin-top: 14px;
  color: var(--dim);
  font-size: 13px;
}

/* Окно мигает красным при неверном ответе и зелёным при верном */
.quiz-card.flash-bad {
  animation: flashBad 0.6s ease;
}

.quiz-card.flash-good {
  animation: flashGood 0.5s ease;
}

/* ---------- Экран настроек ---------- */
.screen-settings {
  justify-content: flex-end;
  align-items: stretch;
  background: linear-gradient(90deg, transparent 30%, rgba(7, 11, 31, 0.6));
}

.settings-panel {
  width: min(420px, 100%);
  max-height: 100%;
  overflow-x: hidden;
  overflow-y: auto;
  padding: 18px 20px 20px;
  border: 1px solid var(--line);
  border-radius: 24px;
  background: var(--panel);
  backdrop-filter: blur(10px);
  animation: slideIn 0.3s ease;
}

.settings-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.settings-header h2 {
  margin: 0;
  font-family: 'Russo One', sans-serif;
  font-size: 24px;
  font-weight: 400;
}

.setting {
  padding: 12px 0;
  border-bottom: 1px solid var(--line);
}

.setting h3 {
  margin: 0 0 10px;
  color: var(--dim);
  font-size: 12px;
  font-weight: 700;
  letter-spacing: 0.06em;
  text-transform: uppercase;
}

.row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
}

.options {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.options-cards .option {
  flex: 1;
  min-width: 100px;
}

.option {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
  padding: 10px 8px;
  border: 2px solid transparent;
  border-radius: 14px;
  background: rgba(255, 255, 255, 0.05);
  color: var(--dim);
  font-family: inherit;
  font-size: 12px;
  cursor: pointer;
  transition: transform 0.15s, border-color 0.15s, background 0.15s;
}

.option:hover {
  transform: translateY(-2px);
}

.option.selected {
  border-color: var(--accent);
  background: rgba(255, 255, 255, 0.1);
  color: #fff;
}

.option-icon {
  font-size: 26px;
}

.option-title {
  color: #fff;
  font-size: 14px;
  font-weight: 700;
}

.settings-footer {
  display: flex;
  justify-content: space-between;
  gap: 10px;
  padding-top: 16px;
}

/* Переключатель (вкл/выкл) */
.switch {
  position: relative;
  flex-shrink: 0;
  width: 52px;
  height: 30px;
  cursor: pointer;
}

.switch input {
  position: absolute;
  opacity: 0;
}

.switch-slider {
  position: absolute;
  inset: 0;
  border-radius: 999px;
  background: rgba(255, 255, 255, 0.15);
  transition: background 0.2s;
}

.switch-slider::before {
  content: '';
  position: absolute;
  top: 4px;
  left: 4px;
  width: 22px;
  height: 22px;
  border-radius: 50%;
  background: #fff;
  transition: transform 0.2s;
}

.switch input:checked + .switch-slider {
  background: var(--amber);
}

.switch input:checked + .switch-slider::before {
  transform: translateX(22px);
}

.volume {
  flex: 1;
  accent-color: var(--amber);
}

.volume:disabled {
  opacity: 0.35;
}

/* ---------- Экран «Игра окончена» ---------- */
.over-title {
  margin: 0 0 14px;
  font-family: 'Russo One', sans-serif;
  font-size: 46px;
  font-weight: 400;
  color: var(--bad);
  -webkit-text-stroke: 4px var(--outline);
  paint-order: stroke fill;
  text-shadow: 0 0 24px var(--bad);
}

.stats {
  display: flex;
  gap: 10px;
  margin-bottom: 14px;
}

.stat {
  flex: 1;
  padding: 10px 6px;
  border-radius: 16px;
  background: rgba(255, 255, 255, 0.07);
}

.stat span {
  display: block;
  color: var(--dim);
  font-size: 12px;
}

.stat b {
  font-family: 'Russo One', sans-serif;
  font-size: 28px;
  font-weight: 400;
}

.new-record {
  display: inline-block;
  margin-bottom: 14px;
  padding: 6px 16px;
  border-radius: 999px;
  background: linear-gradient(90deg, var(--accent), var(--amber));
  color: #10132e;
  font-weight: 700;
  animation: pulse 1s ease-in-out infinite;
}

/* ---------- Обратный отсчёт и всплывающая надпись ---------- */
.countdown {
  position: absolute;
  inset: 0;
  display: grid;
  place-items: center;
  pointer-events: none;
  font-family: 'Russo One', sans-serif;
  font-size: 150px;
  color: #fff;
  -webkit-text-stroke: 6px var(--outline);
  paint-order: stroke fill;
  text-shadow: 0 0 30px var(--accent), 0 8px 0 var(--outline);
}

.countdown span {
  animation: countPop 0.7s ease-out both;
}

.countdown .go {
  font-size: 90px;
  color: var(--accent);
}

.toast {
  position: absolute;
  top: 26%;
  left: 50%;
  transform: translateX(-50%);
  opacity: 0;
  pointer-events: none;
  white-space: nowrap;
  font-family: 'Russo One', sans-serif;
  font-size: 30px;
  color: #fff;
  -webkit-text-stroke: 4px var(--outline);
  paint-order: stroke fill;
  text-shadow: 0 0 18px var(--accent);
}

.toast.show {
  animation: toast 1.6s ease both;
}

/* ---------- Анимации ---------- */
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes rise {
  from { opacity: 0; transform: translateY(24px) scale(0.96); }
  to { opacity: 1; transform: none; }
}

@keyframes slideIn {
  from { opacity: 0; transform: translateX(40px); }
  to { opacity: 1; transform: none; }
}

@keyframes pulse {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.07); }
}

@keyframes flashBad {
  0%, 100% { background: var(--panel); }
  20%, 60% { background: rgba(255, 77, 109, 0.75); }
  40%, 80% { background: rgba(255, 77, 109, 0.25); }
}

@keyframes flashGood {
  0%, 100% { background: var(--panel); }
  40% { background: rgba(57, 217, 122, 0.6); }
}

@keyframes countPop {
  0% { opacity: 0; transform: scale(2.2); }
  30% { opacity: 1; transform: scale(1); }
  80% { opacity: 1; transform: scale(0.95); }
  100% { opacity: 0; transform: scale(0.7); }
}

@keyframes toast {
  0% { opacity: 0; transform: translate(-50%, 20px) scale(0.8); }
  15% { opacity: 1; transform: translate(-50%, 0) scale(1.05); }
  25% { transform: translate(-50%, 0) scale(1); }
  80% { opacity: 1; }
  100% { opacity: 0; transform: translate(-50%, -20px); }
}

/* ---------- Телефоны ---------- */
@media (max-width: 640px) {
  .screen-settings {
    align-items: flex-end;
    padding: 10px;
  }

  .settings-panel {
    max-height: 80%;
  }

  .hud-hp {
    min-width: 110px;
  }

  .hud-box b {
    font-size: 18px;
  }

  .title-top { font-size: 42px; }
  .title-bottom { font-size: 46px; }
  .quiz-question { font-size: 32px; }
  .quiz-option { font-size: 20px; }
  .choice-grid { grid-template-columns: 1fr; }
  .choice { flex-direction: row; text-align: left; }
  .choice-icon { font-size: 38px; }
  .countdown { font-size: 120px; }
}

/* Если в системе включено «уменьшить движение» */
@media (prefers-reduced-motion: reduce) {
  .new-record {
    animation: none;
  }
}

/* =========================================================
   ДОПОЛНЕНИЯ ДЛЯ ЗМЕЙКИ
   ========================================================= */

/* Вопрос на поле — крупная плашка сверху по центру */
.question-banner {
  position: absolute;
  top: 10px;
  left: 50%;
  max-width: 46%;
  padding: 6px 16px;
  border: 2px solid var(--accent);
  border-radius: 16px;
  background: var(--panel);
  box-shadow: 0 0 20px -4px var(--accent);
  transform: translateX(-50%);
  font-family: 'Russo One', sans-serif;
  font-size: 24px;
  text-align: center;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.question-banner small {
  display: block;
  color: var(--dim);
  font-family: 'Rubik', sans-serif;
  font-size: 11px;
}

.hud-left,
.hud-right {
  display: flex;
  gap: 8px;
  align-items: flex-start;
}

.hearts {
  font-size: 20px;
  letter-spacing: 2px;
}

@media (max-width: 700px) {
  .hud {
    padding: 6px;
  }

  .hud-box {
    padding: 3px 7px;
  }

  .hud-label {
    font-size: 8px;
  }

  .hud-box b,
  .hearts {
    font-size: 14px;
  }

  .question-banner {
    top: 46px;
    max-width: 90%;
    font-size: 18px;
  }

  .icon-btn {
    width: 34px;
    height: 34px;
  }
}

/* =========================================================
   РОВНАЯ ВЕРХНЯЯ ПАНЕЛЬ И ЭКРАНЫ
   ========================================================= */

/* Сетка: левая и правая колонки одинаковой ширины (1fr),
   поэтому средний блок стоит точно по центру */
.hud-top {
  display: grid;
  grid-template-columns: 1fr 1fr;
  align-items: start;
}

.hud-top > :first-child {
  justify-self: start;
}

.hud-top > :last-child {
  justify-self: end;
}

.hud-left,
.hud-right {
  display: flex;
  gap: 8px;
  align-items: stretch;
}

/* Все блоки одной высоты, содержимое по центру */
.hud-top .hud-box {
  justify-content: center;
  min-width: 80px;
  height: 54px;
  text-align: center;
}

.hud-top .hud-label {
  line-height: 16px;
}

.hud-top .icon-btn {
  width: 54px;
  height: 54px;
}

/* Экраны (меню, вопрос, улучшения, настройки…) — на всё окно,
   даже если игровое поле на телефоне узкой полосой */
.screen {
  position: fixed;
}

/* Длинные подсказки переносятся, а не обрезаются краем экрана */
.toast {
  width: max-content;
  max-width: 92%;
  white-space: normal;
  text-align: center;
  line-height: 1.15;
}

@media (max-width: 700px) {
  .hud-top .hud-box {
    min-width: 0;
    height: 38px;
    padding: 2px 8px;
  }

  .hud-top .hud-label {
    line-height: 12px;
  }

  .hud-top .icon-btn {
    width: 38px;
    height: 38px;
  }

  .hud-left,
  .hud-right {
    gap: 4px;
  }
}

.question-banner {
  top: 12px;
  min-height: 54px;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

@media (max-width: 700px) {
  .question-banner {
    top: 52px;
    min-height: 0;
  }
}
````

### `script.js`

````javascript
/* =========================================================
   QUIZ SNAKE — змейка + образовательный квиз + улучшения
   Чистый JavaScript и Canvas, графика из эмодзи
   ---------------------------------------------------------
   Как устроен файл (читайте сверху вниз):
     0. Лаборатория: 10 настроек для экспериментов
     1. Данные: миры, предметы, сложность
     2. Вопросы: математика, русский, английский
     3. Улучшения (перки)
     4. Настройки игрока и рекорд
     5. Звуки
     6. Холст и поле
     7. Состояние игры, змейка, ответы, враги
     8. Рисование: фон, змейка, ответы, частицы
     9. Окно вопроса (для режима «Ввод»)
    10. Экран улучшений
    11. Игровые действия
    12. Обновление мира за один кадр
    13. Рисование одного кадра
    14. Экраны и интерфейс
    15. Экран настроек
    16. Управление
    17. Игровой цикл и запуск
   ========================================================= */


/* ---------------------------------------------------------
   0. ЛАБОРАТОРИЯ ИГРЫ 🧪
   Поменяй одно значение, и игра изменится!
   Сейчас здесь обычные значения: игра такая, как задумано.
   --------------------------------------------------------- */

const LAB = {
  // 1. Название игры в меню: верхнее и нижнее слово (до 8 букв)
  titleTop: 'Quiz',
  titleBottom: 'Snake',

  // 2. Своя голова змейки-эмодзи, например '🐸'. Пусто '' — голова мира
  snakeHead: '',

  // 3. Длина змейки на старте: 4 — обычно, 10 — длинная
  startLength: 4,

  // 4. Жизни: 3 — обычно, 10 — много
  lives: 3,

  // 5. Скорость змейки: 1 — обычная, 0.6 — медленная, 1.5 — быстрая
  snakeSpeed: 1,

  // 6. Сколько врагов может быть на поле: 4 — обычно, 0 — без врагов
  maxEnemies: 4,

  // 7. На сколько змейка растёт за верный ответ: 2 — обычно, 5 — сильно
  growPerAnswer: 2,

  // 8. Сквозь стены: false — стена = минус жизнь, true — выход с другой стороны
  wrapWalls: false,

  // 9. Скорость врагов: 1 — обычная, 0.5 — медленные
  enemySpeed: 1,

  // 10. Радужная змейка: false — цвет мира, true — радуга
  rainbowSnake: false
};


/* ---------------------------------------------------------
   1. ДАННЫЕ: МИРЫ, ПРЕДМЕТЫ, СЛОЖНОСТЬ
   --------------------------------------------------------- */

const THEMES = [
  {
    id: 'fantasy', icon: '🏰', label: 'Фэнтези', desc: 'Рыцари и магия',
    accent: '#FFD700', body: '#d9a600',
    sky: ['#0b1030', '#1b1f5a'], board: 'rgba(40,46,110,0.55)', cellLight: 'rgba(255,255,255,0.035)',
    head: '🐲', enemies: ['👹', '💀'], boss: { emoji: '🐉', name: 'Дракон' }
  },
  {
    id: 'space', icon: '🚀', label: 'Космос', desc: 'Лазеры и НЛО',
    accent: '#00FFFF', body: '#00a8c0',
    sky: ['#000008', '#0d0a2e'], board: 'rgba(10,20,60,0.6)', cellLight: 'rgba(0,255,255,0.04)',
    head: '🤖', enemies: ['☄️', '🛸'], boss: { emoji: '👾', name: 'Мега-Мозг' }
  },
  {
    id: 'eco', icon: '🌿', label: 'Эко-Панк', desc: 'Защита природы',
    accent: '#39FF14', body: '#22b30c',
    sky: ['#04140c', '#123d22'], board: 'rgba(10,50,25,0.6)', cellLight: 'rgba(57,255,20,0.04)',
    head: '🐛', enemies: ['🗑️', '🦗'], boss: { emoji: '🧟', name: 'Токсичный Монстр' }
  }
];

const SUBJECTS = [
  { id: 'math', icon: '🧮', label: 'Математика', desc: 'Примеры' },
  { id: 'rus',  icon: '📝', label: 'Русский', desc: 'Пропущенная буква' },
  { id: 'eng',  icon: '🇬🇧', label: 'Английский', desc: 'Перевод слов' }
];

// «Кнопки» — 4 ответа лежат на поле, съешь правильный.
// «Ввод» — съешь ❓ и напечатай ответ
const INPUT_MODES = [
  { id: 'buttons',  icon: '👆', label: 'Кнопки', desc: 'Ответы на поле' },
  { id: 'keyboard', icon: '⌨️', label: 'Ввод', desc: 'Напечатать ответ' }
];

// Сложность: через сколько секунд змейка делает шаг и скорость врагов
const DIFFICULTIES = [
  { id: 'easy',   icon: '🙂', label: 'Легко',     desc: 'Спокойная змейка',         step: 0.18, enemy: 0.75, score: 1 },
  { id: 'normal', icon: '😎', label: 'Нормально', desc: 'Быстрее, очки ×1.5',        step: 0.14, enemy: 0.6,  score: 1.5 },
  { id: 'hard',   icon: '😈', label: 'Сложно',    desc: 'Очень быстро, очки ×2',     step: 0.11, enemy: 0.45, score: 2 }
];

function findById(list, id) {
  return list.find(item => item.id === id) || list[0];
}

function currentTheme() {
  return findById(THEMES, settings.theme);
}

function difficulty() {
  return findById(DIFFICULTIES, settings.difficulty);
}


/* ---------------------------------------------------------
   2. ВОПРОСЫ
   Для каждого предмета — пул заданий. Задания перемешиваются
   и идут по очереди, поэтому подряд не повторяются.
   --------------------------------------------------------- */

// Русский язык: слово, номер пропущенной буквы (с нуля)
const RUS_WORDS = [
  ['МОЛОКО', 1], ['СОБАКА', 1], ['КОРОВА', 1], ['ВОРОНА', 1], ['БЕРЁЗА', 1],
  ['ЗАЯЦ', 1], ['ПЕТУХ', 1], ['ЛИСИЦА', 1], ['МЕДВЕДЬ', 1], ['КАРАНДАШ', 1],
  ['ТЕТРАДЬ', 1], ['ПЕНАЛ', 1], ['ЯГОДА', 3], ['МАГАЗИН', 1], ['ГОРОД', 1],
  ['УЧЕНИК', 2], ['ДЕЖУРНЫЙ', 1], ['РЕБЯТА', 1], ['КАПУСТА', 1], ['ПОГОДА', 1],
  ['СОРОКА', 1], ['ВЕТЕР', 1], ['МОРКОВЬ', 1], ['ПАЛЬТО', 1], ['ОГУРЕЦ', 2],
  ['ХОРОШО', 1], ['ДЕВОЧКА', 1], ['ЯЗЫК', 1]
];
const VOWELS = ['А', 'О', 'Е', 'И', 'Я', 'У'];

// Английский: слово и перевод
const ENG_WORDS = [
  ['APPLE', 'Яблоко'], ['DOG', 'Собака'], ['CAT', 'Кошка'], ['SPACE', 'Космос'],
  ['STAR', 'Звезда'], ['SUN', 'Солнце'], ['MOON', 'Луна'], ['HOUSE', 'Дом'],
  ['BOOK', 'Книга'], ['WATER', 'Вода'], ['TREE', 'Дерево'], ['BIRD', 'Птица'],
  ['FISH', 'Рыба'], ['SCHOOL', 'Школа'], ['FRIEND', 'Друг'], ['MOTHER', 'Мама'],
  ['FATHER', 'Папа'], ['GREEN', 'Зелёный'], ['RED', 'Красный'], ['BIG', 'Большой'],
  ['MILK', 'Молоко'], ['BREAD', 'Хлеб'], ['SKY', 'Небо'], ['EARTH', 'Земля'],
  ['FLOWER', 'Цветок'], ['CAR', 'Машина']
];

let questionPool = [];

function shuffle(list) {
  const copy = list.slice();
  for (let i = copy.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [copy[i], copy[j]] = [copy[j], copy[i]];
  }
  return copy;
}

function randInt(min, max) {
  return Math.floor(min + Math.random() * (max - min + 1));
}

// Один пример по математике. Чем сложнее игра — тем больше числа
function makeMathQuestion(level) {
  const kind = randInt(0, 2);
  let a, b, text, answer;
  if (kind === 0) {
    a = randInt(2, 10 + level * 20);
    b = randInt(2, 10 + level * 20);
    text = a + ' + ' + b + ' = ?';
    answer = a + b;
  } else if (kind === 1) {
    a = randInt(5, 15 + level * 20);
    b = randInt(1, a);           // без отрицательных ответов
    text = a + ' − ' + b + ' = ?';
    answer = a - b;
  } else {
    a = randInt(2, 5 + level * 2);
    b = randInt(2, 9);
    text = a + ' × ' + b + ' = ?';
    answer = a * b;
  }
  // Неверные варианты — близкие числа
  const wrong = new Set();
  while (wrong.size < 3) {
    const w = answer + randInt(-10, 10);
    if (w !== answer && w >= 0) wrong.add(w);
  }
  return { text: text, hint: '', answer: String(answer), options: shuffle([answer, ...wrong].map(String)), numeric: true };
}

function makeRusQuestion([word, index]) {
  const letter = word[index] === 'Ё' ? 'Е' : word[index];
  const text = word.slice(0, index) + '_' + word.slice(index + 1);
  const wrong = shuffle(VOWELS.filter(v => v !== letter)).slice(0, 3);
  return { text: text, hint: 'Какая буква пропущена?', answer: letter, options: shuffle([letter, ...wrong]) };
}

function makeEngQuestion([word, translation]) {
  const wrong = shuffle(ENG_WORDS.filter(w => w[1] !== translation)).slice(0, 3).map(w => w[1]);
  return { text: word, hint: 'Переведи на русский', answer: translation, options: shuffle([translation, ...wrong]) };
}

// Пул из 20+ заданий для выбранного предмета
function buildQuestionPool() {
  const subject = settings.subject;
  if (subject === 'rus') {
    questionPool = shuffle(RUS_WORDS).map(makeRusQuestion);
  } else if (subject === 'eng') {
    questionPool = shuffle(ENG_WORDS).map(makeEngQuestion);
  } else {
    const level = DIFFICULTIES.indexOf(findById(DIFFICULTIES, settings.difficulty));
    questionPool = [];
    for (let i = 0; i < 20; i++) questionPool.push(makeMathQuestion(level));
  }
}

function nextQuestion() {
  if (questionPool.length === 0) buildQuestionPool();
  return questionPool.pop();
}

// Сравнение ответов: без пробелов, без разницы в регистре, Ё = Е
function normalize(text) {
  return String(text).trim().toUpperCase().replace(/Ё/g, 'Е');
}


/* ---------------------------------------------------------
   3. УЛУЧШЕНИЯ (ПЕРКИ)
   Каждые 3 верных ответа — новый уровень и выбор улучшения.
   Каждое можно взять один раз за игру
   --------------------------------------------------------- */

const PERKS = [
  { id: 'slow',   icon: '🐢', title: 'Спокойнее',        desc: 'Змейка на 15% медленнее',           apply: p => { p.stepBonus *= 1.15; } },
  { id: 'life',   icon: '❤️', title: '+1 жизнь',         desc: 'Ещё одна попытка',                  apply: p => { game.lives++; } },
  { id: 'wrap',   icon: '🌀', title: 'Сквозь стены',     desc: 'Выход с другой стороны поля',       apply: p => { p.wrap = true; } },
  { id: 'shield', icon: '🛡️', title: 'Щит',              desc: 'Один удар врага не страшен',        apply: p => { p.shield = true; } },
  { id: 'hint',   icon: '✂️', title: 'Подсказка 50/50',  desc: 'Неверных ответов на поле меньше',   apply: p => { p.hint = true; } },
  { id: 'score',  icon: '⭐', title: 'Двойные очки',     desc: 'Очки ×2',                           apply: p => { p.scoreMult *= 2; } },
  { id: 'freeze', icon: '❄️', title: 'Заморозка',        desc: 'Враги ходят медленнее',             apply: p => { p.enemySlow *= 1.6; } },
  { id: 'fruit',  icon: '🍎', title: 'Фрукты',           desc: 'На поле появляются яблоки',         apply: p => { p.fruit = true; } },
  { id: 'armor',  icon: '🦴', title: 'Гибкий хвост',     desc: 'Можно проходить сквозь себя',       apply: p => { p.armor = true; } },
  { id: 'hunter', icon: '🍽️', title: 'Охотник',          desc: 'Можно съедать врагов',              apply: p => { p.hunter = true; } },
  { id: 'safe',   icon: '⏳', title: 'Долгая защита',    desc: 'После промаха 4 с неуязвимости',   apply: p => { p.safeTime = 4; } },
  { id: 'short',  icon: '✂️', title: 'Короче хвост',     desc: 'Змейка короче на 3',                apply: p => { snake.splice(Math.max(3, snake.length - 3)); } }
];


/* ---------------------------------------------------------
   4. НАСТРОЙКИ ИГРОКА И РЕКОРД
   --------------------------------------------------------- */

const DEFAULT_SETTINGS = {
  theme: 'eco',
  subject: 'math',
  inputMode: 'buttons',
  difficulty: 'easy',
  sound: true,
  volume: 0.7
};

let settings = loadSettings();
let bestScore = loadBestScore();

function loadSettings() {
  try {
    const saved = JSON.parse(localStorage.getItem('quizsnake-settings'));
    return Object.assign({}, DEFAULT_SETTINGS, saved);
  } catch (error) {
    return Object.assign({}, DEFAULT_SETTINGS);
  }
}

function saveSettings() {
  try {
    localStorage.setItem('quizsnake-settings', JSON.stringify(settings));
  } catch (error) {
    // Если браузер запретил сохранение — просто играем без него
  }
}

function loadBestScore() {
  try {
    return Number(localStorage.getItem('quizsnake-best')) || 0;
  } catch (error) {
    return 0;
  }
}

function saveBestScore() {
  try {
    localStorage.setItem('quizsnake-best', String(bestScore));
  } catch (error) {
    // ничего страшного
  }
}


/* ---------------------------------------------------------
   5. ЗВУКИ (Web Audio — звук играет мгновенно)
   --------------------------------------------------------- */

const isCodePen = location.hostname.includes('codepen') || location.hostname.includes('cdpn.io');
const SOUND_PATH = isCodePen ? 'https://si0683.github.io/QuizSnake/sound/' : 'sound/';

const SOUND_FILES = {
  jump:  'jump.wav',   // съели фрукт или врага
  coin:  'coin.mp3',   // верный ответ
  tick:  'tick.wav',   // отсчёт
  start: 'start.wav',  // новый уровень, босс
  crash: 'crash.wav'   // минус жизнь
};

const SOUND_START = { coin: 0.045 };

const audioContext = new (window.AudioContext || window.webkitAudioContext)();
const sounds = {};

for (const name in SOUND_FILES) {
  fetch(SOUND_PATH + SOUND_FILES[name])
    .then(response => response.arrayBuffer())
    .then(data => audioContext.decodeAudioData(data))
    .then(buffer => { sounds[name] = buffer; })
    .catch(() => {});
}

function playSound(name) {
  if (!settings.sound || !sounds[name]) return;
  if (audioContext.state === 'suspended') audioContext.resume();
  const source = audioContext.createBufferSource();
  source.buffer = sounds[name];
  const volume = audioContext.createGain();
  volume.gain.value = settings.volume;
  source.connect(volume);
  volume.connect(audioContext.destination);
  source.start(0, SOUND_START[name] || 0);
}


/* ---------------------------------------------------------
   6. ХОЛСТ И ПОЛЕ
   Поле — сетка 20 × 13 клеток по 40 «игровых пикселей».
   Сверху полоса 64 пикселя для панели с вопросом.
   --------------------------------------------------------- */

const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
const stage = document.getElementById('stage');

const CELL = 40;
const COLS = 20;
const ROWS = 13;
const TOP = 64;                 // отступ сверху под панель
const W = COLS * CELL;          // 800
const H = TOP + ROWS * CELL;    // 584
let scale = 1;

function resizeGame() {
  scale = Math.min(window.innerWidth / W, window.innerHeight / H);
  stage.style.width = W * scale + 'px';
  stage.style.height = H * scale + 'px';
  const ratio = window.devicePixelRatio || 1;
  canvas.width = Math.round(W * scale * ratio);
  canvas.height = Math.round(H * scale * ratio);
  ctx.setTransform(scale * ratio, 0, 0, scale * ratio, 0, 0);
}

// Центр клетки в пикселях
function cellX(c) { return c * CELL + CELL / 2; }
function cellY(r) { return TOP + r * CELL + CELL / 2; }


/* ---------------------------------------------------------
   7. СОСТОЯНИЕ ИГРЫ, ЗМЕЙКА, ОТВЕТЫ, ВРАГИ
   --------------------------------------------------------- */

// Состояния: 'menu', 'choose', 'countdown', 'playing',
// 'quiz' (вопрос в режиме «Ввод»), 'perk', 'paused', 'over'
const game = {
  state: 'menu',
  score: 0,
  lives: 0,         // ставится из LAB.lives при старте
  level: 1,
  correct: 0,        // верных ответов
  time: 0,
  stepTimer: 0,
  grow: 0,           // на сколько клеток змейка ещё вырастет
  safe: 0,           // секунды неуязвимости
  flash: 0,          // красная вспышка поля
  usedPerks: []
};

let perks = null;    // эффекты взятых улучшений

function resetPerks() {
  perks = {
    stepBonus: 1, wrap: LAB.wrapWalls, shield: false, hint: false, scoreMult: 1,
    enemySlow: 1, fruit: false, armor: false, hunter: false, safeTime: 2
  };
}

let snake = [];        // клетки змейки, [0] — голова: { c, r }
let direction = { x: 1, y: 0 };
let turnQueue = [];    // повороты, нажатые между шагами
let tokens = [];       // ответы на поле: { c, r, text, correct }
let enemies = [];      // { c, r, emoji, timer, boss, life }
let fruits = [];       // { c, r }
let particles = [];
let currentQuestion = null;

function headEmoji() {
  return LAB.snakeHead || currentTheme().head;
}

// Свободна ли клетка (нет змейки, ответов, врагов)
function isFree(c, r) {
  return !snake.some(s => s.c === c && s.r === r) &&
    !tokens.some(t => t.c === c && t.r === r) &&
    !enemies.some(e => e.c === c && e.r === r) &&
    !fruits.some(f => f.c === c && f.r === r);
}

// Случайная свободная клетка подальше от головы
function randomFreeCell(minDistance) {
  const head = snake[0] || { c: 10, r: 6 };
  for (let tries = 0; tries < 300; tries++) {
    const c = Math.floor(Math.random() * COLS);
    const r = Math.floor(Math.random() * ROWS);
    const far = Math.abs(c - head.c) + Math.abs(r - head.r) >= minDistance;
    if (far && isFree(c, r)) return { c: c, r: r };
  }
  return { c: 0, r: 0 };
}

function placeSnake(length) {
  snake = [];
  const r = Math.floor(ROWS / 2);
  for (let i = 0; i < length; i++) snake.push({ c: 6 - i, r: r });
  direction = { x: 1, y: 0 };
  turnQueue = [];
}

// Новый вопрос: ответы на поле или один ❓
function newQuestion() {
  tokens = [];
  currentQuestion = nextQuestion();
  if (settings.inputMode === 'keyboard') {
    const cell = randomFreeCell(5);
    tokens.push({ c: cell.c, r: cell.r, text: '❓', quiz: true });
  } else {
    // С подсказкой 50/50 на поле только 2 ответа: верный и один неверный
    let options = currentQuestion.options;
    if (perks.hint) {
      const wrong = options.filter(o => normalize(o) !== normalize(currentQuestion.answer));
      options = shuffle([currentQuestion.answer, wrong[0]]);
    }
    for (const text of options) {
      const cell = randomFreeCell(5);
      tokens.push({ c: cell.c, r: cell.r, text: text, correct: normalize(text) === normalize(currentQuestion.answer) });
    }
  }
  updateBanner();
}

function spawnEnemy(isBoss) {
  const theme = currentTheme();
  const cell = randomFreeCell(7);
  enemies.push({
    c: cell.c, r: cell.r,
    emoji: isBoss ? theme.boss.emoji : theme.enemies[Math.floor(Math.random() * theme.enemies.length)],
    timer: 1,
    boss: isBoss,
    life: isBoss ? 15 : Infinity    // босс уходит через 15 секунд
  });
}


/* ---------------------------------------------------------
   8. РИСОВАНИЕ: ФОН, ЗМЕЙКА, ОТВЕТЫ, ЧАСТИЦЫ
   --------------------------------------------------------- */

const TAU = Math.PI * 2;

function clamp(value, min, max) {
  return Math.max(min, Math.min(max, value));
}

function seeded(n) {
  const x = Math.sin(n * 127.1 + 311.7) * 43758.5453;
  return x - Math.floor(x);
}

function drawCircle(x, y, r, color) {
  ctx.fillStyle = color;
  ctx.beginPath();
  ctx.arc(x, y, r, 0, TAU);
  ctx.fill();
}

function drawGlow(x, y, r, color) {
  const g = ctx.createRadialGradient(x, y, 0, x, y, r);
  g.addColorStop(0, color);
  g.addColorStop(1, 'rgba(0,0,0,0)');
  ctx.fillStyle = g;
  ctx.fillRect(x - r, y - r, r * 2, r * 2);
}

function drawEmoji(emoji, x, y, size) {
  ctx.font = size + 'px "Apple Color Emoji", "Segoe UI Emoji", "Noto Color Emoji", sans-serif';
  ctx.textAlign = 'center';
  ctx.textBaseline = 'middle';
  ctx.fillText(emoji, x, y);
}

function hexToRgba(color, alpha) {
  const n = parseInt(color.slice(1), 16);
  return 'rgba(' + ((n >> 16) & 255) + ',' + ((n >> 8) & 255) + ',' + (n & 255) + ',' + alpha + ')';
}

function drawBackground() {
  const theme = currentTheme();
  const g = ctx.createLinearGradient(0, 0, 0, H);
  g.addColorStop(0, theme.sky[0]);
  g.addColorStop(1, theme.sky[1]);
  ctx.fillStyle = g;
  ctx.fillRect(0, 0, W, H);

  // Мерцающие звёзды или светлячки
  for (let i = 0; i < 60; i++) {
    ctx.globalAlpha = 0.25 + 0.5 * Math.abs(Math.sin(game.time * 1.5 + i));
    drawCircle(seeded(i) * W, seeded(i + 50) * H, 0.8 + seeded(i + 9) * 1.4, theme.id === 'eco' ? '#9dff8a' : '#ffffff');
  }
  ctx.globalAlpha = 1;

  // Игровое поле: шахматка
  ctx.fillStyle = game.flash > 0 ? 'rgba(255,60,90,' + (0.25 + game.flash) + ')' : theme.board;
  ctx.fillRect(0, TOP, W, ROWS * CELL);
  ctx.fillStyle = theme.cellLight;
  for (let c = 0; c < COLS; c++) {
    for (let r = 0; r < ROWS; r++) {
      if ((c + r) % 2 === 0) ctx.fillRect(c * CELL, TOP + r * CELL, CELL, CELL);
    }
  }
  // Рамка: если стены «проходимые» — пунктир
  ctx.strokeStyle = hexToRgba(theme.accent, 0.7);
  ctx.lineWidth = 3;
  ctx.setLineDash(perks && perks.wrap ? [10, 8] : []);
  ctx.strokeRect(1.5, TOP + 1.5, W - 3, ROWS * CELL - 3);
  ctx.setLineDash([]);
}

function drawSnake() {
  const theme = currentTheme();
  // Тело рисуем с хвоста, чтобы голова была сверху
  for (let i = snake.length - 1; i >= 1; i--) {
    const s = snake[i];
    const x = cellX(s.c);
    const y = cellY(s.r);
    const size = CELL * (0.42 - i / snake.length * 0.12);
    const color = LAB.rainbowSnake ? 'hsl(' + ((i * 25 + game.time * 120) % 360) + ', 90%, 55%)' : (i % 2 ? theme.accent : theme.body);
    drawCircle(x, y, size, color);
    drawCircle(x - size * 0.3, y - size * 0.3, size * 0.3, 'rgba(255,255,255,0.35)');
  }
  const head = snake[0];
  if (!head) return;
  // Неуязвимость — змейка мигает
  if (game.safe > 0 && Math.floor(game.time * 10) % 2 === 0) ctx.globalAlpha = 0.4;
  drawGlow(cellX(head.c), cellY(head.r), CELL, hexToRgba(theme.accent, 0.5));
  drawEmoji(headEmoji(), cellX(head.c), cellY(head.r), CELL * 0.95);
  if (perks.shield) {
    ctx.strokeStyle = 'rgba(127,220,255,0.8)';
    ctx.lineWidth = 2;
    ctx.beginPath();
    ctx.arc(cellX(head.c), cellY(head.r), CELL * 0.62, 0, TAU);
    ctx.stroke();
  }
  ctx.globalAlpha = 1;
}

// Ответ на поле — светящийся пузырь с текстом
function drawTokens() {
  const theme = currentTheme();
  const pulse = 1 + Math.sin(game.time * 4) * 0.06;
  for (const t of tokens) {
    const x = cellX(t.c);
    const y = cellY(t.r);
    if (t.quiz) {
      drawGlow(x, y, CELL * 1.1, 'rgba(255,215,0,0.5)');
      drawEmoji('❓', x, y, CELL * 0.9 * pulse);
      continue;
    }
    drawGlow(x, y, CELL * 1.1, hexToRgba(theme.accent, 0.35));
    ctx.fillStyle = 'rgba(10,14,40,0.9)';
    ctx.strokeStyle = theme.accent;
    ctx.lineWidth = 2;
    // Ширина пузыря зависит от длины текста
    ctx.font = 'bold 18px "Russo One", Rubik, sans-serif';
    const width = Math.max(CELL * 0.9, ctx.measureText(t.text).width + 14) * pulse;
    ctx.beginPath();
    ctx.roundRect(x - width / 2, y - CELL * 0.4 * pulse, width, CELL * 0.8 * pulse, 12);
    ctx.fill();
    ctx.stroke();
    ctx.fillStyle = '#ffffff';
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';
    ctx.fillText(t.text, x, y + 1);
  }
}

function drawEnemies() {
  for (const e of enemies) {
    const x = cellX(e.c);
    const y = cellY(e.r);
    if (e.boss) drawGlow(x, y, CELL * 1.4, 'rgba(255,60,90,0.45)');
    if (perks.hunter) drawGlow(x, y, CELL, 'rgba(57,255,120,0.35)'); // можно съесть
    drawEmoji(e.emoji, x, y, CELL * (e.boss ? 1.25 : 0.9));
  }
  for (const f of fruits) drawEmoji('🍎', cellX(f.c), cellY(f.r), CELL * 0.8);
}

function spawnParticles(x, y, count, color, power) {
  for (let i = 0; i < count; i++) {
    const angle = Math.random() * TAU;
    const speed = (0.3 + Math.random()) * power;
    particles.push({ x: x, y: y, vx: Math.cos(angle) * speed, vy: Math.sin(angle) * speed, life: 0.4 + Math.random() * 0.5, size: 2 + Math.random() * 4, color: color });
  }
}

function updateParticles(dt) {
  for (const p of particles) {
    p.x += p.vx * dt;
    p.y += p.vy * dt;
    p.vy += 150 * dt;
    p.life -= dt;
  }
  particles = particles.filter(p => p.life > 0);
}

function drawParticles() {
  for (const p of particles) {
    ctx.globalAlpha = Math.max(0, p.life * 2);
    drawCircle(p.x, p.y, p.size, p.color);
  }
  ctx.globalAlpha = 1;
}


/* ---------------------------------------------------------
   9. ОКНО ВОПРОСА (для режима «Ввод»: съели ❓)
   --------------------------------------------------------- */

let quizLocked = false;

function openQuiz() {
  game.state = 'quiz';
  quizLocked = false;
  const subject = findById(SUBJECTS, settings.subject);
  document.getElementById('quizSubject').textContent = subject.icon + ' ' + subject.label;
  document.getElementById('quizQuestion').innerHTML = currentQuestion.text +
    (currentQuestion.hint ? '<small>' + currentQuestion.hint + '</small>' : '');
  document.getElementById('quizHint').textContent = perks.hint
    ? 'Подсказка: начинается на «' + String(currentQuestion.answer)[0] + '»'
    : 'Верный ответ — змейка растёт!';
  document.getElementById('quizCard').classList.remove('flash-bad', 'flash-good');
  document.getElementById('quizOptions').classList.add('hidden');
  document.getElementById('quizForm').classList.remove('hidden');
  const inputEl = document.getElementById('quizInput');
  inputEl.value = '';
  inputEl.inputMode = currentQuestion.numeric ? 'numeric' : 'text';
  showScreen('quiz');
  setTimeout(() => inputEl.focus(), 50);
}

function answerQuiz(text) {
  if (quizLocked || game.state !== 'quiz') return;
  quizLocked = true;
  const correct = normalize(text) === normalize(currentQuestion.answer);
  restartAnimation(document.getElementById('quizCard'), correct ? 'flash-good' : 'flash-bad');
  if (correct) {
    setTimeout(() => {
      game.state = 'playing';
      showScreen(null);
      rightAnswer(snake[0]);
    }, 500);
  } else {
    playSound('crash');
    document.getElementById('quizHint').textContent = 'Правильно: ' + currentQuestion.answer;
    setTimeout(() => {
      game.state = 'playing';
      showScreen(null);
      wrongAnswer(snake[0]);
    }, 1100);
  }
}

document.getElementById('quizForm').addEventListener('submit', (event) => {
  event.preventDefault();
  const value = document.getElementById('quizInput').value;
  if (value.trim()) answerQuiz(value);
});


/* ---------------------------------------------------------
   10. ЭКРАН УЛУЧШЕНИЙ
   --------------------------------------------------------- */

let perkOffer = [];

function openPerks() {
  const free = PERKS.filter(perk => !game.usedPerks.includes(perk.id));
  if (free.length === 0) return;
  game.state = 'perk';
  perkOffer = shuffle(free).slice(0, 3);
  const box = document.getElementById('perkChoices');
  box.innerHTML = '';
  perkOffer.forEach((perk, i) => {
    box.appendChild(makeChoice(perk.icon, perk.title, perk.desc, () => pickPerk(perk), i + 1));
  });
  showScreen('perk');
}

function pickPerk(perk) {
  if (game.state !== 'perk') return;
  perk.apply(perks);
  game.usedPerks.push(perk.id);
  playSound('start');
  showToast(perk.icon + ' ' + perk.title);
  game.state = 'playing';
  game.safe = 1;   // секунда, чтобы осмотреться
  showScreen(null);
  // С подсказкой сразу убираем лишние ответы
  if (perk.id === 'hint') newQuestion();
  updateHud();
}


/* ---------------------------------------------------------
   11. ИГРОВЫЕ ДЕЙСТВИЯ
   --------------------------------------------------------- */

function addScore(points) {
  game.score += Math.round(points * difficulty().score * perks.scoreMult);
}

// Съели верный ответ
function rightAnswer(cell) {
  game.correct++;
  game.grow += LAB.growPerAnswer;
  addScore(10 * game.level);
  playSound('coin');
  spawnParticles(cellX(cell.c), cellY(cell.r), 24, currentTheme().accent, 220);
  showToast('Верно! +' + LAB.growPerAnswer);
  newQuestion();

  // Каждые 3 верных ответа — новый уровень: быстрее, ещё враг, улучшение
  if (game.correct % 3 === 0) {
    game.level++;
    if (enemies.filter(e => !e.boss).length < LAB.maxEnemies) spawnEnemy(false);
    if (game.correct % 9 === 0) {
      spawnEnemy(true);
      showToast('Босс: ' + currentTheme().boss.name + '!');
    }
    openPerks();
  }
  updateHud();
}

// Съели неверный ответ: минус жизнь и подсказка
function wrongAnswer(cell) {
  showToast('Правильно: ' + currentQuestion.answer);
  loseLife(false);
  if (game.state !== 'over') newQuestion();
}

// Минус жизнь. respawn — вернуть змейку в центр (после стены или врага)
function loseLife(respawn) {
  game.lives--;
  game.flash = 0.4;
  playSound('crash');
  const head = snake[0];
  spawnParticles(cellX(head.c), cellY(head.r), 26, '#ff4d6d', 240);
  if (game.lives <= 0) {
    game.lives = 0;
    gameOver();
    return;
  }
  if (respawn) {
    placeSnake(Math.max(LAB.startLength, snake.length));
    // Враги рядом с центром отходят
    enemies = enemies.filter(e => Math.abs(e.c - 5) + Math.abs(e.r - 6) > 4);
  }
  game.safe = perks.safeTime;
  updateHud();
}

// Удар врага: щит может спасти
function hitByEnemy(enemy) {
  if (game.safe > 0) return;
  if (perks.hunter && !enemy.boss) {
    enemies = enemies.filter(e => e !== enemy);
    addScore(20);
    game.grow += 1;
    playSound('jump');
    spawnParticles(cellX(enemy.c), cellY(enemy.r), 18, '#39d97a', 200);
    setTimeout(() => { if (game.state === 'playing') spawnEnemy(false); }, 3000);
    return;
  }
  if (perks.shield) {
    perks.shield = false;
    game.safe = 1.5;
    showToast('🛡️ Щит!');
    return;
  }
  loseLife(true);
}

function gameOver() {
  game.state = 'over';
  const isRecord = game.score > bestScore;
  if (isRecord) {
    bestScore = game.score;
    saveBestScore();
  }
  document.getElementById('finalScore').textContent = game.score;
  document.getElementById('finalLength').textContent = snake.length;
  document.getElementById('finalAnswers').textContent = game.correct;
  document.getElementById('finalBest').textContent = bestScore;
  document.getElementById('newRecord').classList.toggle('hidden', !isRecord || game.score === 0);
  setTimeout(() => showScreen('over'), 700);
}

function resetGame() {
  resetPerks();
  game.score = 0;
  game.lives = LAB.lives;
  game.level = 1;
  game.correct = 0;
  game.grow = 0;
  game.safe = 0;
  game.flash = 0;
  game.stepTimer = 0;
  game.usedPerks = [];
  tokens = [];
  enemies = [];
  fruits = [];
  particles = [];
  placeSnake(LAB.startLength);
  buildQuestionPool();
  newQuestion();
  if (LAB.maxEnemies > 0) spawnEnemy(false);
  updateHud();
}

function openDifficultySelect() {
  game.state = 'choose';
  const box = document.getElementById('difficultyChoices');
  box.innerHTML = '';
  DIFFICULTIES.forEach((d, i) => {
    box.appendChild(makeChoice(d.icon, d.label, d.desc, () => {
      settings.difficulty = d.id;
      saveSettings();
      startGame();
    }, i + 1));
  });
  showScreen('difficulty');
}

let countdownTimer = null;

function startGame() {
  if (audioContext.state === 'suspended') audioContext.resume();
  resetGame();
  showScreen(null);
  game.state = 'countdown';
  let number = 3;
  showCountdown(number);
  clearInterval(countdownTimer);
  countdownTimer = setInterval(() => {
    number--;
    if (number > 0) {
      showCountdown(number);
    } else {
      clearInterval(countdownTimer);
      showCountdown('Вперёд!');
      game.state = 'playing';
      playSound('start');
    }
  }, 750);
}

function pauseGame() {
  if (game.state !== 'playing') return;
  game.state = 'paused';
  showScreen('pause');
}

function resumeGame() {
  if (game.state !== 'paused') return;
  game.state = 'playing';
  showScreen(null);
}

function goToMenu() {
  clearInterval(countdownTimer);
  hideCountdown();
  game.state = 'menu';
  showScreen('menu');
}

// Поворот: нельзя развернуться назад, в себя
function turn(x, y) {
  const last = turnQueue.length ? turnQueue[turnQueue.length - 1] : direction;
  if (last.x === -x && last.y === -y) return;
  if (last.x === x && last.y === y) return;
  if (turnQueue.length < 2) turnQueue.push({ x: x, y: y });
}


/* ---------------------------------------------------------
   12. ОБНОВЛЕНИЕ МИРА ЗА ОДИН КАДР
   Змейка двигается шагами по клеткам: раз в stepTime секунд
   --------------------------------------------------------- */

function stepTime() {
  const base = difficulty().step / LAB.snakeSpeed * perks.stepBonus;
  return Math.max(0.06, base * Math.pow(0.95, game.level - 1));
}

function update(dt) {
  if (['quiz', 'perk', 'paused'].includes(game.state)) return;
  game.time += dt;
  updateParticles(dt);
  if (game.flash > 0) game.flash -= dt;
  if (game.state !== 'playing') return;
  if (game.safe > 0) game.safe -= dt;

  game.stepTimer += dt;
  if (game.stepTimer >= stepTime()) {
    game.stepTimer = 0;
    moveSnake();
  }
  if (game.state === 'playing') updateEnemies(dt);
}

function moveSnake() {
  if (turnQueue.length) direction = turnQueue.shift();
  const head = snake[0];
  let c = head.c + direction.x;
  let r = head.r + direction.y;

  // Стены
  if (c < 0 || c >= COLS || r < 0 || r >= ROWS) {
    if (perks.wrap) {
      c = (c + COLS) % COLS;
      r = (r + ROWS) % ROWS;
    } else {
      loseLife(true);
      return;
    }
  }

  // Врезались в себя
  if (!perks.armor && game.safe <= 0 && snake.some((s, i) => i > 0 && i < snake.length - 1 && s.c === c && s.r === r)) {
    loseLife(true);
    return;
  }

  snake.unshift({ c: c, r: r });
  if (game.grow > 0) game.grow--;
  else snake.pop();

  // Съели ответ?
  const token = tokens.find(t => t.c === c && t.r === r);
  if (token) {
    if (token.quiz) {
      tokens = [];
      openQuiz();
    } else if (token.correct) {
      rightAnswer(token);
    } else {
      wrongAnswer(token);
    }
  }

  // Съели фрукт?
  const fruit = fruits.find(f => f.c === c && f.r === r);
  if (fruit) {
    fruits = fruits.filter(f => f !== fruit);
    game.grow += 1;
    addScore(5);
    playSound('jump');
    spawnParticles(cellX(c), cellY(r), 12, '#ff5c5c', 160);
  }
  if (perks.fruit && fruits.length < 2 && Math.random() < 0.02) fruits.push(randomFreeCell(3));

  // Наткнулись на врага?
  const enemy = enemies.find(e => e.c === c && e.r === r);
  if (enemy) hitByEnemy(enemy);
  updateHud();
}

// Враги ходят по клеткам: обычные — случайно, босс — за головой
function updateEnemies(dt) {
  const head = snake[0];
  for (const e of enemies) {
    e.life -= dt;
    e.timer -= dt;
    if (e.timer > 0) continue;
    e.timer = difficulty().enemy / LAB.enemySpeed * perks.enemySlow * (e.boss ? 0.8 : 1);
    let options = [[1, 0], [-1, 0], [0, 1], [0, -1]];
    if (e.boss) {
      // Босс выбирает шаг, который ближе к голове змейки
      options.sort((a, b) =>
        (Math.abs(e.c + a[0] - head.c) + Math.abs(e.r + a[1] - head.r)) -
        (Math.abs(e.c + b[0] - head.c) + Math.abs(e.r + b[1] - head.r)));
    } else {
      options = shuffle(options);
    }
    for (const [dx, dy] of options) {
      const c = e.c + dx;
      const r = e.r + dy;
      if (c < 0 || c >= COLS || r < 0 || r >= ROWS) continue;
      if (tokens.some(t => t.c === c && t.r === r)) continue;
      if (enemies.some(o => o !== e && o.c === c && o.r === r)) continue;
      e.c = c;
      e.r = r;
      break;
    }
    if (e.c === head.c && e.r === head.r) hitByEnemy(e);
    if (game.state !== 'playing') return;
  }
  // Босс уходит, когда время вышло
  enemies = enemies.filter(e => e.life > 0);
}


/* ---------------------------------------------------------
   13. РИСОВАНИЕ ОДНОГО КАДРА
   --------------------------------------------------------- */

function draw() {
  drawBackground();
  if (game.state !== 'menu' && snake.length) {
    drawTokens();
    drawEnemies();
    drawSnake();
  }
  drawParticles();
}


/* ---------------------------------------------------------
   14. ЭКРАНЫ И ИНТЕРФЕЙС
   --------------------------------------------------------- */

const screens = {
  menu: document.getElementById('menuScreen'),
  difficulty: document.getElementById('difficultyScreen'),
  quiz: document.getElementById('quizScreen'),
  perk: document.getElementById('perkScreen'),
  settings: document.getElementById('settingsScreen'),
  pause: document.getElementById('pauseScreen'),
  over: document.getElementById('gameOverScreen')
};

function showScreen(name) {
  for (const key in screens) {
    screens[key].classList.toggle('hidden', key !== name);
  }
  const inGame = name === null || ['pause', 'quiz', 'perk'].includes(name);
  document.getElementById('hud').classList.toggle('hidden', !inGame);
  document.getElementById('menuBest').textContent = bestScore;
  if (document.activeElement && document.activeElement.blur) document.activeElement.blur();
}

function isVisible(el) {
  return !el.classList.contains('hidden');
}

function makeChoice(icon, title, desc, onPick, key) {
  const button = document.createElement('button');
  button.type = 'button';
  button.className = 'choice';
  button.innerHTML = '<span class="choice-icon">' + icon + '</span><span><span class="choice-title">' + title + '</span>' +
    (desc ? '<br><span class="choice-desc">' + desc + '</span>' : '') + '</span>';
  button.title = 'Клавиша ' + key;
  button.addEventListener('click', onPick);
  return button;
}

function updateBanner() {
  const banner = document.getElementById('questionBanner');
  if (settings.inputMode === 'keyboard') {
    banner.innerHTML = 'Съешь ❓<small>и ответь на вопрос</small>';
  } else {
    banner.innerHTML = currentQuestion.text + (currentQuestion.hint ? '<small>' + currentQuestion.hint + '</small>' : '<small>Съешь правильный ответ</small>');
  }
}

function updateHud() {
  document.getElementById('lives').textContent = game.lives > 6 ? '❤️×' + game.lives : '❤️'.repeat(game.lives) || '💔';
  document.getElementById('score').textContent = game.score;
  document.getElementById('length').textContent = snake.length;
  document.getElementById('level').textContent = game.level;
  document.getElementById('perkChips').innerHTML = game.usedPerks
    .map(id => '<span class="chip">' + PERKS.find(p => p.id === id).icon + '</span>').join('');
}

function showCountdown(value) {
  const el = document.getElementById('countdown');
  el.classList.remove('hidden');
  const isText = typeof value === 'string';
  el.innerHTML = '<span class="' + (isText ? 'go' : '') + '">' + value + '</span>';
  if (!isText) playSound('tick');
  if (isText) setTimeout(hideCountdown, 600);
}

function hideCountdown() {
  document.getElementById('countdown').classList.add('hidden');
}

function showToast(text) {
  const el = document.getElementById('toast');
  el.textContent = text;
  restartAnimation(el, 'show');
}

function restartAnimation(el, className) {
  el.classList.remove(className);
  void el.offsetWidth;
  el.classList.add(className);
}

function applyTheme() {
  const theme = currentTheme();
  const subject = findById(SUBJECTS, settings.subject);
  document.documentElement.style.setProperty('--accent', theme.accent);
  document.getElementById('menuInfo').textContent =
    theme.icon + ' ' + theme.label + ' · ' + subject.icon + ' ' + subject.label + ' · ' + findById(INPUT_MODES, settings.inputMode).label;
}


/* ---------------------------------------------------------
   15. ЭКРАН НАСТРОЕК
   --------------------------------------------------------- */

function buildOptions(containerId, items, settingName) {
  const container = document.getElementById(containerId);
  container.innerHTML = '';
  for (const item of items) {
    const button = document.createElement('button');
    button.type = 'button';
    button.className = 'option' + (settings[settingName] === item.id ? ' selected' : '');
    button.innerHTML = '<span class="option-icon">' + item.icon + '</span><span class="option-title">' + item.label + '</span><span>' + item.desc + '</span>';
    button.addEventListener('click', () => {
      settings[settingName] = item.id;
      saveSettings();
      applyTheme();
      buildSettings();
    });
    container.appendChild(button);
  }
}

function buildSettings() {
  buildOptions('themeOptions', THEMES, 'theme');
  buildOptions('subjectOptions', SUBJECTS, 'subject');
  buildOptions('inputOptions', INPUT_MODES, 'inputMode');
  document.getElementById('soundToggle').checked = settings.sound;
  document.getElementById('volumeRange').value = Math.round(settings.volume * 100);
  document.getElementById('volumeRange').disabled = !settings.sound;
}

document.getElementById('soundToggle').addEventListener('change', (event) => {
  settings.sound = event.target.checked;
  saveSettings();
  buildSettings();
  playSound('coin');
});

document.getElementById('volumeRange').addEventListener('change', (event) => {
  settings.volume = Number(event.target.value) / 100;
  saveSettings();
  playSound('coin');
});

document.getElementById('resetBestButton').addEventListener('click', () => {
  bestScore = 0;
  saveBestScore();
  showToast('Рекорд сброшен');
});


/* ---------------------------------------------------------
   16. УПРАВЛЕНИЕ
   --------------------------------------------------------- */

document.addEventListener('keydown', (event) => {
  if (document.activeElement && document.activeElement.id === 'quizInput') return;
  const code = event.code;
  if (['ArrowLeft', 'ArrowRight', 'ArrowUp', 'ArrowDown', 'Space'].includes(code)) event.preventDefault();

  if (game.state === 'playing' || game.state === 'countdown') {
    if (code === 'ArrowLeft' || code === 'KeyA') turn(-1, 0);
    if (code === 'ArrowRight' || code === 'KeyD') turn(1, 0);
    if (code === 'ArrowUp' || code === 'KeyW') turn(0, -1);
    if (code === 'ArrowDown' || code === 'KeyS') turn(0, 1);
  }

  const digit = Number(event.key);
  if (digit >= 1 && digit <= 3) {
    let list = null;
    if (isVisible(screens.difficulty)) list = document.querySelectorAll('#difficultyChoices .choice');
    else if (game.state === 'perk') list = document.querySelectorAll('#perkChoices .choice');
    if (list && list[digit - 1]) list[digit - 1].click();
  }

  if (code === 'Space' || code === 'Enter') {
    if (isVisible(screens.menu)) openDifficultySelect();
    else if (isVisible(screens.over)) startGame();
  }
  if (code === 'Escape' || code === 'KeyP') {
    if (game.state === 'playing') pauseGame();
    else if (game.state === 'paused') resumeGame();
    else if (isVisible(screens.settings)) showScreen('menu');
  }
});

// Свайпы: запоминаем, где палец коснулся, и куда сдвинулся
let swipeStart = null;

canvas.addEventListener('pointerdown', (event) => {
  swipeStart = { x: event.clientX, y: event.clientY };
});

canvas.addEventListener('pointerup', (event) => {
  if (!swipeStart) return;
  const dx = event.clientX - swipeStart.x;
  const dy = event.clientY - swipeStart.y;
  swipeStart = null;
  if (Math.max(Math.abs(dx), Math.abs(dy)) < 20) return;
  if (Math.abs(dx) > Math.abs(dy)) turn(dx > 0 ? 1 : -1, 0);
  else turn(0, dy > 0 ? 1 : -1);
});

document.getElementById('startButton').addEventListener('click', openDifficultySelect);
document.getElementById('settingsButton').addEventListener('click', () => {
  buildSettings();
  showScreen('settings');
});
document.getElementById('settingsClose').addEventListener('click', () => showScreen('menu'));
document.getElementById('settingsDone').addEventListener('click', () => showScreen('menu'));
document.getElementById('pauseButton').addEventListener('click', pauseGame);
document.getElementById('resumeButton').addEventListener('click', resumeGame);
document.getElementById('pauseMenuButton').addEventListener('click', goToMenu);
document.getElementById('restartButton').addEventListener('click', startGame);
document.getElementById('overMenuButton').addEventListener('click', goToMenu);

document.addEventListener('visibilitychange', () => {
  if (document.hidden) pauseGame();
});

window.addEventListener('resize', resizeGame);


/* ---------------------------------------------------------
   17. ИГРОВОЙ ЦИКЛ И ЗАПУСК
   --------------------------------------------------------- */

let lastTime = performance.now();

function loop(now) {
  let dt = (now - lastTime) / 1000;
  lastTime = now;
  if (dt > 0.05) dt = 0.05;
  update(dt);
  draw();
  requestAnimationFrame(loop);
}

document.querySelector('.title-top').textContent = LAB.titleTop;
document.querySelector('.title-bottom').textContent = LAB.titleBottom;
document.title = LAB.titleTop + ' ' + LAB.titleBottom;
resetPerks();
resizeGame();
applyTheme();
showScreen('menu');
requestAnimationFrame(loop);
````

<!-- SOURCES:END -->
