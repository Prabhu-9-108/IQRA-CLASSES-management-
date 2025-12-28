
import React, { useState, useEffect, useMemo } from 'react';
import { 
  LayoutDashboard, Users, BookOpen, Wallet, CreditCard, 
  Briefcase, Monitor, BarChart3, Settings, Plus, Trash2, 
  Save, Download, Search, Menu, X, ChevronRight, FileText,
  Sparkles, MessageSquare, Copy, Loader2, XCircle, BrainCircuit, ScrollText
} from 'lucide-react';

// Firebase Imports
import { initializeApp } from 'firebase/app';
import { 
  getAuth, 
  signInAnonymously, 
  onAuthStateChanged,
  signInWithCustomToken 
} from 'firebase/auth';
import { 
  getFirestore, 
  collection, 
  addDoc, 
  deleteDoc, 
  updateDoc, 
  doc, 
  onSnapshot, 
  query, 
  orderBy, 
  serverTimestamp 
} from 'firebase/firestore';

// --- Firebase Initialization ---
const firebaseConfig = JSON.parse(__firebase_config);
const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getFirestore(app);
const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-iqra-app';

// --- Gemini AI Helper ---
const callGemini = async (prompt) => {
  const apiKey = ""; // Runtime provided
  try {
    const response = await fetch(
      `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`,
      {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ contents: [{ parts: [{ text: prompt }] }] })
      }
    );
    if (!response.ok) throw new Error('AI Call failed');
    const data = await response.json();
    return data.candidates?.[0]?.content?.parts?.[0]?.text || "No response generated.";
  } catch (error) {
    console.error("Gemini API Error:", error);
    return "Sorry, I couldn't generate a response at this moment. Please try again.";
  }
};

// --- Components ---

// 1. Generic UI Components
const Card = ({ children, className = "" }) => (
  <div className={`bg-white rounded-xl shadow-sm border border-slate-200 p-6 ${className}`}>
    {children}
  </div>
);

const Button = ({ children, onClick, variant = 'primary', className = "", disabled = false }) => {
  const baseStyle = "px-4 py-2 rounded-lg font-medium transition-all duration-200 flex items-center justify-center gap-2 text-sm";
  const variants = {
    primary: "bg-[#2180a5] hover:bg-[#1c6a8b] text-white shadow-md shadow-[#2180a5]/20",
    danger: "bg-red-50 text-red-600 hover:bg-red-100 border border-red-200",
    secondary: "bg-slate-100 text-slate-700 hover:bg-slate-200",
    outline: "border border-slate-300 text-slate-600 hover:bg-slate-50",
    magic: "bg-gradient-to-r from-violet-600 to-indigo-600 text-white shadow-lg shadow-indigo-500/30 hover:shadow-indigo-500/40 border-none"
  };
  return (
    <button 
      onClick={onClick} 
      disabled={disabled}
      className={`${baseStyle} ${variants[variant]} ${disabled ? 'opacity-50 cursor-not-allowed' : ''} ${className}`}
    >
      {children}
    </button>
  );
};

const Input = ({ label, ...props }) => (
  <div className="mb-4">
    <label className="block text-xs font-semibold text-slate-600 uppercase tracking-wider mb-1.5">{label}</label>
    <input 
      className="w-full px-3 py-2 bg-slate-50 border border-slate-200 rounded-lg focus:outline-none focus:ring-2 focus:ring-[#2180a5]/20 focus:border-[#2180a5] transition-all text-sm"
      {...props}
    />
  </div>
);

const Select = ({ label, children, ...props }) => (
  <div className="mb-4">
    <label className="block text-xs font-semibold text-slate-600 uppercase tracking-wider mb-1.5">{label}</label>
    <select 
      className="w-full px-3 py-2 bg-slate-50 border border-slate-200 rounded-lg focus:outline-none focus:ring-2 focus:ring-[#2180a5]/20 focus:border-[#2180a5] transition-all text-sm"
      {...props}
    >
      {children}
    </select>
  </div>
);

const StatCard = ({ title, value, icon: Icon, colorClass, subtext }) => (
  <div className="bg-white p-5 rounded-xl border border-slate-100 shadow-sm hover:shadow-md transition-shadow">
    <div className="flex justify-between items-start">
      <div>
        <p className="text-xs font-bold text-slate-500 uppercase tracking-wider mb-1">{title}</p>
        <h3 className={`text-2xl font-bold ${colorClass}`}>{value}</h3>
        {subtext && <p className="text-xs text-slate-400 mt-1">{subtext}</p>}
      </div>
      <div className={`p-2 rounded-lg ${colorClass.replace('text-', 'bg-').replace('600', '100')} ${colorClass}`}>
        <Icon size={20} />
      </div>
    </div>
  </div>
);

// Generic Modal Component (Closes on backdrop click or Escape)
const Modal = ({ isOpen, onClose, title, children, icon: Icon, iconColor = "text-[#2180a5]" }) => {
  useEffect(() => {
    const handleEsc = (e) => {
      if (e.key === 'Escape') onClose();
    };
    if (isOpen) window.addEventListener('keydown', handleEsc);
    return () => window.removeEventListener('keydown', handleEsc);
  }, [isOpen, onClose]);

  if (!isOpen) return null;

  return (
    <div 
      className="fixed inset-0 bg-black/60 z-50 flex items-center justify-center p-4 backdrop-blur-sm animate-in fade-in duration-200"
      onClick={(e) => {
        if (e.target === e.currentTarget) onClose();
      }}
    >
      <div className="bg-white rounded-xl shadow-2xl max-w-lg w-full p-6 animate-in zoom-in-95 duration-200 flex flex-col max-h-[90vh]">
        <div className="flex justify-between items-start mb-4 shrink-0">
          <div>
            <h3 className="text-lg font-bold text-slate-800 flex items-center gap-2">
              {Icon && <Icon size={20} className={iconColor} />}
              {title}
            </h3>
          </div>
          <button onClick={onClose} className="text-slate-400 hover:text-slate-600 p-1 rounded-full hover:bg-slate-100 transition-colors">
            <XCircle size={24}/>
          </button>
        </div>
        <div className="overflow-y-auto custom-scrollbar">
          {children}
        </div>
      </div>
    </div>
  );
};

// --- Feature Components ---

const Dashboard = ({ data }) => {
  const { fees, expenses, students, batches, staff } = data;
  const [aiInsight, setAiInsight] = useState(null);
  const [isAnalyzing, setIsAnalyzing] = useState(false);

  const totalFees = fees.reduce((sum, f) => sum + Number(f.amount || 0), 0);
  const totalExpenses = expenses.reduce((sum, e) => sum + Number(e.amount || 0), 0);
  const netProfit = totalFees - totalExpenses;

  // Recent transactions (Fees + Expenses mixed)
  const recentTransactions = [
    ...fees.map(f => ({ ...f, type: 'Fee', dateObj: new Date(f.date) })),
    ...expenses.map(e => ({ ...e, type: 'Expense', dateObj: new Date(e.date) }))
  ].sort((a, b) => b.dateObj - a.dateObj).slice(0, 5);

  const handleAnalyze = async () => {
    setIsAnalyzing(true);
    const prompt = `
      Act as a financial advisor for a coaching institute called 'Iqra Classes'.
      Analyze this data and provide a 2-3 sentence executive summary of the financial health.
      Be encouraging but realistic.
      
      Data:
      - Total Fees Collected: ₹${totalFees}
      - Total Expenses: ₹${totalExpenses}
      - Net Profit: ₹${netProfit}
      - Active Students: ${students.length}
      - Active Batches: ${batches.length}
      - Staff Count: ${staff.length}
    `;
    const result = await callGemini(prompt);
    setAiInsight(result);
    setIsAnalyzing(false);
  };

  return (
    <div className="space-y-6">
      {/* AI Header Section */}
      <div className="flex flex-col sm:flex-row justify-between items-center bg-gradient-to-r from-slate-800 to-slate-900 p-6 rounded-2xl text-white shadow-lg gap-4">
        <div>
          <h2 className="text-xl font-bold mb-1">Financial Overview</h2>
          <p className="text-slate-400 text-sm">Track your institute's performance</p>
        </div>
        <Button variant="magic" onClick={handleAnalyze} disabled={isAnalyzing}>
          {isAnalyzing ? <Loader2 className="animate-spin" size={16}/> : <Sparkles size={16}/>}
          {isAnalyzing ? 'Analyzing...' : 'AI Analysis'}
        </Button>
      </div>

      {/* AI Insight Card */}
      {aiInsight && (
        <div className="bg-gradient-to-r from-indigo-50 to-violet-50 border border-indigo-100 p-6 rounded-xl relative animate-in fade-in slide-in-from-top-4 duration-500">
          <button 
            onClick={() => setAiInsight(null)}
            className="absolute top-4 right-4 text-indigo-400 hover:text-indigo-600"
          >
            <XCircle size={20} />
          </button>
          <div className="flex gap-4">
            <div className="bg-white p-3 rounded-full h-fit shadow-sm text-indigo-600 hidden sm:block">
              <Sparkles size={24} />
            </div>
            <div>
              <h3 className="font-bold text-indigo-900 mb-2">AI Smart Insight</h3>
              <p className="text-indigo-800 leading-relaxed text-sm md:text-base">{aiInsight}</p>
            </div>
          </div>
        </div>
      )}

      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
        <StatCard 
          title="Total Collection" 
          value={`₹${totalFees.toLocaleString()}`} 
          icon={Wallet} 
          colorClass="text-emerald-600" 
        />
        <StatCard 
          title="Total Expenses" 
          value={`₹${totalExpenses.toLocaleString()}`} 
          icon={CreditCard} 
          colorClass="text-rose-600" 
        />
        <StatCard 
          title="Net Profit" 
          value={`₹${netProfit.toLocaleString()}`} 
          icon={BarChart3} 
          colorClass="text-[#2180a5]" 
        />
        <StatCard 
          title="Active Students" 
          value={students.length} 
          icon={Users} 
          colorClass="text-indigo-600" 
          subtext={`${batches.length} Active Batches`}
        />
      </div>

      <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
        <Card className="lg:col-span-2">
          <div className="flex justify-between items-center mb-6">
            <h2 className="text-lg font-bold text-slate-800">Recent Transactions</h2>
            <Button variant="outline" className="text-xs py-1 px-2">View All</Button>
          </div>
          <div className="overflow-x-auto">
            <table className="w-full text-sm text-left">
              <thead className="bg-slate-50 text-slate-600 font-semibold border-b border-slate-200">
                <tr>
                  <th className="p-3">Date</th>
                  <th className="p-3">Description</th>
                  <th className="p-3">Type</th>
                  <th className="p-3 text-right">Amount</th>
                </tr>
              </thead>
              <tbody className="divide-y divide-slate-100">
                {recentTransactions.map((t, idx) => (
                  <tr key={idx} className="hover:bg-slate-50">
                    <td className="p-3 text-slate-500">{t.date}</td>
                    <td className="p-3 font-medium text-slate-700">
                      {t.type === 'Fee' ? `${t.studentName} (${t.month})` : t.description}
                    </td>
                    <td className="p-3">
                      <span className={`px-2 py-1 rounded-full text-xs font-medium ${
                        t.type === 'Fee' ? 'bg-emerald-100 text-emerald-700' : 'bg-rose-100 text-rose-700'
                      }`}>
                        {t.type}
                      </span>
                    </td>
                    <td className={`p-3 text-right font-bold ${
                      t.type === 'Fee' ? 'text-emerald-600' : 'text-rose-600'
                    }`}>
                      {t.type === 'Fee' ? '+' : '-'}₹{Number(t.amount).toLocaleString()}
                    </td>
                  </tr>
                ))}
                {recentTransactions.length === 0 && (
                  <tr><td colSpan="4" className="p-8 text-center text-slate-400">No transactions recorded yet</td></tr>
                )}
              </tbody>
            </table>
          </div>
        </Card>

        <Card>
          <h2 className="text-lg font-bold text-slate-800 mb-4">Quick Stats</h2>
          <div className="space-y-4">
            <div className="flex justify-between items-center p-3 bg-slate-50 rounded-lg">
              <div className="flex items-center gap-3">
                <div className="bg-blue-100 p-2 rounded text-blue-600"><Users size={16} /></div>
                <span className="text-sm font-medium text-slate-600">Total Staff</span>
              </div>
              <span className="font-bold text-slate-800">{staff.length}</span>
            </div>
            <div className="flex justify-between items-center p-3 bg-slate-50 rounded-lg">
              <div className="flex items-center gap-3">
                <div className="bg-purple-100 p-2 rounded text-purple-600"><BookOpen size={16} /></div>
                <span className="text-sm font-medium text-slate-600">Total Batches</span>
              </div>
              <span className="font-bold text-slate-800">{batches.length}</span>
            </div>
          </div>
        </Card>
      </div>
    </div>
  );
};

const Batches = ({ data, onAdd, onDelete }) => {
  const [form, setForm] = useState({ name: '', grade: '', schedule: '', fee: '', teacher: '' });
  const [syllabusData, setSyllabusData] = useState(null);
  const [loadingBatchId, setLoadingBatchId] = useState(null);

  const handleSubmit = (e) => {
    e.preventDefault();
    if (!form.name || !form.grade) return;
    onAdd('batches', { ...form, createdAt: serverTimestamp() });
    setForm({ name: '', grade: '', schedule: '', fee: '', teacher: '' });
  };

  const handleGenerateSyllabus = async (batch) => {
    setLoadingBatchId(batch.id);
    const prompt = `
      Create a simplified 4-week study syllabus for a batch.
      
      Details:
      - Batch Name: ${batch.name}
      - Grade/Class: ${batch.grade}
      
      Output format: plain text list with Week 1, Week 2, Week 3, Week 4 headings. Keep it concise.
    `;
    const result = await callGemini(prompt);
    setSyllabusData({ batchName: batch.name, text: result });
    setLoadingBatchId(null);
  };

  return (
    <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
      <Modal 
        isOpen={!!syllabusData} 
        onClose={() => setSyllabusData(null)} 
        title={`Syllabus Plan: ${syllabusData?.batchName}`}
        icon={BrainCircuit}
        iconColor="text-violet-600"
      >
        <div className="bg-slate-50 p-4 rounded-lg text-sm text-slate-700 whitespace-pre-wrap leading-relaxed border border-slate-200">
          {syllabusData?.text}
        </div>
        <div className="mt-4">
          <Button className="w-full" onClick={() => { navigator.clipboard.writeText(syllabusData.text); setSyllabusData(null); }}>
            <Copy size={16} /> Copy to Clipboard
          </Button>
        </div>
      </Modal>

      <Card className="lg:col-span-1 h-fit">
        <h2 className="text-lg font-bold text-slate-800 mb-4 flex items-center gap-2">
          <Plus size={20} className="text-[#2180a5]" /> New Batch
        </h2>
        <form onSubmit={handleSubmit}>
          <Input label="Batch Name" placeholder="e.g. 10th Science" value={form.name} onChange={e => setForm({...form, name: e.target.value})} required />
          <Input label="Class/Grade" placeholder="e.g. 10" value={form.grade} onChange={e => setForm({...form, grade: e.target.value})} required />
          <Input label="Schedule" placeholder="Mon-Fri 4PM" value={form.schedule} onChange={e => setForm({...form, schedule: e.target.value})} />
          <Input label="Monthly Fee (₹)" type="number" value={form.fee} onChange={e => setForm({...form, fee: e.target.value})} />
          <Select label="Teacher" value={form.teacher} onChange={e => setForm({...form, teacher: e.target.value})}>
            <option value="">Select Teacher</option>
            {data.staff.filter(s => s.position === 'Teacher').map(s => (
              <option key={s.id} value={s.name}>{s.name}</option>
            ))}
          </Select>
          <Button className="w-full">Create Batch</Button>
        </form>
      </Card>

      <Card className="lg:col-span-2">
        <h2 className="text-lg font-bold text-slate-800 mb-4">All Batches</h2>
        <div className="overflow-x-auto">
          <table className="w-full text-sm text-left">
            <thead className="bg-slate-50 text-slate-600 font-semibold border-b border-slate-200">
              <tr>
                <th className="p-3">Name</th>
                <th className="p-3">Class</th>
                <th className="p-3">Teacher</th>
                <th className="p-3">Fee</th>
                <th className="p-3 text-right">Actions</th>
              </tr>
            </thead>
            <tbody className="divide-y divide-slate-100">
              {data.batches.map(batch => (
                <tr key={batch.id} className="hover:bg-slate-50">
                  <td className="p-3 font-medium text-slate-800">{batch.name}</td>
                  <td className="p-3 text-slate-600">{batch.grade}</td>
                  <td className="p-3 text-slate-600">{batch.teacher || '-'}</td>
                  <td className="p-3 text-emerald-600 font-medium">₹{batch.fee}</td>
                  <td className="p-3 text-right flex justify-end gap-2">
                    <button 
                      onClick={() => handleGenerateSyllabus(batch)}
                      className="text-violet-600 hover:bg-violet-50 p-1.5 rounded-md transition-colors"
                      title="Generate Syllabus"
                    >
                      {loadingBatchId === batch.id ? <Loader2 size={16} className="animate-spin"/> : <BrainCircuit size={16} />}
                    </button>
                    <button onClick={() => onDelete('batches', batch.id)} className="text-rose-500 hover:text-rose-700 p-1.5 hover:bg-rose-50 rounded-md">
                      <Trash2 size={16} />
                    </button>
                  </td>
                </tr>
              ))}
              {data.batches.length === 0 && <tr><td colSpan="5" className="p-6 text-center text-slate-400">No batches found</td></tr>}
            </tbody>
          </table>
        </div>
      </Card>
    </div>
  );
};

const Students = ({ data, onAdd, onDelete }) => {
  const [form, setForm] = useState({ name: '', roll: '', phone: '', batchId: '', joinDate: new Date().toISOString().split('T')[0] });
  const [draftingId, setDraftingId] = useState(null);
  const [draftedMessage, setDraftedMessage] = useState(null);

  const handleSubmit = (e) => {
    e.preventDefault();
    if (!form.name || !form.roll) return;
    onAdd('students', { ...form, createdAt: serverTimestamp() });
    setForm({ name: '', roll: '', phone: '', batchId: '', joinDate: new Date().toISOString().split('T')[0] });
  };

  const handleDraftMessage = async (student) => {
    setDraftingId(student.id);
    const batch = data.batches.find(b => b.id === student.batchId);
    const prompt = `
      Draft a polite, professional, and short WhatsApp message to the parents of a student.
      
      Details:
      - Student Name: ${student.name}
      - Student Roll: ${student.roll}
      - Batch/Class: ${batch ? batch.name : 'Class'}
      - Context: Just a general update that the student is attending classes regularly and we appreciate their effort. 
      - Tone: Warm, professional, encouraging.
      - Signature: Iqra Classes Administration.
      
      Don't include subject lines. Just the message body.
    `;
    
    const message = await callGemini(prompt);
    setDraftedMessage({ studentName: student.name, text: message });
    setDraftingId(null);
  };

  return (
    <div className="grid grid-cols-1 lg:grid-cols-3 gap-6 relative">
      {/* AI Message Modal */}
      <Modal
        isOpen={!!draftedMessage}
        onClose={() => setDraftedMessage(null)}
        title="AI Drafted Message"
        icon={MessageSquare}
        iconColor="text-violet-600"
      >
        <div className="bg-slate-50 p-4 rounded-lg border border-slate-200 text-sm text-slate-700 whitespace-pre-wrap leading-relaxed">
          {draftedMessage?.text}
        </div>
        <div className="mt-4">
          <Button className="w-full" onClick={() => {
            navigator.clipboard.writeText(draftedMessage.text);
            alert('Message copied to clipboard!');
            setDraftedMessage(null);
          }}>
            <Copy size={16} /> Copy to Clipboard
          </Button>
        </div>
      </Modal>

      <Card className="lg:col-span-1 h-fit">
        <h2 className="text-lg font-bold text-slate-800 mb-4 flex items-center gap-2">
          <Plus size={20} className="text-[#2180a5]" /> Add Student
        </h2>
        <form onSubmit={handleSubmit}>
          <Input label="Roll Number" placeholder="101" value={form.roll} onChange={e => setForm({...form, roll: e.target.value})} required />
          <Input label="Full Name" placeholder="Student Name" value={form.name} onChange={e => setForm({...form, name: e.target.value})} required />
          <Input label="Phone" placeholder="Parent/Student Phone" value={form.phone} onChange={e => setForm({...form, phone: e.target.value})} />
          <Select label="Assign Batch" value={form.batchId} onChange={e => setForm({...form, batchId: e.target.value})}>
            <option value="">Select Batch</option>
            {data.batches.map(b => (
              <option key={b.id} value={b.id}>{b.name}</option>
            ))}
          </Select>
          <Input label="Join Date" type="date" value={form.joinDate} onChange={e => setForm({...form, joinDate: e.target.value})} />
          <Button className="w-full">Register Student</Button>
        </form>
      </Card>

      <Card className="lg:col-span-2">
        <h2 className="text-lg font-bold text-slate-800 mb-4">Student Directory</h2>
        <div className="overflow-x-auto">
          <table className="w-full text-sm text-left">
            <thead className="bg-slate-50 text-slate-600 font-semibold border-b border-slate-200">
              <tr>
                <th className="p-3">Roll</th>
                <th className="p-3">Name</th>
                <th className="p-3">Batch</th>
                <th className="p-3">Phone</th>
                <th className="p-3 text-right">Actions</th>
              </tr>
            </thead>
            <tbody className="divide-y divide-slate-100">
              {data.students.map(student => {
                const batch = data.batches.find(b => b.id === student.batchId);
                return (
                  <tr key={student.id} className="hover:bg-slate-50">
                    <td className="p-3 font-mono text-slate-500">{student.roll}</td>
                    <td className="p-3 font-medium text-slate-800">{student.name}</td>
                    <td className="p-3 text-slate-600">
                      <span className={`px-2 py-1 rounded-full text-xs bg-blue-50 text-blue-700`}>
                        {batch?.name || 'Unassigned'}
                      </span>
                    </td>
                    <td className="p-3 text-slate-600">{student.phone}</td>
                    <td className="p-3 text-right flex justify-end gap-2">
                      <button 
                        onClick={() => handleDraftMessage(student)} 
                        className="text-violet-600 hover:text-violet-800 p-1 flex items-center gap-1 bg-violet-50 rounded-md px-2 transition-all hover:scale-105"
                        disabled={draftingId === student.id}
                      >
                        {draftingId === student.id ? <Loader2 size={14} className="animate-spin"/> : <MessageSquare size={14} />}
                        <span className="text-xs font-medium hidden sm:inline">Draft Msg</span>
                      </button>
                      <button onClick={() => onDelete('students', student.id)} className="text-rose-500 hover:text-rose-700 p-1">
                        <Trash2 size={16} />
                      </button>
                    </td>
                  </tr>
                );
              })}
              {data.students.length === 0 && <tr><td colSpan="5" className="p-6 text-center text-slate-400">No students registered</td></tr>}
            </tbody>
          </table>
        </div>
      </Card>
    </div>
  );
};

const Fees = ({ data, onAdd, onDelete }) => {
  const [form, setForm] = useState({ 
    batchId: '', studentId: '', amount: '', 
    month: 'January', date: new Date().toISOString().split('T')[0], mode: 'Cash' 
  });

  const filteredStudents = useMemo(() => {
    return form.batchId ? data.students.filter(s => s.batchId === form.batchId) : [];
  }, [form.batchId, data.students]);

  // Auto-fill amount based on batch
  useEffect(() => {
    if (form.batchId) {
      const batch = data.batches.find(b => b.id === form.batchId);
      if (batch) setForm(prev => ({ ...prev, amount: batch.fee }));
    }
  }, [form.batchId, data.batches]);

  const handleSubmit = (e) => {
    e.preventDefault();
    if (!form.studentId || !form.amount) return;
    
    const student = data.students.find(s => s.id === form.studentId);
    
    onAdd('fees', { 
      ...form, 
      studentName: student?.name || 'Unknown',
      studentRoll: student?.roll || '?',
      createdAt: serverTimestamp() 
    });
    // Reset minimal fields
    setForm(prev => ({ ...prev, studentId: '', amount: '' }));
  };

  return (
    <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
      <Card className="lg:col-span-1 h-fit">
        <h2 className="text-lg font-bold text-slate-800 mb-4 flex items-center gap-2">
          <Wallet size={20} className="text-[#2180a5]" /> Record Payment
        </h2>
        <form onSubmit={handleSubmit}>
          <Select label="Batch" value={form.batchId} onChange={e => setForm({...form, batchId: e.target.value})}>
            <option value="">Select Batch First</option>
            {data.batches.map(b => <option key={b.id} value={b.id}>{b.name}</option>)}
          </Select>

          <Select label="Student" value={form.studentId} onChange={e => setForm({...form, studentId: e.target.value})} disabled={!form.batchId}>
            <option value="">Select Student</option>
            {filteredStudents.map(s => <option key={s.id} value={s.id}>{s.name} ({s.roll})</option>)}
          </Select>

          <div className="grid grid-cols-2 gap-2">
            <Input label="Amount (₹)" type="number" value={form.amount} onChange={e => setForm({...form, amount: e.target.value})} required />
            <Select label="Mode" value={form.mode} onChange={e => setForm({...form, mode: e.target.value})}>
              <option>Cash</option>
              <option>UPI</option>
              <option>Bank</option>
            </Select>
          </div>

          <div className="grid grid-cols-2 gap-2">
            <Select label="Month" value={form.month} onChange={e => setForm({...form, month: e.target.value})}>
              {['January', 'February', 'March', 'April', 'May', 'June', 'July', 'August', 'September', 'October', 'November', 'December'].map(m => <option key={m}>{m}</option>)}
            </Select>
            <Input label="Date" type="date" value={form.date} onChange={e => setForm({...form, date: e.target.value})} />
          </div>

          <Button className="w-full" disabled={!form.studentId}>Confirm Payment</Button>
        </form>
      </Card>

      <Card className="lg:col-span-2">
        <h2 className="text-lg font-bold text-slate-800 mb-4">Fee History</h2>
        <div className="overflow-x-auto">
          <table className="w-full text-sm text-left">
            <thead className="bg-slate-50 text-slate-600 font-semibold border-b border-slate-200">
              <tr>
                <th className="p-3">Date</th>
                <th className="p-3">Student</th>
                <th className="p-3">Month</th>
                <th className="p-3">Mode</th>
                <th className="p-3 text-right">Amount</th>
                <th className="p-3"></th>
              </tr>
            </thead>
            <tbody className="divide-y divide-slate-100">
              {data.fees.sort((a,b) => new Date(b.date) - new Date(a.date)).map(fee => (
                <tr key={fee.id} className="hover:bg-slate-50">
                  <td className="p-3 text-slate-500">{fee.date}</td>
                  <td className="p-3 font-medium text-slate-800">
                    {fee.studentName} <span className="text-slate-400 text-xs">({fee.studentRoll})</span>
                  </td>
                  <td className="p-3 text-slate-600">{fee.month}</td>
                  <td className="p-3">
                    <span className="px-2 py-1 rounded border border-slate-200 text-xs text-slate-500">{fee.mode}</span>
                  </td>
                  <td className="p-3 text-right text-emerald-600 font-bold">+₹{Number(fee.amount).toLocaleString()}</td>
                  <td className="p-3 text-right">
                    <button onClick={() => onDelete('fees', fee.id)} className="text-rose-400 hover:text-rose-600">
                      <Trash2 size={14} />
                    </button>
                  </td>
                </tr>
              ))}
              {data.fees.length === 0 && <tr><td colSpan="6" className="p-6 text-center text-slate-400">No fee records</td></tr>}
            </tbody>
          </table>
        </div>
      </Card>
    </div>
  );
};

const Expenses = ({ data, onAdd, onDelete }) => {
  const [form, setForm] = useState({ 
    description: '', category: 'Utilities', amount: '', 
    date: new Date().toISOString().split('T')[0] 
  });

  const handleSubmit = (e) => {
    e.preventDefault();
    if (!form.description || !form.amount) return;
    onAdd('expenses', { ...form, createdAt: serverTimestamp() });
    setForm({ description: '', category: 'Utilities', amount: '', date: new Date().toISOString().split('T')[0] });
  };

  return (
    <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
      <Card className="lg:col-span-1 h-fit">
        <h2 className="text-lg font-bold text-slate-800 mb-4 flex items-center gap-2">
          <CreditCard size={20} className="text-[#2180a5]" /> Add Expense
        </h2>
        <form onSubmit={handleSubmit}>
          <Select label="Category" value={form.category} onChange={e => setForm({...form, category: e.target.value})}>
            {['Salary', 'Rent', 'Utilities', 'Materials', 'Maintenance', 'Marketing', 'Other'].map(c => <option key={c}>{c}</option>)}
          </Select>
          <Input label="Description" placeholder="e.g. Electricity Bill" value={form.description} onChange={e => setForm({...form, description: e.target.value})} required />
          <Input label="Amount (₹)" type="number" value={form.amount} onChange={e => setForm({...form, amount: e.target.value})} required />
          <Input label="Date" type="date" value={form.date} onChange={e => setForm({...form, date: e.target.value})} />
          <Button variant="danger" className="w-full bg-rose-50 text-rose-600 hover:bg-rose-100 border-rose-200">Record Expense</Button>
        </form>
      </Card>

      <Card className="lg:col-span-2">
        <h2 className="text-lg font-bold text-slate-800 mb-4">Expense Register</h2>
        <div className="overflow-x-auto">
          <table className="w-full text-sm text-left">
            <thead className="bg-slate-50 text-slate-600 font-semibold border-b border-slate-200">
              <tr>
                <th className="p-3">Date</th>
                <th className="p-3">Category</th>
                <th className="p-3">Description</th>
                <th className="p-3 text-right">Amount</th>
                <th className="p-3"></th>
              </tr>
            </thead>
            <tbody className="divide-y divide-slate-100">
              {data.expenses.sort((a,b) => new Date(b.date) - new Date(a.date)).map(exp => (
                <tr key={exp.id} className="hover:bg-slate-50">
                  <td className="p-3 text-slate-500">{exp.date}</td>
                  <td className="p-3">
                    <span className="px-2 py-1 rounded bg-slate-100 text-slate-600 text-xs font-medium">{exp.category}</span>
                  </td>
                  <td className="p-3 text-slate-700">{exp.description}</td>
                  <td className="p-3 text-right text-rose-600 font-bold">-₹{Number(exp.amount).toLocaleString()}</td>
                  <td className="p-3 text-right">
                    <button onClick={() => onDelete('expenses', exp.id)} className="text-rose-400 hover:text-rose-600">
                      <Trash2 size={14} />
                    </button>
                  </td>
                </tr>
              ))}
              {data.expenses.length === 0 && <tr><td colSpan="5" className="p-6 text-center text-slate-400">No expenses recorded</td></tr>}
            </tbody>
          </table>
        </div>
      </Card>
    </div>
  );
};

const Staff = ({ data, onAdd, onDelete }) => {
  const [form, setForm] = useState({ name: '', position: 'Teacher', salary: '', phone: '' });
  const [reviewData, setReviewData] = useState(null);
  const [loadingStaffId, setLoadingStaffId] = useState(null);

  const handleSubmit = (e) => {
    e.preventDefault();
    if (!form.name) return;
    onAdd('staff', { ...form, createdAt: serverTimestamp() });
    setForm({ name: '', position: 'Teacher', salary: '', phone: '' });
  };

  const handleReview = async (member) => {
    setLoadingStaffId(member.id);
    const prompt = `
      Draft a positive and constructive annual performance review for a staff member.
      
      Details:
      - Name: ${member.name}
      - Position: ${member.position}
      
      Structure:
      1. Appreciation for dedication.
      2. Encouragement for future growth.
      Keep it professional and about 100 words.
    `;
    const result = await callGemini(prompt);
    setReviewData({ name: member.name, text: result });
    setLoadingStaffId(null);
  };

  return (
    <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
      <Modal
        isOpen={!!reviewData}
        onClose={() => setReviewData(null)}
        title={`Performance Review: ${reviewData?.name}`}
        icon={ScrollText}
        iconColor="text-orange-500"
      >
        <div className="bg-slate-50 p-4 rounded-lg text-sm text-slate-700 whitespace-pre-wrap leading-relaxed border border-slate-200">
          {reviewData?.text}
        </div>
        <div className="mt-4">
          <Button className="w-full" onClick={() => { navigator.clipboard.writeText(reviewData.text); setReviewData(null); }}>
             <Copy size={16} /> Copy to Clipboard
          </Button>
        </div>
      </Modal>

      <Card className="lg:col-span-1 h-fit">
        <h2 className="text-lg font-bold text-slate-800 mb-4 flex items-center gap-2">
          <Briefcase size={20} className="text-[#2180a5]" /> Add Staff
        </h2>
        <form onSubmit={handleSubmit}>
          <Input label="Name" placeholder="Full Name" value={form.name} onChange={e => setForm({...form, name: e.target.value})} required />
          <Select label="Position" value={form.position} onChange={e => setForm({...form, position: e.target.value})}>
            {['Teacher', 'Assistant', 'Manager', 'Admin', 'Cleaner'].map(p => <option key={p}>{p}</option>)}
          </Select>
          <Input label="Salary" type="number" placeholder="Monthly Salary" value={form.salary} onChange={e => setForm({...form, salary: e.target.value})} />
          <Input label="Phone" placeholder="Contact Number" value={form.phone} onChange={e => setForm({...form, phone: e.target.value})} />
          <Button className="w-full">Add Staff Member</Button>
        </form>
      </Card>

      <Card className="lg:col-span-2">
        <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
          {data.staff.map(member => (
            <div key={member.id} className="p-4 border border-slate-200 rounded-lg flex justify-between items-center hover:shadow-md transition-shadow bg-white">
              <div className="flex items-center gap-3">
                <div className="w-10 h-10 rounded-full bg-slate-100 flex items-center justify-center text-slate-500 font-bold">
                  {member.name.charAt(0)}
                </div>
                <div>
                  <h3 className="font-bold text-slate-800">{member.name}</h3>
                  <p className="text-xs text-slate-500 uppercase tracking-wide">{member.position}</p>
                  <div className="flex gap-3 mt-1 text-xs text-slate-400">
                    <span>{member.phone || 'No Phone'}</span>
                    {member.salary && <span>• ₹{member.salary}/mo</span>}
                  </div>
                </div>
              </div>
              <div className="flex gap-2">
                <button 
                  onClick={() => handleReview(member)}
                  className="text-orange-500 hover:bg-orange-50 p-1.5 rounded-md"
                  title="Draft Review"
                >
                  {loadingStaffId === member.id ? <Loader2 size={16} className="animate-spin"/> : <ScrollText size={16} />}
                </button>
                <button onClick={() => onDelete('staff', member.id)} className="text-slate-300 hover:text-rose-500 p-1.5 rounded-md hover:bg-rose-50">
                  <Trash2 size={16} />
                </button>
              </div>
            </div>
          ))}
          {data.staff.length === 0 && <p className="text-slate-400 col-span-2 text-center py-8">No staff members added</p>}
        </div>
      </Card>
    </div>
  );
};

const Reports = ({ data }) => {
  const [month, setMonth] = useState('January');
  
  const filteredFees = data.fees.filter(f => f.month === month);
  const totalMonthFees = filteredFees.reduce((sum, f) => sum + Number(f.amount), 0);
  
  const filteredExpenses = data.expenses.filter(e => {
    const d = new Date(e.date);
    const mName = d.toLocaleString('default', { month: 'long' });
    return mName === month;
  });
  const totalMonthExpenses = filteredExpenses.reduce((sum, e) => sum + Number(e.amount), 0);

  const downloadReport = () => {
    const headers = ['Date', 'Type', 'Category/Student', 'Amount', 'Description'];
    const csvRows = [headers.join(',')];
    
    // Add Fees
    filteredFees.forEach(f => {
      csvRows.push([f.date, 'Fee', `"${f.studentName}"`, f.amount, `"${f.month} Fee"`].join(','));
    });
    
    // Add Expenses
    filteredExpenses.forEach(e => {
      csvRows.push([e.date, 'Expense', `"${e.category}"`, `-${e.amount}`, `"${e.description}"`].join(','));
    });
    
    const csvString = csvRows.join('\n');
    const blob = new Blob([csvString], { type: 'text/csv' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `Iqra_Report_${month}_${new Date().getFullYear()}.csv`;
    a.click();
    URL.revokeObjectURL(url);
  };

  return (
    <div className="space-y-6">
      <Card>
        <div className="flex flex-col sm:flex-row justify-between items-center mb-6 gap-4">
          <h2 className="text-xl font-bold text-slate-800 flex items-center gap-2">
            <FileText className="text-[#2180a5]" /> Financial Report
          </h2>
          <div className="flex gap-2">
             <select 
                className="px-3 py-2 bg-slate-50 border border-slate-200 rounded-lg text-sm"
                value={month}
                onChange={e => setMonth(e.target.value)}
              >
                {['January', 'February', 'March', 'April', 'May', 'June', 'July', 'August', 'September', 'October', 'November', 'December'].map(m => <option key={m}>{m}</option>)}
              </select>
             <Button variant="secondary" onClick={downloadReport}>
               <Download size={16}/> Download CSV
             </Button>
             <Button variant="outline" onClick={() => window.print()}>Print</Button>
          </div>
        </div>

        <div className="grid grid-cols-1 md:grid-cols-3 gap-6 mb-8">
          <div className="p-4 rounded-xl bg-emerald-50 border border-emerald-100 text-center">
             <p className="text-xs font-bold text-emerald-600 uppercase">Collection ({month})</p>
             <p className="text-2xl font-bold text-emerald-700 mt-1">₹{totalMonthFees.toLocaleString()}</p>
          </div>
          <div className="p-4 rounded-xl bg-rose-50 border border-rose-100 text-center">
             <p className="text-xs font-bold text-rose-600 uppercase">Expenses ({month})</p>
             <p className="text-2xl font-bold text-rose-700 mt-1">₹{totalMonthExpenses.toLocaleString()}</p>
          </div>
          <div className="p-4 rounded-xl bg-slate-50 border border-slate-200 text-center">
             <p className="text-xs font-bold text-slate-600 uppercase">Net Profit</p>
             <p className={`text-2xl font-bold mt-1 ${totalMonthFees - totalMonthExpenses >= 0 ? 'text-[#2180a5]' : 'text-rose-600'}`}>
                ₹{(totalMonthFees - totalMonthExpenses).toLocaleString()}
             </p>
          </div>
        </div>
      </Card>
    </div>
  );
};


// --- Main App Component ---

export default function App() {
  const [user, setUser] = useState(null);
  const [activeTab, setActiveTab] = useState('dashboard');
  const [sidebarOpen, setSidebarOpen] = useState(true);
  
  // Data State
  const [data, setData] = useState({
    batches: [], students: [], fees: [], 
    expenses: [], staff: [], equipment: []
  });

  // Auth & Data Sync
  useEffect(() => {
    const initAuth = async () => {
      if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
        await signInWithCustomToken(auth, __initial_auth_token);
      } else {
        await signInAnonymously(auth);
      }
    };
    initAuth();
    
    return onAuthStateChanged(auth, (u) => setUser(u));
  }, []);

  useEffect(() => {
    if (!user) return;

    // Set up listeners for all collections
    const collections = ['batches', 'students', 'fees', 'expenses', 'staff', 'equipment'];
    const unsubscribes = collections.map(colName => {
      const q = query(
        collection(db, 'artifacts', appId, 'users', user.uid, colName),
        orderBy('createdAt', 'desc')
      ); // Basic query
      
      return onSnapshot(q, (snap) => {
        const items = snap.docs.map(doc => ({ id: doc.id, ...doc.data() }));
        setData(prev => ({ ...prev, [colName]: items }));
      }, (err) => console.error(`Error fetching ${colName}`, err));
    });

    return () => unsubscribes.forEach(unsub => unsub());
  }, [user]);

  // Actions
  const handleAdd = async (colName, payload) => {
    if (!user) return;
    try {
      await addDoc(collection(db, 'artifacts', appId, 'users', user.uid, colName), payload);
    } catch (e) {
      console.error("Add failed", e);
      alert("Failed to add record");
    }
  };

  const handleDelete = async (colName, id) => {
    if (!user) return;
    if (!confirm("Are you sure you want to delete this?")) return;
    try {
      await deleteDoc(doc(db, 'artifacts', appId, 'users', user.uid, colName, id));
    } catch (e) {
      console.error("Delete failed", e);
    }
  };

  // Nav Config
  const navItems = [
    { id: 'dashboard', label: 'Dashboard', icon: LayoutDashboard },
    { id: 'batches', label: 'Batches', icon: BookOpen },
    { id: 'students', label: 'Students', icon: Users },
    { id: 'fees', label: 'Fee Register', icon: Wallet },
    { id: 'expenses', label: 'Expenses', icon: CreditCard },
    { id: 'staff', label: 'Staff', icon: Briefcase },
    { id: 'reports', label: 'Reports', icon: FileText },
  ];

  if (!user) return <div className="h-screen w-full flex items-center justify-center bg-slate-50 text-slate-400">Loading System...</div>;

  return (
    <div className="flex h-screen bg-[#fcfcf9] font-sans text-slate-800 overflow-hidden">
      {/* Sidebar */}
      <aside 
        className={`${sidebarOpen ? 'w-64 translate-x-0' : 'w-0 -translate-x-full'} 
        bg-white border-r border-slate-200 transition-all duration-300 absolute z-20 h-full md:relative md:translate-x-0 md:block
        flex flex-col shadow-xl md:shadow-none`}
      >
        <div className="p-6 border-b border-slate-100 flex justify-between items-center">
          <h1 className="font-bold text-xl text-[#2180a5] flex items-center gap-2">
            <BarChart3 className="fill-[#2180a5]" size={24} /> Iqra Manager
          </h1>
          <button onClick={() => setSidebarOpen(false)} className="md:hidden text-slate-400 hover:text-rose-500"><X size={20}/></button>
        </div>
        
        <nav className="flex-1 p-4 space-y-1 overflow-y-auto">
          {navItems.map(item => (
            <button
              key={item.id}
              onClick={() => { setActiveTab(item.id); setSidebarOpen(false); }}
              className={`w-full flex items-center gap-3 px-4 py-3 rounded-lg text-sm font-medium transition-colors ${
                activeTab === item.id 
                  ? 'bg-[#2180a5] text-white shadow-lg shadow-[#2180a5]/20' 
                  : 'text-slate-600 hover:bg-slate-50 hover:text-slate-900'
              }`}
            >
              <item.icon size={18} />
              {item.label}
            </button>
          ))}
        </nav>

        <div className="p-4 border-t border-slate-100">
           <div className="p-3 bg-slate-50 rounded-lg">
             <p className="text-xs font-semibold text-slate-500 uppercase">System Status</p>
             <div className="flex items-center gap-2 mt-1">
               <div className="w-2 h-2 bg-emerald-500 rounded-full animate-pulse"></div>
               <span className="text-xs text-slate-600 font-medium">Online & Synced</span>
             </div>
           </div>
        </div>
      </aside>

      {/* Main Content */}
      <main className="flex-1 flex flex-col h-full overflow-hidden relative">
        {/* Header */}
        <header className="h-16 bg-white border-b border-slate-200 flex items-center justify-between px-6 shrink-0">
          <div className="flex items-center gap-4">
             {!sidebarOpen && (
               <button onClick={() => setSidebarOpen(true)} className="md:hidden text-slate-500 p-2 hover:bg-slate-50 rounded-lg">
                 <Menu size={20} />
               </button>
             )}
             <h2 className="font-bold text-lg text-slate-700 capitalize">{activeTab.replace('-', ' ')}</h2>
          </div>
          <div className="flex items-center gap-4">
            <div className="text-xs text-right hidden sm:block">
              <p className="font-bold text-slate-700">Admin User</p>
              <p className="text-slate-400">Iqra Coaching Center</p>
            </div>
            <div className="w-9 h-9 bg-[#2180a5] rounded-full flex items-center justify-center text-white font-bold text-sm">
              A
            </div>
          </div>
        </header>

        {/* Scrollable Content Area */}
        <div className="flex-1 overflow-auto p-4 md:p-8 bg-[#fcfcf9]">
          <div className="max-w-6xl mx-auto">
            {activeTab === 'dashboard' && <Dashboard data={data} />}
            {activeTab === 'batches' && <Batches data={data} onAdd={handleAdd} onDelete={handleDelete} />}
            {activeTab === 'students' && <Students data={data} onAdd={handleAdd} onDelete={handleDelete} />}
            {activeTab === 'fees' && <Fees data={data} onAdd={handleAdd} onDelete={handleDelete} />}
            {activeTab === 'expenses' && <Expenses data={data} onAdd={handleAdd} onDelete={handleDelete} />}
            {activeTab === 'staff' && <Staff data={data} onAdd={handleAdd} onDelete={handleDelete} />}
            {activeTab === 'reports' && <Reports data={data} />}
          </div>
        </div>
      </main>
    </div>
  );
}
