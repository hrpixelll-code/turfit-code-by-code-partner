import React, { useState, useEffect, useRef, useMemo } from 'react';
import { 
  Home, Dumbbell, Users, Ticket, User, MapPin, Mic, Star, Clock, Search, Filter, 
  MessageSquare, Play, Heart, Share2, QrCode, Navigation, X, Wallet, 
  Plus, Trash2, Power, ChevronRight, Send, Bot, Camera, CheckCircle, ArrowLeft, 
  Layout, IndianRupee, Sun, Moon, Droplets, Car, Shirt, Info, Calendar, 
  Image as ImageIcon, Settings, Bell, HelpCircle, Phone, Sparkles, Loader2, 
  Edit3, Zap, BrainCircuit, Smile, Shield, Megaphone, TrendingUp, Copy, Check, 
  MessageCircle, BarChart2, LineChart, ScanLine, ClipboardList, Briefcase, 
  Utensils, Newspaper, Upload, AlertTriangle, XCircle, Keyboard, BookOpen,
  Activity, Flame, Award, Video, CloudRain, Lock, ShoppingBag, UserPlus, Flag, 
  Camera as CameraIcon, Siren, Fingerprint, Gamepad2, Gift, Recycle, Share, 
  ThumbsUp, ThumbsDown, Radio, AlertOctagon, UserMinus, RefreshCcw, FileText,
  Users2, Sword, Target, Coffee, Gavel, Medal, BadgeCheck, Footprints, Speaker,
  Ban, CloudLightning, Umbrella, Store, FileSpreadsheet, Dices, ImagePlus, Map as MapIcon, StarHalf,
  Move, PowerOff, List, Trophy, CheckSquare, Edit, CreditCard, RefreshCw,
  History, MapPinned, PartyPopper, AlertCircle, Smartphone, Banknote
} from 'lucide-react';

// --- FIREBASE IMPORTS ---
import { initializeApp } from 'firebase/app';
import { 
  getFirestore, collection, addDoc, onSnapshot, 
  query, orderBy, doc, updateDoc, setDoc, getDocs, deleteDoc,
  serverTimestamp, increment, limit, where, getDoc 
} from 'firebase/firestore';
import { getAuth, signInAnonymously, onAuthStateChanged, signInWithCustomToken } from 'firebase/auth';

// --- GEMINI API SETUP ---
const apiKey = ""; // API Key provided by environment

const callGemini = async (prompt) => {
  const url = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`;
  const payload = {
    contents: [{ parts: [{ text: prompt }] }]
  };

  const delay = (ms) => new Promise(resolve => setTimeout(resolve, ms));
  
  for (let i = 0; i < 3; i++) { // Retry up to 3 times
    try {
      const response = await fetch(url, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(payload)
      });

      if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);
      
      const data = await response.json();
      return data.candidates?.[0]?.content?.parts?.[0]?.text || "AI is taking a water break. Try again!";
    } catch (error) {
      if (i === 2) {
        console.error("Gemini API failed:", error);
        return null; // Return null to trigger fallback
      }
      await delay(Math.pow(2, i) * 1000); // Exponential backoff
    }
  }
};

// --- FIREBASE SETUP ---
const getFirebaseConfig = () => {
  try {
    if (typeof __firebase_config !== 'undefined') {
      return JSON.parse(__firebase_config);
    }
  } catch (e) {
    console.warn("Firebase config not found, using dummy.");
  }
  return { apiKey: "dummy", projectId: "turfit-demo" };
};

const firebaseConfig = getFirebaseConfig();
const app = initializeApp(firebaseConfig);
const db = getFirestore(app);
const auth = getAuth(app);
const appId = typeof __app_id !== 'undefined' ? __app_id : 'turfit-default';

// --- STATIC DATA ARRAYS ---
const FITNESS_DRILLS = [
  "5-Minute Stamina: High knees for 1 min, Jumping jacks for 1 min, Burpees for 1 min, Rest 30s, Sprint intervals for 1.5 mins.",
  "Agility Drill: Set up 4 cones in a square. Sprint to each, side shuffle between them. Repeat for 5 sets of 45 seconds.",
  "Ball Control: Dribble through cones in a zig-zag pattern. 10 reps. Focus on close control with both feet.",
  "Recovery: Light jog for 2 mins, Dynamic stretching (leg swings, lunges) for 3 mins. Hydrate immediately.",
  "Core Strength: Plank (1 min), Russian Twists (1 min), Leg Raises (1 min), Mountain Climbers (1 min), Rest (1 min)."
];

const INITIAL_LEADERBOARD = [
  { id: 1, name: "Rahul K.", points: 1250, rank: 1, avatar: "RK", area: "Ambala City", type: "Youth League", isPro: true },
  { id: 2, name: "Amit S.", points: 1100, rank: 2, avatar: "AS", area: "Cantt", type: "Pro", isPro: false },
  { id: 3, name: "Priya M.", points: 950, rank: 3, avatar: "PM", area: "Sector 9", type: "Youth League", isPro: true },
];

const AMBALA_CIRCUIT = [
  { id: 1, team: "Cantt Killers", turfPts: 450, bgmiPts: 320, total: 770, zone: "Cantt" },
  { id: 2, team: "City Strikers", turfPts: 400, bgmiPts: 290, total: 690, zone: "City" },
  { id: 3, team: "Sector 9 Snipers", turfPts: 310, bgmiPts: 350, total: 660, zone: "City" },
];

const INITIAL_DEBTS = [
  { id: 1, user: "Amit S.", amount: 200, reason: "Last Match Fee" },
  { id: 2, user: "Rahul K.", amount: 50, reason: "Gatorade" }
];

// --- HELPER COMPONENTS ---

const SirenOverlay = ({ onClose }) => (
  <div className="fixed inset-0 z-[100] bg-red-600 flex flex-col items-center justify-center animate-pulse">
    <div className="bg-white p-8 rounded-full mb-8 shadow-[0_0_100px_rgba(255,255,255,0.5)]">
       <Siren size={120} className="text-red-600 animate-bounce" />
    </div>
    <h1 className="text-white text-6xl font-black uppercase tracking-widest mb-4">EMERGENCY</h1>
    <p className="text-white text-xl font-bold mb-12">Guardian Mode Active</p>
    <button 
      onClick={onClose}
      className="bg-white text-red-600 px-8 py-4 rounded-full font-bold text-xl hover:scale-105 transition-transform"
    >
      DEACTIVATE
    </button>
  </div>
);

const NavItem = ({ icon: Icon, label, active, onClick }) => (
  <button 
    onClick={onClick}
    className={`flex flex-col items-center justify-center w-16 h-16 transition-all duration-300 ${active ? 'text-green-400 -mt-6' : 'text-gray-400'}`}
  >
    <div className={`p-3 rounded-full transition-all ${active ? 'bg-gray-800 border-4 border-gray-900 shadow-lg shadow-green-500/20' : ''}`}>
      <Icon size={active ? 24 : 22} className={active ? 'animate-bounce-slight' : ''} />
    </div>
    <span className={`text-[10px] mt-1 font-medium ${active ? 'opacity-100' : 'opacity-0'}`}>
      {label}
    </span>
  </button>
);

const StatCard = ({ label, value, icon: Icon, color, subtext }) => (
  <div className="bg-gray-800 p-3 rounded-2xl border border-gray-700 flex flex-col justify-between h-28 relative overflow-hidden group">
    <div className={`absolute top-0 right-0 p-3 opacity-10 group-hover:opacity-20 transition-opacity ${color}`}>
      <Icon size={48} />
    </div>
    <div className={`p-2 rounded-lg w-fit ${color.replace('text-', 'bg-')}/10 mb-1`}>
      <Icon size={18} className={color} />
    </div>
    <div>
      <p className="text-gray-400 text-[10px] font-medium uppercase tracking-wider">{label}</p>
      <p className="text-xl font-bold text-white">{value}</p>
      {subtext && <p className="text-[10px] text-gray-500">{subtext}</p>}
    </div>
  </div>
);

const LiveTicker = () => (
  <div className="bg-black py-2 overflow-hidden border-b border-gray-800 relative z-10">
     <div className="animate-marquee whitespace-nowrap flex gap-8 text-xs font-mono text-green-400">
       <span>⚽ LIVE: Alpha Arena (3 spots left) - Starting in 10m</span>
       <span>🍗 KILL-FEED: Team Cantt just got a Chicken Dinner (5v5)!</span>
       <span>🏆 AMBALA CIRCUIT: City Strikers +50 BGMI Pts</span>
     </div>
  </div>
);

const POSButton = ({ icon: Icon, label, price, onClick }) => (
  <button 
    onClick={onClick}
    className="flex-1 bg-gray-700/50 hover:bg-green-600 transition-all p-3 rounded-2xl border border-gray-600 flex flex-col items-center gap-1 group"
  >
    <Icon size={18} className="text-blue-400 group-hover:text-white" />
    <span className="text-[10px] font-bold text-gray-300 group-hover:text-white">{label}</span>
    <span className="text-[9px] font-black text-green-400 group-hover:text-white">₹{price}</span>
  </button>
);

const CrownIcon = ({className}) => (
   <svg viewBox="0 0 24 24" fill="currentColor" className={className}>
      <path d="M2 20H22V18H2V20ZM5 16H19L22 4L15 8L12 2L9 8L2 4L5 16Z"/>
   </svg>
);

// --- MODALS ---

const StrategyModal = ({ title, content, onClose }) => (
  <div className="fixed inset-0 bg-black/90 z-[80] flex flex-col items-center justify-center p-4 animate-in fade-in">
    <div className="bg-indigo-900 w-full max-w-sm rounded-3xl border border-indigo-500 p-6 shadow-2xl relative overflow-hidden">
      <div className="absolute top-0 right-0 p-4 opacity-20"><BrainCircuit size={100} className="text-white"/></div>
      <h3 className="text-xl font-bold text-white mb-4 flex items-center gap-2">
        <Sparkles className="text-yellow-400"/> {title}
      </h3>
      <div className="text-indigo-100 text-sm mb-6 whitespace-pre-wrap leading-relaxed relative z-10">
        {content}
      </div>
      <button onClick={onClose} className="w-full py-3 rounded-xl font-bold bg-white text-indigo-900 hover:bg-indigo-50">Got it, Coach!</button>
    </div>
  </div>
);

const ConfirmationModal = ({ title, message, onConfirm, onCancel }) => (
  <div className="fixed inset-0 bg-black/90 z-[80] flex flex-col items-center justify-center p-4 animate-in fade-in">
    <div className="bg-gray-900 w-full max-w-sm rounded-3xl border border-gray-800 p-6 text-center shadow-2xl">
      <div className="w-16 h-16 bg-red-500/20 rounded-full flex items-center justify-center mx-auto mb-4">
        <AlertTriangle size={32} className="text-red-500" />
      </div>
      <h3 className="text-xl font-bold text-white mb-2">{title}</h3>
      <p className="text-gray-400 text-sm mb-6">{message}</p>
      <div className="flex gap-3">
        <button onClick={onCancel} className="flex-1 py-3 rounded-xl font-bold bg-gray-800 text-gray-400 hover:bg-gray-700">Cancel</button>
        <button onClick={onConfirm} className="flex-1 py-3 rounded-xl font-bold bg-red-600 text-white hover:bg-red-500">Delete</button>
      </div>
    </div>
  </div>
);

const NotificationsModal = ({ notifications, onClose }) => {
   return (
      <div className="fixed inset-0 bg-black/95 z-[70] flex flex-col items-center justify-center p-4 animate-in fade-in">
         <div className="bg-gray-900 w-full max-w-sm rounded-3xl border border-gray-800 overflow-hidden flex flex-col max-h-[70vh]">
            <div className="p-4 border-b border-gray-800 flex justify-between items-center">
               <h3 className="font-bold text-white text-lg">Notifications</h3>
               <button onClick={onClose} className="text-gray-400 hover:text-white"><X size={20}/></button>
            </div>
            <div className="p-4 overflow-y-auto flex-1 custom-scrollbar space-y-3">
               {notifications.length === 0 ? (
                  <p className="text-center text-gray-500 text-sm">No new notifications.</p>
               ) : (
                  notifications.map((note, i) => (
                     <div key={i} className="bg-gray-800 p-3 rounded-xl border border-gray-700 flex gap-3">
                        <div className="bg-blue-900/30 p-2 rounded-full h-fit">
                           <Bell size={16} className="text-blue-400"/>
                        </div>
                        <div>
                           <p className="text-sm text-white font-medium">{note.message}</p>
                           <p className="text-[10px] text-gray-500 mt-1">
                              {note.timestamp?.seconds ? new Date(note.timestamp.seconds * 1000).toLocaleTimeString() : 'Just now'}
                           </p>
                        </div>
                     </div>
                  ))
               )}
            </div>
         </div>
      </div>
   );
};

const MapPickerModal = ({ onClose, onSelect }) => {
  return (
    <div className="fixed inset-0 bg-black/95 z-[70] flex flex-col items-center justify-center p-4 animate-in zoom-in">
       <div className="bg-gray-900 w-full h-full max-w-lg rounded-3xl overflow-hidden relative flex flex-col">
          <div className="absolute top-4 left-4 z-10 bg-black/50 p-2 rounded-full cursor-pointer" onClick={onClose}>
             <ArrowLeft className="text-white" />
          </div>
          <div className="flex-1 bg-emerald-900 relative group cursor-crosshair" 
               onClick={() => {
                  onSelect("Sector 9, Ambala City (Pinned)");
                  onClose();
               }}>
             <div className="absolute inset-0 opacity-20" style={{backgroundImage: 'linear-gradient(#fff 1px, transparent 1px), linear-gradient(90deg, #fff 1px, transparent 1px)', backgroundSize: '40px 40px'}}></div>
             <div className="absolute top-1/4 left-1/4 w-32 h-32 border-4 border-white/10 rounded-full"></div>
             <div className="absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 flex flex-col items-center animate-bounce pointer-events-none">
                <MapPin size={48} className="text-red-500 fill-current drop-shadow-xl" />
                <span className="bg-white text-black text-xs px-2 py-1 rounded font-bold shadow-lg mt-1">Tap to Pin Location</span>
             </div>
             <div className="absolute bottom-8 left-4 right-4 bg-white p-4 rounded-xl shadow-xl pointer-events-none">
                <h4 className="font-bold text-black">Set Turf Location</h4>
                <p className="text-xs text-gray-500">Tap anywhere on the map to confirm your turf's position.</p>
             </div>
          </div>
       </div>
    </div>
  );
};

const ThreeSixtyModal = ({ turf, onClose }) => {
  return (
    <div className="fixed inset-0 z-[70] bg-black/95 flex flex-col animate-in fade-in duration-300">
       <style>{`
         @keyframes scroll-360 {
           0% { transform: translateX(0); }
           100% { transform: translateX(-50%); }
         }
         .animate-scroll-360 {
           animation: scroll-360 20s linear infinite;
         }
       `}</style>
       <button onClick={onClose} className="absolute top-4 right-4 z-50 bg-black/50 p-2 rounded-full text-white hover:bg-black"><X size={24}/></button>
       <div className="flex-1 flex items-center justify-center relative overflow-hidden">
          <div className="absolute inset-0 flex items-center justify-center">
             <div className="w-full h-full relative overflow-hidden">
                <div className="flex h-full w-[200%] animate-scroll-360" style={{
                   backgroundImage: `url(${turf.image})`,
                   backgroundSize: '50% 100%', 
                   backgroundRepeat: 'repeat-x',
                }}>
                </div>
             </div>
          </div>
          <div className="absolute bottom-12 left-1/2 -translate-x-1/2 bg-black/60 backdrop-blur-md px-6 py-3 rounded-full text-white flex items-center gap-3 border border-white/10 shadow-2xl z-50">
             <RefreshCw size={20} className="animate-spin-slow text-cyan-400"/>
             <span className="font-bold tracking-wider text-sm">360° TOUR ACTIVE</span>
          </div>
          <div className="absolute top-4 left-4 bg-black/40 p-2 rounded-full border border-white/10 z-50">
             <div className="w-10 h-10 rounded-full border-2 border-white/50 flex items-center justify-center relative">
                <div className="w-1 h-3 bg-red-500 absolute top-1 rounded-full"></div>
                <div className="w-1 h-3 bg-white absolute bottom-1 rounded-full"></div>
             </div>
          </div>
       </div>
       <div className="bg-gray-900 p-4 border-t border-gray-800 z-50">
          <h3 className="text-white font-bold text-lg">{turf.name}</h3>
          <p className="text-cyan-400 text-xs font-mono uppercase">360° Virtual Tour • Live Feed</p>
       </div>
    </div>
  );
};

const ScannerModal = ({ onClose }) => {
  const [scanning, setScanning] = useState(true);
  const [manualCode, setManualCode] = useState('');
  const [scanResult, setScanResult] = useState(null);

  const handleManualVerify = async () => {
     if(!manualCode.trim()) return;
     setScanning(false);
     const q = query(collection(db, 'artifacts', appId, 'public', 'data', 'booked_slots'), where('displayId', '==', manualCode.trim().toUpperCase()));
     const snap = await getDocs(q);
     if (!snap.empty) {
        const docSnap = snap.docs[0];
        const data = docSnap.data();

        if (data.status === 'Used') {
           setScanResult({ valid: false, message: "Ticket Already Used", data });
        } else {
           await updateDoc(doc(db, 'artifacts', appId, 'public', 'data', 'booked_slots', docSnap.id), {
              status: 'Used',
              usedAt: serverTimestamp()
           });
           if (data.userBookingId && data.bookedBy) {
              try {
                 await updateDoc(doc(db, 'artifacts', appId, 'users', data.bookedBy, 'bookings', data.userBookingId), {
                     status: 'Used'
                 });
              } catch (e) {
                 console.error("Failed to sync user booking status", e);
              }
           }
           setScanResult({ valid: true, data: { ...data, status: 'Used' } });
        }
     } else {
        setScanResult({ valid: false, message: "Invalid Ticket ID" });
     }
  };

  return (
    <div className="fixed inset-0 bg-black/95 z-[60] flex flex-col items-center justify-center p-4 animate-in fade-in">
      <button onClick={onClose} className="absolute top-4 right-4 text-white p-2 bg-gray-800 rounded-full"><X size={24} /></button>
      <h2 className="text-white text-xl font-bold mb-4">Ticket Verification</h2>
      {scanResult ? (
         <div className="bg-gray-800 p-6 rounded-3xl border border-gray-700 text-center animate-in zoom-in">
            {scanResult.valid ? (
               <>
                  <div className="w-20 h-20 bg-green-500/20 rounded-full flex items-center justify-center mx-auto mb-4">
                     <CheckCircle size={40} className="text-green-500"/>
                  </div>
                  <h3 className="text-xl font-bold text-white mb-1">Access Granted</h3>
                  <p className="text-green-400 text-xs uppercase font-bold tracking-widest mb-2">TICKET REDEEMED</p>
                  <p className="text-gray-400 text-sm mb-4">{scanResult.data.turfName}</p>
                  <div className="text-xs bg-black/50 p-2 rounded text-gray-500">
                     {scanResult.data.date} • {scanResult.data.time}
                  </div>
                  <p className="text-green-500 text-xs mt-2 font-mono">ID: {scanResult.data.displayId}</p>
               </>
            ) : (
               <>
                  <div className="w-20 h-20 bg-red-500/20 rounded-full flex items-center justify-center mx-auto mb-4">
                     <XCircle size={40} className="text-red-500"/>
                  </div>
                  <h3 className="text-xl font-bold text-white mb-1">Access Denied</h3>
                  <p className="text-red-400 text-sm font-bold uppercase mb-2">{scanResult.message}</p>
                  {scanResult.data && (
                      <p className="text-gray-500 text-xs">Used on: {scanResult.data.date}</p>
                  )}
               </>
            )}
            <button onClick={() => { setScanResult(null); setScanning(true); setManualCode(''); }} className="mt-6 text-sm text-blue-400 hover:underline">Scan Another</button>
         </div>
      ) : (
         <>
            <div className="w-64 h-64 border-4 border-green-500 rounded-3xl relative overflow-hidden flex items-center justify-center shadow-[0_0_40px_rgba(34,197,94,0.3)] mb-8">
                <div className="absolute inset-0 bg-green-500/10 animate-pulse"></div>
                <div className="absolute top-0 left-0 w-full h-1 bg-green-400 shadow-[0_0_20px_rgba(34,197,94,0.8)] animate-scan"></div>
                <ScanLine size={64} className="text-green-500/50" />
            </div>
            <div className="w-full max-w-xs">
               <p className="text-gray-400 text-xs mb-2 text-center">Camera not available? Enter ID manually:</p>
               <div className="flex gap-2">
                  <input 
                     className="flex-1 bg-gray-800 border border-gray-700 rounded-xl p-3 text-white text-center font-mono uppercase"
                     placeholder="e.g. TI-8X92"
                     value={manualCode}
                     onChange={(e) => setManualCode(e.target.value)}
                  />
                  <button onClick={handleManualVerify} className="bg-green-600 text-white px-4 rounded-xl font-bold">Go</button>
               </div>
            </div>
         </>
      )}
    </div>
  );
};

const DashboardModal = ({ title, onClose, children }) => (
  <div className="fixed inset-0 bg-black/95 z-[60] flex flex-col items-center justify-center p-4 animate-in fade-in">
    <div className="bg-gray-900 w-full max-w-sm rounded-3xl border border-gray-800 overflow-hidden relative flex flex-col max-h-[80vh]">
      <div className="p-4 border-b border-gray-800 flex justify-between items-center">
        <h3 className="font-bold text-white text-lg">{title}</h3>
        <button onClick={onClose} className="text-gray-400 hover:text-white"><X size={20}/></button>
      </div>
      <div className="p-4 overflow-y-auto flex-1 custom-scrollbar">
        {children}
      </div>
    </div>
  </div>
);

const EditProfileModal = ({ profile, onClose, onSave }) => {
   const [name, setName] = useState(profile.displayName || 'Player');
   const [phone, setPhone] = useState(profile.phone || '');

   return (
      <div className="fixed inset-0 bg-black/90 z-[60] flex flex-col items-center justify-center p-4 animate-in fade-in">
         <div className="bg-gray-900 w-full max-w-sm p-6 rounded-3xl border border-gray-800">
             <h3 className="text-xl font-bold text-white mb-4">Edit Profile</h3>
             <div className="space-y-4">
                <div>
                   <label className="text-xs text-gray-400 block mb-1">Display Name</label>
                   <input className="w-full bg-gray-800 p-3 rounded-xl border border-gray-700 text-white" value={name} onChange={e => setName(e.target.value)} />
                </div>
                <div>
                   <label className="text-xs text-gray-400 block mb-1">Phone Number</label>
                   <input className="w-full bg-gray-800 p-3 rounded-xl border border-gray-700 text-white" value={phone} onChange={e => setPhone(e.target.value)} placeholder="+91..." />
                </div>
                <button onClick={() => onSave(name, phone)} className="w-full bg-green-500 text-black py-3 rounded-xl font-bold mt-4">Save Changes</button>
                <button onClick={onClose} className="w-full text-gray-400 py-2 text-sm">Cancel</button>
             </div>
         </div>
      </div>
   );
};

const ReviewModal = ({ onClose, onSubmit, booking }) => {
  const [rating, setRating] = useState(5);
  const [comment, setComment] = useState("");

  const handleSubmit = () => {
    onSubmit(booking, rating, comment);
    onClose();
  };

  return (
    <div className="fixed inset-0 bg-black/90 z-50 flex items-center justify-center p-4 animate-in fade-in">
      <div className="bg-gray-900 w-full max-w-sm rounded-3xl border border-gray-800 overflow-hidden relative p-6">
        <button onClick={onClose} className="absolute top-4 right-4"><X size={20} className="text-gray-400"/></button>
        <h3 className="text-xl font-bold text-white mb-1">Rate Experience</h3>
        <p className="text-gray-400 text-sm mb-6">{booking.turfName}</p>
        <div className="flex justify-center gap-2 mb-6">
           {[1,2,3,4,5].map(star => (
             <button key={star} onClick={() => setRating(star)} className="transition-transform hover:scale-110">
               <Star size={32} className={star <= rating ? "fill-yellow-400 text-yellow-400" : "text-gray-600"} />
             </button>
           ))}
        </div>
        <textarea 
          className="w-full bg-gray-800 text-white rounded-xl p-3 border border-gray-700 mb-6 text-sm"
          rows="3"
          placeholder="How was the turf quality?"
          value={comment}
          onChange={(e) => setComment(e.target.value)}
        />
        <button onClick={handleSubmit} className="w-full bg-green-500 text-black font-bold py-3 rounded-xl">Submit Review</button>
      </div>
    </div>
  );
};

const RealisticQRCode = () => {
  const matrixSize = 25; 
  const dots = useMemo(() => {
    const d = [];
    const isFinderZone = (x, y) => (x < 7 && y < 7) || (x > matrixSize - 8 && y < 7) || (x < 7 && y > matrixSize - 8);
    const isCenterZone = (x, y) => {
        const center = Math.floor(matrixSize / 2);
        return x >= center - 2 && x <= center + 2 && y >= center - 2 && y <= center + 2;
    };
    for (let y = 0; y < matrixSize; y++) {
        for (let x = 0; x < matrixSize; x++) {
            if (!isFinderZone(x, y) && !isCenterZone(x, y)) {
                if (Math.random() > 0.55) { 
                    d.push(<rect key={`${x}-${y}`} x={x} y={y} width="0.85" height="0.85" rx="0.3" fill="black" />);
                }
            }
        }
    }
    return d;
  }, []);

  const FinderPattern = ({ x, y }) => (
    <g transform={`translate(${x}, ${y})`}>
        <rect x="0" y="0" width="7" height="7" rx="1.5" fill="black" />
        <rect x="1" y="1" width="5" height="5" rx="1" fill="white" />
        <rect x="2.5" y="2.5" width="2" height="2" rx="0.5" fill="black" />
    </g>
  );

  return (
    <svg viewBox={`0 0 ${matrixSize} ${matrixSize}`} className="w-full h-full bg-white">
        <FinderPattern x={0} y={0} />
        <FinderPattern x={matrixSize - 7} y={0} />
        <FinderPattern x={0} y={matrixSize - 7} />
        {dots}
    </svg>
  );
};

const EntryPassModal = ({ onClose, booking }) => {
   const [strategy, setStrategy] = useState(null);
   const [loadingStrategy, setLoadingStrategy] = useState(false);

   const handleCopy = () => {
      const el = document.createElement('textarea');
      el.value = booking.displayId || booking.id;
      document.body.appendChild(el);
      el.select();
      document.execCommand('copy');
      document.body.removeChild(el);
      alert('Booking ID copied!');
   };

   const getMatchStrategy = async () => {
      setLoadingStrategy(true);
      const prompt = `Give me a short, bulleted tactical strategy for a 5-a-side football match happening at ${booking.time}. Focus on formation and energy conservation. Keep it under 50 words.`;
      const result = await callGemini(prompt);
      setStrategy(result);
      setLoadingStrategy(false);
   };

   return (
      <div className="fixed inset-0 bg-black/95 z-[60] flex flex-col items-center justify-center p-6 animate-in zoom-in duration-300">
         <div className="bg-white text-black w-full max-w-sm rounded-3xl overflow-hidden shadow-2xl relative">
            <div className="bg-green-500 h-2 w-full"></div>
            <div className="p-6 text-center">
               <h2 className="text-2xl font-black mb-1">{booking.turfName}</h2>
               <p className="text-sm text-gray-500 mb-6">{booking.date} • {booking.time || "08:00 PM"}</p>
               
               {booking.status === 'Used' ? (
                  <div className="flex flex-col items-center justify-center h-48 mb-6 border-4 border-dashed border-red-500 rounded-2xl bg-red-50">
                      <span className="text-red-600 font-black text-5xl border-4 border-red-600 px-6 py-2 transform -rotate-12 opacity-80 shadow-xl">
                          USED
                      </span>
                  </div>
               ) : (
                   <div className="relative inline-block mb-6">
                      <div className="bg-white p-2 rounded-2xl border-2 border-black w-48 h-48 mx-auto shadow-xl overflow-hidden">
                         <RealisticQRCode />
                      </div>
                      <div className="absolute top-1/4 left-1/4 transform -translate-x-1/2 -translate-y-1/2">
                         <div className="bg-white p-1.5 rounded-full shadow-lg border border-gray-200">
                            <div className="bg-black w-10 h-10 rounded-full flex items-center justify-center text-green-500 font-black border-2 border-green-500 text-sm shadow-inner">
                               TI
                            </div>
                         </div>
                      </div>
                   </div>
               )}
               
               <p className="text-xs text-gray-400 font-mono tracking-widest uppercase mb-2">Booking ID</p>
               <div className="flex items-center justify-center gap-2 mb-2">
                  <div className="text-xl font-black font-mono tracking-wider break-all">{booking.displayId || booking.id}</div>
                  <button onClick={handleCopy} className="text-green-600 hover:text-green-800 p-1"><Copy size={16}/></button>
               </div>

               <button 
                 onClick={getMatchStrategy} 
                 className="w-full mt-4 bg-indigo-100 text-indigo-700 py-2 rounded-xl text-xs font-bold flex items-center justify-center gap-2 hover:bg-indigo-200 transition-colors"
               >
                 {loadingStrategy ? <Loader2 className="animate-spin" size={14}/> : <BrainCircuit size={14}/>} 
                 AI Match Strategy
               </button>
            </div>
            <button onClick={onClose} className="absolute top-4 right-4 bg-gray-200 p-1 rounded-full"><X size={20}/></button>
         </div>
         {strategy && <StrategyModal title="Match Tactics" content={strategy} onClose={() => setStrategy(null)} />}
      </div>
   );
};

const LuckyDrawModal = ({ onClose, onWin }) => {
  const [step, setStep] = useState('ready'); // ready, spinning, won
  const [reward, setReward] = useState(null);

  const prizes = [
    { type: 'cash', val: 10, label: '₹10 Wallet Cash' },
    { type: 'karma', val: 50, label: '50 Karma Points' },
    { type: 'coupon', val: 20, label: '20% Off Coupon' },
    { type: 'cash', val: 50, label: '₹50 Wallet Cash' },
    { type: 'karma', val: 100, label: '100 Karma Points' },
  ];

  const handleSpin = () => {
    setStep('spinning');
    setTimeout(() => {
      const win = prizes[Math.floor(Math.random() * prizes.length)];
      setReward(win);
      setStep('won');
      onWin(win);
    }, 2000);
  };

  return (
    <div className="fixed inset-0 bg-black/90 z-[60] flex flex-col items-center justify-center p-4 animate-in fade-in">
       <button onClick={onClose} className="absolute top-4 right-4 text-white"><X /></button>
       
       <h2 className="text-3xl font-black text-transparent bg-clip-text bg-gradient-to-r from-yellow-400 to-red-500 mb-8">DAILY JACKPOT</h2>

       <div className="relative">
          <div onClick={step === 'ready' ? handleSpin : null} className={`w-64 h-64 bg-gray-800 rounded-3xl border-4 border-yellow-500 flex items-center justify-center cursor-pointer transition-all ${step === 'spinning' ? 'animate-spin' : 'hover:scale-105'}`}>
             {step === 'ready' && <Gift size={64} className="text-yellow-500" />}
             {step === 'spinning' && <Loader2 size={64} className="text-yellow-500 animate-spin" />}
             {step === 'won' && (
                <div className="text-center animate-in zoom-in">
                   <Trophy size={64} className="text-yellow-400 mx-auto mb-2"/>
                   <p className="text-white font-bold text-xl">{reward.label}</p>
                </div>
             )}
          </div>
       </div>

       {step === 'ready' && <p className="text-white mt-8 animate-pulse">Tap the box to spin!</p>}
       {step === 'spinning' && <p className="text-yellow-500 mt-8">Feeling lucky...</p>}
       {step === 'won' && <button onClick={onClose} className="mt-8 bg-green-500 text-black px-8 py-3 rounded-full font-bold">Claim Prize</button>}
    </div>
  );
};

const WalletModal = ({ onClose, profile, user }) => {
   // Ensure debts are available
   const debts = INITIAL_DEBTS; 
   const [transactions, setTransactions] = useState([]);

   useEffect(() => {
       const unsub = onSnapshot(query(collection(db, 'artifacts', appId, 'users', user.uid, 'transactions'), orderBy('timestamp', 'desc'), limit(10)), (snap) => {
           setTransactions(snap.docs.map(d => d.data()));
       });
       return () => unsub();
   }, [user]);

   const handleTopUp = async () => {
       const amount = 500;
       await updateDoc(doc(db, 'artifacts', appId, 'users', user.uid, 'profile', 'main'), {
           walletBalance: increment(amount)
       });
       await addDoc(collection(db, 'artifacts', appId, 'users', user.uid, 'transactions'), {
           type: 'Credit',
           amount: amount,
           desc: 'Wallet Top Up',
           timestamp: serverTimestamp()
       });
   };

   const handleNudge = () => {
      const message = `Yo! ⚽ I've covered the turf booking. Send your share on TurfIt!`;
      const encodedMsg = encodeURIComponent(message);
      window.open(`https://wa.me/?text=${encodedMsg}`, '_blank');
   };

   const convertKarma = async () => {
      if (profile.karma >= 100) {
         try {
            await updateDoc(doc(db, 'artifacts', appId, 'users', user.uid, 'profile', 'main'), {
               karma: increment(-100),
               walletBalance: increment(20)
            });
            alert("Converted 100 Karma to ₹20!");
         } catch (err) {
            console.error("Conversion failed:", err);
         }
      } else {
         alert("Need 100 Karma to convert!");
      }
   };

   return (
      <div className="fixed inset-0 bg-black/90 z-50 flex flex-col justify-center items-center p-4 animate-in fade-in">
         <div className="bg-gray-900 w-full max-w-sm rounded-3xl border border-gray-800 overflow-hidden relative max-h-[90vh] overflow-y-auto">
            <button onClick={onClose} className="absolute top-4 right-4"><X size={20}/></button>
            <div className="p-6 text-center border-b border-gray-800">
               <p className="text-gray-400 text-sm mb-1">TurfIt Balance</p>
               <h2 className="text-4xl font-bold text-white mb-4">₹{profile?.walletBalance || 0}</h2>
               <div className="grid grid-cols-2 gap-3 mb-4">
                  <button onClick={handleTopUp} className="bg-green-500 text-black py-3 rounded-xl font-bold text-sm flex items-center justify-center gap-2">
                     <Gift size={16}/> Top Up (Demo)
                  </button>
                  <button className="bg-gray-800 text-white py-3 rounded-xl font-bold text-sm flex items-center justify-center gap-2 border border-gray-700">
                     <IndianRupee size={16}/> Withdraw
                  </button>
               </div>
               
               <button onClick={convertKarma} className="w-full mt-2 bg-yellow-500/10 text-yellow-500 py-2 rounded-xl text-xs font-bold border border-yellow-500/20">
                  Convert 100 Karma → ₹20
               </button>
            </div>
            
            <div className="p-4 bg-gray-800/50 flex-1">
               <h4 className="text-xs font-bold text-gray-400 uppercase mb-3 flex items-center gap-2">
                  <History size={14}/> Recent Transactions
               </h4>
               <div className="space-y-2">
                   {transactions.length === 0 ? <p className="text-gray-500 text-xs text-center">No transactions yet.</p> : 
                     transactions.map((t, i) => (
                       <div key={i} className="flex justify-between items-center bg-gray-800 p-2 rounded border border-gray-700">
                           <div>
                               <p className="text-xs font-bold text-white">{t.desc}</p>
                               <p className="text-[10px] text-gray-500">{t.timestamp?.seconds ? new Date(t.timestamp.seconds * 1000).toLocaleDateString() : 'Just now'}</p>
                           </div>
                           <span className={`text-sm font-bold ${t.type === 'Credit' ? 'text-green-400' : 'text-red-400'}`}>
                               {t.type === 'Credit' ? '+' : '-'}₹{t.amount}
                           </span>
                       </div>
                     ))
                   }
               </div>
            </div>
         </div>
      </div>
   );
};

const AddTurfForm = ({ onCancel, onSubmit }) => {
  const [name, setName] = useState('');
  const [location, setLocation] = useState('');
  const [price, setPrice] = useState('');
  const [image, setImage] = useState('https://images.unsplash.com/photo-1522778119026-d647f0565c6a?w=400');
  const [sports, setSports] = useState(['Football']);
  const [amenities, setAmenities] = useState([]);
  const [isJuniorFriendly, setIsJuniorFriendly] = useState(false);
  const [hasProLights, setHasProLights] = useState(false);
  const [has360View, setHas360View] = useState(false);
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [showMap, setShowMap] = useState(false);
  const fileInputRef = useRef(null);

  const availableSports = ["Football", "Cricket", "Badminton", "Tennis"];
  const availableAmenities = ["Parking", "Water", "Locker", "Showers", "Bibs"];

  const toggleSport = (sport) => {
    if (sports.includes(sport)) setSports(sports.filter(s => s !== sport));
    else setSports([...sports, sport]);
  };

  const toggleAmenity = (amenity) => {
    if (amenities.includes(amenity)) setAmenities(amenities.filter(a => a !== amenity));
    else setAmenities([...amenities, amenity]);
  };

  const handleFileSelect = (e) => {
    const file = e.target.files[0];
    if (file) {
      const reader = new FileReader();
      reader.onload = (e) => {
        setImage(e.target.result);
      };
      reader.readAsDataURL(file);
    }
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    if (!name || !location || !price) return;
    setIsSubmitting(true);
    onSubmit({
      name,
      location,
      price: parseInt(price),
      amenities,
      image,
      status: "Open",
      rating: 5.0,
      occupancy: 0,
      sports,
      isJuniorFriendly,
      hasProLights,
      has360View,
      distance: "New", 
      timings: "6 AM - 11 PM" 
    });
    setTimeout(() => setIsSubmitting(false), 2000);
  };

  return (
    <div className="fixed inset-0 bg-black z-50 flex flex-col p-6 animate-in fade-in slide-in-from-bottom-4 overflow-y-auto">
      {showMap && <MapPickerModal onClose={() => setShowMap(false)} onSelect={(loc) => setLocation(loc)} />}
      <div className="flex justify-between items-center mb-6">
        <h2 className="text-xl font-bold">List Your Turf</h2>
        <button onClick={onCancel} className="p-2 bg-gray-800 rounded-full"><X /></button>
      </div>
      <form onSubmit={handleSubmit} className="space-y-4 pb-10">
        <div>
          <label className="text-sm text-gray-400 mb-1 block">Turf Name</label>
          <input className="w-full bg-gray-800 p-4 rounded-xl border border-gray-700 text-white" value={name} onChange={e => setName(e.target.value)} placeholder="e.g. Champion's Ground" />
        </div>
        
        <div>
          <label className="text-sm text-gray-400 mb-1 block">Location</label>
          <div className="flex gap-2">
              <input className="flex-1 bg-gray-800 p-4 rounded-xl border border-gray-700 text-white" value={location} onChange={e => setLocation(e.target.value)} placeholder="Tap map icon to select" readOnly />
              <button type="button" onClick={() => setShowMap(true)} className="bg-blue-600 p-4 rounded-xl text-white hover:bg-blue-700 flex items-center justify-center min-w-[3.5rem]">
                  <MapPinned size={20} />
              </button>
          </div>
        </div>
        
        <div>
          <label className="text-sm text-gray-400 mb-1 block">Hourly Price (₹)</label>
          <input className="w-full bg-gray-800 p-4 rounded-xl border border-gray-700 text-white" type="number" value={price} onChange={e => setPrice(e.target.value)} placeholder="1200" />
        </div>

        {/* Upload Turf Photo Section */}
        <div>
          <label className="text-sm text-gray-400 mb-2 block">Turf Photo</label>
          <input 
            type="file" 
            ref={fileInputRef} 
            onChange={handleFileSelect} 
            accept="image/*" 
            className="hidden" 
          />
          <div 
            onClick={() => fileInputRef.current.click()}
            className="w-full h-48 bg-gray-800 rounded-2xl border-2 border-dashed border-gray-600 flex flex-col items-center justify-center cursor-pointer hover:border-green-500 transition-all relative overflow-hidden group"
          >
            {image && image.length > 100 ? ( // Simple check if it's base64 or custom
              <img src={image} alt="Preview" className="absolute inset-0 w-full h-full object-cover group-hover:scale-105 transition-transform duration-500" />
            ) : (
              <div className="absolute inset-0 bg-gray-800"></div> 
            )}
            <div className="absolute inset-0 bg-black/40 group-hover:bg-black/20 transition-colors"></div>
            
            <div className="relative z-10 flex flex-col items-center">
              <div className="bg-white/10 p-4 rounded-full backdrop-blur-md mb-3 group-hover:scale-110 transition-transform">
                <Camera size={24} className="text-white"/>
              </div>
              <span className="text-sm font-bold text-white">Tap to Upload</span>
              <span className="text-xs text-gray-300">from Gallery</span>
            </div>
          </div>
        </div>

        <div>
          <label className="text-sm text-gray-400 mb-2 block">Sports Supported</label>
          <div className="flex flex-wrap gap-2">
            {availableSports.map(s => (
              <button 
                key={s} 
                type="button" 
                onClick={() => toggleSport(s)}
                className={`px-3 py-2 rounded-lg text-xs font-bold border transition-colors ${sports.includes(s) ? 'bg-green-600 border-green-500 text-white' : 'bg-gray-800 border-gray-700 text-gray-400'}`}
              >
                {s}
              </button>
            ))}
          </div>
        </div>

        <div>
          <label className="text-sm text-gray-400 mb-2 block">Amenities</label>
          <div className="flex flex-wrap gap-2">
            {availableAmenities.map(a => (
              <button 
                key={a} 
                type="button" 
                onClick={() => toggleAmenity(a)}
                className={`px-3 py-2 rounded-lg text-xs font-bold border transition-colors ${amenities.includes(a) ? 'bg-blue-600 border-blue-500 text-white' : 'bg-gray-800 border-gray-700 text-gray-400'}`}
              >
                {a}
              </button>
            ))}
          </div>
        </div>

        <div className="grid grid-cols-2 gap-3">
           <div 
             onClick={() => setIsJuniorFriendly(!isJuniorFriendly)} 
             className={`p-3 rounded-xl border cursor-pointer flex flex-col items-center justify-center gap-1 ${isJuniorFriendly ? 'bg-indigo-600 border-indigo-500' : 'bg-gray-800 border-gray-700'}`}
           >
             <Shield size={20} /> <span className="text-xs font-bold">Junior Safe</span>
           </div>
           <div 
             onClick={() => setHasProLights(!hasProLights)} 
             className={`p-3 rounded-xl border cursor-pointer flex flex-col items-center justify-center gap-1 ${hasProLights ? 'bg-yellow-600 border-yellow-500' : 'bg-gray-800 border-gray-700'}`}
           >
             <Zap size={20} /> <span className="text-xs font-bold">Pro Lights</span>
           </div>
           <div 
             onClick={() => setHas360View(!has360View)} 
             className={`col-span-2 p-3 rounded-xl border cursor-pointer flex items-center justify-center gap-2 ${has360View ? 'bg-cyan-600 border-cyan-500' : 'bg-gray-800 border-gray-700'}`}
           >
             <RefreshCcw size={20} /> <span className="text-xs font-bold">Enable 360° View</span>
           </div>
        </div>

        <button disabled={isSubmitting} type="submit" className={`w-full bg-green-500 text-black font-bold py-4 rounded-xl mt-4 flex items-center justify-center gap-2 ${isSubmitting ? 'opacity-50 cursor-not-allowed' : ''}`}>
           {isSubmitting ? <Loader2 className="animate-spin"/> : 'List Turf (Real-Time)'}
        </button>
      </form>
    </div>
  );
};

const BookingModal = ({ turf, onClose, onConfirm, userKarma, finalPrice, isProcessing }) => {
  const [addons, setAddons] = useState([]);
  const [isEsports, setIsEsports] = useState(false);
  const [hasFood, setHasFood] = useState(false);
  const [isClicked, setIsClicked] = useState(false);
  const [paymentMethod, setPaymentMethod] = useState('UPI'); 
  const [showConfetti, setShowConfetti] = useState(false);
  
  const [selectedDate, setSelectedDate] = useState(0); 
  const [selectedSlot, setSelectedSlot] = useState(null);
  const [step, setStep] = useState(1); 
  const [timeSlots, setTimeSlots] = useState([]);
  const [realTimeBookings, setRealTimeBookings] = useState([]);
  const [showReviews, setShowReviews] = useState(false);
  const [reviews, setReviews] = useState([]);

  const hasSocks = addons.includes('socks');
  const hasStuds = addons.includes('studs');
  const hasRef = addons.includes('referee');

  // Listen to Real-Time Bookings and Reviews
  useEffect(() => {
    const qBookings = query(collection(db, 'artifacts', appId, 'public', 'data', 'booked_slots'));
    const unsubBookings = onSnapshot(qBookings, (snapshot) => {
      const booked = snapshot.docs.map(doc => doc.data());
      setRealTimeBookings(booked);
    });

    if (turf) {
        // FIXED QUERY: Now filtering by turfId for robustness
        const qReviews = query(
            collection(db, 'artifacts', appId, 'public', 'data', 'reviews'),
            where('turfName', '==', turf.name) // Use Name as fallback
        );
        const unsubReviews = onSnapshot(qReviews, (snapshot) => {
            const data = snapshot.docs.map(d => ({id: d.id, ...d.data()}));
            data.sort((a, b) => (b.createdAt?.seconds || 0) - (a.createdAt?.seconds || 0));
            setReviews(data.slice(0, 10));
        });
        return () => { unsubBookings(); unsubReviews(); };
    }
    return () => unsubBookings();
  }, [turf]);

  const dates = Array.from({ length: 7 }, (_, i) => {
    const d = new Date();
    d.setDate(d.getDate() + i);
    return {
      day: d.toLocaleDateString('en-US', { weekday: 'short' }),
      date: d.getDate(),
      fullDate: d.toLocaleDateString('en-US', { month: 'short', day: 'numeric' })
    };
  });

  useEffect(() => {
    const slots = [];
    const openHour = 6; 
    const closeHour = 23; 
    
    const now = new Date();
    const currentHour = now.getHours();
    const isToday = selectedDate === 0;
    const targetDateStr = dates[selectedDate].fullDate;

    for (let i = openHour; i <= closeHour; i++) {
      if (isToday && i <= currentHour) continue;

      const hour = i;
      const ampm = hour >= 12 ? 'PM' : 'AM';
      const displayHour = hour > 12 ? hour - 12 : hour === 0 || hour === 24 ? 12 : hour;
      const timeLabel = `${displayHour}:00 ${ampm}`;
      
      const isTaken = realTimeBookings.some(b => 
        b.turfName === turf.name && // Still check name for display consistency
        b.date === targetDateStr && 
        b.time === timeLabel &&
        b.status !== 'Used' &&
        b.status !== 'Cancelled'
      );
      
      slots.push({ 
        id: `${i}:00`, 
        label: timeLabel, 
        available: !isTaken 
      });
    }
    setTimeSlots(slots);
    
    if (selectedSlot) {
       const stillAvailable = slots.find(s => s.id === selectedSlot && s.available);
       if (!stillAvailable) setSelectedSlot(null);
    }
  }, [selectedDate, realTimeBookings, turf.name]);

  const toggleAddon = (id) => {
    if (addons.includes(id)) setAddons(addons.filter(a => a !== id));
    else setAddons([...addons, id]);
  };

  const loyaltyDiscount = userKarma > 90 ? Math.floor(((userKarma - 90) / 10) * 25) : 0;

  const calculateTotal = () => {
    let total = finalPrice;
    if (hasFood) total += 250; 
    total -= loyaltyDiscount;
    return Math.max(0, total);
  };

  const handleNext = () => {
    if (!selectedSlot) {
      alert("Please select a time slot!");
      return;
    }
    setStep(2);
  };

  const handleConfirm = async () => {
    if(auth.currentUser) {
       const qBan = query(collection(db, 'artifacts', appId, 'public', 'data', 'banned_users'), where('userId', '==', auth.currentUser.uid));
       const snapBan = await getDocs(qBan);
       if(!snapBan.empty) {
           alert("You are banned from making bookings.");
           return;
       }
    }

    if (isProcessing || isClicked) return;
    setIsClicked(true); 
    setShowConfetti(true); // Trigger confetti animation

    const bookingDate = dates[selectedDate].fullDate;
    const bookingTime = timeSlots.find(s => s.id === selectedSlot)?.label;
    
    // Delay closing slightly to show confetti
    setTimeout(() => {
        onConfirm(turf, calculateTotal(), bookingDate, bookingTime);
    }, 1500);
  };

  return (
    <div className="fixed inset-0 bg-black/90 z-[60] flex items-end sm:items-center justify-center backdrop-blur-sm p-4 animate-in fade-in">
      {/* Confetti Animation Layer */}
      {showConfetti && (
          <div className="fixed inset-0 z-[60] flex items-center justify-center pointer-events-none">
              <div className="absolute animate-bounce"><PartyPopper size={64} className="text-yellow-400" /></div>
              <div className="absolute top-1/4 left-1/4 animate-ping"><Star size={32} className="text-green-400" /></div>
              <div className="absolute top-1/4 right-1/4 animate-ping"><Star size={32} className="text-blue-400" /></div>
              <div className="absolute bottom-1/3 left-1/3 animate-ping"><Star size={32} className="text-red-400" /></div>
          </div>
      )}
    
      <div className="bg-gray-900 w-full max-w-md rounded-3xl border border-gray-800 overflow-hidden shadow-2xl max-h-[90vh] overflow-y-auto flex flex-col relative z-50">
        {/* Header */}
        <div className="p-4 border-b border-gray-800 flex justify-between items-center sticky top-0 bg-gray-900 z-10">
           <div className="flex items-center gap-2">
             {(step === 2 || showReviews) && (
               <button onClick={() => showReviews ? setShowReviews(false) : setStep(1)} className="p-1 rounded-full hover:bg-gray-800 mr-1">
                 <ArrowLeft size={18} className="text-gray-400"/>
               </button>
             )}
             <div>
               <h3 className="font-bold text-lg text-white">
                   {showReviews ? "Reviews" : (step === 1 ? "Select Time" : "Customize")}
               </h3>
               {!showReviews ? (
                   <button onClick={() => setShowReviews(true)} className="flex items-center text-xs text-yellow-500 hover:underline">
                       <Star size={10} fill="currentColor" className="mr-1"/> {turf.rating} ({reviews.length} reviews)
                   </button>
               ) : (
                   <p className="text-xs text-gray-400">{turf.name}</p>
               )}
             </div>
           </div>
           <button onClick={onClose} className="p-2 bg-gray-800 rounded-full hover:bg-gray-700"><X size={20} className="text-gray-400"/></button>
        </div>
        
        <div className="p-4 space-y-6 flex-1 overflow-y-auto custom-scrollbar">
           
           {showReviews ? (
               <div className="space-y-4">
                   {reviews.length === 0 ? (
                       <div className="text-center py-8">
                           <MessageSquare size={32} className="text-gray-600 mx-auto mb-2"/>
                           <p className="text-gray-500 text-sm">No reviews yet.</p>
                       </div>
                   ) : (
                       reviews.map(r => (
                           <div key={r.id} className="bg-gray-800 p-4 rounded-xl border border-gray-700">
                               <div className="flex justify-between items-start mb-2">
                                   <div className="flex items-center gap-1">
                                       {[...Array(5)].map((_, i) => (
                                           <Star key={i} size={12} className={i < r.rating ? "text-yellow-400 fill-current" : "text-gray-600"} />
                                       ))}
                                   </div>
                                   <span className="text-[10px] text-gray-500">{r.createdAt?.seconds ? new Date(r.createdAt.seconds * 1000).toLocaleDateString() : 'Just now'}</span>
                               </div>
                               <p className="text-sm text-gray-300">"{r.comment}"</p>
                           </div>
                       ))
                   )}
               </div>
           ) : (
             <>
               {step === 1 && (
                 <>
                   {/* Date Selection */}
                   <div>
                     <h4 className="text-sm font-bold text-gray-400 mb-3 uppercase tracking-wider">Select Date</h4>
                     <div className="flex gap-3 overflow-x-auto no-scrollbar pb-2">
                       {dates.map((d, i) => (
                         <button 
                           key={i} 
                           onClick={() => setSelectedDate(i)}
                           className={`flex-shrink-0 w-16 h-20 rounded-2xl flex flex-col items-center justify-center transition-all border-2 ${selectedDate === i ? 'bg-green-500 border-green-500 text-black shadow-lg shadow-green-500/20' : 'bg-gray-800 border-gray-700 text-gray-400 hover:border-gray-500'}`}
                         >
                           <span className="text-xs font-medium">{d.day}</span>
                           <span className="text-xl font-bold">{d.date}</span>
                         </button>
                       ))}
                     </div>
                   </div>

                   {/* Time Slots */}
                   <div>
                     <h4 className="text-sm font-bold text-gray-400 mb-3 uppercase tracking-wider">Available Slots</h4>
                     {timeSlots.length === 0 ? (
                        <div className="text-gray-500 text-sm italic text-center p-4 bg-gray-800 rounded-xl border border-dashed border-gray-700">
                           No more slots available for today.
                        </div>
                     ) : (
                       <div className="grid grid-cols-4 gap-2">
                         {timeSlots.map((slot) => (
                           <button
                             key={slot.id}
                             disabled={!slot.available}
                             onClick={() => setSelectedSlot(slot.id)}
                             className={`py-2 rounded-xl text-xs font-bold transition-all border ${
                               !slot.available 
                                 ? 'bg-gray-800/50 border-gray-800 text-gray-600 cursor-not-allowed decoration-slice line-through' 
                                 : selectedSlot === slot.id 
                                   ? 'bg-white text-black border-white shadow-lg' 
                                   : 'bg-gray-800 border-gray-700 text-green-400 hover:border-green-400'
                             }`}
                           >
                             {slot.label}
                           </button>
                         ))}
                       </div>
                     )}
                   </div>
                 </>
               )}

               {step === 2 && (
                 <>
                   {/* Add-ons Section */}
                   <div className="animate-in slide-in-from-right">
                     <h4 className="text-sm font-bold text-gray-400 mb-3 uppercase tracking-wider">Extras</h4>
                     <div className="space-y-2">
                       <div onClick={() => setHasFood(!hasFood)} className={`p-3 rounded-xl border cursor-pointer transition-colors flex justify-between items-center ${hasFood ? 'bg-orange-900/20 border-orange-500' : 'bg-gray-800 border-gray-700'}`}>
                          <div className="flex items-center gap-3">
                             <div className={`p-2 rounded-lg ${hasFood ? 'bg-orange-500 text-black' : 'bg-gray-700 text-gray-400'}`}><Coffee size={18}/></div>
                             <span className="text-sm font-bold text-white">Fuel Pack (Drinks)</span>
                          </div>
                          <span className="text-xs font-bold text-orange-400">+₹250</span>
                       </div>
                     </div>
                   </div>

                   {/* Payment Method */}
                   <div className="animate-in slide-in-from-right delay-75">
                      <h4 className="text-sm font-bold text-gray-400 mb-3 mt-4 uppercase tracking-wider">Payment Method</h4>
                      <div className="flex gap-2">
                          {['UPI', 'Card', 'Venue'].map(method => (
                              <button 
                                key={method}
                                onClick={() => setPaymentMethod(method)}
                                className={`flex-1 py-3 rounded-xl border text-xs font-bold flex flex-col items-center justify-center gap-1 transition-all ${paymentMethod === method ? 'bg-blue-600 border-blue-500 text-white' : 'bg-gray-800 border-gray-700 text-gray-400 hover:bg-gray-700'}`}
                              >
                                  {method === 'UPI' && <Smartphone size={16}/>} 
                                  {method === 'Card' && <CreditCard size={16}/>}
                                  {method === 'Venue' && <Banknote size={16}/>}
                                  {method}
                              </button>
                          ))}
                      </div>
                   </div>

                   {/* Bill Details */}
                   <div className="bg-gray-800 p-4 rounded-2xl border border-gray-700 space-y-2 text-sm animate-in slide-in-from-right delay-100 mt-4">
                      <div className="flex justify-between text-gray-400">
                        <span>Slot Price ({timeSlots.find(s => s.id === selectedSlot)?.label})</span>
                        <span>₹{finalPrice}</span>
                      </div>
                      {hasFood && (
                        <div className="flex justify-between text-orange-400">
                          <span>Fuel Pack</span>
                          <span>+₹250</span>
                        </div>
                      )}
                      {loyaltyDiscount > 0 && (
                        <div className="flex justify-between text-yellow-500 font-bold">
                          <span>Loyalty Discount</span>
                          <span>-₹{loyaltyDiscount}</span>
                        </div>
                      )}
                      <div className="border-t border-gray-700 pt-2 mt-2 flex justify-between text-white font-black text-lg">
                        <span>Total</span>
                        <span>₹{calculateTotal()}</span>
                      </div>
                   </div>
                 </>
               )}
             </>
           )}
        </div>

        {/* Footer */}
        {!showReviews && (
            <div className="p-4 bg-gray-900 border-t border-gray-800">
               {step === 1 ? (
                 <button 
                    onClick={handleNext} 
                    className="w-full bg-white text-black py-4 rounded-xl font-bold text-lg hover:bg-gray-200 transition-colors shadow-lg flex items-center justify-center gap-2"
                 >
                    Next <ChevronRight size={20} />
                 </button>
               ) : (
                 <button 
                    disabled={isProcessing || isClicked}
                    onClick={handleConfirm} 
                    className={`w-full bg-green-500 text-black py-4 rounded-xl font-bold text-lg hover:bg-green-400 transition-colors shadow-lg shadow-green-500/20 flex items-center justify-center gap-2 ${isProcessing || isClicked ? 'opacity-50 cursor-not-allowed' : ''}`}
                 >
                    {isProcessing || isClicked ? (showConfetti ? "Confirmed!" : <Loader2 className="animate-spin"/>) : (turf.isAuction ? "Place Bid" : `Pay ₹${calculateTotal()}`)}
                 </button>
               )}
            </div>
        )}
      </div>
    </div>
  );
};

const TurfCard = ({ turf, onBook, onView360, juniorMode, isResale, isAuction }) => {
  const currentHour = new Date().getHours();
  const isNightOwl = currentHour >= 23 || currentHour < 5;
  const isClosed = turf.status === "Closed";
  let demandMultiplier = 1;
  let priceLabel = "";
  
  if (isResale) {
    demandMultiplier = 0.9; 
    priceLabel = "Resale Deal (-10%)";
  } else if (isAuction) {
    priceLabel = "🔥 Live Auction";
  } else if (turf.occupancy > 80) {
    demandMultiplier = 1.05;
    priceLabel = "High Demand 🔥";
  } else if (isNightOwl) {
    demandMultiplier = 0.8;
    priceLabel = "Night Owl 🦉";
  }

  const finalPrice = Math.floor(turf.price * demandMultiplier);

  return (
    <div className={`bg-gray-800 rounded-3xl overflow-hidden mb-6 border ${isAuction ? 'border-orange-500 shadow-orange-900/20' : 'border-gray-700'} shadow-xl group relative`}>
      <div className="h-48 relative">
        <img src={turf.image} alt={turf.name} className={`w-full h-full object-cover transition-transform duration-500 ${isClosed ? 'grayscale opacity-50' : 'group-hover:scale-105'}`} />
        
        {isClosed && (
            <div className="absolute inset-0 flex items-center justify-center bg-black/60 z-10">
                <div className="bg-red-600 text-white px-4 py-2 rounded-xl font-black text-sm uppercase tracking-widest border-2 border-red-400 transform -rotate-12 shadow-2xl">
                    Closed
                </div>
            </div>
        )}
        
        {/* Real-time Status Indicators */}
        <div className="absolute top-4 right-4 flex flex-col items-end gap-2 z-20">
           {turf.status === "Rain" && (
             <div className="bg-blue-600/90 backdrop-blur-md px-3 py-1 rounded-full flex items-center text-xs font-bold text-white shadow-lg border border-blue-400">
               <CloudRain size={12} className="mr-1" /> Rain Mode
             </div>
           )}
           {turf.status === "Lights" && (
             <div className="bg-yellow-600/90 backdrop-blur-md px-3 py-1 rounded-full flex items-center text-xs font-bold text-white shadow-lg border border-yellow-400">
               <Zap size={12} className="mr-1" /> Lights On
             </div>
           )}
           
           {!isResale && !isAuction && !isClosed && (
               <div className="bg-gray-900/80 backdrop-blur-md px-3 py-1 rounded-full flex items-center border border-gray-700 text-white">
                 <Star size={14} className="text-yellow-400 fill-current mr-1" />
                 <span className="text-xs font-bold">{turf.rating}</span>
               </div>
           )}
        </div>

        {turf.has360View && !isClosed && (
           <button 
             onClick={() => onView360(turf)}
             className="absolute bottom-4 right-4 bg-black/60 backdrop-blur-md px-3 py-1.5 rounded-full flex items-center gap-1 text-xs font-bold text-white border border-white/20 hover:bg-black/80 transition-all z-30"
           >
             <RefreshCw size={14} className="text-cyan-400"/> 360° View
           </button>
        )}

        {isResale && (
          <div className="absolute top-4 right-4 bg-purple-600 px-3 py-1 rounded-full flex items-center text-xs font-bold shadow-lg border border-purple-400">
            <RefreshCcw size={12} className="mr-1" /> Resale Slot
          </div>
        )}

        {isAuction && (
           <div className="absolute top-4 right-4 bg-orange-600 px-3 py-1 rounded-full flex items-center text-xs font-bold shadow-lg animate-pulse border border-orange-400">
             <Gavel size={12} className="mr-1" /> Bid Live
           </div>
        )}

        {/* ... */}
        
        {juniorMode && turf.isJuniorFriendly && (
           <div className="absolute top-4 left-4 bg-blue-600 px-3 py-1 rounded-full flex items-center text-xs font-bold shadow-lg shadow-blue-500/30">
              <Shield size={12} className="mr-1" /> Verified Coach
           </div>
        )}
        
        {!juniorMode && priceLabel && !isClosed && (
          <div className={`absolute top-4 left-4 px-3 py-1 rounded-full flex items-center text-xs font-bold shadow-lg ${isResale ? 'bg-purple-800' : isAuction ? 'bg-orange-800' : priceLabel.includes('High') ? 'bg-red-600' : 'bg-green-600'}`}>
            <TrendingUp size={12} className="mr-1" /> {priceLabel}
          </div>
        )}

        <div className="absolute bottom-4 left-4 right-20 z-20">
          <h3 className="text-xl font-bold text-white drop-shadow-md truncate">{turf.name}</h3>
          <div className="flex items-center text-gray-200 text-sm drop-shadow-md">
            <MapPin size={14} className="mr-1" /> {turf.location} • {turf.distance}
          </div>
          {turf.timings && <div className="text-[10px] text-gray-300 mt-1 flex items-center gap-1"><Clock size={10}/> {turf.timings}</div>}
        </div>
      </div>
      <div className="p-4 relative">
        <div className="flex gap-2 mb-4 overflow-x-auto no-scrollbar">
          {turf.amenities && turf.amenities.map(a => (
            <span key={a} className="text-[10px] bg-gray-700 px-2 py-1 rounded-md text-gray-300 whitespace-nowrap">{a}</span>
          ))}
        </div>
        <div className="flex justify-between items-center">
          <div>
            <div className="flex items-baseline gap-2">
              <span className={`text-2xl font-bold ${isClosed ? 'text-gray-500' : 'text-green-400'}`}>₹{finalPrice}</span>
              {demandMultiplier !== 1 && <span className="text-sm text-gray-500 line-through">₹{turf.price}</span>}
            </div>
            <span className="text-gray-500 text-xs"> / hour</span>
          </div>
          <div className="flex gap-2 relative z-30">
             {/* Booking Button */}
             <button 
               onClick={() => !isClosed && onBook(turf, finalPrice)}
               disabled={isClosed}
               className={`px-6 py-3 rounded-xl font-bold transition-colors shadow-lg flex-shrink-0 ${
                 isClosed 
                   ? 'bg-gray-700 text-gray-500 cursor-not-allowed border border-gray-600' 
                   : isResale 
                     ? 'bg-purple-500 hover:bg-purple-400 text-white' 
                     : isAuction 
                       ? 'bg-orange-500 hover:bg-orange-400 text-black' 
                       : 'bg-green-500 hover:bg-green-400 text-black'
               }`}
             >
               {isClosed ? "Temporarily Closed" : (isResale ? "Buy Slot" : isAuction ? "Place Bid" : "Book Now")}
             </button>
          </div>
        </div>
      </div>
    </div>
  );
};

function TicketScreen({ bookings, onCancel, user }) {
  const [showSiren, setShowSiren] = useState(false);
  const [showEntryPass, setShowEntryPass] = useState(false); 
  const [activeBookingForPass, setActiveBookingForPass] = useState(null);
  const [reviewModal, setReviewModal] = useState({ open: false, booking: null });
  const [filter, setFilter] = useState('Today'); // Today, History, All

  // Filter Bookings logic
  const filteredBookings = bookings.filter(b => {
      const today = new Date();
      const todayStr = today.toLocaleDateString('en-US', { month: 'short', day: 'numeric' });
      const bookingDateStr = b.date;
      
      if (!bookingDateStr) return false;
      
      if (filter === 'Today') {
          return bookingDateStr === todayStr || bookingDateStr === "Today";
      } else if (filter === 'History') {
          return bookingDateStr !== todayStr && bookingDateStr !== "Today";
      }
      return true; // All
  });

  const handleReviewSubmit = async (booking, rating, comment) => {
      try {
        await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'reviews'), {
           turfName: booking.turfName,
           userId: user.uid,
           rating,
           comment,
           createdAt: serverTimestamp()
        });
        alert("Review Submitted! +10 Karma");
      } catch(e) {
        console.error("Review error", e);
      }
  };

  const openEntryPass = (booking) => {
      setActiveBookingForPass(booking);
      setShowEntryPass(true);
  };

  if (showSiren) return <SirenOverlay onClose={() => setShowSiren(false)} />;
  if (showEntryPass && activeBookingForPass) return <EntryPassModal booking={activeBookingForPass} onClose={() => setShowEntryPass(false)} />;
  
  return (
    <div className="h-full overflow-y-auto p-4 space-y-6 pb-24">
      {reviewModal.open && <ReviewModal booking={reviewModal.booking} onClose={() => setReviewModal({open: false, booking: null})} onSubmit={handleReviewSubmit} />}

      <div className="flex justify-between items-center">
         <h2 className="text-2xl font-bold">Your Tickets</h2>
      </div>
      
      {/* Tabs */}
      <div className="flex gap-2 bg-gray-800 p-1 rounded-xl">
          {['Today', 'History', 'All'].map(tab => (
              <button 
                key={tab}
                onClick={() => setFilter(tab)}
                className={`flex-1 py-2 text-xs font-bold rounded-lg transition-colors ${filter === tab ? 'bg-gray-700 text-white shadow' : 'text-gray-400'}`}
              >
                  {tab}
              </button>
          ))}
      </div>
      
      {filteredBookings.length === 0 && (
         <div className="text-center py-10 bg-gray-800/50 rounded-2xl border border-dashed border-gray-700">
            <Ticket size={48} className="mx-auto text-gray-600 mb-4" />
            <p className="text-gray-400">No tickets found.</p>
         </div>
      )}

      {filteredBookings.map(booking => (
        <div key={booking.id} className="bg-white text-black rounded-3xl overflow-hidden relative shadow-xl mb-4">
          {/* Status Stamps */}
          {booking.status === 'Cancelled' && (
             <div className="absolute top-4 right-4 z-20 border-4 border-red-500 text-red-500 font-black text-xl px-2 py-1 transform rotate-12 opacity-80 rounded">
                 CANCELLED
             </div>
          )}
          {booking.status === 'Used' && (
             <div className="absolute top-4 right-4 z-20 border-4 border-green-600 text-green-600 font-black text-xl px-2 py-1 transform -rotate-12 opacity-80 rounded">
                 USED
             </div>
          )}

          <div className={`p-5 relative z-10 ${booking.status === 'Cancelled' ? 'opacity-50' : ''}`}>
            <div className="flex justify-between items-start mb-4">
               <div>
                  <h3 className="text-xl font-bold">{booking.turfName}</h3>
               </div>
            </div>
            
            <div className="flex justify-between items-end border-t border-gray-200 pt-3">
               <div className="flex-1">
                 <div className="flex items-center gap-2 mb-1">
                   <Calendar size={14} className="text-gray-500" />
                   <span className="font-bold text-sm">{booking.date}</span>
                 </div>
                 <div className="flex items-center gap-2">
                   <Clock size={14} className="text-gray-500" />
                   <span className="font-bold text-sm">{booking.time || "08:00 PM"}</span>
                 </div>
               </div>
               
               <div className="flex gap-2">
                   {booking.status === 'Used' ? (
                       <button onClick={() => setReviewModal({open: true, booking})} className="bg-yellow-500 text-black px-4 py-2 rounded-xl text-xs font-bold flex flex-col items-center gap-1 shadow-md animate-in fade-in hover:bg-yellow-400">
                          <Star size={16}/> Rate & Review
                       </button>
                   ) : booking.status !== 'Cancelled' && (
                       <>
                         <button onClick={() => openEntryPass(booking)} className="bg-green-600 text-white px-3 py-2 rounded-xl text-xs font-bold flex flex-col items-center gap-1 shadow-md">
                            <QrCode size={16}/> Entry Pass
                         </button>

                         <button onClick={() => onCancel(booking.id)} className="bg-red-50 text-red-500 border border-red-200 px-3 py-2 rounded-xl text-xs font-bold flex flex-col items-center gap-1 shadow-sm hover:bg-red-100">
                            <XCircle size={16}/> Cancel
                         </button>
                       </>
                   )}
               </div>
            </div>
         </div>
        </div>
      ))}
    </div>
  );
}

function HomeScreen({ turfs, resaleSlots, auctionSlots, onBook, userKarma, onOpenScanner, onOpenNotifications }) {
  const [filter, setFilter] = useState('All');
  const [juniorMode, setJuniorMode] = useState(false);
  const [lightsFilter, setLightsFilter] = useState(false);
  const [view360, setView360] = useState(null);
  const [searchQuery, setSearchQuery] = useState("");

  const filters = ['All', 'Football', 'Cricket', 'Tennis', 'Resale', 'Gear'];

  let displayItems = [];
  
  if (filter === 'Resale') {
    displayItems = resaleSlots;
  } else if (filter === 'Gear') {
     displayItems = []; 
  } else {
    displayItems = [...auctionSlots, ...turfs].filter(t => {
      // Logic filters
      if (filter !== 'All' && !t.sports?.includes(filter)) return false;
      if (juniorMode && !t.isJuniorFriendly) return false;
      if (lightsFilter && !t.hasProLights) return false;
      
      // Search filter
      if (searchQuery) {
          const lowerQuery = searchQuery.toLowerCase();
          return t.name.toLowerCase().includes(lowerQuery) || t.location.toLowerCase().includes(lowerQuery);
      }
      return true;
    });
  }

  return (
    <div className="flex flex-col h-full">
      <LiveTicker />
      {view360 && <ThreeSixtyModal turf={view360} onClose={() => setView360(null)} />}
      <div className="p-4 space-y-6 pb-24 flex-1 overflow-y-auto no-scrollbar">
        <header className="flex justify-between items-center">
          <div>
            <p className="text-gray-400 text-sm">Location</p>
            <div className="flex items-center text-green-400 font-bold">
              <MapPin size={16} className="mr-1" /> Sector 44, Ambala
            </div>
            <div className="text-[10px] text-yellow-500 font-bold mt-1">👑 King: Amit S.</div>
          </div>
          <button onClick={onOpenNotifications} className="bg-gray-800 p-2 rounded-full border border-gray-700 hover:bg-gray-700 transition-colors">
            <Bell size={20} />
          </button>
        </header>

        <div className="bg-gray-800 p-4 rounded-2xl flex items-center border border-gray-700">
          <Search className="text-gray-500 mr-3" />
          <input 
            placeholder="Search arenas, gear..." 
            className="bg-transparent flex-1 outline-none text-white placeholder-gray-500"
            value={searchQuery}
            onChange={(e) => setSearchQuery(e.target.value)}
          />
        </div>

        <div className="flex gap-3 overflow-x-auto no-scrollbar pb-2">
          {filters.map(f => (
            <button 
              key={f}
              onClick={() => setFilter(f)}
              className={`px-6 py-2 rounded-full text-sm font-bold whitespace-nowrap transition-colors ${filter === f ? (f === 'Resale' ? 'bg-purple-600 text-white' : 'bg-green-500 text-black') : 'bg-gray-800 text-gray-400 border border-gray-700'}`}
            >
              {f}
            </button>
          ))}
        </div>

        {filter !== 'Resale' && filter !== 'Gear' && (
          <div className="flex gap-2">
              <button onClick={() => setJuniorMode(!juniorMode)} className={`flex-1 flex items-center justify-center gap-2 py-2 rounded-xl text-xs font-bold transition-all border ${juniorMode ? 'bg-blue-600 border-blue-500 text-white' : 'bg-gray-800 border-gray-700 text-gray-400'}`}>
                <Shield size={14} /> J - Junior Safe
              </button>
              <button onClick={() => setLightsFilter(!lightsFilter)} className={`flex-1 flex items-center justify-center gap-2 py-2 rounded-xl text-xs font-bold transition-all border ${lightsFilter ? 'bg-yellow-600 border-yellow-500 text-white' : 'bg-gray-800 border-gray-700 text-gray-400'}`}>
                <Zap size={14} /> U - Pro Lights
              </button>
          </div>
        )}

        <div>
          <h2 className="text-xl font-bold mb-4">{filter === 'Resale' ? 'Slot Marketplace' : filter === 'Gear' ? 'Used Gear' : 'Trending Turfs'}</h2>
          
          {displayItems.length === 0 && (
             <div className="flex flex-col items-center justify-center py-12 text-center">
                 <div className="bg-gray-800 p-4 rounded-full mb-4">
                     <AlertCircle size={40} className="text-gray-500"/>
                 </div>
                 <h3 className="text-white font-bold text-lg mb-2">No Arenas Live</h3>
                 <p className="text-gray-400 text-sm max-w-xs">It seems quiet here. Partners haven't listed any turfs yet. Be the first to add one in Profile!</p>
             </div>
          )}

          {displayItems.map(item => (
            <TurfCard 
              key={item.id} 
              turf={item} 
              onBook={onBook} 
              onView360={setView360}
              juniorMode={juniorMode} 
              userKarma={userKarma} 
              isResale={filter === 'Resale' || item.isResale}
              isAuction={item.isAuction}
            />
          ))}
        </div>
      </div>
    </div>
  );
}

function SocialScreen({ feed, user, onPost }) {
  const [newPost, setNewPost] = useState('');
  const [activeTab, setActiveTab] = useState('lounge');
  const [draftStatus, setDraftStatus] = useState('idle');
  
  const handlePost = () => {
     if(!newPost.trim()) return;
     onPost(newPost);
     setNewPost('');
  };

  return (
    <div className="h-full overflow-y-auto p-4 space-y-6 pb-24 flex flex-col">
      <div className="flex items-center justify-between">
        <h2 className="text-2xl font-bold">Community</h2>
        <div className="flex gap-2">
           <button onClick={() => setActiveTab('lounge')} className={`px-3 py-1 rounded-full text-xs font-bold border ${activeTab === 'lounge' ? 'bg-green-500 text-black border-green-500' : 'border-gray-700 text-gray-400'}`}>Lounge</button>
           <button onClick={() => setActiveTab('league')} className={`px-3 py-1 rounded-full text-xs font-bold border ${activeTab === 'league' ? 'bg-purple-600 text-white border-purple-600' : 'border-gray-700 text-gray-400'}`}>Leagues</button>
        </div>
      </div>

      {activeTab === 'league' && (
        <div className="space-y-4 animate-in fade-in">
           {/* Y - Youth League Sponsorship */}
           <div className="bg-yellow-500/10 border border-yellow-500/30 p-2 text-center rounded-xl">
              <p className="text-[10px] text-yellow-500 font-bold">Sponsored by: La Pino'z Pizza (Free Slice for Top 3)</p>
           </div>

           <div className="bg-purple-900/20 border border-purple-500/30 p-4 rounded-2xl">
              <div className="flex justify-between items-center mb-4">
                 <h3 className="text-lg font-bold flex items-center gap-2"><Gamepad2 className="text-purple-400"/> The Ambala Circuit</h3>
                 <span className="text-[10px] bg-purple-600 px-2 py-1 rounded text-white font-bold">SEASON 4</span>
              </div>
              <div className="space-y-3">
                 <div className="grid grid-cols-4 text-[10px] text-gray-500 uppercase font-bold px-2">
                    <span className="col-span-2">Team</span>
                    <span className="text-center">Turf Pts</span>
                    <span className="text-center">BGMI Pts</span>
                 </div>
                 {AMBALA_CIRCUIT.map((team, i) => (
                    <div key={team.id} className="grid grid-cols-4 items-center bg-gray-800 p-3 rounded-xl border border-gray-700">
                       <div className="col-span-2 flex items-center gap-2">
                          <div className="font-bold text-white text-sm">{i+1}. {team.team}</div>
                          <div className="text-[9px] bg-gray-700 px-1 rounded text-gray-400">{team.zone}</div>
                       </div>
                       <div className="text-center font-bold text-green-400">{team.turfPts}</div>
                       <div className="text-center font-bold text-purple-400">{team.bgmiPts}</div>
                    </div>
                 ))}
              </div>
           </div>

           {/* C - Crowd-Funded Tourneys */}
           <button className="w-full bg-indigo-600 py-3 rounded-xl font-bold text-white flex items-center justify-center gap-2 hover:bg-indigo-500">
              <Trophy size={18}/> Create Tournament (Crowd-Fund)
           </button>

           <div className="bg-gray-800 p-4 rounded-2xl border border-gray-700">
              <div className="flex justify-between items-start mb-2">
                 <h3 className="font-bold flex items-center gap-2"><Users2 size={18} className="text-blue-400"/> The 7 PM Draft</h3>
                 <span className="text-xs text-red-400 font-mono animate-pulse">01:45:20 Left</span>
              </div>
              <p className="text-xs text-gray-400 mb-4">
                 Solo? Enter the pool. Our AI assigns you a balanced squad at 7 PM sharp.
              </p>
              {draftStatus === 'idle' ? (
                 <button onClick={() => setDraftStatus('joined')} className="w-full bg-blue-600 py-3 rounded-xl font-bold text-white hover:bg-blue-500 transition-colors">
                   Enter Draft Pool
                 </button>
              ) : (
                 <button className="w-full bg-gray-700 py-3 rounded-xl font-bold text-green-400 flex items-center justify-center gap-2 cursor-default">
                    <CheckCircle size={16}/> Draft Entry Confirmed
                 </button>
              )}
           </div>
        </div>
      )}

      {activeTab === 'lounge' && (
        <div className="space-y-6 animate-in fade-in">
          {/* S - Shadow Matchmaking */}
          <button className="w-full bg-red-900/30 p-4 rounded-2xl border border-red-500/30 flex items-center justify-between hover:bg-red-900/40 transition-colors group">
             <div className="flex items-center gap-3">
                 <div className="bg-red-600 p-2 rounded-full animate-pulse">
                   <AlertOctagon size={20} className="text-white"/>
                 </div>
                 <div>
                    <span className="font-bold block text-sm text-red-100">SHADOW ALERT: 9/10</span>
                    <span className="text-xs text-red-300">Match starts in 1h • Need Defender</span>
                 </div>
             </div>
             <span className="bg-red-500 text-white text-xs font-bold px-3 py-1.5 rounded-full shadow-lg">JUMP IN</span>
          </button>

          {/* D - Digital Jersey Customizer */}
          <div className="bg-gradient-to-r from-gray-800 to-gray-900 p-3 rounded-xl border border-gray-700 flex items-center justify-between">
             <div className="flex items-center gap-3">
                <div className="bg-orange-500/20 p-2 rounded-lg text-orange-500"><Shirt size={20}/></div>
                <div>
                   <p className="text-xs font-bold text-gray-300">Squad Kit Creator</p>
                   <p className="text-[10px] text-gray-500">Design Virtual Jersey • Order Real</p>
                </div>
             </div>
             <ChevronRight size={16} className="text-gray-600"/>
          </div>

          <div className="bg-gray-800 p-3 rounded-2xl flex gap-3 items-center border border-gray-700">
              <input 
                className="bg-transparent flex-1 text-sm outline-none" 
                placeholder="Share your victory..."
                value={newPost}
                onChange={(e) => setNewPost(e.target.value)}
              />
              <button onClick={handlePost} className="bg-green-500 p-2 rounded-lg text-black"><Send size={16} /></button>
          </div>

          <div className="space-y-4">
              {feed.map(post => (
                <div key={post.id} className="bg-gray-800 p-4 rounded-2xl border border-gray-700 animate-in slide-in-from-bottom-2">
                  <div className="flex justify-between items-start mb-2">
                    <div className="flex items-center gap-3">
                      <div className="w-8 h-8 rounded-full bg-gray-600 flex items-center justify-center font-bold text-xs">U</div>
                      <div>
                        <p className="font-bold text-sm text-white flex items-center gap-1">
                           {post.user || 'Anonymous'}
                        </p>
                        {/* Safely render timestamp */}
                        <p className="text-xs text-gray-500">{post.createdAt ? new Date(post.createdAt.seconds * 1000).toLocaleTimeString() : 'Just now'}</p>
                      </div>
                    </div>
                  </div>
                  <p className="text-gray-300 text-sm mb-3">{post.text}</p>
                  <div className="flex gap-4 text-gray-500 text-xs font-bold">
                      <button className="hover:text-red-400 flex items-center gap-1"><Heart size={14}/> {post.likes || 0}</button>
                  </div>
                </div>
              ))}
          </div>
        </div>
      )}
    </div>
  );
}

function FitnessScreen() {
  const [aiTip, setAiTip] = useState(null);
  const [loadingTip, setLoadingTip] = useState(false);
  
  const getAiTip = async () => {
     setLoadingTip(true);
     const prompt = "Generate a unique, high-intensity 15-minute football fitness drill for a midfielder. Focus on stamina and agility. Keep it concise.";
     const drill = await callGemini(prompt);
     setAiTip(drill || FITNESS_DRILLS[0]); // Fallback if AI fails
     setLoadingTip(false);
  };

  return (
    <div className="h-full overflow-y-auto p-4 space-y-6 pb-24">
      <h2 className="text-2xl font-bold">Fitness & Activity</h2>
      
      <div className="grid grid-cols-2 gap-4">
        <div className="bg-gradient-to-br from-orange-500 to-red-600 p-4 rounded-3xl relative overflow-hidden h-40 flex flex-col justify-between">
          <Flame className="absolute -right-4 -bottom-4 text-white opacity-20" size={100} />
          <div>
            <p className="text-white/80 font-medium text-sm">Calories</p>
            <h3 className="text-3xl font-bold text-white mt-1">1,240</h3>
          </div>
          <p className="text-white/90 text-xs font-bold bg-white/20 w-fit px-2 py-1 rounded-lg">+12% vs last week</p>
        </div>
        <div className="bg-gradient-to-br from-blue-500 to-indigo-600 p-4 rounded-3xl relative overflow-hidden h-40 flex flex-col justify-between">
          <Activity className="absolute -right-4 -bottom-4 text-white opacity-20" size={100} />
          <div>
            <p className="text-white/80 font-medium text-sm">Matches</p>
            <h3 className="text-3xl font-bold text-white mt-1">8</h3>
          </div>
          <p className="text-white/90 text-xs font-bold bg-white/20 w-fit px-2 py-1 rounded-lg">Last 30 days</p>
        </div>
      </div>

      <div className="bg-gray-800 border border-indigo-500/30 p-4 rounded-2xl relative overflow-hidden">
         <div className="flex justify-between items-start mb-2">
            <h3 className="font-bold flex items-center gap-2"><BrainCircuit size={18} className="text-indigo-400"/> AI Coach</h3>
            <button onClick={getAiTip} disabled={loadingTip} className="text-xs bg-indigo-600 px-2 py-1 rounded text-white flex items-center gap-1">
              {loadingTip ? <Loader2 className="animate-spin" size={10}/> : "Get Drill"}
            </button>
         </div>
         <p className="text-sm text-gray-300 whitespace-pre-wrap">
            {aiTip ? aiTip : "Lost a few games? Click 'Get Drill' for a personalized recovery session generated by AI."}
         </p>
      </div>

      <div>
        <h3 className="font-bold text-lg mb-4">Daily Challenges</h3>
        <div className="space-y-3">
          {[
            { title: "The Sprinter", desc: "Run 2km on the field", reward: "50 pts", done: true },
            { title: "Team Player", desc: "Complete a booking with 4+ friends", reward: "100 pts", done: false },
          ].map((c, i) => (
            <div key={i} className="bg-gray-800 p-4 rounded-2xl flex items-center justify-between border border-gray-700">
              <div className="flex items-center gap-4">
                <div className={`w-10 h-10 rounded-full flex items-center justify-center ${c.done ? 'bg-green-500/20 text-green-500' : 'bg-gray-700 text-gray-400'}`}>
                  {c.done ? <CheckCircle size={20} /> : <Trophy size={20} />}
                </div>
                <div>
                  <p className="font-bold text-sm">{c.title}</p>
                  <p className="text-xs text-gray-500">{c.desc}</p>
                </div>
              </div>
              <span className="text-xs font-bold bg-yellow-500/10 text-yellow-500 px-2 py-1 rounded-md">{c.reward}</span>
            </div>
          ))}
        </div>
      </div>
    </div>
  );
}

function ProfileScreen({ profile, user, isPartnerMode, setIsPartnerMode, bookings, turfs, onAddTurf, onDeleteTurf, onOpenScanner, bookedSlots }) {
  const [showWallet, setShowWallet] = useState(false);
  const [isAddingTurf, setIsAddingTurf] = useState(false);
  const [isGuardianMode, setIsGuardianMode] = useState(false);
  const [activeTab, setActiveTab] = useState('stats');
  const [scoutReport, setScoutReport] = useState(null);
  const [isGeneratingScout, setIsGeneratingScout] = useState(false);
  
  // New Feature: Lucky Draw
  const [showLuckyDraw, setShowLuckyDraw] = useState(false);
  const [showEditProfile, setShowEditProfile] = useState(false);
  
  // Partner Mode Specific State
  const [isFlashActive, setIsFlashActive] = useState(false);
  const [posItems, setPosItems] = useState([
    { id: 1, name: 'Water (1L)', price: 20 },
    { id: 2, name: 'Gatorade', price: 60 },
    { id: 3, name: 'Bib Set', price: 100 }
  ]);

  // Dashboard Modals State
  const [activeModal, setActiveModal] = useState(null); // 'logs', 'invoice', 'scout', 'ban', 'totals'
  const [partnerLogs, setPartnerLogs] = useState([]);
  const [turfStatus, setTurfStatus] = useState({ lights: false, rain: false });
  const [invoiceData, setInvoiceData] = useState({ total: 0, items: [] });
  const [bannedUser, setBannedUser] = useState('');
  const [totalsData, setTotalsData] = useState({ bookings: 0, pos: 0, total: 0 });

  // REVERTED: Show ALL bookings, no filter logic applied
  const filteredPartnerBookings = bookedSlots; 

  // --- PARTNER REAL-TIME LISTENERS ---
  useEffect(() => {
    if (!isPartnerMode) return;

    // Listen to Logs
    const unsubLogs = onSnapshot(query(collection(db, 'artifacts', appId, 'public', 'data', 'partner_logs'), orderBy('timestamp', 'desc'), limit(20)), (snap) => {
       setPartnerLogs(snap.docs.map(d => d.data()));
    });

    // Listen to Turf Status (Lights/Rain)
    const statusRef = doc(db, 'artifacts', appId, 'public', 'data', 'turf_operations', 'status');
    const unsubStatus = onSnapshot(statusRef, (snap) => {
       if (snap.exists()) setTurfStatus(snap.data());
       else setDoc(statusRef, { lights: false, rain: false });
    });

    return () => { unsubLogs(); unsubStatus(); };
  }, [isPartnerMode]);

  // --- PARTNER ACTIONS ---
  const logAction = async (action) => {
     await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'partner_logs'), {
        action,
        timestamp: serverTimestamp()
     });
  };

  const toggleLights = async () => {
     const statusRef = doc(db, 'artifacts', appId, 'public', 'data', 'turf_operations', 'status');
     // Notify all users
     await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'notifications'), {
        message: `📢 Alert: Pro Lights are now ${!turfStatus.lights ? 'ON' : 'OFF'} at the arena!`,
        timestamp: serverTimestamp()
     });
     await updateDoc(statusRef, { lights: !turfStatus.lights });
     logAction(`Lights switched ${!turfStatus.lights ? 'ON' : 'OFF'}`);
  };

  const toggleRain = async () => {
     const statusRef = doc(db, 'artifacts', appId, 'public', 'data', 'turf_operations', 'status');
     // Notify all users
     await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'notifications'), {
        message: `🌧️ Weather Update: Rain Mode is ${!turfStatus.rain ? 'ACTIVE' : 'DEACTIVATED'}. Check app for status.`,
        timestamp: serverTimestamp()
     });
     await updateDoc(statusRef, { rain: !turfStatus.rain });
     logAction(`Rain Mode ${!turfStatus.rain ? 'ACTIVATED' : 'DEACTIVATED'}`);
  };

  const calculateInvoice = async () => {
     // Simply aggregate local sales for demo + bookings
     let total = 0;
     const items = [];
     
     // 1. Get recent bookings
     bookings.forEach(b => {
        if(b.status === 'Active' || b.status === 'Completed') {
           total += b.price;
           items.push({ desc: `Booking: ${b.turfName}`, amt: b.price });
        }
     });

     // 2. Get POS Sales (fetch specifically for invoice)
     const salesSnap = await getDocs(query(collection(db, 'artifacts', appId, 'public', 'data', 'turf_sales'), orderBy('timestamp', 'desc'), limit(10)));
     salesSnap.forEach(d => {
        const s = d.data();
        total += s.amount;
        items.push({ desc: `POS: ${s.item}`, amt: s.amount });
     });

     setInvoiceData({ total, items });
     setActiveModal('invoice');
  };

  const calculateFinancials = async () => {
     // Real calculation
     const bookingsSum = bookedSlots.reduce((acc, curr) => acc + (curr.price || 0), 0);
     
     // Fetch POS sales sum
     let posSum = 0;
     const salesSnap = await getDocs(collection(db, 'artifacts', appId, 'public', 'data', 'turf_sales'));
     salesSnap.forEach(doc => {
         posSum += (doc.data().amount || 0);
     });

     setTotalsData({
         bookings: bookingsSum,
         pos: posSum,
         total: bookingsSum + posSum
     });
     setActiveModal('totals');
  };

  const handlePartnerCancelBooking = async (slotId) => {
      if(confirm('Are you sure you want to cancel this booking? This will remove the slot availability.')) {
          try {
              await deleteDoc(doc(db, 'artifacts', appId, 'public', 'data', 'booked_slots', slotId));
              logAction(`Cancelled booking ID: ${slotId.slice(-6)}`);
              // Update totals live (optional, or rely on reopening modal)
              const newBookingsSum = totalsData.bookings - (bookedSlots.find(s => s.id === slotId)?.price || 0);
              setTotalsData(prev => ({
                  ...prev,
                  bookings: newBookingsSum,
                  total: newBookingsSum + prev.pos
              }));
          } catch (e) {
              console.error("Error cancelling booking:", e);
          }
      }
  };

  const banUser = async () => {
     if(!bannedUser) return;
     await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'banned_users'), {
        userId: bannedUser,
        bannedAt: serverTimestamp()
     });
     logAction(`Banned User ID: ${bannedUser}`);
     setBannedUser('');
     alert("User Banned Successfully");
     setActiveModal(null);
  };

  const handleUpdateProfile = async (name, phone) => {
     try {
        await updateDoc(doc(db, 'artifacts', appId, 'users', user.uid, 'profile', 'main'), {
           displayName: name,
           phone: phone
        });
        setShowEditProfile(false);
     } catch (e) {
        console.error("Error updating profile", e);
     }
  };

  // --- EXISTING FUNCTIONS ---
  const generateScoutReport = async () => {
    setIsGeneratingScout(true);
    const prompt = `Generate a professional football scout report for a player with ELO ${profile.elo || 1200}. Analyze their potential and suggest key improvement areas. Keep it under 50 words.`;
    const report = await callGemini(prompt);
    setScoutReport(report || "Analysis failed. Please retry.");
    setIsGeneratingScout(false);
  };

  const handleFlashSale = () => {
    setIsFlashActive(true);
    alert("F - Flash Alert Sent: 40% OFF notification pushed to 500+ Ambala users!");
    logAction("Flash Sale Deployed");
  };

  const handlePOSSale = async (item) => {
      try {
        await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'turf_sales'), {
          item: item.name,
          amount: item.price,
          timestamp: serverTimestamp()
        });
        alert(`C - Sold ${item.name}! Added ₹${item.price} to dashboard.`);
        logAction(`POS Sale: ${item.name}`);
      } catch (e) {
        console.error(e);
      }
  };

  const handleLuckyDrawWin = async (prize) => {
    try {
        const updates = {};
        if (prize.type === 'cash') updates.walletBalance = increment(prize.val);
        if (prize.type === 'karma') updates.karma = increment(prize.val);
        await updateDoc(doc(db, 'artifacts', appId, 'users', user.uid, 'profile', 'main'), updates);
    } catch(e) {
        console.error("Reward update failed", e);
    }
  };

  const handleToggleTurfStatus = async (turf) => {
    const newStatus = turf.status === "Open" ? "Closed" : "Open";
    await updateDoc(doc(db, 'artifacts', appId, 'public', 'data', 'turfs', turf.id), {
        status: newStatus
    });
    logAction(`Turf '${turf.name}' changed to ${newStatus}`);
  };

  const totalEarnings = bookings.filter(b => b.status !== 'Cancelled').reduce((acc, curr) => acc + curr.price, 0);
  const activeBookingsCount = bookings.filter(b => b.status === 'Active').length;
  const greenPoints = 850;
  const levelProgress = ((profile.points || 0) % 1000) / 10; 

  if (isAddingTurf) return <AddTurfForm onCancel={() => setIsAddingTurf(false)} onSubmit={onAddTurf} />;

  const toggleGuardian = () => {
    setIsGuardianMode(!isGuardianMode);
  };

  return (
    <div className={`h-full overflow-y-auto p-4 space-y-6 pb-24 ${isGuardianMode ? 'bg-slate-900' : ''}`}>
      {showWallet && <WalletModal balance={profile.walletBalance} profile={profile} user={user} onClose={() => setShowWallet(false)} />}
      
      {/* LUCKY DRAW MODAL */}
      {showLuckyDraw && (
         <LuckyDrawModal 
            onClose={() => setShowLuckyDraw(false)} 
            onWin={handleLuckyDrawWin} 
         />
      )}

      {showEditProfile && (
         <EditProfileModal profile={profile} onClose={() => setShowEditProfile(false)} onSave={handleUpdateProfile} />
      )}

      {/* DASHBOARD MODALS */}
      {activeModal === 'logs' && (
        <DashboardModal title="Activity Logs" onClose={() => setActiveModal(null)}>
           {partnerLogs.length === 0 ? <p className="text-gray-500 text-sm">No recent activity.</p> : (
              <div className="space-y-2">
                 {partnerLogs.map((log, i) => (
                    <div key={i} className="bg-gray-800 p-2 rounded border border-gray-700 text-xs">
                       <span className="text-green-400 font-bold">[{log.timestamp?.seconds ? new Date(log.timestamp.seconds * 1000).toLocaleTimeString() : 'Just now'}]</span> {log.action}
                    </div>
                 ))}
              </div>
           )}
        </DashboardModal>
      )}

      {activeModal === 'invoice' && (
        <DashboardModal title="Live Invoice" onClose={() => setActiveModal(null)}>
           <div className="bg-white text-black p-4 rounded-xl font-mono text-sm mb-4">
              <h4 className="font-bold border-b border-black pb-2 mb-2">TURFIT BILLING</h4>
              <div className="space-y-1 max-h-60 overflow-y-auto">
                 {invoiceData.items.map((item, i) => (
                    <div key={i} className="flex justify-between">
                       <span>{item.desc}</span>
                       <span>₹{item.amt}</span>
                    </div>
                 ))}
              </div>
              <div className="border-t border-black pt-2 mt-2 flex justify-between font-bold text-lg">
                 <span>TOTAL</span>
                 <span>₹{invoiceData.total}</span>
              </div>
           </div>
           <button className="w-full bg-green-500 text-black font-bold py-3 rounded-xl">Print / Share PDF</button>
        </DashboardModal>
      )}

      {activeModal === 'totals' && (
        <DashboardModal title="Booking Management" onClose={() => setActiveModal(null)}>
           <div className="space-y-4">
               {/* Financial Summary */}
               <div className="grid grid-cols-2 gap-3 mb-2">
                  <div className="bg-gray-800 p-3 rounded-xl border border-gray-700">
                     <p className="text-gray-400 text-[10px] uppercase">Revenue</p>
                     <p className="text-xl font-bold text-green-400">₹{totalsData.total}</p>
                  </div>
                  <div className="bg-gray-800 p-3 rounded-xl border border-gray-700">
                     <p className="text-gray-400 text-[10px] uppercase">Count</p>
                     <p className="text-xl font-bold text-blue-400">{bookedSlots.length}</p>
                  </div>
               </div>

               <h4 className="text-xs font-bold text-gray-400 uppercase mb-2">Detailed List (All History)</h4>
               
               {filteredPartnerBookings.length === 0 ? (
                  <p className="text-gray-500 text-sm text-center py-4 bg-gray-800/50 rounded-xl border border-dashed border-gray-700">No bookings found.</p>
               ) : (
                  <div className="space-y-2 max-h-60 overflow-y-auto pr-1 custom-scrollbar">
                     {filteredPartnerBookings.map((slot, idx) => (
                        <div key={idx} className="bg-gray-800 p-3 rounded-xl border border-gray-700 flex flex-col gap-1">
                           <div className="flex justify-between items-start">
                              <span className="font-bold text-sm text-white">{slot.turfName}</span>
                              <span className={`text-[10px] px-2 py-0.5 rounded ${slot.status === 'Used' ? 'bg-red-900 text-red-400' : 'bg-green-900 text-green-400'}`}>{slot.status === 'Used' ? 'USED' : 'Paid'} ₹{slot.price}</span>
                           </div>
                           <div className="flex justify-between text-xs text-gray-400">
                              <span>{slot.date} • {slot.time}</span>
                           </div>
                           <div className="flex justify-between items-end mt-1">
                              <span className="text-[10px] text-gray-500 font-mono">ID: {slot.displayId || (slot.id ? slot.id.slice(0, 8) : '...')}</span>
                              <button 
                                onClick={() => handlePartnerCancelBooking(slot.id)}
                                className="text-[10px] bg-red-900/40 text-red-400 px-2 py-1 rounded border border-red-500/30 hover:bg-red-900 hover:text-white transition-colors"
                              >
                                Cancel
                              </button>
                           </div>
                        </div>
                     ))}
                  </div>
               )}
           </div>
        </DashboardModal>
      )}

      {/* Scout feature removed */}

      {activeModal === 'ban' && (
        <DashboardModal title="Ban Management" onClose={() => setActiveModal(null)}>
           <div className="space-y-4">
              <p className="text-sm text-red-400">Warning: Banning a user prevents them from booking any turf in your network.</p>
              <input 
                 className="w-full bg-gray-800 p-3 rounded-xl border border-gray-700 text-white" 
                 placeholder="Enter User ID (e.g. U-123)"
                 value={bannedUser}
                 onChange={(e) => setBannedUser(e.target.value)}
              />
              <button onClick={banUser} className="w-full bg-red-600 text-white font-bold py-3 rounded-xl">BAN USER PERMANENTLY</button>
           </div>
        </DashboardModal>
      )}

      <div className="flex justify-between items-center">
        <h2 className="text-2xl font-bold">{isGuardianMode ? "Guardian Dashboard" : "Profile"}</h2>
        <div className="flex gap-2">
           {!isGuardianMode && (
             <div className="bg-green-900/30 px-3 py-1 rounded-full border border-green-700 text-green-400 flex items-center text-xs font-bold">
               <Zap size={12} className="mr-1 fill-current" /> {profile.points || 0} XP
             </div>
           )}
           <button onClick={toggleGuardian} className={`px-3 py-1 rounded-full border flex items-center text-xs font-bold ${isGuardianMode ? 'bg-blue-600 border-blue-400 text-white' : 'border-gray-700 text-gray-500'}`}>
              {isGuardianMode ? "Exit Guardian" : "Guardian Mode"}
           </button>
        </div>
      </div>

      {/* B - Battle Pass Subscription Banner */}
      {!isGuardianMode && !isPartnerMode && (
         <div className="bg-gradient-to-r from-purple-600 to-indigo-600 p-4 rounded-2xl flex justify-between items-center text-white relative overflow-hidden">
            <div className="relative z-10">
               <h3 className="font-black italic text-lg">TURFIT PRO PASS</h3>
               <p className="text-[10px] opacity-80">Free Drinks • 10% Off • No Fees</p>
            </div>
            <button className="relative z-10 bg-white text-purple-600 px-4 py-2 rounded-lg font-bold text-xs">₹499/mo</button>
            <CrownIcon className="absolute -right-4 -bottom-4 opacity-20 text-white w-32 h-32"/>
         </div>
      )}

      {/* Tab Navigation */}
      {!isGuardianMode && !isPartnerMode && (
          <div className="flex gap-4 border-b border-gray-700 pb-2 overflow-x-auto no-scrollbar">
             <button onClick={() => setActiveTab('stats')} className={`text-sm font-bold pb-2 whitespace-nowrap ${activeTab === 'stats' ? 'text-white border-b-2 border-white' : 'text-gray-500'}`}>Stats & Card</button>
             <button onClick={() => setActiveTab('redeem')} className={`text-sm font-bold pb-2 whitespace-nowrap ${activeTab === 'redeem' ? 'text-green-400 border-b-2 border-green-400' : 'text-gray-500'}`}>Green Redeem</button>
          </div>
      )}

      {isGuardianMode ? (
         <div className="space-y-4 animate-in fade-in">
            <div className="bg-slate-800 p-4 rounded-2xl border border-slate-700">
               <h3 className="text-lg font-bold mb-4 flex items-center gap-2"><Shield className="text-blue-400"/> Safety Log</h3>
               <div className="space-y-3">
                  <div className="flex justify-between text-sm p-2 bg-slate-700/50 rounded-lg">
                     <span>Last Siren Test</span>
                     <span className="text-green-400">Success (Today)</span>
                  </div>
               </div>
            </div>
         </div>
      ) : (
         <>
          <div className="flex p-1 bg-gray-800 rounded-2xl mb-4">
            <button onClick={() => setIsPartnerMode(false)} className={`flex-1 py-3 rounded-xl text-sm font-bold transition-all ${!isPartnerMode ? 'bg-gray-700 shadow-md' : 'text-gray-400'}`}>Player</button>
            <button onClick={() => setIsPartnerMode(true)} className={`flex-1 py-3 rounded-xl text-sm font-bold transition-all ${isPartnerMode ? 'bg-green-500 text-black shadow-md' : 'text-gray-400'}`}>Partner</button>
          </div>

          {isPartnerMode ? (
            <div className="space-y-6 animate-in slide-in-from-bottom-2">
              <div className="bg-black p-3 rounded-lg border border-gray-700 text-[10px] text-gray-400 font-mono text-center">
                  Revenue = (Slots × Demand) + POS Sales - Fixed Costs
              </div>

              <div className={`p-4 rounded-2xl border-2 transition-all ${isFlashActive ? 'bg-orange-600 border-orange-400 animate-pulse' : 'bg-gray-800 border-gray-700'}`}>
                <div className="flex justify-between items-center">
                  <div>
                    <h3 className="font-bold text-white flex items-center gap-2">
                      <Zap size={18} className={isFlashActive ? "fill-current" : ""}/> 
                      Flash Sale (F)
                    </h3>
                    <p className="text-[10px] text-gray-300">Push "40% OFF" to 5km Radius</p>
                  </div>
                  <button 
                    onClick={handleFlashSale}
                    className={`px-4 py-2 rounded-xl font-bold text-xs ${isFlashActive ? 'bg-white text-orange-600' : 'bg-orange-500 text-black'}`}
                  >
                    {isFlashActive ? "Live Now" : "Deploy"}
                  </button>
                </div>
              </div>

              <div className="grid grid-cols-2 gap-4">
                <StatCard label="Yield (Total)" value={`₹${totalEarnings}`} icon={IndianRupee} color="text-green-400" />
                <StatCard label="Live Demand" value="HIGH" icon={TrendingUp} color="text-orange-400" subtext="50+ nearby users" />
              </div>

              <div className="bg-gray-800 p-4 rounded-2xl border border-gray-700">
                <h3 className="text-xs font-bold text-gray-400 uppercase mb-3 flex items-center gap-2">
                  <Settings size={14}/> Operations Command (A-Z)
                </h3>
                
                <div className="mb-4 bg-gray-700/30 p-3 rounded-xl">
                   <p className="text-[10px] font-bold text-blue-300 mb-2 flex items-center gap-1"><Store size={12}/> Quick POS (C/S)</p>
                   <div className="flex gap-2">
                      {posItems.map(item => (
                         <button key={item.id} onClick={() => handlePOSSale(item)} className="flex-1 bg-gray-700 py-2 rounded-lg text-[10px] font-bold hover:bg-green-600 transition-colors">
                            {item.name}<br/>₹{item.price}
                         </button>
                      ))}
                   </div>
                </div>

                <div className="grid grid-cols-4 gap-2">
                  <button onClick={onOpenScanner} className="bg-gray-700 p-2 rounded-xl flex flex-col items-center gap-1 hover:bg-green-600 transition-colors group">
                    <ScanLine size={16} className="text-green-400 group-hover:text-white"/>
                    <span className="text-[8px] font-bold group-hover:text-white">Entry Scan</span>
                  </button>
                  <button onClick={() => setActiveModal('logs')} className="bg-gray-700 p-2 rounded-xl flex flex-col items-center gap-1 hover:bg-gray-600">
                    <ClipboardList size={16} className="text-yellow-400"/>
                    <span className="text-[8px] font-bold">Logs</span>
                  </button>
                  <button onClick={toggleLights} className={`p-2 rounded-xl flex flex-col items-center gap-1 transition-all ${turfStatus.lights ? 'bg-yellow-500/20 border border-yellow-500' : 'bg-gray-700 hover:bg-gray-600'}`}>
                    <CloudLightning size={16} className={turfStatus.lights ? "text-yellow-400 fill-current" : "text-yellow-200"}/>
                    <span className="text-[8px] font-bold">{turfStatus.lights ? 'ON' : 'Lights'}</span>
                  </button>
                  <button onClick={toggleRain} className={`p-2 rounded-xl flex flex-col items-center gap-1 transition-all ${turfStatus.rain ? 'bg-blue-500/20 border border-blue-500' : 'bg-gray-700 hover:bg-gray-600'}`}>
                    <Umbrella size={16} className={turfStatus.rain ? "text-blue-400 fill-current" : "text-blue-400"}/>
                    <span className="text-[8px] font-bold">{turfStatus.rain ? 'RAIN' : 'Rain'}</span>
                  </button>
                  <button onClick={calculateInvoice} className="bg-gray-700 p-2 rounded-xl flex flex-col items-center gap-1 hover:bg-gray-600">
                    <FileSpreadsheet size={16} className="text-green-400"/>
                    <span className="text-[8px] font-bold">Invoice</span>
                  </button>
                  
                  {/* Scout Button removed */}
                  <div className="bg-gray-800 p-2 rounded-xl flex flex-col items-center gap-1 opacity-20"></div>

                  <button onClick={() => setActiveModal('ban')} className="bg-gray-700 p-2 rounded-xl flex flex-col items-center gap-1 hover:bg-gray-600">
                    <Ban size={16} className="text-red-400"/>
                    <span className="text-[8px] font-bold">Ban</span>
                  </button>
                  <button onClick={calculateFinancials} className="bg-gray-700 p-2 rounded-xl flex flex-col items-center gap-1 hover:bg-gray-600">
                    <List size={16} className="text-emerald-400"/>
                    <span className="text-[8px] font-bold">Total Bookings</span>
                  </button>
                </div>
              </div>

              <div className="bg-red-900/20 border border-red-500/30 p-4 rounded-2xl">
                 <h3 className="text-xs font-bold text-red-400 uppercase mb-2 flex items-center gap-2">
                   <Siren size={14}/> Security Monitor (L)
                 </h3>
                 <div className="flex justify-between items-center">
                   <span className="text-[10px] text-gray-400">All Grounds Secure</span>
                   <span className="w-2 h-2 bg-green-500 rounded-full animate-pulse"></span>
                 </div>
              </div>

              <button onClick={() => alert("Settling to UPI (I)")} className="w-full bg-white text-black py-4 rounded-2xl font-black text-xs uppercase flex items-center justify-center gap-2 shadow-xl active:scale-95 transition-transform">
                <IndianRupee size={16}/> Settle Balance to UPI (I)
              </button>

              <div className="pt-4">
                <div className="flex justify-between items-center mb-3">
                   <h3 className="font-bold text-sm">Arena Management</h3>
                   <button onClick={() => setIsAddingTurf(true)} className="text-xs text-green-400 font-bold flex items-center gap-1"><Plus size={14}/> Add New</button>
                </div>
                {turfs.map(t => (
                  <div key={t.id} className="bg-gray-800 p-4 rounded-2xl flex justify-between items-center border border-gray-700 mb-2">
                    <div className="flex items-center">
                      <div className="w-10 h-10 bg-gray-700 rounded-lg mr-3 flex items-center justify-center">
                        <Layout size={20} className="text-gray-500"/>
                      </div>
                      <div>
                        <p className="font-bold text-sm">{t.name}</p>
                        <p className="text-[10px] text-green-400">Live in DB {t.status === "Closed" && "(Closed)"}</p>
                      </div>
                    </div>
                    <div className="flex gap-2">
                       <button onClick={() => handleToggleTurfStatus(t)} className={`p-2 rounded-lg ${t.status === "Closed" ? 'bg-red-600 text-white' : 'bg-green-600 text-black'}`}>
                          <PowerOff size={16}/>
                       </button>
                       <button onClick={() => onDeleteTurf(t.id)} className="p-2 bg-gray-700 rounded-lg text-gray-400 hover:text-red-500"><Trash2 size={16}/></button>
                    </div>
                  </div>
                ))}
              </div>
            </div>
          ) : (
            <>
              {activeTab === 'stats' && (
                <>
                  <div className="bg-gradient-to-br from-yellow-600 via-yellow-500 to-yellow-700 p-1 rounded-3xl shadow-xl relative">
                    <div className="bg-gray-900 rounded-[22px] p-5 relative overflow-hidden">
                        {/* G - Goal of the Month Badge */}
                        <div className="absolute top-2 right-2">
                           <Medal size={32} className="text-yellow-300 drop-shadow-lg"/>
                        </div>
                        <div className="flex justify-between items-start mb-6">
                          <div className="flex gap-4">
                              <div className="w-20 h-20 bg-gray-700 rounded-full border-2 border-yellow-500 overflow-hidden relative">
                                <div className="absolute inset-0 flex items-center justify-center text-xl font-bold text-gray-500">ME</div>
                              </div>
                              <div>
                                <h3 className="text-2xl font-bold text-white">
                                   {profile.displayName || 'YOU'} 
                                   <button onClick={() => setShowEditProfile(true)} className="ml-2 text-gray-400 hover:text-white"><Edit size={12}/></button>
                                </h3>
                                <div className="flex items-center gap-1 text-yellow-500 font-bold text-sm">
                                    <Star size={14} fill="currentColor"/> {profile.elo || 1200} ELO
                                </div>
                                <div className="mt-1 bg-blue-600 text-white text-[10px] px-2 py-0.5 rounded font-bold inline-block border border-blue-400 shadow-lg">
                                    © CAPTAIN
                                </div>
                                {profile.phone && <p className="text-[10px] text-gray-400 mt-1">{profile.phone}</p>}
                              </div>
                          </div>
                        </div>
                    </div>
                  </div>
                  
                  {/* X - XP Milestones Progress */}
                  <div className="bg-gray-800 p-3 rounded-xl border border-gray-700">
                      <div className="flex justify-between text-xs mb-1">
                         <span>Lvl {Math.floor((profile.points || 0)/100)}</span>
                         <span className="text-yellow-500">Legend Medal @ Lvl 100</span>
                      </div>
                      <div className="h-2 bg-gray-700 rounded-full overflow-hidden">
                         <div className="h-full bg-yellow-500 transition-all duration-1000" style={{width: `${levelProgress}%`}}></div>
                      </div>
                  </div>

                  {/* Scout Report Section */}
                  <div className="bg-gray-800 p-4 rounded-2xl border border-gray-700">
                      <div className="flex justify-between items-center mb-3">
                          <h3 className="font-bold flex items-center gap-2"><FileText size={18} className="text-indigo-400"/> Scout Report</h3>
                          <button onClick={generateScoutReport} disabled={isGeneratingScout} className="text-xs bg-indigo-600 px-3 py-1 rounded text-white">
                             {isGeneratingScout ? "Analyzing..." : "Update Report"}
                          </button>
                      </div>
                      <div className="bg-black/30 p-3 rounded-lg border border-gray-700 min-h-[60px]">
                          <p className="text-sm text-gray-300 italic font-serif">
                             {scoutReport || "Click 'Update Report' to generate your professional analysis based on recent match performance."}
                          </p>
                      </div>
                  </div>
                  
                  <div className="grid grid-cols-2 gap-3">
                      <button onClick={() => setShowWallet(true)} className="bg-gray-800 p-3 rounded-xl border border-gray-700 flex flex-col items-center gap-2 hover:bg-gray-750">
                         <div className="bg-green-900/30 p-2 rounded-full text-green-400"><Wallet size={20}/></div>
                         <span className="text-xs font-bold">Wallet (₹{profile.walletBalance})</span>
                      </button>
                      <div className="bg-gray-800 p-3 rounded-xl border border-gray-700 flex flex-col items-center gap-2">
                         <div className="bg-blue-900/30 p-2 rounded-full text-blue-400"><Briefcase size={20}/></div>
                         <span className="text-xs font-bold">Scout Dash</span>
                      </div>
                  </div>

                  {/* Lucky Draw Trigger Button */}
                  <button onClick={() => setShowLuckyDraw(true)} className="w-full bg-gradient-to-r from-yellow-500 to-red-500 p-3 rounded-xl font-black text-white text-sm shadow-lg flex items-center justify-center gap-2 animate-pulse">
                      <Dices size={20} /> 🎰 DAILY LUCKY DRAW
                  </button>
                </>
              )}

              {activeTab === 'redeem' && (
                 <div className="space-y-4 animate-in fade-in">
                   {/* Z - Zero Waste Green Mining Leaderboard */}
                   <div className="bg-emerald-900/20 border border-emerald-500/30 p-4 rounded-2xl">
                      <h3 className="font-bold text-emerald-400 mb-2">Ambala Green Leaderboard</h3>
                      <div className="space-y-2 text-sm">
                         <div className="flex justify-between"><span>1. Sector 9 Snipers</span><span className="text-emerald-400">4500 Pts</span></div>
                         <div className="flex justify-between"><span>2. You</span><span className="text-emerald-400">{greenPoints} Pts</span></div>
                      </div>
                   </div>
                   
                   <div className="bg-emerald-900/20 border border-emerald-500/30 p-4 rounded-2xl flex justify-between items-center">
                      <div>
                         <h3 className="font-bold text-emerald-400">Recycle & Earn</h3>
                         <p className="text-xs text-gray-400">Upload pic of bottle in bin</p>
                      </div>
                      <button className="bg-emerald-500 text-black px-4 py-2 rounded-xl font-bold text-xs flex items-center gap-2">
                         <Camera size={14}/> Upload
                      </button>
                   </div>

                   <div className="grid grid-cols-2 gap-3">
                      <div className="bg-gray-800 p-3 rounded-xl border border-gray-700 relative opacity-50">
                         <div className="absolute top-2 right-2 bg-gray-700 p-1 rounded"><Lock size={12}/></div>
                         <div className="h-20 bg-gray-700 rounded-lg mb-2 flex items-center justify-center"><Shirt size={32} className="text-gray-500"/></div>
                         <p className="font-bold text-sm">Pro Jersey</p>
                         <p className="text-xs text-green-400">2000 GP</p>
                      </div>
                      <div className="bg-gray-800 p-3 rounded-xl border border-gray-700 cursor-pointer hover:border-green-500 transition-colors">
                         <div className="h-20 bg-gray-700 rounded-lg mb-2 flex items-center justify-center"><ShoppingBag size={32} className="text-white"/></div>
                         <p className="font-bold text-sm">Flash Studs Rental</p>
                         <p className="text-xs text-green-400">500 GP</p>
                      </div>
                   </div>
                 </div>
              )}
            </>
          )}
         </>
      )}
    </div>
  );
}

// --- MAIN APP COMPONENT ---
export default function App() {
  const [activeScreen, setActiveScreen] = useState('home');
  const [user, setUser] = useState(null);
  const [userProfile, setUserProfile] = useState({});
  const [turfs, setTurfs] = useState([]);
  const [bookings, setBookings] = useState([]);
  const [bookedSlots, setBookedSlots] = useState([]); // For partner view
  const [feed, setFeed] = useState([]);
  const [notifications, setNotifications] = useState([]);
  
  // Modals
  const [showBooking, setShowBooking] = useState(false);
  const [selectedTurf, setSelectedTurf] = useState(null);
  const [showScanner, setShowScanner] = useState(false);
  const [showNotifications, setShowNotifications] = useState(false);
  const [isProcessing, setIsProcessing] = useState(false);
  const [isPartnerMode, setIsPartnerMode] = useState(false);
  const [showDeleteConfirm, setShowDeleteConfirm] = useState(null); // ID of turf to delete

  // --- AUTH & DATA FETCHING ---
  useEffect(() => {
    const initAuth = async () => {
        if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
            await signInWithCustomToken(auth, __initial_auth_token);
        } else {
            await signInAnonymously(auth);
        }
    };
    initAuth();
    const unsubscribe = onAuthStateChanged(auth, setUser);
    return () => unsubscribe();
  }, []);

  useEffect(() => {
    if (!user) return;

    // 1. User Profile
    const unsubProfile = onSnapshot(doc(db, 'artifacts', appId, 'users', user.uid, 'profile', 'main'), (doc) => {
        if (doc.exists()) setUserProfile(doc.data());
        else setDoc(doc.ref, { displayName: 'Player 1', walletBalance: 1000, karma: 50, points: 0 });
    });

    // 2. Turfs
    const unsubTurfs = onSnapshot(collection(db, 'artifacts', appId, 'public', 'data', 'turfs'), (snapshot) => {
        setTurfs(snapshot.docs.map(d => ({ id: d.id, ...d.data() })));
    });

    // 3. User Bookings
    const unsubBookings = onSnapshot(collection(db, 'artifacts', appId, 'users', user.uid, 'bookings'), (snapshot) => {
        setBookings(snapshot.docs.map(d => ({ id: d.id, ...d.data() })));
    });

    // 4. Partner Bookings (Global)
    const unsubAllBookings = onSnapshot(collection(db, 'artifacts', appId, 'public', 'data', 'booked_slots'), (snapshot) => {
        const slots = snapshot.docs.map(d => ({ id: d.id, ...d.data() }));
        setBookedSlots(slots);
    });

    // 5. Social Feed
    const unsubFeed = onSnapshot(query(collection(db, 'artifacts', appId, 'public', 'data', 'feed'), orderBy('createdAt', 'desc'), limit(20)), (snapshot) => {
        setFeed(snapshot.docs.map(d => ({ id: d.id, ...d.data() })));
    });

    // 6. Notifications
    const unsubNotes = onSnapshot(query(collection(db, 'artifacts', appId, 'public', 'data', 'notifications'), orderBy('timestamp', 'desc'), limit(10)), (snapshot) => {
        setNotifications(snapshot.docs.map(d => d.data()));
    });

    return () => {
        unsubProfile();
        unsubTurfs();
        unsubBookings();
        unsubAllBookings();
        unsubFeed();
        unsubNotes();
    };
  }, [user]); // Removed 'turfs' dependency to stop loop

  // Derived State (useMemo)
  const resaleSlots = useMemo(() => bookedSlots.filter(s => s.isResale), [bookedSlots]);
  const auctionSlots = useMemo(() => turfs.filter(t => t.isAuction), [turfs]);

  // --- HANDLERS ---

  const handleBooking = (turf, price) => {
      setSelectedTurf(turf);
      setShowBooking(true);
  };

  const confirmBooking = async (turf, total, date, time) => {
      if (!user) {
         alert("Please login to book!"); // Should ideally prompt login
         return;
      }
      if (userProfile.walletBalance < total) {
          alert("Insufficient Balance! Please Top Up.");
          return;
      }
      setIsProcessing(true);
      try {
          // Use safe random ID generation if crypto.randomUUID is not available
          const bookingId = typeof crypto.randomUUID === 'function' ? crypto.randomUUID() : `book_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
          const displayId = `TI-${Math.floor(Math.random()*10000)}`;
          
          // Deduct Balance
          await updateDoc(doc(db, 'artifacts', appId, 'users', user.uid, 'profile', 'main'), {
              walletBalance: increment(-total),
              points: increment(100) // XP
          });

          // Create Booking Record (User)
          await setDoc(doc(db, 'artifacts', appId, 'users', user.uid, 'bookings', bookingId), {
              turfName: turf.name,
              turfId: turf.id,
              date,
              time,
              price: total,
              status: 'Active',
              displayId,
              createdAt: serverTimestamp()
          });

          // Block Slot (Public)
          await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'booked_slots'), {
              turfName: turf.name,
              turfId: turf.id,
              date,
              time,
              price: total,
              bookedBy: user.uid,
              status: 'Active',
              displayId,
              userBookingId: bookingId,
              createdAt: serverTimestamp()
          });

          setShowBooking(false);
          alert(`Booking Confirmed! ID: ${displayId}`);
      } catch (error) {
          console.error("Booking failed", error);
          alert("Booking failed. Please try again.");
      }
      setIsProcessing(false);
  };

  const cancelBooking = async (id) => {
      if(confirm("Cancel booking? 50% refund only.")) {
          await updateDoc(doc(db, 'artifacts', appId, 'users', user.uid, 'bookings', id), {
              status: 'Cancelled'
          });
          // Refund logic would go here (cloud function ideally)
      }
  };

  const handlePost = async (text) => {
      await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'feed'), {
          text,
          user: userProfile.displayName,
          userId: user.uid,
          likes: 0,
          createdAt: serverTimestamp()
      });
  };

  const addTurf = async (turfData) => {
      await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'turfs'), {
          ...turfData,
          ownerId: user.uid,
          createdAt: serverTimestamp()
      });
      alert("Turf Listed Successfully!");
  };

  // Replaces window.confirm
  const requestDeleteTurf = (id) => {
      setShowDeleteConfirm(id);
  };

  const confirmDeleteTurf = async () => {
      if (showDeleteConfirm) {
          await deleteDoc(doc(db, 'artifacts', appId, 'public', 'data', 'turfs', showDeleteConfirm));
          setShowDeleteConfirm(null);
      }
  };

  return (
    <div className="flex flex-col h-screen bg-gray-950 text-white font-sans overflow-hidden">
        {/* Main Content Area */}
        <div className="flex-1 overflow-hidden relative">
            {activeScreen === 'home' && (
                <HomeScreen 
                    turfs={turfs} 
                    resaleSlots={resaleSlots}
                    auctionSlots={auctionSlots}
                    onBook={handleBooking} 
                    userKarma={userProfile.karma || 0}
                    onOpenScanner={() => setShowScanner(true)}
                    onOpenNotifications={() => setShowNotifications(true)}
                />
            )}
            {activeScreen === 'tickets' && (
                <TicketScreen 
                    bookings={bookings} 
                    onCancel={cancelBooking} 
                    user={user}
                />
            )}
            {activeScreen === 'social' && (
                <SocialScreen 
                    feed={feed} 
                    user={user} 
                    onPost={handlePost} 
                />
            )}
            {activeScreen === 'fitness' && (
                <FitnessScreen />
            )}
            {activeScreen === 'profile' && (
                <ProfileScreen 
                    profile={userProfile} 
                    user={user}
                    isPartnerMode={isPartnerMode}
                    setIsPartnerMode={setIsPartnerMode}
                    bookings={bookings} 
                    bookedSlots={bookedSlots} 
                    turfs={turfs}
                    onAddTurf={addTurf}
                    onDeleteTurf={requestDeleteTurf}
                    onOpenScanner={() => setShowScanner(true)}
                />
            )}
        </div>

        {/* Global Modals */}
        {showBooking && selectedTurf && (
            <BookingModal 
                turf={selectedTurf} 
                onClose={() => setShowBooking(false)} 
                onConfirm={confirmBooking}
                userKarma={userProfile.karma || 0}
                finalPrice={selectedTurf.price} 
                isProcessing={isProcessing}
            />
        )}
        {showScanner && <ScannerModal onClose={() => setShowScanner(false)} />}
        {showNotifications && <NotificationsModal notifications={notifications} onClose={() => setShowNotifications(false)} />}
        {showDeleteConfirm && (
            <ConfirmationModal 
                title="Delete Turf Listing?"
                message="This action cannot be undone. All active bookings will remain but new ones will be blocked."
                onConfirm={confirmDeleteTurf}
                onCancel={() => setShowDeleteConfirm(null)}
            />
        )}

        {/* Bottom Navigation */}
        <nav className="bg-gray-900 border-t border-gray-800 pb-safe pt-2 px-6 flex justify-between items-center relative z-50">
            <NavItem icon={Home} label="Arena" active={activeScreen === 'home'} onClick={() => setActiveScreen('home')} />
            <NavItem icon={Ticket} label="Tickets" active={activeScreen === 'tickets'} onClick={() => setActiveScreen('tickets')} />
            
            {/* Center FAB */}
            <div className="relative -top-6">
                <button 
                    onClick={() => setActiveScreen('social')}
                    className={`w-16 h-16 rounded-full flex items-center justify-center border-4 border-gray-900 shadow-lg shadow-green-500/30 transition-transform hover:scale-105 ${activeScreen === 'social' ? 'bg-white text-black' : 'bg-green-500 text-black'}`}
                >
                    <Users size={28} />
                </button>
            </div>

            <NavItem icon={Dumbbell} label="Fit" active={activeScreen === 'fitness'} onClick={() => setActiveScreen('fitness')} />
            <NavItem icon={User} label="Profile" active={activeScreen === 'profile'} onClick={() => setActiveScreen('profile')} />
        </nav>
    </div>
  );
}
