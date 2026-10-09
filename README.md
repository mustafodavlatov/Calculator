<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Apple Calculator</title>
  <style>
    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      user-select: none;
      -webkit-user-select: none;
      font-family: -apple-system, BlinkMacSystemFont, "SF Pro Display", "SF Pro Text", "Helvetica Neue", sans-serif;
    }

    body {
      min-height: 100vh;
      display: flex;
      justify-content: center;
      align-items: center;
      background-color: #121212;
    }

    .calculator {
      width: 320px;
      background-color: #000000;
      border-radius: 40px;
      padding: 24px 18px;
      box-shadow: 0 20px 40px rgba(0, 0, 0, 0.6);
      display: flex;
      flex-direction: column;
    }

    .display {
      height: 110px;
      color: #ffffff;
      display: flex;
      align-items: flex-end;
      justify-content: flex-end;
      padding: 0 12px 16px 12px;
      font-size: 64px;
      font-weight: 300;
      letter-spacing: -1px;
      overflow: hidden;
      white-space: nowrap;
    }

    .keypad {
      display: grid;
      grid-template-columns: repeat(4, 1fr);
      gap: 12px;
    }

    button {
      height: 62px;
      width: 62px;
      border-radius: 50%;
      border: none;
      font-size: 26px;
      font-weight: 400;
      cursor: pointer;
      display: flex;
      align-items: center;
      justify-content: center;
      transition: filter 0.15s ease, opacity 0.15s ease;
      outline: none;
    }

    button:active {
      filter: brightness(130%);
    }

    /* Верхние функциональные кнопки */
    .btn-top {
      background-color: #a5a5a5;
      color: #000000;
    }

    /* Операторы справа */
    .btn-op {
      background-color: #ff9f0a;
      color: #ffffff;
      font-size: 30px;
    }

    .btn-op.active {
      background-color: #ffffff;
      color: #ff9f0a;
    }

    /* Цифры */
    .btn-num {
      background-color: #333333;
      color: #ffffff;
    }

    /* Кнопка 0 (широкая) */
    .btn-zero {
      grid-column: span 2;
      width: 100%;
      border-radius: 35px;
      justify-content: flex-start;
      padding-left: 24px;
    }
  </style>
</head>
<body>

<div class="calculator">
  <div class="display" id="display">0</div>
  <div class="keypad">
    <button class="btn-top" onclick="handleClear()" id="clear-btn">AC</button>
    <button class="btn-top" onclick="handleToggleSign()">±</button>
    <button class="btn-top" onclick="handlePercent()">%</button>
    <button class="btn-op" onclick="handleOperator('÷')" id="op-div">÷</button>

    <button class="btn-num" onclick="inputDigit('7')">7</button>
    <button class="btn-num" onclick="inputDigit('8')">8</button>
    <button class="btn-num" onclick="inputDigit('9')">9</button>
    <button class="btn-op" onclick="handleOperator('×')" id="op-mul">×</button>

    <button class="btn-num" onclick="inputDigit('4')">4</button>
    <button class="btn-num" onclick="inputDigit('5')">5</button>
    <button class="btn-num" onclick="inputDigit('6')">6</button>
    <button class="btn-op" onclick="handleOperator('-')" id="op-sub">−</button>

    <button class="btn-num" onclick="inputDigit('1')">1</button>
    <button class="btn-num" onclick="inputDigit('2')">2</button>
    <button class="btn-num" onclick="inputDigit('3')">3</button>
    <button class="btn-op" onclick="handleOperator('+')" id="op-add">+</button>

    <button class="btn-num btn-zero" onclick="inputDigit('0')">0</button>
    <button class="btn-num" onclick="inputDecimal()">,</button>
    <button class="btn-op" onclick="handleEquals()">=</button>
  </div>
</div>

<script>
  let currentValue = '0';
  let previousValue = null;
  let selectedOperator = null;
  let waitingForSecondOperand = false;

  const display = document.getElementById('display');
  const clearBtn = document.getElementById('clear-btn');

  function updateDisplay() {
    let formatted = currentValue.replace('.', ',');
    
    // Динамическое уменьшение шрифта при длинных числах
    if (formatted.length > 6) {
      display.style.fontSize = '40px';
    } else if (formatted.length > 8) {
      display.style.fontSize = '32px';
    } else {
      display.style.fontSize = '64px';
    }

    display.textContent = formatted;
    clearBtn.textContent = currentValue === '0' && !previousValue ? 'AC' : 'C';
  }

  function clearActiveOperators() {
    document.querySelectorAll('.btn-op').forEach(btn => btn.classList.remove('active'));
  }

  function inputDigit(digit) {
    clearActiveOperators();
    if (waitingForSecondOperand) {
      currentValue = digit;
      waitingForSecondOperand = false;
    } else {
      currentValue = currentValue === '0' ? digit : currentValue + digit;
    }
    updateDisplay();
  }

  function inputDecimal() {
    clearActiveOperators();
    if (waitingForSecondOperand) {
      currentValue = '0.';
      waitingForSecondOperand = false;
      updateDisplay();
      return;
    }
    if (!currentValue.includes('.')) {
      currentValue += '.';
      updateDisplay();
    }
  }

  function handleOperator(nextOperator) {
    clearActiveOperators();
    
    // Подсветка текущего оператора
    if (nextOperator === '÷') document.getElementById('op-div').classList.add('active');
    if (nextOperator === '×') document.getElementById('op-mul').classList.add('active');
    if (nextOperator === '-') document.getElementById('op-sub').classList.add('active');
    if (nextOperator === '+') document.getElementById('op-add').classList.add('active');

    const inputValue = parseFloat(currentValue);

    if (previousValue !== null && !waitingForSecondOperand) {
      const result = calculate(previousValue, inputValue, selectedOperator);
      currentValue = String(result);
      previousValue = result;
      updateDisplay();
    } else {
      previousValue = inputValue;
    }

    waitingForSecondOperand = true;
    selectedOperator = nextOperator;
  }

  function calculate(first, second, op) {
    switch (op) {
      case '+': return first + second;
      case '-': return first - second;
      case '×': return first * second;
      case '÷': return second === 0 ? 'Ошибка' : first / second;
      default: return second;
    }
  }

  function handleEquals() {
    clearActiveOperators();
    if (selectedOperator === null || waitingForSecondOperand) return;

    const result = calculate(previousValue, parseFloat(currentValue), selectedOperator);
    currentValue = String(result);
    previousValue = null;
    selectedOperator = null;
    waitingForSecondOperand = false;
    updateDisplay();
  }

  function handleClear() {
    clearActiveOperators();
    currentValue = '0';
    previousValue = null;
    selectedOperator = null;
    waitingForSecondOperand = false;
    updateDisplay();
  }

  function handleToggleSign() {
    currentValue = String(-parseFloat(currentValue));
    updateDisplay();
  }

  function handlePercent() {
    currentValue = String(parseFloat(currentValue) / 100);
    updateDisplay();
  }
</script>

</body>
</html>
