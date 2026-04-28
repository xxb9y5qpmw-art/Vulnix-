# Vulnix-
“Find vulnerabilities. Fix them faster.”
Frontend (Next.js)
   ↓
API (Node.js / Express)
   ↓
Scanner Engine (rules + AI)
   ↓
Response (vulns + explanation + fixes)import express from "express";
import cors from "cors";

const app = express();
app.use(cors());
app.use(express.json());

app.post("/scan", async (req, res) => {
  const { code } = req.body;

  const results = [];

  // Simple rule-based detection
  if (code.includes("SELECT") && code.includes("+")) {
    results.push({
      type: "SQL Injection",
      severity: "High",
      description: "Possible string concatenation in SQL query",
      fix: "Use parameterized queries"
    });
  }

  if (code.includes("<script>")) {
    results.push({
      type: "XSS",
      severity: "Medium",
      description: "Raw script tag detected",
      fix: "Sanitize user input"
    });
  }

  res.json({ results });
});

app.listen(3001, () => console.log("API running on port 3001"));import { useState } from "react";

export default function Home() {
  const [code, setCode] = useState("");
  const [results, setResults] = useState([]);

  const scanCode = async () => {
    const res = await fetch("http://localhost:3001/scan", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ code })
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

      <button
        onClick={scanCode}
        className="mt-4 bg-black text-white px-4 py-2"
      >
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
}import OpenAI from "openai";

const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

const aiResponse = await openai.chat.completions.create({
  model: "gpt-4.1",
  messages: [
    {
      role: "system",
      content: "You are a cybersecurity expert."
    },
    {
      role: "user",
      content: `Analyze this code for vulnerabilities:\n${code}`
    }
  ]
});npm install firebaseimport { initializeApp } from "firebase/app";
import { getAuth } from "firebase/auth";

const firebaseConfig = {
  apiKey: "YOUR_KEY",
  authDomain: "YOUR_DOMAIN",
  projectId: "YOUR_PROJECT_ID",
};

const app = initializeApp(firebaseConfig);

export const auth = getAuth(app);import { auth } from "../firebase";
import { createUserWithEmailAndPassword, signInWithEmailAndPassword } from "firebase/auth";

const signup = async (email, password) => {
  await createUserWithEmailAndPassword(auth, email, password);
};

const login = async (email, password) => {
  await signInWithEmailAndPassword(auth, email, password);
};npm install stripeimport Stripe from "stripe";
const stripe = new Stripe(process.env.STRIPE_SECRET);

app.post("/create-checkout", async (req, res) => {
  const { userId } = req.body;

  const session = await stripe.checkout.sessions.create({
    payment_method_types: ["card"],
    mode: "subscription",
    line_items: [
      {
        price: "price_12345", // from Stripe dashboard
        quantity: 1,
      },
    ],
    success_url: "http://localhost:3000/success",
    cancel_url: "http://localhost:3000/cancel",
    metadata: {
      userId,
    },
  });

  res.json({ url: session.url });
});const upgrade = async () => {
  const res = await fetch("http://localhost:3001/create-checkout", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      userId: auth.currentUser.uid,
    }),
  });

  const data = await res.json();
  window.location.href = data.url;
};import bodyParser from "body-parser";

app.post("/webhook", bodyParser.raw({ type: "application/json" }), async (req, res) => {
  const event = req.body;

  if (event.type === "checkout.session.completed") {
    const session = event.data.object;
    const userId = session.metadata.userId;

    // Mark user as pro in DB
    await db.collection("users").doc(userId).set({
      pro: true,
    }, { merge: true });
  }

  res.sendStatus(200);
});app.post("/scan", async (req, res) => {
  const { userId } = req.body;

  const userDoc = await db.collection("users").doc(userId).get();
  const isPro = userDoc.data()?.pro;

  if (!isPro) {
    return res.status(403).json({
      error: "Upgrade required",
    });
  }

  // run AI scan...
});import admin from "firebase-admin";

const decoded = await admin.auth().verifyIdToken(idToken);
const userId = decoded.uid;const findings = [];

// Rule-based checks
if (code.includes("SELECT") && code.includes("+")) {
  findings.push("Possible SQL Injection");
}

if (code.includes("<script>")) {
  findings.push("Possible XSS");
}

// Only call AI if needed
if (findings.length > 0 || isPro) {
  // call AI
}const lines = code.split("\n");

const suspiciousLines = lines.filter(line =>
  line.includes("SELECT") ||
  line.includes("<script>") ||
  line.includes("eval(") ||
  line.includes("innerHTML")
);

const trimmedCode = suspiciousLines.join("\n").slice(0, 2000);You are a security scanner.

Only return:
- vulnerability type
- severity (low/medium/high)
- 1 sentence explanation
- 1 fix

Be concise.const model = isPro ? "gpt-4.1" : "gpt-4.1-mini";import crypto from "crypto";

const hash = crypto.createHash("sha256").update(code).digest("hex");

// check cache
const cached = await db.collection("scans").doc(hash).get();

if (cached.exists) {
  return res.json(cached.data());
}

// otherwise run AI and storeconst scansToday = await getUserScanCount(userId);

if (!isPro && scansToday > 5) {
  return res.status(429).json({
    error: "Daily limit reached"
  });
}const response = await openai.chat.completions.create({
  model,
  messages,
  max_tokens: 200, // LIMIT COST
});response_format: {
  type: "json_object"
}User submits code
   ↓
Rule-based scan
   ↓
No issues? → return fast result (FREE)
   ↓
Issues found?
   ↓
Trim code
   ↓
Check cache
   ↓
Call AI (cheap model)
   ↓
Return structured resultYou are Vulnix, a professional penetration testing assistant.

Your job:
- Identify real security vulnerabilities in code
- Avoid false positives
- Be concise and structured

Rules:
- Only report real, plausible vulnerabilities
- Do NOT guess or speculate
- If unsure, say "No confirmed vulnerability"
- Focus on OWASP Top 10 issues
- Keep responses short and technical

Output format (JSON only):
{
  "vulnerabilities": [
    {
      "type": "",
      "severity": "low | medium | high",
      "location": "",
      "explanation": "",
      "fix": ""
    }
  ]
}const messages = [
  {
    role: "system",
    content: BASE_PROMPT
  },
  {
    role: "user",
    content: `Analyze this code for security vulnerabilities:\n${trimmedCode}`
  }
];content: `
Language: JavaScript (Node.js, Express)
Context: Backend API handling user input

Code:
${trimmedCode}
`content: `
Language: JavaScript (Node.js, Express)
Context: Backend API handling user input

Code:
${trimmedCode}
`Severity guidelines:
- Low: minor issues, unlikely to be exploited
- Medium: possible vulnerability with constraints
- High: easily exploitable, serious impactconst response = await openai.chat.completions.create({
  model,
  messages,
  response_format: { type: "json_object" },
  max_tokens: 200
});Explain this vulnerability clearly:

Type: SQL Injection
Code: ${snippet}

Explain:
- Why it's vulnerable
- Real-world impact
- How to fix it securelyBefore reporting a vulnerability:
- Verify it is actually exploitable
- Do not flag safe patterns
- Prefer missing a vuln over false positivesFocus on:
- unsafe eval/exec
- SQL injection (sqlite, psycopg2)
- insecure deserialization (pickle)Focus on:
- XSS (innerHTML, dangerouslySetInnerHTML)
- prototype pollution
- insecure JWT handlingFind vulnerabilities. Return JSON only.You are Vulnix, a professional penetration testing assistant.

Analyze the provided code for real security vulnerabilities.

Rules:
- Only report confirmed, realistic vulnerabilities
- Avoid false positives
- If none found, return empty array
- Be concise

Focus on OWASP Top 10:
- SQL Injection
- XSS
- Auth issues
- Insecure deserialization
- RCE

Severity:
- low / medium / high

Return JSON only:
{
  "vulnerabilities": [
    {
      "type": "",
      "severity": "",
      "location": "",
      "explanation": "",
      "fix": ""
    }
  ]
}Additionally:
- Show attack scenario
- Suggest secure code rewrite
- Mention tools used in real pentestsif (code.includes("SELECT") && code.includes("+"))const query = "SELECT * FROM users " + userInput;VariableDeclaration
 └── VariableDeclarator
      ├── Identifier: query
      └── BinaryExpression (+)
           ├── StringLiteral: "SELECT * FROM users "
           └── Identifier: userInputnpm install @babel/parser @babel/traverseimport * as parser from "@babel/parser";

export function parseCode(code) {
  return parser.parse(code, {
    sourceType: "unambiguous",
    plugins: ["jsx", "typescript"]
  });
}import traverse from "@babel/traverse";

export function analyzeAST(ast) {
  const findings = [];

  traverse(ast, {
    // Detect dangerous function calls
    CallExpression(path) {
      const name = path.node.callee.name;

      if (name === "eval") {
        findings.push({
          type: "Remote Code Execution",
          severity: "high",
          location: path.node.loc,
          explanation: "Use of eval() allows code injection",
          fix: "Avoid eval; use safe parsing methods"
        });
      }
    },

    // Detect SQL injection patterns
    BinaryExpression(path) {
      if (path.node.operator === "+") {
        const left = path.node.left;
        const right = path.node.right;

        if (
          left.type === "StringLiteral" &&
          left.value.toLowerCase().includes("select") &&
          right.type === "Identifier"
        ) {
          findings.push({
            type: "SQL Injection",
            severity: "high",
            location: path.node.loc,
            explanation: "User input concatenated into SQL query",
            fix: "Use parameterized queries"
          });
        }
      }
    }
  });

  return findings;
}if (node.type === "Identifier" && node.name === "req.body") {
  markAsSource(path);
}if (isSink(node) && isTainted(node)) {
  reportVulnerability();
}const taintedVars = new Set();

function markTainted(name) {
  taintedVars.add(name);
}

function isTainted(name) {
  return taintedVars.has(name);
}User Code
   ↓
AST Parser
   ↓
AST Traverser
   ↓
Source/Sink Tracker
   ↓
Vulnerability Engine
   ↓
AI Explanation (optional)[req.body] → [userInput] → [query] → [db.execute]
   SOURCE        VAR           VAR         SINKclass Node {
  constructor(id, type, value) {
    this.id = id;
    this.type = type; // source | var | sink | function
    this.value = value;
    this.edges = [];
  }
}import traverse from "@babel/traverse";

let nodeId = 0;
const graphNodes = new Map();

function createNode(type, value) {
  const node = new Node(nodeId++, type, value);
  graphNodes.set(node.id, node);
  return node;
}traverse(ast, {
  VariableDeclarator(path) {
    const name = path.node.id.name;

    const node = createNode("var", name);

    // track assignment
    if (path.node.init) {
      const init = path.node.init;

      if (init.type === "MemberExpression") {
        if (init.object.name === "req") {
          node.type = "source";
        }
      }
    }
  }
});const SOURCES = [
  "req.body",
  "req.query",
  "req.params",
  "localStorage",
  "document.cookie"
];if (code.includes("req.body")) {
  createNode("source", "req.body");
}const SINKS = [
  "eval",
  "exec",
  "innerHTML",
  "db.query",
  "document.write"
];CallExpression(path) {
  const name = path.node.callee.name;

  if (SINKS.includes(name)) {
    createNode("sink", name);
  }
}const tainted = new Set();

function markTainted(node) {
  tainted.add(node);
}

function isTainted(node) {
  return tainted.has(node);
}function propagate(node, fromNode) {
  if (fromNode.type === "source" || isTainted(fromNode)) {
    markTainted(node);
  }
}function findPaths(node, targetType, path = [], visited = new Set()) {
  if (visited.has(node.id)) return [];
  visited.add(node.id);

  path.push(node);

  if (node.type === targetType) {
    return [path];
  }

  let results = [];

  for (const edge of node.edges) {
    results.push(...findPaths(edge, targetType, [...path], visited));
  }

  return results;
}{
  "exploit_paths": [
    {
      "path": [
        "req.body.username",
        "userInput",
        "queryString",
        "db.query()"
      ],
      "risk": "HIGH",
      "attack": "SQL Injection",
      "description": "User input flows directly into SQL query without sanitization"
    }
  ]
}AST → Raw Taint Graph → AI Graph Cleaner → Ranked Exploit Paths{
  "paths": [
    {
      "nodes": [
        "req.body.username",
        "sanitize()",
        "query",
        "db.execute"
      ]
    },
    {
      "nodes": [
        "req.body.id",
        "parseInt()",
        "query",
        "db.execute"
      ]
    }
  ]
}You are Vulnix Graph Auditor.

Your job is to analyze exploit paths generated from static taint analysis.

You do NOT detect new vulnerabilities.

You ONLY:
- Remove false positive paths
- Mark paths as "valid exploit" or "invalid"
- Explain why invalid paths are safe
- Rank valid paths by exploitability

Rules:
- Assume modern frameworks use sanitization unless clearly bypassed
- Ignore paths blocked by sanitizers, type conversion, ORM escaping
- Prefer precision over recall (avoid false positives)
- Only mark HIGH confidence when exploit is realistically possible

Output JSON only:
{
  "valid_paths": [...],
  "rejected_paths": [...],
  "notes": ""
}const response = await openai.chat.completions.create({
  model: "gpt-4.1-mini",
  messages: [
    {
      role: "system",
      content: GRAPH_CLEANUP_PROMPT
    },
    {
      role: "user",
      content: JSON.stringify({
        paths: rawPaths
      })
    }
  ],
  response_format: { type: "json_object" }
});req.body.name → escapeHtml() → innerHTMLreq.body.id → parseInt() → SQL queryUser.find({ id: req.body.id })if (false) {
  req.body → db.query
}{
  "confidence": 0.0 - 1.0,
  "exploit_difficulty": "low | medium | high",
  "impact": "low | medium | high | critical"
}req.body → sanitize → query → db
req.body → parseInt → query → db
req.body → concat → query → db{
  "valid_paths": [
    {
      "path": ["req.body → concat → query → db"],
      "confidence": 0.92
    }
  ],
  "rejected_paths": [
    "sanitize path",
    "parseInt path"
  ]
}Attack Probability =
  (Source Risk)
+ (Sink Severity)
+ (Taint Strength)
+ (Sanitization Weakness)
+ (Reachability)
- (Framework Protection)eval() → 30
db.query() → 25
innerHTML → 20
console.log → 2Probability =
  Source
+ Sink
+ Taint
+ Reachability
+ SanitizationPenalty
+ FrameworkPenaltyMath.max(0, Math.min(100, score))function calculateAttackProbability(path) {
  let score = 0;

  score += scoreSource(path.source);
  score += scoreSink(path.sink);
  score += scoreTaint(path.taintLevel);
  score += scoreReachability(path.reachable);
  score += scoreSanitization(path.sanitizers);
  score += scoreFramework(path.frameworkProtections);

  return Math.max(0, Math.min(100, score));
}req.body.username → concat → db.query0–30   → Low (theoretical)
31–60  → Medium (requires conditions)
61–85  → High (likely exploitable)
86–100 → Critical (immediate exploit possible)You are a security validation engine.

Given a computed exploit probability and path metadata:

- Confirm or adjust score
- Detect missing context
- Reduce false positives
- Increase score only if clearly justified

Do NOT invent vulnerabilities.AST Graph
   ↓
Deterministic Scoring Engine
   ↓
Attack Probability Score
   ↓
AI Calibration Layer
   ↓
Final Risk Score + Explanation{
  "attack_path": [
    "req.body.username",
    "concat",
    "db.query"
  ],
  "probability": 87,
  "severity": "high",
  "exploit_difficulty": "low",
  "impact": "critical",
  "summary": "User input flows directly into SQL query without sanitization",
  "confidence": 0.91
}
