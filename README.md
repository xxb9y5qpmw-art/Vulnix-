# Vulnix-

“Find vulnerabilities. Fix them faster.”

## Architecture

```text
Frontend (Next.js)
   ↓
API (Node.js / Express)
   ↓
Scanner Engine (rules + AI)
   ↓
Response (vulns + explanation + fixes)
```

## Scanner API (Example)

```js
import express from "express";
import cors from "cors";

const app = express();
app.use(cors());
app.use(express.json());

app.post("/scan", async (req, res) => {
  const { code } = req.body;
  const results = [];

  if (code.includes("SELECT") && code.includes("+")) {
    results.push({
      type: "SQL Injection",
      severity: "High",
      description: "Possible string concatenation in SQL query",
      fix: "Use parameterized queries",
    });
  }

  if (code.includes("<script>")) {
    results.push({
      type: "XSS",
      severity: "Medium",
      description: "Raw script tag detected",
      fix: "Sanitize user input",
    });
  }

  res.json({ results });
});

app.listen(3001, () => console.log("API running on port 3001"));
```

## Frontend (Example)

```jsx
import { useState } from "react";

export default function Home() {
  const [code, setCode] = useState("");
  const [results, setResults] = useState([]);

  const scanCode = async () => {
    const res = await fetch("http://localhost:3001/scan", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ code }),
    });

    const data = await res.json();
    setResults(data.results);
  };

  return (
    <div className="p-10">
      <h1 className="text-3xl font-bold mb-4">Vulnix Scanner</h1>

      <textarea
        className="w-full h-40 p-2 border"
        placeholder="Paste code here..."
        onChange={(e) => setCode(e.target.value)}
      />

      <button onClick={scanCode} className="mt-4 bg-black text-white px-4 py-2">
        Scan
      </button>

      <div className="mt-6">
        {results.map((r, i) => (
          <div key={i} className="border p-3 mb-2">
            <h2 className="font-bold">{r.type}</h2>
            <p>{r.description}</p>
            <p className="text-green-600">Fix: {r.fix}</p>
          </div>
        ))}
      </div>
    </div>
  );
}
```

## Added Feature: Exploit Chain Analysis

Vulnix now supports exploit chain analysis to correlate multiple medium/high issues into a realistic attack path. This is designed for **defensive prioritization** so teams can fix root causes faster.

### What an exploit chain includes

- **Entry point** (e.g., reflected XSS)
- **Privilege step** (e.g., token theft / session replay)
- **Impact step** (e.g., admin action abuse, data exfiltration)
- **Business impact score** (Low / Medium / High / Critical)

### Example chain output

```json
{
  "exploitChains": [
    {
      "id": "chain-01",
      "title": "Reflected XSS → Session Hijack → Privileged API Abuse",
      "risk": "Critical",
      "confidence": 0.86,
      "steps": [
        {
          "vulnType": "XSS",
          "role": "entry",
          "explanation": "Unsanitized HTML enables JavaScript execution in victim browser."
        },
        {
          "vulnType": "Weak Session Controls",
          "role": "pivot",
          "explanation": "Session token can be replayed after theft."
        },
        {
          "vulnType": "Broken Access Control",
          "role": "impact",
          "explanation": "Hijacked session can call admin-grade endpoints."
        }
      ],
      "recommendedFixOrder": [
        "Enforce output encoding and sanitization",
        "Harden session lifecycle and token binding",
        "Add server-side authorization checks for privileged routes"
      ]
    }
  ]
}
```

### API contract extension

`POST /scan` now returns:

- `results`: individual findings
- `exploitChains`: linked multi-step attack paths

This enables dashboards to prioritize by exploitability, not only per-finding severity.
