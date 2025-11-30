# calculator-
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Scientific Calculator</title>
  <style>
    :root{--bg:#0f1724;--panel:#0b1220;--accent:#06b6d4;--muted:#9aa6b2}
    *{box-sizing:border-box}
    body{font-family:Inter,Segoe UI,Arial;background:linear-gradient(180deg,#071025 0%,#071a2a 100%);display:flex;align-items:center;justify-content:center;height:100vh;margin:0;color:white}
    .calc{width:420px;background:linear-gradient(180deg,var(--panel),#071428);border-radius:12px;padding:18px;box-shadow:0 10px 30px rgba(2,6,23,0.6)}
    .top{display:flex;justify-content:space-between;align-items:center;margin-bottom:12px}
    .title{font-weight:600}
    .mode{font-size:13px;color:var(--muted)}

    .display{background:#071827;border-radius:8px;padding:12px;min-height:72px;display:flex;flex-direction:column;justify-content:center;align-items:flex-end}
    .expr{color:var(--muted);font-size:14px;word-break:break-all}
    .result{font-size:24px;font-weight:600}

    .keys{display:grid;grid-template-columns:repeat(6,1fr);gap:8px;margin-top:12px}
    button.key{padding:12px;border-radius:8px;border:0;background:transparent;color:white;font-size:14px;cursor:pointer}
    button.key:hover{background:rgba(255,255,255,0.04)}
    button.op{background:linear-gradient(180deg,#062a2f,#073338);border:1px solid rgba(255,255,255,0.03)}
    button.fn{background:linear-gradient(180deg,#172532,#0f3640);border:1px solid rgba(255,255,255,0.03)}
    button.equal{grid-column:span 2;background:var(--accent);color:#042; font-weight:700}
    .wide{grid-column:span 2}

    .footer{margin-top:12px;color:var(--muted);font-size:13px;text-align:center}

    @media (max-width:480px){.calc{width:96vw}}
  </style>
</head>
<body>
  <div class="calc" role="application" aria-label="Scientific calculator">
    <div class="top">
      <div>
        <div class="title">Scientific Calculator</div>
        <div class="mode"><span id="modeText">RAD</span> • Memory: <span id="memText">0</span></div>
      </div>
      <div>
        <button id="degRadBtn" class="key op" title="Toggle Degrees/Radians">Deg/Rad</button>
        <button id="clearMem" class="key fn" title="Clear Memory">MC</button>
      </div>
    </div>

    <div class="display" id="display">
      <div class="expr" id="expr">0</div>
      <div class="result" id="result">0</div>
    </div>

    <div class="keys" id="keys">
      <!-- Row: functions -->
      <button class="key fn" data-val="sin(" title="sine">sin</button>
      <button class="key fn" data-val="cos(">cos</button>
      <button class="key fn" data-val="tan(">tan</button>
      <button class="key fn" data-val="log10(" title="log base 10">log</button>
      <button class="key fn" data-val="log(" title="natural log">ln</button>
      <button class="key fn" data-val="sqrt(">√</button>

      <!-- Row: memory & constants -->
      <button class="key" id="mPlus" title="M+">M+</button>
      <button class="key" id="mMinus" title="M-">M-</button>
      <button class="key" id="mr" title="MR">MR</button>
      <button class="key" data-val="PI" title="Pi">π</button>
      <button class="key" data-val="E" title="Euler's e">e</button>
      <button class="key" data-val="(">(</button>

      <!-- Row: numbers & ops -->
      <button class="key" data-val="7">7</button>
      <button class="key" data-val="8">8</button>
      <button class="key" data-val="9">9</button>
      <button class="key op" data-val="/">÷</button>
      <button class="key op" data-val="^">^</button>
      <button class="key" data-val=")">)</button>

      <button class="key" data-val="4">4</button>
      <button class="key" data-val="5">5</button>
      <button class="key" data-val="6">6</button>
      <button class="key op" data-val="*">×</button>
      <button class="key fn" data-val="exp(" title="e^x">exp</button>
      <button class="key" id="factBtn" title="factorial">!</button>

      <button class="key" data-val="1">1</button>
      <button class="key" data-val="2">2</button>
      <button class="key" data-val="3">3</button>
      <button class="key op" data-val="-">−</button>
      <button class="key fn" data-val="pow(" title="power">pow</button>
      <button class="key op" data-val="%">%</button>

      <button class="key wide" data-val="0">0</button>
      <button class="key" data-val=".">.</button>
      <button class="key op" data-val="+">+</button>
      <button class="key equal" id="equals">=</button>

      <!-- Bottom row actions -->
      <button class="key" id="clear">C</button>
      <button class="key" id="allClear">AC</button>
      <button class="key" id="ansBtn">Ans</button>
      <button class="key" id="copyRes">Copy</button>
      <button class="key" id="invBtn" title="Inverse (1/x)">1/x</button>
      <button class="key" id="sqrtBtn" title="square">x²</button>
    </div>

    <div class="footer">Supports: + - × ÷ ^ ( ) % · trig, log, ln, exp, pow, factorial. Click buttons or type on keyboard.</div>
  </div>

<script>
  // State
  let exprEl = document.getElementById('expr');
  let resEl = document.getElementById('result');
  let modeText = document.getElementById('modeText');
  let memText = document.getElementById('memText');

  let expression = '';
  let lastAnswer = 0;
  let memory = 0;
  let useDegrees = false; // false = radians

  function updateDisplay(){
    exprEl.textContent = expression || '0';
    resEl.textContent = lastAnswer === Infinity ? '∞' : (String(lastAnswer));
    modeText.textContent = useDegrees ? 'DEG' : 'RAD';
    memText.textContent = memory;
  }

  // helper functions
  function fact(n){
    n = Number(n);
    if(!Number.isFinite(n) || n<0 || Math.floor(n)!==n) return NaN;
    let res=1;
    for(let i=2;i<=n;i++) res*=i;
    return res;
  }

  function sinW(x){ return useDegrees ? Math.sin(x*Math.PI/180) : Math.sin(x); }
  function cosW(x){ return useDegrees ? Math.cos(x*Math.PI/180) : Math.cos(x); }
  function tanW(x){ return useDegrees ? Math.tan(x*Math.PI/180) : Math.tan(x); }

  // safe evaluate: build a function with allowed helpers
  function safeEval(raw){
    try{
      if(!raw) return 0;
      // replace common unicode operators
      let e = raw.replace(/÷/g,'/').replace(/×/g,'*').replace(/–/g,'-');
      // replace caret with JS power
      e = e.replace(/\^/g,'**');
      // handle percentage: convert n% to (n/100)
      e = e.replace(/(\d+(?:\.\d+)?)%/g,'($1/100)');

      // replace constants
      e = e.replace(/\bPI\b/g,'Math.PI');
      e = e.replace(/\bE\b/g,'Math.E');

      // handle factorial: replace occurrences like 5! or (3+2)! with fact(...)
      // simple regex to catch number or parenthesized expression before '!'
      e = e.replace(/(\([^()]+\)|\d+\.?\d*)!/g,'fact($1)');

      // final expression string
      const expr = e;

      // create function with allowed identifiers (we inject wrappers)
      const fn = new Function('sin','cos','tan','log','log10','sqrt','pow','exp','fact','PI','E','Math', 'return ('+expr+');');
      const value = fn(sinW,cosW,tanW,Math.log,Math.log10||((x)=>Math.log(x)/Math.LN10),Math.sqrt,Math.pow,Math.exp,fact,Math.PI,Math.E,Math);
      return value;
    }catch(err){
      return NaN;
    }
  }

  // keyboard & button handling
  document.getElementById('keys').addEventListener('click', (ev)=>{
    const b = ev.target.closest('button');
    if(!b) return;
    const val = b.dataset.val;
    if(val!==undefined){
      // insertion
      expression += val;
    } else {
      // actions
      if(b.id==='equals'){
        const v = safeEval(expression);
        if(Number.isFinite(v)) lastAnswer = v; else lastAnswer = NaN;
        expression = '';
      } else if(b.id==='clear'){
        expression = expression.slice(0,-1);
      } else if(b.id==='allClear'){
        expression = '';
        lastAnswer = 0;
      } else if(b.id==='ansBtn'){
        expression += String(lastAnswer);
      } else if(b.id==='copyRes'){
        navigator.clipboard?.writeText(String(lastAnswer
