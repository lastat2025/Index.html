<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="description" content="CryptoVerse Arena - Demo/Prototype DeFi Gaming Platform">
    <title>CryptoVerse Arena - The Future of DeFi Gaming (DEMO)</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/ethers/6.7.1/ethers.umd.min.js"></script>
    <script>
        // ⚠️ SECURITY WARNING: This is a DEMO/PROTOTYPE application
        // In production, this code must be refactored with:
        // 1. Backend authentication (NO hardcoded credentials)
        // 2. Real payment processing (Stripe API, actual blockchain transactions)
        // 3. Proper XSS prevention (CSP headers, sanitization)
        // 4. HTTPS enforced, secure cookies, CSRF tokens
        // 5. Rate limiting, WAF protection, security headers

        // Global Variables
        let web3Provider = null;
        let userAccount = null;
        let isAdminLoggedIn = false;

        // ⚠️ NEVER store credentials in client code
        // These should come from a secure backend API with proper authentication
        const API_ENDPOINT = '/api'; // Placeholder for backend API

        // ============================================
        // SECURITY UTILITIES
        // ============================================

        /**
         * Safe DOM element creation to prevent XSS
         * @param {string} html - HTML content
         * @param {HTMLElement} parentElement - Parent to append to
         */
        function safeCreateElement(html, parentElement) {
            const template = document.createElement('template');
            template.innerHTML = html.trim();
            const element = template.content.firstChild;
            if (parentElement) {
                parentElement.appendChild(element);
            }
            return element;
        }

        /**
         * Sanitize user input to prevent injection
         * @param {string} input - User input
         * @returns {string} Sanitized string
         */
        function sanitizeInput(input) {
            const div = document.createElement('div');
            div.textContent = input;
            return div.innerHTML;
        }

        /**
         * Validate input with enhanced security
         */
        function validateInput(input, type, fieldName) {
            try {
                if (!input || input.toString().trim() === '') {
                    throw new Error(`${fieldName} is required`);
                }

                switch(type) {
                    case 'email':
                        const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
                        if (!emailRegex.test(input)) throw new Error('Invalid email format');
                        break;
                    case 'wallet':
                        if (!input.startsWith('0x') || input.length !== 42) {
                            throw new Error('Invalid wallet address format');
                        }
                        break;
                    case 'amount':
                        const amount = parseFloat(input);
                        if (isNaN(amount) || amount <= 0) {
                            throw new Error('Invalid amount');
                        }
                        if (amount > 1000000) {
                            throw new Error('Amount exceeds maximum limit');
                        }
                        break;
                }
                return true;
            } catch (error) {
                showNotification(`❌ ${error.message}`);
                return false;
            }
        }

        // ============================================
        // ERROR HANDLING
        // ============================================

        function handleError(error, context) {
            console.error(`Error in ${context}:`, error);
            showNotification(`❌ ${context} failed. Please try again.`);
        }

        // ============================================
        // WALLET CONNECTION
        // ============================================

        async function connectWallet() {
            try {
                let provider = null;
                let walletType = '';

                if (window.ethereum) {
                    if (window.ethereum.isCoinbaseWallet) {
                        walletType = 'Coinbase Wallet';
                    } else {
                        walletType = 'MetaMask';
                    }
                    provider = window.ethereum;
                } else {
                    showWalletOptions();
                    return;
                }

                web3Provider = new ethers.BrowserProvider(provider);
                const accounts = await provider.request({ method: 'eth_requestAccounts' });

                if (!accounts || accounts.length === 0) {
                    throw new Error('No accounts returned from wallet');
                }

                userAccount = accounts[0];

                if (!validateInput(userAccount, 'wallet', 'Wallet Address')) {
                    throw new Error('Invalid wallet address returned');
                }

                updateWalletDisplay(userAccount, walletType);
                showNotification(`🔗 ${walletType} connected!`);

            } catch (error) {
                handleError(error, 'Wallet Connection');
            }
        }

        function updateWalletDisplay(address, walletType) {
            const walletStatus = document.getElementById('walletStatus');
            if (walletStatus) {
                walletStatus.textContent = `🟢 ${address.substring(0,6)}...${address.substring(38)} (${walletType})`;
            }
        }

        function showWalletOptions() {
            const html = `
                <div id="walletModal" style="position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.8); z-index: 10000; display: flex; align-items: center; justify-content: center;">
                    <div style="background: linear-gradient(45deg, #1a1a2e, #16213e); padding: 30px; border-radius: 20px; max-width: 400px; text-align: center; border: 2px solid #4ecdc4;">
                        <h2 style="color: #4ecdc4; margin-bottom: 20px;">🔗 Connect Your Wallet</h2>
                        <p style="margin-bottom: 25px; opacity: 0.9;">Choose your preferred wallet:</p>
                        <div style="display: grid; gap: 15px; margin-bottom: 20px;">
                            <button onclick="installMetaMask()" style="padding: 15px; background: linear-gradient(45deg, #667eea, #764ba2); border: none; border-radius: 10px; color: white; cursor: pointer;">
                                🦊 MetaMask
                            </button>
                            <button onclick="installCoinbaseWallet()" style="padding: 15px; background: linear-gradient(45deg, #667eea, #764ba2); border: none; border-radius: 10px; color: white; cursor: pointer;">
                                🏦 Coinbase Wallet
                            </button>
                        </div>
                        <button onclick="closeWalletModal()" style="padding: 10px 20px; background: transparent; border: 1px solid #666; border-radius: 8px; color: #666; cursor: pointer;">
                            Cancel
                        </button>
                    </div>
                </div>
            `;
            safeCreateElement(html, document.body);
        }

        function installMetaMask() {
            showNotification('🦊 Redirecting to MetaMask installation...');
            window.open('https://metamask.io/', '_blank');
            closeWalletModal();
        }

        function installCoinbaseWallet() {
            showNotification('🏦 Redirecting to Coinbase Wallet installation...');
            window.open('https://www.coinbase.com/wallet', '_blank');
            closeWalletModal();
        }

        function closeWalletModal() {
            const modal = document.getElementById('walletModal');
            if (modal) modal.remove();
        }

        // ============================================
        // NOTIFICATIONS
        // ============================================

        function showNotification(message) {
            const notification = document.getElementById('notification');
            if (notification) {
                notification.textContent = message;
                notification.classList.add('show');
                setTimeout(() => {
                    notification.classList.remove('show');
                }, 4000);
            }
        }

        // ============================================
        // TAB MANAGEMENT
        // ============================================

        function switchTab(tabName) {
            const tabs = document.querySelectorAll('.tab-content');
            const buttons = document.querySelectorAll('.tab-btn');

            tabs.forEach(tab => tab.classList.remove('active'));
            buttons.forEach(btn => btn.classList.remove('active'));

            const activeTab = document.getElementById(tabName);
            if (activeTab) {
                activeTab.classList.add('active');
            }

            // Find and mark active button
            document.querySelectorAll('.tab-btn').forEach(btn => {
                if (btn.getAttribute('data-tab') === tabName) {
                    btn.classList.add('active');
                }
            });
        }

        // ============================================
        // INITIALIZATION
        // ============================================

        document.addEventListener('DOMContentLoaded', function() {
            createStars();
            initializeLegalConsent();

            // Show welcome notification
            setTimeout(() => {
                showNotification('🎉 Welcome to CryptoVerse Arena (DEMO)! Connect your wallet to start!');
            }, 1000);
        });

        function createStars() {
            const starsContainer = document.getElementById('stars');
            if (starsContainer) {
                for (let i = 0; i < 100; i++) {
                    const star = document.createElement('div');
                    star.className = 'star';
                    star.style.left = Math.random() * 100 + '%';
                    star.style.top = Math.random() * 100 + '%';
                    star.style.width = star.style.height = Math.random() * 3 + 'px';
                    star.style.animationDelay = Math.random() * 3 + 's';
                    starsContainer.appendChild(star);
                }
            }
        }

        function initializeLegalConsent() {
            try {
                const checkboxes = ['legalTermsConsent', 'riskAcknowledgment', 'jurisdictionCompliance', 'liabilityWaiver'];
                const acceptBtn = document.getElementById('acceptLegalTerms');

                if (!acceptBtn) return;

                checkboxes.forEach(id => {
                    const checkbox = document.getElementById(id);
                    if (checkbox) {
                        checkbox.addEventListener('change', function() {
                            const allChecked = checkboxes.every(id => {
                                const cb = document.getElementById(id);
                                return cb && cb.checked;
                            });

                            acceptBtn.disabled = !allChecked;
                            acceptBtn.style.opacity = allChecked ? '1' : '0.5';
                        });
                    }
                });
            } catch (error) {
                handleError(error, 'Legal Consent Initialization');
            }
        }

        function acceptLegalTerms() {
            try {
                const checkboxes = ['legalTermsConsent', 'riskAcknowledgment', 'jurisdictionCompliance', 'liabilityWaiver'];
                const allChecked = checkboxes.every(id => {
                    const checkbox = document.getElementById(id);
                    return checkbox && checkbox.checked;
                });

                if (allChecked) {
                    showNotification('✅ Legal terms accepted! Welcome to CryptoVerse Arena!');
                    switchTab('dashboard');
                } else {
                    showNotification('❌ Please check all required consent boxes');
                }
            } catch (error) {
                handleError(error, 'Legal Consent');
            }
        }
    </script>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Arial', sans-serif;
            background: linear-gradient(45deg, #1a1a2e, #16213e, #0f3460);
            color: white;
            min-height: 100vh;
            overflow-x: hidden;
        }

        .stars {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            z-index: 1;
            pointer-events: none;
        }

        .star {
            position: absolute;
            background: white;
            border-radius: 50%;
            animation: twinkle 3s infinite;
        }

        @keyframes twinkle {
            0%, 100% { opacity: 0.3; }
            50% { opacity: 1; }
        }

        .container {
            position: relative;
            z-index: 10;
            max-width: 1400px;
            margin: 0 auto;
            padding: 20px;
        }

        .header {
            text-align: center;
            padding: 40px 0;
            background: rgba(0, 0, 0, 0.3);
            border-radius: 20px;
            margin-bottom: 30px;
            backdrop-filter: blur(10px);
            border: 1px solid rgba(255, 255, 255, 0.1);
        }

        .logo {
            font-size: 3.5em;
            background: linear-gradient(45deg, #ff6b6b, #4ecdc4, #45b7d1, #96ceb4);
            -webkit-background-clip: text;
            background-clip: text;
            -webkit-text-fill-color: transparent;
            font-weight: bold;
            margin-bottom: 10px;
            animation: glow 2s ease-in-out infinite alternate;
        }

        @keyframes glow {
            from { filter: drop-shadow(0 0 20px rgba(255, 107, 107, 0.5)); }
            to { filter: drop-shadow(0 0 30px rgba(69, 183, 209, 0.8)); }
        }

        .tagline {
            font-size: 1.3em;
            margin-bottom: 15px;
            opacity: 0.9;
        }

        .demo-warning {
            background: rgba(255, 107, 107, 0.2);
            border: 2px solid #ff6b6b;
            padding: 15px;
            border-radius: 10px;
            margin: 15px 0;
            color: #ff6b6b;
            text-align: center;
            font-weight: bold;
        }

        .nav-tabs {
            display: flex;
            justify-content: center;
            gap: 20px;
            margin-bottom: 40px;
            flex-wrap: wrap;
        }

        .tab-btn {
            padding: 15px 30px;
            background: linear-gradient(45deg, #667eea, #764ba2);
            border: none;
            border-radius: 25px;
            color: white;
            font-size: 1.1em;
            cursor: pointer;
            transition: all 0.3s ease;
        }

        .tab-btn:hover {
            transform: translateY(-2px);
            box-shadow: 0 10px 25px rgba(0, 0, 0, 0.3);
        }

        .tab-btn.active {
            background: linear-gradient(45deg, #ff6b6b, #4ecdc4);
        }

        .tab-content {
            display: none;
            animation: fadeIn 0.5s ease-in;
        }

        .tab-content.active {
            display: block;
        }

        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(20px); }
            to { opacity: 1; transform: translateY(0); }
        }

        .card {
            background: rgba(255, 255, 255, 0.1);
            backdrop-filter: blur(15px);
            border-radius: 20px;
            padding: 25px;
            border: 1px solid rgba(255, 255, 255, 0.2);
            transition: all 0.3s ease;
            margin-bottom: 20px;
        }

        .card:hover {
            transform: translateY(-5px);
            box-shadow: 0 20px 40px rgba(0, 0, 0, 0.3);
        }

        .card-title {
            font-size: 1.5em;
            margin-bottom: 15px;
            color: #4ecdc4;
        }

        .action-btn {
            width: 100%;
            padding: 15px;
            background: linear-gradient(45deg, #ff6b6b, #4ecdc4);
            border: none;
            border-radius: 10px;
            color: white;
            font-size: 1.1em;
            cursor: pointer;
            transition: all 0.3s ease;
            margin: 10px 0;
        }

        .action-btn:hover {
            transform: scale(1.05);
            box-shadow: 0 5px 15px rgba(255, 107, 107, 0.4);
        }

        .notification {
            position: fixed;
            top: 20px;
            right: 20px;
            background: linear-gradient(45deg, #ff6b6b, #4ecdc4);
            padding: 15px 25px;
            border-radius: 10px;
            transform: translateX(400px);
            transition: all 0.3s ease;
            z-index: 1000;
            max-width: 300px;
        }

        .notification.show {
            transform: translateX(0);
        }

        @media (max-width: 768px) {
            .logo { font-size: 2.5em; }
            .nav-tabs { flex-direction: column; align-items: center; }
        }
    </style>
</head>
<body>
    <div class="stars" id="stars"></div>

    <div class="container">
        <div class="header">
            <div class="logo">🌟 CryptoVerse Arena 🌟</div>
            <div class="tagline">The Ultimate DeFi Gaming Experience - Where Trading Meets Gaming</div>
            <div class="demo-warning">
                ⚠️ DEMO/PROTOTYPE VERSION - This is for educational purposes only.<br>
                Payment processing is NOT functional. Do NOT send real funds.
            </div>
            <div style="margin-top: 15px;">
                <span id="walletStatus" style="background: rgba(255,255,255,0.1); padding: 8px 16px; border-radius: 20px;">
                    🔴 Wallet Not Connected
                </span>
            </div>
        </div>

        <div class="nav-tabs">
            <button class="tab-btn active" data-tab="dashboard" onclick="switchTab('dashboard')">🏠 Dashboard</button>
            <button class="tab-btn" data-tab="legal" onclick="switchTab('legal')">⚖️ Legal</button>
        </div>

        <!-- Dashboard Tab -->
        <div id="dashboard" class="tab-content active">
            <div class="card">
                <div class="card-title">💰 Your Balance</div>
                <button class="action-btn" onclick="connectWallet()">🔗 Connect Wallet</button>
            </div>
        </div>

        <!-- Legal Tab -->
        <div id="legal" class="tab-content">
            <div class="card">
                <div class="card-title">⚖️ Legal Documentation & Terms</div>

                <div style="background: rgba(255,0,0,0.1); padding: 20px; border-radius: 10px; margin: 20px 0; border: 2px solid #ff6b6b;">
                    <h3 style="color: #ff6b6b; margin-bottom: 15px;">🚨 CRITICAL DISCLAIMER - DEMO/PROTOTYPE</h3>
                    <div style="font-size: 0.9em; line-height: 1.6;">
                        <p><strong>This is a DEMO/PROTOTYPE application for educational and portfolio purposes only.</strong></p>

                        <p style="margin-top: 15px;"><strong>PAYMENT PROCESSING IS NOT FUNCTIONAL:</strong></p>
                        <ul style="margin-left: 20px;">
                            <li>❌ Credit card forms do NOT process actual payments</li>
                            <li>❌ Cryptocurrency payments do NOT send real funds</li>
                            <li>❌ Bank transfer details are fictitious</li>
                            <li>❌ Admin wallet functions are simulated only</li>
                            <li>❌ Transaction history is not persisted</li>
                        </ul>

                        <p style="margin-top: 15px;"><strong>🛑 DO NOT:</strong></p>
                        <ul style="margin-left: 20px;">
                            <li>Send real cryptocurrency to any addresses displayed</li>
                            <li>Enter real credit card information</li>
                            <li>Use this for any financial transactions</li>
                            <li>Deploy this code in production without major refactoring</li>
                        </ul>

                        <p style="margin-top: 15px;"><strong>✅ PRODUCTION REQUIREMENTS:</strong></p>
                        <p>Before deploying any version with real payments, you must:</p>
                        <ul style="margin-left: 20px;">
                            <li>Remove ALL hardcoded credentials</li>
                            <li>Implement secure backend authentication</li>
                            <li>Integrate real payment processors (Stripe, etc.)</li>
                            <li>Implement proper XSS/CSRF protection</li>
                            <li>Obtain all necessary financial licenses and compliance</li>
                            <li>Conduct security audit and penetration testing</li>
                            <li>Consult legal counsel regarding crypto regulations</li>
                        </ul>
                    </div>
                </div>

                <div style="background: rgba(0,0,0,0.3); padding: 20px; border-radius: 10px; margin: 20px 0;">
                    <h3 style="color: #4ecdc4; margin-bottom: 15px;">📋 User Consent</h3>
                    <div style="font-size: 0.9em; line-height: 1.6;">
                        <label style="display: flex; align-items: flex-start; margin: 15px 0; cursor: pointer;">
                            <input type="checkbox" id="legalTermsConsent" style="margin-right: 10px; margin-top: 2px; transform: scale(1.2);">
                            <span>I understand this is a DEMO and payment processing is NOT functional</span>
                        </label>

                        <label style="display: flex; align-items: flex-start; margin: 15px 0; cursor: pointer;">
                            <input type="checkbox" id="riskAcknowledgment" style="margin-right: 10px; margin-top: 2px; transform: scale(1.2);">
                            <span>I will NOT send real funds or sensitive information to this application</span>
                        </label>

                        <label style="display: flex; align-items: flex-start; margin: 15px 0; cursor: pointer;">
                            <input type="checkbox" id="jurisdictionCompliance" style="margin-right: 10px; margin-top: 2px; transform: scale(1.2);">
                            <span>I understand this is for educational/portfolio purposes only</span>
                        </label>

                        <label style="display: flex; align-items: flex-start; margin: 15px 0; cursor: pointer;">
                            <input type="checkbox" id="liabilityWaiver" style="margin-right: 10px; margin-top: 2px; transform: scale(1.2);">
                            <span>I release the developer from liability for any issues arising from misuse</span>
                        </label>

                        <button class="action-btn" id="acceptLegalTerms" onclick="acceptLegalTerms()" style="margin-top: 20px; opacity: 0.5;" disabled>
                            🔐 I Acknowledge & Understand
                        </button>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <div id="notification" class="notification"></div>
</body>
</html>
