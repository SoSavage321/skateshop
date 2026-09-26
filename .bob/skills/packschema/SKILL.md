---
name: packschema
description: Exact JSON schema and evidence rules for LegacyLens onboarding-pack files (overview, architecture, workflows, reading-order, quiz, tasks). Use whenever writing or checking onboarding-pack JSON.
---
# LegacyLens onboarding pack schema v1
Rules: paths are repo-relative, no leading "./" or "/"; folders end with "/"; lines are [start,end] counted from 1;
every Claim has at least 1 evidence item; meta.repoCommit = "e954d54379a7a6f798d3f49ffc6664449220adfc"; meta.mock = false.
Exactly 5 workflows. 6 to 9 reading steps. 8 to 10 questions, at least 1 per workflow, 2 for checkout_payment.

```ts
type Basis = "observed" | "inferred";
type WorkflowId = "auth" | "add_to_cart" | "checkout_payment" | "seller_product" | "data_layer";
interface Evidence { path: string; lines?: [number, number]; note?: string }
interface Claim { text: string; basis: Basis; evidence: Evidence[] }
interface Meta { schemaVersion: 1; mock: boolean; repoCommit: string; generatedBy: { member: string; bobTask: string; mode: string } }

// overview.json
interface Overview { meta: Meta; repo: { name: string; url: string; licence: string }; oneLine: string;
  stack: Claim[]; entryPoints: Claim[]; keyFacts: Claim[] }
// architecture.json  (mermaid: "graph TD", max 12 nodes, node ids = layer ids)
interface Architecture { meta: Meta;
  layers: { id: string; name: string; summary: string; paths: string[]; evidence: Evidence[] }[];
  edges: { from: string; to: string; label: string; basis: Basis; evidence: Evidence[] }[]; mermaid: string }
// workflows.json
interface Workflows { meta: Meta; workflows: { id: WorkflowId; name: string;
  actor: "visitor" | "customer" | "seller" | "developer"; summary: string;
  steps: { order: number; text: string; evidence: Evidence[] }[];
  externalServices: string[]; needsSecretsToRun: boolean; basis: Basis }[] }
// reading-order.json
interface ReadingOrder { meta: Meta; steps: { id: string; order: number; title: string; why: string;
  paths: string[]; workflowIds: WorkflowId[]; required: boolean; minutes: number }[] }
// quiz.json
interface Quiz { meta: Meta; questions: { id: string; workflowId: WorkflowId; readingStepId: string; prompt: string;
  options: { id: string; text: string }[]; answerId: string; explanation: string; evidence: Evidence[] }[];
  readiness: { thresholdPercent: number; weights: { reading: 30; quiz: 50; workflows: 20 }; requiredWorkflowIds: WorkflowId[] } }
// tasks.json
interface Tasks { meta: Meta; candidates: { id: string; title: string; workflowId: WorkflowId; files: string[];
  risk: "low" | "medium" | "high"; validation: string[]; externalDependencyRisk: string;
  decision: "selected" | "rejected"; reason: string; evidence: Evidence[] }[]; selectedId: string;
  blastRadius: { changedFiles: string[]; directDependents: Evidence[]; indirectImpact: Claim[]; risks: Claim[];
    validation: { command: string; env: string; expected: string; result: "not_run" | "pass" | "fail_code" | "fail_env"; log: string }[];
    rollback: string } }
```