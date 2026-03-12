# InvisaClaim-Demo

import React, { useMemo, useState } from "react";
import { motion } from "framer-motion";
import {
  Search,
  Upload,
  FileText,
  ShieldCheck,
  DollarSign,
  Clock3,
  CheckCircle2,
  AlertTriangle,
  Filter,
  Brain,
  ArrowRight,
  Download,
  UserCheck,
  Activity,
  Bell,
  Menu,
  PlusCircle,
  RefreshCw,
} from "lucide-react";
import { Card, CardContent, CardHeader, CardTitle, CardDescription } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Badge } from "@/components/ui/badge";
import { Tabs, TabsContent, TabsList, TabsTrigger } from "@/components/ui/tabs";
import { Progress } from "@/components/ui/progress";
import { Textarea } from "@/components/ui/textarea";
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "@/components/ui/select";
import { Table, TableBody, TableCell, TableHead, TableHeader, TableRow } from "@/components/ui/table";

const seedCases = [
  {
    id: "IC-2026-001",
    patient: "Patient A",
    payer: "BlueCross Commercial",
    denialCode: "CARC 50",
    denialText: "Medical necessity not established.",
    category: "Medical Necessity",
    confidence: 91,
    claimAmount: 2480,
    deadline: "2026-03-28",
    status: "Ready for Review",
    recommendedAction: "Generate Appeal Draft",
    workflowRisk: "Low",
    evidenceStrength: 86,
    route: "Reviewer Approval",
    specialty: "Pain Management",
    summary:
      "The claim was denied for lack of documented medical necessity. The chart includes chronic pain history, failed conservative treatment, and physician assessment supporting the requested procedure.",
    evidence: [
      {
        label: "Progress Note - 2026-02-14",
        excerpt:
          "Patient has persistent lumbar radiculopathy for 6+ months despite PT, NSAIDs, and home exercise program.",
        relevance: 96,
      },
      {
        label: "Assessment & Plan - 2026-02-14",
        excerpt:
          "Symptoms continue to impair function and sleep. Provider recommends intervention after failed conservative management.",
        relevance: 92,
      },
      {
        label: "Procedure Request",
        excerpt:
          "Requested CPT aligns with diagnosis and treatment pathway documented in chart.",
        relevance: 81,
      },
    ],
    draft: `RE: Appeal of claim denial for medical necessity

This appeal requests reconsideration of the denial associated with claim IC-2026-001. The patient record documents chronic lumbar radiculopathy with persistent symptoms despite conservative treatment, including physical therapy, NSAID therapy, and home exercise efforts. Clinical notes dated 2026-02-14 show ongoing functional limitation and physician assessment supporting the requested service. Based on the accompanying documentation, the requested procedure meets the medical necessity standard reflected in the patient chart and should be reconsidered for payment.`,
  },
  {
    id: "IC-2026-002",
    patient: "Patient B",
    payer: "UnitedHealth",
    denialCode: "CARC 16",
    denialText: "Claim/service lacks information which is needed for adjudication.",
    category: "Missing Documentation",
    confidence: 84,
    claimAmount: 1190,
    deadline: "2026-03-25",
    status: "Needs Documents",
    recommendedAction: "Request Missing Documents",
    workflowRisk: "Medium",
    evidenceStrength: 52,
    route: "RCM Follow-Up",
    specialty: "Dermatology",
    summary:
      "The denial appears tied to incomplete supporting documentation. The current record lacks a sufficiently detailed treatment note and image attachment.",
    evidence: [
      {
        label: "Visit Summary - 2026-02-20",
        excerpt:
          "Procedure was performed, but the note is brief and does not fully describe prior treatment history.",
        relevance: 68,
      },
    ],
    draft: `RE: Documentation request support

The current denial indicates missing information required for adjudication. Additional documentation should be assembled before appeal submission, including the full provider note, treatment history, and any supporting images or prior response history.`,
  },
  {
    id: "IC-2026-003",
    patient: "Patient C",
    payer: "Aetna",
    denialCode: "CARC 197",
    denialText: "Authorization/Precertification absent.",
    category: "Authorization Issue",
    confidence: 78,
    claimAmount: 6840,
    deadline: "2026-03-19",
    status: "Escalated",
    recommendedAction: "Escalate to Compliance Review",
    workflowRisk: "High",
    evidenceStrength: 73,
    route: "Compliance Review",
    specialty: "Orthopedics",
    summary:
      "The case may involve an authorization mismatch. There is possible evidence of a submitted authorization, but the record is incomplete and requires human review before response.",
    evidence: [
      {
        label: "Authorization Log",
        excerpt:
          "Reference number appears in intake notes, but payer confirmation is not fully attached.",
        relevance: 79,
      },
      {
        label: "Scheduling Record",
        excerpt:
          "Procedure scheduled after utilization review communication, but definitive approval artifact missing.",
        relevance: 71,
      },
    ],
    draft: `RE: Authorization review required

This case should not be auto-routed. The current record suggests a possible authorization pathway, but the support is not sufficient for automated submission. Human review is recommended before any payer response is made.`,
  },
];

function createMockCase(idNum, form) {
  const amount = Number(form.claimAmount || 1750);
  const categoryMap = {
    medical: ["Medical Necessity", "Medical necessity not established.", "Low", 89, 84, "Generate Appeal Draft"],
    docs: ["Missing Documentation", "Claim/service lacks information needed for adjudication.", "Medium", 82, 57, "Request Missing Documents"],
    auth: ["Authorization Issue", "Authorization or precertification absent.", "High", 76, 71, "Escalate to Compliance Review"],
  };
  const [category, denialText, workflowRisk, confidence, evidenceStrength, recommendedAction] = categoryMap[form.caseType] || categoryMap.medical;
  return {
    id: `IC-2026-${String(idNum).padStart(3, "0")}`,
    patient: form.patient || `Patient ${idNum}`,
    payer: form.payer || "BlueCross Commercial",
    denialCode: form.caseType === "auth" ? "CARC 197" : form.caseType === "docs" ? "CARC 16" : "CARC 50",
    denialText,
    category,
    confidence,
    claimAmount: amount,
    deadline: "2026-03-30",
    status: "New",
    recommendedAction,
    workflowRisk,
    evidenceStrength,
    route: workflowRisk === "High" ? "Compliance Review" : "Reviewer Approval",
    specialty: form.specialty || "General",
    summary: `This mock denial case was created from the demo uploader for ${form.payer || "a payer"}. The system proposes ${recommendedAction.toLowerCase()} based on the initial denial pattern and available simulated evidence.`,
    evidence: [
      {
        label: "Uploaded Chart Note",
        excerpt: `${form.patient || "The patient"} record includes simulated support relevant to the denial pattern selected in this demo.`,
        relevance: 84,
      },
      {
        label: "Claim Summary",
        excerpt: `Claim amount of $${amount.toLocaleString()} and denial details were ingested into the review queue.`,
        relevance: 77,
      },
    ],
    draft: `RE: Preliminary response for ${category.toLowerCase()} denial

This draft was generated from the demo uploader. The system identified the claim as a ${category.toLowerCase()} case for ${form.payer || "the payer"}. Reviewer validation is still required before submission.`
  };
}

function StatusBadge({ value }) {
  const map = {
    "Ready for Review": "bg-emerald-50 text-emerald-700 border-emerald-200",
    "Needs Documents": "bg-amber-50 text-amber-700 border-amber-200",
    Escalated: "bg-rose-50 text-rose-700 border-rose-200",
    Submitted: "bg-sky-50 text-sky-700 border-sky-200",
    Approved: "bg-indigo-50 text-indigo-700 border-indigo-200",
    Paid: "bg-emerald-50 text-emerald-700 border-emerald-200",
    New: "bg-slate-100 text-slate-700 border-slate-200",
  };
  return <span className={`inline-flex rounded-full border px-3 py-1 text-xs font-semibold ${map[value] || "bg-slate-50 text-slate-700 border-slate-200"}`}>{value}</span>;
}

export default function InvisaClaimDemo() {
  const [cases, setCases] = useState(seedCases);
  const [query, setQuery] = useState("");
  const [statusFilter, setStatusFilter] = useState("all");
  const [selectedId, setSelectedId] = useState(seedCases[0].id);
  const [draft, setDraft] = useState(seedCases[0].draft);
  const [notes, setNotes] = useState("Approved after reviewer verification of chart support.");
  const [toast, setToast] = useState("");
  const [uploadForm, setUploadForm] = useState({
    patient: "",
    payer: "",
    claimAmount: "",
    specialty: "",
    caseType: "medical",
  });

  const filteredCases = useMemo(() => {
    return cases.filter((c) => {
      const matchesQuery = [c.id, c.payer, c.category, c.specialty, c.denialText]
        .join(" ")
        .toLowerCase()
        .includes(query.toLowerCase());
      const matchesStatus = statusFilter === "all" ? true : c.status === statusFilter;
      return matchesQuery && matchesStatus;
    });
  }, [cases, query, statusFilter]);

  const selected = filteredCases.find((c) => c.id === selectedId) || cases.find((c) => c.id === selectedId) || filteredCases[0] || cases[0];

  const metrics = useMemo(() => {
    const openDenials = cases.filter((c) => !["Paid", "Closed"].includes(c.status)).length;
    const atRiskRevenue = cases.filter((c) => !["Paid", "Closed"].includes(c.status)).reduce((sum, c) => sum + c.claimAmount, 0);
    const ready = cases.filter((c) => c.status === "Ready for Review").length;
    const recovered = cases.filter((c) => c.status === "Paid").reduce((sum, c) => sum + c.claimAmount, 0);
    return [
      { label: "Open Denials", value: String(openDenials), icon: AlertTriangle },
      { label: "At-Risk Revenue", value: `$${atRiskRevenue.toLocaleString()}`, icon: DollarSign },
      { label: "Ready for Review", value: String(ready), icon: UserCheck },
      { label: "Recovered", value: `$${recovered.toLocaleString()}`, icon: CheckCircle2 },
    ];
  }, [cases]);

  React.useEffect(() => {
    if (selected) {
      setSelectedId(selected.id);
      setDraft(selected.draft);
    }
  }, [selected?.id]);

  React.useEffect(() => {
    if (!toast) return;
    const timer = setTimeout(() => setToast(""), 2200);
    return () => clearTimeout(timer);
  }, [toast]);

  function updateCaseStatus(status) {
    if (!selected) return;
    setCases((prev) =>
      prev.map((c) =>
        c.id === selected.id
          ? {
              ...c,
              status,
              draft,
            }
          : c
      )
    );
    setToast(`Case ${selected.id} moved to ${status}.`);
  }

  function addMockCase() {
    const nextIdNum = cases.length + 1;
    const newCase = createMockCase(nextIdNum, uploadForm);
    setCases((prev) => [newCase, ...prev]);
    setSelectedId(newCase.id);
    setDraft(newCase.draft);
    setUploadForm({ patient: "", payer: "", claimAmount: "", specialty: "", caseType: "medical" });
    setToast(`Demo case ${newCase.id} added to queue.`);
  }

  return (
    <div className="min-h-screen bg-slate-50 text-slate-900">
      <div className="border-b bg-white/90 backdrop-blur">
        <div className="mx-auto flex max-w-7xl items-center justify-between px-6 py-4">
          <div className="flex items-center gap-4">
            <div className="rounded-2xl border border-slate-200 p-2 lg:hidden">
              <Menu className="h-5 w-5 text-slate-700" />
            </div>
            <div className="flex items-center gap-3">
              <div className="flex h-11 w-11 items-center justify-center rounded-2xl bg-gradient-to-br from-sky-500 to-emerald-400 text-white shadow-lg">
                <ShieldCheck className="h-5 w-5" />
              </div>
              <div>
                <div className="text-xl font-bold tracking-tight">InvisaClaim</div>
                <div className="text-xs text-slate-500">Working demo: denial triage, evidence review, draft appeals</div>
              </div>
            </div>
          </div>
          <div className="flex items-center gap-3">
            <Button variant="outline" className="rounded-xl">
              <Bell className="mr-2 h-4 w-4" /> Notifications
            </Button>
            <Button className="rounded-xl bg-slate-900 hover:bg-slate-800">Live Pilot Mode</Button>
          </div>
        </div>
      </div>

      <div className="mx-auto max-w-7xl px-6 py-8">
        {toast && (
          <motion.div initial={{ opacity: 0, y: -8 }} animate={{ opacity: 1, y: 0 }} className="mb-5 rounded-2xl border border-emerald-200 bg-emerald-50 px-4 py-3 text-sm font-medium text-emerald-700 shadow-sm">
            {toast}
          </motion.div>
        )}

        <motion.div initial={{ opacity: 0, y: 12 }} animate={{ opacity: 1, y: 0 }} className="mb-8 grid gap-4 md:grid-cols-4">
          {metrics.map((item) => {
            const Icon = item.icon;
            return (
              <Card key={item.label} className="rounded-3xl border-0 shadow-sm">
                <CardContent className="flex items-center justify-between p-6">
                  <div>
                    <p className="text-sm text-slate-500">{item.label}</p>
                    <p className="mt-2 text-3xl font-bold tracking-tight">{item.value}</p>
                  </div>
                  <div className="rounded-2xl bg-slate-100 p-3">
                    <Icon className="h-5 w-5 text-slate-700" />
                  </div>
                </CardContent>
              </Card>
            );
          })}
        </motion.div>

        <Tabs defaultValue="workbench" className="space-y-6">
          <TabsList className="grid w-full max-w-2xl grid-cols-4 rounded-2xl bg-white p-1 shadow-sm">
            <TabsTrigger value="workbench" className="rounded-xl">Denial Workbench</TabsTrigger>
            <TabsTrigger value="uploader" className="rounded-xl">Upload Demo Case</TabsTrigger>
            <TabsTrigger value="pipeline" className="rounded-xl">Workflow</TabsTrigger>
            <TabsTrigger value="architecture" className="rounded-xl">Architecture</TabsTrigger>
          </TabsList>

          <TabsContent value="workbench" className="space-y-6">
            <div className="grid gap-6 xl:grid-cols-[1.05fr_1.45fr]">
              <Card className="rounded-3xl border-0 shadow-sm">
                <CardHeader>
                  <div className="flex flex-col gap-4 lg:flex-row lg:items-center lg:justify-between">
                    <div>
                      <CardTitle className="text-xl">Case Queue</CardTitle>
                      <CardDescription>Pick a denial, review support, then approve, escalate, or mark paid.</CardDescription>
                    </div>
                    <Badge className="rounded-full bg-slate-100 px-3 py-1 text-slate-700">{filteredCases.length} Visible</Badge>
                  </div>
                  <div className="grid gap-3 pt-3 md:grid-cols-[1fr_180px]">
                    <div className="relative">
                      <Search className="absolute left-3 top-1/2 h-4 w-4 -translate-y-1/2 text-slate-400" />
                      <Input value={query} onChange={(e) => setQuery(e.target.value)} placeholder="Search payer, denial, specialty, or case ID" className="rounded-xl pl-10" />
                    </div>
                    <Select value={statusFilter} onValueChange={setStatusFilter}>
                      <SelectTrigger className="rounded-xl">
                        <Filter className="mr-2 h-4 w-4 text-slate-500" />
                        <SelectValue placeholder="Filter status" />
                      </SelectTrigger>
                      <SelectContent>
                        <SelectItem value="all">All Statuses</SelectItem>
                        <SelectItem value="New">New</SelectItem>
                        <SelectItem value="Ready for Review">Ready for Review</SelectItem>
                        <SelectItem value="Needs Documents">Needs Documents</SelectItem>
                        <SelectItem value="Escalated">Escalated</SelectItem>
                        <SelectItem value="Approved">Approved</SelectItem>
                        <SelectItem value="Submitted">Submitted</SelectItem>
                        <SelectItem value="Paid">Paid</SelectItem>
                      </SelectContent>
                    </Select>
                  </div>
                </CardHeader>
                <CardContent>
                  <div className="overflow-hidden rounded-2xl border border-slate-200">
                    <Table>
                      <TableHeader>
                        <TableRow>
                          <TableHead>Case</TableHead>
                          <TableHead>Status</TableHead>
                          <TableHead>Amount</TableHead>
                          <TableHead>Confidence</TableHead>
                        </TableRow>
                      </TableHeader>
                      <TableBody>
                        {filteredCases.map((item) => (
                          <TableRow key={item.id} className={`cursor-pointer ${selected?.id === item.id ? "bg-slate-50" : ""}`} onClick={() => { setSelectedId(item.id); setDraft(item.draft); }}>
                            <TableCell>
                              <div className="font-semibold">{item.id}</div>
                              <div className="text-xs text-slate-500">{item.payer} • {item.category}</div>
                            </TableCell>
                            <TableCell><StatusBadge value={item.status} /></TableCell>
                            <TableCell>${item.claimAmount.toLocaleString()}</TableCell>
                            <TableCell>{item.confidence}%</TableCell>
                          </TableRow>
                        ))}
                      </TableBody>
                    </Table>
                  </div>
                </CardContent>
              </Card>

              <div className="space-y-6">
                <Card className="rounded-3xl border-0 shadow-sm">
                  <CardHeader>
                    <div className="flex flex-col gap-4 lg:flex-row lg:items-center lg:justify-between">
                      <div>
                        <CardTitle className="text-xl">Case Review</CardTitle>
                        <CardDescription>{selected.id} • {selected.payer} • {selected.specialty}</CardDescription>
                      </div>
                      <StatusBadge value={selected.status} />
                    </div>
                  </CardHeader>
                  <CardContent className="space-y-6">
                    <div className="grid gap-4 md:grid-cols-4">
                      <MetricCard icon={Brain} label="Denial Category" value={selected.category} />
                      <MetricCard icon={Activity} label="Confidence" value={`${selected.confidence}%`} />
                      <MetricCard icon={DollarSign} label="Claim Amount" value={`$${selected.claimAmount.toLocaleString()}`} />
                      <MetricCard icon={Clock3} label="Deadline" value={selected.deadline} />
                    </div>

                    <div className="grid gap-6 lg:grid-cols-[1fr_340px]">
                      <div className="rounded-2xl border border-slate-200 bg-white p-5">
                        <div className="mb-2 text-sm font-semibold text-slate-500">System Summary</div>
                        <p className="text-sm leading-7 text-slate-700">{selected.summary}</p>
                        <div className="mt-5 flex flex-wrap gap-2">
                          <Badge className="rounded-full bg-sky-50 text-sky-700">Action: {selected.recommendedAction}</Badge>
                          <Badge className="rounded-full bg-emerald-50 text-emerald-700">Evidence Strength: {selected.evidenceStrength}%</Badge>
                          <Badge className="rounded-full bg-slate-100 text-slate-700">Route: {selected.route}</Badge>
                          <Badge className={`rounded-full ${selected.workflowRisk === "High" ? "bg-rose-50 text-rose-700" : selected.workflowRisk === "Medium" ? "bg-amber-50 text-amber-700" : "bg-emerald-50 text-emerald-700"}`}>Risk: {selected.workflowRisk}</Badge>
                        </div>
                      </div>
                      <div className="rounded-2xl border border-slate-200 bg-white p-5">
                        <div className="mb-3 text-sm font-semibold text-slate-500">Automation Safety Gate</div>
                        <div className="space-y-4">
                          <RiskRow label="Classification Confidence" value={selected.confidence} />
                          <RiskRow label="Evidence Strength" value={selected.evidenceStrength} />
                          <RiskRow label="Workflow Certainty" value={selected.workflowRisk === "High" ? 38 : selected.workflowRisk === "Medium" ? 67 : 88} />
                          <RiskRow label="Compliance Risk" value={selected.workflowRisk === "High" ? 72 : selected.workflowRisk === "Medium" ? 41 : 18} inverse />
                        </div>
                      </div>
                    </div>
                  </CardContent>
                </Card>

                <div className="grid gap-6 lg:grid-cols-2">
                  <Card className="rounded-3xl border-0 shadow-sm">
                    <CardHeader>
                      <CardTitle className="text-lg">Evidence Packet</CardTitle>
                      <CardDescription>Verified support the reviewer can inspect before approving.</CardDescription>
                    </CardHeader>
                    <CardContent className="space-y-4">
                      {selected.evidence.map((item, idx) => (
                        <div key={idx} className="rounded-2xl border border-slate-200 p-4">
                          <div className="mb-2 flex items-center justify-between gap-3">
                            <div className="font-semibold text-slate-900">{item.label}</div>
                            <Badge className="rounded-full bg-slate-100 text-slate-700">Relevance {item.relevance}%</Badge>
                          </div>
                          <p className="text-sm leading-6 text-slate-700">“{item.excerpt}”</p>
                        </div>
                      ))}
                    </CardContent>
                  </Card>

                  <Card className="rounded-3xl border-0 shadow-sm">
                    <CardHeader>
                      <CardTitle className="text-lg">Draft Work Product</CardTitle>
                      <CardDescription>Edit the draft, then change state with the action buttons.</CardDescription>
                    </CardHeader>
                    <CardContent className="space-y-4">
                      <Textarea value={draft} onChange={(e) => setDraft(e.target.value)} className="min-h-[250px] rounded-2xl" />
                      <Textarea value={notes} onChange={(e) => setNotes(e.target.value)} className="min-h-[90px] rounded-2xl" placeholder="Reviewer notes" />
                      <div className="grid gap-3 md:grid-cols-2">
                        <Button className="rounded-xl bg-emerald-600 hover:bg-emerald-700" onClick={() => updateCaseStatus("Approved")}>
                          <CheckCircle2 className="mr-2 h-4 w-4" /> Approve
                        </Button>
                        <Button variant="outline" className="rounded-xl" onClick={() => updateCaseStatus("Escalated")}>
                          <AlertTriangle className="mr-2 h-4 w-4" /> Escalate
                        </Button>
                        <Button variant="outline" className="rounded-xl" onClick={() => updateCaseStatus("Submitted")}>
                          <ArrowRight className="mr-2 h-4 w-4" /> Mark Submitted
                        </Button>
                        <Button variant="outline" className="rounded-xl" onClick={() => updateCaseStatus("Paid")}>
                          <DollarSign className="mr-2 h-4 w-4" /> Mark Paid
                        </Button>
                      </div>
                      <div className="flex flex-wrap gap-3">
                        <Button variant="outline" className="rounded-xl">
                          <Download className="mr-2 h-4 w-4" /> Export Packet
                        </Button>
                        <Button variant="outline" className="rounded-xl" onClick={() => setDraft(selected.draft)}>
                          <RefreshCw className="mr-2 h-4 w-4" /> Reset Draft
                        </Button>
                      </div>
                    </CardContent>
                  </Card>
                </div>
              </div>
            </div>
          </TabsContent>

          <TabsContent value="uploader" className="space-y-6">
            <div className="grid gap-6 xl:grid-cols-[0.95fr_1.05fr]">
              <Card className="rounded-3xl border-0 shadow-sm">
                <CardHeader>
                  <CardTitle>Upload Demo Case</CardTitle>
                  <CardDescription>This simulates how a billing team would open a new denial case in version 1.</CardDescription>
                </CardHeader>
                <CardContent className="space-y-4">
                  <div className="grid gap-4 md:grid-cols-2">
                    <Input className="rounded-xl" placeholder="Patient name" value={uploadForm.patient} onChange={(e) => setUploadForm({ ...uploadForm, patient: e.target.value })} />
                    <Input className="rounded-xl" placeholder="Payer" value={uploadForm.payer} onChange={(e) => setUploadForm({ ...uploadForm, payer: e.target.value })} />
                    <Input className="rounded-xl" placeholder="Claim amount" value={uploadForm.claimAmount} onChange={(e) => setUploadForm({ ...uploadForm, claimAmount: e.target.value })} />
                    <Input className="rounded-xl" placeholder="Specialty" value={uploadForm.specialty} onChange={(e) => setUploadForm({ ...uploadForm, specialty: e.target.value })} />
                  </div>
                  <Select value={uploadForm.caseType} onValueChange={(value) => setUploadForm({ ...uploadForm, caseType: value })}>
                    <SelectTrigger className="rounded-xl">
                      <SelectValue placeholder="Select denial pattern" />
                    </SelectTrigger>
                    <SelectContent>
                      <SelectItem value="medical">Medical Necessity</SelectItem>
                      <SelectItem value="docs">Missing Documentation</SelectItem>
                      <SelectItem value="auth">Authorization Issue</SelectItem>
                    </SelectContent>
                  </Select>
                  <div className="rounded-2xl border border-dashed border-slate-300 bg-slate-50 p-6 text-center">
                    <Upload className="mx-auto mb-3 h-8 w-8 text-slate-500" />
                    <div className="font-medium">Simulated denial and chart upload</div>
                    <div className="mt-1 text-sm text-slate-500">In the real app, this is where 835 files and chart PDFs would be added.</div>
                  </div>
                  <Button className="w-full rounded-xl bg-slate-900 hover:bg-slate-800" onClick={addMockCase}>
                    <PlusCircle className="mr-2 h-4 w-4" /> Add Demo Case to Queue
                  </Button>
                </CardContent>
              </Card>

              <Card className="rounded-3xl border-0 shadow-sm">
                <CardHeader>
                  <CardTitle>What this demo proves</CardTitle>
                  <CardDescription>The product does not need full EHR integration to become usable in a first pilot.</CardDescription>
                </CardHeader>
                <CardContent className="space-y-4">
                  {[
                    ["Users can create a denial case", "Start with manual uploads and clean workflow states."],
                    ["The system can classify and route", "Rules-first logic determines the safest next action."],
                    ["Evidence can be assembled", "Support is shown transparently instead of hidden in a black box."],
                    ["A reviewer can approve or escalate", "Human oversight keeps the MVP safe and usable."],
                  ].map(([title, text]) => (
                    <div key={title} className="rounded-2xl border border-slate-200 p-4">
                      <div className="font-semibold text-slate-900">{title}</div>
                      <div className="mt-1 text-sm leading-6 text-slate-600">{text}</div>
                    </div>
                  ))}
                </CardContent>
              </Card>
            </div>
          </TabsContent>

          <TabsContent value="pipeline" className="space-y-6">
            <div className="grid gap-6 lg:grid-cols-5">
              {[
                { title: "1. Ingest", text: "Upload 835s, CSVs, remittance exports, and chart PDFs.", icon: Upload },
                { title: "2. Normalize", text: "Standardize denial codes, dates, documents, and case state.", icon: FileText },
                { title: "3. Route", text: "Apply deterministic payer and workflow rules before AI drafting.", icon: ArrowRight },
                { title: "4. Review", text: "Present evidence-backed drafts to billing teams for approval.", icon: UserCheck },
                { title: "5. Track", text: "Log outcomes, recovered revenue, and reviewer edits for learning.", icon: Activity },
              ].map((step) => {
                const Icon = step.icon;
                return (
                  <Card key={step.title} className="rounded-3xl border-0 shadow-sm">
                    <CardContent className="p-6">
                      <div className="mb-4 flex h-12 w-12 items-center justify-center rounded-2xl bg-slate-100">
                        <Icon className="h-5 w-5 text-slate-800" />
                      </div>
                      <div className="mb-2 text-lg font-bold tracking-tight">{step.title}</div>
                      <p className="text-sm leading-6 text-slate-600">{step.text}</p>
                    </CardContent>
                  </Card>
                );
              })}
            </div>

            <Card className="rounded-3xl border-0 shadow-sm">
              <CardHeader>
                <CardTitle>Production-Safe Workflow State Model</CardTitle>
                <CardDescription>The first live version should be a reviewer-centered denial workbench, not a blind auto-submission engine.</CardDescription>
              </CardHeader>
              <CardContent>
                <div className="grid gap-4 md:grid-cols-3 xl:grid-cols-6">
                  {["NEW", "INGESTING", "READY FOR REVIEW", "APPROVED", "SUBMITTED", "PAID"].map((state, i) => (
                    <div key={state} className="rounded-2xl border border-slate-200 bg-white p-4 text-center">
                      <div className="text-xs font-semibold uppercase tracking-[0.18em] text-slate-400">Stage {i + 1}</div>
                      <div className="mt-2 text-sm font-bold text-slate-900">{state}</div>
                    </div>
                  ))}
                </div>
              </CardContent>
            </Card>
          </TabsContent>

          <TabsContent value="architecture" className="space-y-6">
            <div className="grid gap-6 xl:grid-cols-[1fr_1fr]">
              <Card className="rounded-3xl border-0 shadow-sm">
                <CardHeader>
                  <CardTitle>Program Architecture</CardTitle>
                  <CardDescription>What you would build behind this demo to make it truly live.</CardDescription>
                </CardHeader>
                <CardContent className="space-y-4 text-sm leading-7 text-slate-700">
                  <ArchitectureRow title="Frontend" value="Next.js + TypeScript + Tailwind + shadcn/ui" />
                  <ArchitectureRow title="Backend" value="FastAPI service for cases, workflows, drafts, evidence, and audit logs" />
                  <ArchitectureRow title="Database" value="PostgreSQL for organizations, claims, denial cases, evidence, drafts, and outcomes" />
                  <ArchitectureRow title="Storage" value="S3-compatible object storage for uploads, chart packets, and exports" />
                  <ArchitectureRow title="Workers" value="Redis + Celery for parsing, retrieval, and draft generation jobs" />
                  <ArchitectureRow title="AI Layer" value="Rules engine first, retrieval second, LLM drafting third, human review when needed" />
                </CardContent>
              </Card>

              <Card className="rounded-3xl border-0 shadow-sm">
                <CardHeader>
                  <CardTitle>MVP Build Order</CardTitle>
                  <CardDescription>Smallest scope that still proves value with real billing teams.</CardDescription>
                </CardHeader>
                <CardContent className="space-y-5">
                  {[
                    ["1", "Authentication + organizations", "Secure login, organization switching, and roles."],
                    ["2", "Case ingestion", "Manual upload of denial files and chart docs."],
                    ["3", "Rules-based triage", "Map denial categories and deadlines using auditable logic."],
                    ["4", "Evidence retrieval", "Extract and rank support from uploaded chart documents."],
                    ["5", "Draft generator", "Produce appeal drafts grounded in evidence only."],
                    ["6", "Reviewer workflow", "Approve, edit, reject, escalate, export."],
                  ].map(([num, title, text]) => (
                    <div key={num} className="flex gap-4 rounded-2xl border border-slate-200 p-4">
                      <div className="flex h-10 w-10 shrink-0 items-center justify-center rounded-2xl bg-slate-900 text-sm font-bold text-white">{num}</div>
                      <div>
                        <div className="font-semibold text-slate-900">{title}</div>
                        <div className="text-sm leading-6 text-slate-600">{text}</div>
                      </div>
                    </div>
                  ))}
                </CardContent>
              </Card>
            </div>
          </TabsContent>
        </Tabs>
      </div>
    </div>
  );
}

function MetricCard({ icon: Icon, label, value }) {
  return (
    <div className="rounded-2xl border border-slate-200 bg-white p-4">
      <div className="mb-3 flex h-10 w-10 items-center justify-center rounded-2xl bg-slate-100">
        <Icon className="h-4 w-4 text-slate-800" />
      </div>
      <div className="text-xs font-semibold uppercase tracking-[0.18em] text-slate-400">{label}</div>
      <div className="mt-2 text-sm font-bold text-slate-900">{value}</div>
    </div>
  );
}

function RiskRow({ label, value, inverse = false }) {
  const display = Math.max(0, Math.min(100, value));
  const tone = inverse ? (display > 60 ? "text-rose-600" : "text-emerald-600") : display > 75 ? "text-emerald-600" : display > 55 ? "text-amber-600" : "text-rose-600";
  return (
    <div>
      <div className="mb-2 flex items-center justify-between text-sm">
        <span className="text-slate-600">{label}</span>
        <span className={`font-semibold ${tone}`}>{display}%</span>
      </div>
      <Progress value={display} className="h-2" />
    </div>
  );
}

function ArchitectureRow({ title, value }) {
  return (
    <div className="rounded-2xl border border-slate-200 p-4">
      <div className="mb-1 text-sm font-semibold text-slate-900">{title}</div>
      <div className="text-sm text-slate-600">{value}</div>
    </div>
  );
}
