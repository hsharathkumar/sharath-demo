# sharath-demo
This is my first git repository
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title> PAYMENT METHODS </title>
    <link rel="stylesheet" href="2style.css">
</head>
<body>

<div class="card">
    <h2>Selected payment method</h2>

        <label class="payment-option">
        <input type="radio" name="payment"checked>
        <div class="content">
            <span class="icon">💳</span>
            <div>
                <h4>Credit /Debits</h4>
                <p>Visa, MasterCard</p>
            </div>
            </span>
        </div>
    </label>

        <label class="payment-option">
        <input type="radio" name="payment"checked>
        <div class="content">
            <span class="icon">🏛️</span>
            <div>
                <h4>Net Banking</h4>
                <p>All major Banks</p>
            </div>
            </span>
        </div>
    </label>

        <label class="payment-option">
        <input type="radio" name="payment"checked>
        <div class="content">
            <span class="icon">📱</span>
            <div>
                <h4>UPI</h4>
                <p>Google pay, Phone pay </p>
            </div>
            </span>
        </div>
    </label>
<button class="btn" type="submit">continue</button>
    <script type="text/javascript" src="script.js"></script>
    
</div>
</body>
</html>
° {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
    font-family: Arial, sans-serif;

}

body{
    height: 100vh;
    background: linear-gradient(135deg, #020617,#0f172a);
    display: flex;
    justify-content: center;
    align-items: center;

}
.card {
    width: 360px;
    background: #020617;
    padding: 25px;
    border-radius: 18px;
    color: #fff;
    box-shadow: 0 25px 50px rgba(0, 0, 0, 0.6);
    transition: 0.3s;
}
h2,h4,p{
    color:#fff
}

.card:hover{
    transform: translateY(-12px);
    box-shadow: 0 0 10px rgb(14, 255, 102),
    0 0 20px rgb(180, 14, 141);

}

.card h2 {
    text-align: center;
    margin-bottom: 20px;

}
.payment-option{
    display: block;
    background: #000000;
    padding: 15px;
    border-radius: 14px;
    margin-bottom: 15px;
    cursor: pointer;
    transition: 0.3s;
    border: 2px solid transparent;

}
.payment-option:hover{
    transform: translateY(-5px);
    box-shadow: 0 0 10px rgb(135, 94, 197),
    0 0 20px rgba(34, 197, 94);
}
.payment-option input{
    display: none;
}
.payment-option input:checked + .content {
    border-left: 4px solid #22c55e;
}
.content{
    display: flex;
    align-items: center;
    gap: 35px;
    padding-left: 10px;
}
.icon {
    font-size: 35px;
}

.content p{
    font-size: 15px;
    color: #9ca3af;
}
.btn {
    width: 100%;
    padding: 14px;
    margin-top: 10px;
    border: none;
    border-radius: 30px;
    background: #22c55e;
    color: #000;
    font-weight: bold;
    transition: 0.3s;
}

.btn:hover {
    background: #15d35b;
    transform: translateY(-10px);
    box-shadow: 0 0 5px rgba(34, 197, 94), 0 0 10px rgb(8, 5, 185);
}
/**
 * payment-method.js
 *
 * Client-side JavaScript to handle a simple "payment method" UI and submission flow.
 * Intended to be used with an HTML model like "2index.html" that provides:
 *
 * - A <form id="payment-form">
 * - Radio buttons or inputs with name="paymentMethod" and values: "card", "paypal", "bank"
 * - A container #card-section with inputs: #card-number, #card-expiry, #card-cvc, #card-name
 * - A container #paypal-section (could hold a button or info)
 * - A container #bank-section with inputs: #bank-account, #bank-routing
 * - A checkbox #save-card (optional)
 * - A submit button #pay-button
 * - A status element #payment-status
 *
 * This file:
 * - Shows/hides method-specific sections
 * - Validates basic card and bank fields
 * - Performs a mock tokenization for card data (replace with real gateway tokenization)
 * - Submits a JSON payload to /api/payments (example endpoint) via fetch
 * - Handles UI state and error display
 *
 * Note: For production, replace tokenization with your payment provider SDK (Stripe/PayPal/etc.),
 * use HTTPS, and never send raw card data to your server.
 */

(function () {
    'use strict';

    // ---- Utilities ----
    const $ = (sel, ctx = document) => ctx.querySelector(sel);
    const $$ = (sel, ctx = document) => Array.from(ctx.querySelectorAll(sel));

    function show(el) { if (!el) return; el.style.display = ''; }
    function hide(el) { if (!el) return; el.style.display = 'none'; }
    function setText(el, text) { if (!el) return; el.textContent = text; }

    // Basic Luhn check for card number (not exhaustive)
    function luhnCheck(number) {
        const digits = number.replace(/\D/g, '').split('').reverse().map(Number);
        let sum = 0;
        for (let i = 0; i < digits.length; i++) {
            let d = digits[i];
            if (i % 2 === 1) {
                d *= 2;
                if (d > 9) d -= 9;
            }
            sum += d;
        }
        return sum % 10 === 0;
    }

    // Simple expiry check MM/YY or MM/YYYY
    function validateExpiry(raw) {
        const m = raw.match(/^\s*(\d{1,2})\s*\/\s*(\d{2,4})\s*$/);
        if (!m) return false;
        let month = parseInt(m[1], 10);
        let year = parseInt(m[2], 10);
        if (year < 100) year += 2000;
        if (month < 1 || month > 12) return false;
        const now = new Date();
        const expiry = new Date(year, month - 1, 1);
        // set to last day of month for safe compare
        const lastDayOfMonth = new Date(expiry.getFullYear(), expiry.getMonth() + 1, 0);
        return lastDayOfMonth >= new Date(now.getFullYear(), now.getMonth(), 1);
    }

    // Basic validation helpers
    function isNonEmpty(value) { return value != null && String(value).trim().length > 0; }

    // Mock tokenization - in production call PCI-compliant provider SDK
    async function tokenizeCard({ number, expiry, cvc, name }) {
        // Minimal simulation of async tokenization
        await new Promise(r => setTimeout(r, 600));
        // Very small fake token (do not use in production)
        const token = 'tok_' + Math.random().toString(36).slice(2, 12);
        return { success: true, token, brand: detectCardBrand(number) };
    }

    function detectCardBrand(number) {
        const n = (number || '').replace(/\D/g, '');
        if (/^4/.test(n)) return 'visa';
        if (/^5[1-5]/.test(n) || /^2(2[2-9]|[3-6]\d|7[01])/.test(n)) return 'mastercard';
        if (/^3[47]/.test(n)) return 'amex';
        if (/^6(?:011|5)/.test(n)) return 'discover';
        return 'unknown';
    }

    // ---- Main flow ----

    document.addEventListener('DOMContentLoaded', init);

    function init() {
        const form = $('#payment-form');
        const payButton = $('#pay-button');
        const statusEl = $('#payment-status');

        const cardSection = $('#card-section');
        const paypalSection = $('#paypal-section');
        const bankSection = $('#bank-section');

        // Payment inputs
        const cardNumber = $('#card-number');
        const cardExpiry = $('#card-expiry');
        const cardCvc = $('#card-cvc');
        const cardName = $('#card-name');
        const saveCard = $('#save-card');

        const bankAccount = $('#bank-account');
        const bankRouting = $('#bank-routing');

        // Wire up method toggles
        const methodInputs = $$('input[name="paymentMethod"]');
        methodInputs.forEach(i => i.addEventListener('change', toggleMethodVisibility));
        toggleMethodVisibility(); // initial

        // Form submit
        if (form) form.addEventListener('submit', async (e) => {
            e.preventDefault();
            clearStatus();
            setBusy(true);
            try {
                const method = getSelectedMethod();
                if (method === 'card') {
                    // validate card fields
                    const num = cardNumber ? cardNumber.value.trim() : '';
                    const exp = cardExpiry ? cardExpiry.value.trim() : '';
                    const cvc = cardCvc ? cardCvc.value.trim() : '';
                    const name = cardName ? cardName.value.trim() : '';

                    const errors = [];
                    if (!isNonEmpty(num) || !/^\d[\d\s-]{7,}$/.test(num) || !luhnCheck(num)) {
                        errors.push('Enter a valid card number.');
                    }
                    if (!validateExpiry(exp)) errors.push('Enter a valid expiry date (MM/YY).');
                    if (!/^\d{3,4}$/.test(cvc)) errors.push('Enter a valid CVC.');
                    if (!isNonEmpty(name)) errors.push('Cardholder name is required.');

                    if (errors.length) {
                        setError(errors.join(' '));
                        setBusy(false);
                        return;
                    }

                    // Tokenize card (mock)
                    const tokenRes = await tokenizeCard({ number: num, expiry: exp, cvc, name });
                    if (!tokenRes.success) {
                        setError('Card tokenization failed.');
                        setBusy(false);
                        return;
                    }

                    // Build payload
                    const payload = {
                        method: 'card',
                        token: tokenRes.token,
                        brand: tokenRes.brand,
                        amount: getAmountFromForm(),
                        saveCard: !!(saveCard && saveCard.checked)
                    };

                    await submitPayment(payload);

                } else if (method === 'paypal') {
                    // For PayPal, typically open provider flow. Here we mock a server call to create order
                    const payload = {
                        method: 'paypal',
                        amount: getAmountFromForm()
                    };
                    await submitPayment(payload);

                } else if (method === 'bank') {
                    const acct = bankAccount ? bankAccount.value.trim() : '';
                    const routing = bankRouting ? bankRouting.value.trim() : '';
                    const errors = [];
                    if (!/^\d{6,20}$/.test(acct)) errors.push('Enter a valid bank account number.');
                    if (!/^\d{4,9}$/.test(routing)) errors.push('Enter a valid routing number.');
                    if (errors.length) {
                        setError(errors.join(' '));
                        setBusy(false);
                        return;
                    }
                    const payload = {
                        method: 'bank',
                        account: maskAccount(acct),
                        routing,
                        amount: getAmountFromForm()
                    };
                    await submitPayment(payload);
                } else {
                    setError('Select a payment method.');
                }
            } catch (err) {
                console.error(err);
                setError('An unexpected error occurred.');
            } finally {
                setBusy(false);
            }
        });

        function toggleMethodVisibility() {
            const method = getSelectedMethod();
            if (cardSection) (method === 'card' ? show(cardSection) : hide(cardSection));
            if (paypalSection) (method === 'paypal' ? show(paypalSection) : hide(paypalSection));
            if (bankSection) (method === 'bank' ? show(bankSection) : hide(bankSection));
            clearStatus();
        }

        function getSelectedMethod() {
            const selected = $('input[name="paymentMethod"]:checked');
            return selected ? selected.value : null;
        }

        function maskAccount(acct) {
            const s = String(acct);
            if (s.length <= 4) return '****';
            return '****' + s.slice(-4);
        }

        function getAmountFromForm() {
            // Try to read a data-amount attribute on the form or a field #amount; default to 0.00
            const aField = $('#amount');
            if (aField && aField.value) return parseFloat(aField.value) || 0;
            const formAmount = form && form.dataset && form.dataset.amount;
            if (formAmount) return parseFloat(formAmount) || 0;
            return 0;
        }

        async function submitPayment(payload) {
            setText(statusEl, 'Processing payment...');
            try {
                // Example: change URL to your server endpoint
                const res = await fetch('/api/payments', {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload),
                });
                if (!res.ok) {
                    let text = await res.text().catch(() => null);
                    throw new Error(text || `Server returned ${res.status}`);
                }
                const data = await res.json().catch(() => ({}));
                if (data && data.status === 'requires_action') {
                    // Example: handle 3D Secure or other provider actions
                    await handleAdditionalAction(data);
                } else {
                    setText(statusEl, 'Payment successful.');
                }
            } catch (err) {
                setError('Payment failed: ' + (err.message || err));
            }
        }

        async function handleAdditionalAction(actionData) {
            // This is a placeholder. Real providers will require SDK flows (e.g., stripe.handleCardAction)
            // Here we simulate user completing an action and then calling the server to confirm.
            setText(statusEl, 'Additional authentication required. Completing...');
            await new Promise(r => setTimeout(r, 1000));
            // After "action", call confirm endpoint
            const res = await fetch('/api/payments/confirm', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ paymentId: actionData.paymentId }),
            });
            if (!res.ok) throw new Error('Confirmation failed');
            setText(statusEl, 'Payment completed after authentication.');
        }

        function setBusy(flag) {
            if (payButton) payButton.disabled = !!flag;
            if (flag) {
                if (statusEl) setText(statusEl, 'Please wait...');
            }
        }

        function setError(msg) {
            if (statusEl) {
                statusEl.classList.add('error');
                setText(statusEl, msg);
            } else {
                alert(msg);
            }
        }

        function clearStatus() {
            if (!statusEl) return;
            statusEl.classList.remove('error');
            setText(statusEl, '');
        }
    }
})();
