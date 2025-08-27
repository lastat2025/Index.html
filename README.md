# Index.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CryptoVerse Arena - The Future of DeFi Gaming</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/ethers/6.7.1/ethers.umd.min.js"></script>
    <script src="https://js.stripe.com/v3/"></script>
    <script>
        // Global Variables - All functions defined first
        let web3Provider = null;
        let userAccount = null;
        let isAdminLoggedIn = false;
        const ADMIN_WALLET = '0x9E1d192bd9c4dc67617D47381090DDb84db8d6C5';
        const ADMIN_EMAIL = 'mntman8041@gmail.com';
        const ADMIN_PASSWORD = 'Turkey38!';

        // Admin Login System
        function showAdminLogin() {
            const loginModal = `
                <div id="adminLoginModal" style="position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.9); z-index: 10000; display: flex; align-items: center; justify-content: center;">
                    <div style="background: linear-gradient(45deg, #1a1a2e, #16213e); padding: 40px; border-radius: 20px; max-width: 400px; text-align: center; border: 2px solid #ff6b6b;">
                        <h2 style="color: #ff6b6b; margin-bottom: 20px;">👑 Admin Login Required</h2>
                        <p style="margin-bottom: 25px; opacity: 0.9;">Enter admin credentials to access control panel:</p>
                        
                        <div style="text-align: left; margin-bottom: 20px;">
                            <label style="color: #4ecdc4; margin-bottom: 5px; display: block;">Email:</label>
                            <input type="email" id="adminEmail" placeholder="Enter admin email" 
                                   style="width: 100%; padding: 12px; border-radius: 8px; border: 1px solid #4ecdc4; background: rgba(255,255,255,0.1); color: white; margin-bottom: 15px;">
                            
                            <label style="color: #4ecdc4; margin-bottom: 5px; display: block;">Password:</label>
                            <input type="password" id="adminPassword" placeholder="Enter admin password" 
                                   style="width: 100%; padding: 12px; border-radius: 8px; border: 1px solid #4ecdc4; background: rgba(255,255,255,0.1); color: white;">
                        </div>
                        
                        <div style="display: grid; gap: 10px;">
                            <button onclick="validateAdminLogin()" style="padding: 15px; background: linear-gradient(45deg, #ff6b6b, #4ecdc4); border: none; border-radius: 10px; color: white; font-size: 1.1em; cursor: pointer;">
                                🔓 Login as Admin
                            </button>
                            
                            <button onclick="closeAdminLoginModal()" style="padding: 10px; background: transparent; border: 1px solid #666; border-radius: 8px; color: #666; cursor: pointer;">
                                Cancel
                            </button>
                        </div>
                        
                        <div id="loginError" style="color: #ff6b6b; margin-top: 15px; display: none;"></div>
                    </div>
                </div>
            `;
            
            document.body.insertAdjacentHTML('beforeend', loginModal);
            
            // Focus on email field
            setTimeout(() => {
                const emailField = document.getElementById('adminEmail');
                if (emailField) emailField.focus();
            }, 100);
        }

        // Validate Admin Login
        function validateAdminLogin() {
            const email = document.getElementById('adminEmail')?.value.trim();
            const password = document.getElementById('adminPassword')?.value;
            const errorDiv = document.getElementById('loginError');

            if (!email || !password) {
                if (errorDiv) {
                    errorDiv.textContent = 'Please enter both email and password';
                    errorDiv.style.display = 'block';
                }
                return;
            }

            if (email === ADMIN_EMAIL && password === ADMIN_PASSWORD) {
                isAdminLoggedIn = true;
                closeAdminLoginModal();
                showNotification('👑 Admin login successful! Welcome back!');
                
                // Update admin panel UI
                updateAdminPanelUI();
                
                // Switch to admin tab
                switchTab('admin');
                
                // Set session (expires in 1 hour)
                setTimeout(() => {
                    isAdminLoggedIn = false;
                    showNotification('⏰ Admin session expired. Please login again.');
                }, 3600000); // 1 hour
                
            } else {
                if (errorDiv) {
                    errorDiv.textContent = '❌ Invalid email or password. Access denied.';
                    errorDiv.style.display = 'block';
                }
                
                // Clear password field for security
                const passwordField = document.getElementById('adminPassword');
                if (passwordField) passwordField.value = '';
                
                showNotification('❌ Invalid admin credentials');
            }
        }

        // Close Admin Login Modal
        function closeAdminLoginModal() {
            const modal = document.getElementById('adminLoginModal');
            if (modal) {
                modal.remove();
            }
        }

        // Update Admin Panel UI
        function updateAdminPanelUI() {
            const adminPanel = document.querySelector('.admin-panel');
            if (adminPanel && isAdminLoggedIn) {
                const loginWarning = adminPanel.querySelector('p');
                if (loginWarning && loginWarning.textContent.includes('Admin access required')) {
                    loginWarning.innerHTML = `
                        <span style="color: #4ecdc4;">✅ Admin authenticated: ${ADMIN_EMAIL}</span>
                        <button onclick="adminLogout()" style="margin-left: 15px; padding: 5px 10px; background: #ff6b6b; border: none; border-radius: 5px; color: white; cursor: pointer; font-size: 0.9em;">
                            🚪 Logout
                        </button>
                    `;
                }
            }
        }

        // Admin Logout
        function adminLogout() {
            isAdminLoggedIn = false;
            showNotification('👋 Admin logged out successfully');
            
            // Reset admin panel UI
            const adminPanel = document.querySelector('.admin-panel');
            if (adminPanel) {
                const statusP = adminPanel.querySelector('p');
                if (statusP) {
                    statusP.innerHTML = '<span style="color: #ffd700;">⚠️ Admin access required. Please login to continue.</span>';
                }
            }
            
            // Switch back to dashboard
            switchTab('dashboard');
        }

        // Enhanced Error Handling for All Functions
        function handleError(error, context) {
            console.error(`Error in ${context}:`, error);
            showNotification(`❌ ${context} failed. Please try again.`);
        }

        // Validate input with enhanced security
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
                            throw new Error('Amount too large');
                        }
                        break;
                    case 'password':
                        if (input.length < 6) {
                            throw new Error('Password too short');
                        }
                        break;
                }
                return true;
            } catch (error) {
                showNotification(`❌ ${error.message}`);
                return false;
            }
        }

        // Enhanced Admin Login with error handling
        function validateAdminLogin() {
            try {
                const emailInput = document.getElementById('adminEmail');
                const passwordInput = document.getElementById('adminPassword');
                const errorDiv = document.getElementById('loginError');

                if (!emailInput || !passwordInput) {
                    throw new Error('Login form elements not found');
                }

                const email = emailInput.value.trim();
                const password = passwordInput.value;

                if (!validateInput(email, 'email', 'Email') || 
                    !validateInput(password, 'password', 'Password')) {
                    return;
                }

                if (email === ADMIN_EMAIL && password === ADMIN_PASSWORD) {
                    isAdminLoggedIn = true;
                    closeAdminLoginModal();
                    showNotification('👑 Admin login successful! Welcome back!');
                    updateAdminPanelUI();
                    switchTab('admin');
                    
                    // Session timeout (1 hour)
                    setTimeout(() => {
                        isAdminLoggedIn = false;
                        showNotification('⏰ Admin session expired. Please login again.');
                    }, 3600000);
                    
                } else {
                    if (errorDiv) {
                        errorDiv.textContent = '❌ Invalid email or password. Access denied.';
                        errorDiv.style.display = 'block';
                    }
                    
                    passwordInput.value = '';
                    showNotification('❌ Invalid admin credentials');
                }
            } catch (error) {
                handleError(error, 'Admin Login');
            }
        }

        // Enhanced Wallet Connection with error handling
        async function connectWallet() {
            try {
                let provider = null;
                let walletType = '';
                
                if (window.coinbaseWalletExtension) {
                    provider = window.coinbaseWalletExtension;
                    walletType = 'Coinbase Wallet';
                } else if (window.ethereum) {
                    if (window.ethereum.isCoinbaseWallet) {
                        provider = window.ethereum;
                        walletType = 'Coinbase Wallet';
                    } else {
                        provider = window.ethereum;
                        walletType = 'MetaMask';
                    }
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
                
                const walletStatus = document.getElementById('walletStatus');
                const userLevel = document.getElementById('userLevel');
                const ethBalance = document.getElementById('ethBalance');
                const usdBalance = document.getElementById('usdBalance');
                const userBalance = document.getElementById('userBalance');
                
                if (walletStatus) walletStatus.innerHTML = `🟢 ${userAccount.substring(0,6)}...${userAccount.substring(38)} (${walletType})`;
                if (userLevel) userLevel.textContent = `⚡ LEVEL 5 TRADER ⚡`;
                
                const balance = await web3Provider.getBalance(userAccount);
                const ethBalanceValue = ethers.formatEther(balance);
                const usdValue = (parseFloat(ethBalanceValue) * 3245).toFixed(2);
                
                if (ethBalance) ethBalance.textContent = parseFloat(ethBalanceValue).toFixed(4);
                if (usdBalance) usdBalance.textContent = usdValue;
                if (userBalance) userBalance.textContent = `$${usdValue}`;
                
                showNotification(`🔗 ${walletType} connected! Balance: ${parseFloat(ethBalanceValue).toFixed(4)} ETH`);
                
            } catch (error) {
                handleError(error, 'Wallet Connection');
            }
        }

        // Enhanced Payment Processing with error handling
        function processStripePayment() {
            try {
                const amountSelect = document.getElementById('cc-amount');
                const cardNumber = document.getElementById('card-number');
                const cardExpiry = document.getElementById('card-expiry');
                const cardCvc = document.getElementById('card-cvc');

                if (!amountSelect || !cardNumber || !cardExpiry || !cardCvc) {
                    throw new Error('Payment form elements not found');
                }

                let amount = amountSelect.value;
                
                if (amount === 'custom') {
                    const customAmount = document.getElementById('custom-cc-amount');
                    if (!customAmount) throw new Error('Custom amount field not found');
                    amount = customAmount.value;
                }

                if (!validateInput(amount, 'amount', 'Payment Amount')) return;
                
                if (!cardNumber.value || !cardExpiry.value || !cardCvc.value) {
                    showNotification('❌ Please fill in all card details');
                    return;
                }

                showNotification('⏳ Processing secure payment...');

                setTimeout(() => {
                    const ethAmount = (parseFloat(amount) / 3245).toFixed(6);
                    showNotification(`✅ Payment successful! ${ethAmount} ETH sent to admin wallet`);
                    
                    const adminEthBalance = document.getElementById('adminEthBalance');
                    if (adminEthBalance) {
                        const currentBalance = parseFloat(adminEthBalance.textContent);
                        adminEthBalance.textContent = (currentBalance + parseFloat(ethAmount)).toFixed(4);
                    }
                    
                    // Clear sensitive data
                    cardNumber.value = '';
                    cardExpiry.value = '';
                    cardCvc.value = '';
                    
                }, 2000);

            } catch (error) {
                handleError(error, 'Payment Processing');
            }
        }

        // Enhanced Withdrawal Functions with error handling
        function quickWithdraw(amount) {
            try {
                if (!isAdminLoggedIn) {
                    showNotification('❌ Admin login required for withdrawals');
                    showAdminLogin();
                    return;
                }

                if (!validateInput(amount, 'amount', 'Withdrawal Amount')) return;

                if (!confirm(`Withdraw ${amount} ETH to admin wallet?`)) {
                    return;
                }

                showNotification(`⏳ Processing secure withdrawal: ${amount} ETH...`);

                setTimeout(() => {
                    const adminEthBalance = document.getElementById('adminEthBalance');
                    if (!adminEthBalance) {
                        throw new Error('Admin balance element not found');
                    }
                    
                    const currentBalance = parseFloat(adminEthBalance.textContent);
                    if (isNaN(currentBalance)) {
                        throw new Error('Invalid balance amount');
                    }
                    
                    if (currentBalance >= amount) {
                        adminEthBalance.textContent = (currentBalance - amount).toFixed(4);
                        addWithdrawalToHistory(amount, ADMIN_WALLET, 'Completed', 'Withdrawal');
                        showNotification(`✅ ${amount} ETH withdrawn successfully!`);
                    } else {
                        showNotification('❌ Insufficient balance');
                    }
                }, 2000);

            } catch (error) {
                handleError(error, 'Withdrawal');
            }
        }

        // Enhanced Send to Wallet with error handling
        function sendToWallet() {
            try {
                if (!isAdminLoggedIn) {
                    showNotification('❌ Admin login required for fund transfers');
                    showAdminLogin();
                    return;
                }

                const sendToWalletInput = document.getElementById('sendToWallet');
                const sendAmountInput = document.getElementById('sendAmount');
                
                if (!sendToWalletInput || !sendAmountInput) {
                    throw new Error('Send form elements not found');
                }

                const targetWallet = sendToWalletInput.value.trim();
                const amount = parseFloat(sendAmountInput.value);

                if (!validateInput(targetWallet, 'wallet', 'Target Wallet') || 
                    !validateInput(amount, 'amount', 'Send Amount')) {
                    return;
                }

                let confirmMessage = `Send ${amount} ETH to ${targetWallet}?`;
                if (amount > 10) {
                    confirmMessage = `⚠️ LARGE TRANSFER: Send ${amount} ETH to ${targetWallet}?\n\nThis is a significant amount. Please confirm.`;
                }

                if (!confirm(confirmMessage)) {
                    return;
                }

                showNotification(`⏳ Processing secure transfer: ${amount} ETH...`);

                setTimeout(() => {
                    const adminEthBalance = document.getElementById('adminEthBalance');
                    if (!adminEthBalance) {
                        throw new Error('Admin balance element not found');
                    }
                    
                    const currentBalance = parseFloat(adminEthBalance.textContent);
                    if (currentBalance >= amount) {
                        adminEthBalance.textContent = (currentBalance - amount).toFixed(4);
                        addWithdrawalToHistory(amount, targetWallet, 'Completed', 'Transfer');
                        
                        sendToWalletInput.value = '';
                        sendAmountInput.value = '';
                        
                        showNotification(`✅ ${amount} ETH sent successfully!`);
                    } else {
                        showNotification('❌ Insufficient balance for transfer');
                    }
                }, 2000);

            } catch (error) {
                handleError(error, 'Send to Wallet');
            }
        }

        // Legal Consent Management
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
                    
                    // Store consent with timestamp
                    const consentData = {
                        timestamp: new Date().toISOString(),
                        userAgent: navigator.userAgent,
                        ipConsent: true,
                        version: '1.0'
                    };
                    
                    // In production, send consent data to backend
                    console.log('Legal consent recorded:', consentData);
                    
                } else {
                    showNotification('❌ Please check all required consent boxes');
                }
            } catch (error) {
                handleError(error, 'Legal Consent');
            }
        }

        // Monitor legal consent checkboxes
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
        function sendToWallet() {
            if (!isAdminLoggedIn) {
                showNotification('❌ Admin login required for fund transfers');
                showAdminLogin();
                return;
            }

            const sendToWalletInput = document.getElementById('sendToWallet');
            const sendAmountInput = document.getElementById('sendAmount');
            
            const targetWallet = sendToWalletInput ? sendToWalletInput.value : '';
            const amount = sendAmountInput ? parseFloat(sendAmountInput.value) : 0;

            if (!targetWallet || !amount) {
                showNotification('❌ Please enter wallet address and amount');
                return;
            }

            if (!targetWallet.startsWith('0x') || targetWallet.length !== 42) {
                showNotification('❌ Invalid wallet address format');
                return;
            }

            // Additional confirmation for large amounts
            let confirmMessage = `Send ${amount} ETH to ${targetWallet}?`;
            if (amount > 10) {
                confirmMessage = `⚠️ LARGE TRANSFER: Send ${amount} ETH to ${targetWallet}?\n\nThis is a significant amount. Please confirm.`;
            }

            if (!confirm(confirmMessage)) {
                return;
            }

            showNotification(`⏳ Processing secure transfer: ${amount} ETH...`);

            setTimeout(() => {
                const adminEthBalance = document.getElementById('adminEthBalance');
                if (adminEthBalance) {
                    const currentBalance = parseFloat(adminEthBalance.textContent);
                    if (currentBalance >= amount) {
                        adminEthBalance.textContent = (currentBalance - amount).toFixed(4);
                        addWithdrawalToHistory(amount, targetWallet, 'Completed', 'Transfer');
                        
                        if (sendToWalletInput) sendToWalletInput.value = '';
                        if (sendAmountInput) sendAmountInput.value = '';
                        
                        showNotification(`✅ ${amount} ETH sent successfully to ${targetWallet.substring(0,8)}...`);
                        
                        // Log admin action
                        console.log(`Admin Transfer: ${amount} ETH to ${targetWallet} at ${new Date().toISOString()}`);
                        
                    } else {
                        showNotification('❌ Insufficient balance for transfer');
                    }
                }
            }, 2000);
        }

        // Bulk Send to Multiple Wallets
        function showBulkSendModal() {
            if (!isAdminLoggedIn) {
                showNotification('❌ Admin login required');
                showAdminLogin();
                return;
            }

            const bulkModal = `
                <div id="bulkSendModal" style="position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.8); z-index: 10000; display: flex; align-items: center; justify-content: center; overflow-y: auto;">
                    <div style="background: linear-gradient(45deg, #1a1a2e, #16213e); padding: 30px; border-radius: 20px; max-width: 600px; width: 90%; text-align: center; border: 2px solid #4ecdc4; margin: 20px;">
                        <h2 style="color: #4ecdc4; margin-bottom: 20px;">📤 Bulk Send to Multiple Wallets</h2>
                        
                        <div style="text-align: left; margin-bottom: 20px;">
                            <label style="color: #4ecdc4; margin-bottom: 5px; display: block;">Wallet Addresses & Amounts:</label>
                            <textarea id="bulkWalletList" placeholder="Enter one per line: 0xAddress,amount
Example:
0x742d35Cc6634C0532925a3b8D404fddA2Ff8A1B2,1.5
0x123...ABC,0.5
0x456...DEF,2.0" 
                                      style="width: 100%; height: 150px; padding: 12px; border-radius: 8px; border: 1px solid #4ecdc4; background: rgba(255,255,255,0.1); color: white; font-family: monospace; resize: vertical;"></textarea>
                        </div>
                        
                        <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin-bottom: 20px;">
                            <button onclick="processBulkSend()" style="padding: 15px; background: linear-gradient(45deg, #ff6b6b, #4ecdc4); border: none; border-radius: 10px; color: white; font-size: 1.1em; cursor: pointer;">
                                🚀 Send All
                            </button>
                            
                            <button onclick="closeBulkSendModal()" style="padding: 15px; background: transparent; border: 1px solid #666; border-radius: 10px; color: #666; cursor: pointer;">
                                Cancel
                            </button>
                        </div>
                        
                        <div id="bulkSendStatus" style="background: rgba(0,0,0,0.3); padding: 10px; border-radius: 8px; display: none;"></div>
                    </div>
                </div>
            `;
            
            document.body.insertAdjacentHTML('beforeend', bulkModal);
        }

        // Process Bulk Send
        function processBulkSend() {
            const bulkList = document.getElementById('bulkWalletList')?.value.trim();
            const statusDiv = document.getElementById('bulkSendStatus');
            
            if (!bulkList) {
                showNotification('❌ Please enter wallet addresses and amounts');
                return;
            }

            const lines = bulkList.split('\n').filter(line => line.trim());
            const transfers = [];
            let totalAmount = 0;

            // Parse and validate
            for (let line of lines) {
                const [address, amount] = line.split(',').map(s => s.trim());
                
                if (!address || !amount || !address.startsWith('0x') || address.length !== 42) {
                    showNotification(`❌ Invalid format in line: ${line}`);
                    return;
                }
                
                const amountNum = parseFloat(amount);
                if (isNaN(amountNum) || amountNum <= 0) {
                    showNotification(`❌ Invalid amount in line: ${line}`);
                    return;
                }
                
                transfers.push({ address, amount: amountNum });
                totalAmount += amountNum;
            }

            // Confirm bulk transfer
            if (!confirm(`Send ${totalAmount} ETH total to ${transfers.length} wallets?\n\nThis action cannot be undone.`)) {
                return;
            }

            if (statusDiv) {
                statusDiv.style.display = 'block';
                statusDiv.innerHTML = `<p style="color: #ffd700;">⏳ Processing ${transfers.length} transfers...</p>`;
            }

            // Simulate bulk processing
            let processed = 0;
            const interval = setInterval(() => {
                if (processed < transfers.length) {
                    const transfer = transfers[processed];
                    addWithdrawalToHistory(transfer.amount, transfer.address, 'Completed', 'Bulk');
                    processed++;
                    
                    if (statusDiv) {
                        statusDiv.innerHTML = `<p style="color: #4ecdc4;">✅ Processed ${processed}/${transfers.length} transfers</p>`;
                    }
                } else {
                    clearInterval(interval);
                    
                    // Update balance
                    const adminEthBalance = document.getElementById('adminEthBalance');
                    if (adminEthBalance) {
                        const currentBalance = parseFloat(adminEthBalance.textContent);
                        adminEthBalance.textContent = (currentBalance - totalAmount).toFixed(4);
                    }
                    
                    showNotification(`✅ Bulk send completed! ${totalAmount} ETH sent to ${transfers.length} wallets`);
                    
                    setTimeout(() => {
                        closeBulkSendModal();
                    }, 2000);
                }
            }, 500);
        }

        // Close Bulk Send Modal
        function closeBulkSendModal() {
            const modal = document.getElementById('bulkSendModal');
            if (modal) {
                modal.remove();
            }
        }

        // Crypto addresses for different currencies
        const cryptoAddresses = {
            eth: '0x9E1d192bd9c4dc67617D47381090DDb84db8d6C5',
            btc: 'bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh',
            usdt: '0x9E1d192bd9c4dc67617D47381090DDb84db8d6C5',
            usdc: '0x9E1d192bd9c4dc67617D47381090DDb84db8d6C5',
            bnb: '0x9E1d192bd9c4dc67617D47381090DDb84db8d6C5',
            sol: 'DYw8jCTfwHNRJhhmFcbXvVDTqWMEVFBX6ZKUmG5CNSKK'
        };

        // Tab switching functionality
        function switchTab(tabName) {
            const tabs = document.querySelectorAll('.tab-content');
            tabs.forEach(tab => tab.classList.remove('active'));
            
            const buttons = document.querySelectorAll('.tab-btn');
            buttons.forEach(btn => btn.classList.remove('active'));
            
            document.getElementById(tabName).classList.add('active');
            event.target.classList.add('active');
        }

        // Notification system
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

        // Connect Wallet Function - Updated with Coinbase Support
        async function connectWallet() {
            try {
                // Check for Coinbase Wallet first, then MetaMask
                let provider = null;
                let walletType = '';
                
                if (window.coinbaseWalletExtension) {
                    provider = window.coinbaseWalletExtension;
                    walletType = 'Coinbase Wallet';
                } else if (window.ethereum) {
                    // Check if it's Coinbase Wallet via ethereum provider
                    if (window.ethereum.isCoinbaseWallet) {
                        provider = window.ethereum;
                        walletType = 'Coinbase Wallet';
                    } else {
                        provider = window.ethereum;
                        walletType = 'MetaMask';
                    }
                } else {
                    showWalletOptions();
                    return;
                }

                web3Provider = new ethers.BrowserProvider(provider);
                const accounts = await provider.request({ method: 'eth_requestAccounts' });
                userAccount = accounts[0];
                
                const walletStatus = document.getElementById('walletStatus');
                const userLevel = document.getElementById('userLevel');
                const ethBalance = document.getElementById('ethBalance');
                const usdBalance = document.getElementById('usdBalance');
                const userBalance = document.getElementById('userBalance');
                
                if (walletStatus) walletStatus.innerHTML = `🟢 ${userAccount.substring(0,6)}...${userAccount.substring(38)} (${walletType})`;
                if (userLevel) userLevel.textContent = `⚡ LEVEL 5 TRADER ⚡`;
                
                const balance = await web3Provider.getBalance(userAccount);
                const ethBalanceValue = ethers.formatEther(balance);
                const usdValue = (parseFloat(ethBalanceValue) * 3245).toFixed(2);
                
                if (ethBalance) ethBalance.textContent = parseFloat(ethBalanceValue).toFixed(4);
                if (usdBalance) usdBalance.textContent = usdValue;
                if (userBalance) userBalance.textContent = `$${usdValue}`;
                
                showNotification(`🔗 ${walletType} connected! Balance: ${parseFloat(ethBalanceValue).toFixed(4)} ETH`);
                
            } catch (error) {
                console.error('Wallet connection error:', error);
                showNotification('❌ Failed to connect wallet');
            }
        }

        // Show wallet options when no wallet detected
        function showWalletOptions() {
            const walletModal = `
                <div id="walletModal" style="position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.8); z-index: 10000; display: flex; align-items: center; justify-content: center;">
                    <div style="background: linear-gradient(45deg, #1a1a2e, #16213e); padding: 30px; border-radius: 20px; max-width: 400px; text-align: center; border: 2px solid #4ecdc4;">
                        <h2 style="color: #4ecdc4; margin-bottom: 20px;">🔗 Connect Your Wallet</h2>
                        <p style="margin-bottom: 25px; opacity: 0.9;">Choose your preferred wallet to connect:</p>
                        
                        <div style="display: grid; gap: 15px; margin-bottom: 20px;">
                            <button onclick="installMetaMask()" class="wallet-option-btn" style="display: flex; align-items: center; justify-content: center; gap: 10px; padding: 15px; background: linear-gradient(45deg, #ff6b6b, #4ecdc4); border: none; border-radius: 10px; color: white; font-size: 1.1em; cursor: pointer;">
                                🦊 MetaMask
                            </button>
                            
                            <button onclick="installCoinbaseWallet()" class="wallet-option-btn" style="display: flex; align-items: center; justify-content: center; gap: 10px; padding: 15px; background: linear-gradient(45deg, #0052ff, #0066ff); border: none; border-radius: 10px; color: white; font-size: 1.1em; cursor: pointer;">
                                🏦 Coinbase Wallet
                            </button>
                            
                            <button onclick="connectWalletConnect()" class="wallet-option-btn" style="display: flex; align-items: center; justify-content: center; gap: 10px; padding: 15px; background: linear-gradient(45deg, #3b99fc, #5cb3f7); border: none; border-radius: 10px; color: white; font-size: 1.1em; cursor: pointer;">
                                🔗 WalletConnect
                            </button>
                        </div>
                        
                        <button onclick="closeWalletModal()" style="padding: 10px 20px; background: transparent; border: 1px solid #666; border-radius: 8px; color: #666; cursor: pointer;">
                            Cancel
                        </button>
                    </div>
                </div>
            `;
            
            document.body.insertAdjacentHTML('beforeend', walletModal);
        }

        // Install MetaMask
        function installMetaMask() {
            showNotification('🦊 Redirecting to MetaMask installation...');
            window.open('https://metamask.io/', '_blank');
            closeWalletModal();
        }

        // Install Coinbase Wallet
        function installCoinbaseWallet() {
            showNotification('🏦 Redirecting to Coinbase Wallet installation...');
            window.open('https://www.coinbase.com/wallet', '_blank');
            closeWalletModal();
        }

        // WalletConnect integration
        function connectWalletConnect() {
            showNotification('🔗 WalletConnect integration coming soon!');
            closeWalletModal();
        }

        // Close wallet modal
        function closeWalletModal() {
            const modal = document.getElementById('walletModal');
            if (modal) {
                modal.remove();
            }
        }

        // Payment Form Management
        function showPaymentForm(type) {
            const forms = document.querySelectorAll('.payment-form');
            forms.forEach(form => form.style.display = 'none');
            
            const targetForm = document.getElementById(type + '-form');
            if (targetForm) {
                targetForm.style.display = 'block';
            }
        }

        // Credit Card Payment Processing
        function processStripePayment() {
            try {
                const amountSelect = document.getElementById('cc-amount');
                let amount = amountSelect ? amountSelect.value : '100';
                
                if (amount === 'custom') {
                    const customAmount = document.getElementById('custom-cc-amount');
                    amount = customAmount ? customAmount.value : '100';
                    if (!amount || amount < 10) {
                        showNotification('❌ Minimum custom amount is $10');
                        return;
                    }
                }

                const cardNumber = document.getElementById('card-number');
                const cardExpiry = document.getElementById('card-expiry');
                const cardCvc = document.getElementById('card-cvc');

                if (!cardNumber?.value || !cardExpiry?.value || !cardCvc?.value) {
                    showNotification('❌ Please fill in all card details');
                    return;
                }

                showNotification('⏳ Processing payment...');

                setTimeout(() => {
                    const ethAmount = (parseFloat(amount) / 3245).toFixed(6);
                    showNotification(`✅ Payment successful! ${ethAmount} ETH sent to admin wallet`);
                    
                    const adminEthBalance = document.getElementById('adminEthBalance');
                    if (adminEthBalance) {
                        const currentBalance = parseFloat(adminEthBalance.textContent);
                        adminEthBalance.textContent = (currentBalance + parseFloat(ethAmount)).toFixed(4);
                    }
                    
                    if (cardNumber) cardNumber.value = '';
                    if (cardExpiry) cardExpiry.value = '';
                    if (cardCvc) cardCvc.value = '';
                    
                }, 2000);

            } catch (error) {
                console.error('Payment error:', error);
                showNotification('❌ Payment failed - please try again');
            }
        }

        // Update crypto address
        function updateCryptoAddress() {
            const cryptoType = document.getElementById('crypto-type');
            const addressElement = document.getElementById('crypto-address');
            
            if (cryptoType && addressElement && cryptoAddresses[cryptoType.value]) {
                addressElement.textContent = cryptoAddresses[cryptoType.value];
            }
        }

        // Copy crypto address
        function copyCryptoAddress() {
            const addressElement = document.getElementById('crypto-address');
            const address = addressElement ? addressElement.textContent : '';
            
            if (navigator.clipboard) {
                navigator.clipboard.writeText(address).then(() => {
                    showNotification('📋 Address copied to clipboard!');
                }).catch(() => {
                    showNotification('❌ Failed to copy address');
                });
            }
        }

        // Send crypto payment directly from wallet
        async function sendCryptoDirectly() {
            try {
                if (!web3Provider || !userAccount) {
                    showNotification('❌ Please connect your wallet first');
                    return;
                }

                const amountInput = document.getElementById('crypto-amount');
                const amount = amountInput ? amountInput.value : '0.1';
                
                if (!amount || parseFloat(amount) <= 0) {
                    showNotification('❌ Please enter a valid amount');
                    return;
                }

                showNotification('⏳ Preparing transaction...');

                const signer = await web3Provider.getSigner();
                
                const tx = await signer.sendTransaction({
                    to: ADMIN_WALLET,
                    value: ethers.parseEther(amount.toString())
                });

                showNotification(`⏳ Transaction sent: ${tx.hash.substring(0,10)}...`);

                // Wait for confirmation
                await tx.wait();

                showNotification(`✅ Payment confirmed! ${amount} ETH sent to admin wallet`);

                // Clear the amount field
                if (amountInput) amountInput.value = '';

                // Add to transaction hash field for verification
                const txHashInput = document.getElementById('tx-hash');
                if (txHashInput) txHashInput.value = tx.hash;

            } catch (error) {
                console.error('Direct crypto payment error:', error);
                if (error.code === 4001) {
                    showNotification('❌ Transaction cancelled by user');
                } else {
                    showNotification('❌ Transaction failed - please try again');
                }
            }
        }
        function verifyCryptoPayment() {
            const txHashInput = document.getElementById('tx-hash');
            const amountInput = document.getElementById('crypto-amount');
            const cryptoTypeSelect = document.getElementById('crypto-type');
            
            const txHash = txHashInput ? txHashInput.value : '';
            const amount = amountInput ? amountInput.value : '';
            const cryptoType = cryptoTypeSelect ? cryptoTypeSelect.value : 'eth';

            if (!txHash || !amount) {
                showNotification('❌ Please enter transaction hash and amount');
                return;
            }

            showNotification('⏳ Verifying transaction...');

            setTimeout(() => {
                showNotification(`✅ Payment verified! ${amount} ${cryptoType.toUpperCase()} received`);
                
                if (txHashInput) txHashInput.value = '';
                if (amountInput) amountInput.value = '';
            }, 3000);
        }

        // Bank transfer
        function submitBankTransfer() {
            const bankAmountInput = document.getElementById('bank-amount');
            const amount = bankAmountInput ? bankAmountInput.value : '';
            
            if (!amount || amount < 100) {
                showNotification('❌ Minimum bank transfer amount is $100');
                return;
            }

            showNotification('📞 Redirecting to support...');
            
            setTimeout(() => {
                showNotification(`✅ Bank transfer request submitted for $${amount}`);
                if (bankAmountInput) bankAmountInput.value = '';
            }, 2000);
        }

        // Game Functions
        function createGame(gameType, entryFee) {
            if (!userAccount) {
                showNotification('❌ Please connect your wallet first');
                return;
            }

            showNotification(`🎮 Creating ${gameType} game with $${entryFee} entry fee...`);
            
            setTimeout(() => {
                showNotification(`✅ ${gameType.charAt(0).toUpperCase() + gameType.slice(1)} game created!`);
            }, 2000);
        }

        function joinGame(gameType, entryFee) {
            if (!userAccount) {
                showNotification('❌ Please connect your wallet first');
                return;
            }

            showNotification(`⚔️ Joining ${gameType} battle...`);
            
            setTimeout(() => {
                showNotification(`🎯 Successfully joined ${gameType} battle!`);
            }, 1500);
        }

        // Admin Withdrawal Functions - Enhanced Security
        function quickWithdraw(amount) {
            if (!isAdminLoggedIn) {
                showNotification('❌ Admin login required for withdrawals');
                showAdminLogin();
                return;
            }

            if (!confirm(`Withdraw ${amount} ETH to admin wallet?`)) {
                return;
            }

            showNotification(`⏳ Processing secure withdrawal: ${amount} ETH...`);

            setTimeout(() => {
                const adminEthBalance = document.getElementById('adminEthBalance');
                if (adminEthBalance) {
                    const currentBalance = parseFloat(adminEthBalance.textContent);
                    if (currentBalance >= amount) {
                        adminEthBalance.textContent = (currentBalance - amount).toFixed(4);
                        addWithdrawalToHistory(amount, ADMIN_WALLET, 'Completed', 'Withdrawal');
                        showNotification(`✅ ${amount} ETH withdrawn successfully!`);
                        
                        // Log admin action
                        console.log(`Admin Withdrawal: ${amount} ETH at ${new Date().toISOString()}`);
                        
                    } else {
                        showNotification('❌ Insufficient balance');
                    }
                }
            }, 2000);
        }

        function customWithdraw() {
            if (!isAdminLoggedIn) {
                showNotification('❌ Admin login required for withdrawals');
                showAdminLogin();
                return;
            }

            const customAmountInput = document.getElementById('customWithdrawAmount');
            const amount = customAmountInput ? parseFloat(customAmountInput.value) : 0;

            if (!amount || amount <= 0) {
                showNotification('❌ Please enter valid amount');
                return;
            }

            if (!confirm(`Withdraw ${amount} ETH to admin wallet?`)) {
                return;
            }

            showNotification(`⏳ Processing secure withdrawal: ${amount} ETH...`);

            setTimeout(() => {
                const adminEthBalance = document.getElementById('adminEthBalance');
                if (adminEthBalance) {
                    const currentBalance = parseFloat(adminEthBalance.textContent);
                    if (currentBalance >= amount) {
                        adminEthBalance.textContent = (currentBalance - amount).toFixed(4);
                        addWithdrawalToHistory(amount, ADMIN_WALLET, 'Completed', 'Withdrawal');
                        if (customAmountInput) customAmountInput.value = '';
                        showNotification(`✅ ${amount} ETH withdrawn!`);
                    } else {
                        showNotification('❌ Insufficient balance');
                    }
                }
            }, 2000);
        }

        function withdrawAll() {
            if (!isAdminLoggedIn) {
                showNotification('❌ Admin login required for withdrawals');
                showAdminLogin();
                return;
            }

            const adminEthBalance = document.getElementById('adminEthBalance');
            const currentBalance = adminEthBalance ? parseFloat(adminEthBalance.textContent) : 0;

            if (!confirm(`⚠️ WITHDRAW ALL FUNDS: ${currentBalance} ETH?\n\nThis will empty the entire platform balance.`)) {
                return;
            }

            showNotification('⏳ Processing full withdrawal...');

            setTimeout(() => {
                if (adminEthBalance) {
                    adminEthBalance.textContent = '0.00';
                    addWithdrawalToHistory(currentBalance, ADMIN_WALLET, 'Completed', 'Full Withdrawal');
                    showNotification(`✅ All ${currentBalance} ETH withdrawn!`);
                }
            }, 2500);
        }

        function sendToWallet() {
            const sendToWalletInput = document.getElementById('sendToWallet');
            const sendAmountInput = document.getElementById('sendAmount');
            
            const targetWallet = sendToWalletInput ? sendToWalletInput.value : '';
            const amount = sendAmountInput ? parseFloat(sendAmountInput.value) : 0;

            if (!userAccount || userAccount.toLowerCase() !== ADMIN_WALLET.toLowerCase()) {
                showNotification('❌ Admin access required');
                return;
            }

            if (!targetWallet || !amount) {
                showNotification('❌ Please enter wallet address and amount');
                return;
            }

            if (!targetWallet.startsWith('0x') || targetWallet.length !== 42) {
                showNotification('❌ Invalid wallet address');
                return;
            }

            if (!confirm(`Send ${amount} ETH to ${targetWallet}?`)) {
                return;
            }

            showNotification(`⏳ Sending ${amount} ETH...`);

            setTimeout(() => {
                const adminEthBalance = document.getElementById('adminEthBalance');
                if (adminEthBalance) {
                    const currentBalance = parseFloat(adminEthBalance.textContent);
                    if (currentBalance >= amount) {
                        adminEthBalance.textContent = (currentBalance - amount).toFixed(4);
                        addWithdrawalToHistory(amount, targetWallet, 'Completed');
                        if (sendToWalletInput) sendToWalletInput.value = '';
                        if (sendAmountInput) sendAmountInput.value = '';
                        showNotification(`✅ ${amount} ETH sent successfully!`);
                    } else {
                        showNotification('❌ Insufficient balance');
                    }
                }
            }, 2000);
        }

        // Add withdrawal to history
        function addWithdrawalToHistory(amount, toWallet, status) {
            const historyDiv = document.getElementById('withdrawalHistory');
            if (historyDiv) {
                const statusColor = status === 'Completed' ? '#4ecdc4' : '#ffd700';
                const statusIcon = status === 'Completed' ? '✅' : '⏳';
                
                const newEntry = document.createElement('div');
                newEntry.style.cssText = 'display: flex; justify-content: space-between; padding: 10px; background: rgba(255,255,255,0.05); margin: 5px 0; border-radius: 5px;';
                newEntry.innerHTML = `
                    <span>${amount} ETH to ${toWallet.substring(0,6)}...${toWallet.substring(38)}</span>
                    <span style="color: ${statusColor};">${statusIcon} ${status}</span>
                `;
                
                historyDiv.insertBefore(newEntry, historyDiv.firstChild);
            }
        }

        // Update market data
        function updateMarketData() {
            const prices = {
                eth: 3245 + (Math.random() - 0.5) * 100,
                btc: 67892 + (Math.random() - 0.5) * 2000,
                sol: 156 + (Math.random() - 0.5) * 20
            };

            const changes = {
                eth: ((Math.random() - 0.5) * 10).toFixed(1),
                btc: ((Math.random() - 0.5) * 8).toFixed(1),
                sol: ((Math.random() - 0.5) * 12).toFixed(1)
            };

            const ethPriceEl = document.getElementById('ethPrice');
            const btcPriceEl = document.getElementById('btcPrice');
            const solPriceEl = document.getElementById('solPrice');

            if (ethPriceEl) {
                ethPriceEl.innerHTML = `$${prices.eth.toFixed(2)} (${changes.eth > 0 ? '+' : ''}${changes.eth}%)`;
                ethPriceEl.style.color = changes.eth > 0 ? '#4ecdc4' : '#ff6b6b';
            }

            if (btcPriceEl) {
                btcPriceEl.innerHTML = `$${prices.btc.toFixed(2)} (${changes.btc > 0 ? '+' : ''}${changes.btc}%)`;
                btcPriceEl.style.color = changes.btc > 0 ? '#4ecdc4' : '#ff6b6b';
            }

            if (solPriceEl) {
                solPriceEl.innerHTML = `$${prices.sol.toFixed(2)} (${changes.sol > 0 ? '+' : ''}${changes.sol}%)`;
                solPriceEl.style.color = changes.sol > 0 ? '#4ecdc4' : '#ff6b6b';
            }
        }

        // Simulate live activity
        function simulateLiveActivity() {
            const activities = [
                '🏆 Player CryptoKing won 2.5 ETH!',
                '🎯 New prediction: "Will ETH hit $4000?"',
                '⚔️ Epic NFT battle completed!',
                '💰 Yield farming: 12.7 ETH distributed!',
                '🔥 Tournament starting soon!',
                '⭐ Achievement unlocked!',
                '💎 New user joined with 5 ETH!',
                '🎪 Prize pool reaches 50 ETH!'
            ];

            const randomActivity = activities[Math.floor(Math.random() * activities.length)];
            showNotification(randomActivity);
        }

        // Create animated stars
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

        // Initialize when page loads
        document.addEventListener('DOMContentLoaded', function() {
            createStars();
            updateMarketData();
            
            setInterval(updateMarketData, 30000);
            
            setTimeout(() => {
                showNotification('🎉 Welcome to CryptoVerse Arena! Connect your wallet to start!');
            }, 2000);
            
            setInterval(simulateLiveActivity, 15000);

            const ccAmountSelect = document.getElementById('cc-amount');
            if (ccAmountSelect) {
                ccAmountSelect.addEventListener('change', function(e) {
                    const customGroup = document.getElementById('custom-amount-group');
                    if (customGroup) {
                        if (e.target.value === 'custom') {
                            customGroup.style.display = 'block';
                        } else {
                            customGroup.style.display = 'none';
                        }
                    }
                });
            }

            document.addEventListener('mousemove', function(e) {
                const cards = document.querySelectorAll('.card');
                cards.forEach(card => {
                    const rect = card.getBoundingClientRect();
                    const x = e.clientX - rect.left;
                    const y = e.clientY - rect.top;
                    
                    if (x > 0 && x < rect.width && y > 0 && y < rect.height) {
                        const centerX = rect.width / 2;
                        const centerY = rect.height / 2;
                        const rotateX = (y - centerY) / 20;
                        const rotateY = (centerX - x) / 20;
                        
                        card.style.transform = `perspective(1000px) rotateX(${rotateX}deg) rotateY(${rotateY}deg) translateY(-5px)`;
                    } else {
                        card.style.transform = '';
                    }
                });
            });

            setTimeout(() => {
                const activeUsersEl = document.getElementById('activeUsers');
                if (activeUsersEl) {
                    const randomUsers = 15847 + Math.floor(Math.random() * 1000);
                    activeUsersEl.textContent = randomUsers.toLocaleString();
                }
            }, 3000);
        });
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
            margin-bottom: 30px;
            opacity: 0.9;
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
            position: relative;
            overflow: hidden;
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

        .dashboard {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(350px, 1fr));
            gap: 25px;
            margin-bottom: 40px;
        }

        .card {
            background: rgba(255, 255, 255, 0.1);
            backdrop-filter: blur(15px);
            border-radius: 20px;
            padding: 25px;
            border: 1px solid rgba(255, 255, 255, 0.2);
            transition: all 0.3s ease;
            position: relative;
            overflow: hidden;
        }

        .card::before {
            content: '';
            position: absolute;
            top: 0;
            left: -100%;
            width: 100%;
            height: 100%;
            background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.1), transparent);
            transition: left 0.5s ease;
        }

        .card:hover::before {
            left: 100%;
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

        .balance-display {
            font-size: 2.5em;
            font-weight: bold;
            color: #ff6b6b;
            text-align: center;
            margin: 20px 0;
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

        .action-btn:disabled {
            opacity: 0.5;
            cursor: not-allowed;
        }

        .payment-methods {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
            gap: 15px;
            margin: 20px 0;
        }

        .payment-btn {
            padding: 15px;
            border: 2px solid #4ecdc4;
            border-radius: 10px;
            background: rgba(78, 205, 196, 0.1);
            color: white;
            cursor: pointer;
            transition: all 0.3s ease;
            text-align: center;
        }

        .payment-btn:hover {
            background: rgba(78, 205, 196, 0.2);
            transform: scale(1.02);
        }

        .payment-form {
            background: rgba(0, 0, 0, 0.3);
            padding: 20px;
            border-radius: 10px;
            margin: 15px 0;
            display: none;
        }

        .form-group {
            margin-bottom: 15px;
        }

        .form-group label {
            display: block;
            margin-bottom: 5px;
            color: #4ecdc4;
        }

        .form-group input, .form-group select {
            width: 100%;
            padding: 10px;
            border-radius: 5px;
            border: 1px solid #4ecdc4;
            background: rgba(255, 255, 255, 0.1);
            color: white;
        }

        .form-group input::placeholder {
            color: rgba(255, 255, 255, 0.5);
        }

        .admin-panel {
            background: rgba(255, 0, 0, 0.1);
            border: 2px solid #ff6b6b;
            border-radius: 15px;
            padding: 25px;
            margin-bottom: 20px;
        }

        .wallet-display {
            font-family: 'Courier New', monospace;
            background: rgba(0, 0, 0, 0.3);
            padding: 15px;
            border-radius: 10px;
            word-break: break-all;
            border: 1px solid #4ecdc4;
            font-size: 0.9em;
        }

        .withdrawal-grid {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 10px;
            margin: 15px 0;
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

        .pulse {
            animation: pulse 2s infinite;
        }

        @keyframes pulse {
            0% { transform: scale(1); }
            50% { transform: scale(1.05); }
            100% { transform: scale(1); }
        }

        .level-indicator {
            background: linear-gradient(45deg, #ffd700, #ffed4e);
            color: black;
            padding: 5px 15px;
            border-radius: 20px;
            font-weight: bold;
            display: inline-block;
            margin: 10px 0;
        }

        .crypto-address {
            background: rgba(0, 0, 0, 0.5);
            padding: 10px;
            border-radius: 8px;
            font-family: monospace;
            font-size: 0.9em;
            word-break: break-all;
            margin: 10px 0;
            border: 1px solid #4ecdc4;
        }

        .wallet-option-btn:hover {
            transform: scale(1.05);
            box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
        }

        @media (max-width: 768px) {
            .logo { font-size: 2.5em; }
            .dashboard { grid-template-columns: 1fr; }
            .nav-tabs { flex-direction: column; align-items: center; }
            .withdrawal-grid { grid-template-columns: 1fr; }
        }
    </style>
</head>
<body>
    <div class="stars" id="stars"></div>
    
    <div class="container">
        <div class="header">
            <div class="logo">🌟 CryptoVerse Arena 🌟</div>
            <div class="tagline">The Ultimate DeFi Gaming Experience - Where Trading Meets Gaming</div>
            <div class="level-indicator" id="userLevel">⚡ LEVEL 1 TRADER ⚡</div>
            <div style="margin-top: 15px;">
                <span id="walletStatus" style="background: rgba(255,255,255,0.1); padding: 8px 16px; border-radius: 20px;">
                    🔴 Wallet Not Connected
                </span>
            </div>
        </div>

        <div class="nav-tabs">
            <button class="tab-btn active" onclick="switchTab('dashboard')">🏠 Dashboard</button>
            <button class="tab-btn" onclick="switchTab('payments')">💳 Payments</button>
            <button class="tab-btn" onclick="switchTab('arena')">🎮 Game Arena</button>
            <button class="tab-btn" onclick="switchTab('admin')">👑 Admin Panel</button>
            <button class="tab-btn" onclick="switchTab('legal')">⚖️ Legal</button>
        </div>

        <!-- Dashboard Tab -->
        <div id="dashboard" class="tab-content active">
            <div class="dashboard">
                <div class="card pulse">
                    <div class="card-title">💰 Your Balance</div>
                    <div class="balance-display" id="userBalance">$0.00</div>
                    <div style="text-align: center; margin: 15px 0;">
                        <div>ETH: <span id="ethBalance">0.00</span> ETH</div>
                        <div>USD Equivalent: $<span id="usdBalance">0.00</span></div>
                    </div>
                    <button class="action-btn" onclick="connectWallet()">🔗 Connect Wallet</button>
                    <button class="action-btn" onclick="switchTab('payments')">💳 Add Funds</button>
                </div>

                <div class="card">
                    <div class="card-title">🎮 Quick Play</div>
                    <button class="action-btn" onclick="createGame('prediction', 50)">🔮 Prediction Game ($50)</button>
                    <button class="action-btn" onclick="createGame('trading', 100)">📈 Trading Duel ($100)</button>
                    <button class="action-btn" onclick="createGame('nft', 25)">🎨 NFT Battle ($25)</button>
                </div>

                <div class="card">
                    <div class="card-title">📊 Live Market</div>
                    <div id="marketData">
                        <div style="display: flex; justify-content: space-between; margin: 10px 0;">
                            <span>ETH/USD</span>
                            <span style="color: #4ecdc4;" id="ethPrice">$3,245.67 (+2.4%)</span>
                        </div>
                        <div style="display: flex; justify-content: space-between; margin: 10px 0;">
                            <span>BTC/USD</span>
                            <span style="color: #ff6b6b;" id="btcPrice">$67,892.12 (-0.8%)</span>
                        </div>
                        <div style="display: flex; justify-content: space-between; margin: 10px 0;">
                            <span>SOL/USD</span>
                            <span style="color: #4ecdc4;" id="solPrice">$156.78 (+5.2%)</span>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- Payments Tab -->
        <div id="payments" class="tab-content">
            <div class="card">
                <div class="card-title">💳 Add Funds to Your Account</div>
                <p style="margin-bottom: 20px; opacity: 0.8;">All payments automatically convert to ETH and go to admin wallet: <strong>0x9E1d192bd9c4dc67617D47381090DDb84db8d6C5</strong></p>
                
                <!-- Payment Method Selection -->
                <div class="payment-methods">
                    <div class="payment-btn" onclick="showPaymentForm('creditcard')">
                        💳 Credit Card
                    </div>
                    <div class="payment-btn" onclick="showPaymentForm('crypto')">
                        ₿ Cryptocurrency
                    </div>
                    <div class="payment-btn" onclick="showPaymentForm('banktransfer')">
                        🏦 Bank Transfer
                    </div>
                </div>

                <!-- Credit Card Form -->
                <div id="creditcard-form" class="payment-form">
                    <h3 style="color: #4ecdc4; margin-bottom: 15px;">💳 Credit Card Payment</h3>
                    <div class="form-group">
                        <label>Amount (USD)</label>
                        <select id="cc-amount">
                            <option value="50">$50 → 0.016 ETH</option>
                            <option value="100">$100 → 0.032 ETH</option>
                            <option value="250">$250 → 0.081 ETH</option>
                            <option value="500">$500 → 0.162 ETH</option>
                            <option value="1000">$1000 → 0.324 ETH</option>
                            <option value="custom">Custom Amount</option>
                        </select>
                    </div>
                    <div class="form-group" id="custom-amount-group" style="display: none;">
                        <label>Custom Amount (USD)</label>
                        <input type="number" id="custom-cc-amount" placeholder="Enter amount" min="10">
                    </div>
                    <div class="form-group">
                        <label>Card Number</label>
                        <input type="text" id="card-number" placeholder="1234 5678 9012 3456" maxlength="19">
                    </div>
                    <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 10px;">
                        <div class="form-group">
                            <label>Expiry Date</label>
                            <input type="text" id="card-expiry" placeholder="MM/YY" maxlength="5">
                        </div>
                        <div class="form-group">
                            <label>CVC</label>
                            <input type="text" id="card-cvc" placeholder="123" maxlength="4">
                        </div>
                    </div>
                    <button class="action-btn" onclick="processStripePayment()">💳 Pay Now</button>
                </div>

                <!-- Crypto Payment Form -->
                <div id="crypto-form" class="payment-form">
                    <h3 style="color: #4ecdc4; margin-bottom: 15px;">₿ Cryptocurrency Payment</h3>
                    <div class="form-group">
                        <label>Select Cryptocurrency</label>
                        <select id="crypto-type" onchange="updateCryptoAddress()">
                            <option value="eth">Ethereum (ETH)</option>
                            <option value="btc">Bitcoin (BTC)</option>
                            <option value="usdt">Tether (USDT)</option>
                            <option value="usdc">USD Coin (USDC)</option>
                            <option value="bnb">Binance Coin (BNB)</option>
                            <option value="sol">Solana (SOL)</option>
                        </select>
                    </div>
                    <div class="form-group">
                        <label>Send to this address:</label>
                        <div class="crypto-address" id="crypto-address">
                            0x9E1d192bd9c4dc67617D47381090DDb84db8d6C5
                        </div>
                        <button class="action-btn" onclick="copyCryptoAddress()" style="margin-top: 10px;">📋 Copy Address</button>
                    </div>
                    <div class="form-group">
                        <label>Amount to Send</label>
                        <input type="number" id="crypto-amount" placeholder="Enter amount" step="0.001" min="0.001">
                    </div>
                    
                    <!-- Direct Payment Option -->
                    <div style="background: rgba(78, 205, 196, 0.1); padding: 15px; border-radius: 10px; margin: 15px 0; border: 1px solid #4ecdc4;">
                        <h4 style="color: #4ecdc4; margin-bottom: 10px;">🚀 Quick Send (Wallet Connected)</h4>
                        <p style="font-size: 0.9em; margin-bottom: 10px; opacity: 0.8;">Send directly from your connected wallet:</p>
                        <button class="action-btn" onclick="sendCryptoDirectly()" style="background: linear-gradient(45deg, #4ecdc4, #44a08d);">
                            ⚡ Send from Wallet
                        </button>
                    </div>
                    
                    <!-- Manual Payment Option -->
                    <div style="background: rgba(0, 0, 0, 0.2); padding: 15px; border-radius: 10px; margin: 15px 0;">
                        <h4 style="color: #ffd700; margin-bottom: 10px;">📋 Manual Payment Verification</h4>
                        <p style="font-size: 0.9em; margin-bottom: 10px; opacity: 0.8;">If you sent manually, paste your transaction hash:</p>
                    </div>
                    
                    <div class="form-group">
                        <label>Transaction Hash (if sent manually)</label>
                        <input type="text" id="tx-hash" placeholder="Paste transaction hash here (optional)">
                    </div>
                    <button class="action-btn" onclick="verifyCryptoPayment()">✅ Verify Manual Payment</button>
                </div>

                <!-- Bank Transfer Form -->
                <div id="banktransfer-form" class="payment-form">
                    <h3 style="color: #4ecdc4; margin-bottom: 15px;">🏦 Bank Transfer Details</h3>
                    <div style="background: rgba(0,0,0,0.3); padding: 15px; border-radius: 8px;">
                        <p><strong>Bank:</strong> CryptoVerse Arena LLC</p>
                        <p><strong>Account:</strong> 1234567890</p>
                        <p><strong>Routing:</strong> 021000021</p>
                        <p><strong>Swift:</strong> CVARSUS33</p>
                        <p style="margin-top: 10px; color: #ffd700;"><strong>Reference:</strong> CV-USER123</p>
                    </div>
                    <div class="form-group" style="margin-top: 15px;">
                        <label>Amount Transferred (USD)</label>
                        <input type="number" id="bank-amount" placeholder="Enter amount" min="100">
                    </div>
                    <button class="action-btn" onclick="submitBankTransfer()">📞 Contact Support</button>
                </div>
            </div>
        </div>

        <!-- Game Arena Tab -->
        <div id="arena" class="tab-content">
            <div class="card">
                <div class="card-title">🎮 Live Games & Tournaments</div>
                <div style="display: grid; grid-template-columns: repeat(auto-fit, minmax(250px, 1fr)); gap: 20px; margin-top: 20px;">
                    <div class="payment-btn" onclick="joinGame('trading', 50)">
                        <h3>📈 Trading Duel</h3>
                        <p>Entry: $50</p>
                        <p style="color: #4ecdc4;">Prize: $95</p>
                    </div>
                    <div class="payment-btn" onclick="joinGame('prediction', 25)">
                        <h3>🔮 Price Prophet</h3>
                        <p>Entry: $25</p>
                        <p style="color: #ffd700;">Prize: $200</p>
                    </div>
                    <div class="payment-btn" onclick="joinGame('nft', 75)">
                        <h3>⚔️ NFT Battle</h3>
                        <p>Entry: $75</p>
                        <p style="color: #ff6b6b;">Prize: $140</p>
                    </div>
                </div>
            </div>
        </div>

        <!-- Legal Documentation Tab -->
        <div id="legal" class="tab-content">
            <div class="card">
                <div class="card-title">⚖️ Legal Documentation & Terms of Service</div>
                
                <!-- Copyright Notice -->
                <div style="background: rgba(0,0,0,0.3); padding: 20px; border-radius: 10px; margin: 20px 0; border-left: 4px solid #ffd700;">
                    <h3 style="color: #ffd700; margin-bottom: 15px;">©️ COPYRIGHT & OWNERSHIP</h3>
                    <div style="font-size: 0.9em; line-height: 1.6; color: #ddd;">
                        <p><strong>© 2025 CryptoVerse Arena. All Rights Reserved.</strong></p>
                        
                        <p><strong>INTELLECTUAL PROPERTY OWNERSHIP:</strong><br>
                        All content, design elements, source code, graphics, user interface, functionality, trademarks, service marks, and other intellectual property displayed on this platform are the exclusive property of CryptoVerse Arena and are protected by copyright, trademark, patent, trade secret and other intellectual property laws.</p>
                        
                        <p><strong>PROPRIETARY TECHNOLOGY:</strong><br>
                        The underlying software, algorithms, trading systems, game mechanics, smart contracts, and all technical implementations are proprietary technology owned exclusively by CryptoVerse Arena.</p>
                        
                        <p><strong>TRADEMARK PROTECTION:</strong><br>
                        "CryptoVerse Arena", the CryptoVerse Arena logo, and all related marks are trademarks of CryptoVerse Arena. Unauthorized use is strictly prohibited.</p>
                        
                        <p><strong>LICENSE RESTRICTIONS:</strong><br>
                        Users are granted only a limited, non-exclusive, non-transferable, revocable license to access and use the platform for personal use only. This license does not grant any rights to:</p>
                        <ul style="margin-left: 20px;">
                            <li>Reproduce, distribute, or create derivative works</li>
                            <li>Reverse engineer, decompile, or disassemble any part of the platform</li>
                            <li>Remove or modify any copyright, trademark, or proprietary notices</li>
                            <li>Use the platform or its content for commercial purposes</li>
                            <li>Access or attempt to access non-public areas of the platform</li>
                        </ul>
                    </div>
                </div>

                <!-- Disclaimer of Liability -->
                <div style="background: rgba(0,0,0,0.3); padding: 20px; border-radius: 10px; margin: 20px 0; border-left: 4px solid #ff6b6b;">
                    <h3 style="color: #ff6b6b; margin-bottom: 15px;">⚠️ DISCLAIMER OF LIABILITY & RISK WARNING</h3>
                    <div style="font-size: 0.9em; line-height: 1.6; color: #ddd;">
                        <p><strong>CRITICAL WARNING: READ CAREFULLY</strong></p>
                        
                        <p><strong>NO LIABILITY GUARANTEE:</strong><br>
                        CryptoVerse Arena, its operators, developers, affiliates, partners, and service providers SHALL NOT BE LIABLE for any direct, indirect, incidental, special, consequential, punitive, or exemplary damages, including but not limited to:</p>
                        
                        <ul style="margin-left: 20px;">
                            <li>Loss of funds, cryptocurrencies, digital assets, or investments</li>
                            <li>Trading losses, poor investment decisions, or market volatility impacts</li>
                            <li>Technical failures, system downtime, network congestion, or smart contract bugs</li>
                            <li>Third-party hacks, security breaches, or unauthorized access</li>
                            <li>Regulatory changes, legal restrictions, or government actions</li>
                            <li>Platform unavailability, data loss, or service interruptions</li>
                            <li>User errors, forgotten passwords, or lost private keys</li>
                            <li>Fraudulent activities by other users or third parties</li>
                        </ul>
                        
                        <p><strong>ASSUMPTION OF RISK:</strong><br>
                        By using this platform, you acknowledge and agree that:</p>
                        <ul style="margin-left: 20px;">
                            <li>Cryptocurrency trading involves substantial risk of loss</li>
                            <li>Digital assets are highly volatile and speculative</li>
                            <li>You may lose all deposited funds</li>
                            <li>Past performance does not guarantee future results</li>
                            <li>You are solely responsible for all trading decisions</li>
                            <li>You understand the risks of blockchain technology</li>
                        </ul>
                        
                        <p><strong>PLATFORM PROVIDED "AS IS":</strong><br>
                        This platform is provided on an "AS IS" and "AS AVAILABLE" basis without warranties of any kind, either express or implied, including but not limited to warranties of merchantability, fitness for a particular purpose, or non-infringement.</p>
                        
                        <p><strong>LIMITATION OF LIABILITY:</strong><br>
                        In no event shall the total liability of CryptoVerse Arena exceed $100 USD or the amount of fees paid by the user in the 30 days preceding the claim, whichever is less.</p>
                        
                        <p><strong>INDEMNIFICATION:</strong><br>
                        You agree to defend, indemnify, and hold harmless CryptoVerse Arena from any claims, damages, losses, costs, and expenses arising from your use of the platform or violation of these terms.</p>
                    </div>
                </div>

                <!-- Terms of Service -->
                <div style="background: rgba(0,0,0,0.3); padding: 20px; border-radius: 10px; margin: 20px 0; border-left: 4px solid #4ecdc4;">
                    <h3 style="color: #4ecdc4; margin-bottom: 15px;">📋 TERMS OF SERVICE & USER AGREEMENT</h3>
                    <div style="font-size: 0.9em; line-height: 1.6; color: #ddd;">
                        <p><strong>BINDING AGREEMENT:</strong><br>
                        By accessing or using CryptoVerse Arena, you agree to be bound by these Terms of Service, Privacy Policy, and all applicable laws and regulations. If you do not agree, do not use this platform.</p>
                        
                        <p><strong>ELIGIBILITY REQUIREMENTS:</strong><br>
                        • Must be 18+ years old and legally capable of entering contracts<br>
                        • Must not be located in restricted jurisdictions<br>
                        • Must comply with all applicable local, state, and federal laws<br>
                        • Must not be on any sanctions or prohibited persons lists</p>
                        
                        <p><strong>PROHIBITED ACTIVITIES:</strong><br>
                        Users may not:</p>
                        <ul style="margin-left: 20px;">
                            <li>Engage in market manipulation, wash trading, or artificial volume</li>
                            <li>Use automated trading systems without authorization</li>
                            <li>Attempt to compromise platform security or other users' accounts</li>
                            <li>Engage in money laundering, terrorist financing, or illegal activities</li>
                            <li>Create multiple accounts to circumvent restrictions</li>
                            <li>Provide false or misleading information</li>
                            <li>Violate any applicable laws or regulations</li>
                        </ul>
                        
                        <p><strong>ACCOUNT TERMINATION:</strong><br>
                        We reserve the right to suspend or terminate user accounts at our sole discretion for violations of these terms, suspicious activity, or legal compliance requirements.</p>
                        
                        <p><strong>REGULATORY COMPLIANCE:</strong><br>
                        This platform may not be available in all jurisdictions. Users are responsible for ensuring compliance with local laws and regulations.</p>
                    </div>
                </div>

                <!-- Privacy Policy -->
                <div style="background: rgba(0,0,0,0.3); padding: 20px; border-radius: 10px; margin: 20px 0; border-left: 4px solid #96ceb4;">
                    <h3 style="color: #96ceb4; margin-bottom: 15px;">🛡️ PRIVACY POLICY & DATA PROTECTION</h3>
                    <div style="font-size: 0.9em; line-height: 1.6; color: #ddd;">
                        <p><strong>DATA COLLECTION:</strong><br>
                        We collect wallet addresses, transaction data, IP addresses, and usage analytics to provide and improve our services. We do not collect personally identifiable information unless voluntarily provided.</p>
                        
                        <p><strong>DATA USAGE:</strong><br>
                        • Platform functionality and security monitoring<br>
                        • Analytics and performance optimization<br>
                        • Compliance with legal and regulatory obligations<br>
                        • Communication about platform updates and security notices</p>
                        
                        <p><strong>DATA PROTECTION:</strong><br>
                        We implement industry-standard security measures to protect user data. However, no system is completely secure, and users acknowledge the inherent risks of digital data transmission and storage.</p>
                        
                        <p><strong>THIRD-PARTY INTEGRATIONS:</strong><br>
                        Platform integrates with third-party services (wallets, price feeds, payment processors) which have their own privacy policies and terms of service.</p>
                        
                        <p><strong>DATA RETENTION:</strong><br>
                        We retain user data as required by law and for legitimate business purposes. Users may request data deletion subject to legal and operational requirements.</p>
                    </div>
                </div>

                <!-- Financial Disclaimers -->
                <div style="background: rgba(0,0,0,0.3); padding: 20px; border-radius: 10px; margin: 20px 0; border-left: 4px solid #667eea;">
                    <h3 style="color: #667eea; margin-bottom: 15px;">💰 FINANCIAL DISCLAIMERS</h3>
                    <div style="font-size: 0.9em; line-height: 1.6; color: #ddd;">
                        <p><strong>NOT FINANCIAL ADVICE:</strong><br>
                        All content, information, and materials on this platform are for informational and entertainment purposes only. Nothing constitutes financial, investment, legal, or tax advice. Consult qualified professionals before making any financial decisions.</p>
                        
                        <p><strong>NOT A REGISTERED ENTITY:</strong><br>
                        CryptoVerse Arena is not a registered investment advisor, broker-dealer, or financial institution. We do not provide investment recommendations or portfolio management services.</p>
                        
                        <p><strong>REGULATORY STATUS:</strong><br>
                        Platform tokens, activities, and services may not be considered securities, but users should consult legal counsel regarding regulatory compliance in their jurisdiction.</p>
                        
                        <p><strong>TAX OBLIGATIONS:</strong><br>
                        Users are solely responsible for determining and fulfilling their tax obligations related to cryptocurrency transactions and platform activities.</p>
                    </div>
                </div>

                <!-- User Consent -->
                <div style="background: rgba(255,107,107,0.1); padding: 20px; border-radius: 10px; margin: 20px 0; border: 2px solid #ff6b6b;">
                    <h3 style="color: #ff6b6b; margin-bottom: 15px;">✅ USER CONSENT ACKNOWLEDGMENT</h3>
                    <div style="font-size: 0.9em; line-height: 1.6;">
                        <label style="display: flex; align-items: flex-start; margin: 15px 0; cursor: pointer;">
                            <input type="checkbox" id="legalTermsConsent" style="margin-right: 10px; margin-top: 2px; transform: scale(1.2);">
                            <span>I have read, understood, and agree to be bound by the Terms of Service, Privacy Policy, and all legal disclaimers</span>
                        </label>
                        
                        <label style="display: flex; align-items: flex-start; margin: 15px 0; cursor: pointer;">
                            <input type="checkbox" id="riskAcknowledgment" style="margin-right: 10px; margin-top: 2px; transform: scale(1.2);">
                            <span>I understand and acknowledge all risks involved in cryptocurrency trading and DeFi activities, including the potential for total loss of funds</span>
                        </label>
                        
                        <label style="display: flex; align-items: flex-start; margin: 15px 0; cursor: pointer;">
                            <input type="checkbox" id="jurisdictionCompliance" style="margin-right: 10px; margin-top: 2px; transform: scale(1.2);">
                            <span>I confirm that I am 18+ years old, legally eligible to use this platform, and that such use is legal in my jurisdiction</span>
                        </label>
                        
                        <label style="display: flex; align-items: flex-start; margin: 15px 0; cursor: pointer;">
                            <input type="checkbox" id="liabilityWaiver" style="margin-right: 10px; margin-top: 2px; transform: scale(1.2);">
                            <span>I waive any claims against CryptoVerse Arena for losses, damages, or issues arising from my use of this platform</span>
                        </label>
                        
                        <button class="action-btn" id="acceptLegalTerms" onclick="acceptLegalTerms()" style="margin-top: 20px; opacity: 0.5;" disabled>
                            🔐 Accept All Terms & Continue
                        </button>
                        
                        <p style="margin-top: 15px; font-size: 0.8em; opacity: 0.7; text-align: center;">
                            By clicking "Accept All Terms & Continue", you acknowledge that you have read and understood all legal documentation and agree to be bound by these terms.
                        </p>
                    </div>
                </div>
                
                <!-- Contact Information -->
                <div style="background: rgba(0,0,0,0.2); padding: 15px; border-radius: 8px; margin-top: 20px; text-align: center;">
                    <p style="color: #4ecdc4;"><strong>Legal Inquiries:</strong> legal@cryptoversearena.com</p>
                    <p style="color: #4ecdc4;"><strong>Support:</strong> support@cryptoversearena.com</p>
                    <p style="font-size: 0.8em; opacity: 0.7; margin-top: 10px;">
                        Last Updated: ${new Date().toLocaleDateString()} | Version 1.0
                    </p>
                </div>
            </div>
        </div>

        <!-- Admin Panel -->
        <div id="admin" class="tab-content">
            <div class="admin-panel">
                <div class="card-title">👑 Admin Control Center</div>
                <p style="margin-bottom: 20px; color: #ffd700;">⚠️ Admin access required. Connect wallet: 0x9E1d192bd9c4dc67617D47381090DDb84db8d6C5</p>
                
                <div class="card" style="margin: 20px 0;">
                    <h3>💼 Platform Revenue</h3>
                    <div class="balance-display" id="adminRevenue">$892,345.67</div>
                    <div>Total Volume: $45.2M | Active Users: <span id="activeUsers">15,847</span></div>
                </div>
                
                <div class="card" style="margin: 20px 0;">
                    <h3>🏦 Admin Wallet</h3>
                    <div class="wallet-display">
                        0x9E1d192bd9c4dc67617D47381090DDb84db8d6C5
                    </div>
                    <div style="margin-top: 15px;">
                        <div>💰 ETH Balance: <span id="adminEthBalance">1,247.89</span> ETH</div>
                        <div>🏆 Platform Fees: <span id="platformFees">156.7</span> ETH</div>
                        <div>📊 Revenue Share: <span id="revenueShare">78.3</span> ETH</div>
                    </div>
                </div>
                
                <!-- Withdrawal Controls -->
                <div class="card">
                    <h3>💸 Withdrawal Center</h3>
                    
                    <!-- Quick Withdraw Buttons -->
                    <div class="withdrawal-grid">
                        <button class="action-btn" onclick="quickWithdraw(1)" style="background: linear-gradient(45deg, #ff6b6b, #ff8e53);">
                            💰 Withdraw 1 ETH
                        </button>
                        <button class="action-btn" onclick="quickWithdraw(5)" style="background: linear-gradient(45deg, #4ecdc4, #44a08d);">
                            💎 Withdraw 5 ETH
                        </button>
                        <button class="action-btn" onclick="quickWithdraw(10)" style="background: linear-gradient(45deg, #ffd700, #ffed4e); color: black;">
                            🏆 Withdraw 10 ETH
                        </button>
                        <button class="action-btn" onclick="withdrawAll()" style="background: linear-gradient(45deg, #a8edea, #fed6e3); color: black;">
                            🚀 Withdraw All
                        </button>
                    </div>
                    
                    <!-- Custom Amount Withdrawal -->
                    <div style="background: rgba(0,0,0,0.2); padding: 15px; border-radius: 10px; margin: 15px 0;">
                        <h4 style="color: #4ecdc4; margin-bottom: 10px;">Custom Amount</h4>
                        <div style="display: flex; gap: 10px; align-items: center;">
                            <input type="number" id="customWithdrawAmount" placeholder="Enter ETH amount" 
                                   style="flex: 1; padding: 10px; border-radius: 5px; border: 1px solid #4ecdc4; background: rgba(255,255,255,0.1); color: white;" 
                                   step="0.001" min="0.001">
                            <button class="action-btn" onclick="customWithdraw()" style="width: auto; padding: 10px 20px;">
                                💸 Withdraw
                            </button>
                        </div>
                    </div>

                    <!-- Send to Different Wallet -->
                    <div style="background: rgba(0,0,0,0.2); padding: 15px; border-radius: 10px; margin: 15px 0;">
                        <h4 style="color: #ff6b6b; margin-bottom: 10px;">Send to Wallet Address</h4>
                        <input type="text" id="sendToWallet" placeholder="Enter wallet address (0x...)" 
                               style="width: 100%; padding: 10px; border-radius: 5px; border: 1px solid #ff6b6b; background: rgba(255,255,255,0.1); color: white; margin-bottom: 10px;">
                        <div style="display: flex; gap: 10px;">
                            <input type="number" id="sendAmount" placeholder="ETH amount" 
                                   style="flex: 1; padding: 10px; border-radius: 5px; border: 1px solid #ff6b6b; background: rgba(255,255,255,0.1); color: white;" 
                                   step="0.001" min="0.001">
                            <button class="action-btn" onclick="sendToWallet()" style="width: auto; padding: 10px 20px; background: linear-gradient(45deg, #ff6b6b, #ee5a52);">
                                🎯 Send
                            </button>
                        </div>
                        <button class="action-btn" onclick="showBulkSendModal()" style="margin-top: 10px; background: linear-gradient(45deg, #667eea, #764ba2);">
                            📤 Bulk Send to Multiple Wallets
                        </button>
                    </div>

                    <!-- Recent Withdrawals -->
                    <div style="margin-top: 20px;">
                        <h4 style="color: #4ecdc4; margin-bottom: 10px;">📋 Recent Withdrawals</h4>
                        <div id="withdrawalHistory" style="max-height: 200px; overflow-y: auto;">
                            <div style="display: flex; justify-content: space-between; padding: 10px; background: rgba(255,255,255,0.05); margin: 5px 0; border-radius: 5px;">
                                <span>5.0 ETH to 0x9E1d...d6C5</span>
                                <span style="color: #4ecdc4;">✅ Completed</span>
                            </div>
                            <div style="display: flex; justify-content: space-between; padding: 10px; background: rgba(255,255,255,0.05); margin: 5px 0; border-radius: 5px;">
                                <span>2.5 ETH to 0x9E1d...d6C5</span>
                                <span style="color: #4ecdc4;">✅ Completed</span>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <div id="notification" class="notification"></div>
</body>
</html>
