<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Finansal Analiz</title>
    
    <!-- Firebase SDK -->
    <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-auth-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-firestore-compat.js"></script>
    
    <!-- Chart.js -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.min.js"></script>
    
    <!-- Tailwind CSS -->
    <link href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css" rel="stylesheet">
    
    <style>
        * { box-sizing: border-box; }
        body { margin: 0; padding: 0; font-family: system-ui, -apple-system, sans-serif; font-size: 12px; }
        .gradient-bg { background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); }
        .chart-container { position: relative; height: 120px; margin: 5px 0; }
        .compact-input { padding: 4px 8px; font-size: 11px; }
        .compact-card { padding: 8px; }
        .section { display: none; }
        .section.active { display: block; }
        .hidden { display: none !important; }
        .tab-btn { padding: 6px 12px; font-size: 11px; border-radius: 4px; margin: 2px; }
        .tab-btn.active { background: #3b82f6; color: white; }
        .tab-btn:not(.active) { background: #e5e7eb; color: #374151; }
        .mini-card { padding: 6px; margin: 4px 0; border-radius: 6px; }
        .text-xs { font-size: 10px; }
        .text-sm { font-size: 11px; }
        @media (max-width: 640px) {
            .chart-container { height: 100px; }
            .compact-card { padding: 6px; }
            .text-xs { font-size: 9px; }
        }
    </style>
</head>
<body class="bg-gray-50">
    <!-- Login Screen -->
    <div id="loginScreen" class="min-h-screen gradient-bg flex items-center justify-center p-2">
        <div class="bg-white rounded-lg shadow-lg p-4 w-full max-w-sm">
            <div class="text-center mb-4">
                <h1 class="text-lg font-bold text-gray-800">💰 Finansal Analiz</h1>
                <p class="text-xs text-gray-600">Güvenli Giriş</p>
            </div>
            
            <!-- Important Info -->
            <div class="bg-amber-50 border border-amber-300 rounded p-3 mb-3">
                <div class="flex items-start space-x-2">
                    <span class="text-amber-600">⚠️</span>
                    <div class="flex-1">
                        <h3 class="text-xs font-bold text-amber-800 mb-1">Önemli Bilgi</h3>
                        <div class="text-xs text-amber-700 space-y-1">
                            <p>• Google hesabınızla giriş yapınız</p>
                            <p>• Veriler Firebase'de güvenle saklanır</p>
                            <p>• Veri güvenliği Google Firebase sorumluluğundadır</p>
                        </div>
                        
                        <div class="mt-2 p-2 bg-white rounded border">
                            <label class="flex items-start space-x-2 cursor-pointer">
                                <input type="checkbox" id="termsAccepted" required class="mt-0.5 w-3 h-3">
                                <span class="text-xs text-amber-800">
                                    <strong>Zorunlu:</strong> Yukarıdaki bilgileri okudum ve kabul ediyorum.
                                </span>
                            </label>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Login/Register Toggle -->
            <div class="flex mb-3">
                <button id="loginTab" class="flex-1 py-2 px-3 text-xs font-semibold bg-blue-600 text-white rounded-l">Giriş Yap</button>
                <button id="registerTab" class="flex-1 py-2 px-3 text-xs font-semibold bg-gray-300 text-gray-700 rounded-r">Kayıt Ol</button>
            </div>

            <!-- Login Form -->
            <div id="loginForm" class="space-y-2">
                <!-- Google Priority Warning -->
                <div class="bg-blue-50 border border-blue-300 rounded p-2 mb-3">
                    <div class="flex items-start space-x-2">
                        <span class="text-blue-600">🔑</span>
                        <div class="flex-1">
                            <h3 class="text-xs font-bold text-blue-800 mb-1">Önerilen Giriş</h3>
                            <p class="text-xs text-blue-700">
                                <strong>Google ile giriş yapmanız önerilir!</strong><br>
                                • Daha güvenli ve hızlı<br>
                                • Şifre unutma sorunu yok<br>
                                • Otomatik hesap koruması
                            </p>
                        </div>
                    </div>
                </div>
                
                <button id="googleSignInBtn" class="w-full bg-red-600 hover:bg-red-700 text-white font-semibold py-2 px-3 rounded text-xs flex items-center justify-center space-x-2">
                    <span>🔑 Google ile Giriş (Önerilen)</span>
                </button>
                
                <div class="text-center text-xs text-gray-500">veya</div>
                
                <form id="companyLoginForm" class="space-y-2">
                    <input type="text" id="loginCompanyName" placeholder="Şirket Adı" required class="w-full compact-input border rounded">
                    <input type="password" id="loginPassword" placeholder="Şifre" required class="w-full compact-input border rounded">
                    <button type="submit" class="w-full bg-blue-600 hover:bg-blue-700 text-white py-2 px-3 rounded text-xs">Şirket Girişi</button>
                </form>
                

            </div>

            <!-- Register Form -->
            <div id="registerForm" class="space-y-2 hidden">
                <!-- Google Priority Warning for Registration -->
                <div class="bg-orange-50 border border-orange-300 rounded p-2 mb-3">
                    <div class="flex items-start space-x-2">
                        <span class="text-orange-600">⚠️</span>
                        <div class="flex-1">
                            <h3 class="text-xs font-bold text-orange-800 mb-1">Dikkat!</h3>
                            <p class="text-xs text-orange-700">
                                <strong>Önce Google ile giriş yapmanız önerilir!</strong><br>
                                • Şirket kaydı yapmadan önce Google hesabınızla deneyin<br>
                                • Daha güvenli ve pratik<br>
                                • Şifre yönetimi gerektirmez
                            </p>
                        </div>
                    </div>
                </div>
                
                <form id="companyRegisterForm" class="space-y-2">
                    <input type="text" id="registerCompanyName" placeholder="Şirket Adı *" required class="w-full compact-input border rounded">
                    <input type="email" id="registerEmail" placeholder="E-posta Adresi *" required class="w-full compact-input border rounded">
                    <input type="password" id="registerPassword" placeholder="Şifre (min 6 karakter) *" required minlength="6" class="w-full compact-input border rounded">
                    <input type="password" id="registerPasswordConfirm" placeholder="Şifre Tekrar *" required class="w-full compact-input border rounded">
                    
                    <select id="registerSector" required class="w-full compact-input border rounded">
                        <option value="">Sektör Seçin *</option>
                        <option value="teknoloji">Teknoloji</option>
                        <option value="imalat">İmalat</option>
                        <option value="hizmet">Hizmet</option>
                        <option value="ticaret">Ticaret</option>
                        <option value="finans">Finans</option>
                        <option value="sağlık">Sağlık</option>
                        <option value="eğitim">Eğitim</option>
                        <option value="inşaat">İnşaat</option>
                        <option value="turizm">Turizm</option>
                        <option value="diğer">Diğer</option>
                    </select>
                    
                    <select id="registerEmployeeCount" required class="w-full compact-input border rounded">
                        <option value="">Çalışan Sayısı *</option>
                        <option value="1-10">1-10 Kişi</option>
                        <option value="11-50">11-50 Kişi</option>
                        <option value="51-250">51-250 Kişi</option>
                        <option value="251-1000">251-1000 Kişi</option>
                        <option value="1000+">1000+ Kişi</option>
                    </select>
                    
                    <input type="text" id="registerPhone" placeholder="Telefon (Opsiyonel)" class="w-full compact-input border rounded">
                    
                    <button type="submit" class="w-full bg-green-600 hover:bg-green-700 text-white py-2 px-3 rounded text-xs font-semibold">🏢 Şirket Kaydı Oluştur</button>
                </form>
            </div>
        </div>
    </div>

    <!-- Account Setup Modal -->
    <div id="accountSetupModal" class="hidden fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-2 z-50">
        <div class="bg-white rounded-lg p-4 w-full max-w-sm">
            <h2 class="text-sm font-bold mb-3">Hesap Kurulumu</h2>
            <form id="accountSetupForm" class="space-y-3">
                <input type="text" id="setupCompanyName" placeholder="Şirket Adı" required class="w-full compact-input border rounded">
                <select id="setupSector" required class="w-full compact-input border rounded">
                    <option value="">Sektör Seçin</option>
                    <option value="teknoloji">Teknoloji</option>
                    <option value="imalat">İmalat</option>
                    <option value="hizmet">Hizmet</option>
                    <option value="ticaret">Ticaret</option>
                    <option value="finans">Finans</option>
                    <option value="diğer">Diğer</option>
                </select>
                <select id="setupEmployeeCount" required class="w-full compact-input border rounded">
                    <option value="">Çalışan Sayısı</option>
                    <option value="1-10">1-10</option>
                    <option value="11-50">11-50</option>
                    <option value="51-250">51-250</option>
                    <option value="250+">250+</option>
                </select>
                <button type="submit" class="w-full bg-blue-600 hover:bg-blue-700 text-white py-2 px-3 rounded text-xs">Hesabı Tamamla</button>
            </form>
        </div>
    </div>

    <!-- Main Dashboard -->
    <div id="dashboard" class="hidden min-h-screen bg-gray-50">
        <!-- Compact Header -->
        <header class="bg-white shadow-sm border-b p-2">
            <div class="flex items-center justify-between">
                <h1 class="text-sm font-bold text-gray-800">💰 Finansal Analiz</h1>
                <div class="flex items-center space-x-2">
                    <span id="currentCompany" class="text-xs text-gray-600 hidden sm:block"></span>
                    <button id="logoutBtn" class="bg-red-500 hover:bg-red-600 text-white px-2 py-1 rounded text-xs">Çıkış</button>
                </div>
            </div>
        </header>

        <!-- Tab Navigation -->
        <div class="bg-white border-b p-2">
            <div class="flex flex-wrap justify-center">
                <button class="tab-btn active" data-section="dataEntry">📊 Veri</button>
                <button class="tab-btn" data-section="reports">📋 Rapor</button>
                <button id="superAdminTab" class="tab-btn hidden" data-section="superAdmin">👑 Admin</button>
            </div>
        </div>

        <!-- Main Content -->
        <main class="p-2">
            <!-- Data Entry Section -->
            <div id="dataEntrySection" class="section active">
                <div class="bg-white rounded-lg shadow compact-card mb-2">
                    <div class="flex items-center justify-between mb-3">
                        <h2 class="text-sm font-bold text-gray-800">Finansal Veri Girişi</h2>
                        <div class="flex items-center space-x-1">
                            <label class="flex items-center cursor-pointer">
                                <input type="checkbox" id="aiToggle" checked class="sr-only">
                                <div class="relative">
                                    <div class="block bg-green-600 w-10 h-5 rounded-full"></div>
                                    <div class="dot absolute left-0.5 top-0.5 bg-white w-4 h-4 rounded-full transition"></div>
                                </div>
                                <span class="ml-1 text-xs">🤖 AI</span>
                            </label>
                        </div>
                    </div>
                    
                    <div class="grid grid-cols-1 sm:grid-cols-2 gap-3">
                        <!-- Income Statement -->
                        <div class="space-y-2">
                            <h3 class="text-xs font-semibold text-gray-700 border-b pb-1">Gelir Tablosu</h3>
                            <div class="space-y-1">
                                <input type="number" id="netSales" placeholder="Net Satışlar (TL)" class="w-full compact-input border rounded">
                                <input type="number" id="costOfSales" placeholder="Satış Maliyeti (TL)" class="w-full compact-input border rounded">
                                <input type="number" id="adminExpenses" placeholder="Yönetim Giderleri (TL)" class="w-full compact-input border rounded">
                                <input type="number" id="ebitda" placeholder="FAVÖK/EBITDA (TL)" class="w-full compact-input border rounded">
                                <input type="number" id="netProfit" placeholder="Net Kâr/Zarar (TL)" class="w-full compact-input border rounded">
                            </div>
                        </div>

                        <!-- Balance Sheet -->
                        <div class="space-y-2">
                            <h3 class="text-xs font-semibold text-gray-700 border-b pb-1">Bilanço</h3>
                            <div class="space-y-1">
                                <input type="number" id="currentAssets" placeholder="Dönen Varlıklar (TL)" class="w-full compact-input border rounded">
                                <input type="number" id="fixedAssets" placeholder="Duran Varlıklar (TL)" class="w-full compact-input border rounded">
                                <input type="number" id="shortTermDebt" placeholder="Kısa Vadeli Borçlar (TL)" class="w-full compact-input border rounded">
                                <input type="number" id="longTermDebt" placeholder="Uzun Vadeli Borçlar (TL)" class="w-full compact-input border rounded">
                                <input type="number" id="equity" placeholder="Özkaynaklar (TL)" class="w-full compact-input border rounded">
                            </div>
                        </div>
                    </div>

                    <!-- Demo Scenarios -->
                    <div class="mt-3 flex flex-wrap gap-1">
                        <button id="loadScenario1" class="bg-red-500 hover:bg-red-600 text-white px-2 py-1 rounded text-xs">📚 S1</button>
                        <button id="loadScenario2" class="bg-orange-500 hover:bg-orange-600 text-white px-2 py-1 rounded text-xs">📚 S2</button>
                        <button id="loadScenario3" class="bg-red-600 hover:bg-red-700 text-white px-2 py-1 rounded text-xs">📚 S3</button>
                        <button id="loadScenario4" class="bg-green-500 hover:bg-green-600 text-white px-2 py-1 rounded text-xs">📚 S4</button>
                    </div>
                    
                    <button id="saveDataBtn" class="w-full bg-green-600 hover:bg-green-700 text-white py-2 px-3 rounded font-semibold text-xs mt-3">
                        💾 Kaydet ve AI Analizi Yap
                    </button>
                </div>

                <!-- Quick Charts -->
                <div class="grid grid-cols-2 gap-2">
                    <div class="bg-white rounded shadow p-2">
                        <h4 class="text-xs font-semibold mb-1">Gelir Trendi</h4>
                        <div class="chart-container">
                            <canvas id="revenueChart"></canvas>
                        </div>
                    </div>
                    <div class="bg-white rounded shadow p-2">
                        <h4 class="text-xs font-semibold mb-1">Kârlılık</h4>
                        <div class="chart-container">
                            <canvas id="profitChart"></canvas>
                        </div>
                    </div>
                    <div class="bg-white rounded shadow p-2">
                        <h4 class="text-xs font-semibold mb-1">Varlık Dağılımı</h4>
                        <div class="chart-container">
                            <canvas id="assetChart"></canvas>
                        </div>
                    </div>
                    <div class="bg-white rounded shadow p-2">
                        <h4 class="text-xs font-semibold mb-1">Borç Yapısı</h4>
                        <div class="chart-container">
                            <canvas id="debtChart"></canvas>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Reports Section -->
            <div id="reportsSection" class="section">
                <div class="bg-white rounded-lg shadow compact-card">
                    <h2 class="text-sm font-bold text-gray-800 mb-3">📋 Finansal Raporlar</h2>
                    <div id="reportContent" class="space-y-3">
                        <p class="text-gray-600 text-xs">Raporlar için önce veri girişi yapınız.</p>
                    </div>
                </div>
            </div>

            <!-- Super Admin Section -->
            <div id="superAdminSection" class="section">
                <div class="bg-white rounded-lg shadow compact-card">
                    <h2 class="text-sm font-bold text-red-800 mb-3">👑 Süper Admin</h2>
                    <div id="allCompaniesData" class="overflow-x-auto">
                        <table class="min-w-full text-xs">
                            <thead class="bg-gray-50">
                                <tr>
                                    <th class="px-1 py-1 text-left">Şirket Adı</th>
                                    <th class="px-1 py-1 text-left">Email</th>
                                    <th class="px-1 py-1 text-left">Giriş Tipi</th>
                                    <th class="px-1 py-1 text-left">Durum</th>
                                    <th class="px-1 py-1 text-left">İşlemler</th>
                                </tr>
                            </thead>
                            <tbody id="companiesTableBody">
                                <!-- Firebase data will be populated here -->
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>
        </main>
    </div>

    <script>
        // Firebase Configuration
        const firebaseConfig = {
            apiKey: "AIzaSyAF8ZcI4lYPjnojma094lo_orSfX8I9Fh8",
            authDomain: "finansal-analiz-platform.firebaseapp.com",
            projectId: "finansal-analiz-platform",
            storageBucket: "finansal-analiz-platform.appspot.com",
            messagingSenderId: "123456789012",
            appId: "1:123456789012:web:abcdef123456"
        };
        
        firebase.initializeApp(firebaseConfig);
        const auth = firebase.auth();
        const db = firebase.firestore();
        const googleProvider = new firebase.auth.GoogleAuthProvider();

        // Global Variables
        let currentUser = null;
        let isSuperAdmin = false;
        let financialData = {};
        let charts = {};
        let userProfile = null;
        let aiAnalysisEnabled = true;
        let previousData = null;

        // Gemini AI Configuration
        const GEMINI_API_KEY = 'AIzaSyAF8ZcI4lYPjnojma094lo_orSfX8I9Fh8';
        const GEMINI_API_URL = 'https://generativelanguage.googleapis.com/v1beta/models/gemini-pro:generateContent';

        // DOM Elements
        const loginScreen = document.getElementById('loginScreen');
        const dashboard = document.getElementById('dashboard');
        const googleSignInBtn = document.getElementById('googleSignInBtn');

        const accountSetupModal = document.getElementById('accountSetupModal');
        const accountSetupForm = document.getElementById('accountSetupForm');

        // Firebase Auth State Observer
        auth.onAuthStateChanged(async (user) => {
            if (user) {
                currentUser = user.uid;
                try {
                    const userDoc = await db.collection('users').doc(user.uid).get();
                    if (userDoc.exists) {
                        userProfile = userDoc.data();
                        document.getElementById('currentCompany').textContent = userProfile.companyName;
                        if (user.email === 'admin@finansalanaliz.com') {
                            isSuperAdmin = true;
                            document.getElementById('superAdminTab').classList.remove('hidden');
                        }
                        showDashboard();
                        await loadUserData();
                    } else {
                        showAccountSetup();
                    }
                } catch (error) {
                    console.error('User profile error:', error);
                    alert('Kullanıcı bilgileri yüklenirken hata oluştu.');
                }
            } else {
                currentUser = null;
                userProfile = null;
                isSuperAdmin = false;
                document.getElementById('superAdminTab').classList.add('hidden');
                showLoginScreen();
            }
        });

        // Login/Register Tab Toggle
        document.getElementById('loginTab').addEventListener('click', function() {
            document.getElementById('loginForm').classList.remove('hidden');
            document.getElementById('registerForm').classList.add('hidden');
            this.classList.add('bg-blue-600', 'text-white');
            this.classList.remove('bg-gray-300', 'text-gray-700');
            document.getElementById('registerTab').classList.add('bg-gray-300', 'text-gray-700');
            document.getElementById('registerTab').classList.remove('bg-blue-600', 'text-white');
        });

        document.getElementById('registerTab').addEventListener('click', function() {
            document.getElementById('registerForm').classList.remove('hidden');
            document.getElementById('loginForm').classList.add('hidden');
            this.classList.add('bg-blue-600', 'text-white');
            this.classList.remove('bg-gray-300', 'text-gray-700');
            document.getElementById('loginTab').classList.add('bg-gray-300', 'text-gray-700');
            document.getElementById('loginTab').classList.remove('bg-blue-600', 'text-white');
        });

        // Google Sign In (Demo Mode)
        googleSignInBtn.addEventListener('click', async () => {
            const termsAccepted = document.getElementById('termsAccepted');
            if (!termsAccepted.checked) {
                alert('⚠️ Lütfen şartları okuyup onaylayın!');
                termsAccepted.focus();
                return;
            }
            
            // Demo Google Login
            const demoEmail = 'demo@gmail.com';
            const demoCompany = 'Demo Şirketi (Google)';
            
            try {
                googleSignInBtn.disabled = true;
                googleSignInBtn.textContent = 'Google bağlanıyor...';
                
                // Simulate Google login
                setTimeout(async () => {
                    currentUser = 'google_demo_user';
                    userProfile = {
                        companyName: demoCompany,
                        email: demoEmail,
                        sector: 'teknoloji',
                        employeeCount: '11-50',
                        createdAt: new Date(),
                        lastLogin: new Date(),
                        isActive: true,
                        loginMethod: 'google'
                    };
                    
                    document.getElementById('currentCompany').textContent = demoCompany;
                    showDashboard();
                    alert(`✅ Google ile giriş başarılı!\n📧 ${demoEmail}\n🏢 ${demoCompany}`);
                    
                    googleSignInBtn.disabled = false;
                    googleSignInBtn.innerHTML = '<span>🔑 Google ile Giriş</span>';
                }, 1500);
                
            } catch (error) {
                console.error('Google sign in error:', error);
                alert('❌ Google ile giriş hatası: ' + error.message);
                googleSignInBtn.disabled = false;
                googleSignInBtn.innerHTML = '<span>🔑 Google ile Giriş</span>';
            }
        });

        // Company Registration
        document.getElementById('companyRegisterForm').addEventListener('submit', async (e) => {
            e.preventDefault();
            
            const termsAccepted = document.getElementById('termsAccepted');
            if (!termsAccepted.checked) {
                alert('⚠️ Lütfen şartları okuyup onaylayın!');
                termsAccepted.focus();
                return;
            }
            
            const companyName = document.getElementById('registerCompanyName').value.trim();
            const email = document.getElementById('registerEmail').value.trim();
            const password = document.getElementById('registerPassword').value;
            const passwordConfirm = document.getElementById('registerPasswordConfirm').value;
            const sector = document.getElementById('registerSector').value;
            const employeeCount = document.getElementById('registerEmployeeCount').value;
            const phone = document.getElementById('registerPhone').value.trim();
            
            // Validation
            if (!companyName || !email || !password || !sector || !employeeCount) {
                alert('⚠️ Lütfen zorunlu alanları doldurun!\n(* ile işaretli alanlar)');
                return;
            }
            
            if (password !== passwordConfirm) {
                alert('⚠️ Şifreler eşleşmiyor!');
                return;
            }
            
            if (password.length < 6) {
                alert('⚠️ Şifre en az 6 karakter olmalıdır!');
                return;
            }
            
            const submitBtn = e.target.querySelector('button[type="submit"]');
            submitBtn.disabled = true;
            submitBtn.textContent = 'Kaydediliyor...';
            
            try {
                // Check if company already exists in localStorage
                const companies = JSON.parse(localStorage.getItem('companies') || '{}');
                const companyId = companyName.toLowerCase().replace(/[^a-z0-9]/g, '_');
                
                if (companies[companyId]) {
                    alert('⚠️ Bu şirket adı zaten kayıtlı! Lütfen farklı bir ad seçin.');
                    return;
                }
                
                // Check email
                const existingEmail = Object.values(companies).find(comp => comp.email === email);
                if (existingEmail) {
                    alert('⚠️ Bu e-posta adresi zaten kayıtlı!');
                    return;
                }
                
                const companyData = {
                    companyName,
                    email,
                    password, // In real app, this should be hashed
                    sector,
                    employeeCount,
                    phone: phone || null,
                    createdAt: new Date().toISOString(),
                    lastLogin: null,
                    isActive: true,
                    loginMethod: 'company'
                };
                
                // Save to localStorage
                companies[companyId] = companyData;
                localStorage.setItem('companies', JSON.stringify(companies));
                
                // Also try Firebase if available
                try {
                    await db.collection('companies').doc(companyId).set(companyData);
                } catch (error) {
                    console.log('Firebase not available, saved to localStorage only');
                }
                
                alert(`🎉 ${companyName} başarıyla kaydedildi!\n\n📧 E-posta: ${email}\n🏢 Şirket: ${companyName}\n\nŞimdi giriş yapabilirsiniz.`);
                
                // Switch to login tab
                document.getElementById('loginTab').click();
                document.getElementById('loginCompanyName').value = companyName;
                document.getElementById('companyRegisterForm').reset();
                
            } catch (error) {
                console.error('Registration error:', error);
                alert('❌ Kayıt hatası: ' + error.message);
            } finally {
                submitBtn.disabled = false;
                submitBtn.textContent = '🏢 Şirket Kaydı Oluştur';
            }
        });

        // Company Login
        document.getElementById('companyLoginForm').addEventListener('submit', async (e) => {
            e.preventDefault();
            
            const termsAccepted = document.getElementById('termsAccepted');
            if (!termsAccepted.checked) {
                alert('⚠️ Lütfen şartları okuyup onaylayın!');
                termsAccepted.focus();
                return;
            }
            
            const companyName = document.getElementById('loginCompanyName').value.trim();
            const password = document.getElementById('loginPassword').value;
            
            if (!companyName || !password) {
                alert('⚠️ Lütfen şirket adı ve şifrenizi girin!');
                return;
            }
            
            // SÜPER ADMİN GİRİŞİ - Şirket adı "süper admin" ve şifre "030714"
            if (companyName.toLowerCase() === 'süper admin' && password === '030714') {
                currentUser = 'süper admin';
                isSuperAdmin = true;
                userProfile = {
                    companyName: 'Süper Admin',
                    email: 'admin@sistem.com',
                    sector: 'yönetim',
                    employeeCount: '1',
                    isActive: true,
                    loginMethod: 'admin'
                };
                document.getElementById('currentCompany').textContent = 'Süper Admin';
                document.getElementById('superAdminTab').classList.remove('hidden');
                showDashboard();
                loadAllCompaniesData();
                document.getElementById('companyLoginForm').reset();
                alert('👑 Süper Admin girişi başarılı!\n🔧 Tüm şirket verilerine erişiminiz var.');
                return;
            }
            
            try {
                const submitBtn = e.target.querySelector('button[type="submit"]');
                submitBtn.disabled = true;
                submitBtn.textContent = 'Giriş yapılıyor...';
                
                const companyId = companyName.toLowerCase().replace(/[^a-z0-9]/g, '_');
                let companyData = null;
                
                // Try Firebase first
                try {
                    const doc = await db.collection('companies').doc(companyId).get();
                    if (doc.exists) {
                        companyData = doc.data();
                    }
                } catch (error) {
                    // Fallback to localStorage
                    const companies = JSON.parse(localStorage.getItem('companies') || '{}');
                    companyData = companies[companyId];
                }
                
                if (!companyData) {
                    alert('❌ Şirket bulunamadı! Lütfen kayıt olun.');
                    return;
                }
                
                if (companyData.password !== password) {
                    alert('❌ Yanlış şifre!');
                    return;
                }
                
                if (!companyData.isActive) {
                    alert('❌ Hesabınız pasif durumda. Lütfen yönetici ile iletişime geçin.');
                    return;
                }
                
                // Update last login
                try {
                    await db.collection('companies').doc(companyId).update({
                        lastLogin: firebase.firestore.FieldValue.serverTimestamp()
                    });
                } catch (error) {
                    // Update localStorage
                    const companies = JSON.parse(localStorage.getItem('companies') || '{}');
                    if (companies[companyId]) {
                        companies[companyId].lastLogin = new Date().toISOString();
                        localStorage.setItem('companies', JSON.stringify(companies));
                    }
                }
                
                // Set current user
                currentUser = companyId;
                userProfile = companyData;
                isSuperAdmin = false;
                document.getElementById('currentCompany').textContent = companyData.companyName;
                
                showDashboard();
                await loadUserData();
                
                alert(`✅ Hoş geldiniz!\n🏢 ${companyData.companyName}\n📧 ${companyData.email}`);
                
            } catch (error) {
                console.error('Login error:', error);
                alert('❌ Giriş hatası: ' + error.message);
            } finally {
                const submitBtn = e.target.querySelector('button[type="submit"]');
                submitBtn.disabled = false;
                submitBtn.textContent = 'Şirket Girişi';
            }
        });

        // Account Setup
        accountSetupForm.addEventListener('submit', async (e) => {
            e.preventDefault();
            const companyName = document.getElementById('setupCompanyName').value.trim();
            const sector = document.getElementById('setupSector').value;
            const employeeCount = document.getElementById('setupEmployeeCount').value;
            
            if (!companyName || !sector || !employeeCount) {
                alert('⚠️ Tüm alanları doldurun!');
                return;
            }
            
            try {
                const user = auth.currentUser;
                const userProfile = {
                    companyName, email: user.email, sector, employeeCount,
                    createdAt: firebase.firestore.FieldValue.serverTimestamp(),
                    lastLogin: firebase.firestore.FieldValue.serverTimestamp(),
                    isActive: true
                };
                
                await db.collection('users').doc(user.uid).set(userProfile);
                currentUser = user.uid;
                window.userProfile = userProfile;
                document.getElementById('currentCompany').textContent = companyName;
                showDashboard();
                alert(`🎉 ${companyName} hesabınız oluşturuldu!`);
            } catch (error) {
                console.error('Account setup error:', error);
                alert('❌ Hesap oluşturma hatası: ' + error.message);
            }
        });



        // Logout
        document.getElementById('logoutBtn').addEventListener('click', async function() {
            try {
                await auth.signOut();
            } catch (error) {
                console.error('Logout error:', error);
            }
        });

        // Navigation
        document.querySelectorAll('.tab-btn').forEach(btn => {
            btn.addEventListener('click', function() {
                const targetSection = this.dataset.section;
                
                document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active'));
                this.classList.add('active');
                
                document.querySelectorAll('.section').forEach(section => section.classList.remove('active'));
                document.getElementById(targetSection + 'Section').classList.add('active');
            });
        });

        // Save Data
        document.getElementById('saveDataBtn').addEventListener('click', async function() {
            const saveBtn = this;
            const originalText = saveBtn.textContent;
            saveBtn.textContent = 'Kaydediliyor...';
            saveBtn.disabled = true;
            
            const data = {
                netSales: parseFloat(document.getElementById('netSales').value) || 0,
                costOfSales: parseFloat(document.getElementById('costOfSales').value) || 0,
                adminExpenses: parseFloat(document.getElementById('adminExpenses').value) || 0,
                ebitda: parseFloat(document.getElementById('ebitda').value) || 0,
                netProfit: parseFloat(document.getElementById('netProfit').value) || 0,
                currentAssets: parseFloat(document.getElementById('currentAssets').value) || 0,
                fixedAssets: parseFloat(document.getElementById('fixedAssets').value) || 0,
                shortTermDebt: parseFloat(document.getElementById('shortTermDebt').value) || 0,
                longTermDebt: parseFloat(document.getElementById('longTermDebt').value) || 0,
                equity: parseFloat(document.getElementById('equity').value) || 0,
                lastUpdated: new Date().toISOString()
            };

            const hasData = Object.values(data).some(val => val > 0);
            if (!hasData) {
                alert('⚠️ En az bir veri girin!');
                saveBtn.textContent = originalText;
                saveBtn.disabled = false;
                return;
            }

            try {
                if (currentUser && currentUser !== 'süper admin') {
                    const saveData = {
                        ...data,
                        lastUpdated: new Date().toISOString(),
                        companyName: userProfile?.companyName || 'Bilinmeyen Şirket'
                    };
                    
                    // Try Firebase first
                    try {
                        await db.collection('financialData').doc(currentUser).set({
                            ...saveData,
                            lastUpdated: firebase.firestore.FieldValue.serverTimestamp()
                        });
                    } catch (error) {
                        // Fallback to localStorage
                        localStorage.setItem(`financialData_${currentUser}`, JSON.stringify(saveData));
                    }
                }
                
                if (!financialData) financialData = {};
                financialData[currentUser] = data;
                
                updateCharts();
                await generateReports();
                
                alert('✅ Veriler kaydedildi ve analiz tamamlandı!');
                
                setTimeout(() => {
                    if (confirm('📋 Raporlar sekmesine geçmek ister misiniz?')) {
                        document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active'));
                        document.querySelector('[data-section="reports"]').classList.add('active');
                        document.querySelectorAll('.section').forEach(section => section.classList.remove('active'));
                        document.getElementById('reportsSection').classList.add('active');
                    }
                }, 1000);
                
            } catch (error) {
                console.error('Save error:', error);
                alert('❌ Kaydetme hatası: ' + error.message);
            } finally {
                saveBtn.textContent = originalText;
                saveBtn.disabled = false;
            }
        });

        // Load User Data
        async function loadUserData() {
            if (!currentUser || currentUser === 'süper admin') return;
            
            try {
                let data = null;
                
                // Try Firebase first
                try {
                    const docRef = db.collection('financialData').doc(currentUser);
                    const doc = await docRef.get();
                    if (doc.exists) {
                        data = doc.data();
                    }
                } catch (error) {
                    // Fallback to localStorage
                    const localData = localStorage.getItem(`financialData_${currentUser}`);
                    if (localData) {
                        data = JSON.parse(localData);
                    }
                }
                
                if (data) {
                    if (!financialData) financialData = {};
                    financialData[currentUser] = data;
                    
                    populateFormFields(data);
                    updateCharts();
                    await generateReports();
                }
            } catch (error) {
                console.error('Load data error:', error);
            }
        }

        function populateFormFields(data) {
            Object.keys(data).forEach(key => {
                const element = document.getElementById(key);
                if (element && typeof data[key] === 'number') {
                    element.value = data[key];
                }
            });
        }

        // Update Charts
        function updateCharts() {
            let data;
            
            if (isSuperAdmin && financialData[currentUser]) {
                data = financialData[currentUser];
            } else {
                data = {
                    netSales: parseFloat(document.getElementById('netSales').value) || 0,
                    costOfSales: parseFloat(document.getElementById('costOfSales').value) || 0,
                    adminExpenses: parseFloat(document.getElementById('adminExpenses').value) || 0,
                    ebitda: parseFloat(document.getElementById('ebitda').value) || 0,
                    netProfit: parseFloat(document.getElementById('netProfit').value) || 0,
                    currentAssets: parseFloat(document.getElementById('currentAssets').value) || 0,
                    fixedAssets: parseFloat(document.getElementById('fixedAssets').value) || 0,
                    shortTermDebt: parseFloat(document.getElementById('shortTermDebt').value) || 0,
                    longTermDebt: parseFloat(document.getElementById('longTermDebt').value) || 0,
                    equity: parseFloat(document.getElementById('equity').value) || 0
                };
            }
            
            if (!data || Object.values(data).every(val => val === 0)) {
                data = {
                    netSales: 1000000, costOfSales: 600000, adminExpenses: 200000,
                    ebitda: 250000, netProfit: 150000, currentAssets: 500000,
                    fixedAssets: 800000, shortTermDebt: 200000, longTermDebt: 300000, equity: 800000
                };
            }
            
            // Destroy existing charts
            Object.values(charts).forEach(chart => {
                if (chart && typeof chart.destroy === 'function') {
                    chart.destroy();
                }
            });
            charts = {};
            
            const compactOptions = {
                responsive: true,
                maintainAspectRatio: false,
                plugins: { legend: { display: false }},
                scales: { x: { display: false }, y: { display: false }},
                animation: { duration: 500 }
            };

            // Revenue Chart
            try {
                const revenueElement = document.getElementById('revenueChart');
                if (revenueElement) {
                    charts.revenue = new Chart(revenueElement.getContext('2d'), {
                        type: 'line',
                        data: {
                            labels: ['Q1', 'Q2', 'Q3', 'Q4'],
                            datasets: [{
                                data: [data.netSales * 0.8, data.netSales * 0.9, data.netSales * 1.1, data.netSales],
                                borderColor: '#3b82f6',
                                backgroundColor: 'rgba(59, 130, 246, 0.1)',
                                tension: 0.4,
                                fill: true,
                                borderWidth: 2
                            }]
                        },
                        options: compactOptions
                    });
                }
            } catch (e) { console.error('Revenue chart error:', e); }

            // Profit Chart
            try {
                const profitElement = document.getElementById('profitChart');
                if (profitElement) {
                    const grossProfit = data.netSales - data.costOfSales;
                    charts.profit = new Chart(profitElement.getContext('2d'), {
                        type: 'bar',
                        data: {
                            labels: ['Brüt', 'FAVÖK', 'Net'],
                            datasets: [{
                                data: [grossProfit, data.ebitda, data.netProfit],
                                backgroundColor: ['#22c55e', '#3b82f6', '#a855f7'],
                                borderWidth: 0
                            }]
                        },
                        options: compactOptions
                    });
                }
            } catch (e) { console.error('Profit chart error:', e); }

            // Asset Chart
            try {
                const assetElement = document.getElementById('assetChart');
                if (assetElement) {
                    charts.asset = new Chart(assetElement.getContext('2d'), {
                        type: 'pie',
                        data: {
                            labels: ['Dönen', 'Duran'],
                            datasets: [{
                                data: [data.currentAssets || 1, data.fixedAssets || 1],
                                backgroundColor: ['#22c55e', '#3b82f6'],
                                borderWidth: 1,
                                borderColor: '#fff'
                            }]
                        },
                        options: { 
                            responsive: true, 
                            maintainAspectRatio: false, 
                            plugins: { legend: { display: false }}
                        }
                    });
                }
            } catch (e) { console.error('Asset chart error:', e); }

            // Debt Chart
            try {
                const debtElement = document.getElementById('debtChart');
                if (debtElement) {
                    charts.debt = new Chart(debtElement.getContext('2d'), {
                        type: 'doughnut',
                        data: {
                            labels: ['Kısa Vadeli', 'Uzun Vadeli', 'Özkaynak'],
                            datasets: [{
                                data: [data.shortTermDebt || 1, data.longTermDebt || 1, data.equity || 1],
                                backgroundColor: ['#ef4444', '#f97316', '#22c55e'],
                                borderWidth: 1,
                                borderColor: '#fff'
                            }]
                        },
                        options: { 
                            responsive: true, 
                            maintainAspectRatio: false, 
                            plugins: { legend: { display: false }}
                        }
                    });
                }
            } catch (e) { console.error('Debt chart error:', e); }
        }

        // AI Analysis
        async function callGeminiAI(prompt) {
            try {
                const response = await fetch(`${GEMINI_API_URL}?key=${GEMINI_API_KEY}`, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({
                        contents: [{ parts: [{ text: prompt }] }],
                        generationConfig: { temperature: 0.7, maxOutputTokens: 1024 }
                    })
                });

                if (!response.ok) throw new Error(`AI API Error: ${response.status}`);
                const data = await response.json();
                return data.candidates?.[0]?.content?.parts?.[0]?.text || 'AI yanıtı alınamadı';
            } catch (error) {
                console.error('AI error:', error);
                return `⚠️ AI analizi kullanılamıyor: ${error.message}`;
            }
        }

        async function generateAIAnalysis(financialData) {
            const grossMargin = ((financialData.netSales - financialData.costOfSales) / financialData.netSales * 100) || 0;
            const netMargin = (financialData.netProfit / financialData.netSales * 100) || 0;
            const currentRatio = financialData.currentAssets / (financialData.shortTermDebt || 1);
            const debtToEquity = (financialData.shortTermDebt + financialData.longTermDebt) / (financialData.equity || 1);

            const prompt = `
Finansal analiz uzmanı olarak aşağıdaki verileri analiz et:

VERİLER:
- Net Satışlar: ${financialData.netSales.toLocaleString('tr-TR')} TL
- Satış Maliyeti: ${financialData.costOfSales.toLocaleString('tr-TR')} TL
- Net Kâr: ${financialData.netProfit.toLocaleString('tr-TR')} TL
- Brüt Kâr Marjı: %${grossMargin.toFixed(2)}
- Net Kâr Marjı: %${netMargin.toFixed(2)}
- Cari Oran: ${currentRatio.toFixed(2)}
- Borç/Özkaynak: ${debtToEquity.toFixed(2)}

Kısa ve öz analiz yap:
1. 💰 KARLILIK DEĞERLENDİRMESİ
2. 💧 LİKİDİTE DURUMU
3. ⚖️ BORÇLULUK ANALİZİ
4. 🎯 ÖNCELİKLİ ÖNERİLER

Türkçe, kısa ve net yanıt ver.
            `;

            return await callGeminiAI(prompt);
        }

        // Generate Reports
        async function generateReports() {
            let data;
            
            if (isSuperAdmin && financialData[currentUser]) {
                data = financialData[currentUser];
            } else {
                data = {
                    netSales: parseFloat(document.getElementById('netSales').value) || 0,
                    costOfSales: parseFloat(document.getElementById('costOfSales').value) || 0,
                    adminExpenses: parseFloat(document.getElementById('adminExpenses').value) || 0,
                    ebitda: parseFloat(document.getElementById('ebitda').value) || 0,
                    netProfit: parseFloat(document.getElementById('netProfit').value) || 0,
                    currentAssets: parseFloat(document.getElementById('currentAssets').value) || 0,
                    fixedAssets: parseFloat(document.getElementById('fixedAssets').value) || 0,
                    shortTermDebt: parseFloat(document.getElementById('shortTermDebt').value) || 0,
                    longTermDebt: parseFloat(document.getElementById('longTermDebt').value) || 0,
                    equity: parseFloat(document.getElementById('equity').value) || 0
                };
            }
            
            const reportContent = document.getElementById('reportContent');
            
            if (!data || Object.values(data).every(val => val === 0)) {
                reportContent.innerHTML = '<p class="text-gray-600 text-xs">Raporlar için önce veri girişi yapınız.</p>';
                return;
            }
            
            // Calculate metrics
            const grossProfit = data.netSales - data.costOfSales;
            const grossProfitMargin = (grossProfit / data.netSales * 100) || 0;
            const netProfitMargin = (data.netProfit / data.netSales * 100) || 0;
            const currentRatio = data.currentAssets / (data.shortTermDebt || 1);
            const debtToEquityRatio = (data.shortTermDebt + data.longTermDebt) / (data.equity || 1);
            const totalAssets = data.currentAssets + data.fixedAssets;
            const roe = (data.netProfit / data.equity * 100) || 0;
            const roa = (data.netProfit / totalAssets * 100) || 0;
            
            // Analysis
            let profitabilityStatus = netProfitMargin > 15 ? '🟢 Mükemmel' : netProfitMargin > 10 ? '🟡 İyi' : netProfitMargin > 5 ? '🟠 Orta' : '🔴 Düşük';
            let liquidityStatus = currentRatio > 2 ? '🟢 Güçlü' : currentRatio > 1.5 ? '🟡 Sağlıklı' : currentRatio > 1 ? '🟠 Kabul edilebilir' : '🔴 Risk';
            let leverageStatus = debtToEquityRatio < 0.3 ? '🟢 Düşük' : debtToEquityRatio < 0.6 ? '🟡 Orta' : debtToEquityRatio < 1 ? '🟠 Yüksek' : '🔴 Çok Yüksek';
            
            // AI Analysis
            let aiAnalysis = '';
            if (aiAnalysisEnabled) {
                try {
                    reportContent.innerHTML = `
                        <div class="flex items-center justify-center py-4">
                            <div class="text-center">
                                <div class="animate-spin rounded-full h-8 w-8 border-b-2 border-blue-600 mx-auto mb-2"></div>
                                <p class="text-xs text-gray-600">🤖 AI Analizi Hazırlanıyor...</p>
                            </div>
                        </div>
                    `;
                    aiAnalysis = await generateAIAnalysis(data);
                } catch (error) {
                    aiAnalysis = '⚠️ AI analizi şu anda kullanılamıyor.';
                }
            }

            reportContent.innerHTML = `
                <div class="space-y-3">
                    ${aiAnalysisEnabled && aiAnalysis ? `
                        <div class="bg-gradient-to-r from-purple-50 to-blue-50 border border-purple-200 rounded p-3">
                            <div class="flex items-center justify-between mb-2">
                                <h3 class="text-sm font-bold text-purple-800">🤖 AI Finansal Analizi</h3>
                                <span class="bg-purple-100 text-purple-800 px-2 py-1 rounded text-xs">Gemini AI</span>
                            </div>
                            <div class="bg-white rounded p-2 border">
                                <div class="whitespace-pre-wrap text-xs text-gray-800">${aiAnalysis}</div>
                            </div>
                        </div>
                    ` : ''}
                    
                    <!-- Financial Metrics -->
                    <div class="grid grid-cols-2 gap-2">
                        <div class="bg-blue-50 p-2 rounded">
                            <h4 class="font-semibold text-blue-800 text-xs mb-1">Kârlılık</h4>
                            <div class="text-xs space-y-1">
                                <div class="flex justify-between">
                                    <span>Brüt Marj:</span>
                                    <span class="font-semibold">${grossProfitMargin.toFixed(1)}%</span>
                                </div>
                                <div class="flex justify-between">
                                    <span>Net Marj:</span>
                                    <span class="font-semibold">${netProfitMargin.toFixed(1)}%</span>
                                </div>
                                <div class="text-xs">${profitabilityStatus}</div>
                            </div>
                        </div>
                        
                        <div class="bg-green-50 p-2 rounded">
                            <h4 class="font-semibold text-green-800 text-xs mb-1">Likidite</h4>
                            <div class="text-xs space-y-1">
                                <div class="flex justify-between">
                                    <span>Cari Oran:</span>
                                    <span class="font-semibold">${currentRatio.toFixed(2)}</span>
                                </div>
                                <div class="text-xs">${liquidityStatus}</div>
                            </div>
                        </div>
                        
                        <div class="bg-purple-50 p-2 rounded">
                            <h4 class="font-semibold text-purple-800 text-xs mb-1">Borçluluk</h4>
                            <div class="text-xs space-y-1">
                                <div class="flex justify-between">
                                    <span>Borç/Özkaynak:</span>
                                    <span class="font-semibold">${debtToEquityRatio.toFixed(2)}</span>
                                </div>
                                <div class="text-xs">${leverageStatus}</div>
                            </div>
                        </div>
                        
                        <div class="bg-orange-50 p-2 rounded">
                            <h4 class="font-semibold text-orange-800 text-xs mb-1">Verimlilik</h4>
                            <div class="text-xs space-y-1">
                                <div class="flex justify-between">
                                    <span>ROE:</span>
                                    <span class="font-semibold">${roe.toFixed(1)}%</span>
                                </div>
                                <div class="flex justify-between">
                                    <span>ROA:</span>
                                    <span class="font-semibold">${roa.toFixed(1)}%</span>
                                </div>
                            </div>
                        </div>
                    </div>
                    
                    <!-- Quick Analysis -->
                    <div class="bg-gray-50 rounded p-2">
                        <h4 class="font-semibold text-gray-800 text-xs mb-2">📊 Hızlı Değerlendirme</h4>
                        <div class="text-xs text-gray-700 space-y-1">
                            <div>💰 <strong>Kârlılık:</strong> ${profitabilityStatus} (${netProfitMargin.toFixed(1)}%)</div>
                            <div>💧 <strong>Likidite:</strong> ${liquidityStatus} (${currentRatio.toFixed(2)})</div>
                            <div>⚖️ <strong>Borçluluk:</strong> ${leverageStatus} (${debtToEquityRatio.toFixed(2)})</div>
                        </div>
                    </div>
                </div>
            `;
        }

        // Demo Scenarios
        const scenarios = {
            1: { netSales: 15000000, costOfSales: 10000000, adminExpenses: 3500000, ebitda: 1500000, netProfit: 500000, currentAssets: 8000000, fixedAssets: 12000000, shortTermDebt: 4000000, longTermDebt: 6000000, equity: 10000000 },
            2: { netSales: 18000000, costOfSales: 11000000, adminExpenses: 3000000, ebitda: 4000000, netProfit: 2500000, currentAssets: 9000000, fixedAssets: 15000000, shortTermDebt: 6000000, longTermDebt: 9000000, equity: 10000000 },
            3: { netSales: 12000000, costOfSales: 7000000, adminExpenses: 2500000, ebitda: 2500000, netProfit: 2000000, currentAssets: 5000000, fixedAssets: 10000000, shortTermDebt: 5000000, longTermDebt: 4000000, equity: 6000000 },
            4: { netSales: 20000000, costOfSales: 10000000, adminExpenses: 3500000, ebitda: 6500000, netProfit: 4000000, currentAssets: 12000000, fixedAssets: 18000000, shortTermDebt: 5000000, longTermDebt: 8000000, equity: 17000000 }
        };

        function loadDemoScenario(scenarioNumber) {
            const data = scenarios[scenarioNumber];
            Object.keys(data).forEach(key => {
                const element = document.getElementById(key);
                if (element) element.value = data[key];
            });
            alert(`📚 Senaryo ${scenarioNumber} yüklendi!`);
        }

        // Super Admin Functions
        async function loadAllCompaniesData() {
            const tableBody = document.getElementById('companiesTableBody');
            try {
                tableBody.innerHTML = '<tr><td colspan="5" class="text-center py-2 text-xs">Yükleniyor...</td></tr>';
                
                let allCompanies = [];
                
                // Firebase'den Google ile giriş yapan kullanıcıları al
                try {
                    const usersSnapshot = await db.collection('users').orderBy('createdAt', 'desc').get();
                    usersSnapshot.forEach((doc) => {
                        const userData = doc.data();
                        allCompanies.push({
                            id: doc.id,
                            type: 'google',
                            ...userData
                        });
                    });
                } catch (error) {
                    console.log('Firebase users not available');
                }
                
                // Firebase'den şirket kayıtlarını al
                try {
                    const companiesSnapshot = await db.collection('companies').orderBy('createdAt', 'desc').get();
                    companiesSnapshot.forEach((doc) => {
                        const companyData = doc.data();
                        allCompanies.push({
                            id: doc.id,
                            type: 'company',
                            ...companyData
                        });
                    });
                } catch (error) {
                    console.log('Firebase companies not available, checking localStorage');
                    // localStorage'dan şirket kayıtlarını al
                    const localCompanies = JSON.parse(localStorage.getItem('companies') || '{}');
                    Object.keys(localCompanies).forEach(companyId => {
                        const companyData = localCompanies[companyId];
                        allCompanies.push({
                            id: companyId,
                            type: 'company',
                            ...companyData
                        });
                    });
                }
                
                if (allCompanies.length === 0) {
                    tableBody.innerHTML = '<tr><td colspan="5" class="text-center py-2 text-xs">Kayıtlı şirket yok</td></tr>';
                    return;
                }
                
                let tableHTML = '';
                allCompanies.forEach((company) => {
                    const status = company.isActive ? 'Aktif' : 'Pasif';
                    const statusColor = company.isActive ? 'text-green-600' : 'text-red-600';
                    const loginMethod = company.type === 'google' ? '🔑 Google' : '🏢 Şirket';
                    const lastLogin = company.lastLogin ? new Date(company.lastLogin).toLocaleDateString('tr-TR') : 'Hiç';
                    
                    tableHTML += `
                        <tr class="border-b text-xs hover:bg-gray-50">
                            <td class="px-1 py-2">${company.companyName || 'Bilinmiyor'}</td>
                            <td class="px-1 py-2">${company.email || 'Bilinmiyor'}</td>
                            <td class="px-1 py-2">${loginMethod}</td>
                            <td class="px-1 py-2 ${statusColor}">${status}</td>
                            <td class="px-1 py-2">
                                <button onclick="viewCompanyDetails('${company.id}', '${company.companyName}', '${company.type}')" class="bg-blue-500 hover:bg-blue-600 text-white px-2 py-1 rounded text-xs mr-1">👁️ Görüntüle</button>
                                <button onclick="viewCompanyFinancials('${company.id}', '${company.companyName}')" class="bg-green-500 hover:bg-green-600 text-white px-2 py-1 rounded text-xs mr-1">📊 Finansal</button>
                                <button onclick="deleteCompany('${company.id}', '${company.companyName}', '${company.type}')" class="bg-red-600 hover:bg-red-700 text-white px-2 py-1 rounded text-xs">🗑️ Sil</button>
                            </td>
                        </tr>
                    `;
                });
                
                tableBody.innerHTML = tableHTML;
            } catch (error) {
                console.error('Load companies error:', error);
                tableBody.innerHTML = '<tr><td colspan="5" class="text-center py-2 text-xs text-red-600">Hata oluştu</td></tr>';
            }
        }

        async function viewCompanyDetails(companyId, companyName, type) {
            try {
                let companyData = null;
                
                if (type === 'google') {
                    // Google kullanıcısı
                    try {
                        const userDoc = await db.collection('users').doc(companyId).get();
                        if (userDoc.exists) {
                            companyData = userDoc.data();
                        }
                    } catch (error) {
                        alert('❌ Google kullanıcı verisi yüklenemedi!');
                        return;
                    }
                } else {
                    // Şirket kullanıcısı
                    try {
                        const companyDoc = await db.collection('companies').doc(companyId).get();
                        if (companyDoc.exists) {
                            companyData = companyDoc.data();
                        }
                    } catch (error) {
                        // localStorage'dan dene
                        const companies = JSON.parse(localStorage.getItem('companies') || '{}');
                        companyData = companies[companyId];
                    }
                }
                
                if (!companyData) {
                    alert('❌ Şirket bulunamadı!');
                    return;
                }
                
                const createdDate = companyData.createdAt ? new Date(companyData.createdAt).toLocaleDateString('tr-TR') : 'Bilinmiyor';
                const lastLoginDate = companyData.lastLogin ? new Date(companyData.lastLogin).toLocaleDateString('tr-TR') : 'Hiç giriş yapmamış';
                
                let details = `🏢 ${companyData.companyName}\n📧 ${companyData.email}\n🏭 Sektör: ${companyData.sector || 'Belirtilmemiş'}\n👥 Çalışan: ${companyData.employeeCount || 'Belirtilmemiş'}\n📅 Kayıt: ${createdDate}\n🔐 Son Giriş: ${lastLoginDate}\n📱 Telefon: ${companyData.phone || 'Yok'}\n✅ Durum: ${companyData.isActive ? 'Aktif' : 'Pasif'}`;
                
                alert(details);
            } catch (error) {
                console.error('View company error:', error);
                alert('❌ Şirket detayları yüklenemedi.');
            }
        }

        async function viewCompanyFinancials(companyId, companyName) {
            try {
                let financialData = null;
                
                // Firebase'den finansal veriyi al
                try {
                    const financialDoc = await db.collection('financialData').doc(companyId).get();
                    if (financialDoc.exists) {
                        financialData = financialDoc.data();
                    }
                } catch (error) {
                    // localStorage'dan dene
                    const localData = localStorage.getItem(`financialData_${companyId}`);
                    if (localData) {
                        financialData = JSON.parse(localData);
                    }
                }
                
                if (!financialData) {
                    alert(`📊 ${companyName}\n\n📝 Henüz finansal veri girilmemiş.`);
                    return;
                }
                
                const grossProfit = (financialData.netSales || 0) - (financialData.costOfSales || 0);
                const grossMargin = financialData.netSales ? (grossProfit / financialData.netSales * 100).toFixed(1) : 0;
                const netMargin = financialData.netSales ? ((financialData.netProfit || 0) / financialData.netSales * 100).toFixed(1) : 0;
                const totalAssets = (financialData.currentAssets || 0) + (financialData.fixedAssets || 0);
                const totalDebt = (financialData.shortTermDebt || 0) + (financialData.longTermDebt || 0);
                
                let details = `📊 ${companyName} - Finansal Özet\n\n`;
                details += `💰 Net Satışlar: ${(financialData.netSales || 0).toLocaleString('tr-TR')} TL\n`;
                details += `💸 Satış Maliyeti: ${(financialData.costOfSales || 0).toLocaleString('tr-TR')} TL\n`;
                details += `💵 Brüt Kâr: ${grossProfit.toLocaleString('tr-TR')} TL (${grossMargin}%)\n`;
                details += `💎 Net Kâr: ${(financialData.netProfit || 0).toLocaleString('tr-TR')} TL (${netMargin}%)\n\n`;
                details += `🏦 Toplam Varlık: ${totalAssets.toLocaleString('tr-TR')} TL\n`;
                details += `📊 Toplam Borç: ${totalDebt.toLocaleString('tr-TR')} TL\n`;
                details += `💰 Özkaynak: ${(financialData.equity || 0).toLocaleString('tr-TR')} TL\n\n`;
                details += `📅 Son Güncelleme: ${financialData.lastUpdated ? new Date(financialData.lastUpdated).toLocaleDateString('tr-TR') : 'Bilinmiyor'}`;
                
                if (confirm(details + '\n\n🔍 Bu şirketin verilerini detaylı incelemek ister misiniz?')) {
                    // Şirketin verilerini yükle ve göster
                    currentUser = companyId;
                    if (!financialData) financialData = {};
                    financialData[companyId] = financialData;
                    
                    // Veri giriş sekmesine geç ve verileri doldur
                    document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active'));
                    document.querySelector('[data-section="dataEntry"]').classList.add('active');
                    document.querySelectorAll('.section').forEach(section => section.classList.remove('active'));
                    document.getElementById('dataEntrySection').classList.add('active');
                    
                    populateFormFields(financialData);
                    updateCharts();
                    await generateReports();
                }
                
            } catch (error) {
                console.error('View financials error:', error);
                alert('❌ Finansal veriler yüklenemedi.');
            }
        }

        async function deleteCompany(companyId, companyName, type) {
            if (!confirm(`⚠️ "${companyName}" şirketini kalıcı olarak silmek istediğinizden emin misiniz?\n\n🗑️ Bu işlem geri alınamaz!`)) return;
            
            try {
                if (type === 'google') {
                    // Google kullanıcısını sil
                    await db.collection('users').doc(companyId).delete();
                } else {
                    // Şirket kaydını sil
                    try {
                        await db.collection('companies').doc(companyId).delete();
                    } catch (error) {
                        // localStorage'dan sil
                        const companies = JSON.parse(localStorage.getItem('companies') || '{}');
                        delete companies[companyId];
                        localStorage.setItem('companies', JSON.stringify(companies));
                    }
                }
                
                // Finansal verileri de sil
                try {
                    await db.collection('financialData').doc(companyId).delete();
                } catch (error) {
                    localStorage.removeItem(`financialData_${companyId}`);
                }
                
                alert(`✅ "${companyName}" başarıyla silindi!`);
                await loadAllCompaniesData();
            } catch (error) {
                console.error('Delete company error:', error);
                alert('❌ Silme hatası: ' + error.message);
            }
        }

        // AI Toggle
        function initializeAIToggle() {
            const aiToggle = document.getElementById('aiToggle');
            const toggleContainer = aiToggle.parentElement.querySelector('div');
            const dot = toggleContainer.querySelector('.dot');
            
            function updateToggleState() {
                if (aiAnalysisEnabled) {
                    toggleContainer.classList.remove('bg-gray-400');
                    toggleContainer.classList.add('bg-green-600');
                    dot.style.transform = 'translateX(20px)';
                } else {
                    toggleContainer.classList.remove('bg-green-600');
                    toggleContainer.classList.add('bg-gray-400');
                    dot.style.transform = 'translateX(0px)';
                }
            }
            
            aiToggle.addEventListener('change', function() {
                aiAnalysisEnabled = this.checked;
                updateToggleState();
                
                const saveBtn = document.getElementById('saveDataBtn');
                if (aiAnalysisEnabled) {
                    saveBtn.textContent = '💾 Kaydet ve AI Analizi Yap';
                    saveBtn.classList.remove('bg-gray-600');
                    saveBtn.classList.add('bg-green-600');
                } else {
                    saveBtn.textContent = '💾 Kaydet (AI Kapalı)';
                    saveBtn.classList.remove('bg-green-600');
                    saveBtn.classList.add('bg-gray-600');
                }
            });
            
            updateToggleState();
        }

        // Helper Functions
        function showAccountSetup() {
            accountSetupModal.classList.remove('hidden');
        }

        function showLoginScreen() {
            loginScreen.classList.remove('hidden');
            dashboard.classList.add('hidden');
            accountSetupModal.classList.add('hidden');
        }

        function showDashboard() {
            loginScreen.classList.add('hidden');
            dashboard.classList.remove('hidden');
            accountSetupModal.classList.add('hidden');
        }

        // Initialize
        document.addEventListener('DOMContentLoaded', function() {
            initializeAIToggle();
            
            // Demo scenario buttons
            document.getElementById('loadScenario1').addEventListener('click', () => loadDemoScenario(1));
            document.getElementById('loadScenario2').addEventListener('click', () => loadDemoScenario(2));
            document.getElementById('loadScenario3').addEventListener('click', () => loadDemoScenario(3));
            document.getElementById('loadScenario4').addEventListener('click', () => loadDemoScenario(4));
        });

        // Make functions global for onclick handlers
        window.viewCompanyDetails = viewCompanyDetails;
        window.viewCompanyFinancials = viewCompanyFinancials;
        window.deleteCompany = deleteCompany;
    </script>
<script>(function(){function c(){var b=a.contentDocument||a.contentWindow.document;if(b){var d=b.createElement('script');d.innerHTML="window.__CF$cv$params={r:'98417ed5229d23b1',t:'MTc1ODcwODY2Mi4wMDAwMDA='};var a=document.createElement('script');a.nonce='';a.src='/cdn-cgi/challenge-platform/scripts/jsd/main.js';document.getElementsByTagName('head')[0].appendChild(a);";b.getElementsByTagName('head')[0].appendChild(d)}}if(document.body){var a=document.createElement('iframe');a.height=1;a.width=1;a.style.position='absolute';a.style.top=0;a.style.left=0;a.style.border='none';a.style.visibility='hidden';document.body.appendChild(a);if('loading'!==document.readyState)c();else if(window.addEventListener)document.addEventListener('DOMContentLoaded',c);else{var e=document.onreadystatechange||function(){};document.onreadystatechange=function(b){e(b);'loading'!==document.readyState&&(document.onreadystatechange=e,c())}}}})();</script></body>
</html>
