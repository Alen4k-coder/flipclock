<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <title>Centered Flip Clock (Even Bigger)</title>
  <!-- Подключаем Montserrat из Google Fonts -->
  <link
    href="https://fonts.googleapis.com/css2?family=Montserrat:wght@700&display=swap"
    rel="stylesheet"
  />
  <style>
    * {
      margin: 0; 
      padding: 0; 
      box-sizing: border-box;
    }
    body {
      font-family: 'Montserrat', sans-serif;
      background: #f0f0f0;
      height: 100vh;
      display: flex;
      justify-content: center;
      align-items: center;
    }
    .clock-container {
      display: flex;
      gap: 50px; /* увеличенный отступ между плашками */
    }
    /* -- Каждая «плашка» (часы / минуты) -- */
    .flip-card {
      position: relative;
      width: 340px;   /* ещё больше ширина */
      height: 420px;  /* ещё больше высота */
      background-color: #640505; /* Бордовый цвет */
      color: #ffffff;
      perspective: 1000px; /* для 3D flip-эффекта */
      border-radius: 8px;
      overflow: hidden;
    }
    /* -- Внутренний контейнер для лицевой и оборотной сторон -- */
    .flip-inner {
      position: relative;
      width: 100%;
      height: 100%;
      transform-style: preserve-3d; 
      transition: transform 0.7s ease;
    }
    /* -- Лицевая и оборотная стороны карточки -- */
    .flip-face {
      position: absolute;
      width: 100%;
      height: 100%;
      backface-visibility: hidden; 
      display: flex;
      justify-content: center;
      align-items: center;
      font-size: 192px;    /* УВЕЛИЧЕННЫЙ размер шрифта для цифр */
      font-weight: 700;
      line-height: 1;
    }
    /* Лицевая сторона (текущие/старые значения) */
    .flip-front {
      /* rotateX(0) по умолчанию */
    }
    /* Оборотная сторона (новое значение) */
    .flip-back {
      transform: rotateX(180deg);
    }

    /* -- Метки даты (слева) и дня недели (справа) -- */
    .date-label, .weekday-label {
      position: absolute;
      font-size: 24px; /* Увеличено с 20px */
      text-transform: uppercase;
      opacity: 0.9;
    }
    .date-label {
      bottom: 15px;
      left: 15px;
    }
    .weekday-label {
      bottom: 15px;
      right: 15px;
    }

    /* -- Класс, который запускает переворот -- */
    .flip-card.flip .flip-inner {
      transform: rotateX(-180deg);
    }
  </style>
</head>
<body>
<div class="clock-container">
  <!-- Левая плашка (ЧАСЫ) -->
  <div class="flip-card" id="hour-card">
    <div class="flip-inner" id="hour-inner">
      <!-- Старая (текущая) цифра часов -->
      <div class="flip-face flip-front" id="hour-front">00</div>
      <!-- Новая цифра (которая появится после flip) -->
      <div class="flip-face flip-back" id="hour-back">00</div>
    </div>
    <!-- Метка даты (например, 22 FEB) -->
    <div class="date-label" id="date-label">22 FEB</div>
  </div>

  <!-- Правая плашка (МИНУТЫ) -->
  <div class="flip-card" id="minute-card">
    <div class="flip-inner" id="minute-inner">
      <!-- Старая (текущая) цифра минут -->
      <div class="flip-face flip-front" id="minute-front">00</div>
      <!-- Новая цифра (которая появится после flip) -->
      <div class="flip-face flip-back" id="minute-back">00</div>
    </div>
    <!-- Метка дня недели (например, SATURDAY) -->
    <div class="weekday-label" id="weekday-label">SATURDAY</div>
  </div>
</div>

<script>
  const hourCard   = document.getElementById('hour-card');
  const hourInner  = document.getElementById('hour-inner');
  const hourFront  = document.getElementById('hour-front');
  const hourBack   = document.getElementById('hour-back');

  const minuteCard  = document.getElementById('minute-card');
  const minuteInner = document.getElementById('minute-inner');
  const minuteFront = document.getElementById('minute-front');
  const minuteBack  = document.getElementById('minute-back');

  const dateLabel    = document.getElementById('date-label');
  const weekdayLabel = document.getElementById('weekday-label');

  // Дни недели (англ., верхний регистр)
  const weekdays = [
    "SUNDAY","MONDAY","TUESDAY","WEDNESDAY",
    "THURSDAY","FRIDAY","SATURDAY"
  ];
  // Сокращения месяцев (англ., верхний регистр)
  const months = [
    "JAN","FEB","MAR","APR","MAY","JUN",
    "JUL","AUG","SEP","OCT","NOV","DEC"
  ];

  let currentHour   = null;
  let currentMinute = null;

  function updateClock() {
    const now       = new Date();
    const hour      = now.getHours();   
    const minute    = now.getMinutes();
    const day       = now.getDate();
    const month     = now.getMonth();
    const weekday   = now.getDay();

    // Форматированные часы и минуты (00..23, 00..59)
    const hourStr   = hour < 10 ? "0"+hour : String(hour);
    const minuteStr = minute < 10 ? "0"+minute : String(minute);

    // Обновляем дату и день недели
    dateLabel.textContent    = `${day} ${months[month]}`;
    weekdayLabel.textContent = weekdays[weekday];

    // Если час изменился - запускаем «переворот»
    if (hourStr !== currentHour) {
      flipAnimation(
        hourCard, hourInner, hourFront, hourBack,
        currentHour, hourStr
      );
      currentHour = hourStr;
    }

    // Если минута изменилась - запускаем «переворот»
    if (minuteStr !== currentMinute) {
      flipAnimation(
        minuteCard, minuteInner, minuteFront, minuteBack,
        currentMinute, minuteStr
      );
      currentMinute = minuteStr;
    }
  }

  /**
   * flipAnimation - меняет значение на карточке с анимацией переворота
   * @param {HTMLElement} card   - контейнер .flip-card
   * @param {HTMLElement} inner  - элемент .flip-inner
   * @param {HTMLElement} front  - элемент .flip-front (старое значение)
   * @param {HTMLElement} back   - элемент .flip-back  (новое значение)
   * @param {string|null} oldVal - предыдущее значение (или null при первом запуске)
   * @param {string} newVal      - новое значение
   */
  function flipAnimation(card, inner, front, back, oldVal, newVal) {
    // Если старое значение отсутствует (при первом запуске),
    // сразу устанавливаем новое, без анимации
    if (oldVal === null) {
      front.textContent = newVal;
      back.textContent  = newVal;
      return;
    }

    // Ставим "старое" значение на face-front, новое — на face-back
    front.textContent = oldVal;
    back.textContent  = newVal;

    // Запускаем анимацию
    card.classList.add('flip');
    // По окончании transition (0.7s):
    setTimeout(() => {
      // Теперь во front записываем новое значение
      front.textContent = newVal;
      // Убираем класс flip, возвращая rotateX(0)
      card.classList.remove('flip');
    }, 700);
  }

  // Инициализация
  updateClock();
  setInterval(updateClock, 1000);
</script>
</body>
</html>
