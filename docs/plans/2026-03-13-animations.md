# Animations Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Add cinematic CSS + Intersection Observer animations to the businka photographer landing page.

**Architecture:** All changes in a single `index.html`. CSS keyframes + transition classes handle visuals. One Intersection Observer adds `.visible` to elements as they enter viewport. Separate `mousemove` handler drives hero parallax.

**Tech Stack:** Plain HTML/CSS/JS, no dependencies.

---

### Task 1: CSS — базовые классы появления

**Files:**
- Modify: `index.html` (в блок `<style>`, перед закрывающим `</style>`)

**Step 1: Добавить CSS-анимации**

Вставить после строки `@media (max-width: 600px) { ... }` (перед `</style>`):

```css
/* ── ANIMATIONS ── */

/* Начальные состояния (невидимые) */
.fade-up,
.fade-left,
.fade-right,
.stagger-item {
  opacity: 0;
  transition: opacity 1.0s cubic-bezier(0.22, 1, 0.36, 1),
              transform 1.0s cubic-bezier(0.22, 1, 0.36, 1);
}
.fade-up    { transform: translateY(30px); }
.fade-left  { transform: translateX(-40px); }
.fade-right { transform: translateX(40px); }
.stagger-item { transform: translateY(24px); }

/* Видимое состояние */
.fade-up.visible,
.fade-left.visible,
.fade-right.visible,
.stagger-item.visible {
  opacity: 1;
  transform: translate(0);
}

/* Hero entrance — элементы изначально скрыты */
.hero-animate {
  opacity: 0;
  transform: translateY(20px);
  transition: opacity 1.0s cubic-bezier(0.22, 1, 0.36, 1),
              transform 1.0s cubic-bezier(0.22, 1, 0.36, 1);
}
.hero-animate.visible {
  opacity: 1;
  transform: translateY(0);
}

/* Hero photo cards — parallax transition */
.hero-photo-card {
  transition: transform 0.8s cubic-bezier(0.25, 0.46, 0.45, 0.94) !important;
}
```

**Step 2: Проверить в браузере**

Открыть http://localhost:3000 — страница должна быть полностью пустой (все элементы с классами анимаций невидимы). Это ожидаемо.

**Step 3: Commit**

```bash
cd ~/Projects/businka
git add index.html
git commit -m "feat: add animation CSS classes (elements hidden until JS triggers)"
```

---

### Task 2: HTML — расставить классы анимаций

**Files:**
- Modify: `index.html` (HTML-разметка)

**Step 1: Hero элементы — добавить `.hero-animate`**

Найти в hero-секции и добавить класс `hero-animate` к:
- `<div class="hero-tag">` → `<div class="hero-tag hero-animate">`
- `<h1>` → `<h1 class="hero-animate">`
- `<p class="hero-sub">` → `<p class="hero-sub hero-animate">`
- `<div class="hero-cta">` → `<div class="hero-cta hero-animate">`
- `<div class="hero-stats">` → `<div class="hero-stats hero-animate">`

**Step 2: Portfolio секция**

- `<div class="section-header">` → добавить класс `fade-up`
- `<div class="tabs">` → добавить класс `fade-up`
- Три `<div class="tab-panel ...">` — оставить как есть (masonry-items получат класс через JS)

**Step 3: About секция**

- `<div class="about-photo-wrap">` → добавить класс `fade-left`
- `<div class="about-text">` → добавить класс `fade-right`

**Step 4: Contact секция**

- `<div class="contact-inner">` → добавить класс `fade-up`

**Step 5: Проверить**

Открыть http://localhost:3000 — страница должна быть пустой (анимируемые элементы скрыты). JS ещё не написан.

**Step 6: Commit**

```bash
git add index.html
git commit -m "feat: add animation classes to HTML elements"
```

---

### Task 3: JS — Hero entrance sequence

**Files:**
- Modify: `index.html` (в блок `<script>`, после существующего кода)

**Step 1: Добавить hero entrance в конец `<script>`**

```js
// ── Hero entrance sequence ──
(function () {
  const items = document.querySelectorAll('.hero-animate');
  items.forEach((el, i) => {
    setTimeout(() => el.classList.add('visible'), i * 150);
  });
})();
```

**Step 2: Проверить**

Открыть http://localhost:3000, перезагрузить. Hero-элементы должны появляться один за другим: badge → h1 → subtitle → кнопки → статистика, с интервалом ~150ms. Плавно и медленно.

**Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add hero entrance stagger animation"
```

---

### Task 4: JS — Intersection Observer для скролл-анимаций

**Files:**
- Modify: `index.html` (в блок `<script>`, после hero entrance)

**Step 1: Добавить observer**

```js
// ── Scroll reveal ──
(function () {
  const observer = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
      if (!entry.isIntersecting) return;
      const el = entry.target;
      const delay = el.dataset.delay ? parseInt(el.dataset.delay) : 0;
      setTimeout(() => el.classList.add('visible'), delay);
      observer.unobserve(el);
    });
  }, { threshold: 0.15 });

  document.querySelectorAll('.fade-up, .fade-left, .fade-right').forEach(el => {
    observer.observe(el);
  });
})();
```

**Step 2: Проверить**

Открыть http://localhost:3000. Прокрутить страницу вниз — секции портфолио, «О себе» и контакты должны плавно появляться при скролле.

**Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add Intersection Observer scroll reveal"
```

---

### Task 5: JS — Stagger для masonry-карточек

**Files:**
- Modify: `index.html` (функция `buildMasonry` в `<script>`)

**Step 1: Обновить `buildMasonry`**

В функции `buildMasonry` добавить класс и data-delay к каждому item:

```js
function buildMasonry(id, category) {
  const grid = document.getElementById('masonry-' + id);
  photos[category].forEach((slug, i) => {
    const item = document.createElement('div');
    item.className = 'masonry-item stagger-item';
    item.dataset.delay = i * 60;   // ← добавить эту строку
    item.innerHTML = `
      <img src="${BASE}${slug}" alt="">
      <div class="masonry-overlay">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
          <path d="M15 3h6v6M9 21H3v-6M21 3l-7 7M3 21l7-7"/>
        </svg>
      </div>`;
    grid.appendChild(item);
  });
}
```

**Step 2: Обновить observer — добавить наблюдение за `.stagger-item`**

В существующем observer изменить последнюю строку:

```js
// было:
document.querySelectorAll('.fade-up, .fade-left, .fade-right').forEach(el => {
// стало:
document.querySelectorAll('.fade-up, .fade-left, .fade-right, .stagger-item').forEach(el => {
```

**Step 3: Проверить**

Перейти в портфолио — карточки должны появляться волной слева направо при загрузке вкладки. При переключении вкладок карточки новой вкладки тоже анимируются.

**Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add stagger animation to masonry cards"
```

---

### Task 6: JS — Hero parallax на mousemove

**Files:**
- Modify: `index.html` (в блок `<script>`, после observer)

**Step 1: Добавить parallax handler**

```js
// ── Hero parallax ──
(function () {
  const hero = document.querySelector('.hero');
  const cards = document.querySelectorAll('.hero-photo-card');
  const depths = [0.035, 0.02, 0.015];

  hero.addEventListener('mousemove', (e) => {
    const cx = hero.offsetWidth / 2;
    const cy = hero.offsetHeight / 2;
    const dx = e.clientX - cx;
    const dy = e.clientY - cy;

    cards.forEach((card, i) => {
      const x = dx * depths[i];
      const y = dy * depths[i];
      card.style.transform = `${card.style.transform.replace(/translate\([^)]*\)/, '')} translate(${x}px, ${y}px)`;
    });
  });

  hero.addEventListener('mouseleave', () => {
    cards.forEach(card => {
      card.style.transform = card.style.transform.replace(/translate\([^)]*\)/, '');
    });
  });
})();
```

**Step 2: Проверить**

Навести мышь на hero-секцию и двигать — три карточки должны слегка смещаться с разной скоростью, создавая эффект глубины.

**Step 3: Final commit**

```bash
git add index.html
git commit -m "feat: add hero parallax mouse tracking"
git push origin main
```

---

## Итог

После выполнения всех задач:
- Hero появляется кинематографично при загрузке
- Секции плавно въезжают при скролле
- Карточки портфолио появляются волной
- Hero-фотокарточки реагируют на мышь

Проверить финальный результат на http://localhost:3000 и на https://amnovoselov.github.io/businka/
