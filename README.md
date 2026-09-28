desi_gym_app/
├── app.py
├── desi_food.json
├── make_images.py
├── templates/
│   └── index.html
└── static/
    ├── logo.jpg      ← aapka DS logo
    └── (6 exercise images, make_images.py banayega)
    # app.py - Desi Gym + Diet Chart (100% OFFLINE)
# Flask sirf local server chalane ke liye hai. Koi API / internet / online database nahi.
from flask import Flask, render_template, send_from_directory

app = Flask(__name__)

@app.route("/")
def home():
    # Poori app ek hi page (templates/index.html) hai, 5 sections ke saath
    return render_template("index.html")

@app.route("/desi_food.json")
def food_data():
    # Local desi_food.json file JavaScript ko bhejta hai (calculator ke liye)
    return send_from_directory(app.root_path, "desi_food.json")

if __name__ == "__main__":
    # 0.0.0.0 = same WiFi ke phone se bhi khul jayegi
    app.run(host="0.0.0.0", port=5000, debug=FalsFalse)
    <!DOCTYPE html>
<html lang="hi">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Desi Gym + Diet Chart - Offline</title>
<link rel="icon" href="/static/logo.jpg">
<style>
/* ===== Theme: Orange + Green (desi style) ===== */
:root{--o:#ef6c00;--g:#2e7d32;--bg:#fff8ec;--ink:#3e2723}
*{box-sizing:border-box;margin:0}
body{font-family:"Noto Sans Devanagari",Mangal,system-ui,sans-serif;background:var(--bg);color:var(--ink);padding-bottom:76px}
header{background:linear-gradient(90deg,var(--o),var(--g));color:#fff;display:flex;align-items:center;justify-content:center;gap:10px;padding:10px;font-size:1.2rem;font-weight:700}
/* Logo: header me chhota gol, Home page par bada */
.logo-s{width:42px;height:42px;border-radius:50%;object-fit:cover;border:2px solid #fff}
.logo-big{display:block;width:170px;height:170px;margin:8px auto 14px;border-radius:24px;object-fit:cover;box-shadow:0 4px 14px #0003}
.page{display:none;max-width:760px;margin:auto;padding:14px}
.page.on{display:block}
h2{color:var(--g);margin:6px 0 10px}
.card{background:#fff;border-radius:14px;padding:12px;margin:10px 0;box-shadow:0 2px 8px #0002;border-left:6px solid var(--o)}
.btn{display:block;width:100%;min-height:52px;margin:10px 0;border:0;border-radius:12px;font-size:1.1rem;font-weight:700;color:#fff;background:var(--o);cursor:pointer}
.btn.green{background:var(--g)}
.btn.off{background:#ddd;color:#555}
.row{display:flex;align-items:center;justify-content:space-between;gap:8px}
input[type=number]{width:74px;padding:8px;font-size:1rem;border:2px solid var(--o);border-radius:8px;text-align:center}
.big{font-size:2.6rem;font-weight:800;color:var(--g);text-align:center}
.grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(230px,1fr));gap:12px}
.grid img{width:100%;border-radius:10px}
.tag{display:inline-block;background:var(--g);color:#fff;border-radius:20px;padding:2px 10px;font-size:.85rem}
label.chk{display:block;padding:8px 0;font-size:1.05rem}
nav{position:fixed;bottom:0;left:0;right:0;display:flex;background:#fff;border-top:3px solid var(--g)}
nav button{flex:1;border:0;background:none;padding:8px 2px;font-size:.72rem;color:var(--ink)}
nav button.on{background:var(--o);color:#fff}
nav span{display:block;font-size:1.3rem}
</style>
</head>
<body>
<header><img class="logo-s" src="/static/logo.jpg" alt="DS logo"> Desi Gym + Diet Chart - Offline</header>

<!-- ===== PAGE 1: HOME ===== -->
<section class="page on" id="p-home">
  <img class="logo-big" src="/static/logo.jpg" alt="DS logo">
  <h2 style="text-align:center">Desi Gym + Diet Chart - Offline</h2>
  <div class="card">Namaste! Apna goal chuno 👇</div>
  <button class="btn" onclick="setGoal('gain')">💪 Weight Badhana Hai (Gain)</button>
  <button class="btn green" onclick="setGoal('loss')">🔥 Weight Ghatana Hai (Loss)</button>
  <!-- static/chart.jpg rakhoge to yahan dikhegi, warna chhup jayegi -->
  <img src="/static/chart.jpg" alt="" style="width:100%;border-radius:12px" onerror="this.remove()">
</section>

<!-- ===== PAGE 2: DESI PROTEIN CALCULATOR ===== -->
<section class="page" id="p-calc">
  <h2>🧮 Desi Protein Calculator</h2>
  <div id="foods">Loading...</div>
  <div class="card">
    <div class="row"><b>Total Protein</b></div>
    <div class="big"><span id="total">0</span> g</div>
    <div style="text-align:center;font-size:1.3rem;font-weight:700;color:var(--o)">🔥 <span id="cal">0</span> Cal</div>
    <div id="formula" style="text-align:center"></div>
  </div>
  <button class="btn off" onclick="resetQty()">🔄 Sab Zero Karo</button>
</section>

<!-- ===== PAGE 3: GYM EXERCISE LIBRARY ===== -->
<section class="page" id="p-gym">
  <h2>🏋️ Gym Exercise Library</h2>
  <div class="card" style="text-align:center">
    <b>⏱️ Workout Timer</b>
    <div class="big" id="tm">30:00</div>
    <div class="row">
      <button class="btn" id="tGo" onclick="timerToggle()">▶ Start</button>
      <button class="btn off" onclick="timerReset()">🔄 Reset</button>
    </div>
  </div>
  <div class="grid" id="gym"></div>
</section>

<!-- ===== PAGE 4: DESI DIET CHART ===== -->
<section class="page" id="p-diet">
  <h2>🍛 Desi Diet Chart</h2>
  <div class="row">
    <button class="btn" id="dGain" onclick="showDiet('gain')">💪 Gain 3000</button>
    <button class="btn" id="dLoss" onclick="showDiet('loss')">🔥 Loss 1800</button>
  </div>
  <div id="diet"></div>
</section>

<!-- ===== PAGE 5: MY TRACKER ===== -->
<section class="page" id="p-track">
  <h2>📒 My Tracker</h2>
  <div class="card"><div>Aaj ka Protein</div><div class="big"><span id="tp">0</span> g</div></div>
  <div class="card"><b>Aaj kaunsa workout kiya?</b><div id="tw"></div></div>
  <div class="card">
    <b>Wajan (kg)</b><br>
    <input type="number" id="wt" step="0.1" placeholder="65.5">
    <button class="btn green" onclick="saveTracker()">💾 Save Karo</button>
    <div id="msg"></div>
  </div>
</section>

<nav id="nav"></nav>

<script>
/* ===== Chhote helper functions ===== */
const $ = id => document.getElementById(id);
const load = (k, d) => { try { return JSON.parse(localStorage.getItem(k)) ?? d } catch (e) { return d } };
const save = (k, v) => localStorage.setItem(k, JSON.stringify(v));   // sab kuch phone/browser me save
const today = () => new Date().toLocaleDateString('en-CA');           // aaj ki date (YYYY-MM-DD)
const round = n => Math.round(n * 10) / 10;

/* ===== Page badalna ===== */
function show(p) {
  document.querySelectorAll('.page').forEach(s => s.classList.toggle('on', s.id === 'p-' + p));
  document.querySelectorAll('nav button').forEach(b => b.classList.toggle('on', b.dataset.p === p));
  if (p === 'track') renderTracker();
  scrollTo(0, 0);
}
[['home','🏠','Home'],['calc','🧮','Protein'],['gym','🏋️','Gym'],['diet','🍛','Diet'],['track','📒','Tracker']]
  .forEach(([p, icon, name]) => {
    const b = document.createElement('button');
    b.dataset.p = p; b.innerHTML = `<span>${icon}</span>${name}`; b.onclick = () => show(p);
    $('nav').appendChild(b);
  });

/* Home ke button: goal save karo aur diet page kholo */
function setGoal(g) { showDiet(g); show('diet'); }

/* ===== PAGE 2: Protein + Calorie Calculator (data desi_food.json se) ===== */
let foods = [];
fetch('/desi_food.json').then(r => r.json()).then(d => { foods = Array.isArray(d) ? d : d.foods; buildFoods(); calc(); })
  .catch(() => $('foods').textContent = '❌ desi_food.json nahi mili. Terminal me "python app.py" chalao aur http://localhost:5000 se kholo (HTML file seedha mat kholo).');

function buildFoods() {
  const s = load('qty', {});
  const q0 = s.date === today() ? (s.q || {}) : {};   // aaj ka saved data; na mile to JSON ka "default" (purana data hua to bhi 0 nahi dikhega)
  $('foods').innerHTML = foods.map((f, i) => `
    <div class="card row">
      <div><b>${f.name}</b><br><small>${f.protein}g protein • ${f.calorie} Cal</small></div>
      <div style="text-align:center"><input type="number" min="0" step="0.5" id="q-${i}"
        value="${q0[f.name] ?? (f.default || 0)}" oninput="calc(true)"><br><small id="r-${i}"></small></div>
    </div>`).join('');
}

function calc(persist) {
  let protein = 0, cal = 0, q = {}, parts = [];
  foods.forEach((f, i) => {
    const n = Math.max(0, +$('q-' + i).value || 0);
    const p = round(n * f.protein), c = round(n * f.calorie);   // total = quantity x (protein / calorie)
    q[f.name] = n; protein += p; cal += c;
    $('r-' + i).textContent = `${n} × ${f.protein} = ${p}g`;
    if (p > 0) parts.push(`${f.name.split(' (')[0]} ${p}`);       // jaise: Roti 6 + Dahi 3 + Chana 19
  });
  protein = round(protein); cal = round(cal);
  $('total').textContent = protein;
  $('cal').textContent = cal;
  $('formula').textContent = parts.length ? parts.join(' + ') + ` = ${protein}g` : '';
  if (persist) {                                    // sirf user ke badalne par save (demo values Tracker me nahi jaati)
    save('qty', { date: today(), q });
    const t = getTracker(); t.protein = protein; save('tracker', t);   // Tracker page ke liye total
  }
}
function resetQty() { foods.forEach((f, i) => $('q-' + i).value = 0); calc(true); }

/* ===== PAGE 3: Gym Exercises (photo = static folder ki offline SVG) ===== */
const exercises = [
  { img: 'desi_dand', hi: 'देसी दंड',        en: 'Desi Dand', body: 'Chest, Shoulders, Back',   sets: '2 set × 10-15 reps' },
  { img: 'squats',    hi: 'बैठक (स्क्वाट)',   en: 'Squats',    body: 'Legs, Glutes',             sets: '2 set × 15 reps' },
  { img: 'pullups',   hi: 'पुल-अप्स',        en: 'Pull-ups',  body: 'Back, Biceps',             sets: '2 set × 6-8 reps' },
  { img: 'pushups',   hi: 'पुश-अप्स',        en: 'Push-ups',  body: 'Chest, Triceps',           sets: '2 set × 8-10 reps' },
  { img: 'deadlift',  hi: 'डेडलिफ्ट',        en: 'Deadlift',  body: 'Back, Hamstrings, Glutes', sets: '2 set × 8-10 reps' },
  { img: 'plank',     hi: 'प्लैंक',          en: 'Plank',     body: 'Core / Abs',               sets: '2 set × 30-45 sec' }
];
$('gym').innerHTML = exercises.map(e => `
  <div class="card"><img src="/static/${e.img}.svg" alt="${e.en}">
  <h3>${e.hi} / ${e.en}</h3><span class="tag">${e.body}</span><p>🔁 ${e.sets}</p></div>`).join('');

/* ===== 30:00 Workout Timer (countdown) ===== */
const TIMER_SEC = 30 * 60;                          // 30 minute
let left = TIMER_SEC, endAt = 0, tick = null;
const fmt = sec => String(Math.floor(sec / 60)).padStart(2, '0') + ':' + String(sec % 60).padStart(2, '0');
function timerToggle() {
  if (tick) { clearInterval(tick); tick = null; $('tGo').textContent = '▶ Start'; return; }   // chal raha tha -> Pause
  if (left <= 0) left = TIMER_SEC;
  endAt = Date.now() + left * 1000;                 // end time se ginte hain, isliye timer galat nahi hota
  $('tm').textContent = fmt(left); $('tGo').textContent = '⏸ Pause';
  tick = setInterval(() => {
    left = Math.max(0, Math.ceil((endAt - Date.now()) / 1000));
    $('tm').textContent = fmt(left);
    if (left === 0) {                               // time khatam
      clearInterval(tick); tick = null; $('tGo').textContent = '▶ Start'; $('tm').textContent = '🎉 Done!';
      if (navigator.vibrate) navigator.vibrate([300, 150, 300]);
    }
  }, 250);
}
function timerReset() { clearInterval(tick); tick = null; left = TIMER_SEC; $('tm').textContent = fmt(left); $('tGo').textContent = '▶ Start'; }

/* ===== PAGE 4: Diet Chart (meal, khana, calories) ===== */
const diets = {
  gain: { title: '💪 Weight Gain - 3000 Cal', meals: [
    ['🌅 Subah (7-8 AM)',    '2 glass doodh + 3 kele + 10 badam', 700],
    ['☀️ Dopahar (1-2 PM)',  '4 roti + 1 katori chawal + dal + dahi + 100g paneer sabzi', 1200],
    ['🌇 Sham (5 PM)',       '2 glass sattu sharbat + 50g moongfali', 600],
    ['🌙 Raat (9-10 PM)',    '2 roti + sabzi + 1 glass garam doodh', 500]] },
  loss: { title: '🔥 Weight Loss - 1800 Cal', meals: [
    ['🌅 Subah (7-8 AM)',    '1 katori sprouts + 1 glass toned doodh + 1 kela', 400],
    ['☀️ Dopahar (1-2 PM)',  '2 roti + 1 katori dal + 1 katori dahi + bada salad + sabzi', 600],
    ['🌇 Sham (5 PM)',       '50g bhuna chana + bina cheeni ki chai + 1 seb', 300],
    ['🌙 Raat (8 PM)',       '2 roti + paneer/soya sabzi + salad', 500]] }
};
function showDiet(g) {                               // toggle button yahi chalata hai
  const d = diets[g];
  $('dGain').className = 'btn' + (g === 'gain' ? '' : ' off');
  $('dLoss').className = 'btn' + (g === 'loss' ? ' green' : ' off');
  $('diet').innerHTML = `<h3>${d.title}</h3>` + d.meals.map(m =>
    `<div class="card"><b>${m[0]}</b><br>${m[1]}<br><span class="tag">~${m[2]} Cal</span></div>`).join('')
    + `<div class="card"><b>Total: ~${d.meals.reduce((s, m) => s + m[2], 0)} Cal</b></div>`;
  save('goal', g);
}

/* ===== PAGE 5: My Tracker (localStorage) ===== */
function getTracker() {
  let t = load('tracker', {});
  if (t.date !== today()) t = { date: today(), protein: 0, workouts: [], weight: t.weight || '' };  // naya din = nayi shuruaat
  return t;
}
function renderTracker() {
  const t = getTracker();
  $('tp').textContent = t.protein;
  $('wt').value = t.weight;
  $('tw').innerHTML = exercises.map(e => `<label class="chk"><input type="checkbox"
    ${t.workouts.includes(e.en) ? 'checked' : ''} onchange="toggleWork('${e.en}', this.checked)"> ${e.hi} / ${e.en}</label>`).join('');
}
function toggleWork(name, on) {
  const t = getTracker();
  t.workouts = t.workouts.filter(x => x !== name);
  if (on) t.workouts.push(name);
  save('tracker', t);
}
function saveTracker() {
  const t = getTracker(); t.weight = $('wt').value; save('tracker', t);
  $('msg').textContent = '✅ Save ho gaya (sirf is device me)';
}

/* ===== Start ===== */
showDiet(load('goal', 'gain'));
show('home');
</script>
</body>
</html>
# make_images.py - static folder me 6 exercise images (SVG) banata hai.
# Ek baar chalao: python make_images.py
import os
os.makedirs("static", exist_ok=True)

# Har image ka drawing part (stick figure)
IMAGES = {
    "desi_dand": '<circle cx="50" cy="84" r="9" fill="#ef6c00"/><path d="M40 122L66 82L110 38L172 122"/>',
    "squats":    '<circle cx="92" cy="30" r="10" fill="#ef6c00"/><path d="M92 42L84 80L124 88L116 122M92 52L132 56"/>',
    "pullups":   '<path d="M50 14H150" stroke="#2e7d32"/><circle cx="100" cy="36" r="10" fill="#ef6c00"/><path d="M80 14L98 48L120 14M100 48V88L90 122M100 88L110 122"/>',
    "pushups":   '<circle cx="44" cy="78" r="9" fill="#ef6c00"/><path d="M56 86L170 116M76 94V122"/>',
    "deadlift":  '<circle cx="56" cy="46" r="10" fill="#ef6c00"/><path d="M64 54L106 80L100 122M78 64L82 108"/><path d="M50 108H116" stroke="#333"/><circle cx="48" cy="108" r="12" fill="#333"/><circle cx="118" cy="108" r="12" fill="#333"/>',
    "plank":     '<circle cx="44" cy="76" r="9" fill="#ef6c00"/><path d="M56 84L168 114M74 92V122H52"/>',
}

# Sab images ka same background + zameen
TEMPLATE = ('<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 200 140">'
            '<rect width="200" height="140" rx="12" fill="#fff3e0"/>'
            '<path d="M10 124H190" stroke="#2e7d32" stroke-width="4"/>'
            '<g stroke="#ef6c00" stroke-width="7" stroke-linecap="round" '
            'stroke-linejoin="round" fill="none">{}</g></svg>')

for name, shapes in IMAGES.items():
    with open(f"static/{name}.svg", "w") as f:
        f.write(TEMPLATE.format(shapes))
print("6 images ban gayi -> static folder")
 
