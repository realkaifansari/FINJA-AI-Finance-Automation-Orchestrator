# FINJA-AI-Finance-Automation-Orchestrator
FINJA AI automates end-to-end finance operations — invoice processing, payment approvals, reconciliation, and anomaly detection — by orchestrating watsonx skills, integrations, and workflow automations to reduce manual effort and speed up month-end close.

finja-ai/
│
├── backend/
│   ├── server.js
│   ├── routes/
│   │   ├── invoice.js
│   │   ├── payment.js
│   │   └── ledger.js
│   ├── services/
│   │   ├── ocr.js
│   │   ├── orchestrate.js
│   │   └── anomaly.js
│   ├── utils/
│   │   └── logger.js
│   └── package.json
│
├── frontend/
│   ├── src/
│   │   ├── App.js
│   │   ├── api.js
│   │   └── components/
│   │        ├── InvoiceCard.jsx
│   │        └── AlertBox.jsx
│   ├── package.json
│
├── watsonx_workflows/
│   └── invoice_workflow.json
│
└── README.md
{
  "name": "finja-backend",
  "version": "1.0.0",
  "main": "server.js",
  "type": "module",
  "dependencies": {
    "express": "^4.18.2",
    "multer": "^1.4.5",
    "axios": "^1.6.0",
    "tesseract.js": "^5.0.2",
    "cors": "^2.8.5"
  }
}
import express from "express";
import cors from "cors";
import invoiceRoute from "./routes/invoice.js";
import paymentRoute from "./routes/payment.js";
import ledgerRoute from "./routes/ledger.js";

const app = express();

app.use(cors());
app.use(express.json());

app.use("/invoice", invoiceRoute);
app.use("/payment", paymentRoute);
app.use("/ledger", ledgerRoute);

app.listen(5000, () => {
  console.log("FINJA AI backend running on port 5000");
});
import express from "express";
import multer from "multer";
import { extractText } from "../services/ocr.js";
import { triggerWatsonWorkflow } from "../services/orchestrate.js";

const router = express.Router();
const upload = multer({ dest: "uploads/" });

router.post("/upload", upload.single("invoice"), async (req, res) => {
  const text = await extractText(req.file.path);

  const result = await triggerWatsonWorkflow({
    action: "INVOICE_RECEIVED",
    payload: { text }
  });

  res.json({
    message: "Invoice processed successfully",
    extractedText: text,
    watsonResponse: result
  });
});

export default router;
import Tesseract from "tesseract.js";

export const extractText = async (filePath) => {
  const result = await Tesseract.recognize(filePath, "eng");
  return result.data.text;
};
import axios from "axios";

const WATSONX_URL = process.env.WATSONX_URL;
const WATSONX_KEY = process.env.WATSONX_KEY;

export const triggerWatsonWorkflow = async (data) => {
  try {
    const res = await axios.post(
      `${WATSONX_URL}/workflow/trigger`,
      data,
      {
        headers: {
          Authorization: `Bearer ${WATSONX_KEY}`,
          "Content-Type": "application/json"
        }
      }
    );

    return res.data;
  } catch (err) {
    return { error: err.message };
  }
};
import express from "express";

const router = express.Router();

router.post("/pay", (req, res) => {
  const { amount, vendor } = req.body;

  res.json({
    status: "SUCCESS",
    transactionId: "TXN" + Date.now(),
    vendor,
    amount
  });
});

export default router;
import express from "express";

const router = express.Router();

let ledger = [];

router.post("/add", (req, res) => {
  ledger.push(req.body);
  res.json({ message: "Ledger updated", ledger });
});

router.get("/all", (req, res) => {
  res.json(ledger);
});

export default router;
{
  "name": "finja-frontend",
  "version": "1.0.0",
  "dependencies": {
    "axios": "^1.6.0",
    "react": "^18.0.0",
    "react-dom": "^18.0.0"
  }
}
import axios from "axios";

const API = axios.create({
  baseURL: "http://localhost:5000"
});

export default API;
import React, { useState } from "react";
import API from "./api";

function App() {
  const [file, setFile] = useState(null);
  const [result, setResult] = useState(null);

  const uploadInvoice = async () => {
    const form = new FormData();
    form.append("invoice", file);

    const res = await API.post("/invoice/upload", form);
    setResult(res.data);
  };

  return (
    <div style={{ padding: 40 }}>
      <h1>FINJA AI — Invoice Automation</h1>

      <input type="file" onChange={(e) => setFile(e.target.files[0])} />
      <button onClick={uploadInvoice}>Upload Invoice</button>

      {result && (
        <pre>{JSON.stringify(result, null, 2)}</pre>
      )}
    </div>
  );
}

export default App;
{
  "workflow": {
    "name": "FINJA_Invoice_Automation",
    "steps": [
      "Extract Invoice",
      "Validate",
      "Approval",
      "Payment",
      "Ledger_Update",
      "Reconciliation"
    ]
  }
}
