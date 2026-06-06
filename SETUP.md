# SwiftZone Hotspot Backend — Setup Guide

## What This Does
Fully automates WiFi voucher sales:
Customer pays M-Pesa → Voucher created on router → SMS sent to customer → Owner notified

Zero manual work for the owner.

---

## What You Need Before Starting

1. **Safaricom Daraja API account** — developer.safaricom.co.ke (free)
2. **Africa's Talking account** — africastalking.com (free sandbox for testing)
3. **MikroTik router** with API enabled
4. **A server** — Truehost VPS (Ksh 500/month) or any Node.js host
5. **A domain/subdomain** — e.g. api.swiftzone.co.ke (for M-Pesa callback URL)

---

## Step 1: Enable MikroTik API

On the router (via Winbox or SSH):
```
/ip service enable api
/ip service set api port=8728
```

Also create hotspot user profiles matching your bundle names:
```
/ip hotspot user profile add name=1hr rate-limit=5M/5M session-timeout=1h
/ip hotspot user profile add name=daily rate-limit=10M/10M session-timeout=24h
/ip hotspot user profile add name=weekly rate-limit=15M/15M session-timeout=168h
/ip hotspot user profile add name=monthly rate-limit=20M/20M session-timeout=720h
```

---

## Step 2: Get Daraja API Credentials

1. Go to developer.safaricom.co.ke
2. Create an app → get Consumer Key & Consumer Secret
3. Go Live → get your Shortcode & Passkey
4. Set Callback URL to: https://yourserver.com/api/mpesa-callback

---

## Step 3: Get Africa's Talking Credentials

1. Sign up at africastalking.com
2. Create an app → copy API Key & Username
3. For testing: use username = "sandbox"
4. For live: register a Sender ID (e.g. "SwiftZone")

---

## Step 4: Deploy to Truehost

```bash
# SSH into your Truehost VPS
ssh root@your-server-ip

# Install Node.js
curl -fsSL https://deb.nodesource.com/setup_18.x | bash -
apt-get install -y nodejs

# Upload your files (via FileZilla or scp)
# Then:
cd /var/www/swiftzone-backend
npm install

# Copy and fill in environment variables
cp .env.example .env
nano .env   # Fill in all values

# Start with PM2 (keeps it running forever)
npm install -g pm2
pm2 start server.js --name swiftzone
pm2 startup
pm2 save
```

---

## Step 5: Connect Frontend to Backend

In your hotspot-login-portal.html, update the pay button to call:

```javascript
// Replace the buyBundle() simulation with:
async function buyBundle() {
  const phone = document.getElementById('buyPhone').value.trim();
  const response = await fetch('https://yourserver.com/api/pay', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ phone, bundle: selectedBundle.id })
  });
  const data = await response.json();
  if (data.success) {
    // Show "Check your phone for M-Pesa prompt"
    showMpesaWaiting(data.checkoutRequestID);
  }
}
```

---

## Pricing This to Your Client

| Component              | Your Cost        |
|------------------------|-----------------|
| Truehost VPS hosting   | Ksh 500/month   |
| Africa's Talking SMS   | ~Ksh 1/SMS      |
| Daraja API             | Free            |
| MikroTik router        | Client buys it  |

**What to charge:**
- Setup fee: Ksh 25,000 – 40,000 (one time)
- Monthly maintenance: Ksh 2,000 – 3,000/month
- This is fully passive income for the owner once set up

---

## Testing Checklist

- [ ] MikroTik API connects successfully
- [ ] STK push appears on test phone
- [ ] Callback received after payment
- [ ] Voucher appears in MikroTik users list
- [ ] SMS received with voucher code
- [ ] Owner notification SMS received
- [ ] Voucher works on login portal
