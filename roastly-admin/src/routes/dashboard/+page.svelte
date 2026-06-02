<script>
  // ── Roastly Dashboard — Svelte 5 ──
  // Pages: Daily Report, Employees, Payroll, Audit Logs
  // All data read-only from JSON. No inline edits.

  const iso = d => d.toISOString().slice(0, 10);
  const HOURS = ["08","09","10","11","12","13","14","15","16","17","18","19","20"];
  const TODAY = iso(new Date());

  // ── Nav state ──
  let activePage = $state('report');
  let sidebarCollapsed = $state(false);

  // ── Report state ──
  let period = $state('today');
  let customDate = $state(TODAY);
  let toastMsg = $state('');
  let toastVisible = $state(false);
  let spinning = $state(false);

  // ── Audit filter state ──
  let auditFilter = $state('all'); // 'all' | 'actions' | 'shifts'

  // ── Static JSON data ──
  const EMPLOYEES = [
    { id: "E001", name: "Marco Reyes",  role: "admin",       status: "active",   shift: "Morning",   joined: "2024-01-15", hourlyRate: 120 },
    { id: "E002", name: "Lia Santos",   role: "barista",     status: "active",   shift: "Morning",   joined: "2024-03-10", hourlyRate: 85  },
    { id: "E003", name: "Jed Flores",   role: "barista",     status: "active",   shift: "Afternoon", joined: "2024-05-20", hourlyRate: 85  },
    { id: "E004", name: "Mia Cruz",     role: "order_taker", status: "active",   shift: "Morning",   joined: "2024-06-01", hourlyRate: 75  },
    { id: "E005", name: "Paolo Tan",    role: "barista",     status: "on_leave", shift: "Afternoon", joined: "2024-07-14", hourlyRate: 85  },
    { id: "E006", name: "Nina Abad",    role: "order_taker", status: "active",   shift: "Afternoon", joined: "2025-01-08", hourlyRate: 75  },
  ];

  const PAYROLL = [
    { id: "E001", name: "Marco Reyes",  role: "admin",       hoursThisWeek: 40, hourlyRate: 120, deductions: 500, bonus: 1000 },
    { id: "E002", name: "Lia Santos",   role: "barista",     hoursThisWeek: 38, hourlyRate: 85,  deductions: 200, bonus: 200  },
    { id: "E003", name: "Jed Flores",   role: "barista",     hoursThisWeek: 36, hourlyRate: 85,  deductions: 200, bonus: 0    },
    { id: "E004", name: "Mia Cruz",     role: "order_taker", hoursThisWeek: 40, hourlyRate: 75,  deductions: 150, bonus: 150  },
    { id: "E005", name: "Paolo Tan",    role: "barista",     hoursThisWeek: 0,  hourlyRate: 85,  deductions: 0,   bonus: 0    },
    { id: "E006", name: "Nina Abad",    role: "order_taker", hoursThisWeek: 32, hourlyRate: 75,  deductions: 150, bonus: 0    },
  ];

  // Audit log: type = 'action' | 'shift'
  const AUDIT_LOGS = [
    { id: "AL-031", date: "2026-05-31", time: "10:58", actor: "Marco Reyes",  type: "action", category: "menu",   action: "Updated price",    detail: "Sea Salt Latte ₱155 → ₱165" },
    { id: "AL-030", date: "2026-05-31", time: "10:45", actor: "Mia Cruz",     type: "shift",  category: "in",     action: "Shift check-in",   detail: "Morning shift · Terminal A" },
    { id: "AL-029", date: "2026-05-31", time: "09:15", actor: "Marco Reyes",  type: "action", category: "staff",  action: "Added employee",   detail: "Nina Abad registered as Order Taker" },
    { id: "AL-028", date: "2026-05-31", time: "08:01", actor: "Lia Santos",   type: "shift",  category: "in",     action: "Shift check-in",   detail: "Morning shift · Terminal B" },
    { id: "AL-027", date: "2026-05-31", time: "08:00", actor: "Mia Cruz",     type: "action", category: "order",  action: "Order voided",     detail: "ORD-1054 cancelled — wrong item" },
    { id: "AL-026", date: "2026-05-30", time: "21:03", actor: "Jed Flores",   type: "shift",  category: "out",    action: "Shift check-out",  detail: "Afternoon shift — 8h logged · ₱2,840 revenue" },
    { id: "AL-025", date: "2026-05-30", time: "19:45", actor: "Nina Abad",    type: "shift",  category: "out",    action: "Shift check-out",  detail: "Afternoon shift — 6.5h logged" },
    { id: "AL-024", date: "2026-05-30", time: "16:30", actor: "Marco Reyes",  type: "action", category: "inventory", action: "Stock updated", detail: "Oat Milk restocked: 2L → 8L" },
    { id: "AL-023", date: "2026-05-30", time: "13:02", actor: "Lia Santos",   type: "shift",  category: "out",    action: "Shift check-out",  detail: "Morning shift — 6h logged · ₱3,120 revenue" },
    { id: "AL-022", date: "2026-05-30", time: "13:01", actor: "Mia Cruz",     type: "shift",  category: "out",    action: "Shift check-out",  detail: "Morning shift — 6h logged" },
    { id: "AL-021", date: "2026-05-30", time: "07:02", actor: "Jed Flores",   type: "shift",  category: "in",     action: "Shift check-in",   detail: "Afternoon shift · Terminal C" },
    { id: "AL-020", date: "2026-05-30", time: "07:00", actor: "Nina Abad",    type: "shift",  category: "in",     action: "Shift check-in",   detail: "Afternoon shift · Terminal D" },
    { id: "AL-019", date: "2026-05-29", time: "20:58", actor: "Jed Flores",   type: "shift",  category: "out",    action: "Shift check-out",  detail: "Afternoon shift — 8h logged · ₱4,255 revenue" },
    { id: "AL-018", date: "2026-05-29", time: "11:30", actor: "Marco Reyes",  type: "action", category: "menu",   action: "Item deactivated", detail: "Houjicha Latte set to inactive" },
    { id: "AL-017", date: "2026-05-29", time: "07:05", actor: "Lia Santos",   type: "shift",  category: "in",     action: "Shift check-in",   detail: "Morning shift · Terminal A" },
  ];

  // ── Report dataset generator ──
  const ITEMS = ["Sea Salt Latte","Cold Brew Original","Oat Mocha","Caramel Macchiato","Americano","Caphe Sua Da"];
  const STAFF_LIST = [
    { name: "Lia Santos", role: "Barista" },
    { name: "Mia Cruz", role: "Order Taker" },
    { name: "Jed Flores", role: "Barista" }
  ];

  function generateDay(offset) {
    const day = new Date(); day.setDate(day.getDate() - offset);
    const dow = day.getDay();
    const base = (dow === 0 || dow === 6) ? 1.5 : 1;
    const seed = (offset * 7 + 3) % 5;
    const orders = Math.round((38 + seed * 6) * base);
    const cancelled = (orders > 0 && offset !== 0) ? (seed % 3) : 0;
    const completed = orders - cancelled - 2;
    const avg = 95 + seed * 8;
    const gross = completed * avg;
    const curve = [0.4,0.9,1.0,0.7,1.1,1.3,0.8,0.6,0.7,0.9,0.6,0.3,0.2];
    const sum = curve.reduce((a, b) => a + b, 0);
    const byHour = {}; let rem = completed;
    HOURS.forEach((h, i) => {
      const v = i === HOURS.length - 1 ? rem : Math.round(completed * curve[i] / sum);
      byHour[h] = Math.max(0, v); rem -= v;
    });
    return {
      branch: "Roastly — Main Branch", generatedBy: "admin",
      revenueToday: gross, ordersTotal: orders, ordersCompleted: completed,
      ordersDraft: 2, ordersCancelled: cancelled,
      staffOnShift: base > 1 ? 3 : 2, employeesTotal: 6, lowStockCount: seed % 2,
      payments: { cash: Math.round(gross * 0.45), card: Math.round(gross * 0.30), gcash: Math.round(gross * 0.25) },
      discounts: Math.round(gross * 0.03), refunds: cancelled * avg, tax: Math.round(gross * 0.12),
      ordersByHour: byHour,
      orderTypes: { dineIn: Math.round(completed * 0.6), takeout: Math.round(completed * 0.4) },
      topItems: ITEMS.slice(0, 5).map((n, i) => ({ name: n, qty: Math.round(completed * (0.3 - i * 0.05)), revenue: Math.round(gross * (0.3 - i * 0.05)) })),
      cancellationReasons: cancelled ? [{ reason: "Out of stock", count: cancelled }] : [],
      staff: STAFF_LIST.slice(0, base > 1 ? 3 : 2).map(s => ({ ...s, hours: 8, sales: Math.round(gross / (base > 1 ? 3 : 2)) })),
      lowStockItems: (seed % 2) ? [{ name: "Oat Milk", qty: 2, threshold: 5 }] : []
    };
  }

  function offsetISO(o) { const d = new Date(); d.setDate(d.getDate() - o); return iso(d); }

  const DATASET = {};
  for (let o = 0; o < 8; o++) DATASET[offsetISO(o)] = generateDay(o);

  function prevDay(k) {
    const d = new Date(k); d.setDate(d.getDate() - 1);
    return DATASET[iso(d)] || null;
  }

  function emptyDay() {
    return {
      branch: "Roastly — Main Branch", generatedBy: "admin",
      revenueToday: 0, ordersTotal: 0, ordersCompleted: 0, ordersDraft: 0, ordersCancelled: 0,
      staffOnShift: 0, employeesTotal: 6, lowStockCount: 0,
      payments: { cash: 0, card: 0, gcash: 0 }, discounts: 0, refunds: 0, tax: 0,
      ordersByHour: Object.fromEntries(HOURS.map(h => [h, 0])),
      orderTypes: { dineIn: 0, takeout: 0 },
      topItems: [], cancellationReasons: [], staff: [], lowStockItems: []
    };
  }

  function aggregate(days) {
    const a = JSON.parse(JSON.stringify(days[0]));
    const numeric = ["revenueToday","ordersTotal","ordersCompleted","ordersDraft","ordersCancelled","discounts","refunds","tax"];
    for (let i = 1; i < days.length; i++) {
      const d = days[i];
      numeric.forEach(k => a[k] += d[k]);
      ["cash","card","gcash"].forEach(k => a.payments[k] += d.payments[k]);
      HOURS.forEach(h => a.ordersByHour[h] += d.ordersByHour[h]);
      a.orderTypes.dineIn += d.orderTypes.dineIn; a.orderTypes.takeout += d.orderTypes.takeout;
    }
    a.staffOnShift = Math.max(...days.map(d => d.staffOnShift));
    const m = {};
    days.forEach(d => d.topItems.forEach(t => {
      m[t.name] = m[t.name] || { name: t.name, qty: 0, revenue: 0 };
      m[t.name].qty += t.qty; m[t.name].revenue += t.revenue;
    }));
    a.topItems = Object.values(m).sort((x, y) => y.qty - x.qty).slice(0, 5);
    return a;
  }

  // ── Resolve raw period data ──
  let raw = $derived.by(() => {
    if (period === 'today') {
      return { d: { ...DATASET[TODAY], _date: TODAY }, prev: prevDay(TODAY), label: 'Today', name: TODAY };
    }
    if (period === 'yesterday') {
      const k = offsetISO(1);
      return { d: { ...DATASET[k], _date: k }, prev: prevDay(k), label: 'Yesterday', name: k };
    }
    if (period === 'week') {
      const days = Array.from({ length: 7 }, (_, o) => DATASET[offsetISO(o)]);
      const d = { ...aggregate(days), _date: `${offsetISO(6)}_to_${TODAY}` };
      return { d, prev: DATASET[offsetISO(7)] || days[0], label: 'This Week', name: `week_${TODAY}` };
    }
    const k = customDate || TODAY;
    return { d: { ...(DATASET[k] || emptyDay()), _date: k }, prev: prevDay(k), label: k, name: k };
  });

  // ── Build report ──
 let report = $derived.by(() => {
  const { d, prev, label } = raw;
  const pct = (c, p) => p === 0 ? (c > 0 ? 100 : 0) : +(((c - p) / p) * 100).toFixed(1);
  const net = d.revenueToday - d.refunds - d.discounts;
  const aov = d.ordersTotal ? +(d.revenueToday / d.ordersTotal).toFixed(2) : 0;
  
  return {
    meta: { schemaVersion: "1.1", reportType: "daily", period: label, reportDate: d._date || TODAY, generatedAt: new Date().toISOString(), timezone: Intl.DateTimeFormat().resolvedOptions().timeZone, branch: d.branch, generatedBy: d.generatedBy },
    revenue: { gross: d.revenueToday, net, currency: "PHP", averageOrderValue: aov, discounts: d.discounts, refunds: d.refunds, tax: d.tax, byPaymentMethod: d.payments, vsPrevious: prev ? { gross: prev.revenueToday, changePercent: pct(d.revenueToday, prev.revenueToday) } : null },
    orders: { total: d.ordersTotal, completed: d.ordersCompleted, draft: d.ordersDraft, cancelled: d.ordersCancelled, cancellationRate: d.ordersTotal ? +((d.ordersCancelled / d.ordersTotal) * 100).toFixed(1) : 0, byType: d.orderTypes, byHour: d.ordersByHour, vsPrevious: prev ? { total: prev.ordersTotal, changePercent: pct(d.ordersTotal, prev.ordersTotal) } : null, cancellationReasons: d.cancellationReasons },
    topItems: d.topItems,
    staff: { onShift: d.staffOnShift, details: d.staff },
    
    // Updated systemManagement to process audit security streams from your raw object
    systemManagement: { 
      employeesTotal: d.employeesTotal, 
      payroll: { staff: d.staff.map(s => ({ name: s.name, role: s.role, hours: s.hours })) }, 
      audit: { 
        criticalEventCount: d.auditAlertsCount || 0, // Fallback to 0 if data isn't fetched yet
        recentLogs: d.recentAuditLogs || []        // Array container for logs tracking
      } 
    }
  };
});

  // ── Payroll derived ──
  let payrollRows = $derived(PAYROLL.map(p => {
    const gross = p.hoursThisWeek * p.hourlyRate;
    const net = gross - p.deductions + p.bonus;
    return { ...p, gross, net };
  }));
  let payrollTotal = $derived(payrollRows.reduce((s, r) => s + r.net, 0));

  // ── Audit filtered ──
  let filteredAudit = $derived(
    auditFilter === 'all' ? AUDIT_LOGS :
    auditFilter === 'actions' ? AUDIT_LOGS.filter(l => l.type === 'action') :
    AUDIT_LOGS.filter(l => l.type === 'shift')
  );

  // ── Delta derivations ──
  let revDelta = $derived(delta(report.revenue.vsPrevious?.changePercent));
  let ordDelta = $derived(delta(report.orders.vsPrevious?.changePercent));

  // ── JSON ──
  let jsonStr = $derived(JSON.stringify(report, null, 2));

  // ── SVG charts ──
  let hoursSvg = $derived.by(() => {
    const data = report.orders.byHour;
    const vals = HOURS.map(h => data[h] || 0);
    const max = Math.max(...vals, 1);
    if (report.orders.total === 0) return '';
    const W = 560, H = 180, pad = 24, bw = (W - pad * 2) / HOURS.length;
    let bars = '', labels = '';
    HOURS.forEach((h, i) => {
      const bh = (vals[i] / max) * (H - 40);
      const x = pad + i * bw + bw * 0.18;
      const y = H - 20 - bh;
      const w = bw * 0.64;
      const peak = vals[i] === max;
      bars += `<rect x="${x.toFixed(1)}" y="${y.toFixed(1)}" width="${w.toFixed(1)}" height="${bh.toFixed(1)}" rx="4" fill="${peak ? '#4940A1' : '#7B74D8'}" opacity="${peak ? 1 : 0.6}"><title>${h}:00 — ${vals[i]} orders</title></rect>`;
      if (i % 2 === 0) labels += `<text x="${(x + w / 2).toFixed(1)}" y="${H - 6}" fill="#9E9CC8" font-size="10" text-anchor="middle" font-family="monospace">${h}</text>`;
    });
    return `<svg viewBox="0 0 ${W} ${H}" style="width:100%;height:auto;display:block">${bars}${labels}</svg>`;
  });

  let donutData = $derived.by(() => {
    const p = report.revenue.byPaymentMethod;
    const total = p.cash + p.card + p.gcash;
    if (total === 0) return null;
    const segs = [['Cash', p.cash, '#4940A1'], ['Card', p.card, '#FFA500'], ['GCash', p.gcash, '#7B74D8']];
    const r0 = 54, cx = 70, cy = 70, C = 2 * Math.PI * r0;
    let off = 0;
    const circles = segs.map(([, v, c]) => {
      const len = (v / total) * C;
      const el = `<circle cx="${cx}" cy="${cy}" r="${r0}" fill="none" stroke="${c}" stroke-width="22" stroke-dasharray="${len.toFixed(2)} ${(C - len).toFixed(2)}" stroke-dashoffset="${(-off).toFixed(2)}" transform="rotate(-90 ${cx} ${cy})"/>`;
      off += len; return el;
    }).join('');
    const svg = `<svg viewBox="0 0 140 140" style="width:120px;height:120px;flex-shrink:0">${circles}<text x="70" y="66" text-anchor="middle" fill="#2E2570" font-size="13" font-weight="700" font-family="serif">${peso(total)}</text><text x="70" y="82" text-anchor="middle" fill="#9E9CC8" font-size="9" font-family="sans-serif">total</text></svg>`;
    return { svg, segs, total };
  });

  // ── Helpers ──
  const peso = n => '₱' + Number(Math.round(n)).toLocaleString('en-PH');
  const syncTime = new Date().toLocaleTimeString('en-PH', { hour: '2-digit', minute: '2-digit' });

  function delta(v) {
    if (v == null) return { cls: 'flat', txt: '—' };
    return { cls: v > 0 ? 'up' : v < 0 ? 'down' : 'flat', txt: (v > 0 ? '▲ ' : v < 0 ? '▼ ' : '– ') + Math.abs(v) + '% vs prev' };
  }

  function hlJson(j) {
    return j.replace(/&/g, '&amp;').replace(/</g, '&lt;')
      .replace(/"([^"]+)":/g, '<span class="jkey">"$1"</span>:')
      .replace(/: "([^"]*)"/g, ': <span class="jstr">"$1"</span>')
      .replace(/: (-?\d+\.?\d*)/g, ': <span class="jnum">$1</span>');
  }

  function showToast(msg) {
    toastMsg = msg; toastVisible = true;
    setTimeout(() => toastVisible = false, 2200);
  }

  function downloadJSON() {
    const blob = new Blob([jsonStr], { type: 'application/json' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a'); a.href = url; a.download = `roastly_report_${raw.name}.json`; a.click();
    URL.revokeObjectURL(url); showToast('✓ JSON downloaded');
  }

  function downloadCSV() {
    const r = report;
    const rows = [['Section','Metric','Value'],
      ['Meta','Report Date', r.meta.reportDate], ['Meta','Period', r.meta.period], ['Meta','Branch', r.meta.branch],
      ['Revenue','Gross', r.revenue.gross], ['Revenue','Net', r.revenue.net], ['Revenue','Avg Order Value', r.revenue.averageOrderValue],
      ['Revenue','Discounts', r.revenue.discounts], ['Revenue','Refunds', r.revenue.refunds], ['Revenue','Tax', r.revenue.tax],
      ['Revenue','Cash', r.revenue.byPaymentMethod.cash], ['Revenue','Card', r.revenue.byPaymentMethod.card], ['Revenue','GCash', r.revenue.byPaymentMethod.gcash],
      ['Orders','Total', r.orders.total], ['Orders','Completed', r.orders.completed], ['Orders','Cancelled', r.orders.cancelled],
      ['Orders','Cancellation Rate %', r.orders.cancellationRate], ['Orders','Dine-in', r.orders.byType.dineIn], ['Orders','Takeout', r.orders.byType.takeout],
      ...HOURS.map(h => ['Orders by Hour', h + ':00', r.orders.byHour[h]]),
      ...r.topItems.map(t => ['Top Items', t.name, `qty ${t.qty} / ₱${t.revenue}`]),
      ...r.staff.details.map(s => ['Staff', `${s.name} (${s.role})`, `${s.hours}h / ₱${s.sales || 0}`])
    ];
    const csv = rows.map(r => r.map(c => `"${String(c).replace(/"/g, '""')}"`).join(',')).join('\n');
    const blob = new Blob([csv], { type: 'text/csv' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a'); a.href = url; a.download = `roastly_report_${raw.name}.csv`; a.click();
    URL.revokeObjectURL(url); showToast('✓ CSV downloaded');
  }

  function copyJSON() { navigator.clipboard.writeText(jsonStr); showToast('✓ JSON copied'); }
  function handleRefresh() { spinning = true; setTimeout(() => { spinning = false; showToast('↻ Refreshed'); }, 600); }
  function setPeriod(p) { period = p; if (p === 'custom' && !customDate) customDate = TODAY; }

  function roleLabel(r) { return { admin: 'Admin', barista: 'Barista', order_taker: 'Order Taker' }[r] || r; }
  function roleCls(r) { return { admin: 'role-admin', barista: 'role-barista', order_taker: 'role-taker' }[r] || ''; }
  function statusCls(s) { return s === 'active' ? 'status-active' : 'status-leave'; }

  function auditCategoryCls(cat) {
    return { in: 'cat-in', out: 'cat-out', menu: 'cat-menu', staff: 'cat-staff', order: 'cat-order', inventory: 'cat-inv' }[cat] || 'cat-order';
  }
  function auditCategoryLabel(cat) {
    return { in: 'Check-in', out: 'Check-out', menu: 'Menu', staff: 'Staff', order: 'Order', inventory: 'Inventory' }[cat] || cat;
  }

  const NAV = [
    { id: 'report',    icon: '📄', label: 'Daily Report' },
    { id: 'employees', icon: '👥', label: 'Employees'    },
    { id: 'payroll',   icon: '💰', label: 'Payroll'      },
    { id: 'audit',     icon: '🗂', label: 'Audit Logs'   },
  ];
</script>

<svelte:head>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin="">
  <link href="https://fonts.googleapis.com/css2?family=Fraunces:opsz,wght@9..144,400;9..144,600;9..144,700&family=Hanken+Grotesk:wght@400;500;600;700&family=JetBrains+Mono:wght@400;500;600&display=swap" rel="stylesheet">
</svelte:head>

<!-- ── Shell ── -->
<div class="shell">

  <!-- ── Sidebar ── -->
<aside class="sidebar" class:collapsed={sidebarCollapsed}>
  <div class="sb-top">
    {#if !sidebarCollapsed}
      <div class="sb-brand">
        <div class="sb-logo">☕</div>
        <div>
          <div class="sb-name">Roastly</div>
          <div class="sb-sub">Management</div>
        </div>
      </div>
    {:else}
      <div class="sb-logo small">☕</div>
    {/if}
    <button class="sb-toggle" onclick={() => sidebarCollapsed = !sidebarCollapsed} aria-label="Toggle sidebar">
      {sidebarCollapsed ? '›' : '‹'}
    </button>
  </div>

  <nav class="sb-nav">
    {#each NAV as n}
      <button
        class="sb-item"
        class:active={activePage === n.id}
        title={n.label}
        onclick={() => activePage = n.id}
      >
        <span class="sb-icon">{n.icon}</span>
        {#if !sidebarCollapsed}<span class="sb-label">{n.label}</span>{/if}
      </button>
    {/each}
  </nav>

  <div class="sb-bottom-stack">
 <button 
  class="sb-item logout-btn" 
  title="Log Out"
  onclick={() => {
    // 1. (Optional) Clear session storage/tokens if needed
    // supabase.auth.signOut(); 
    
    // 2. Clear UI state memory and route back to the index page
    window.location.href = '/'; 
  }}
>
  <span class="sb-icon">🚪</span>
  {#if !sidebarCollapsed}<span class="sb-label">Logout</span>{/if}
</button>

    {#if !sidebarCollapsed}
      <div class="sb-footer">
        <div class="sb-user">
          <div class="sb-avatar">MR</div>
          <div>
            <div class="sb-uname">Marco Reyes</div>
            <div class="sb-urole">Administrator</div>
          </div>
        </div>
      </div>
    {/if}
  </div>
</aside>

  <!-- ── Main content ── -->
  <div class="main">

    <!-- ════ DAILY REPORT ════ -->
    {#if activePage === 'report'}
      <div class="page-wrap">

        <div class="print-head">
          <div class="pt">☕ Roastly — Daily Report</div>
          <div class="pm">{report.meta.branch} · {report.meta.period} · Generated {new Date().toLocaleString('en-PH')}</div>
        </div>

        <header>
          <div class="brand">
            <div class="logo">☕</div>
            <div>
              <h1>Daily Report</h1>
              <p>Welcome back, admin</p>
            </div>
          </div>
          <div class="head-right">
            <div class="synced"><span class="pulse"></span> Last synced <b>{syncTime}</b></div>
            <div class="period-title">{report.meta.period}</div>
          </div>
        </header>

        <div class="controls">
          <div class="seg">
            {#each [['today','Today'],['yesterday','Yesterday'],['week','This Week'],['custom','Custom']] as [k, l]}
              <button class:active={period === k} onclick={() => setPeriod(k)}>{l}</button>
            {/each}
          </div>
          <div class="ctl-date">
            <input type="date" bind:value={customDate} max={TODAY} onchange={() => { period = 'custom'; }} />
            <button class="icon-btn" class:spin={spinning} onclick={handleRefresh} aria-label="Refresh">
              <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"><polyline points="23 4 23 10 17 10"/><polyline points="1 20 1 14 7 14"/><path d="M3.51 9a9 9 0 0 1 14.85-3.36L23 10M1 14l4.64 4.36A9 9 0 0 0 20.49 15"/></svg>
            </button>
          </div>
          <div class="export-group">
            <button class="btn ghost" onclick={downloadCSV}>
              <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/><polyline points="14 2 14 8 20 8"/></svg>
              CSV
            </button>
            <button class="btn ghost" onclick={() => window.print()}>
              <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"><polyline points="6 9 6 2 18 2 18 9"/><path d="M6 18H4a2 2 0 0 1-2-2v-5a2 2 0 0 1 2-2h16a2 2 0 0 1 2 2v5a2 2 0 0 1-2 2h-2"/><rect x="6" y="14" width="12" height="8"/></svg>
              Print / PDF
            </button>
            <button class="btn primary" onclick={downloadJSON}>
              <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.4" stroke-linecap="round" stroke-linejoin="round"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/><polyline points="7 10 12 15 17 10"/><line x1="12" y1="15" x2="12" y2="3"/></svg>
              JSON
            </button>
          </div>
        </div>

        <div class="sec-label">Overview</div>
        <div class="grid overview">
          <div class="card">
            <div class="ico green">₱</div>
            <div class="val">{peso(report.revenue.gross)}</div>
            <div class="lbl">Revenue</div>
            <div class="delta {revDelta.cls}">{revDelta.txt}</div>
          </div>
          <div class="card">
            <div class="ico violet">🧾</div>
            <div class="val">{report.orders.total}</div>
            <div class="lbl">Orders Placed</div>
            <div class="delta {ordDelta.cls}">{ordDelta.txt}</div>
          </div>
          <div class="card">
            <div class="ico amber">👥</div>
            <div class="val">{report.staff.onShift}</div>
            <div class="lbl">Staff on Shift</div>
            <div class="delta flat">active</div>
          </div>
          <div class="card">
            <div class="ico red">⊘</div>
            <div class="val">{report.orders.cancelled}</div>
            <div class="lbl">Cancelled</div>
            <div class="delta flat">{report.orders.cancellationRate}% of total</div>
          </div>
        </div>

        <div class="sec-label">Trends</div>
        <div class="charts">
          <div class="panel">
            <h3>Peak Hours</h3>
            <div class="sub">Orders by hour of day</div>
            {#if report.orders.total === 0}
              <div class="empty">No orders recorded for this period.</div>
            {:else}
              {@html hoursSvg}
            {/if}
          </div>
          <div class="panel">
            <h3>Payment Breakdown</h3>
            <div class="sub">Share of revenue by method</div>
            {#if donutData}
              <div class="donut-wrap">
                {@html donutData.svg}
                <div class="legend">
                  {#each donutData.segs as [name, val, color]}
                    <div class="li">
                      <span class="sw" style="background:{color}"></span>
                      {name}
                      <span class="amt">{peso(val)} · {Math.round(val / donutData.total * 100)}%</span>
                    </div>
                  {/each}
                </div>
              </div>
            {:else}
              <div class="empty">No payments recorded.</div>
            {/if}
          </div>
        </div>

        <div class="sec-label">System Management</div>
        <div class="grid system">
          <div class="row-card" onclick={() => activePage = 'employees'}>
            <div class="ico amber">👥</div>
            <div class="meta"><h3>Employees</h3><p>View roles and staff details</p></div>
            <div class="count">{report.systemManagement.employeesTotal}</div>
          </div>
          <div class="row-card" onclick={() => activePage = 'payroll'}>
            <div class="ico green">💰</div>
            <div class="meta"><h3>Payroll & Work Logs</h3><p>Hours and pay summary</p></div>
            <div class="chev">›</div>
          </div>
        <div class="row-card" onclick={() => activePage = 'audit'}>
    <div class="ico violet">🗂</div>
    <div class="meta">
        <h3>Audit Logs</h3>
        <p>Recent database changes and access history</p>
    </div>
    <div class="count">{report.systemManagement.audit.criticalEventCount}</div> 
</div>
        </div>

        <div class="preview-shell">
          <div class="preview-bar">
            <div class="t">
              <div class="dots">
                <span class="dot" style="background:#ff5f56"></span>
                <span class="dot" style="background:#ffbd2e"></span>
                <span class="dot" style="background:#27c93f"></span>
              </div>
              <span>roastly_report_{raw.name}.json</span>
            </div>
            <button class="copy-btn" onclick={copyJSON}>Copy</button>
          </div>
          <pre>{@html hlJson(jsonStr)}</pre>
        </div>

        <footer>Roastly POS • Daily Report Module • <b>Crafted in Zamboanga City</b></footer>
      </div>

    <!-- ════ EMPLOYEES ════ -->
    {:else if activePage === 'employees'}
      <div class="page-wrap">
        <div class="page-header">
          <div>
            <h2 class="page-title">Employees</h2>
            <p class="page-sub">All registered staff — read-only view</p>
          </div>
        </div>

        <div class="stat-row">
          <div class="stat-chip"><span class="stat-n">{EMPLOYEES.length}</span><span class="stat-l">Total Staff</span></div>
          <div class="stat-chip"><span class="stat-n green">{EMPLOYEES.filter(e => e.status === 'active').length}</span><span class="stat-l">Active</span></div>
          <div class="stat-chip"><span class="stat-n amber">{EMPLOYEES.filter(e => e.status === 'on_leave').length}</span><span class="stat-l">On Leave</span></div>
          <div class="stat-chip"><span class="stat-n purple">{EMPLOYEES.filter(e => e.role === 'barista').length}</span><span class="stat-l">Baristas</span></div>
        </div>

        <div class="emp-table-wrap">
          <table class="data-table">
            <thead>
              <tr>
                <th>ID</th>
                <th>Name</th>
                <th>Role</th>
                <th>Shift</th>
                <th>Status</th>
                <th>Joined</th>
                <th>Rate / hr</th>
              </tr>
            </thead>
            <tbody>
              {#each EMPLOYEES as e}
                <tr>
                  <td class="mono muted">{e.id}</td>
                  <td class="name-cell">
                    <div class="avatar {roleCls(e.role)}">{e.name.split(' ').map(n => n[0]).join('')}</div>
                    {e.name}
                  </td>
                  <td><span class="badge {roleCls(e.role)}">{roleLabel(e.role)}</span></td>
                  <td class="muted">{e.shift}</td>
                  <td><span class="badge {statusCls(e.status)}">{e.status.replace('_', ' ')}</span></td>
                  <td class="mono muted">{e.joined}</td>
                  <td class="mono">{peso(e.hourlyRate)}</td>
                </tr>
              {/each}
            </tbody>
          </table>
        </div>
      </div>

    <!-- ════ PAYROLL ════ -->
    {:else if activePage === 'payroll'}
      <div class="page-wrap">
        <div class="page-header">
          <div>
            <h2 class="page-title">Payroll</h2>
            <p class="page-sub">Weekly pay summary — read-only view</p>
          </div>
          <div class="total-chip">
            <span class="total-label">Total Payable</span>
            <span class="total-val">{peso(payrollTotal)}</span>
          </div>
        </div>

        <div class="stat-row">
          <div class="stat-chip"><span class="stat-n">{PAYROLL.filter(p => p.hoursThisWeek > 0).length}</span><span class="stat-l">Worked This Week</span></div>
          <div class="stat-chip"><span class="stat-n amber">{PAYROLL.reduce((s, p) => s + p.hoursThisWeek, 0)}</span><span class="stat-l">Total Hours</span></div>
          <div class="stat-chip"><span class="stat-n green">{peso(PAYROLL.reduce((s, p) => s + p.bonus, 0))}</span><span class="stat-l">Total Bonuses</span></div>
          <div class="stat-chip"><span class="stat-n red">{peso(PAYROLL.reduce((s, p) => s + p.deductions, 0))}</span><span class="stat-l">Total Deductions</span></div>
        </div>

        <div class="emp-table-wrap">
          <table class="data-table">
            <thead>
              <tr>
                <th>Employee</th>
                <th>Role</th>
                <th>Hours</th>
                <th>Rate/hr</th>
                <th>Gross</th>
                <th>Deductions</th>
                <th>Bonus</th>
                <th>Net Pay</th>
              </tr>
            </thead>
            <tbody>
              {#each payrollRows as p}
                <tr class:inactive-row={p.hoursThisWeek === 0}>
                  <td class="name-cell">
                    <div class="avatar {roleCls(p.role)}">{p.name.split(' ').map(n => n[0]).join('')}</div>
                    {p.name}
                  </td>
                  <td><span class="badge {roleCls(p.role)}">{roleLabel(p.role)}</span></td>
                  <td class="mono {p.hoursThisWeek === 0 ? 'muted' : ''}">{p.hoursThisWeek}h</td>
                  <td class="mono muted">{peso(p.hourlyRate)}</td>
                  <td class="mono">{peso(p.gross)}</td>
                  <td class="mono red">{p.deductions > 0 ? '−' + peso(p.deductions) : '—'}</td>
                  <td class="mono green">{p.bonus > 0 ? '+' + peso(p.bonus) : '—'}</td>
                  <td class="mono bold purple">{peso(p.net)}</td>
                </tr>
              {/each}
            </tbody>
            <tfoot>
              <tr class="tfoot-row">
                <td colspan="4" class="tfoot-label">Weekly Total</td>
                <td class="mono bold">{peso(payrollRows.reduce((s, r) => s + r.gross, 0))}</td>
                <td class="mono bold red">−{peso(payrollRows.reduce((s, r) => s + r.deductions, 0))}</td>
                <td class="mono bold green">+{peso(payrollRows.reduce((s, r) => s + r.bonus, 0))}</td>
                <td class="mono bold purple">{peso(payrollTotal)}</td>
              </tr>
            </tfoot>
          </table>
        </div>
      </div>

    <!-- ════ AUDIT LOGS ════ -->
    {:else if activePage === 'audit'}
      <div class="page-wrap">
        <div class="page-header">
          <div>
            <h2 class="page-title">Audit Logs</h2>
            <p class="page-sub">Action updates and shift check-in/out records</p>
          </div>
        </div>

        <!-- Filter tabs -->
        <div class="filter-tabs">
          {#each [['all', 'All Logs'], ['actions', 'Action Updates'], ['shifts', 'Shift In/Out']] as [k, l]}
            <button class="ftab" class:active={auditFilter === k} onclick={() => auditFilter = k}>{l}</button>
          {/each}
          <div class="tab-count">{filteredAudit.length} entries</div>
        </div>

        <div class="audit-list">
          {#each filteredAudit as log}
            <div class="audit-row {log.type === 'shift' ? 'audit-shift' : 'audit-action'}">
              <div class="audit-time">
                <div class="audit-date">{log.date.slice(5)}</div>
                <div class="audit-clock mono">{log.time}</div>
              </div>
              <div class="audit-line"></div>
              <div class="audit-body">
                <div class="audit-top">
                  <span class="badge {auditCategoryCls(log.category)}">{auditCategoryLabel(log.category)}</span>
                  <span class="audit-action">{log.action}</span>
                  <span class="audit-id mono muted">{log.id}</span>
                </div>
                <div class="audit-detail">{log.detail}</div>
                <div class="audit-actor">by {log.actor}</div>
              </div>
            </div>
          {/each}
        </div>
      </div>
    {/if}

  </div><!-- /main -->
</div><!-- /shell -->

<div class="toast" class:show={toastVisible}>{toastMsg}</div>

<style>
  :global(*) { box-sizing: border-box; margin: 0; padding: 0; }
  :global(html) { -webkit-font-smoothing: antialiased; }
  :global(body) {
    font-family: 'Hanken Grotesk', sans-serif;
    background: #F5FFFA;
    color: #2E2570;
    height: 100vh;
    overflow: hidden;
  }

  /* ── Shell ── */
  .shell { display: flex; height: 100vh; overflow: hidden; }

  /* ── Sidebar ── */
  .sidebar {
    width: 220px; flex-shrink: 0;
    background: #2E2570;
    display: flex; flex-direction: column;
    transition: width 0.22s ease;
    overflow: hidden;
  }
  .sidebar.collapsed { width: 60px; }

  .sb-top {
    display: flex; align-items: center; justify-content: space-between;
    padding: 18px 14px; border-bottom: 1px solid rgba(255,255,255,.07);
    min-height: 64px; gap: 8px;
  }
  .sb-brand { display: flex; align-items: center; gap: 10px; overflow: hidden; flex: 1; }
  .sb-logo {
    width: 36px; height: 36px; border-radius: 9px; flex-shrink: 0;
    background: linear-gradient(145deg, #FFA500, #C97F00);
    display: grid; place-items: center; font-size: 18px;
    box-shadow: 0 4px 12px rgba(255,165,0,.30);
  }
  .sb-logo.small { width: 32px; height: 32px; font-size: 16px; }
  .sb-name { font-weight: 800; font-size: 15px; color: #fff; white-space: nowrap; letter-spacing: -.01em; }
  .sb-sub { font-size: 10px; color: #FFA500; font-weight: 600; white-space: nowrap; }
  .sb-toggle {
    background: rgba(255,255,255,.08); border: none; border-radius: 6px;
    width: 26px; height: 26px; color: rgba(255,255,255,.5);
    cursor: pointer; font-size: 14px; display: grid; place-items: center; flex-shrink: 0;
    transition: background 0.15s, color 0.15s;
  }
  .sb-toggle:hover { background: rgba(255,255,255,.15); color: #fff; }

  .sb-nav { flex: 1; padding: 10px 8px; display: flex; flex-direction: column; gap: 2px; overflow-y: auto; }
  .sb-item {
    display: flex; align-items: center; gap: 10px;
    width: 100%; padding: 9px 10px; border: none; border-radius: 7px;
    background: transparent; color: rgba(255,255,255,.6);
    cursor: pointer; font-size: 13px; font-weight: 500; text-align: left;
    transition: all 0.15s; white-space: nowrap; font-family: 'Hanken Grotesk', sans-serif;
  }
  .sb-item:hover { background: rgba(255,255,255,.08); color: #fff; }
  .sb-item.active { background: #4940A1; color: #fff; font-weight: 700; }
  .sb-icon { font-size: 15px; flex-shrink: 0; }

  .sb-footer { padding: 12px; border-top: 1px solid rgba(255,255,255,.07); }
  .sb-user { display: flex; align-items: center; gap: 10px; }
  .sb-avatar { width: 34px; height: 34px; border-radius: 50%; background: #4940A1; display: grid; place-items: center; font-size: 12px; font-weight: 700; color: #fff; flex-shrink: 0; }
  .sb-uname { font-size: 13px; font-weight: 700; color: #fff; white-space: nowrap; }
  .sb-urole { font-size: 10px; color: #FFA500; white-space: nowrap; }

  /* ── Main ── */
  .main { flex: 1; overflow-y: auto; min-width: 0; background: #F5FFFA; }
  .page-wrap { max-width: 1100px; margin: 0 auto; padding: clamp(20px, 3vw, 44px); }

  /* ── Page header ── */
  .page-header { display: flex; justify-content: space-between; align-items: flex-start; margin-bottom: 24px; flex-wrap: wrap; gap: 16px; }
  .page-title { font-family: 'Fraunces', serif; font-size: 26px; font-weight: 700; color: #4940A1; letter-spacing: -.02em; }
  .page-sub { color: #6B66A8; font-size: 13px; margin-top: 4px; }

  /* ── Stat row ── */
  .stat-row { display: flex; gap: 12px; flex-wrap: wrap; margin-bottom: 24px; }
  .stat-chip { background: #fff; border: 1px solid rgba(73,64,161,.12); border-radius: 12px; padding: 14px 20px; display: flex; flex-direction: column; gap: 4px; min-width: 120px; }
  .stat-n { font-family: 'Fraunces', serif; font-size: 26px; font-weight: 700; color: #2E2570; line-height: 1; }
  .stat-n.green { color: #2E7D4F; }
  .stat-n.amber { color: #C97F00; }
  .stat-n.purple { color: #4940A1; }
  .stat-n.red { color: #B03030; }
  .stat-l { font-size: 11px; font-weight: 600; color: #9E9CC8; text-transform: uppercase; letter-spacing: .04em; }

  /* ── Total chip ── */
  .total-chip { background: #4940A1; border-radius: 12px; padding: 12px 20px; text-align: right; }
  .total-label { display: block; font-size: 11px; font-weight: 600; color: rgba(255,255,255,.6); text-transform: uppercase; letter-spacing: .04em; }
  .total-val { display: block; font-family: 'Fraunces', serif; font-size: 24px; font-weight: 700; color: #fff; margin-top: 2px; }

  /* ── Table ── */
  .emp-table-wrap { overflow-x: auto; border-radius: 14px; border: 1px solid rgba(73,64,161,.12); background: #fff; }
  .data-table { width: 100%; border-collapse: collapse; font-size: 13.5px; }
  .data-table th { padding: 12px 16px; background: #F5F3FF; font-weight: 700; font-size: 11px; text-transform: uppercase; letter-spacing: .05em; color: #6B66A8; border-bottom: 1px solid rgba(73,64,161,.10); text-align: left; white-space: nowrap; }
  .data-table td { padding: 13px 16px; border-bottom: 1px solid rgba(73,64,161,.07); vertical-align: middle; color: #2E2570; }
  .data-table tbody tr:last-child td { border-bottom: none; }
  .data-table tbody tr:hover td { background: #FAFAFF; }
  .inactive-row td { opacity: 0.45; }
  tfoot .tfoot-row td { background: #F5F3FF; border-top: 2px solid rgba(73,64,161,.15); padding: 13px 16px; font-size: 13px; }
  .tfoot-label { font-weight: 700; color: #4940A1; text-transform: uppercase; font-size: 11px; letter-spacing: .05em; }

  /* ── Name cell with avatar ── */
  .name-cell { display: flex; align-items: center; gap: 10px; font-weight: 600; }
  .avatar { width: 32px; height: 32px; border-radius: 50%; display: grid; place-items: center; font-size: 11px; font-weight: 800; flex-shrink: 0; color: #fff; }
  .role-admin { background: #6B2FA0; }
  .role-barista { background: #8B5E00; }
  .role-taker { background: #1E5A7A; }

  /* ── Badges ── */
  .badge { font-size: 10px; font-weight: 700; padding: 3px 8px; border-radius: 5px; text-transform: uppercase; letter-spacing: .04em; white-space: nowrap; }
  .role-admin.badge { background: #EDE0F5; color: #6B2FA0; }
  .role-barista.badge { background: #FFF0D9; color: #8B5E00; }
  .role-taker.badge { background: #E8EFF5; color: #1E5A7A; }
  .status-active { background: #E8F5EE; color: #2E7D4F; }
  .status-leave { background: #FDE8E8; color: #B03030; }

  /* ── Audit category badges ── */
  .cat-in   { background: #E8F5EE; color: #2E7D4F; }
  .cat-out  { background: #EDE0F5; color: #6B2FA0; }
  .cat-menu { background: #FFF0D9; color: #8B5E00; }
  .cat-staff{ background: #E8EFF5; color: #1E5A7A; }
  .cat-order{ background: #FDE8E8; color: #B03030; }
  .cat-inv  { background: #F0EFF5; color: #4940A1; }

  /* ── Filter tabs ── */
  .filter-tabs { display: flex; align-items: center; gap: 6px; margin-bottom: 20px; flex-wrap: wrap; }
  .ftab { font-family: 'Hanken Grotesk', sans-serif; font-size: 13px; font-weight: 600; padding: 7px 16px; border-radius: 8px; border: 1px solid rgba(73,64,161,.15); background: #fff; color: #6B66A8; cursor: pointer; transition: all .15s; }
  .ftab:hover { border-color: #4940A1; color: #4940A1; }
  .ftab.active { background: #4940A1; color: #fff; border-color: #4940A1; }
  .tab-count { margin-left: auto; font-size: 12px; color: #9E9CC8; font-weight: 600; }

  /* ── Audit timeline ── */
  .audit-list { display: flex; flex-direction: column; gap: 0; }
  .audit-row {
    display: grid; grid-template-columns: 64px 28px 1fr;
    gap: 0; align-items: stretch; min-height: 72px;
  }
  .audit-time { padding: 14px 8px 14px 0; text-align: right; display: flex; flex-direction: column; justify-content: center; gap: 3px; }
  .audit-date { font-size: 11px; color: #9E9CC8; font-weight: 600; }
  .audit-clock { font-size: 12px; color: #6B66A8; }
  .audit-line { display: flex; flex-direction: column; align-items: center; }
  .audit-line::before { content: ''; width: 10px; height: 10px; border-radius: 50%; margin-top: 18px; flex-shrink: 0; }
  .audit-line::after { content: ''; width: 2px; flex: 1; margin-bottom: 0; }
  .audit-shift .audit-line::before { background: #4940A1; }
  .audit-shift .audit-line::after { background: rgba(73,64,161,.15); }
  .audit-action .audit-line::before { background: #FFA500; }
  .audit-action .audit-line::after { background: rgba(255,165,0,.15); }
  .audit-list .audit-row:last-child .audit-line::after { display: none; }
  .audit-body { padding: 12px 0 12px 16px; display: flex; flex-direction: column; gap: 4px; justify-content: center; }
  .audit-top { display: flex; align-items: center; gap: 8px; flex-wrap: wrap; }
  .audit-action { font-weight: 700; font-size: 13.5px; color: #2E2570; }
  .audit-id { font-size: 11px; }
  .audit-detail { font-size: 13px; color: #4940A1; font-weight: 500; }
  .audit-actor { font-size: 11px; color: #9E9CC8; }

  /* ── Utilities ── */
  .mono { font-family: 'JetBrains Mono', monospace; }
  .muted { color: #9E9CC8; }
  .bold { font-weight: 700; }
  .green { color: #2E7D4F; }
  .amber { color: #C97F00; }
  .red { color: #B03030; }
  .purple { color: #4940A1; }

  /* ──────────────────────────────────────────
     Below: Daily Report page styles (scoped)
  ────────────────────────────────────────── */

  header { display: flex; align-items: flex-start; justify-content: space-between; gap: 24px; flex-wrap: wrap; margin-bottom: 22px; }
  .brand { display: flex; align-items: center; gap: 14px; }
  .logo { width: 46px; height: 46px; border-radius: 13px; flex-shrink: 0; background: linear-gradient(145deg, #FFA500, #C97F00); display: grid; place-items: center; font-size: 24px; box-shadow: 0 8px 24px rgba(255,165,0,.30); }
  .brand h1 { font-family: 'Fraunces', serif; font-weight: 700; font-size: 28px; letter-spacing: -.02em; line-height: 1; color: #4940A1; }
  .brand p { color: #6B66A8; font-size: 13px; margin-top: 5px; }
  .head-right { display: flex; flex-direction: column; align-items: flex-end; gap: 10px; }
  .synced { font-size: 11.5px; color: #9E9CC8; display: flex; align-items: center; gap: 6px; }
  .synced b { color: #6B66A8; margin-left: 2px; }
  .pulse { width: 7px; height: 7px; border-radius: 50%; background: #2E7D4F; animation: pulse 2s infinite; flex-shrink: 0; }
  @keyframes pulse { 0% { box-shadow: 0 0 0 0 rgba(46,125,79,.5); } 70% { box-shadow: 0 0 0 7px rgba(46,125,79,0); } 100% { box-shadow: 0 0 0 0 rgba(46,125,79,0); } }
  .period-title { font-family: 'Fraunces', serif; font-size: 15px; color: #4940A1; font-weight: 600; }

  .controls { display: flex; align-items: center; gap: 10px; flex-wrap: wrap; margin-bottom: 26px; padding: 12px; background: #fff; border: 1px solid rgba(73,64,161,.10); border-radius: 14px; box-shadow: 0 2px 12px rgba(73,64,161,.06); }
  .seg { display: flex; background: #F5F3FF; border: 1px solid rgba(73,64,161,.10); border-radius: 10px; padding: 3px; gap: 2px; }
  .seg button { font-family: 'Hanken Grotesk', sans-serif; font-weight: 600; font-size: 13px; color: #6B66A8; background: transparent; border: none; padding: 8px 14px; border-radius: 8px; cursor: pointer; transition: .15s; white-space: nowrap; }
  .seg button.active { background: #4940A1; color: #fff; box-shadow: 0 2px 8px rgba(73,64,161,.25); }
  .seg button:hover:not(.active) { color: #4940A1; }
  .ctl-date { display: flex; align-items: center; gap: 8px; margin-left: auto; }
  .ctl-date input[type=date] { font-family: 'Hanken Grotesk', sans-serif; font-size: 13px; background: #F5F3FF; color: #2E2570; border: 1px solid rgba(73,64,161,.15); border-radius: 10px; padding: 8px 12px; outline: none; }
  .icon-btn { width: 38px; height: 38px; display: grid; place-items: center; background: #F5F3FF; border: 1px solid rgba(73,64,161,.15); border-radius: 10px; cursor: pointer; color: #6B66A8; transition: .15s; }
  .icon-btn:hover { color: #4940A1; border-color: #4940A1; }
  .icon-btn svg { width: 17px; height: 17px; }
  .icon-btn.spin svg { animation: spin .6s ease; }
  @keyframes spin { from { transform: rotate(0); } to { transform: rotate(360deg); } }
  .export-group { display: flex; gap: 8px; }
  .btn { display: inline-flex; align-items: center; gap: 8px; cursor: pointer; border: none; font-family: 'Hanken Grotesk', sans-serif; font-weight: 700; font-size: 13px; padding: 10px 16px; border-radius: 11px; transition: transform .15s, box-shadow .15s; }
  .btn svg { width: 15px; height: 15px; }
  .btn.primary { color: #fff; background: #4940A1; box-shadow: 0 6px 18px rgba(73,64,161,.30); }
  .btn.primary:hover { transform: translateY(-2px); box-shadow: 0 10px 24px rgba(73,64,161,.40); background: #2E2570; }
  .btn.ghost { color: #2E2570; background: #fff; border: 1px solid rgba(73,64,161,.20); }
  .btn.ghost:hover { border-color: #4940A1; color: #4940A1; }

  .sec-label { display: flex; align-items: center; gap: 8px; color: #9E9CC8; font-size: 11.5px; font-weight: 700; letter-spacing: .14em; text-transform: uppercase; margin: 28px 2px 16px; }
  .sec-label::after { content: ""; flex: 1; height: 1px; background: rgba(73,64,161,.10); }

  .grid { display: grid; gap: 14px; }
  .grid.overview { grid-template-columns: repeat(4, 1fr); }
  @media (max-width: 820px) { .grid.overview { grid-template-columns: repeat(2, 1fr); } }
  @media (max-width: 460px) { .grid.overview { grid-template-columns: 1fr; } }

  .card { background: #fff; border: 1px solid rgba(73,64,161,.10); border-radius: 16px; padding: 20px; position: relative; overflow: hidden; transition: border-color .2s, transform .2s, box-shadow .2s; box-shadow: 0 2px 10px rgba(73,64,161,.04); }
  .card:hover { border-color: #7B74D8; transform: translateY(-3px); box-shadow: 0 8px 20px rgba(73,64,161,.10); }
  .card::before { content: ""; position: absolute; left: 0; top: 0; bottom: 0; width: 3px; background: #4940A1; opacity: 0; transition: opacity .2s; border-radius: 16px 0 0 16px; }
  .card:hover::before { opacity: 1; }
  .ico { width: 40px; height: 40px; border-radius: 11px; display: grid; place-items: center; font-size: 19px; margin-bottom: 16px; }
  .ico.amber { background: rgba(255,165,0,.14); color: #E8940A; }
  .ico.violet { background: #EAE6FF; color: #4940A1; }
  .ico.red { background: rgba(176,48,48,.12); color: #B03030; }
  .ico.green { background: rgba(46,125,79,.12); color: #2E7D4F; }
  .val { font-family: 'Fraunces', serif; font-weight: 600; font-size: 32px; letter-spacing: -.02em; line-height: 1; color: #2E2570; }
  .lbl { color: #6B66A8; font-size: 13.5px; margin-top: 8px; font-weight: 500; }
  .delta { font-size: 11.5px; margin-top: 10px; font-weight: 600; display: inline-flex; align-items: center; gap: 4px; }
  .delta.up { color: #2E7D4F; } .delta.down { color: #B03030; } .delta.flat { color: #9E9CC8; }

  .charts { display: grid; grid-template-columns: 1.6fr 1fr; gap: 14px; }
  @media (max-width: 820px) { .charts { grid-template-columns: 1fr; } }
  .panel { background: #fff; border: 1px solid rgba(73,64,161,.10); border-radius: 16px; padding: 22px; box-shadow: 0 2px 10px rgba(73,64,161,.04); }
  .panel h3 { font-size: 14px; font-weight: 700; color: #4940A1; margin-bottom: 2px; }
  .sub { color: #9E9CC8; font-size: 12px; margin-bottom: 18px; }
  .empty { color: #9E9CC8; font-size: 13px; text-align: center; padding: 36px 0; }
  .donut-wrap { display: flex; gap: 20px; align-items: center; flex-wrap: wrap; }
  .legend { display: flex; flex-direction: column; gap: 10px; flex: 1; min-width: 140px; }
  .li { display: flex; align-items: center; gap: 10px; font-size: 13px; color: #2E2570; }
  .sw { width: 12px; height: 12px; border-radius: 4px; flex-shrink: 0; }
  .amt { margin-left: auto; font-family: 'JetBrains Mono', monospace; font-size: 12.5px; color: #6B66A8; }

  .grid.system { grid-template-columns: repeat(3, 1fr); }
  @media (max-width: 820px) { .grid.system { grid-template-columns: 1fr; } }
  .row-card { background: #fff; border: 1px solid rgba(73,64,161,.10); border-radius: 16px; padding: 20px; display: flex; align-items: center; gap: 16px; cursor: pointer; transition: border-color .2s, transform .2s, box-shadow .2s; box-shadow: 0 2px 10px rgba(73,64,161,.04); }
  .row-card:hover { border-color: #4940A1; transform: translateY(-3px); box-shadow: 0 8px 20px rgba(73,64,161,.10); }
  .row-card .ico { margin-bottom: 0; flex-shrink: 0; }
  .meta { flex: 1; } .meta h3 { font-size: 15px; font-weight: 700; color: #2E2570; } .meta p { color: #6B66A8; font-size: 12.5px; margin-top: 3px; }
  .count { font-family: 'Fraunces', serif; font-size: 22px; font-weight: 700; color: #4940A1; }
  .chev { color: #9E9CC8; font-size: 20px; }

  .preview-shell { margin-top: 30px; border: 1px solid rgba(73,64,161,.10); border-radius: 16px; overflow: hidden; background: #F5F3FF; }
  .preview-bar { display: flex; align-items: center; justify-content: space-between; padding: 13px 18px; border-bottom: 1px solid rgba(73,64,161,.10); background: #fff; }
  .t { display: flex; align-items: center; gap: 9px; font-size: 13px; font-weight: 600; color: #6B66A8; }
  .dots { display: flex; gap: 6px; } .dot { width: 10px; height: 10px; border-radius: 50%; display: block; }
  .copy-btn { background: #EAE6FF; border: 1px solid rgba(73,64,161,.20); color: #4940A1; font-family: 'Hanken Grotesk', sans-serif; font-size: 12px; font-weight: 700; padding: 6px 12px; border-radius: 8px; cursor: pointer; transition: .15s; }
  .copy-btn:hover { background: #4940A1; color: #fff; border-color: #4940A1; }
  pre { font-family: 'JetBrains Mono', monospace; font-size: 12.5px; line-height: 1.7; padding: 20px; overflow: auto; max-height: 420px; color: #2E2570; }
  pre::-webkit-scrollbar { width: 9px; height: 9px; } pre::-webkit-scrollbar-thumb { background: #EAE6FF; border-radius: 9px; }
  :global(.jkey) { color: #4940A1; font-weight: 600; }
  :global(.jstr) { color: #E8940A; }
  :global(.jnum) { color: #2E7D4F; }

  footer { margin-top: 28px; text-align: center; color: #9E9CC8; font-size: 12px; }
  footer b { color: #6B66A8; }

  .toast { position: fixed; bottom: 26px; left: 50%; transform: translateX(-50%) translateY(20px); background: #4940A1; color: #fff; font-weight: 700; font-size: 13.5px; padding: 12px 22px; border-radius: 12px; box-shadow: 0 12px 30px rgba(73,64,161,.35); opacity: 0; transition: .3s; pointer-events: none; z-index: 100; }
  .toast.show { opacity: 1; transform: translateX(-50%) translateY(0); }

  .print-head { display: none; }
  @media print {
    .sidebar { display: none !important; }
    .main { overflow: visible; }
    .controls, .export-group, .preview-shell, .icon-btn, .synced, .toast, .chev { display: none !important; }
    .print-head { display: flex; justify-content: space-between; align-items: baseline; border-bottom: 2px solid #4940A1; padding-bottom: 8px; margin-bottom: 14px; }
    .pt { font-family: 'Fraunces', serif; font-size: 18px; font-weight: 700; color: #4940A1; }
    .pm { font-size: 11px; color: #444; }
    .card, .panel, .row-card { background: #fff !important; border: 1px solid #ddd !important; box-shadow: none !important; transform: none !important; }
    .card::before { display: none; }
  }
</style>