<script>
	// Svelte 5 Runes for local form state
	let email = $state('');
	let password = $state('');
	let errorMessage = $state('');
	let isLoading = $state(false);

        // Track whether the modal is visible
	let isModalOpen = $state(false); 

	// Form inputs for the new cafe branch
	let cafeName = $state('');
	let branchLocation = $state('');
	let initialFloat = $state(1500); // Default cash drawer starting value

	async function handleLogin(e) {
		e.preventDefault();
		errorMessage = '';
		isLoading = true;

		// Validation edge cases before hitting auth provider/local db
		if (!email || !password) {
			errorMessage = 'Please fill in all fields.';
			isLoading = false;
			return;
		}

		try {
			// Connect your authentication pipeline here (e.g., Supabase / local auth)
			// await auth.signIn(email, password);
			
			console.log('Logging in...', { email });
		} catch (err) {
			errorMessage = err.message || 'Invalid credentials. Please try again.';
		} finally {
			isLoading = false;
		}
	}



	function toggleModal(e) {
		if (e) e.preventDefault(); // Stop the <a> tag from redirecting/reloading
		isModalOpen = !isModalOpen;
	}

	function handleCreateCafe(e) {
		e.preventDefault();
		
		// Handle your backend database insertion or API call here
		console.log("Registering operational node:", { cafeName, branchLocation, initialFloat });
		
		// Reset and close
		isModalOpen = false;
		cafeName = '';
		branchLocation = '';
	}
</script>

{#if isModalOpen}
	<div class="modal-backdrop" onclick={toggleModal} role="presentation">
		<div class="modal-card" onclick={(e) => e.stopPropagation()} role="dialog" aria-modal="true">
			
			<div class="modal-header">
				<h3><span class="info-icon">📋</span> Cafe Node Registration</h3>
				<button class="close-x-btn" onclick={toggleModal}>&times;</button>
			</div>

			<div class="modal-body">
				<div class="info-alert">
			
					<p>
						To <strong>ensure</strong> secure mesh networking and proper cash drawer configuration, all new branch provisionings are vetted by the core engineering group.
					</p>
				</div>
                
                <br />

				<div class="timeline-notice">
					<p><strong>Processing Timeline:</strong> 1 – 6 Business Days max within the Roastly ecosystem.</p>
				</div>

                  <br />

				<p class="instruction-text">
					Please fill out the official initialization matrix on Google Forms to submit your terminal hardware profiles and logistics details.
				</p>
			</div>

			<div class="modal-actions">
				<button type="button" class="cancel-btn" onclick={toggleModal}>Cancel</button>
				<a 
					href="https://forms.google.com/your-specific-form-id" 
					target="_blank" 
					rel="noopener noreferrer" 
					class="confirm-link-btn"
					onclick={toggleModal}
				>
					Open Registration Form ↗
				</a>
			</div>

		</div>
	</div>
{/if}

<div class="login-container">
	<div class="login-card">
		<!-- Brand Header -->
		<div class="brand-header">
			<span class="logo-icon">☕</span>
			<h1>Roastly <span class="accent-text">Admin</span></h1>
			<p class="subtitle">Secure Terminal Access</p>
		</div>

		<!-- Error Callout -->
		{#if errorMessage}
			<div class="error-banner" role="alert">
				<p>{errorMessage}</p>
			</div>
		{/if}

		<!-- Form -->
		<form onsubmit={handleLogin} class="login-form">
			<div class="input-group">
				<label for="email">Admin Email</label>
				<input
					type="email"
					id="email"
					bind:value={email}
					placeholder="name@roastly.com"
					autocomplete="email"
					required
					disabled={isLoading}
				/>
			</div>

			<div class="input-group">
				<div class="label-row">
					<label for="password">Password</label>
					<a href="/forgot-password" class="forgot-link">Forgot?</a>
				</div>
				<input
					type="password"
					id="password"
					bind:value={password}
					placeholder="••••••••"
					autocomplete="current-password"
					required
					disabled={isLoading}
				/>
			</div>

			<button type="submit" class="submit-btn" disabled={isLoading}>
				{#if isLoading}
					<span class="spinner"></span> Authenticating...
				{:else}
					Authorize Access
				{/if}
			</button>
		</form>
        <a href="" class="register-btn" onclick={toggleModal}>
            Create Cafe
        </a>
	</div>
</div>

<style>
	/* Scope-safe CSS implementing your locked color system */
	

	.login-container {
		display: flex;
		min-height: 100vh;
		align-items: center;
		justify-content: center;
		padding: 1.5rem;
	}

	.login-card {
		width: 100%;
		max-width: 420px;
		background-color: #FFFFFF; /* White base */
		padding: 2.5rem;
		border-radius: 16px;
		box-shadow: 0 10px 25px rgba(73, 64, 161, 0.08); /* Soft primary shadow */
		border: 1px solid #EAE6FF; /* Light Purple boundary hint */
	}

	.brand-header {
		text-align: center;
		margin-bottom: 2rem;
	}

	.logo-icon {
		font-size: 2.5rem;
		display: inline-block;
		margin-bottom: 0.5rem;
	}

	.brand-header h1 {
		color: #4940A1; /* Royal Purple Primary */
		font-size: 1.8rem;
		margin: 0;
		font-weight: 800;
		letter-spacing: -0.025em;
	}

	.accent-text {
		color: #FFA500; /* Golden Sun Accent */
	}

	.subtitle {
		color: #2E2570; /* Deep Purple Supporting */
		margin: 0.25rem 0 0 0;
		font-size: 0.9rem;
		opacity: 0.8;
	}

	.error-banner {
		background-color: #FFF0F0;
		border-left: 4px solid #D93838;
		padding: 0.75rem 1rem;
		border-radius: 6px;
		margin-bottom: 1.5rem;
	}

	.error-banner p {
		margin: 0;
		color: #D93838;
		font-size: 0.875rem;
		font-weight: 500;
	}

	.login-form {
		display: flex;
		flex-direction: column;
		gap: 1.25rem;
	}

	.input-group {
		display: flex;
		flex-direction: column;
		gap: 0.5rem;
	}

	.label-row {
		display: flex;
		justify-content: space-between;
		align-items: center;
	}

	label {
		color: #2E2570; /* Deep Purple Supporting */
		font-size: 0.875rem;
		font-weight: 600;
	}

	.forgot-link {
		color: #4940A1; /* Royal Purple */
		font-size: 0.75rem;
		font-weight: 500;
		text-decoration: none;
	}

	.forgot-link:hover {
		text-decoration: underline;
	}

	input {
		padding: 0.75rem 1rem;
		border: 1px solid #D6D1FA; /* Soft Light Purple boundary */
		border-radius: 8px;
		font-size: 1rem;
		color: #2E2570;
		transition: all 0.2s ease;
		background-color: #FFFFFF;
	}

	input:focus {
		outline: none;
		border-color: #4940A1; /* Royal Purple Focus */
		box-shadow: 0 0 0 3px rgba(73, 64, 161, 0.15);
	}

	input:disabled {
		background-color: #F8F9FA;
		cursor: not-allowed;
		opacity: 0.7;
	}

	.submit-btn {
		background-color: #4940A1; /* Royal Purple Primary */
		color: #FFFFFF;
		border: none;
		padding: 0.875rem;
		font-size: 1rem;
		font-weight: 600;
		border-radius: 8px;
		cursor: pointer;
		display: flex;
		align-items: center;
		justify-content: center;
		gap: 0.5rem;
		transition: background-color 0.2s ease, transform 0.1s ease;
		margin-top: 0.5rem;
	}

	.submit-btn:hover:not(:disabled) {
		background-color: #383080; /* Slightly deeper purple tone for active states */
	}

	.submit-btn:active:not(:disabled) {
		transform: scale(0.99);
	}

	.submit-btn:disabled {
		opacity: 0.5;
		cursor: not-allowed;
	}

 /* Trigger Button Styling */
	.register-btn {
		color: #FFFFFF;
		border: none;
		padding: 0.875rem;
		font-size: 1rem;
		font-weight: 600;
		border-radius: 8px;
		cursor: pointer;
		display: flex;
		align-items: center;
		justify-content: center;
		gap: 0.5rem;
		transition: background-color 0.2s ease, transform 0.1s ease;
		margin-top: 0.5rem;
        width: full;
		background-color: #FFA500; /* Royal Purple Primary */
		color: #FFFFFF;
		text-decoration: none;
		padding: 0.75rem 1.5rem;
		border-radius: 8px;
		font-weight: 600;
		transition: background-color 0.2s;
		cursor: pointer;
	}

	.register-btn:hover {
		background-color: #383080;
	}

	/* Modal Context Layering */
	.modal-backdrop {
		position: fixed;
		top: 0;
		left: 0;
		width: 100vw;
		height: 100vh;
		background-color: rgba(46, 37, 112, 0.4); /* Deep Purple base with alpha transparency */
		backdrop-filter: blur(4px);
		display: flex;
		align-items: center;
		justify-content: center;
		z-index: 1000;
	}

	.modal-card {
		background-color: #FFFFFF;
		width: 100%;
		max-width: 480px;
		padding: 2rem;
		border-radius: 16px;
		box-shadow: 0 20px 40px rgba(73, 64, 161, 0.15);
		border: 1px solid #EAE6FF; /* Light purple borders */
		animation: scaleUp 0.15s cubic-bezier(0.16, 1, 0.3, 1) forwards;
	}

	.modal-header {
		display: flex;
		justify-content: space-between;
		align-items: center;
		margin-bottom: 1.5rem;
		border-bottom: 1px solid #F0EEFF;
		padding-bottom: 1rem;
	}

	.modal-header h3 {
		margin: 0;
		color: #4940A1; /* Royal Purple */
		font-size: 1.25rem;
		font-weight: 700;
	}

	.close-x-btn {
		background: none;
		border: none;
		font-size: 1.5rem;
		color: #2E2570; /* Deep Purple */
		opacity: 0.5;
		cursor: pointer;
		transition: opacity 0.2s;
	}

	.close-x-btn:hover {
		opacity: 1;
	}

	/* Form Layout UI */
	.modal-form {
		display: flex;
		flex-direction: column;
		gap: 1.25rem;
	}

	.input-group {
		display: flex;
		flex-direction: column;
		gap: 0.5rem;
	}

	.input-group label {
		color: #2E2570; /* Deep Purple */
		font-size: 0.875rem;
		font-weight: 600;
	}

	.input-group input {
		padding: 0.75rem 1rem;
		border: 1px solid #D6D1FA;
		border-radius: 8px;
		font-size: 1rem;
		color: #2E2570;
		background-color: #F5FFFA; /* Mint Cream input backgrounds */
	}

	.input-group input:focus {
		outline: none;
		border-color: #4940A1;
		box-shadow: 0 0 0 3px rgba(73, 64, 161, 0.15);
	}

	/* Interactive Control Row */
	.modal-actions {
		display: flex;
		justify-content: flex-end;
		gap: 1rem;
		margin-top: 1rem;
		border-top: 1px solid #F0EEFF;
		padding-top: 1.25rem;
	}

	.cancel-btn {
		background: none;
		border: 1px solid #D6D1FA;
		color: #2E2570;
		padding: 0.75rem 1.25rem;
		border-radius: 8px;
		font-weight: 600;
		cursor: pointer;
		transition: background-color 0.2s;
	}

	.cancel-btn:hover {
		background-color: #F5FFFA; /* Mint Cream tint on hover */
	}

	.confirm-btn {
		background-color: #4940A1; /* Royal Purple Action */
		color: #FFFFFF;
		border: none;
		padding: 0.75rem 1.5rem;
		border-radius: 8px;
		font-weight: 600;
		cursor: pointer;
		transition: background-color 0.2s;
	}

	.confirm-btn:hover {
		background-color: #383080;
	}

	/* Micro Entrance Scale Animation */
	@keyframes scaleUp {
		from { transform: scale(0.96); opacity: 0; }
		to { transform: scale(1); opacity: 1; }
	}

	/* Micro-spinner for processing states */
	.spinner {
		width: 16px;
		height: 16px;
		border: 2px solid rgba(255, 255, 255, 0.3);
		border-radius: 50%;
		border-top-color: #FFFFFF;
		animation: spin 0.8s linear infinite;
	}

	@keyframes spin {
		to { transform: rotate(360deg); }
	}

    /* Converted Button to support safe <a> navigation semantics */
	.confirm-link-btn {
		background-color: #4940A1; /* Royal Purple Action */
		color: #FFFFFF;
		text-decoration: none;
		padding: 0.75rem 1.5rem;
		border-radius: 8px;
		font-weight: 600;
		display: inline-flex;
		align-items: center;
		transition: background-color 0.2s;
		cursor: pointer;
	}

	.confirm-link-btn:hover {
		background-color: #383080;
	}

	@keyframes scaleUp {
		from { transform: scale(0.96); opacity: 0; }
		to { transform: scale(1); opacity: 1; }
	}
</style>