<script>
	// Mock analytics payload — easily swapped for an active local/offline stream
	let activeTerminalSales = $state(43250); // PHP or local unit value
	let totalOrders = $state(124);
	let activeBaristas = $state(3);
	let syncStatus = $state('synchronized'); // 'synchronized' | 'syncing' | 'offline'

	// Svelte 5 Derived Rune for real-time ticket averages
	let averageOrderValue = $derived(totalOrders > 0 ? (activeTerminalSales / totalOrders).toFixed(2) : 0);

	// Mock transactional feed for the active shift
	let recentOrders = $state([
		{ id: "RST-9021", items: "2x Oat Latte, 1x Croissant", total: 540, time: "3 mins ago", status: "Completed" },
		{ id: "RST-9020", items: "1x Americano", total: 130, time: "8 mins ago", status: "Preparing" },
		{ id: "RST-9019", items: "1x Matcha Espresso, 1x Ube Tart", total: 380, time: "14 mins ago", status: "Completed" },
		{ id: "RST-9018", items: "3x Cappuccino", total: 450, time: "22 mins ago", status: "Completed" }
	]);

	function triggerManualSync() {
		syncStatus = 'syncing';
		setTimeout(() => {
			syncStatus = 'synchronized';
		}, 1500);
	}
</script>

<div class="dashboard-layout">
	<!-- Sidebar Navigation -->
	<aside class="sidebar">
		<div class="brand">
			<span class="logo">☕</span>
			<h2>Roastly <span class="badge">Admin</span></h2>
		</div>
		
		<nav class="nav-links">
			<a href="#overview" class="nav-item active">
				<span class="icon">📊</span> Overview
			</a>
			<a href="#orders" class="nav-item">
				<span class="icon">📋</span> Live Orders
			</a>
			<a href="#inventory" class="nav-item">
				<span class="icon">📦</span> Stock Inventory
			</a>
			<a href="#terminals" class="nav-item">
				<span class="icon">📱</span> Mesh Terminals
			</a>
			<a href="#settings" class="nav-item">
				<span class="icon">⚙️</span> Settings
			</a>
		</nav>

		<div class="sidebar-footer">
			<p class="user-label">Logged in as</p>
			<p class="user-email">architect@roastly.com</p>
		</div>
	</aside>

	<!-- Main Stage Content Area -->
	<main class="main-content">
		<!-- Header Operations Row -->
		<header class="content-header">
			<div class="welcome">
				<h1>Terminal Overview</h1>
				<p class="date-indicator">Active Shift Ledger</p>
			</div>

			<!-- Local Dynamic Sync Status Controller -->
			<div class="sync-controller">
				<span class="status-indicator {syncStatus}">
					<span class="dot"></span> 
					{syncStatus === 'synchronized' ? 'Local Mesh Synced' : syncStatus === 'syncing' ? 'Syncing Nodes...' : 'Offline Mode'}
				</span>
				<button onclick={triggerManualSync} class="sync-btn" disabled={syncStatus === 'syncing'}>
					🔄 Sync
				</button>
			</div>
		</header>

		<!-- Core Metrics Multi-Grid Grid -->
		<section class="metrics-grid">
			<div class="metric-card">
				<div class="card-meta">
					<span class="title">Shift Gross Revenue</span>
					<span class="trend positive">▲ 14%</span>
				</div>
				<p class="value">₱{activeTerminalSales.toLocaleString()}</p>
			</div>

			<div class="metric-card">
				<div class="card-meta">
					<span class="title">Completed Tickets</span>
				</div>
				<p class="value">{totalOrders}</p>
			</div>

			<div class="metric-card">
				<div class="card-meta">
					<span class="title">Average Basket Value</span>
				</div>
				<p class="value">₱{averageOrderValue}</p>
			</div>

			<div class="metric-card">
				<div class="card-meta">
					<span class="title">Active Node Terminals</span>
				</div>
				<p class="value">{activeBaristas} <span class="sub-value">/ 4 Online</span></p>
			</div>
		</section>

		<!-- Data Section Layout -->
		<div class="data-split">
			<!-- Recent Orders Data Matrix -->
			<section class="data-card orders-table-wrapper">
				<h3>Live Node Orders</h3>
				<table class="orders-table">
					<thead>
						<tr>
							<th>Order ID</th>
							<th>Items Lineup</th>
							<th>Total Amount</th>
							<th>Timestamp</th>
							<th>Status</th>
						</tr>
					</thead>
					<tbody>
						{#each recentOrders as order}
							<tr>
								<td class="order-id">{order.id}</td>
								<td class="order-items">{order.items}</td>
								<td class="order-total">₱{order.total}</td>
								<td class="order-time">{order.time}</td>
								<td>
									<span class="status-pill {order.status.toLowerCase()}">
										{order.status}
									</span>
								</td>
							</tr>
						{/each}
					</tbody>
				</table>
			</section>

			<!-- Quick Inventory Watchlist Panel -->
			<section class="data-card status-panel">
				<h3>Critical Stock Warnings</h3>
				<div class="alert-list">
					<div class="alert-item critical">
						<div class="alert-info">
							<h4>Espresso Blend (Premium Arabica)</h4>
							<p>2.5 kg remaining on Barista Station 1</p>
						</div>
						<span class="alert-tag">Low Stock</span>
					</div>
					<div class="alert-item warning">
						<div class="alert-info">
							<h4>Oat Milk (Barista Edition)</h4>
							<p>12 Liters left in main storage cold unit</p>
						</div>
						<span class="alert-tag">Restock Soon</span>
					</div>
				</div>
			</section>
		</div>
	</main>
</div>

<style>
	/* Scoped Style Blueprint matching your locked palette */
	:global(body) {
		margin: 0;
		font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
		background-color: #F5FFFA; /* Mint Cream baseline background */
		color: #2E2570; /* Deep Purple core text */
	}

	.dashboard-layout {
		display: grid;
		grid-template-columns: 260px 1fr;
		min-height: 100vh;
	}

	/* Sidebar Elements */
	.sidebar {
		background-color: #2E2570; /* Deep Purple Base */
		color: #FFFFFF;
		padding: 2rem 1.5rem;
		display: flex;
		flex-direction: column;
		justify-content: space-between;
		border-right: 1px solid rgba(73, 64, 161, 0.2);
	}

	.brand {
		display: flex;
		align-items: center;
		gap: 0.75rem;
		margin-bottom: 2.5rem;
	}

	.brand h2 {
		font-size: 1.3rem;
		margin: 0;
		color: #FFFFFF;
		font-weight: 700;
	}

	.badge {
		background-color: #FFA500; /* Golden Sun Accent */
		color: #2E2570;
		font-size: 0.7rem;
		padding: 0.2rem 0.5rem;
		border-radius: 4px;
		font-weight: bold;
		vertical-align: middle;
	}

	.nav-links {
		display: flex;
		flex-direction: column;
		gap: 0.5rem;
		flex-grow: 1;
	}

	.nav-item {
		color: rgba(255, 255, 255, 0.7);
		text-decoration: none;
		padding: 0.75rem 1rem;
		border-radius: 8px;
		display: flex;
		align-items: center;
		gap: 0.75rem;
		font-weight: 500;
		transition: all 0.2s ease;
	}

	.nav-item:hover, .nav-item.active {
		background-color: #4940A1; /* Royal Purple */
		color: #FFFFFF;
	}

	.sidebar-footer {
		border-top: 1px solid rgba(255, 255, 255, 0.1);
		padding-top: 1rem;
	}

	.user-label {
		font-size: 0.75rem;
		color: rgba(255, 255, 255, 0.5);
		margin: 0;
	}

	.user-email {
		font-size: 0.85rem;
		margin: 0.2rem 0 0 0;
		font-weight: 500;
	}

	/* Main Canvas Content styling */
	.main-content {
		padding: 2.5rem;
		overflow-y: auto;
	}

	.content-header {
		display: flex;
		justify-content: space-between;
		align-items: center;
		margin-bottom: 2.5rem;
	}

	.welcome h1 {
		font-size: 1.75rem;
		color: #4940A1; /* Royal Purple Headlines */
		margin: 0;
		font-weight: 800;
	}

	.date-indicator {
		margin: 0.25rem 0 0 0;
		color: #2E2570;
		opacity: 0.7;
		font-size: 0.9rem;
	}

	/* Sync State Visual Architecture */
	.sync-controller {
		display: flex;
		align-items: center;
		gap: 1rem;
		background: #FFFFFF;
		padding: 0.5rem 1rem;
		border-radius: 30px;
		box-shadow: 0 4px 12px rgba(73, 64, 161, 0.04);
	}

	.status-indicator {
		font-size: 0.85rem;
		font-weight: 600;
		display: flex;
		align-items: center;
		gap: 0.5rem;
	}

	.status-indicator.synchronized { color: #2E8B57; }
	.status-indicator.syncing { color: #FFA500; }

	.dot {
		width: 8px;
		height: 8px;
		border-radius: 50%;
		background-color: currentColor;
	}

	.sync-btn {
		background: none;
		border: 1px solid #D6D1FA;
		padding: 0.25rem 0.75rem;
		border-radius: 20px;
		font-size: 0.8rem;
		font-weight: 600;
		color: #4940A1;
		cursor: pointer;
		transition: all 0.2s;
	}

	.sync-btn:hover {
		background-color: #EAE6FF;
	}

	/* Metrics Cards Framework */
	.metrics-grid {
		display: grid;
		grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
		gap: 1.5rem;
		margin-bottom: 2.5rem;
	}

	.metric-card {
		background-color: #FFFFFF;
		padding: 1.5rem;
		border-radius: 12px;
		box-shadow: 0 4px 15px rgba(73, 64, 161, 0.03);
		border: 1px solid #EAE6FF; /* Light Purple border hint */
	}

	.card-meta {
		display: flex;
		justify-content: space-between;
		align-items: center;
		color: #2E2570;
		opacity: 0.8;
		font-size: 0.85rem;
		font-weight: 600;
	}

	.metric-card .value {
		font-size: 1.8rem;
		font-weight: 800;
		color: #4940A1; /* Royal Purple values */
		margin: 0.75rem 0 0 0;
	}

	.sub-value {
		font-size: 1rem;
		font-weight: 500;
		color: #2E2570;
		opacity: 0.6;
	}

	.trend.positive {
		color: #2E8B57;
		font-weight: bold;
	}

	/* Layout Split section layout */
	.data-split {
		display: grid;
		grid-template-columns: 2fr 1fr;
		gap: 1.5rem;
		align-items: start;
	}

	.data-card {
		background-color: #FFFFFF;
		border-radius: 12px;
		padding: 1.5rem;
		box-shadow: 0 4px 15px rgba(73, 64, 161, 0.03);
		border: 1px solid #EAE6FF;
	}

	.data-card h3 {
		margin-top: 0;
		margin-bottom: 1.25rem;
		font-size: 1.1rem;
		color: #4940A1;
		font-weight: 700;
	}

	/* Modern Operational Data Table layout */
	.orders-table {
		width: 100%;
		border-collapse: collapse;
		text-align: left;
		font-size: 0.9rem;
	}

	.orders-table th {
		padding: 0.75rem 1rem;
		background-color: #F9F8FF; /* Light Purple tint */
		color: #2E2570;
		font-weight: 600;
		border-bottom: 1px solid #EAE6FF;
	}

	.orders-table td {
		padding: 1rem;
		border-bottom: 1px solid #F0EEFF;
		color: #2E2570;
	}

	.order-id { font-weight: 700; color: #4940A1; }
	.order-items { color: #2E2570; opacity: 0.9; }
	.order-total { font-weight: 600; }

	.status-pill {
		padding: 0.25rem 0.5rem;
		border-radius: 6px;
		font-size: 0.75rem;
		font-weight: 600;
	}

	.status-pill.completed { background-color: #E6F7ED; color: #2E8B57; }
	.status-pill.preparing { background-color: #FFF5E6; color: #FFA500; }

	/* Warnings Checklist styling */
	.alert-list {
		display: flex;
		flex-direction: column;
		gap: 1rem;
	}

	.alert-item {
		padding: 1rem;
		border-radius: 8px;
		display: flex;
		justify-content: space-between;
		align-items: center;
		border-left: 4px solid transparent;
	}

	.alert-item.critical {
		background-color: #FFF0F0;
		border-left-color: #D93838;
	}

	.alert-item.warning {
		background-color: #FFF5E6;
		border-left-color: #FFA500;
	}

	.alert-info h4 {
		margin: 0;
		font-size: 0.85rem;
		color: #2E2570;
		font-weight: 700;
	}

	.alert-info p {
		margin: 0.2rem 0 0 0;
		font-size: 0.75rem;
		opacity: 0.8;
	}

	.alert-tag {
		font-size: 0.7rem;
		font-weight: bold;
		text-transform: uppercase;
		letter-spacing: 0.05em;
	}

	.critical .alert-tag { color: #D93838; }
	.warning .alert-tag { color: #FFA500; }
</style>