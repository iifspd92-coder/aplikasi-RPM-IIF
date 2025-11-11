# aplikasi-RPM-IIF
gunakan sebijak mungkin yaaah
import React, { useState, useEffect } from "react";

// Single-file React component for "Generator RPM IIF SADEWA GOA"
// Uses Tailwind CSS for styling. Put this component into a React app
// (e.g. create-react-app or Vite) with Tailwind configured.

export default function GeneratorRPM() {
  const ADMIN_USERNAME = "IIF SADEWA GOA";
  const ADMIN_PASSWORD = "29021996"; // stored in-file (required by user request)

  // Auth state
  const [loggedIn, setLoggedIn] = useState(false);
  const [isAdmin, setIsAdmin] = useState(false);
  const [tokenLogin, setTokenLogin] = useState("");
  const [showLogin, setShowLogin] = useState(true);

  // Form state
  const initialForm = {
    satuan: "",
    guru: "",
    nipGuru: "",
    kepala: "",
    nipKepala: "",
    jenjang: "SD",
    kelas: "",
    mapel: "",
    capaian: "",
    tujuan: "",
    materi: "",
    pertemuan: 1,
    durasi: "2 × 35 menit",
    praktikPertemuan: [], // array per pertemuan
    dimensi: []
  };
  const [form, setForm] = useState(initialForm);
  const [errors, setErrors] = useState({});
  const [outputHtml, setOutputHtml] = useState("");

  // Utility: load tokens (created by admin) from localStorage
  const TOKENS_KEY = "iif_sadewa_tokens_v1";
  const getTokens = () => JSON.parse(localStorage.getItem(TOKENS_KEY) || "[]");
  const saveTokens = (toks) => localStorage.setItem(TOKENS_KEY, JSON.stringify(toks));

  useEffect(() => {
    // if there is a token in localStorage that matches tokenLogin, log in
    if (tokenLogin) {
      const tokens = getTokens();
      if (tokens.includes(tokenLogin)) {
        setLoggedIn(true);
        setIsAdmin(false);
        setShowLogin(false);
      }
    }
  }, [tokenLogin]);

  function validate() {
    const e = {};
    const required = [
      ["satuan","Nama Satuan Pendidikan"],
      ["guru","Nama Guru"],
      ["nipGuru","NIP Guru"],
      ["kepala","Nama Kepala Sekolah"],
      ["nipKepala","NIP Kepala Sekolah"],
      ["kelas","Kelas/Semester"],
      ["mapel","Mata Pelajaran"],
      ["capaian","Capaian Pembelajaran"],
      ["tujuan","Tujuan Pembelajaran"],
      ["materi","Materi Pelajaran"]
    ];
    required.forEach(([k,label]) => { if(!form[k] || String(form[k]).trim()==='') e[k]=label+" wajib diisi"; });
    if (!form.pertemuan || form.pertemuan < 1) e.pertemuan = "Jumlah pertemuan harus >= 1";
    setErrors(e);
    return Object.keys(e).length===0;
  }

  function handleChange(e) {
    const { name, value, type, checked } = e.target;
    if (name === "praktikPertemuan") {
      // not used here
      return;
    }
    if (type === "checkbox") {
      setForm(prev => ({...prev, [name]: checked ? [...(prev[name]||[]), value] : prev[name].filter(x=>x!==value)}));
      return;
    }
    setForm(prev => ({...prev, [name]: value}));
  }

  function handleDimensiToggle(dim) {
    setForm(prev => {
      const set = new Set(prev.dimensi || []);
      if (set.has(dim)) set.delete(dim); else set.add(dim);
      return {...prev, dimensi: Array.from(set)};
    });
  }

  function buildGeneratedStudents() {
    // simple generation: 30 siswa + variasi usia
    const count = 30;
    const arr = [];
    for (let i=1;i<=count;i++) arr.push(`Siswa ${i}`);
    return arr.join(", ");
  }

  function generateLintasDisiplin(materi) {
    // naive mapping
    const res = [];
    if (/sains|IPA|fisika|kimia|biologi/i.test(materi)) res.push("Sains");
    if (/matematika|angka|aljabar|geometri/i.test(materi)) res.push("Matematika");
    if (/sejarah|peta|peradaban/i.test(materi)) res.push("Ilmu Sosial");
    if (/bahasa|membaca|menulis/i.test(materi)) res.push("Bahasa dan Literasi");
    if (res.length===0) res.push("Penguatan Literasi & Numerasi");
    return res.join(", ");
  }

  function generateKemitraan(materi) {
    return `Kerjasama dengan perpustakaan sekolah / sumber belajar daring (mis. Khan Academy, Zenius), dan ORANG TUA untuk penguatan tugas rumah.`;
  }
  function generateLingkungan(materi) {
    return `Ruang kelas, laboratorium sederhana, atau lingkungan sekitar (lapangan/komunitas) sesuai kebutuhan pembelajaran.`;
  }
  function generatePemanfaatanDigital(materi) {
    const tools = [];
    if (/present/i.test(materi)) tools.push("Google Slides");
    tools.push("Google Drive (penyimpanan)" , "Google Forms (asesmen)" , "Canva (materi visual)");
    return `${tools.join(", ")}. Gunakan sumber terbuka seperti YouTube Education, Khan Academy, dan dokumen referensi online.`;
  }

  function generateTopik(materi) {
    // take first sentence / phrase
    const parts = materi.split(/[\.\n\-–]/).filter(Boolean);
    return parts[0] ? parts[0].slice(0,120) : materi.slice(0,120);
  }

  function buildOutputHtml() {
    // Build an HTML table-like structure for the five sections
    const siswa = buildGeneratedStudents();
    const lintas = generateLintasDisiplin(form.materi || "");
    const kemitraan = generateKemitraan(form.materi || "");
    const lingkungan = generateLingkungan(form.materi || "");
    const digital = generatePemanfaatanDigital(form.materi || "");
    const topik = generateTopik(form.materi || "");

    // Praktik pedagogis: if not provided, generate an alternating list
    const praktik = [];
    const praktikOptions = ["Inkuiri-Discovery Learning","PjBL","Problem Based Learning","Game Based Learning","Station Learning"];
    for (let i=0;i<form.pertemuan;i++) praktik.push(form.praktikPertemuan[i] || praktikOptions[i % praktikOptions.length]);

    const dimensi = (form.dimensi && form.dimensi.length? form.dimensi.join(", ") : "-");

    const html = `
      <div style="font-family: 'Inter', sans-serif; max-width:1100px; margin:0 auto;">
        <h1 style="text-align:center;">Generator RPM IIF SADEWA GOA</h1>
        <table style="width:100%; border-collapse:collapse;">
          <tr><td style="vertical-align:top; width:50%; padding:8px; border:1px solid #ddd;">
            <h2>Identitas</h2>
            <table style="width:100%">
              <tr><td>Nama Satuan Pendidikan</td><td>${escapeHtml(form.satuan)}</td></tr>
              <tr><td>Mata Pelajaran</td><td>${escapeHtml(form.mapel)}</td></tr>
              <tr><td>Kelas/Semester</td><td>${escapeHtml(form.kelas)}</td></tr>
              <tr><td>Durasi Pertemuan</td><td>${escapeHtml(form.durasi)} — ${escapeHtml(String(form.pertemuan))} pertemuan</td></tr>
            </table>
          </td>
          <td style="vertical-align:top; padding:8px; border:1px solid #ddd;">
            <h2>Identifikasi</h2>
            <table style="width:100%">
              <tr><td>Siswa</td><td>${escapeHtml(siswa)}</td></tr>
              <tr><td>Materi Pelajaran</td><td>${escapeHtml(form.materi)}</td></tr>
              <tr><td>Capaian Dimensi Lulusan</td><td>${escapeHtml(dimensi)}</td></tr>
            </table>
          </td></tr>

          <tr><td colspan="2" style="padding:8px; border:1px solid #ddd;">
            <h2>Desain Pembelajaran</h2>
            <table style="width:100%">
              <tr><td>Capaian Pembelajaran</td><td>${escapeHtml(form.capaian)}</td></tr>
              <tr><td>Lintas Disiplin Ilmu</td><td>${escapeHtml(lintas)}</td></tr>
              <tr><td>Tujuan Pembelajaran</td><td>${escapeHtml(form.tujuan)}</td></tr>
              <tr><td>Topik Pembelajaran</td><td>${escapeHtml(topik)}</td></tr>
              <tr><td>Praktik Pedagogis per Pertemuan</td><td>${escapeHtml(praktik.join(" | "))}</td></tr>
              <tr><td>Kemitraan Pembelajaran</td><td>${escapeHtml(kemitraan)}</td></tr>
              <tr><td>Lingkungan Pembelajaran</td><td>${escapeHtml(lingkungan)}</td></tr>
              <tr><td>Pemanfaatan Digital</td><td>${escapeHtml(digital)}</td></tr>
            </table>
          </td></tr>

          <tr><td colspan="2" style="padding:8px; border:1px solid #ddd;">
            <h2>Pengalaman Belajar</h2>
            <table style="width:100%">
              <tr><td>Memahami (kegiatan awal)</td><td>Aktivitas pembuka yang berkesadaran dan bermakna: pengkondisian, apersepsi terkait ${escapeHtml(topik)}.</td></tr>
              <tr><td>Mengaplikasi (kegiatan inti)</td><td>Langkah pembelajaran inti sesuai praktik pedagogis: ${escapeHtml(praktik.join(" → "))} dengan kegiatan berpikir, praktik, dan kolaborasi.</td></tr>
              <tr><td>Refleksi (kegiatan penutup)</td><td>Refleksi dan umpan balik: presentasi singkat, diskusi, dan penugasan tindak lanjut.</td></tr>
            </table>
          </td></tr>

          <tr><td colspan="2" style="padding:8px; border:1px solid #ddd;">
            <h2>Asesmen Pembelajaran</h2>
            <table style="width:100%">
              <tr><td>Asesmen Awal</td><td>Diagnostik/apersepsi singkat berupa pertanyaan lisan atau pre-test singkat.</td></tr>
              <tr><td>Asesmen Proses</td><td>Observasi, rubrik penilaian, diskusi kelompok, catatan keterampilan.</td></tr>
              <tr><td>Asesmen Akhir</td><td>Produk tugas, presentasi, portofolio sesuai tujuan pembelajaran.</td></tr>
            </table>
          </td></tr>
        </table>

        <div style="display:flex; justify-content:space-between; margin-top:30px;">
          <div style="text-align:left;">
            <div>${escapeHtml(form.kepala)}</div>
            <div>NIP: ${escapeHtml(form.nipKepala)}</div>
          </div>
          <div style="text-align:right;">
            <div>${escapeHtml(form.guru)}</div>
            <div>NIP: ${escapeHtml(form.nipGuru)}</div>
          </div>
        </div>
      </div>
    `;

    setOutputHtml(html);
  }

  function escapeHtml(s) {
    if (!s && s !== 0) return "";
    return String(s)
      .replace(/&/g, "&amp;")
      .replace(/</g, "&lt;")
      .replace(/>/g, "&gt;")
      .replace(/\"/g, "&quot;")
      .replace(/'/g, "&#39;");
  }

  function handleGenerate(e) {
    e && e.preventDefault();
    if (!validate()) return;
    buildOutputHtml();
  }

  async function handleCopyAndOpenDocs() {
    if(!outputHtml) return alert("Silakan generate terlebih dahulu.");
    // Copy HTML + plaintext to clipboard
    try {
      // copy HTML
      await navigator.clipboard.writeText(outputHtml);
    } catch (err) {
      console.warn("Clipboard API failed, fallback." , err);
    }
    // Open Google Docs in new tab - user can paste (browser security prevents auto-paste into Google Docs)
    window.open("https://docs.google.com/document/create", "_blank");
    alert("Konten telah disalin ke clipboard. Buka tab Google Dokumen yang terbuka dan lakukan paste (Ctrl+V / Cmd+V). Karena kebijakan keamanan browser, aplikasi tidak dapat menempelkan langsung ke Google Dokumen.");
  }

  function handlePrint() {
    if(!outputHtml) return alert("Silakan generate terlebih dahulu.");
    const w = window.open('', '_blank');
    w.document.write(`<!doctype html><html><head><meta charset=\"utf-8\"><title>RPM</title><style>body{font-family:sans-serif;padding:20px;}table{width:100%;border-collapse:collapse}td,th{border:1px solid #ddd;padding:6px}</style></head><body>${outputHtml}</body></html>`);
    w.document.close();
    w.focus();
    setTimeout(()=>{ w.print(); }, 500);
  }

  function adminLogin(username, password) {
    if (username === ADMIN_USERNAME && password === ADMIN_PASSWORD) {
      setLoggedIn(true); setIsAdmin(true); setShowLogin(false); return true;
    }
    return false;
  }

  function createToken() {
    // admin only
    if (!isAdmin) return;
    const newToken = Math.random().toString(36).slice(2,10).toUpperCase();
    const toks = getTokens(); toks.push(newToken); saveTokens(toks);
    alert("Token baru dibuat: " + newToken + "\nBagikan token ini ke peserta (token tersimpan di localStorage).\nTokens dapat dikelola melalui console (localStorage) atau fungsionalitas tambahan.");
  }

  function logout() { setLoggedIn(false); setIsAdmin(false); setShowLogin(true); setTokenLogin(""); }

  // Render
  return (
    <div className="min-h-screen bg-gray-50 p-6">
      <div className="max-w-6xl mx-auto bg-white shadow rounded p-6">
        <header className="flex items-center justify-between mb-6">
          <h1 className="text-2xl font-semibold">Generator RPM IIF SADEWA GOA</h1>
          <div className="space-x-2">
            {!loggedIn ? null : <button onClick={logout} className="px-3 py-1 border rounded">Logout</button>}
            {isAdmin && <button onClick={createToken} className="px-3 py-1 bg-blue-600 text-white rounded">Buat Token</button>}
          </div>
        </header>

        {showLogin ? (
          <div className="mb-6 p-4 border rounded">
            <h2 className="font-medium mb-2">Login</h2>
            <p className="text-sm text-gray-600 mb-2">Admin: masukkan username & password. Peserta lain: masukkan token yang dibuat oleh admin.</p>
            <LoginForm onAdminLogin={adminLogin} onTokenLogin={(t)=>{ setTokenLogin(t); }} />
          </div>
        ) : (
          <div className="mb-6 p-4 border rounded">
            <div className="text-sm text-gray-700">Anda masuk sebagai {isAdmin? 'Admin' : 'Peserta'}.</div>
          </div>
        )}

        <form onSubmit={handleGenerate}>
          <div className="grid grid-cols-2 gap-4">
            <Input label="Nama Satuan Pendidikan" name="satuan" value={form.satuan} onChange={handleChange} error={errors.satuan} />
            <Input label="Nama Guru" name="guru" value={form.guru} onChange={handleChange} error={errors.guru} />
            <Input label="NIP Guru" name="nipGuru" value={form.nipGuru} onChange={handleChange} error={errors.nipGuru} />
            <Input label="Nama Kepala Sekolah" name="kepala" value={form.kepala} onChange={handleChange} error={errors.kepala} />
            <Input label="NIP Kepala Sekolah" name="nipKepala" value={form.nipKepala} onChange={handleChange} error={errors.nipKepala} />
            <Select label="Jenjang Pendidikan" name="jenjang" value={form.jenjang} onChange={handleChange} options={["SD","SMP","SMA"]} />
            <Input label="Pilihan Kelas/Semester" name="kelas" value={form.kelas} onChange={handleChange} error={errors.kelas} />
            <Input label="Mata Pelajaran (Mapel)" name="mapel" value={form.mapel} onChange={handleChange} error={errors.mapel} />
            <Textarea label="Capaian Pembelajaran (CP)" name="capaian" value={form.capaian} onChange={handleChange} error={errors.capaian} />
            <Textarea label="Tujuan Pembelajaran" name="tujuan" value={form.tujuan} onChange={handleChange} error={errors.tujuan} />
            <Textarea label="Materi Pelajaran" name="materi" value={form.materi} onChange={handleChange} error={errors.materi} />
            <Input label="Jumlah Pertemuan (angka)" name="pertemuan" value={form.pertemuan} onChange={(e)=>setForm({...form, pertemuan: Math.max(1, Number(e.target.value) || 1)})} />
            <Input label="Durasi Setiap Pertemuan" name="durasi" value={form.durasi} onChange={handleChange} />

            <div className="col-span-2">
              <label className="block font-medium mb-1">Praktik Pedagogis per Pertemuan (opsional — bisa diisi per pertemuan)</label>
              <div className="grid grid-cols-3 gap-2">
                {Array.from({length: form.pertemuan}).map((_,i)=> (
                  <select key={i} className="p-2 border rounded" value={form.praktikPertemuan[i]||""} onChange={(ev)=>{
                    const arr = [...form.praktikPertemuan]; arr[i]=ev.target.value; setForm({...form, praktikPertemuan: arr});
                  }}>
                    <option value="">-- Pilih --</option>
                    <option>Inkuiri-Discovery Learning</option>
                    <option>PjBL</option>
                    <option>Problem Based Learning</option>
                    <option>Game Based Learning</option>
                    <option>Station Learning</option>
                  </select>
                ))}
              </div>
            </div>

            <div className="col-span-2">
              <label className="block font-medium mb-1">Dimensi Lulusan (pilih multi)</label>
              <div className="grid grid-cols-4 gap-2">
                {["Keimanan & Ketakwaan","Kewargaan","Penalaran Kritis","Kreativitas","Kolaborasi","Kemandirian","Kesehatan","Komunikasi"].map(d=> (
                  <label key={d} className="inline-flex items-center space-x-2">
                    <input type="checkbox" checked={form.dimensi.includes(d)} onChange={()=>handleDimensiToggle(d)} />
                    <span className="text-sm">{d}</span>
                  </label>
                ))}
              </div>
            </div>
          </div>

          <div className="mt-4 flex space-x-2">
            <button type="submit" className="px-4 py-2 bg-green-600 text-white rounded">Generate RPM</button>
            <button type="button" onClick={handleCopyAndOpenDocs} className="px-4 py-2 border rounded">Salin & Buka di Google Dokumen</button>
            <button type="button" onClick={handlePrint} className="px-4 py-2 border rounded">Cetak PDF</button>
          </div>
        </form>

        {outputHtml && (
          <div className="mt-6 p-4 border rounded bg-gray-50">
            <h2 className="font-medium mb-2">Hasil Rencana Pembelajaran (Pratinjau)</h2>
            <div dangerouslySetInnerHTML={{__html: outputHtml}} />
          </div>
        )}
      </div>
    </div>
  );
}

// Small UI components
function Input({label,name,value,onChange,error}){
  return (
    <label className="block">
      <div className="text-sm font-medium">{label}</div>
      <input name={name} value={value} onChange={onChange} className="mt-1 p-2 border rounded w-full" />
      {error && <div className="text-xs text-red-600">{error}</div>}
    </label>
  );
}
function Select({label,name,value,onChange,options}){
  return (
    <label className="block">
      <div className="text-sm font-medium">{label}</div>
      <select name={name} value={value} onChange={onChange} className="mt-1 p-2 border rounded w-full">
        {options.map(o=> <option key={o} value={o}>{o}</option>)}
      </select>
    </label>
  );
}
function Textarea({label,name,value,onChange,error}){
  return (
    <label className="block">
      <div className="text-sm font-medium">{label}</div>
      <textarea name={name} value={value} onChange={onChange} className="mt-1 p-2 border rounded w-full" rows={3} />
      {error && <div className="text-xs text-red-600">{error}</div>}
    </label>
  );
}

function LoginForm({onAdminLogin,onTokenLogin}){
  const [user,setUser]=useState("");
  const [pass,setPass]=useState("");
  const [token,setToken]=useState("");
  return (
    <div className="grid grid-cols-2 gap-2">
      <div>
        <div className="text-xs text-gray-600 mb-1">Login Admin</div>
        <input placeholder="username" className="p-2 border rounded w-full mb-1" value={user} onChange={e=>setUser(e.target.value)} />
        <input placeholder="password" type="password" className="p-2 border rounded w-full mb-1" value={pass} onChange={e=>setPass(e.target.value)} />
        <button onClick={()=>{ const ok = onAdminLogin(user,pass); if(!ok) alert('Login Admin gagal'); }} className="px-3 py-1 bg-gray-800 text-white rounded">Login Admin</button>
      </div>
      <div>
        <div className="text-xs text-gray-600 mb-1">Login Peserta (Token)</div>
        <input placeholder="Masukkan token" className="p-2 border rounded w-full mb-1" value={token} onChange={e=>setToken(e.target.value)} />
        <button onClick={()=>onTokenLogin(token)} className="px-3 py-1 border rounded">Login dengan Token</button>
      </div>
    </div>
  );
}
