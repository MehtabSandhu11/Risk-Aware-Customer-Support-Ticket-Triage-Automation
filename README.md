# Risk-Aware-Customer-Support-Ticket-Triage-Automation
This is Mehtab Singh's Final Project of his AIML25 Course.
TriageAI (Internal Protocol: 5911)
Python
Streamlit
License: MIT
TriageAI is a high-stakes Human-in-the-Loop (HITL) safety system designed for intelligent ticket triage in support workflows. Developed by Mehtab Singh Sandhu, it prioritizes regret minimization and asymmetric loss prevention, rejecting the "automation-first" paradigm. The core principle: the cost of an incorrect automation (false positive) is catastrophic compared to a manual review (false negative). This ensures safety-critical issues (e.g., data loss or security alerts) always escalate to human auditors.
Built with a layered defense (Swiss Cheese Model), TriageAI classifies support queries using a normalized 8-label taxonomy, applies risk-weighted penalties, and falls back to human review for low-confidence predictions. It's deployed via Streamlit for real-time auditing and includes a data flywheel for continuous model improvement.
🚀 Quick Demo

Input: "My account was hacked and data is missing!"
Output: HUMAN_REVIEW (Policy Veto: DATA_LOSS +0.5 penalty; Confidence: 0.92)
Auditor Action: Correct label → Exports to human_training_data.csv for retraining.

🧠 Core Philosophy

Regret Minimization: Automate only when safety is >95% confident (configurable modes: conservative, balanced, aggressive).
Asymmetric Loss: High-risk labels (e.g., DATA_LOSS) force veto; low-confidence defaults to safe GENERAL_SUPPORT.
HITL First: Every edge case routes to a stateful auditor workstation for corrections, fueling Phase 2 self-improvement.

🏗️ Technical Architecture
TriageAI uses a 3-layer gate system:

Layer 1: Risk Model (Logistic Regression)
Probabilistic base risk: base_risk = 1 - P(safe) from TF-IDF + LR on Suraj520 Kaggle Dataset + synthetic injections.
Layer 2: Intent Engine (Classifier)
Contextual signals (e.g., urgency, complaint) to modulate triage.
Layer 3: Policy Veto Issue Model (Classifier)
Taxonomy-bound classification with Final Risk = Base Risk + Policy Weight.
Weights: DATA_LOSS/ACCOUNT_ACCESS (+0.5), PAYMENT_PROBLEM (+0.4), etc.
Null-Confidence Fallback: If max prob < 0.50 (dynamic up to 0.70 for high-risk), → GENERAL_SUPPORT.


Mehtab Taxonomy (8 normalized labels; no granular sub-labels):

LabelDescriptionPolicy WeightDATA_LOSSData deletion/leak+0.5ACCOUNT_ACCESSLogin/security breach+0.5PAYMENT_PROBLEMBilling/refund issues+0.4SOFTWARE_BUGApp/code defects+0.3HARDWARE_FAILUREDevice/physical faults+0.3CONNECTIVITY_ISSUENetwork/outage problems+0.2DELIVERY_PROBLEMShipping/logistics+0.2GENERAL_SUPPORTEverything else (fallback)+0.0
Performance: V1 baseline AUC 0.99 (weak supervision on heuristics + synthetics).
Key Files:

risk_engine.py: Core prediction logic with absolute paths for cloud deploys.
app.py: Streamlit frontend (Auditor Workstation, Smart Logic for overrides, Pie Chart analytics).
train_master_model.py: Training pipeline (Suraj520 + synthetics; exports to CSV flywheel).

🚀 Installation

Clone the repo:textgit clone https://github.com/yourusername/triageai.git
cd triageai
Create a virtual environment:textpython -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate
Install dependencies:textpip install -r requirements.txt(Includes: streamlit, joblib, scikit-learn, numpy, pandas.)
Download models:
Place pre-trained models in ./models/ (TF-IDF vectorizers + classifiers).
Or run python train_master_model.py to generate from scratch (requires Kaggle API for Suraj520 dataset).


🔧 Usage
Run the Auditor Workstation
textstreamlit run app.py

Modes: Toggle conservative/balanced/aggressive thresholds.
Workflow: Paste query → Get prediction → Override if needed → Export to human_training_data.csv.
Analytics: Real-time Pie Chart of classifications/overrides.

Integrate as API
Pythonfrom risk_engine import RiskEngine

engine = RiskEngine(models_dir="./models")
result = engine.predict("Lost my password and funds are at risk!", mode="conservative")
print(result["decision"])  # e.g., "HUMAN_REVIEW"
Training & Flywheel

Run python train_master_model.py to retrain on human_training_data.csv.
Focus: Focal loss for imbalanced high-risk classes; synthetic injections for rare events.

🤝 Contributing

Fork & clone.
Create a feature branch: git checkout -b feature/safety-patch.
Commit changes: git commit -m "Add dynamic conf threshold".
Push & PR: Ensure tests pass; prioritize safety (no throughput regressions).


Guidelines: Preserve taxonomy; asymmetric logic first. Use weak supervision for labels.

📊 Performance & Monitoring

Metrics: Track AUC, false positive rate (target <0.01%), override rates.
MLOps: CSV exports enable drift detection; integrate with Weights & Biases for prod.

⚠️ Limitations & Roadmap

V1: Heuristic-heavy; Phase 2: LLM fine-tuning on flywheel data.
Scope: Support tickets only; no real-time chat.
Security: Models offline-fallback; audit paths for cloud.

📄 License
This project is licensed under the MIT License - see the LICENSE file for details.
