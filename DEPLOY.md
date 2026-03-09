# MVP Kids Cards — Deployment Guide

Everything deploys to **Vercel** — both the backend API and the frontend HTML pages.
You'll have two separate Vercel projects: one for the API, one for the site.

Total time: ~20 minutes.

---

## Part 1 — Deploy the Backend API

### Step 1: Run the database schema in Supabase
1. Go to your Supabase project → **SQL Editor**
2. Open the file `backend/sql/schema.sql` from your folder
3. Paste the entire contents → click **Run**

### Step 2: Commit and push the backend to GitHub
You already initialized git. Now run in Terminal:
```bash
cd /Users/anameli/Documents/Claude/Scheduled/MVPKidsCards/backend
git add .
git commit -m "Add Vercel config"
```
Then go to **github.com → + → New repository**:
- Name it `mvpkidscards-api`
- Set to **Private**
- Leave everything else unchecked → **Create repository**

GitHub will show you two commands — run them in Terminal:
```bash
git remote add origin https://github.com/YOUR_USERNAME/mvpkidscards-api.git
git push -u origin main
```

### Step 3: Deploy backend to Vercel
1. Go to [vercel.com](https://vercel.com) → **Add New → Project**
2. Import your `mvpkidscards-api` repo
3. Framework Preset: **Other**
4. **Root Directory**: `./` (leave as default)
5. Click **Deploy** — it will build in about 30 seconds
6. Once deployed, click **Settings → Domains** — copy your API URL (e.g. `https://mvpkidscards-api.vercel.app`)

### Step 4: Add environment variables in Vercel
In your API project → **Settings → Environment Variables** → add each of these:

| Variable | Value |
|---|---|
| `SUPABASE_URL` | Your Supabase project URL |
| `SUPABASE_ANON_KEY` | Your Supabase anon key |
| `SUPABASE_SERVICE_ROLE_KEY` | Your Supabase service role key |
| `NODE_ENV` | `production` |
| `ALLOWED_ORIGINS` | `https://mvpkidscards.vercel.app` (update after Step 8) |
| `CONVERTKIT_API_KEY` | Your ConvertKit API key |
| `CONVERTKIT_FORM_ID` | `9180950` |
| `STRIPE_SECRET_KEY` | `sk_test_...` (use test key to start) |
| `STRIPE_WEBHOOK_SECRET` | From Stripe dashboard (see Step 5) |
| `APP_BASE_URL` | `https://mvpkidscards.vercel.app` (update after Step 8) |
| `ADMIN_EMAILS` | `anamelibiz@gmail.com` |

After adding variables → **Deployments → Redeploy** so they take effect.

### Step 5: Set up Stripe webhook
1. Go to [dashboard.stripe.com](https://dashboard.stripe.com) → **Developers → Webhooks → Add endpoint**
2. URL: `https://mvpkidscards-api.vercel.app/api/checkout/webhook`
3. Events to listen for:
   - `checkout.session.completed`
   - `checkout.session.expired`
   - `charge.refunded`
4. Copy the **Signing secret** → paste into Vercel env vars as `STRIPE_WEBHOOK_SECRET` → Redeploy

### Step 6: Verify the API is live
Paste this in your browser or Terminal:
```
https://mvpkidscards-api.vercel.app/health
```
You should see: `{"status":"ok","service":"MVP Kids Cards API",...}`

---

## Part 2 — Deploy the Frontend

### Step 7: Update the API URL in your HTML files
Open Terminal and run (replace the URL with your actual Vercel API URL from Step 3):
```bash
cd /Users/anameli/Documents/Claude/Scheduled/MVPKidsCards

API_URL="https://mvpkidscards-api.vercel.app"

for f in *.html; do
  sed -i '' "s|http://localhost:3001|$API_URL|g" "$f"
done

# Verify — should return nothing if all updated:
grep -rn "localhost:3001" *.html
```

### Step 8: Push frontend to GitHub
```bash
cd /Users/anameli/Documents/Claude/Scheduled/MVPKidsCards

git init
git add index.html login.html dashboard.html athlete-profile.html \
        team-signup.html team-dashboard.html join.html \
        order-success.html reset-password.html admin.html \
        vercel.json .gitignore
git commit -m "Initial MVP Kids Cards frontend"
```
Create another repo on GitHub called `mvpkidscards-web` (Private), then:
```bash
git remote add origin https://github.com/YOUR_USERNAME/mvpkidscards-web.git
git push -u origin main
```

### Step 9: Deploy frontend to Vercel
1. Vercel → **Add New → Project**
2. Import `mvpkidscards-web`
3. Framework Preset: **Other**
4. Build command: *(leave blank)*
5. Output directory: `.`
6. Click **Deploy**
7. Your site is live at something like `https://mvpkidscards-web.vercel.app`

### Step 10: Update ALLOWED_ORIGINS in the API
Go back to your API project on Vercel → **Settings → Environment Variables** → update:
```
ALLOWED_ORIGINS = https://mvpkidscards-web.vercel.app
```
Then Redeploy the API.

### Step 11: Add a custom domain (optional)
In either Vercel project → **Settings → Domains → Add** → type `mvpkidscards.com`
Vercel gives you the exact DNS records to add. SSL is automatic.

---

## Part 3 — Smoke Test Checklist

- [ ] `/health` endpoint returns 200
- [ ] Landing page loads
- [ ] Sign up creates a Supabase user
- [ ] Login redirects to dashboard
- [ ] Team signup returns a team code
- [ ] Join link (`/join.html?code=YOURCODE`) shows org info
- [ ] Buy button on athlete profile goes to Stripe Checkout
- [ ] Test payment completes, order appears in Supabase
- [ ] `admin.html` shows the order (log in with your `ADMIN_EMAILS` address)
- [ ] Forgot password email arrives, reset link works

### Stripe test card numbers
| Scenario | Card Number |
|---|---|
| Success | `4242 4242 4242 4242` |
| Decline | `4000 0000 0000 0002` |
| 3D Secure | `4000 0025 0000 3155` |

Any future expiry, any 3-digit CVC, any ZIP.

---

## Quick Reference

| Thing | URL |
|---|---|
| Frontend | `https://mvpkidscards-web.vercel.app` |
| API | `https://mvpkidscards-api.vercel.app` |
| Admin | `https://mvpkidscards-web.vercel.app/admin.html` |
| Vercel dashboard | `https://vercel.com/dashboard` |
| Supabase dashboard | `https://app.supabase.com` |
| Stripe dashboard | `https://dashboard.stripe.com` |
