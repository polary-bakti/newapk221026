import React, { useState, useEffect, useRef } from 'react';
import { 
  Users, 
  Wallet, 
  Search, 
  Plus, 
  Trash2, 
  Edit2, 
  LayoutDashboard,
  Menu,
  X,
  Bell,
  LogOut,
  Sparkles,
  Send,
  Loader2,
  FileSpreadsheet,
  Info,
  ChevronDown,
  Eye,
  Settings2,
  User,
  Home,
  PhoneCall,
  MapPin
} from 'lucide-react';

const loadXlsxScript = () => {
  return new Promise((resolve) => {
    if (window.XLSX) {
      resolve();
      return;
    }
    const script = document.createElement('script');
    script.src = 'https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js';
    script.onload = () => resolve();
    document.head.appendChild(script);
  });
};

const App = () => {
  const apiKey = ""; 

  // --- STATE UTAMA ---
  const [isLoggedIn, setIsLoggedIn] = useState(false);
  const [loginData, setLoginData] = useState({ username: '', password: '' });
  const [loginError, setLoginError] = useState('');
  const [activeTab, setActiveTab] = useState('dashboard');
  const [isSidebarOpen, setSidebarOpen] = useState(false);
  const [searchTerm, setSearchTerm] = useState('');
  
  // Fitur Kustomisasi Kolom
  const [showColumnPanel, setShowColumnPanel] = useState(false);
  const columnPanelRef = useRef(null);

  const [visibleColumns, setVisibleColumns] = useState({
    nama: true,
    nik: true,
    noKK: false,
    gender: true,
    usia: true,
    pekerjaan: true,
    alamat: true,
    rt: false,
    rw: false,
    kelurahan: false,
    kecamatan: false,
    agama: false,
    phone: true,
    statusHubungan: false
  });

  const columnLabels = {
    nama: "Nama Lengkap",
    nik: "NIK",
    noKK: "No. KK",
    gender: "L/P",
    usia: "Usia",
    pekerjaan: "Pekerjaan",
    alamat: "Alamat",
    rt: "RT",
    rw: "RW",
    kelurahan: "Kelurahan",
    kecamatan: "Kecamatan",
    agama: "Agama",
    phone: "No. HP",
    statusHubungan: "Status Hubungan"
  };

  // Data Warga
  const [warga, setWarga] = useState([
    { 
      id: 1, 
      nama: 'Budi Santoso', 
      noKK: '3201010101010001', 
      nik: '3201010101010001', 
      gender: 'L', 
      tempatLahir: 'Jakarta', 
      tglLahir: '1985-05-12', 
      agama: 'Islam', 
      pendidikan: 'S1', 
      pekerjaan: 'Wiraswasta', 
      statusKawin: 'Kawin', 
      statusHubungan: 'Kepala Keluarga', 
      kewarganegaraan: 'WNI', 
      orangTua: 'Suryo', 
      phone: '0812-3456-7890', 
      usia: 39, 
      alamat: 'Blok A1 No. 5', 
      rt: '001', 
      rw: '010', 
      kelurahan: 'Mekarsari', 
      kecamatan: 'Tambun', 
      kabupaten: 'Bekasi',
      status: 'Tetap', 
      iuran: 'Lunas' 
    }
  ]);

  const [transaksi] = useState([
    { id: 1, tgl: '2023-11-01', ket: 'Iuran Keamanan Nov', tipe: 'masuk', jml: 150000 },
    { id: 2, tgl: '2023-11-15', ket: 'Fogging Lingkungan', tipe: 'keluar', jml: 450000 },
  ]);

  const saldoKas = transaksi.reduce((acc, curr) => curr.tipe === 'masuk' ? acc + curr.jml : acc - curr.jml, 2500000);

  // --- MODAL & FORM STATE ---
  const [showModal, setShowModal] = useState(false);
  const [isEditMode, setIsEditMode] = useState(false);
  const [currentWargaId, setCurrentWargaId] = useState(null);
  const [formWarga, setFormWarga] = useState({
    nama: '', noKK: '', nik: '', gender: 'L', tempatLahir: '', tglLahir: '', 
    agama: 'Islam', pendidikan: '', pekerjaan: '', statusKawin: 'Kawin', 
    statusHubungan: 'Anggota Keluarga', kewarganegaraan: 'WNI', orangTua: '', 
    phone: '', usia: '', alamat: '', rt: '', rw: '', kelurahan: '', 
    kecamatan: '', kabupaten: '', status: 'Tetap', iuran: 'Belum'
  });
  
  const fileInputRef = useRef(null);

  useEffect(() => {
    loadXlsxScript();

    const handleClickOutside = (event) => {
      if (columnPanelRef.current && !columnPanelRef.current.contains(event.target)) {
        setShowColumnPanel(false);
      }
    };
    document.addEventListener("mousedown", handleClickOutside);
    return () => document.removeEventListener("mousedown", handleClickOutside);
  }, []);

  const handleLogin = (e) => {
    e.preventDefault();
    if (loginData.username === 'admin' && loginData.password === 'admin123') {
      setIsLoggedIn(true);
      setLoginError('');
      setActiveTab('dashboard');
    } else {
      setLoginError('Username atau password salah!');
    }
  };

  const handleLogout = () => {
    const confirmLogout = window.confirm('Keluar dari sistem SmartRT?');
    if (confirmLogout) {
      setIsLoggedIn(false);
      setLoginData({ username: '', password: '' });
      setActiveTab('dashboard');
    }
  };

  const toggleColumn = (key) => {
    setVisibleColumns(prev => ({ ...prev, [key]: !prev[key] }));
  };

  const handleImportExcel = (e) => {
    const file = e.target.files[0];
    if (!file) return;

    const reader = new FileReader();
    reader.onload = (evt) => {
      const bstr = evt.target.result;
      const wb = window.XLSX.read(bstr, { type: 'binary' });
      const wsname = wb.SheetNames[0];
      const ws = wb.Sheets[wsname];
      const data = window.XLSX.utils.sheet_to_json(ws);

      const importedWarga = data.map((item, index) => ({
        id: Date.now() + index,
        nama: item['NAMA LENGKAP'] || item.nama || '',
        noKK: String(item['NO. KK'] || ''),
        nik: String(item['NIK'] || ''),
        gender: item['L/P'] || '',
        tempatLahir: item['TEMPAT LAHIR'] || '',
        tglLahir: item['TANGGAL LAHIR'] || '',
        agama: item['AGAMA'] || '',
        pendidikan: item['PENDIDIKAN TERAKHIR'] || '',
        pekerjaan: item['PEKERJAAN'] || '',
        statusKawin: item['STATUS KAWIN'] || '',
        statusHubungan: item['STATUS HUBUNGAN'] || '',
        kewarganegaraan: item['KEWARGANEGARAAN'] || 'WNI',
        orangTua: item['ORANG TUA'] || '',
        phone: String(item['NO.HP'] || ''),
        usia: item['USIA'] || '',
        alamat: item['ALAMAT'] || '',
        rt: item['RT'] || '',
        rw: item['RW'] || '',
        kelurahan: item['KELURAHAN'] || '',
        kecamatan: item['KECAMATAN'] || '',
        kabupaten: item['KABUPATEN'] || '',
        status: item['STATUS'] || 'Tetap',
        iuran: item['IURAN'] || 'Belum'
      }));

      if (importedWarga.length > 0) {
        setWarga(prev => [...prev, ...importedWarga]);
        alert(`Berhasil mengimpor ${importedWarga.length} data warga.`);
      }
      e.target.value = '';
    };
    reader.readAsBinaryString(file);
  };

  const handleOpenAddModal = () => {
    setIsEditMode(false);
    setFormWarga({
      nama: '', noKK: '', nik: '', gender: 'L', tempatLahir: '', tglLahir: '', 
      agama: 'Islam', pendidikan: '', pekerjaan: '', statusKawin: 'Kawin', 
      statusHubungan: 'Anggota Keluarga', kewarganegaraan: 'WNI', orangTua: '', 
      phone: '', usia: '', alamat: '', rt: '', rw: '', kelurahan: '', 
      kecamatan: '', kabupaten: '', status: 'Tetap', iuran: 'Belum'
    });
    setShowModal(true);
  };

  const handleOpenEditModal = (item) => {
    setIsEditMode(true);
    setCurrentWargaId(item.id);
    setFormWarga({ ...item });
    setShowModal(true);
  };

  const handleSubmitWarga = (e) => {
    e.preventDefault();
    if (isEditMode) {
      setWarga(warga.map(w => w.id === currentWargaId ? { ...formWarga, id: currentWargaId } : w));
    } else {
      setWarga([...warga, { ...formWarga, id: Date.now() }]);
    }
    setShowModal(false);
  };

  const handleDeleteWarga = (id) => {
    if (window.confirm('Hapus data warga ini?')) setWarga(warga.filter(w => w.id !== id));
  };

  if (!isLoggedIn) {
    return (
      <div className="min-h-screen bg-slate-50 flex items-center justify-center p-4 font-sans text-slate-900">
        <div className="w-full max-w-md bg-white rounded-[2.5rem] shadow-xl p-10 border border-slate-100">
          <div className="mb-8 text-center">
            <h2 className="text-3xl font-black text-blue-600 mb-2">SmartRT</h2>
            <p className="text-slate-400 text-xs font-black uppercase tracking-widest">Sistem Manajemen RT</p>
          </div>
          <form onSubmit={handleLogin} className="space-y-4">
            {loginError && <div className="p-4 bg-red-50 text-red-600 text-xs font-bold rounded-xl text-center">{loginError}</div>}
            <input type="text" placeholder="Username" className="w-full px-6 py-4 bg-slate-50 rounded-2xl outline-none" onChange={(e) => setLoginData({...loginData, username: e.target.value})} required />
            <input type="password" placeholder="Password" className="w-full px-6 py-4 bg-slate-50 rounded-2xl outline-none" onChange={(e) => setLoginData({...loginData, password: e.target.value})} required />
            <button type="submit" className="w-full bg-blue-600 text-white py-4 rounded-2xl font-black uppercase text-xs tracking-widest hover:bg-blue-700 transition-all">Masuk</button>
          </form>
        </div>
      </div>
    );
  }

  return (
    <div className="min-h-screen bg-[#F8FAFC] flex font-sans text-slate-900 overflow-x-hidden">
      <input type="file" ref={fileInputRef} onChange={handleImportExcel} accept=".xlsx, .xls, .csv" className="hidden" />

      {/* Sidebar */}
      <aside className={`fixed inset-y-0 left-0 z-[100] w-72 bg-white border-r border-slate-200 lg:translate-x-0 ${isSidebarOpen ? 'translate-x-0' : '-translate-x-full'} transition-transform duration-300 shadow-xl lg:shadow-none`}>
        <div className="p-8 font-black text-2xl text-blue-600">SmartRT</div>
        <nav className="px-6 space-y-2">
          {['dashboard', 'warga', 'ai-tools'].map((tab) => (
            <button
              key={tab}
              onClick={() => { setActiveTab(tab); setSidebarOpen(false); }}
              className={`w-full flex items-center space-x-4 px-4 py-3.5 rounded-2xl font-bold transition-all ${
                activeTab === tab ? 'bg-blue-600 text-white shadow-lg shadow-blue-100' : 'text-slate-500 hover:bg-slate-50'
              }`}
            >
              {tab === 'dashboard' && <LayoutDashboard size={18} />}
              {tab === 'warga' && <Users size={18} />}
              {tab === 'ai-tools' && <Sparkles size={18} />}
              <span className="capitalize">{tab === 'ai-tools' ? 'Asisten AI' : tab}</span>
            </button>
          ))}
          <div className="pt-10">
            <button onClick={handleLogout} className="w-full flex items-center space-x-4 px-4 py-3.5 text-red-500 font-bold hover:bg-red-50 rounded-2xl transition-all"><LogOut size={18} /> <span>Keluar App</span></button>
          </div>
        </nav>
      </aside>

      <main className="flex-1 lg:ml-72 p-6 lg:p-10 relative">
        <header className="flex justify-between items-center mb-10">
          <div>
            <h1 className="text-3xl font-black tracking-tight">Panel RT</h1>
            <p className="text-slate-500 font-medium italic">Status: Online • Administrator</p>
          </div>
          <button className="lg:hidden p-3 bg-white border rounded-xl" onClick={() => setSidebarOpen(true)}><Menu size={24} /></button>
        </header>

        {activeTab === 'dashboard' && (
          <div className="grid grid-cols-1 md:grid-cols-3 gap-6 animate-in fade-in duration-500">
            <div className="bg-white p-8 rounded-[2rem] border shadow-sm">
              <p className="text-slate-400 text-xs font-black uppercase tracking-widest">Total Warga</p>
              <h3 className="text-4xl font-black mt-2">{warga.length}</h3>
            </div>
            <div className="bg-white p-8 rounded-[2rem] border shadow-sm">
              <p className="text-slate-400 text-xs font-black uppercase tracking-widest">Saldo Kas</p>
              <h3 className="text-3xl font-black mt-2 text-emerald-600">Rp {saldoKas.toLocaleString()}</h3>
            </div>
          </div>
        )}

        {activeTab === 'warga' && (
          <div className="bg-white rounded-[2rem] border border-slate-100 shadow-sm animate-in fade-in duration-500 overflow-hidden">
            <div className="p-8 border-b border-slate-50 flex flex-col md:flex-row justify-between items-center gap-4 relative z-[50]">
              <div className="relative w-full md:w-64">
                <Search className="absolute left-4 top-1/2 -translate-y-1/2 text-slate-300" size={18} />
                <input type="text" placeholder="Cari..." className="w-full pl-12 pr-6 py-3 bg-slate-50 rounded-xl outline-none" onChange={(e) => setSearchTerm(e.target.value)} />
              </div>
              
              <div className="flex flex-wrap gap-2 w-full md:w-auto">
                <div className="relative" ref={columnPanelRef}>
                  <button 
                    onClick={() => setShowColumnPanel(!showColumnPanel)}
                    className={`flex items-center gap-2 p-3 rounded-xl font-bold uppercase text-[10px] border transition-all ${
                      showColumnPanel 
                        ? 'bg-blue-600 text-white border-blue-600 shadow-lg' 
                        : 'bg-slate-100 text-slate-600 border-slate-200 hover:bg-slate-200'
                    }`}
                  >
                    <Settings2 size={16}/> {showColumnPanel ? 'Tutup Pilihan' : 'Pilih Data'}
                  </button>
                  
                  {showColumnPanel && (
                    <div className="absolute top-full mt-3 right-0 w-72 bg-white border border-slate-100 shadow-2xl rounded-[2rem] z-[100] p-6 animate-in slide-in-from-top-4 duration-300">
                      <div className="text-[11px] font-black text-slate-400 uppercase mb-4 px-1 flex items-center justify-between">
                        <span className="flex items-center gap-2"><Eye size={12}/> Tampilan Kolom</span>
                        <button onClick={() => setShowColumnPanel(false)} className="text-slate-300 hover:text-slate-600"><X size={14}/></button>
                      </div>
                      <div className="space-y-1 max-h-[320px] overflow-y-auto pr-2 custom-scrollbar text-left">
                        {Object.keys(columnLabels).map(key => (
                          <label key={key} className="flex items-center gap-3 p-2.5 hover:bg-blue-50/50 rounded-xl cursor-pointer transition-colors group">
                            <input 
                              type="checkbox" 
                              checked={visibleColumns[key]} 
                              onChange={() => toggleColumn(key)}
                              className="w-5 h-5 rounded-lg text-blue-600 border-slate-300 focus:ring-blue-500 cursor-pointer"
                            />
                            <span className={`text-xs font-bold transition-all ${visibleColumns[key] ? 'text-blue-700' : 'text-slate-400'}`}>
                              {columnLabels[key]}
                            </span>
                          </label>
                        ))}
                      </div>
                      <div className="mt-4 pt-4 border-t border-slate-50">
                        <button 
                          onClick={() => setShowColumnPanel(false)}
                          className="w-full p-3 bg-slate-900 text-white rounded-xl text-[10px] font-black uppercase tracking-widest hover:bg-blue-600 transition-colors"
                        >
                          Selesai
                        </button>
                      </div>
                    </div>
                  )}
                </div>

                <button onClick={() => fileInputRef.current.click()} className="flex items-center gap-2 p-3 bg-emerald-50 text-emerald-600 rounded-xl font-bold uppercase text-[10px] border border-emerald-100 hover:bg-emerald-100 transition-all">
                  <FileSpreadsheet size={16}/> Impor Excel
                </button>
                <button onClick={handleOpenAddModal} className="bg-blue-600 text-white px-6 py-3 rounded-xl font-bold text-sm flex items-center gap-2 hover:bg-blue-700 shadow-lg shadow-blue-100 transition-all">
                  <Plus size={18}/> Tambah Warga
                </button>
              </div>
            </div>

            <div className="overflow-x-auto relative">
              <table className="w-full text-left min-w-max">
                <thead className="bg-slate-50/50 text-slate-400 text-[10px] font-black uppercase tracking-widest">
                  <tr>
                    {visibleColumns.nama && <th className="px-8 py-5 sticky left-0 bg-slate-50/50 z-30 border-b border-slate-100">Nama Lengkap</th>}
                    {visibleColumns.nik && <th className="px-4 py-5 border-b border-slate-100">NIK</th>}
                    {visibleColumns.noKK && <th className="px-4 py-5 border-b border-slate-100">No. KK</th>}
                    {visibleColumns.gender && <th className="px-4 py-5 text-center border-b border-slate-100">L/P</th>}
                    {visibleColumns.usia && <th className="px-4 py-5 text-center border-b border-slate-100">Usia</th>}
                    {visibleColumns.pekerjaan && <th className="px-4 py-5 border-b border-slate-100">Pekerjaan</th>}
                    {visibleColumns.alamat && <th className="px-4 py-5 border-b border-slate-100">Alamat Rumah</th>}
                    {visibleColumns.rt && <th className="px-4 py-5 text-center border-b border-slate-100">RT</th>}
                    {visibleColumns.rw && <th className="px-4 py-5 text-center border-b border-slate-100">RW</th>}
                    {visibleColumns.kelurahan && <th className="px-4 py-5 border-b border-slate-100">Kelurahan</th>}
                    {visibleColumns.kecamatan && <th className="px-4 py-5 border-b border-slate-100">Kecamatan</th>}
                    {visibleColumns.agama && <th className="px-4 py-5 border-b border-slate-100">Agama</th>}
                    {visibleColumns.phone && <th className="px-4 py-5 border-b border-slate-100">Telepon</th>}
                    {visibleColumns.statusHubungan && <th className="px-4 py-5 border-b border-slate-100">Hub. Keluarga</th>}
                    <th className="px-4 py-5 text-center bg-slate-50/50 sticky right-0 z-30 border-b border-slate-100 shadow-[-4px_0_10px_rgba(0,0,0,0.01)]">Aksi</th>
                  </tr>
                </thead>
                <tbody className="divide-y divide-slate-50">
                  {warga.filter(w => w.nama.toLowerCase().includes(searchTerm.toLowerCase()) || w.nik.includes(searchTerm)).map(item => (
                    <tr key={item.id} className="hover:bg-slate-50/30 text-sm transition-colors group">
                      {visibleColumns.nama && (
                        <td className="px-8 py-5 font-bold text-slate-800 sticky left-0 bg-white/95 backdrop-blur z-20 group-hover:bg-slate-50 transition-colors">
                          {item.nama}
                        </td>
                      )}
                      {visibleColumns.nik && <td className="px-4 py-5 text-slate-500 font-mono text-xs">{item.nik}</td>}
                      {visibleColumns.noKK && <td className="px-4 py-5 text-slate-500 font-mono text-xs">{item.noKK}</td>}
                      {visibleColumns.gender && <td className="px-4 py-5 font-bold text-center">{item.gender}</td>}
                      {visibleColumns.usia && <td className="px-4 py-5 text-center">{item.usia} Thn</td>}
                      {visibleColumns.pekerjaan && <td className="px-4 py-5">{item.pekerjaan}</td>}
                      {visibleColumns.alamat && <td className="px-4 py-5 truncate max-w-[200px] text-slate-600">{item.alamat}</td>}
                      {visibleColumns.rt && <td className="px-4 py-5 text-center font-bold text-blue-600">{item.rt}</td>}
                      {visibleColumns.rw && <td className="px-4 py-5 text-center font-bold text-blue-600">{item.rw}</td>}
                      {visibleColumns.kelurahan && <td className="px-4 py-5">{item.kelurahan}</td>}
                      {visibleColumns.kecamatan && <td className="px-4 py-5">{item.kecamatan}</td>}
                      {visibleColumns.agama && <td className="px-4 py-5">{item.agama}</td>}
                      {visibleColumns.phone && <td className="px-4 py-5 font-mono text-xs text-blue-600">{item.phone}</td>}
                      {visibleColumns.statusHubungan && <td className="px-4 py-5 italic text-slate-500">{item.statusHubungan}</td>}
                      
                      <td className="px-4 py-5 sticky right-0 bg-white/95 backdrop-blur z-20 shadow-[-4px_0_10px_rgba(0,0,0,0.01)] group-hover:bg-slate-50 transition-colors">
                        <div className="flex justify-center gap-1">
                          <button onClick={() => handleOpenEditModal(item)} className="p-2 text-blue-500 hover:bg-blue-100 rounded-lg transition-all"><Edit2 size={16}/></button>
                          <button onClick={() => handleDeleteWarga(item.id)} className="p-2 text-red-500 hover:bg-red-100 rounded-lg transition-all"><Trash2 size={16}/></button>
                        </div>
                      </td>
                    </tr>
                  ))}
                </tbody>
              </table>
            </div>
          </div>
        )}

        {/* Modal CRUD - DIKELOMPOKKAN DALAM FOLDER/SEKSI */}
        {showModal && (
          <div className="fixed inset-0 z-[200] bg-slate-900/60 backdrop-blur-sm flex items-center justify-center p-4 overflow-y-auto">
            <div className="bg-white rounded-[2.5rem] w-full max-w-4xl my-auto shadow-2xl animate-in zoom-in-95 duration-200 overflow-hidden">
              <div className="p-8 border-b border-slate-50 flex justify-between items-center sticky top-0 bg-white/90 backdrop-blur-md z-10">
                <div>
                  <h3 className="text-xl font-black uppercase tracking-tighter text-slate-800">{isEditMode ? 'Perbarui' : 'Daftarkan'} Data Warga</h3>
                  <p className="text-[10px] text-slate-400 font-bold uppercase tracking-widest mt-1">Lengkapi formulir di bawah ini</p>
                </div>
                <button onClick={() => setShowModal(false)} className="p-2 hover:bg-slate-100 rounded-xl transition-all text-slate-400"><X size={20}/></button>
              </div>
              
              <form onSubmit={handleSubmitWarga} className="p-8 space-y-10">
                
                {/* SEKSI 1: DATA PRIBADI */}
                <div className="space-y-4">
                  <div className="flex items-center gap-3 pb-2 border-b border-slate-100">
                    <div className="p-2 bg-blue-100 text-blue-600 rounded-xl"><User size={18}/></div>
                    <h4 className="text-sm font-black uppercase tracking-widest text-slate-700">Identitas Pribadi</h4>
                  </div>
                  <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
                    <div className="md:col-span-2">
                      <label className="text-[10px] font-black uppercase text-slate-400 ml-1 mb-1 block">Nama Lengkap</label>
                      <input required placeholder="Contoh: Budi Santoso" className="w-full p-4 bg-slate-50 rounded-2xl outline-none text-sm font-bold border-2 border-transparent focus:border-blue-100 transition-all" value={formWarga.nama} onChange={e => setFormWarga({...formWarga, nama: e.target.value})} />
                    </div>
                    <div>
                      <label className="text-[10px] font-black uppercase text-slate-400 ml-1 mb-1 block">Jenis Kelamin</label>
                      <select className="w-full p-4 bg-slate-50 rounded-2xl outline-none text-sm font-bold border-2 border-transparent focus:border-blue-100 cursor-pointer" value={formWarga.gender} onChange={e => setFormWarga({...formWarga, gender: e.target.value})}>
                        <option value="L">Laki-Laki</option>
                        <option value="P">Perempuan</option>
                      </select>
                    </div>
                    <div className="md:col-span-1">
                      <label className="text-[10px] font-black uppercase text-slate-400 ml-1 mb-1 block">NIK (KTP)</label>
                      <input required placeholder="16 Digit NIK" className="w-full p-4 bg-slate-50 rounded-2xl outline-none text-sm font-mono border-2 border-transparent focus:border-blue-100" value={formWarga.nik} onChange={e => setFormWarga({...formWarga, nik: e.target.value})} />
                    </div>
                    <div className="md:col-span-1">
                      <label className="text-[10px] font-black uppercase text-slate-400 ml-1 mb-1 block">No. Kartu Keluarga</label>
                      <input placeholder="16 Digit No. KK" className="w-full p-4 bg-slate-50 rounded-2xl outline-none text-sm font-mono border-2 border-transparent focus:border-blue-100" value={formWarga.noKK} onChange={e => setFormWarga({...formWarga, noKK: e.target.value})} />
                    </div>
                    <div>
                      <label className="text-[10px] font-black uppercase text-slate-400 ml-1 mb-1 block">Agama</label>
                      <input placeholder="Contoh: Islam" className="w-full p-4 bg-slate-50 rounded-2xl outline-none text-sm font-bold border-2 border-transparent focus:border-blue-100" value={formWarga.agama} onChange={e => setFormWarga({...formWarga, agama: e.target.value})} />
                    </div>
                    <div>
                      <label className="text-[10px] font-black uppercase text-slate-400 ml-1 mb-1 block">Pekerjaan</label>
                      <input placeholder="Contoh: Karyawan Swasta" className="w-full p-4 bg-slate-50 rounded-2xl outline-none text-sm font-bold border-2 border-transparent focus:border-blue-100" value={formWarga.pekerjaan} onChange={e => setFormWarga({...formWarga, pekerjaan: e.target.value})} />
                    </div>
                    <div>
                      <label className="text-[10px] font-black uppercase text-slate-400 ml-1 mb-1 block">Status Hubungan</label>
                      <select className="w-full p-4 bg-slate-50 rounded-2xl outline-none text-sm font-bold border-2 border-transparent focus:border-blue-100" value={formWarga.statusHubungan} onChange={e => setFormWarga({...formWarga, statusHubungan: e.target.value})}>
                        <option value="Kepala Keluarga">Kepala Keluarga</option>
                        <option value="Istri">Istri</option>
                        <option value="Anak">Anak</option>
                        <option value="Anggota Keluarga">Anggota Keluarga Lainnya</option>
                      </select>
                    </div>
                    <div>
                      <label className="text-[10px] font-black uppercase text-slate-400 ml-1 mb-1 block">Usia</label>
                      <input type="number" placeholder="Tahun" className="w-full p-4 bg-slate-50 rounded-2xl outline-none text-sm font-bold border-2 border-transparent focus:border-blue-100" value={formWarga.usia} onChange={e => setFormWarga({...formWarga, usia: e.target.value})} />
                    </div>
                  </div>
                </div>

                {/* SEKSI 2: DOMISILI & ALAMAT */}
                <div className="space-y-4">
                  <div className="flex items-center gap-3 pb-2 border-b border-slate-100">
                    <div className="p-2 bg-emerald-100 text-emerald-600 rounded-xl"><Home size={18}/></div>
                    <h4 className="text-sm font-black uppercase tracking-widest text-slate-700">Alamat & Domisili</h4>
                  </div>
                  <div className="grid grid-cols-1 md:grid-cols-4 gap-4">
                    <div className="md:col-span-3">
                      <label className="text-[10px] font-black uppercase text-slate-400 ml-1 mb-1 block">Alamat Rumah (Blok/No)</label>
                      <input placeholder="Contoh: Blok C No. 12" className="w-full p-4 bg-slate-50 rounded-2xl outline-none text-sm font-bold border-2 border-transparent focus:border-blue-100" value={formWarga.alamat} onChange={e => setFormWarga({...formWarga, alamat: e.target.value})} />
                    </div>
                    <div className="grid grid-cols-2 gap-2">
                      <div>
                        <label className="text-[10px] font-black uppercase text-slate-400 ml-1 mb-1 block">RT</label>
                        <input placeholder="000" className="w-full p-4 bg-slate-50 rounded-2xl outline-none text-sm font-bold text-center border-2 border-transparent focus:border-blue-100" value={formWarga.rt} onChange={e => setFormWarga({...formWarga, rt: e.target.value})} />
                      </div>
                      <div>
                        <label className="text-[10px] font-black uppercase text-slate-400 ml-1 mb-1 block">RW</label>
                        <input placeholder="000" className="w-full p-4 bg-slate-50 rounded-2xl outline-none text-sm font-bold text-center border-2 border-transparent focus:border-blue-100" value={formWarga.rw} onChange={e => setFormWarga({...formWarga, rw: e.target.value})} />
                      </div>
                    </div>
                    <div className="md:col-span-2">
                      <label className="text-[10px] font-black uppercase text-slate-400 ml-1 mb-1 block">Kelurahan</label>
                      <input placeholder="Kelurahan" className="w-full p-4 bg-slate-50 rounded-2xl outline-none text-sm font-bold border-2 border-transparent focus:border-blue-100" value={formWarga.kelurahan} onChange={e => setFormWarga({...formWarga, kelurahan: e.target.value})} />
                    </div>
                    <div className="md:col-span-2">
                      <label className="text-[10px] font-black uppercase text-slate-400 ml-1 mb-1 block">Kecamatan</label>
                      <input placeholder="Kecamatan" className="w-full p-4 bg-slate-50 rounded-2xl outline-none text-sm font-bold border-2 border-transparent focus:border-blue-100" value={formWarga.kecamatan} onChange={e => setFormWarga({...formWarga, kecamatan: e.target.value})} />
                    </div>
                  </div>
                </div>

                {/* SEKSI 3: KONTAK & STATUS */}
                <div className="space-y-4">
                  <div className="flex items-center gap-3 pb-2 border-b border-slate-100">
                    <div className="p-2 bg-amber-100 text-amber-600 rounded-xl"><PhoneCall size={18}/></div>
                    <h4 className="text-sm font-black uppercase tracking-widest text-slate-700">Kontak & Status</h4>
                  </div>
                  <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                    <div>
                      <label className="text-[10px] font-black uppercase text-slate-400 ml-1 mb-1 block">No. HP / WhatsApp</label>
                      <input placeholder="08xx-xxxx-xxxx" className="w-full p-4 bg-slate-50 rounded-2xl outline-none text-sm font-mono border-2 border-transparent focus:border-blue-100" value={formWarga.phone} onChange={e => setFormWarga({...formWarga, phone: e.target.value})} />
                    </div>
                    <div>
                      <label className="text-[10px] font-black uppercase text-slate-400 ml-1 mb-1 block">Status Tempat Tinggal</label>
                      <select className="w-full p-4 bg-slate-50 rounded-2xl outline-none text-sm font-bold border-2 border-transparent focus:border-blue-100 cursor-pointer" value={formWarga.status} onChange={e => setFormWarga({...formWarga, status: e.target.value})}>
                        <option value="Tetap">Warga Tetap</option>
                        <option value="Kontrak">Kontrak/Sewa</option>
                        <option value="Kost">Kost</option>
                      </select>
                    </div>
                  </div>
                </div>

                <div className="pt-4 sticky bottom-0 bg-white/90 backdrop-blur pb-2">
                  <button type="submit" className="w-full bg-blue-600 text-white py-5 rounded-[1.8rem] font-black uppercase text-xs tracking-[0.2em] shadow-xl shadow-blue-100 hover:bg-blue-700 transition-all active:scale-[0.98] flex items-center justify-center gap-3">
                    <Plus size={18}/> {isEditMode ? 'Simpan Perubahan' : 'Tambahkan Ke Data Warga'}
                  </button>
                </div>
              </form>
            </div>
          </div>
        )}
      </main>
    </div>
  );
};

export default App;
