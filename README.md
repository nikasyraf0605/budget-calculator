
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Budget Ledger</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Fraunces:opsz,wght@9..144,400;9..144,600&family=IBM+Plex+Mono:wght@400;500;600&display=swap" rel="stylesheet">
<style>
  :root {
    --paper: #F5F1E6;
    --paper-dark: #EDE6D3;
    --ink: #16302E;
    --ink-soft: #4A6E6B;
    --gold: #B8863C;
    --gold-soft: #DCC694;
    --line: #D8D0BD;
    --good: #3F6B4F;
    --bad: #A2432C;
  }
  * { box-sizing: border-box; }
  body {
    margin: 0;
    background: var(--paper);
    color: var(--ink);
    font-family: 'IBM Plex Mono', monospace;
    min-height: 100vh;
  }
  .wrap { max-width: 720px; margin: 0 auto; padding: 32px 20px 60px; }
  h1 {
    font-family: 'Fraunces', serif;
    font-weight: 600;
    font-size: 32px;
    margin: 0 0 4px;
  }
  .sub { color: var(--ink-soft); font-size: 14px; margin: 0 0 24px; line-height: 1.5; }
  .status-row {
    display: flex; align-items: center; gap: 8px; font-size: 13px; color: var(--ink-soft);
    margin-bottom: 16px;
  }
  .dot { width: 8px; height: 8px; border-radius: 50%; background: var(--good); }
  .dot.off { background: var(--bad); }
  .card {
    border: 1px solid var(--line);
    border-radius: 10px;
    overflow: hidden;
    background: #fff;
  }
  .card-head {
    background: var(--ink);
    color: var(--paper);
    padding: 20px;
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 16px;
  }
  .field label {
    display: block;
    font-size: 11px;
    color: var(--gold-soft);
    margin-bottom: 4px;
  }
  input {
    font-family: inherit;
    font-size: 14px;
  }
  .card-head input {
    width: 100%;
    padding: 8px;
    border-radius: 6px;
    background: rgba(255,255,255,0.08);
    border: 1px solid rgba(255,255,255,0.2);
    color: var(--paper);
  }
  .card-head input::placeholder { color: rgba(245,241,230,0.45); }
  .card-body { padding: 20px; }
  .rows-head {
    display: flex; justify-content: space-between; align-items: center;
    margin-bottom: 8px; font-size: 14px; font-weight: 500;
  }
  .add-btn {
    display: flex; align-items: center; gap: 6px;
    background: none; border: none; color: var(--ink-soft);
    font-family: inherit; font-size: 12px; cursor: pointer; padding: 4px 8px;
  }
  .rows { border-top: 1px solid var(--line); }
  .row {
    display: flex; gap: 8px; align-items: center; padding: 8px 0;
    border-bottom: 1px solid var(--line);
  }
  .row input[type="text"] {
    flex: 1; padding: 8px; border-radius: 6px; border: 1px solid var(--line);
  }
  .row input[type="text"].amt {
    flex: none; width: 110px; text-align: right;
  }
  .row .del {
    background: none; border: none; color: var(--bad); cursor: pointer;
    padding: 6px; font-size: 16px; line-height: 1;
  }
  .totals {
    margin-top: 16px; padding-top: 16px; border-top: 1px solid var(--line);
    display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 12px; font-size: 14px;
  }
  .totals .label { color: var(--ink-soft); font-size: 12px; }
  .totals .value { font-size: 18px; font-weight: 600; font-variant-numeric: tabular-nums; }
  .value.balance-bad { color: var(--bad); }
  .value.balance-good { color: var(--good); }
  input::placeholder { color: #A79E86; }
  input[type="text"], input[type="text"].amt { color: var(--ink); }
  .footer-note { margin-top: 20px; font-size: 12px; color: var(--ink-soft); line-height: 1.5; }
  @media (max-width: 480px) {
    .card-head { grid-template-columns: 1fr; }
  }
</style>
</head>
<body>
<div class="wrap">
  <h1>Budget Ledger</h1>
  <p class="sub">One shared ledger — anything anyone types here updates live for everyone viewing this page.</p>

  <div class="status-row">
    <span class="dot" id="statusDot"></span>
    <span id="statusText">Connecting…</span>
  </div>

  <div class="card">
    <div class="card-head">
      <div class="field">
        <label>Focused product</label>
        <input id="product" type="text" placeholder="e.g. Anti-Diabetic &amp; Anti-Obesity Products" />
      </div>
      <div class="field">
        <label>Budgeted (RM)</label>
        <input id="budgeted" type="text" placeholder="70000.00" />
      </div>
    </div>
    <div class="card-body">
      <div class="rows-head">
        <span>Cost breakdown</span>
        <button class="add-btn" id="addRow">+ Add line</button>
      </div>
      <div class="rows" id="rows"></div>

      <div class="totals">
        <div>
          <div class="label">Budgeted</div>
          <div class="value" id="totalBudgeted">RM 0.00</div>
        </div>
        <div>
          <div class="label">Actual spent</div>
          <div class="value" id="totalSpent">RM 0.00</div>
        </div>
        <div>
          <div class="label">Balance</div>
          <div class="value" id="totalBalance">RM 0.00</div>
        </div>
      </div>
    </div>
  </div>

  <p class="footer-note">
    Changes save automatically a moment after you stop typing. Everyone with this link sees the same numbers.
  </p>
</div>

<!-- Firebase (free tier) — provides the shared live database -->
<script src="https://www.gstatic.com/firebasejs/10.12.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.12.0/firebase-firestore-compat.js"></script>
<script>
/* ============================================================
   1) PASTE YOUR FIREBASE CONFIG BELOW
      (Firebase Console → Project settings → General → Your apps → SDK setup)
   ============================================================ */
const firebaseConfig = {
  apiKey: "PASTE_ME",
  authDomain: "PASTE_ME",
  projectId: "PASTE_ME",
  storageBucket: "PASTE_ME",
  messagingSenderId: "PASTE_ME",
  appId: "PASTE_ME"
};
/* ============================================================ */

firebase.initializeApp(firebaseConfig);
const db = firebase.firestore();
const docRef = db.collection('ledger').doc('shared');

let state = { product: '', budgeted: '', rows: [{ id: 1, label: '', spent: '' }] };
let suppressWrite = false; // avoids re-writing when a snapshot update arrives
let writeTimer = null;

const productEl = document.getElementById('product');
const budgetedEl = document.getElementById('budgeted');
const rowsEl = document.getElementById('rows');
const statusDot = document.getElementById('statusDot');
const statusText = document.getElementById('statusText');

function currency(n) {
  const num = Number(n) || 0;
  return num.toLocaleString('en-MY', { minimumFractionDigits: 2, maximumFractionDigits: 2 });
}

function renderRows() {
  rowsEl.innerHTML = '';
  state.rows.forEach(row => {
    const div = document.createElement('div');
    div.className = 'row';
    div.innerHTML = `
      <input type="text" class="label" placeholder="Gold Sponsorship MYO" value="${escapeHtml(row.label)}" />
      <input type="text" class="amt" placeholder="0.00" value="${escapeHtml(row.spent)}" />
      <button class="del" title="Remove">✕</button>
    `;
    const [labelInput, amtInput] = div.querySelectorAll('input');
    labelInput.addEventListener('input', e => { row.label = e.target.value; renderTotals(); scheduleWrite(); });
    amtInput.addEventListener('input', e => {
      row.spent = e.target.value.replace(/[^0-9.]/g, '');
      if (e.target.value !== row.spent) e.target.value = row.spent;
      renderTotals(); scheduleWrite();
    });
    div.querySelector('.del').addEventListener('click', () => {
      if (state.rows.length > 1) state.rows = state.rows.filter(r => r.id !== row.id);
      renderRows(); renderTotals(); scheduleWrite();
    });
    rowsEl.appendChild(div);
  });
}

function renderTotals() {
  const totalSpent = state.rows.reduce((sum, r) => sum + (Number(r.spent) || 0), 0);
  const balance = (Number(state.budgeted) || 0) - totalSpent;
  document.getElementById('totalBudgeted').textContent = 'RM ' + currency(state.budgeted);
  document.getElementById('totalSpent').textContent = 'RM ' + currency(totalSpent);
  const balEl = document.getElementById('totalBalance');
  balEl.textContent = 'RM ' + currency(balance);
  balEl.className = 'value ' + (balance < 0 ? 'balance-bad' : 'balance-good');
}

function renderAll() {
  productEl.value = state.product;
  budgetedEl.value = state.budgeted;
  renderRows();
  renderTotals();
}

function escapeHtml(s) {
  return String(s || '').replace(/[&<>"']/g, c => ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[c]));
}

function scheduleWrite() {
  if (suppressWrite) return;
  clearTimeout(writeTimer);
  writeTimer = setTimeout(() => {
    docRef.set(state).catch(err => {
      statusText.textContent = 'Could not save — check your connection.';
      statusDot.classList.add('off');
      console.error(err);
    });
  }, 500);
}

productEl.addEventListener('input', e => { state.product = e.target.value; scheduleWrite(); });
budgetedEl.addEventListener('input', e => {
  state.budgeted = e.target.value.replace(/[^0-9.]/g, '');
  if (e.target.value !== state.budgeted) e.target.value = state.budgeted;
  renderTotals(); scheduleWrite();
});
document.getElementById('addRow').addEventListener('click', () => {
  state.rows.push({ id: Date.now(), label: '', spent: '' });
  renderRows(); scheduleWrite();
});

// Live sync: whenever the shared doc changes (by anyone), update the page.
docRef.onSnapshot(snap => {
  statusDot.classList.remove('off');
  statusText.textContent = 'Live — connected';
  if (snap.exists) {
    const data = snap.data();
    state = {
      product: data.product || '',
      budgeted: data.budgeted || '',
      rows: (data.rows && data.rows.length) ? data.rows : [{ id: 1, label: '', spent: '' }]
    };
  } else {
    docRef.set(state); // first-ever load: create the shared doc
  }
  suppressWrite = true;
  renderAll();
  suppressWrite = false;
}, err => {
  statusDot.classList.add('off');
  statusText.textContent = 'Offline — check Firebase config';
  console.error(err);
});
</script>
</body>
</html>
