import React, { useState, useEffect } from 'react';
import { Plus, Trash2, Copy, Check, Save, FolderOpen, ArrowRight } from 'lucide-react';

const palette = {
  paper: '#F5F1E6',
  paperDark: '#EDE6D3',
  ink: '#16302E',
  inkSoft: '#4A6E6B',
  gold: '#B8863C',
  goldSoft: '#DCC694',
  line: '#D8D0BD',
  good: '#3F6B4F',
  bad: '#A2432C',
};

function genCode() {
  const chars = 'ABCDEFGHJKMNPQRSTUVWXYZ23456789';
  let out = '';
  for (let i = 0; i < 6; i++) out += chars[Math.floor(Math.random() * chars.length)];
  return out;
}

function currency(n) {
  const num = Number(n) || 0;
  return num.toLocaleString('en-MY', { minimumFractionDigits: 2, maximumFractionDigits: 2 });
}

export default function BudgetLedger() {
  const [product, setProduct] = useState('');
  const [budgeted, setBudgeted] = useState('');
  const [rows, setRows] = useState([{ id: 1, label: '', spent: '' }]);
  const [code, setCode] = useState(null);
  const [copied, setCopied] = useState(false);
  const [loadInput, setLoadInput] = useState('');
  const [status, setStatus] = useState('');
  const [saving, setSaving] = useState(false);
  const [loading, setLoading] = useState(false);
  const [history, setHistory] = useState([]);
  const [historyLoaded, setHistoryLoaded] = useState(false);

  useEffect(() => {
    (async () => {
      try {
        const res = await window.storage.get('ledger-history', false);
        if (res && res.value) setHistory(JSON.parse(res.value));
      } catch (e) {
        // no history saved yet — that's fine
      } finally {
        setHistoryLoaded(true);
      }
    })();
  }, []);

  const totalSpent = rows.reduce((sum, r) => sum + (Number(r.spent) || 0), 0);
  const balance = (Number(budgeted) || 0) - totalSpent;

  const addRow = () => setRows([...rows, { id: Date.now(), label: '', spent: '' }]);
  const removeRow = (id) => setRows(rows.length > 1 ? rows.filter((r) => r.id !== id) : rows);
  const updateRow = (id, field, value) =>
    setRows(rows.map((r) => (r.id === id ? { ...r, [field]: value } : r)));

  async function handleSave() {
    if (!product.trim() || !budgeted) {
      setStatus('Add a product name and a budgeted amount before saving.');
      return;
    }
    setSaving(true);
    setStatus('');
    const newCode = genCode();
    const data = {
      product,
      budgeted: Number(budgeted),
      rows,
      totalSpent,
      balance,
      savedAt: new Date().toISOString(),
    };
    try {
      const result = await window.storage.set(`calc:${newCode}`, JSON.stringify(data), true);
      if (!result) throw new Error('save failed');
      setCode(newCode);
      const entry = {
        code: newCode,
        product,
        budgeted: Number(budgeted),
        balance,
        savedAt: data.savedAt,
      };
      const nextHistory = [entry, ...history].slice(0, 20);
      setHistory(nextHistory);
      try {
        await window.storage.set('ledger-history', JSON.stringify(nextHistory), false);
      } catch (e) {
        // history is a nice-to-have; ignore if it fails
      }
    } catch (e) {
      setStatus("Couldn't save this calculation. Try again.");
    } finally {
      setSaving(false);
    }
  }

  async function handleLoad(codeToLoad) {
    const target = (codeToLoad || loadInput).trim().toUpperCase();
    if (!target) return;
    setLoading(true);
    setStatus('');
    try {
      const res = await window.storage.get(`calc:${target}`, true);
      if (!res || !res.value) {
        setStatus(`No calculation found for code ${target}.`);
        setLoading(false);
        return;
      }
      const data = JSON.parse(res.value);
      setProduct(data.product || '');
      setBudgeted(String(data.budgeted ?? ''));
      setRows(data.rows && data.rows.length ? data.rows : [{ id: 1, label: '', spent: '' }]);
      setCode(target);
      setLoadInput('');
    } catch (e) {
      setStatus(`Couldn't find a calculation for code ${target}.`);
    } finally {
      setLoading(false);
    }
  }

  function handleCopy() {
    if (!code) return;
    navigator.clipboard?.writeText(code);
    setCopied(true);
    setTimeout(() => setCopied(false), 1500);
  }

  function handleNew() {
    setProduct('');
    setBudgeted('');
    setRows([{ id: 1, label: '', spent: '' }]);
    setCode(null);
    setStatus('');
  }

  return (
    <div
      style={{ background: palette.paper, minHeight: '100%', fontFamily: "'IBM Plex Mono', monospace", color: palette.ink }}
      className="w-full p-6"
    >
      <style>{`
        @import url('https://fonts.googleapis.com/css2?family=Fraunces:opsz,wght@9..144,400;9..144,600&family=IBM+Plex+Mono:wght@400;500;600&display=swap');
        .ledger-num { font-variant-numeric: tabular-nums; }
        input::placeholder { color: #A79E86; }
      `}</style>

      <div className="max-w-3xl mx-auto">
        <div className="flex items-baseline justify-between mb-1 flex-wrap gap-2">
          <h1 style={{ fontFamily: "'Fraunces', serif", color: palette.ink }} className="text-3xl font-semibold">
            Budget Ledger
          </h1>
          {code && (
            <div className="flex items-center gap-2 text-sm" style={{ color: palette.inkSoft }}>
              <span>Code</span>
              <span className="ledger-num px-2 py-0.5 rounded" style={{ background: palette.paperDark, letterSpacing: '0.05em' }}>
                {code}
              </span>
              <button onClick={handleCopy} className="p-1 rounded hover:opacity-70" style={{ color: palette.gold }} title="Copy code">
                {copied ? <Check size={16} /> : <Copy size={16} />}
              </button>
            </div>
          )}
        </div>
        <p className="text-sm mb-6" style={{ color: palette.inkSoft }}>
          Track what's budgeted against what's spent. Save it and you get a code — hand that to anyone and they can open this exact calculation.
        </p>

        <div className="flex gap-2 mb-6">
          <input
            value={loadInput}
            onChange={(e) => setLoadInput(e.target.value)}
            placeholder="Enter a code to open a saved calculation"
            className="flex-1 px-3 py-2 rounded border text-sm ledger-num"
            style={{ borderColor: palette.line, background: '#fff' }}
            onKeyDown={(e) => {
              if (e.key === 'Enter') handleLoad();
            }}
          />
          <button
            onClick={() => handleLoad()}
            disabled={loading}
            className="px-3 py-2 rounded text-sm flex items-center gap-1.5"
            style={{ background: palette.ink, color: palette.paper }}
          >
            <FolderOpen size={15} /> Open
          </button>
        </div>

        <div className="rounded-lg overflow-hidden border" style={{ borderColor: palette.line, background: '#fff' }}>
          <div className="p-5" style={{ background: palette.ink, color: palette.paper }}>
            <div className="grid grid-cols-2 gap-4">
              <div>
                <label className="block text-xs mb-1" style={{ color: palette.goldSoft }}>
                  Focused product
                </label>
                <input
                  value={product}
                  onChange={(e) => setProduct(e.target.value)}
                  placeholder="e.g. Anti-Diabetic & Anti-Obesity Products"
                  className="w-full px-2 py-1.5 rounded text-sm"
                  style={{ background: 'rgba(255,255,255,0.08)', color: palette.paper, border: '1px solid rgba(255,255,255,0.2)' }}
                />
              </div>
              <div>
                <label className="block text-xs mb-1" style={{ color: palette.goldSoft }}>
                  Budgeted (RM)
                </label>
                <input
                  value={budgeted}
                  onChange={(e) => setBudgeted(e.target.value.replace(/[^0-9.]/g, ''))}
                  placeholder="70000.00"
                  className="w-full px-2 py-1.5 rounded text-sm ledger-num"
                  style={{ background: 'rgba(255,255,255,0.08)', color: palette.paper, border: '1px solid rgba(255,255,255,0.2)' }}
                />
              </div>
            </div>
          </div>

          <div className="p-5">
            <div className="flex justify-between items-center mb-2">
              <span className="text-sm font-medium">Cost breakdown</span>
              <button onClick={addRow} className="flex items-center gap-1 text-xs px-2 py-1 rounded" style={{ color: palette.inkSoft }}>
                <Plus size={14} /> Add line
              </button>
            </div>
            <div className="divide-y" style={{ borderColor: palette.line }}>
              {rows.map((row) => (
                <div key={row.id} className="flex items-center gap-2 py-2">
                  <input
                    value={row.label}
                    onChange={(e) => updateRow(row.id, 'label', e.target.value)}
                    placeholder="Gold Sponsorship MYO"
                    className="flex-1 px-2 py-1.5 rounded border text-sm"
                    style={{ borderColor: palette.line }}
                  />
                  <input
                    value={row.spent}
                    onChange={(e) => updateRow(row.id, 'spent', e.target.value.replace(/[^0-9.]/g, ''))}
                    placeholder="0.00"
                    className="w-28 px-2 py-1.5 rounded border text-sm text-right ledger-num"
                    style={{ borderColor: palette.line }}
                  />
                  <button onClick={() => removeRow(row.id)} className="p-1.5 rounded hover:opacity-60" style={{ color: palette.bad }}>
                    <Trash2 size={15} />
                  </button>
                </div>
              ))}
            </div>

            <div className="mt-4 pt-4 border-t grid grid-cols-3 gap-3 text-sm" style={{ borderColor: palette.line }}>
              <div>
                <div style={{ color: palette.inkSoft }}>Budgeted</div>
                <div className="ledger-num text-lg font-semibold">RM {currency(budgeted)}</div>
              </div>
              <div>
                <div style={{ color: palette.inkSoft }}>Actual spent</div>
                <div className="ledger-num text-lg font-semibold">RM {currency(totalSpent)}</div>
              </div>
              <div>
                <div style={{ color: palette.inkSoft }}>Balance</div>
                <div className="ledger-num text-lg font-semibold" style={{ color: balance < 0 ? palette.bad : palette.good }}>
                  RM {currency(balance)}
                </div>
              </div>
            </div>
          </div>
        </div>

        {status && (
          <div className="mt-3 text-sm" style={{ color: palette.bad }}>
            {status}
          </div>
        )}

        <div className="mt-4 flex gap-2">
          <button
            onClick={handleSave}
            disabled={saving}
            className="px-4 py-2 rounded text-sm flex items-center gap-1.5"
            style={{ background: palette.gold, color: palette.ink }}
          >
            <Save size={15} /> {saving ? 'Saving…' : code ? 'Save as new code' : 'Save & get a code'}
          </button>
          <button onClick={handleNew} className="px-4 py-2 rounded text-sm border" style={{ borderColor: palette.line, color: palette.inkSoft }}>
            Start a new one
          </button>
        </div>

        {historyLoaded && history.length > 0 && (
          <div className="mt-8">
            <div className="text-sm font-medium mb-2">Your saved calculations</div>
            <div className="rounded border divide-y" style={{ borderColor: palette.line, background: '#fff' }}>
              {history.map((h) => (
                <button
                  key={h.code + h.savedAt}
                  onClick={() => handleLoad(h.code)}
                  className="w-full flex items-center justify-between px-3 py-2 text-sm text-left hover:opacity-70"
                >
                  <span>{h.product || 'Untitled'}</span>
                  <span className="flex items-center gap-3 ledger-num" style={{ color: palette.inkSoft }}>
                    <span>{h.code}</span>
                    <ArrowRight size={13} />
                  </span>
                </button>
              ))}
            </div>
          </div>
        )}
      </div>
    </div>
  );
}
