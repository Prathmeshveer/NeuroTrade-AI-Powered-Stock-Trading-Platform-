# NeuroTrade-AI-Powered-Stock-Trading-Platform

AI-powered stock trading and analysis platform with Node.js, Express, Supabase, and a Maximalist Dynamic React Trading Terminal.

---

## ⚡ Single Command Startup (Both Backend & Frontend)

From the project root (`c:\Users\TUF GAMING\Desktop\Neurotrade`):

```powershell
npm run dev
```

> **What this does**: Uses `concurrently` to spin up both services in one terminal with color-coded logs:
> - 🟢 **[BACKEND]**: Express API running on `http://localhost:8000`
> - 🔵 **[FRONTEND]**: Cyber-Quant Vite UI running on `http://localhost:5173`

Then open your browser to:  
👉 **`http://localhost:5173`**

---

## 🛑 If Port 8000 is already in use
If you still have an old backend terminal open from earlier, press **`Ctrl + C`** in that terminal to stop it first, or run:
```powershell
Stop-Process -Id (Get-NetTCPConnection -LocalPort 8000).OwningProcess -Force
```
Then run `npm run dev`.

---

## 🗄️ One-Time Database Setup (Supabase)

Make sure you've executed the SQL migration in your Supabase project:
1. Open [Supabase SQL Editor](https://supabase.com/dashboard/project/krpsomrdnprldfzdnrtl).
2. Copy and paste the contents of `ml-service/supabase/migrations/001_initial_schema.sql`.
3. Click **Run**.
