# PSD2’s Strong Customer Authentication: How Regulation Charges Payment-Fraud Analytics

This briefing decodes PSD2’s Strong Customer Authentication (SCA) requirements from a data-science perspective, showing how regulatory mandates can drive better fraud detection pipelines. Ideal for data scientists, risk analysts, and fintech engineers.

---

## 1. A New Data Contract for European Payments

The EU’s second Payment Services Directive (PSD2) re-wires the economics and telemetry of digital payments. At its heart sits **Strong Customer Authentication (SCA)**—the legal obligation for a payment-service provider (PSP) to challenge a user with at least two independent factors whenever they:

- Log into online banking
- Initiate an electronic payment
- Perform any remote action that may imply payment fraud[^1]

Legislators framed SCA as consumer protection, but for fraud-fighters it’s electrifying: every payment produces richer behavioural, device, and cryptographic metadata, and PSPs that omit SCA must refund victims immediately—internalising the cost of fraud.

---

## 2. What the Law _Actually_ Says

- **Trigger** (Art. 97): Apply SCA when users access accounts, send payments, or any remote action that implies fraud risk[^2].
- **Two-Factor Minimum**: Must use ≥ 2 elements from **knowledge**, **possession**, **inherence**, each independent[^2].
- **Dynamic Linking**: Factors must be _cryptographically bound_ to the **exact amount** and **specific payee**, blocking replay attacks[^2].
- **Blueprint** (Art. 98): The EBA drafts Regulatory Technical Standards (RTS) outlining exemptions, security requirements, and open API specs[^3].
- **Policy Intent**: Electronic payments must adopt technologies that “guarantee safe authentication and reduce…the risk of fraud” (Recital 95)[^4].

---

## 3. Signals Unlocked by SCA

| Touch-point             | Data Emitted                                             | Why Modellers Care                                          |
|-------------------------|----------------------------------------------------------|-------------------------------------------------------------|
| **Factor Chosen**       | Channel, delivery latency, failure codes, match-score    | Continuous auth & bot-detection features                    |
| **Dynamic-Link Payload**| Amount, currency, merchant ID, payee IBAN hashed        | Hard labels—securely ties user ↔ merchant ↔ amount           |
| **Challenge Orchestration** | Exemption flag, step-up reason, friction metric      | Cost-of-fraud vs. conversion optimisation                   |
| **Retry Telemetry**     | Failed factor, retry count, time-to-success              | Behavioural outlier detection; mule profiling               |

> _Pre-PSD2, card-not-present flows surfaced only PAN & IP. SCA injects **high-entropy** behavioural and cryptographic fields that materially lift model AUC._

---

## 4. Fraud-Detection Mechanics Reshaped by SCA

1. **Higher Signal-to-Noise**: Possession & inherence artefacts (e.g., secure-element IDs, biometric scores) boost AUC by 30–50 bps in benchmarks.
2. **Continental-Scale Labels**: Art. 96 mandates annual fraud filings, creating a supervised-learning goldmine for benchmarking[^1].
3. **Liability Incentives**: Skipping SCA shifts full loss to the PSP (cap to consumer), driving the need for _monetised_ risk scores[^5].
4. **Open-Banking Graph Reach**: Recital 93 enforces open APIs for AISPs/PISPs, enabling cross-bank graphs and mule detection[^6].

---

## 5. Cost Economics: Step-Up, Abandonments & Refunds

PSD2 reframes real-time fraud decisions:

- **Refund Obligation** (Art. 73): PSPs must return unauthorised funds by the next business day[^7].
- **Consumer Cap** (Art. 74): Liability limited to €50; €0 if SCA required but skipped[^5].
- **Friction Trade-Off**: RTS exemptions (low-value, TRA) avoid SCA but risk refunds. Models must weigh:

```pseudocode
function decideStepUp(transaction):
    risk = model.predictFraudRisk(transaction)
    expectedLoss = risk * transaction.amount
    frictionCost = abandonmentProbability(risk) * revenuePerTransaction
    if expectedLoss > frictionCost and satisfiesRTS(risk):
        return "Challenge with SCA"
    else:
        return "Approve without SCA"
```

---

## 6. Privacy & Governance Guard-Rails

PSD2 aligns with GDPR:

- **Legal Basis** (Art. 94): Processing personal data is allowed _when necessary_ for fraud prevention, subject to purpose limitation and minimisation[^8].
- **Risk Reporting** (Art. 95): Annual operational-risk assessments and control-effectiveness metrics must be filed[^3].

**Best Practices**:

```pseudocode
pipeline SCA_Telemetry:
    ingest(rawAuthEvent)
    dropSensitiveFields(rawAuthEvent)  // keep only hashes, scores
    enrichWithTransactionData(rawAuthEvent)
    storeInSecureVault(rawAuthEvent)
    if retentionExceeded(rawAuthEvent):
        purge(rawAuthEvent)
```

---

## 7. Data-Science Playbook under PSD2

```pseudocode
# 1. Risk-Based Step-Up Simulator
function simulateThresholds(history, thresholds):
    for t in thresholds:
        metrics = monteCarloSim(history, t)
        score[t] = metrics.profit - metrics.cost
    return argmax(score)

# 2. Behavioural-Biometric Embedding
function embedBiometrics(authSession):
    features = []
    features.append(stats(keystrokeDynamics(authSession)))
    features.append(sensorNoiseProfile(authSession))
    features.append(biometricMatchScore(authSession))
    return concatenate(features)

# 3. Graph-Based Mule Detection
function detectMules(transactions, authData):
    graph = buildGraph(transactions, authData)
    communities = communityDetection(graph)
    return highThroughputNodes(communities)

# 4. Real-Time Feature Store
function buildFeatureStore(authEvents, paymentStream):
    store = {}
    for event in authEvents:
        store[event.txId] = extractFeatures(event)
    for payment in paymentStream:
        features = store.get(payment.txId, defaultFeatures)
        score = fraudModel.predict(features)
        yield (payment, score)

# 5. Cost-Sensitive AutoML
function trainCostSensitiveModel(data):
    model = AutoML(classWeights=computeEuroWeights(data))
    return model.fit(data.features, data.labels)
```

---

## 8. What’s Next?

Passkeys and on-device biometrics satisfy possession + inherence without SMS, reducing latency and SIM-swap risks. The EBA’s commitment to update RTS for innovation[^3] signals an iterative loop: new tech → new standards → richer data → stronger models.

> **Bottom line:** PSD2’s SCA makes fraud-prevention data both legally available and economically mandatory. Architect pipelines that respect privacy boundaries yet exploit SCA telemetry to turn compliance into competitive advantage.

---

[^1]: PSD2 Art. 96(6)
[^2]: PSD2 Art. 97
[^3]: PSD2 Art. 98
[^4]: PSD2 Recital 95
[^5]: PSD2 Art. 74
[^6]: PSD2 Recital 93
[^7]: PSD2 Art. 73
[^8]: PSD2 Art. 94

**I, a Data Scientist at EY Greece as of May 1, 2025, confirm that this article is an independent exploration of fintech regulations and their global transaction impacts. It is not, in any way, related to or derived from the proprietary work, clients, or stakeholders of EY Greece.**

