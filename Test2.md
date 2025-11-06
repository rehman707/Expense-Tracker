---
created: 2025-11-07T01:40
updated: 2025-11-07T01:40
---
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Monthly Budget with Wants Tracker (INR)</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/3.9.1/chart.min.js"></script>
  <style>
    /* (kept mostly the same styling; compacted slightly) */
    *{box-sizing:border-box;margin:0;padding:0}
    body{font-family:Segoe UI, Tahoma, Geneva, Verdana, sans-serif;background:linear-gradient(135deg,#667eea 0%,#764ba2 100%);padding:20px;min-height:100vh}
    .container{max-width:1400px;margin:0 auto;background:#fff;border-radius:15px;box-shadow:0 20px 60px rgba(0,0,0,.3);overflow:hidden;padding-bottom:30px}
    .topbar{display:flex;align-items:center;justify-content:space-between;padding:14px 22px;border-bottom:1px solid #eee}
    .month-nav{display:flex;align-items:center;gap:12px}
    .btn{background:#667eea;color:#fff;border:none;padding:8px 12px;border-radius:8px;cursor:pointer;font-weight:600}
    .btn.ghost{background:transparent;color:#667eea;border:2px solid #667eea}
    .small{padding:6px 10px;font-size:13px;border-radius:6px}
    .title{font-size:18px;font-weight:700;color:#2c3e50}
    .controls{display:flex;gap:10px;align-items:center}
    .tabs{display:flex;background:#f8f9fa;border-bottom:2px solid #dee2e6}
    .tab{flex:1;padding:14px;text-align:center;cursor:pointer;font-weight:600;font-size:15px;color:#495057;transition:all .2s;border-bottom:3px solid transparent}
    .tab:hover{background:#e9ecef}
    .tab.active{background:#fff;color:#667eea;border-bottom:3px solid #667eea}
    .tab-content{display:none;padding:22px;animation:fadeIn .25s}
    .tab-content.active{display:block}
    @keyframes fadeIn{from{opacity:0;transform:translateY(6px)}to{opacity:1;transform:none}}
    .summary-cards{display:grid;grid-template-columns:repeat(auto-fit,minmax(200px,1fr));gap:16px;margin:18px 0}
    .card{background:linear-gradient(135deg,#667eea 0%,#764ba2 100%);color:#fff;padding:14px;border-radius:10px;box-shadow:0 4px 12px rgba(0,0,0,.08)}
    .card h3{font-size:13px;font-weight:500;opacity:.95}
    .card .amount{font-size:22px;font-weight:700}
    .budget-layout{display:grid;grid-template-columns:1fr 420px;gap:24px;margin-top:10px;padding:0 22px}
    table{width:100%;border-collapse:collapse;margin-bottom:18px;background:#fff;border-radius:8px;overflow:hidden;box-shadow:0 2px 8px rgba(0,0,0,.06)}
    thead{background:linear-gradient(135deg,#667eea 0%,#764ba2 100%);color:#fff}
    th,td{padding:12px 14px;text-align:left}
    tbody tr:nth-child(even){background:#f8f9fa}
    input[type=text],input[type=number]{width:100%;padding:8px;border:2px solid #e9e9e9;border-radius:6px}
    .section-header{background:#f8f9fa;font-weight:700;color:#2c3e50;font-size:15px;padding:10px 12px}
    .chart-container{background:#fff;padding:16px;border-radius:10px;box-shadow:0 2px 8px rgba(0,0,0,.06);position:sticky;top:16px}
    canvas{max-height:360px}
    .linked-wants{margin-top:14px;padding:12px;background:#f8f9fa;border-radius:10px;border-left:4px solid #667eea}
    .linked-item{padding:10px;background:#fff;margin-bottom:8px;border-radius:6px;display:flex;justify-content:space-between;align-items:center}
    .priority-badge{padding:5px 10px;border-radius:16px;font-size:12px;font-weight:700;color:#fff}
    .buy-now{background:#28a745}
    .consider-later{background:#ffc107;color:#333}
    .avoid{background:#dc3545}
    .bottom-charts{display:grid;grid-template-columns:1fr 1fr;gap:18px;padding:0 22px;margin-top:20px}
    @media (max-width:1100px){.budget-layout{grid-template-columns:1fr}.chart-container{position:static}}
    @media (max-width:600px){.bottom-charts{grid-template-columns:1fr}}
    .small-muted{font-size:13px;color:#6c757d}
  </style>
</head>
<body>
  <div class="container">
    <div class="topbar">
      <div class="month-nav">
        <button class="btn small" id="prevMonthBtn">◀ Prev</button>
        <div style="min-width:200px;text-align:center">
          <div class="title" id="monthDisplay">Loading…</div>
          <div class="small-muted" id="monthLabel">YYYY - MM</div>
        </div>
        <button class="btn small" id="nextMonthBtn">Next ▶</button>
      </div>

      <div class="controls">
        <button class="btn ghost small" id="saveBtn">Save</button>
        <button class="btn small" id="resetBtn" style="background:#e55353">Reset Month</button>
      </div>
    </div>

    <div class="tabs">
      <div class="tab active" onclick="switchTab(0)">Monthly Budget</div>
      <div class="tab" onclick="switchTab(1)">Wants Priority Tracker</div>
    </div>

    <div class="tab-content active" id="budget-tab">
      <div style="padding:18px 22px 6px">
        <div class="summary-cards">
          <div class="card">
            <h3>Salary</h3>
            <div class="amount">₹<span id="salary-display">25,000</span></div>
          </div>
          <div class="card">
            <h3>Total Income</h3>
            <div class="amount">₹<span id="income-display">21,000</span></div>
          </div>
          <div class="card">
            <h3>Total Expenses</h3>
            <div class="amount">₹<span id="expenses-display">0</span></div>
          </div>
          <div class="card">
            <h3>Remaining Balance</h3>
            <div class="amount">₹<span id="balance-display">21,000</span></div>
          </div>
        </div>
      </div>

      <div class="budget-layout">
        <div>
          <table id="budget-table" aria-label="Budget table">
            <thead>
              <tr><th>Category</th><th>Description</th><th>Amount (₹)</th><th>Section</th><th>Remaining (₹)</th></tr>
            </thead>
            <tbody id="budget-body"></tbody>
          </table>

          <div class="linked-wants">
            <h3 style="margin-bottom:10px">Linked Wants (From Tracker)</h3>
            <div id="linked-wants-list"></div>
          </div>
        </div>

        <div class="chart-container">
          <h3 style="text-align:center;margin-bottom:12px;color:#2c3e50">Budget Allocation</h3>
          <canvas id="budgetChart"></canvas>
        </div>
      </div>
    </div>

    <div class="tab-content" id="tracker-tab">
      <div style="padding:18px 22px 6px">
        <h2 style="margin-bottom:12px">Wants Priority Tracker</h2>
        <table>
          <thead>
            <tr>
              <th>Item Name</th><th>Cost (₹)</th><th>Want (0-10)</th><th>Useful (0-10)</th><th>Afford (0-10)</th><th>Value (0-10)</th><th>Improve (0-10)</th><th>Total (0-50)</th><th>Priority %</th><th>Decision</th>
            </tr>
          </thead>
          <tbody id="wants-body"></tbody>
        </table>
        <div class="chart-container" style="margin-top:10px">
          <h3 style="text-align:center;margin-bottom:12px;color:#2c3e50">Priority Comparison</h3>
          <canvas id="priorityChart"></canvas>
        </div>
      </div>
    </div>

    <div class="bottom-charts" style="padding-top:8px">
      <div class="chart-container">
        <h3 style="text-align:center;margin-bottom:12px;color:#2c3e50">Savings Trend (Recent Months)</h3>
        <canvas id="trendChart"></canvas>
      </div>
      <div class="chart-container" style="display:flex;flex-direction:column;justify-content:center;align-items:center">
        <div style="text-align:center">
          <h3 style="color:#2c3e50;margin-bottom:10px">Quick Tips</h3>
          <p class="small-muted" style="max-width:320px;text-align:left">Tip: Mark only confirmed purchases as <strong>Buy Now</strong> and then type <em>Yes</em> in the Sync Flag (in tracker) before pressing Save to subtract cost from your budget. The app auto-saves on edit.</p>
        </div>
      </div>
    </div>

  </div>

  <script>
  // ---------- CONFIG ----------
  const DEFAULT_SALARY = 25000;
  const DEFAULT_TOTAL_INCOME = 21000;
  const ALLOCATIONS = { essentials: 0.60, wants: 0.25, savings: 0.10 };
  const MAX_MONTHS_TO_PLOT = 12;

  // ---------- STATE ----------
  let currentDate = new Date(); // will be adjusted for month navigation
  let budgetData = {
    essentials: [
      {category:'Rent', description:'', amount:0},
      {category:'Groceries', description:'', amount:0},
      {category:'Transport', description:'', amount:0},
      {category:'X', description:'', amount:0},
      {category:'X', description:'', amount:0}
    ],
    wants: [
      {category:'Subscriptions', description:'', amount:0},
      {category:'Entertainment', description:'', amount:0},
      {category:'X', description:'', amount:0},
      {category:'X', description:'', amount:0}
    ],
    savings: [
      {category:'Savings', description:'', amount:0},
      {category:'X', description:'', amount:0}
    ]
  };
  let wantsTrackerData = []; // start empty per your request (no sample items)
  let budgetChart, priorityChart, trendChart;

  // ---------- DOM CACHE ----------
  const monthDisplay = document.getElementById('monthDisplay');
  const monthLabel = document.getElementById('monthLabel');
  const prevBtn = document.getElementById('prevMonthBtn');
  const nextBtn = document.getElementById('nextMonthBtn');
  const saveBtn = document.getElementById('saveBtn');
  const resetBtn = document.getElementById('resetBtn');

  // display constants
  document.getElementById('salary-display').textContent = DEFAULT_SALARY.toLocaleString('en-IN');
  document.getElementById('income-display').textContent = DEFAULT_TOTAL_INCOME.toLocaleString('en-IN');

  // ---------- UTILITIES ----------
  function pad(n){return n<10?('0'+n):(''+n)}
  function keyForMonth(date, type){ // type: 'budget' or 'wants'
    const y = date.getFullYear();
    const m = pad(date.getMonth()+1);
    return `${type}Data_${y}_${m}`;
  }
  function displayMonth(date){
    const opts = { year:'numeric', month:'long' };
    monthDisplay.textContent = date.toLocaleDateString('en-GB', opts);
    monthLabel.textContent = `${date.getFullYear()} - ${pad(date.getMonth()+1)}`;
  }

  // ---------- SAVE / LOAD / RESET ----------
  function saveMonthData(){
    try{
      localStorage.setItem(keyForMonth(currentDate,'budget'), JSON.stringify(budgetData));
      localStorage.setItem(keyForMonth(currentDate,'wants'), JSON.stringify(wantsTrackerData));
      // also save a small summary for trend (remaining)
      const totals = calculateTotals();
      const remaining = DEFAULT_TOTAL_INCOME - totals.totalExpenses;
      localStorage.setItem(`summary_${currentDate.getFullYear()}_${pad(currentDate.getMonth()+1)}`, JSON.stringify({remaining}));
      updateTrendChart(); // refresh trend after save
      console.log('Saved month:', keyForMonth(currentDate,'budget'));
    }catch(e){
      console.error('Save failed', e);
    }
  }

  function loadMonthData(){
    const bKey = keyForMonth(currentDate,'budget');
    const wKey = keyForMonth(currentDate,'wants');
    const b = localStorage.getItem(bKey);
    const w = localStorage.getItem(wKey);
    if(b) budgetData = JSON.parse(b);
    else { /* keep default budgetData */ }
    if(w) wantsTrackerData = JSON.parse(w);
    else wantsTrackerData = []; // start fresh for new months
    renderBudgetTable();
    updateSummary();
    renderWantsTracker();
    updateLinkedWants();
    updatePriorityChart();
    updateTrendChart();
    displayMonth(currentDate);
  }

  function resetMonthData(){
    if(!confirm(`Reset data for ${monthLabel.textContent}? This will clear only this month's saved data.`)) return;
    localStorage.removeItem(keyForMonth(currentDate,'budget'));
    localStorage.removeItem(keyForMonth(currentDate,'wants'));
    localStorage.removeItem(`summary_${currentDate.getFullYear()}_${pad(currentDate.getMonth()+1)}`);
    // restore defaults
    budgetData = {
      essentials:[
        {category:'Rent',description:'',amount:0},
        {category:'Groceries',description:'',amount:0},
        {category:'Transport',description:'',amount:0},
        {category:'X',description:'',amount:0},
        {category:'X',description:'',amount:0}
      ],
      wants:[
        {category:'Subscriptions',description:'',amount:0},
        {category:'Entertainment',description:'',amount:0},
        {category:'X',description:'',amount:0},
        {category:'X',description:'',amount:0}
      ],
      savings:[
        {category:'Savings',description:'',amount:0},
        {category:'X',description:'',amount:0}
      ]
    };
    wantsTrackerData = [];
    saveMonthData();
    loadMonthData();
  }

  // ---------- NAVIGATION ----------
  prevBtn.addEventListener('click', () => {
    saveMonthData();
    currentDate.setMonth(currentDate.getMonth()-1);
    loadMonthData();
  });
  nextBtn.addEventListener('click', () => {
    saveMonthData();
    currentDate.setMonth(currentDate.getMonth()+1);
    loadMonthData();
  });
  saveBtn.addEventListener('click', () => {
    saveMonthData();
    alert('Saved current month.');
  });
  resetBtn.addEventListener('click', resetMonthData);

  // ---------- BUDGET UI ----------
  function formatCurrency(v){ return (parseFloat(v)||0).toLocaleString('en-IN'); }

  function calculateTotals(){
    let totalExpenses=0;
    const sectionTotals={essentials:0,wants:0,savings:0};
    for(const s in budgetData){
      budgetData[s].forEach(it=>{
        const a = parseFloat(it.amount) || 0;
        totalExpenses += a;
        sectionTotals[s] += a;
      });
    }
    return { totalExpenses, sectionTotals };
  }

  function updateSummary(){
    const {totalExpenses, sectionTotals} = calculateTotals();
    const remaining = DEFAULT_TOTAL_INCOME - totalExpenses;
    document.getElementById('expenses-display').textContent = formatCurrency(totalExpenses);
    document.getElementById('balance-display').textContent = formatCurrency(remaining);
    updateBudgetChart(sectionTotals);
    // save summary automatically for trend
    localStorage.setItem(`summary_${currentDate.getFullYear()}_${pad(currentDate.getMonth()+1)}`, JSON.stringify({remaining}));
  }

  function renderBudgetTable(){
    const tbody = document.getElementById('budget-body');
    tbody.innerHTML = '';
    const sections = [
      {name:'essentials', label:`Essentials (${Math.round(ALLOCATIONS.essentials*100)}%)`, allocated:DEFAULT_TOTAL_INCOME*ALLOCATIONS.essentials},
      {name:'wants', label:`Wants (${Math.round(ALLOCATIONS.wants*100)}%)`, allocated:DEFAULT_TOTAL_INCOME*ALLOCATIONS.wants},
      {name:'savings', label:`Savings/Investments (${Math.round(ALLOCATIONS.savings*100)}%)`, allocated:DEFAULT_TOTAL_INCOME*ALLOCATIONS.savings}
    ];
    sections.forEach(section=>{
      const sectionTotal = budgetData[section.name].reduce((s,it)=>s + (parseFloat(it.amount)||0),0);
      const remaining = section.allocated - sectionTotal;
      const headerRow = document.createElement('tr');
      headerRow.className = 'section-header';
      const th = document.createElement('td');
      th.colSpan=5;
      th.textContent = `${section.label} - Allocated: ₹${formatCurrency(section.allocated)} | Spent: ₹${formatCurrency(sectionTotal)} | Remaining: ₹${formatCurrency(remaining)}`;
      headerRow.appendChild(th);
      tbody.appendChild(headerRow);
      budgetData[section.name].forEach((item, idx) => {
        const tr = document.createElement('tr');
        // category
        const tdCat = document.createElement('td');
        const inCat = document.createElement('input');
        inCat.type='text'; inCat.value=item.category;
        inCat.onchange = (e)=>{ budgetData[section.name][idx].category = e.target.value; saveMonthData(); renderBudgetTable(); updateSummary(); };
        tdCat.appendChild(inCat); tr.appendChild(tdCat);
        // desc
        const tdDesc = document.createElement('td');
        const inDesc = document.createElement('input');
        inDesc.type='text'; inDesc.value=item.description;
        inDesc.onchange = (e)=>{ budgetData[section.name][idx].description = e.target.value; saveMonthData(); };
        tdDesc.appendChild(inDesc); tr.appendChild(tdDesc);
        // amount
        const tdAmt = document.createElement('td');
        const inAmt = document.createElement('input');
        inAmt.type='number'; inAmt.value=item.amount;
        inAmt.onchange = (e)=>{ budgetData[section.name][idx].amount = parseFloat(e.target.value)||0; saveMonthData(); renderBudgetTable(); updateSummary(); updateLinkedWants(); };
        tdAmt.appendChild(inAmt); tr.appendChild(tdAmt);
        // section label
        const tdSec = document.createElement('td'); tdSec.textContent = section.label.split(' ')[0]; tr.appendChild(tdSec);
        // remaining
        const tdRem = document.createElement('td'); tdRem.textContent = `₹${formatCurrency(remaining)}`; tr.appendChild(tdRem);
        tbody.appendChild(tr);
      });
    });
  }

  function updateBudgetChart(sectionTotals){
    const ctx = document.getElementById('budgetChart').getContext('2d');
    if(budgetChart) budgetChart.destroy();
    budgetChart = new Chart(ctx, {
      type:'pie',
      data:{ labels:['Essentials','Wants','Savings'], datasets:[{ data:[sectionTotals.essentials, sectionTotals.wants, sectionTotals.savings], backgroundColor:['#667eea','#764ba2','#f093fb'], borderWidth:2, borderColor:'#fff' }]},
      options:{ responsive:true, maintainAspectRatio:true, plugins:{ legend:{position:'bottom'}, tooltip:{callbacks:{ label:function(context){ return context.label + ': ₹' + formatCurrency(context.parsed); }}}}}
    });
  }

  // ---------- WANTS TRACKER UI ----------
  function renderWantsTracker(){
    const tbody = document.getElementById('wants-body');
    tbody.innerHTML = '';
    wantsTrackerData.forEach((item, idx)=>{
      const tr = document.createElement('tr');
      // name
      const tdName = document.createElement('td');
      const inName = document.createElement('input'); inName.type='text'; inName.value=item.name || '';
      inName.onchange=(e)=>{ wantsTrackerData[idx].name = e.target.value; saveMonthData(); updateLinkedWants(); updatePriorityChart(); };
      tdName.appendChild(inName); tr.appendChild(tdName);
      // cost
      const tdCost = document.createElement('td');
      const inCost = document.createElement('input'); inCost.type='number'; inCost.value=item.cost||0;
      inCost.onchange=(e)=>{ wantsTrackerData[idx].cost = parseFloat(e.target.value)||0; saveMonthData(); updateLinkedWants(); };
      tdCost.appendChild(inCost); tr.appendChild(tdCost);
      // five score fields
      ['want','useful','afford','value','improve'].forEach(field=>{
        const td = document.createElement('td');
        const inp = document.createElement('input'); inp.type='number'; inp.min=0; inp.max=10; inp.value = item[field] || 0;
        inp.onchange=(e)=>{ const v = Math.min(10,Math.max(0,parseInt(e.target.value)||0)); wantsTrackerData[idx][field]=v; saveMonthData(); renderWantsTracker(); updateLinkedWants(); updatePriorityChart(); };
        td.appendChild(inp); tr.appendChild(td);
      });
      // totals & priority
      const total = (item.want||0)+(item.useful||0)+(item.afford||0)+(item.value||0)+(item.improve||0);
      const priority = total>0?((total/50)*100):0;
      const decision = priority>=80? 'Buy Now' : priority>=50? 'Consider Later' : 'Avoid';
      const tdTotal = document.createElement('td'); tdTotal.textContent = total; tr.appendChild(tdTotal);
      const tdPri = document.createElement('td'); tdPri.textContent = priority.toFixed(1) + '%'; tr.appendChild(tdPri);
      const tdDec = document.createElement('td'); tdDec.textContent = decision; tr.appendChild(tdDec);
      tbody.appendChild(tr);
    });
    // append a final empty row for quick adding
    const addRow = document.createElement('tr');
    const td = document.createElement('td'); td.colSpan = 10;
    td.innerHTML = `<button class="btn small" id="addWantBtn">+ Add new want</button>`;
    addRow.appendChild(td);
    tbody.appendChild(addRow);
    const addBtn = document.getElementById('addWantBtn');
    if(addBtn) addBtn.onclick = ()=>{ wantsTrackerData.push({name:'',cost:0,want:0,useful:0,afford:0,value:0,improve:0}); saveMonthData(); renderWantsTracker(); updatePriorityChart(); updateLinkedWants(); };
  }

  function updateLinkedWants(){
    const container = document.getElementById('linked-wants-list');
    container.innerHTML = '';
    if(wantsTrackerData.length===0){ container.innerHTML = '<p class="small-muted">No items in tracker yet.</p>'; return; }
    wantsTrackerData.forEach(item=>{
      const total = (item.want||0)+(item.useful||0)+(item.afford||0)+(item.value||0)+(item.improve||0);
      const priority = total>0?((total/50)*100):0;
      const decision = priority>=80? 'Buy Now' : priority>=50? 'Consider Later' : 'Avoid';
      const badgeClass = priority>=80? 'buy-now' : priority>=50? 'consider-later' : 'avoid';
      const div = document.createElement('div'); div.className='linked-item';
      div.innerHTML = `<span><strong>${item.name||'(untitled)'}</strong> - ₹${formatCurrency(item.cost||0)}</span><span class="priority-badge ${badgeClass}">${decision}</span>`;
      container.appendChild(div);
    });
    // Also ensure linked wants appear in budget wants if you want auto-copy (one-way)
    // We won't auto-add amounts into budget; this is only a visible link.
  }

  function updatePriorityChart(){
    const ctx = document.getElementById('priorityChart').getContext('2d');
    if(priorityChart) priorityChart.destroy();
    const labels = wantsTrackerData.map(it => it.name || '(untitled)');
    const priorities = wantsTrackerData.map(it => {
      const total = (it.want||0)+(it.useful||0)+(it.afford||0)+(it.value||0)+(it.improve||0);
      return total>0 ? ((total/50)*100) : 0;
    });
    priorityChart = new Chart(ctx, {
      type:'bar',
      data:{ labels, datasets:[ { label:'Priority %', data:priorities, backgroundColor:'#667eea', borderColor:'#764ba2', borderWidth:1 } ] },
      options:{ responsive:true, maintainAspectRatio:true, scales:{ y:{ beginAtZero:true, max:100, ticks:{ callback:(v)=>v+'%' } } }, plugins:{ legend:{ display:false } } }
    });
  }

  // ---------- TREND CHART ----------
  function collectRecentSummaries(){
    // look back up to MAX_MONTHS_TO_PLOT months and gather summary_{yyyy}_{mm}
    const arr = [];
    const copyDate = new Date(currentDate.getFullYear(), currentDate.getMonth(), 1);
    for(let i=0;i<MAX_MONTHS_TO_PLOT;i++){
      const d = new Date(copyDate.getFullYear(), copyDate.getMonth()-i, 1);
      const key = `summary_${d.getFullYear()}_${pad(d.getMonth()+1)}`;
      const raw = localStorage.getItem(key);
      if(raw){
        try{
          const parsed = JSON.parse(raw);
          arr.push({ label:`${d.getFullYear()}-${pad(d.getMonth()+1)}`, remaining: parsed.remaining||0 });
        }catch(e){ /* ignore */ }
      }
    }
    // reverse to chronological
    return arr.reverse();
  }

  function updateTrendChart(){
    const data = collectRecentSummaries();
    const ctx = document.getElementById('trendChart').getContext('2d');
    if(trendChart) trendChart.destroy();
    trendChart = new Chart(ctx, {
      type:'line',
      data:{ labels: data.map(d=>d.label), datasets:[ { label:'Remaining Balance (₹)', data: data.map(d=>d.remaining), fill:false, tension:0.3, borderColor:'#667eea', backgroundColor:'#667eea' } ] },
      options:{ responsive:true, maintainAspectRatio:true, scales:{ y:{ ticks:{ callback:(v)=>v.toLocaleString('en-IN') } } }, plugins:{ legend:{ display:false } } }
    });
  }

  // ---------- TABS ----------
  function switchTab(n){
    const tabs = document.querySelectorAll('.tab');
    const contents = document.querySelectorAll('.tab-content');
    tabs.forEach((t,i)=>{ t.classList.toggle('active', i===n); contents[i].classList.toggle('active', i===n); });
    if(n===1) updatePriorityChart();
  }

  // ---------- INIT ----------
  function init(){
    displayMonth(currentDate);
    // load any saved month
    loadMonthData();
    // initialize empty trend if none
    updateTrendChart();
    // wire small autosave by interval (safety)
    setInterval(()=>saveMonthData(), 1000*30);
  }

  // ---------- BOOT ----------
  init();

  </script>
</body>
</html>