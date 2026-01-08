import React, { useState, useEffect } from 'react';
import { 
  Smartphone, Wifi, Wallet, User, History, ShieldCheck, 
  Plus, Bell, Lock, AlertTriangle, Eye, EyeOff, 
  Loader2, Trash2, Share2, Copy, TrendingUp, 
  MessageCircle, CheckCircle, ArrowRight, ExternalLink,
  Info, Megaphone, Edit3, Save, X
} from 'lucide-react';
import { initializeApp } from 'firebase/app';
import { 
  getAuth, signInAnonymously, onAuthStateChanged, signOut, signInWithCustomToken 
} from 'firebase/auth';
import { 
  getFirestore, collection, doc, getDoc, setDoc, onSnapshot, 
  updateDoc, addDoc, deleteDoc, query, orderBy, serverTimestamp, 
  increment, where, getDocs, limit 
} from 'firebase/firestore';

// --- Firebase Initialization ---
const firebaseConfig = JSON.parse(__firebase_config);
const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getFirestore(app);
const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';

const ADMIN_EMAIL = 'yauusman032@gmail.com';
const ADMIN_PHONE = '09072727171';

const NETWORKS = [
  { id: 'mtn', name: 'MTN', color: 'bg-yellow-400' },
  { id: 'glo', name: 'Glo', color: 'bg-green-600' },
  { id: 'airtel', name: 'Airtel', color: 'bg-red-600' },
  { id: '9mobile', name: '9mobile', color: 'bg-green-900' },
];

const INITIAL_PLANS = [
  { network: 'mtn', name: '500MB SME', price: 150, agentPrice: 140, costPrice: 125, validity: '30 Days' },
  { network: 'mtn', name: '1.0GB SME', price: 270, agentPrice: 260, costPrice: 245, validity: '30 Days' },
  { network: 'mtn', name: '2.0GB SME', price: 540, agentPrice: 520, costPrice: 490, validity: '30 Days' },
  { network: 'mtn', name: '5.0GB SME', price: 1350, agentPrice: 1300, costPrice: 1225, validity: '30 Days' },
  { network: 'glo', name: '1.35GB', price: 450, agentPrice: 430, costPrice: 400, validity: '30 Days' },
  { network: 'glo', name: '2.9GB', price: 900, agentPrice: 870, costPrice: 800, validity: '30 Days' },
  { network: 'airtel', name: '1.0GB CG', price: 280, agentPrice: 270, costPrice: 250, validity: '30 Days' },
  { network: 'airtel', name: '2.0GB CG', price: 560, agentPrice: 540, costPrice: 500, validity: '30 Days' },
  { network: '9mobile', name: '1.0GB', price: 180, agentPrice: 170, costPrice: 150, validity: '30 Days' },
];

const formatCurrency = (amount) => new Intl.NumberFormat('en-NG', { style: 'currency', currency: 'NGN' }).format(amount || 0);
const formatDate = (timestamp) => timestamp ? new Date(timestamp.seconds * 1000).toLocaleString() : 'Just now';

const App = () => {
  const [user, setUser] = useState(null);
  const [profile, setProfile] = useState(null);
  const [activeTab, setActiveTab] = useState('home');
  const [loading, setLoading] = useState(true);
  const [systemSettings, setSystemSettings] = useState({ maintenance: false, dataEnabled: true, broadcast: "Welcome to Ussy Data! Fast and Reliable." });

  useEffect(() => {
    const initAuth = async () => {
      if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
        await signInWithCustomToken(auth, __initial_auth_token);
      } else {
        await signInAnonymously(auth);
      }
    };
    initAuth();

    const unsubSettings = onSnapshot(doc(db, 'artifacts', appId, 'public', 'settings'), (docSnap) => {
      if (docSnap.exists()) setSystemSettings(docSnap.data());
    });

    const unsubscribe = onAuthStateChanged(auth, (currentUser) => {
      setUser(currentUser);
      if (currentUser) {
        const profileRef = doc(db, 'artifacts', appId, 'users', currentUser.uid, 'profile', 'main');
        onSnapshot(profileRef, (docSnap) => {
          if (docSnap.exists()) setProfile(docSnap.data());
          setLoading(false);
        });
      } else {
        setProfile(null);
        setLoading(false);
      }
    });
    return () => { unsubscribe(); unsubSettings(); };
  }, []);

  if (loading) return <div className="h-screen flex items-center justify-center bg-slate-50"><Loader2 className="w-8 h-8 text-blue-600 animate-spin" /></div>;
  if (!profile) return <AuthScreen />;
  if (systemSettings.maintenance && profile.role !== 'admin') return <MaintenancePage />;

  const renderContent = () => {
    switch (activeTab) {
      case 'home': return <Dashboard profile={profile} setActiveTab={setActiveTab} systemSettings={systemSettings} />;
      case 'buy': return <BuyData profile={profile} setActiveTab={setActiveTab} systemSettings={systemSettings} />;
      case 'wallet': return <WalletPage profile={profile} />;
      case 'transactions': return <HistoryPage profile={profile} />;
      case 'admin': return <AdminPanel profile={profile} systemSettings={systemSettings} />;
      case 'profile': return <ProfilePage profile={profile} onLogout={() => signOut(auth).then(() => window.location.reload())} />;
      default: return <Dashboard profile={profile} setActiveTab={setActiveTab} systemSettings={systemSettings} />;
    }
  };

  return (
    <div className="bg-slate-200 min-h-screen flex justify-center font-sans">
      <div className="w-full max-w-md bg-white min-h-screen shadow-2xl relative flex flex-col">
        <div className="bg-white px-6 py-4 flex justify-between items-center border-b sticky top-0 z-50">
          <h1 className="font-black text-blue-600 text-lg italic tracking-tighter">USSY DATA</h1>
          <div className="flex gap-4 items-center">
             <div className="flex items-center gap-1">
                <div className={`w-2 h-2 rounded-full ${systemSettings.dataEnabled ? 'bg-green-500' : 'bg-red-500'} animate-pulse`}></div>
                <span className="text-[10px] font-black text-gray-400 uppercase tracking-tighter">Live</span>
             </div>
             <button onClick={() => window.open(`https://wa.me/234${ADMIN_PHONE.slice(1)}`)} className="text-green-500"><MessageCircle size={20} /></button>
          </div>
        </div>

        <div className="flex-1 overflow-y-auto pb-24">
          {systemSettings.broadcast && (
            <div className="bg-blue-600 text-white px-6 py-2 flex items-center gap-3">
              <Megaphone size={14} className="flex-shrink-0" />
              <marquee className="text-[11px] font-bold uppercase tracking-tight">{systemSettings.broadcast}</marquee>
            </div>
          )}
          {renderContent()}
        </div>

        <div className="fixed bottom-0 w-full max-w-md bg-white border-t px-6 py-4 flex justify-between items-center z-40 shadow-lg">
          <NavButton icon={Smartphone} label="Home" isActive={activeTab === 'home'} onClick={() => setActiveTab('home')} />
          <NavButton icon={Wifi} label="Data" isActive={activeTab === 'buy'} onClick={() => setActiveTab('buy')} />
          <NavButton icon={Wallet} label="Wallet" isActive={activeTab === 'wallet'} onClick={() => setActiveTab('wallet')} />
          {profile.role === 'admin' && <NavButton icon={ShieldCheck} label="Admin" isActive={activeTab === 'admin'} onClick={() => setActiveTab('admin')} />}
          <NavButton icon={User} label="Profile" isActive={activeTab === 'profile'} onClick={() => setActiveTab('profile')} />
        </div>
      </div>
    </div>
  );
};

const NavButton = ({ icon: Icon, label, isActive, onClick }) => (
  <button onClick={onClick} className={`flex flex-col items-center gap-1 transition-all ${isActive ? 'text-blue-600 scale-110' : 'text-gray-400'}`}>
    <Icon size={20} strokeWidth={isActive ? 3 : 2} />
    <span className="text-[9px] font-black uppercase tracking-tighter">{label}</span>
  </button>
);

const AuthScreen = () => {
  const [isRegistering, setIsRegistering] = useState(false);
  const [formData, setFormData] = useState({ name: '', email: '', phone: '', pin: '', referralCode: '' });
  const [loading, setLoading] = useState(false);

  const handleSubmit = async (e) => {
    e.preventDefault();
    setLoading(true);
    try {
      if (!auth.currentUser) await signInAnonymously(auth);
      const uid = auth.currentUser.uid;
      const userRef = doc(db, 'artifacts', appId, 'users', uid, 'profile', 'main');

      if (isRegistering) {
        const myCode = formData.phone.slice(-4) + Math.floor(Math.random() * 900 + 100);
        await setDoc(userRef, {
          uid, name: formData.name, email: formData.email.toLowerCase(),
          phone: formData.phone, balance: 0, isAgent: false, pin: formData.pin,
          role: formData.email.toLowerCase() === ADMIN_EMAIL.toLowerCase() ? 'admin' : 'user',
          referralCode: myCode.toUpperCase(), createdAt: serverTimestamp()
        });
        await setDoc(doc(db, 'artifacts', appId, 'public', 'users', uid), { uid, referralCode: myCode.toUpperCase() });
        
        // Push initial plans to public on first admin registration
        if (formData.email.toLowerCase() === ADMIN_EMAIL.toLowerCase()) {
           for (const plan of INITIAL_PLANS) {
               await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'plans'), plan);
           }
           await setDoc(doc(db, 'artifacts', appId, 'public', 'settings'), { maintenance: false, dataEnabled: true, broadcast: "System Live. Welcome Admin!" });
        }
      } else {
        const snap = await getDoc(userRef);
        if (!snap.exists()) { alert("Account not found. Please register."); setIsRegistering(true); }
      }
    } catch (e) { alert("Error: " + e.message); } finally { setLoading(false); }
  };

  return (
    <div className="h-screen bg-blue-700 flex flex-col justify-center p-8">
      <div className="bg-white rounded-[3rem] p-10 shadow-2xl">
        <h1 className="text-3xl font-black text-center text-blue-900 mb-8">USSY DATA</h1>
        <form onSubmit={handleSubmit} className="space-y-4">
          {isRegistering && <input placeholder="Full Name" required className="w-full p-4 bg-gray-50 rounded-2xl border-none text-sm font-bold" onChange={e => setFormData({...formData, name: e.target.value})} />}
          <input placeholder="Email" type="email" required className="w-full p-4 bg-gray-50 rounded-2xl border-none text-sm font-bold" onChange={e => setFormData({...formData, email: e.target.value})} />
          <input placeholder="Phone" type="tel" required className="w-full p-4 bg-gray-50 rounded-2xl border-none text-sm font-bold" onChange={e => setFormData({...formData, phone: e.target.value})} />
          <input placeholder="4-Digit PIN" type="password" maxLength={4} required className="w-full p-4 bg-gray-50 rounded-2xl border-none text-sm font-bold" onChange={e => setFormData({...formData, pin: e.target.value})} />
          <button className="w-full bg-blue-600 text-white font-black py-4 rounded-2xl shadow-xl shadow-blue-200 mt-4">
            {loading ? <Loader2 className="animate-spin mx-auto" /> : (isRegistering ? 'REGISTER' : 'LOGIN')}
          </button>
        </form>
        <button className="w-full mt-8 text-[10px] text-gray-400 font-black uppercase tracking-widest" onClick={() => setIsRegistering(!isRegistering)}>
            {isRegistering ? 'Already have an account? Login' : 'No account? Register'}
        </button>
      </div>
    </div>
  );
};

const MaintenancePage = () => (
    <div className="h-screen flex flex-col items-center justify-center p-12 text-center bg-white">
        <AlertTriangle size={64} className="text-orange-500 mb-6" />
        <h1 className="text-2xl font-black mb-2">Service Maintenance</h1>
        <p className="text-gray-500 text-sm">We are upgrading our systems. Please check back later.</p>
    </div>
);

const Dashboard = ({ profile, setActiveTab }) => (
  <div className="p-6 space-y-6">
    <div className="flex justify-between items-end">
      <div>
         <p className="text-[10px] font-black text-gray-400 uppercase tracking-widest mb-1">Available Funds</p>
         <h1 className="text-4xl font-black text-gray-900 tracking-tighter">{formatCurrency(profile.balance)}</h1>
      </div>
      <button onClick={() => setActiveTab('wallet')} className="bg-blue-600 text-white p-3 rounded-2xl shadow-lg shadow-blue-100"><Plus size={24} /></button>
    </div>

    <div className="grid grid-cols-2 gap-4">
      <ActionCard icon={Wifi} label="Data Bundle" color="text-indigo-600" bg="bg-indigo-50" onClick={() => setActiveTab('buy')} />
      <ActionCard icon={Smartphone} label="Airtime VTU" color="text-emerald-600" bg="bg-emerald-50" onClick={() => alert("Airtime coming soon!")} />
    </div>

    <div className="bg-slate-900 rounded-[2.5rem] p-6 text-white flex justify-between items-center">
        <div>
            <p className="text-blue-400 text-[10px] font-black uppercase mb-1">Referral Code</p>
            <h3 className="text-lg font-black">{profile.referralCode}</h3>
        </div>
        <button onClick={() => { document.execCommand('copy'); alert("Copied!"); }} className="bg-white/10 p-4 rounded-3xl"><Copy size={20} /></button>
    </div>

    <div className="bg-blue-50 border border-blue-100 rounded-3xl p-5 flex items-center justify-between">
        <div className="flex gap-4 items-center">
            <div className="bg-blue-600 p-3 rounded-2xl text-white"><MessageCircle size={20} /></div>
            <div>
                <p className="font-black text-xs text-blue-900">Support Center</p>
                <p className="text-[10px] font-bold text-blue-700 uppercase">Chat on WhatsApp</p>
            </div>
        </div>
        <button onClick={() => window.open(`https://wa.me/234${ADMIN_PHONE.slice(1)}`)} className="text-blue-600"><ArrowRight size={20} /></button>
    </div>
  </div>
);

const ActionCard = ({ icon: Icon, label, color, bg, onClick }) => (
  <button onClick={onClick} className="bg-white p-6 border border-gray-100 rounded-[2.5rem] flex flex-col items-center gap-3 transition-all active:scale-95 shadow-sm">
    <div className={`${bg} ${color} p-4 rounded-2xl`}><Icon size={24} strokeWidth={2.5} /></div>
    <p className="font-black text-[11px] text-gray-700 uppercase tracking-tighter">{label}</p>
  </button>
);

const BuyData = ({ profile, setActiveTab, systemSettings }) => {
  const [network, setNetwork] = useState(null);
  const [plan, setPlan] = useState(null);
  const [phone, setPhone] = useState('');
  const [pin, setPin] = useState('');
  const [loading, setLoading] = useState(false);
  const [plans, setPlans] = useState([]);
  const [showPin, setShowPin] = useState(false);

  useEffect(() => {
    return onSnapshot(query(collection(db, 'artifacts', appId, 'public', 'data', 'plans'), orderBy('price')), snap => {
      setPlans(snap.docs.map(d => ({id: d.id, ...d.data()})));
    });
  }, []);

  const handlePurchase = async () => {
    if (pin !== profile.pin) return alert("Incorrect PIN");
    if (profile.balance < (profile.isAgent ? plan.agentPrice : plan.price)) return alert("Insufficient Balance");
    
    setLoading(true);
    const cost = profile.isAgent ? plan.agentPrice : plan.price;
    const profit = cost - plan.costPrice;

    try {
      await new Promise(r => setTimeout(r, 2000)); // Simulate API
      await updateDoc(doc(db, 'artifacts', appId, 'users', profile.uid, 'profile', 'main'), { balance: increment(-cost) });
      const tx = { type: 'data_purchase', plan: plan.name, network: network.name, phone, amount: cost, profit, status: 'successful', date: serverTimestamp() };
      await addDoc(collection(db, 'artifacts', appId, 'users', profile.uid, 'transactions'), tx);
      await addDoc(collection(db, 'artifacts', appId, 'public', 'sales'), { ...tx, buyer: profile.name });
      alert("Purchase Successful!");
      setActiveTab('home');
    } catch (e) { alert("Error"); } finally { setLoading(false); }
  };

  return (
    <div className="p-6 pb-32">
      <h2 className="text-2xl font-black mb-6 tracking-tight">Purchase Data</h2>
      <div className="grid grid-cols-4 gap-3 mb-8">
        {NETWORKS.map(n => (
          <button key={n.id} onClick={() => { setNetwork(n); setPlan(null); }} className={`aspect-square rounded-2xl border-4 flex flex-col items-center justify-center gap-1 ${network?.id === n.id ? 'border-blue-600 bg-blue-50' : 'border-transparent bg-gray-100'}`}>
            <div className={`w-8 h-8 rounded-full ${n.color}`} />
            <span className="text-[10px] font-black uppercase tracking-tighter">{n.name}</span>
          </button>
        ))}
      </div>

      {network && (
        <div className="space-y-4">
            <h2 className="text-xl font-black tracking-tight">Select Package</h2>
            <div className="grid grid-cols-1 gap-3">
            {plans.filter(p => p.network === network.id).map(p => (
                <button key={p.id} onClick={() => setPlan(p)} className={`p-5 border-2 rounded-3xl flex justify-between items-center ${plan?.id === p.id ? 'border-blue-600 bg-blue-50 shadow-lg' : 'border-gray-50 bg-gray-50'}`}>
                    <div><p className="font-black text-xs uppercase text-gray-800">{p.name}</p><p className="text-[10px] text-gray-400 font-bold">{p.validity}</p></div>
                    <p className="text-blue-600 font-black">{formatCurrency(profile.isAgent ? p.agentPrice : p.price)}</p>
                </button>
            ))}
            </div>
        </div>
      )}

      {plan && (
        <div className="mt-8 space-y-4">
          <input type="tel" placeholder="Phone Number" value={phone} onChange={e => setPhone(e.target.value)} className="w-full p-5 bg-gray-100 border-none rounded-[2rem] font-black outline-none text-lg" />
          <button onClick={() => setShowPin(true)} className="w-full bg-blue-600 text-white font-black py-5 rounded-[2rem] shadow-xl uppercase tracking-widest text-xs">Continue</button>
        </div>
      )}

      {showPin && (
          <div className="fixed inset-0 bg-black/60 backdrop-blur-sm z-[100] flex items-center justify-center p-6">
              <div className="bg-white w-full max-w-sm rounded-[3rem] p-10 space-y-6 shadow-2xl">
                  <h3 className="font-black text-xl text-center">Confirm PIN</h3>
                  <input type="password" maxLength={4} autoFocus className="w-full p-5 bg-gray-50 rounded-2xl border-none text-center font-black text-4xl tracking-[0.5em] outline-none" onChange={e => setPin(e.target.value)} />
                  <div className="flex gap-4">
                    <button onClick={() => setShowPin(false)} className="flex-1 py-4 font-black text-xs text-gray-400 uppercase tracking-widest">Back</button>
                    <button onClick={handlePurchase} disabled={loading || pin.length < 4} className="flex-[2] bg-blue-600 text-white py-4 rounded-2xl font-black text-xs uppercase tracking-widest">
                        {loading ? <Loader2 className="animate-spin mx-auto" /> : 'Pay Now'}
                    </button>
                  </div>
              </div>
          </div>
      )}
    </div>
  );
};

const WalletPage = ({ profile }) => {
  const [amount, setAmount] = useState('');
  const [loading, setLoading] = useState(false);

  const handleFund = async () => {
    setLoading(true);
    setTimeout(async () => {
      await updateDoc(doc(db, 'artifacts', appId, 'users', profile.uid, 'profile', 'main'), { balance: increment(parseFloat(amount)) });
      await addDoc(collection(db, 'artifacts', appId, 'users', profile.uid, 'transactions'), { type: 'wallet_fund', amount: parseFloat(amount), status: 'successful', date: serverTimestamp() });
      setLoading(false); setAmount(''); alert("Funded!");
    }, 1500);
  };

  return (
    <div className="p-6 space-y-8">
      <h2 className="text-2xl font-black tracking-tight text-gray-900">Fund Wallet</h2>
      <div className="bg-blue-600 rounded-[3rem] p-10 text-white shadow-xl shadow-blue-100">
        <p className="text-blue-100 text-[10px] font-black uppercase mb-1 tracking-widest">Balance</p>
        <h1 className="text-4xl font-black tracking-tighter">{formatCurrency(profile.balance)}</h1>
      </div>
      <input type="number" placeholder="₦0.00" value={amount} onChange={e => setAmount(e.target.value)} className="w-full p-8 border-no
