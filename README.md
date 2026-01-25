<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sistem Manajemen Warga APW - Login</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@300;400;500;600;700;800&display=swap');
        
        :root {
            --primary: #6366f1;
            --secondary: #4f46e5;
            --dark: #0f172a;
        }

        body {
            background: var(--dark);
            font-family: 'Plus Jakarta Sans', sans-serif;
            color: #1e293b;
            overflow-x: hidden;
        }

        /* Animasi Background Bergerak */
        .animated-bg {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            z-index: -1;
            background: linear-gradient(45deg, #0f172a, #1e1b4b, #312e81);
            background-size: 400% 400%;
            animation: gradientBG 15s ease infinite;
        }

        @keyframes gradientBG {
            0% { background-position: 0% 50%; }
            50% { background-position: 100% 50%; }
            100% { background-position: 0% 50%; }
        }

        /* Floating Orbs */
        .orb {
            position: absolute;
            border-radius: 50%;
            filter: blur(80px);
            opacity: 0.5;
            z-index: -1;
            animation: float 10s ease-in-out infinite;
        }

        @keyframes float {
            0%, 100% { transform: translateY(0) scale(1); }
            50% { transform: translateY(-20px) scale(1.1); }
        }

        .login-container {
            min-height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
            padding: 2rem;
        }

        .login-card {
            background: rgba(255, 255, 255, 0.9);
            backdrop-filter: blur(20px);
            border: 1px solid rgba(255, 255, 255, 0.2);
            border-radius: 3rem;
            box-shadow: 0 40px 100px -20px rgba(0, 0, 0, 0.5);
            width: 100%;
            max-width: 1150px;
            display: flex;
            overflow: hidden;
            min-height: 750px;
            position: relative;
        }

        .login-left {
            background: linear-gradient(135deg, var(--primary) 0%, var(--secondary) 100%);
            width: 45%;
            padding: 4.5rem;
            display: none;
            flex-direction: column;
            justify-content: center;
            color: white;
            position: relative;
            overflow: hidden;
        }

        .digital-grid {
            position: absolute;
            inset: 0;
            background-image: linear-gradient(rgba(255,255,255,0.05) 1px, transparent 1px),
                              linear-gradient(90deg, rgba(255,255,255,0.05) 1px, transparent 1px);
            background-size: 40px 40px;
        }

        @media (min-width: 1024px) {
            .login-left { display: flex; }
        }

        .login-right {
            flex: 1;
            padding: 4.5rem;
            display: flex;
            flex-direction: column;
            justify-content: center;
            background: white;
        }

        /* Input Styling yang lebih 'Clean' */
        .form-control {
            width: 100%;
            padding: 1.15rem 1rem 1.15rem 3.5rem;
            background: #f8fafc;
            border: 2px solid #e2e8f0;
            border-radius: 1.25rem;
            font-size: 0.95rem;
            font-weight: 600;
            transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
        }

        .form-control:focus {
            outline: none;
            background: white;
            border-color: var(--primary);
            box-shadow: 0 10px 25px -5px rgba(99, 102, 241, 0.15);
            transform: translateY(-2px);
        }

        .login-btn {
            width: 100%;
            padding: 1.25rem;
            background: linear-gradient(to right, var(--primary), var(--secondary));
            color: white;
            border-radius: 1.25rem;
            font-weight: 800;
            font-size: 0.95rem;
            text-transform: uppercase;
            letter-spacing: 0.1em;
            box-shadow: 0 15px 30px -10px rgba(79, 70, 229, 0.5);
            transition: all 0.4s ease;
            margin-top: 1.5rem;
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 10px;
        }

        .login-btn:hover {
            transform: translateY(-3px) scale(1.02);
            box-shadow: 0 20px 40px -10px rgba(79, 70, 229, 0.6);
        }

        #loadingOverlay {
            position: fixed;
            inset: 0;
            background: rgba(15, 23, 42, 0.95);
            backdrop-filter: blur(12px);
            z-index: 9999;
            display: none;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            color: white;
        }

        .spinner {
            width: 60px;
            height: 60px;
            border: 5px solid rgba(255,255,255,0.1);
            border-top-color: var(--primary);
            border-radius: 50%;
            animation: spin 0.8s cubic-bezier(0.68, -0.55, 0.265, 1.55) infinite;
        }

        @keyframes spin { to { transform: rotate(360deg); } }

        /* Dashboard Table Styling */
        .table-header {
            background: #f1f5f9;
            color: #475569;
            font-weight: 800;
            font-size: 0.7rem;
            text-transform: uppercase;
            letter-spacing: 0.1em;
        }

        .sticky-col {
            position: sticky;
            left: 0;
            background: white;
            z-index: 10;
        }
    </style>
</head>
<body>

    <div class="animated-bg"></div>
    <div class="orb w-96 h-96 bg-indigo-500 top-[-10%] left-[-10%]"></div>
    <div class="orb w-[500px] h-[500px] bg-purple-600 bottom-[-20%] right-[-10%]" style="animation-delay: -2s;"></div>

    <div id="loadingOverlay">
        <div class="spinner mb-6"></div>
        <p class="font-black tracking-[0.3em] text-sm uppercase text-indigo-400">System Synchronizing...</p>
    </div>

    <!-- Login Section -->
    <div id="loginSection" class="login-container">
        <div class="login-card">
            <!-- Left Side -->
            <div class="login-left">
                <div class="digital-grid"></div>
                
                <div class="relative z-10">
                    <div class="mb-14">
                        <div class="w-24 h-24 bg-white/10 backdrop-blur-2xl border border-white/20 rounded-[2rem] flex items-center justify-center mb-10 shadow-2xl">
                            <i class="fas fa-layer-group text-5xl text-white"></i>
                        </div>
                        <h2 class="text-indigo-200 font-black uppercase tracking-[0.4em] text-xs mb-4">Core Architecture</h2>
                        <h1 class="text-6xl font-black tracking-tighter mb-6 leading-[0.9]">
                            Welcome<br><span class="text-indigo-300">Base Digital</span><br>APW
                        </h1>
                        <div class="h-2 w-32 bg-indigo-400 rounded-full"></div>
                    </div>
                    
                    <div class="space-y-8">
                        <div class="group flex items-center gap-6 p-4 rounded-2xl hover:bg-white/5 transition-all">
                            <div class="w-14 h-14 rounded-2xl bg-white/10 flex items-center justify-center shrink-0 border border-white/10 group-hover:scale-110 transition-transform">
                                <i class="fas fa-shield-check text-2xl"></i>
                            </div>
                            <div>
                                <h4 class="font-extrabold text-xl">Protected Access</h4>
                                <p class="text-sm text-indigo-100/60 font-medium">Lapis keamanan berlapis untuk setiap data.</p>
                            </div>
                        </div>
                        <div class="group flex items-center gap-6 p-4 rounded-2xl hover:bg-white/5 transition-all">
                            <div class="w-14 h-14 rounded-2xl bg-white/10 flex items-center justify-center shrink-0 border border-white/10 group-hover:scale-110 transition-transform">
                                <i class="fas fa-chart-network text-2xl"></i>
                            </div>
                            <div>
                                <h4 class="font-extrabold text-xl">Cloud Native</h4>
                                <p class="text-sm text-indigo-100/60 font-medium">Sinkronisasi real-time antar pengurus RT.</p>
                            </div>
                        </div>
                    </div>
                </div>

                <div class="mt-auto relative z-10 pt-10 flex items-center gap-4 text-white/30 text-[10px] font-black uppercase tracking-[0.2em]">
                    <div class="flex gap-1">
                        <div class="w-2 h-2 rounded-full bg-green-400"></div>
                        <span>Server Online</span>
                    </div>
                    <span class="opacity-20">|</span>
                    <span>APW Digital v4.0</span>
                </div>
            </div>

            <!-- Right Side -->
            <div class="login-right">
                <div class="mb-12">
                    <h2 class="text-5xl font-black text-slate-900 mb-4 tracking-tight">Login.</h2>
                    <p class="text-slate-500 font-semibold text-lg">Silahkan hubungkan identitas Anda ke sistem.</p>
                </div>

                <form id="loginForm" class="space-y-5">
                    <div>
                        <label class="text-[10px] font-black text-slate-400 uppercase tracking-widest ml-1 mb-2 block">Security Role</label>
                        <div class="relative">
                            <select id="roleSelector" class="form-control appearance-none">
                                <option value="warga">Warga Negara / Sipil</option>
                                <option value="admin">Administrator (Ketua RT)</option>
                                <option value="sekretaris">Authorized Secretary</option>
                            </select>
                            <span class="absolute left-5 top-1/2 -translate-y-1/2 text-slate-400"><i class="fas fa-id-badge text-xl"></i></span>
                            <span class="absolute right-5 top-1/2 -translate-y-1/2 text-slate-400 pointer-events-none"><i class="fas fa-angle-down"></i></span>
                        </div>
                    </div>

                    <div>
                        <label class="text-[10px] font-black text-slate-400 uppercase tracking-widest ml-1 mb-2 block">Identity Name</label>
                        <div class="relative">
                            <input type="text" id="username" class="form-control" required placeholder="NAMA LENGKAP SESUAI KTP">
                            <span class="absolute left-5 top-1/2 -translate-y-1/2 text-slate-400"><i class="fas fa-user-tag text-xl"></i></span>
                        </div>
                    </div>

                    <div>
                        <label class="text-[10px] font-black text-slate-400 uppercase tracking-widest ml-1 mb-2 block">Access Code</label>
                        <div class="relative">
                            <input type="password" id="password" class="form-control" required placeholder="••••••••">
                            <span class="absolute left-5 top-1/2 -translate-y-1/2 text-slate-400"><i class="fas fa-key-skeleton text-xl"></i></span>
                        </div>
                    </div>

                    <div class="flex items-center justify-between py-2">
                        <label class="flex items-center gap-3 cursor-pointer group">
                            <div class="w-5 h-5 rounded-md border-2 border-slate-200 group-hover:border-indigo-400 transition-colors flex items-center justify-center">
                                <input type="checkbox" class="hidden">
                                <div class="w-2.5 h-2.5 bg-indigo-500 rounded-sm opacity-0 transition-opacity"></div>
                            </div>
                            <span class="text-xs font-bold text-slate-500 uppercase tracking-wider">Stay Connected</span>
                        </label>
                        <a href="#" class="text-xs font-black text-indigo-600 uppercase tracking-widest hover:text-indigo-800">Support Center</a>
                    </div>

                    <button type="submit" class="login-btn">
                        INITIALIZE ACCESS <i class="fas fa-sign-in-alt"></i>
                    </button>
                </form>

                <div class="mt-16 pt-10 border-t border-slate-100 flex justify-between items-center">
                    <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/b/ba/Logo_Kabupaten_Bekasi.png/1200px-Logo_Kabupaten_Bekasi.png" class="h-8 grayscale opacity-50 hover:grayscale-0 hover:opacity-100 transition-all cursor-help" title="Mitra Pemerintah">
                    <p class="text-slate-300 text-[9px] font-black uppercase tracking-[0.2em]">Encryption AES-256 Active</p>
                </div>
            </div>
        </div>
    </div>

    <!-- Dashboard Section -->
    <div id="dashboardSection" class="hidden">
        <nav class="bg-white/80 backdrop-blur-md border-b border-slate-200 sticky top-0 z-40 shadow-sm">
            <div class="max-w-[1600px] mx-auto px-8 h-24 flex justify-between items-center">
                <div class="flex items-center gap-6">
                    <div class="w-14 h-14 bg-indigo-600 rounded-2xl flex items-center justify-center text-white shadow-lg shadow-indigo-200">
                        <i class="fas fa-database text-2xl"></i>
                    </div>
                    <div>
                        <h2 class="font-black text-2xl leading-none text-slate-900 tracking-tighter">Digital Base APW</h2>
                        <p id="currentDateDisplay" class="text-[10px] text-indigo-500 uppercase font-black tracking-widest mt-2"></p>
                    </div>
                </div>
                <div class="flex items-center gap-6">
                    <div class="text-right hidden sm:block border-r pr-8 border-slate-200">
                        <p id="userInfo" class="text-base font-black text-slate-900"></p>
                        <p id="userRoleBadge" class="text-[10px] text-indigo-600 font-black uppercase tracking-widest mt-1"></p>
                    </div>
                    <button onclick="window.logout()" class="w-14 h-14 flex items-center justify-center rounded-2xl bg-red-50 text-red-500 hover:bg-red-500 hover:text-white transition-all shadow-sm">
                        <i class="fas fa-power-off text-xl"></i>
                    </button>
                </div>
            </div>
        </nav>

        <main class="max-w-[1600px] mx-auto p-8">
            <div class="flex flex-col xl:flex-row justify-between items-start xl:items-center gap-8 mb-10">
                <div>
                    <h3 class="text-4xl font-black text-slate-900 tracking-tight">Katalog Data Warga</h3>
                    <p class="text-slate-400 font-medium text-lg">Manajemen informasi kependudukan sistem terpusat.</p>
                </div>
                <div id="adminActions" class="hidden flex flex-wrap gap-4">
                    <button class="bg-indigo-600 text-white px-8 py-4 rounded-2xl text-[11px] font-black uppercase tracking-widest flex items-center gap-3 shadow-xl hover:bg-indigo-700 transition-all">
                        <i class="fas fa-plus-circle"></i> Entri Data Baru
                    </button>
                </div>
            </div>

            <div class="grid grid-cols-1 lg:grid-cols-4 gap-8 mb-10">
                <div class="lg:col-span-3 bg-white p-2 rounded-3xl shadow-sm border border-slate-200 flex items-center">
                    <i class="fas fa-search ml-6 text-slate-400 text-xl"></i>
                    <input type="text" id="searchInput" class="w-full p-5 bg-transparent border-none focus:ring-0 font-bold text-slate-700" placeholder="Filter data berdasarkan NIK atau Nama Lengkap...">
                </div>
                <div class="bg-white p-6 rounded-3xl shadow-sm border border-slate-200 flex items-center justify-between">
                    <div>
                        <p class="text-[10px] font-black text-slate-400 uppercase tracking-widest mb-1">Total Population</p>
                        <p id="countBadge" class="text-4xl font-black text-indigo-600 tracking-tighter">0</p>
                    </div>
                    <div class="w-16 h-16 bg-indigo-50 rounded-2xl flex items-center justify-center text-indigo-600">
                        <i class="fas fa-users-crown text-2xl"></i>
                    </div>
                </div>
            </div>

            <div class="bg-white rounded-[2.5rem] shadow-xl shadow-slate-200/50 border border-slate-200 overflow-hidden">
                <div class="overflow-x-auto">
                    <table class="w-full text-left border-collapse">
                        <thead class="table-header">
                            <tr>
                                <th class="px-8 py-6 sticky-col text-center">No</th>
                                <th class="px-8 py-6">Informasi Warga</th>
                                <th class="px-8 py-6">NIK / KK</th>
                                <th class="px-8 py-6">Kependudukan</th>
                                <th class="px-8 py-6 text-center">Usia</th>
                                <th class="px-8 py-6">Kontak & Alamat</th>
                            </tr>
                        </thead>
                        <tbody id="wargaTableBody" class="divide-y divide-slate-100">
                        </tbody>
                    </table>
                </div>
            </div>
        </main>
    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, collection, onSnapshot } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        const firebaseConfig = JSON.parse(__firebase_config);
        const app = initializeApp(firebaseConfig);
        const auth = getAuth(app);
        const db = getFirestore(app);
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'apw-rt-db';

        let allWarga = [];
        let currentUserData = null;
        let currentRole = 'warga';
        let isLoggedIn = false;
        let unsubscribeWarga = null;

        const getWargaCol = () => collection(db, 'artifacts', appId, 'public', 'data', 'warga');

        const initAuth = async () => {
            if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                await signInWithCustomToken(auth, __initial_auth_token);
            } else {
                await signInAnonymously(auth);
            }
        };
        initAuth();

        onAuthStateChanged(auth, (user) => {
            if (user) {
                if (unsubscribeWarga) unsubscribeWarga();
                unsubscribeWarga = onSnapshot(getWargaCol(), (snap) => {
                    allWarga = snap.docs.map(d => ({ id: d.id, ...d.data() }));
                    if(isLoggedIn) renderTable();
                });
            }
        });

        document.getElementById('loginForm').addEventListener('submit', (e) => {
            e.preventDefault();
            const role = document.getElementById('roleSelector').value;
            const user = document.getElementById('username').value.trim().toUpperCase();
            const pass = document.getElementById('password').value.trim();

            document.getElementById('loadingOverlay').style.display = 'flex';

            setTimeout(() => {
                if (role === 'warga') {
                    const found = allWarga.find(w => w.NamaLengkap?.toUpperCase() === user);
                    if (found) {
                        currentUserData = found;
                        currentRole = 'warga';
                        isLoggedIn = true;
                        finishLogin();
                    } else {
                        alert("Access Denied: Nama tidak terdaftar di pangkalan data.");
                    }
                } else {
                    if ((user === 'ADMIN' && pass === 'admin123') || (user === 'SEKRE' && pass === 'sekre123')) {
                        currentUserData = { NamaLengkap: user === 'ADMIN' ? 'Ketua RT' : 'Sekretaris' };
                        currentRole = user === 'ADMIN' ? 'admin' : 'sekretaris';
                        isLoggedIn = true;
                        finishLogin();
                    } else {
                        alert("Authentication Failed: Kredensial tidak valid.");
                    }
                }
                document.getElementById('loadingOverlay').style.display = 'none';
            }, 1500);
        });

        function finishLogin() {
            document.getElementById('loginSection').classList.add('hidden');
            document.getElementById('dashboardSection').classList.remove('hidden');
            document.getElementById('userInfo').innerText = currentUserData.NamaLengkap;
            document.getElementById('userRoleBadge').innerText = currentRole;
            
            if (currentRole !== 'warga') {
                document.getElementById('adminActions').classList.remove('hidden');
            }

            const now = new Date();
            const options = { weekday: 'long', day: 'numeric', month: 'long', year: 'numeric' };
            document.getElementById('currentDateDisplay').innerText = now.toLocaleDateString('id-ID', options);
            
            renderTable();
        }

        window.logout = () => location.reload();

        function renderTable() {
            const tbody = document.getElementById('wargaTableBody');
            const search = document.getElementById('searchInput').value.toLowerCase();
            
            let filtered = allWarga;
            if (currentRole === 'warga') {
                filtered = allWarga.filter(w => w.NamaLengkap?.toUpperCase() === currentUserData.NamaLengkap.toUpperCase());
            }

            if (search) {
                filtered = filtered.filter(w => 
                    w.NamaLengkap?.toLowerCase().includes(search) || 
                    w.NIK?.includes(search)
                );
            }

            document.getElementById('countBadge').innerText = filtered.length;

            tbody.innerHTML = filtered.map((w, idx) => `
                <tr class="group hover:bg-slate-50 transition-all cursor-default">
                    <td class="px-8 py-6 sticky-col text-center font-black text-slate-300 group-hover:text-indigo-400">${idx + 1}</td>
                    <td class="px-8 py-6">
                        <div class="flex flex-col">
                            <span class="font-black text-slate-900 uppercase tracking-tight text-base">${w.NamaLengkap || '-'}</span>
                            <span class="text-[10px] font-black text-indigo-500 uppercase mt-1 tracking-widest">${w.StatusHubungan || '-'}</span>
                        </div>
                    </td>
                    <td class="px-8 py-6 font-bold text-slate-500">
                        <div class="text-xs mb-1">NIK: <span class="text-indigo-600">${w.NIK || '-'}</span></div>
                        <div class="text-[10px] opacity-60 uppercase">KK: ${w.KK || '-'}</div>
                    </td>
                    <td class="px-8 py-6">
                        <span class="px-4 py-1.5 rounded-xl text-[10px] font-black uppercase bg-indigo-50 text-indigo-700 border border-indigo-100">${w.JenisKelamin === 'L' ? 'Laki-laki' : 'Perempuan'}</span>
                    </td>
                    <td class="px-8 py-6 font-black text-slate-800 text-center text-lg">${w.Usia || '-'}</td>
                    <td class="px-8 py-6">
                        <div class="flex items-center gap-2 text-emerald-600 font-bold mb-1">
                            <i class="fab fa-whatsapp"></i> ${w.NoHP || '-'}
                        </div>
                        <div class="text-[10px] text-slate-400 font-medium uppercase truncate max-w-[200px]">${w.AlamatLengkap || '-'}</div>
                    </td>
                </tr>
            `).join('');
        }

        document.getElementById('searchInput').addEventListener('keyup', renderTable);
    </script>
</body>
</html>
