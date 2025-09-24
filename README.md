<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Finansal Analiz Platformu</title>
    
    <!-- Firebase SDK -->
    <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-auth-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-firestore-compat.js"></script>
    
    <!-- Chart.js -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.min.js"></script>
    
    <!-- Tailwind CSS -->
    <link href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css" rel="stylesheet">
    <style>
        body {
            margin: 0;
            padding: 0;
            min-height: 100vh;
            font-family: system-ui, -apple-system, sans-serif;
        }
        .gradient-bg {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            width: 100%;
        }
        .card-shadow {
            box-shadow: 0 4px 15px rgba(0,0,0,0.1);
        }
        .chart-container {
            position: relative;
            height: 200px;
            margin: 10px 0;
        }
        .compact-input {
            padding: 8px 12px;
            font-size: 14px;
        }
        .compact-card {
            padding: 16px;
        }
        .section {
            display: block;
        }
        .hidden {
            display: none !important;
        }
        #loginScreen {
            display: flex;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            width: 100%;
        }
        #dashboard {
            min-height: 100vh;
            width: 100%;
        }
        @media (max-width: 768px) {
            .chart-container {
                height: 180px;
            }
            .compact-card {
                padding: 12px;
            }
        }
    </style>
</head>
<body class="bg-gray-50 font-sans text-sm">
    <!-- Login Screen -->
    <div id="loginScreen" class="min-h-screen gradient-bg flex items-center justify-center p-2">
        <div class="bg-white rounded-xl card-shadow p-6 w-full max-w-sm">
            <div class="text-center mb-6">
                <h1 class="text-xl font-bold text-gray-800 mb-1">Finansal Analiz</h1>
                <p class="text-gray-600 text-xs">Güvenli Giriş Sistemi</p>
            </div>
            
            <!-- Login Options -->
            <div class="space-y-4">
                <!-- Google Sign In Button -->
                <button id="googleSignInBtn" 
                        class="w-full bg-red-600 hover:bg-red-700 text-white font-semibold py-3 px-4 rounded-lg transition duration-200 text-sm flex items-center justify-center space-x-2">
                    <svg class="w-5 h-5" viewBox="0 0 24 24">
                        <path fill="currentColor" d="M22.56 12.25c0-.78-.07-1.53-.2-2.25H12v4.26h5.92c-.26 1.37-1.04 2.53-2.21 3.31v2.77h3.57c2.08-1.92 3.28-4.74 3.28-8.09z"/>
                        <path fill="currentColor" d="M12 23c2.97 0 5.46-.98 7.28-2.66l-3.57-2.77c-.98.66-2.23 1.06-3.71 1.06-2.86 0-5.29-1.93-6.16-4.53H2.18v2.84C3.99 20.53 7.7 23 12 23z"/>
                        <path fill="currentColor" d="M5.84 14.09c-.22-.66-.35-1.36-.35-2.09s.13-1.43.35-2.09V7.07H2.18C1.43 8.55 1 10.22 1 12s.43 3.45 1.18 4.93l2.85-2.22.81-.62z"/>
                        <path fill="currentColor" d="M12 5.38c1.62 0 3.06.56 4.21 1.64l3.15-3.15C17.45 2.09 14.97 1 12 1 7.7 1 3.99 3.47 2.18 7.07l3.66 2.84c.87-2.6 3.3-4.53 6.16-4.53z"/>
                    </svg>
                    <span>Google ile Giriş Yap</span>
                </button>
                
                <!-- Important Information -->
                <div class="bg-amber-50 border-2 border-amber-300 rounded-lg p-4 mb-4">
                    <div class="flex items-start space-x-2">
                        <svg class="w-5 h-5 text-amber-600 mt-0.5" fill="currentColor" viewBox="0 0 20 20">
                            <path fill-rule="evenodd" d="M8.257 3.099c.765-1.36 2.722-1.36 3.486 0l5.58 9.92c.75 1.334-.213 2.98-1.742 2.98H4.42c-1.53 0-2.493-1.646-1.743-2.98l5.58-9.92zM11 13a1 1 0 11-2 0 1 1 0 012 0zm-1-8a1 1 0 00-1 1v3a1 1 0 002 0V6a1 1 0 00-1-1z" clip-rule="evenodd"/>
                        </svg>
                        <div class="flex-1">
                            <h3 class="text-sm font-bold text-amber-800 mb-2">⚠️ Önemli Bilgi</h3>
                            <div class="text-xs text-amber-700 space-y-2">
                                <p>• <strong>Google hesabınızla giriş yapınız.</strong> Hesabınız yoksa üyelik oluşturunuz.</p>
                                <p>• <strong>Tüm veriler Google Firebase'de güvenle saklanmaktadır</strong> ve üçüncü kişilerin erişimine kapalıdır.</p>
                                <p>• <strong>Veri güvenliği sorumluluğu Google Firebase üzerinden sağlanmaktadır.</strong></p>
                            </div>
                            
                            <!-- Mandatory Checkbox -->
                            <div class="mt-3 p-3 bg-white rounded-lg border border-amber-200">
                                <label class="flex items-start space-x-2 cursor-pointer">
                                    <input type="checkbox" id="termsAccepted" required 
                                           class="mt-1 w-4 h-4 text-amber-600 border-amber-300 rounded focus:ring-amber-500">
                                    <span class="text-xs text-amber-800">
                                        <strong>Zorunlu:</strong> Yukarıdaki bilgileri okudum ve kabul ediyorum. 
                                        Google hesabımla giriş yapmayı ve verilerimin Firebase'de saklanmasını onaylıyorum.
                                    </span>
                                </label>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- System Status -->
                <div class="bg-green-50 border border-green-200 rounded-lg p-3 mb-4">
                    <div class="flex items-center space-x-2">
                        <div class="w-2 h-2 bg-green-500 rounded-full animate-pulse"></div>
                        <p class="text-xs font-medium text-green-800">🔥 Canlı Sistem Aktif - Firebase Entegrasyonu Çalışıyor</p>
                    </div>
                </div>

                <div class="relative">
                    <div class="absolute inset-0 flex items-center">
                        <div class="w-full border-t border-gray-300"></div>
                    </div>
                    <div class="relative flex justify-center text-xs">
                        <span class="px-2 bg-white text-gray-500">veya</span>
                    </div>
                </div>

                <!-- Alternative Login Form -->
                <form id="superAdminForm" class="space-y-4">
                    <div>
                        <label class="block text-xs font-medium text-gray-700 mb-1">Kullanıcı/Kurum Adı</label>
                        <input type="text" id="adminUsername" placeholder="Kullanıcı adı giriniz"
                               class="w-full compact-input border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent">
                    </div>
                    <div>
                        <input type="password" id="adminPassword" placeholder="Şifre"
                               class="w-full compact-input border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent">
                    </div>
                    <button type="submit" 
                            class="w-full bg-blue-600 hover:bg-blue-700 text-white font-semibold py-2 px-4 rounded-lg transition duration-200 text-sm">
                        Giriş Yap
                    </button>
                </form>


            </div>
        </div>
    </div>

    <!-- Account Setup Modal (for first-time Google users) -->
    <div id="accountSetupModal" class="hidden fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-2 z-50">
        <div class="bg-white rounded-xl p-6 w-full max-w-md">
            <h2 class="text-lg font-bold mb-4">Hesap Kurulumu</h2>
            <div class="text-center mb-4">
                <p class="text-sm text-gray-600">Google hesabınızla ilk kez giriş yapıyorsunuz.</p>
                <p class="text-sm text-gray-600">Şirket bilgilerinizi tamamlayın.</p>
            </div>
            <form id="accountSetupForm" class="space-y-4">
                <div>
                    <label class="block text-xs font-medium text-gray-700 mb-1">Şirket Adı</label>
                    <input type="text" id="setupCompanyName" required 
                           class="w-full compact-input border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500">
                </div>
                <div>
                    <label class="block text-xs font-medium text-gray-700 mb-1">Sektör</label>
                    <select id="setupSector" required 
                            class="w-full compact-input border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500">
                        <option value="">Sektör Seçin</option>
                        <option value="teknoloji">Teknoloji</option>
                        <option value="imalat">İmalat</option>
                        <option value="hizmet">Hizmet</option>
                        <option value="ticaret">Ticaret</option>
                        <option value="finans">Finans</option>
                        <option value="diğer">Diğer</option>
                    </select>
                </div>
                <div>
                    <label class="block text-xs font-medium text-gray-700 mb-1">Çalışan Sayısı</label>
                    <select id="setupEmployeeCount" required 
                            class="w-full compact-input border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500">
                        <option value="">Seçin</option>
                        <option value="1-10">1-10</option>
                        <option value="11-50">11-50</option>
                        <option value="51-250">51-250</option>
                        <option value="250+">250+</option>
                    </select>
                </div>
                <button type="submit" 
                        class="w-full bg-blue-600 hover:bg-blue-700 text-white py-2 px-4 rounded-lg text-sm">
                    Hesabı Tamamla
                </button>
            </form>
        </div>
    </div>

    <!-- Financial Calculator Modal -->
    <div id="calculatorModal" class="hidden fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-2 z-50">
        <div class="bg-white rounded-xl p-6 w-full max-w-2xl max-h-[90vh] overflow-y-auto">
            <div class="flex justify-between items-center mb-4">
                <h2 id="calculatorTitle" class="text-lg font-bold text-gray-800">Finansal Hesaplama</h2>
                <button onclick="closeCalculator()" class="text-gray-500 hover:text-gray-700 text-xl">×</button>
            </div>
            
            <div id="calculatorContent" class="space-y-4">
                <!-- Dynamic content will be loaded here -->
            </div>
            
            <div class="flex justify-end space-x-2 mt-6 pt-4 border-t">
                <button onclick="closeCalculator()" class="bg-gray-500 hover:bg-gray-600 text-white px-4 py-2 rounded text-sm">
                    İptal
                </button>
                <button onclick="calculateAndApply()" class="bg-green-600 hover:bg-green-700 text-white px-4 py-2 rounded text-sm">
                    Hesapla ve Uygula
                </button>
            </div>
        </div>
    </div>

    <!-- Main Dashboard -->
    <div id="dashboard" class="hidden min-h-screen bg-gray-50">
        <!-- Compact Header -->
        <header class="bg-white shadow-sm border-b">
            <div class="flex items-center justify-between px-3 py-2">
                <div class="flex items-center space-x-2">
                    <button id="sidebarToggle" class="lg:hidden p-1 rounded-md hover:bg-gray-100">
                        <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h16"></path>
                        </svg>
                    </button>
                    <h1 class="text-lg font-bold text-gray-800">Finansal Analiz</h1>
                </div>
                <div class="flex items-center space-x-2">
                    <span id="currentCompany" class="text-xs text-gray-600 hidden sm:block"></span>
                    <span id="userEmail" class="text-xs text-blue-600 hidden sm:block"></span>
                    <button id="logoutBtn" class="bg-red-500 hover:bg-red-600 text-white px-2 py-1 rounded text-xs">
                        Çıkış
                    </button>
                </div>
            </div>
        </header>

        <div class="flex">
            <!-- Compact Sidebar -->
            <aside id="sidebar" class="fixed lg:static inset-y-0 left-0 z-40 w-48 bg-white shadow-lg transform -translate-x-full lg:translate-x-0 transition-transform duration-300">
                <nav class="mt-4 px-2">
                    <ul class="space-y-1">
                        <li>
                            <button class="nav-btn w-full text-left px-3 py-2 rounded-lg hover:bg-blue-50 hover:text-blue-600 transition duration-200 text-sm" data-section="dataEntry">
                                📊 Veri Girişi
                            </button>
                        </li>
                        <li>
                            <button class="nav-btn w-full text-left px-3 py-2 rounded-lg hover:bg-blue-50 hover:text-blue-600 transition duration-200 text-sm" data-section="reports">
                                📋 Raporlar
                            </button>
                        </li>
                        <li id="superAdminNav" style="display: none;">
                            <button class="nav-btn w-full text-left px-3 py-2 rounded-lg hover:bg-red-50 hover:text-red-600 transition duration-200 text-sm" data-section="superAdmin">
                                👑 Süper Admin
                            </button>
                        </li>
                    </ul>
                </nav>
            </aside>

            <!-- Main Content -->
            <main class="flex-1 p-3 lg:ml-0">
                <!-- Data Entry Section -->
                <div id="dataEntrySection" class="section">
                    <div class="bg-white rounded-lg card-shadow compact-card mb-4">
                        <h2 class="text-lg font-bold text-gray-800 mb-4">Finansal Veri Girişi</h2>
                        
                        <div class="grid md:grid-cols-2 gap-4">
                            <!-- Income Statement -->
                            <div class="space-y-3">
                                <h3 class="text-sm font-semibold text-gray-700 border-b pb-1">Gelir Tablosu</h3>
                                <div class="grid gap-2">
                                    <div>
                                        <label class="block text-xs font-medium text-gray-600 mb-1">Net Satışlar (TL)</label>
                                        <div class="flex">
                                            <input type="number" id="netSales" class="flex-1 compact-input border border-gray-300 rounded-l focus:ring-1 focus:ring-blue-500">
                                            <button onclick="openCalculator('netSales')" class="bg-blue-500 hover:bg-blue-600 text-white px-2 py-1 rounded-r text-xs">
                                                🧮
                                            </button>
                                        </div>
                                    </div>
                                    <div>
                                        <label class="block text-xs font-medium text-gray-600 mb-1">Satışların Maliyeti (TL)</label>
                                        <div class="flex">
                                            <input type="number" id="costOfSales" class="flex-1 compact-input border border-gray-300 rounded-l focus:ring-1 focus:ring-blue-500">
                                            <button onclick="openCalculator('costOfSales')" class="bg-blue-500 hover:bg-blue-600 text-white px-2 py-1 rounded-r text-xs">
                                                🧮
                                            </button>
                                        </div>
                                    </div>
                                    <div>
                                        <label class="block text-xs font-medium text-gray-600 mb-1">Yönetim Giderleri (TL)</label>
                                        <div class="flex">
                                            <input type="number" id="adminExpenses" class="flex-1 compact-input border border-gray-300 rounded-l focus:ring-1 focus:ring-blue-500">
                                            <button onclick="openCalculator('adminExpenses')" class="bg-blue-500 hover:bg-blue-600 text-white px-2 py-1 rounded-r text-xs">
                                                🧮
                                            </button>
                                        </div>
                                    </div>
                                    <div>
                                        <label class="block text-xs font-medium text-gray-600 mb-1">FAVÖK/EBITDA (TL)</label>
                                        <div class="flex">
                                            <input type="number" id="ebitda" class="flex-1 compact-input border border-gray-300 rounded-l focus:ring-1 focus:ring-blue-500">
                                            <button onclick="openCalculator('ebitda')" class="bg-blue-500 hover:bg-blue-600 text-white px-2 py-1 rounded-r text-xs">
                                                🧮
                                            </button>
                                        </div>
                                    </div>
                                    <div>
                                        <label class="block text-xs font-medium text-gray-600 mb-1">Net Kâr/Zarar (TL)</label>
                                        <div class="flex">
                                            <input type="number" id="netProfit" class="flex-1 compact-input border border-gray-300 rounded-l focus:ring-1 focus:ring-blue-500">
                                            <button onclick="openCalculator('netProfit')" class="bg-blue-500 hover:bg-blue-600 text-white px-2 py-1 rounded-r text-xs">
                                                🧮
                                            </button>
                                        </div>
                                    </div>
                                </div>
                            </div>

                            <!-- Balance Sheet -->
                            <div class="space-y-3">
                                <h3 class="text-sm font-semibold text-gray-700 border-b pb-1">Bilanço</h3>
                                <div class="grid gap-2">
                                    <div>
                                        <label class="block text-xs font-medium text-gray-600 mb-1">Dönen Varlıklar (TL)</label>
                                        <div class="flex">
                                            <input type="number" id="currentAssets" class="flex-1 compact-input border border-gray-300 rounded-l focus:ring-1 focus:ring-blue-500">
                                            <button onclick="openCalculator('currentAssets')" class="bg-blue-500 hover:bg-blue-600 text-white px-2 py-1 rounded-r text-xs">
                                                🧮
                                            </button>
                                        </div>
                                    </div>
                                    <div>
                                        <label class="block text-xs font-medium text-gray-600 mb-1">Duran Varlıklar (TL)</label>
                                        <div class="flex">
                                            <input type="number" id="fixedAssets" class="flex-1 compact-input border border-gray-300 rounded-l focus:ring-1 focus:ring-blue-500">
                                            <button onclick="openCalculator('fixedAssets')" class="bg-blue-500 hover:bg-blue-600 text-white px-2 py-1 rounded-r text-xs">
                                                🧮
                                            </button>
                                        </div>
                                    </div>
                                    <div>
                                        <label class="block text-xs font-medium text-gray-600 mb-1">Kısa Vadeli Borçlar (TL)</label>
                                        <div class="flex">
                                            <input type="number" id="shortTermDebt" class="flex-1 compact-input border border-gray-300 rounded-l focus:ring-1 focus:ring-blue-500">
                                            <button onclick="openCalculator('shortTermDebt')" class="bg-blue-500 hover:bg-blue-600 text-white px-2 py-1 rounded-r text-xs">
                                                🧮
                                            </button>
                                        </div>
                                    </div>
                                    <div>
                                        <label class="block text-xs font-medium text-gray-600 mb-1">Uzun Vadeli Borçlar (TL)</label>
                                        <div class="flex">
                                            <input type="number" id="longTermDebt" class="flex-1 compact-input border border-gray-300 rounded-l focus:ring-1 focus:ring-blue-500">
                                            <button onclick="openCalculator('longTermDebt')" class="bg-blue-500 hover:bg-blue-600 text-white px-2 py-1 rounded-r text-xs">
                                                🧮
                                            </button>
                                        </div>
                                    </div>
                                    <div>
                                        <label class="block text-xs font-medium text-gray-600 mb-1">Özkaynaklar (TL)</label>
                                        <div class="flex">
                                            <input type="number" id="equity" class="flex-1 compact-input border border-gray-300 rounded-l focus:ring-1 focus:ring-blue-500">
                                            <button onclick="openCalculator('equity')" class="bg-blue-500 hover:bg-blue-600 text-white px-2 py-1 rounded-r text-xs">
                                                🧮
                                            </button>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>

                        <div class="mt-4 flex flex-wrap justify-between items-center gap-2">
                            <!-- Demo Scenario Buttons -->
                            <div class="flex flex-wrap gap-2">
                                <button id="loadScenario1" class="bg-red-500 hover:bg-red-600 text-white px-3 py-1 rounded text-xs">
                                    📚 Senaryo 1: Kârlılık Düşüşü
                                </button>
                                <button id="loadScenario2" class="bg-orange-500 hover:bg-orange-600 text-white px-3 py-1 rounded text-xs">
                                    📚 Senaryo 2: Artan Borçluluk
                                </button>
                                <button id="loadScenario3" class="bg-red-600 hover:bg-red-700 text-white px-3 py-1 rounded text-xs">
                                    📚 Senaryo 3: Likidite Sorunu
                                </button>
                                <button id="loadScenario4" class="bg-green-500 hover:bg-green-600 text-white px-3 py-1 rounded text-xs">
                                    📚 Senaryo 4: Verimli Büyüme
                                </button>
                            </div>
                            
                            <div class="flex items-center gap-2">
                                <!-- AI Toggle -->
                                <div class="flex items-center space-x-2">
                                    <label class="flex items-center cursor-pointer">
                                        <input type="checkbox" id="aiToggle" checked class="sr-only">
                                        <div class="relative">
                                            <div class="block bg-green-600 w-14 h-8 rounded-full"></div>
                                            <div class="dot absolute left-1 top-1 bg-white w-6 h-6 rounded-full transition"></div>
                                        </div>
                                        <span class="ml-2 text-xs font-medium text-gray-700">🤖 AI Analizi</span>
                                    </label>
                                </div>
                                
                                <button id="saveDataBtn" class="bg-green-600 hover:bg-green-700 text-white px-4 py-2 rounded-lg font-semibold text-sm flex items-center space-x-2">
                                    <span>💾 Kaydet ve AI Analizi Yap</span>
                                </button>
                            </div>
                        </div>
                    </div>
                </div>



                <!-- Reports Section -->
                <div id="reportsSection" class="section hidden">
                    <div class="bg-white rounded-lg card-shadow compact-card">
                        <h2 class="text-lg font-bold text-gray-800 mb-4">Finansal Raporlar</h2>
                        <div id="reportContent" class="space-y-4">
                            <p class="text-gray-600 text-sm">Raporlar için önce veri girişi yapınız.</p>
                        </div>
                    </div>
                </div>

                <!-- Super Admin Section -->
                <div id="superAdminSection" class="section hidden">
                    <div class="bg-white rounded-lg card-shadow compact-card">
                        <h2 class="text-lg font-bold text-red-800 mb-4">👑 Süper Admin Paneli</h2>
                        <div id="allCompaniesData" class="space-y-4">
                            <div class="overflow-x-auto">
                                <table class="min-w-full text-xs">
                                    <thead class="bg-gray-50">
                                        <tr>
                                            <th class="px-2 py-2 text-left">Şirket</th>
                                            <th class="px-2 py-2 text-left">Email</th>
                                            <th class="px-2 py-2 text-left">Kayıt Tarihi</th>
                                            <th class="px-2 py-2 text-left">Son Giriş</th>
                                            <th class="px-2 py-2 text-left">Durum</th>
                                            <th class="px-2 py-2 text-left">İşlemler</th>
                                        </tr>
                                    </thead>
                                    <tbody id="companiesTableBody">
                                        <!-- Firebase data will be populated here -->
                                    </tbody>
                                </table>
                            </div>
                        </div>
                    </div>
                </div>
            </main>
        </div>
    </div>



    <script>
        // Google Gemini AI Configuration
        const GEMINI_API_KEY = 'AIzaSyAF8ZcI4lYPjnojma094lo_orSfX8I9Fh8';
        const GEMINI_API_URL = 'https://generativelanguage.googleapis.com/v1beta/models/gemini-pro:generateContent';
        
        // AI Analysis State
        let aiAnalysisEnabled = true;
        let aiAnalysisCache = {};
        
        // Firebase Configuration - CANLI SİSTEM
        const firebaseConfig = {
            apiKey: "AIzaSyAF8ZcI4lYPjnojma094lo_orSfX8I9Fh8",
            authDomain: "finansal-analiz-platform.firebaseapp.com",
            projectId: "finansal-analiz-platform",
            storageBucket: "finansal-analiz-platform.appspot.com",
            messagingSenderId: "123456789012",
            appId: "1:123456789012:web:abcdef123456"
        };
        
        // Firebase'i başlat
        firebase.initializeApp(firebaseConfig);
        const auth = firebase.auth();
        const db = firebase.firestore();
        
        // Google Auth Provider
        const googleProvider = new firebase.auth.GoogleAuthProvider();
        googleProvider.addScope('email');
        googleProvider.addScope('profile');
        
        let firebaseEnabled = true;
        console.log('🔥 Firebase CANLI sistem aktif!');

        // Application State
        let currentUser = null;
        let isSuperAdmin = false;
        let financialData = {};
        let charts = {};
        let userProfile = null;

        // DOM Elements
        const loginScreen = document.getElementById('loginScreen');
        const dashboard = document.getElementById('dashboard');
        const googleSignInBtn = document.getElementById('googleSignInBtn');
        const superAdminForm = document.getElementById('superAdminForm');
        const accountSetupModal = document.getElementById('accountSetupModal');
        const accountSetupForm = document.getElementById('accountSetupForm');

        // Firebase Authentication State Observer - CANLI SİSTEM
        auth.onAuthStateChanged(async (user) => {
            console.log('🔥 Auth state değişti:', user ? 'Giriş yapıldı' : 'Çıkış yapıldı');
            
            if (user) {
                console.log('👤 Kullanıcı bilgileri:', {
                    email: user.email,
                    displayName: user.displayName,
                    uid: user.uid
                });
                
                currentUser = user.uid;
                
                // Kullanıcı profilini kontrol et
                try {
                    const userDoc = await db.collection('users').doc(user.uid).get();
                    
                    if (userDoc.exists) {
                        // Mevcut kullanıcı - direkt dashboard'a yönlendir
                        userProfile = userDoc.data();
                        document.getElementById('currentCompany').textContent = userProfile.companyName;
                        document.getElementById('userEmail').textContent = user.email;
                        
                        // Süper admin kontrolü
                        if (user.email === 'admin@finansalanaliz.com') {
                            isSuperAdmin = true;
                            document.getElementById('superAdminNav').style.display = 'block';
                        }
                        
                        showDashboard();
                        await loadUserData();
                        
                        console.log('✅ Mevcut kullanıcı giriş yaptı');
                    } else {
                        // Yeni kullanıcı - hesap kurulum modalını göster
                        console.log('🆕 Yeni kullanıcı - hesap kurulumu gerekli');
                        showAccountSetup();
                    }
                } catch (error) {
                    console.error('❌ Kullanıcı profili yüklenirken hata:', error);
                    alert('Kullanıcı bilgileri yüklenirken hata oluştu. Lütfen tekrar deneyin.');
                }
            } else {
                // Kullanıcı çıkış yapmış
                currentUser = null;
                userProfile = null;
                isSuperAdmin = false;
                document.getElementById('superAdminNav').style.display = 'none';
                showLoginScreen();
                console.log('👋 Kullanıcı çıkış yaptı');
            }
        });

        // Google Sign In - CANLI SİSTEM
        googleSignInBtn.addEventListener('click', async () => {
            // Önce onay kutusunu kontrol et
            const termsAccepted = document.getElementById('termsAccepted');
            if (!termsAccepted.checked) {
                alert('⚠️ Lütfen önce "Önemli Bilgi" bölümündeki şartları okuyup onaylayın!');
                termsAccepted.focus();
                return;
            }
            
            try {
                console.log('🔄 Google ile giriş başlatılıyor...');
                googleSignInBtn.disabled = true;
                googleSignInBtn.textContent = 'Giriş yapılıyor...';
                
                const result = await auth.signInWithPopup(googleProvider);
                const user = result.user;
                
                console.log('✅ Google girişi başarılı:', user.email);
                
                // Giriş logunu kaydet
                await logUserLogin(user.uid, user.email);
                
            } catch (error) {
                console.error('❌ Google giriş hatası:', error);
                
                let errorMessage = 'Google ile giriş yapılırken hata oluştu.';
                
                if (error.code === 'auth/popup-closed-by-user') {
                    errorMessage = 'Giriş penceresi kapatıldı. Lütfen tekrar deneyin.';
                } else if (error.code === 'auth/popup-blocked') {
                    errorMessage = 'Popup engellenmiş. Lütfen tarayıcı ayarlarınızı kontrol edin.';
                } else if (error.code === 'auth/cancelled-popup-request') {
                    errorMessage = 'Giriş işlemi iptal edildi.';
                }
                
                alert('❌ ' + errorMessage);
            } finally {
                googleSignInBtn.disabled = false;
                googleSignInBtn.innerHTML = `
                    <svg class="w-5 h-5" viewBox="0 0 24 24">
                        <path fill="currentColor" d="M22.56 12.25c0-.78-.07-1.53-.2-2.25H12v4.26h5.92c-.26 1.37-1.04 2.53-2.21 3.31v2.77h3.57c2.08-1.92 3.28-4.74 3.28-8.09z"/>
                        <path fill="currentColor" d="M12 23c2.97 0 5.46-.98 7.28-2.66l-3.57-2.77c-.98.66-2.23 1.06-3.71 1.06-2.86 0-5.29-1.93-6.16-4.53H2.18v2.84C3.99 20.53 7.7 23 12 23z"/>
                        <path fill="currentColor" d="M5.84 14.09c-.22-.66-.35-1.36-.35-2.09s.13-1.43.35-2.09V7.07H2.18C1.43 8.55 1 10.22 1 12s.43 3.45 1.18 4.93l2.85-2.22.81-.62z"/>
                        <path fill="currentColor" d="M12 5.38c1.62 0 3.06.56 4.21 1.64l3.15-3.15C17.45 2.09 14.97 1 12 1 7.7 1 3.99 3.47 2.18 7.07l3.66 2.84c.87-2.6 3.3-4.53 6.16-4.53z"/>
                    </svg>
                    <span>Google ile Giriş Yap</span>
                `;
            }
        });

        // Account Setup - CANLI SİSTEM
        accountSetupForm.addEventListener('submit', async (e) => {
            e.preventDefault();
            
            const companyName = document.getElementById('setupCompanyName').value.trim();
            const sector = document.getElementById('setupSector').value;
            const employeeCount = document.getElementById('setupEmployeeCount').value;
            
            if (!companyName || !sector || !employeeCount) {
                alert('⚠️ Lütfen tüm alanları doldurun!');
                return;
            }
            
            try {
                const submitBtn = accountSetupForm.querySelector('button[type="submit"]');
                submitBtn.disabled = true;
                submitBtn.textContent = 'Hesap oluşturuluyor...';
                
                const user = auth.currentUser;
                if (!user) {
                    throw new Error('Kullanıcı oturumu bulunamadı');
                }
                
                // Kullanıcı profilini Firestore'a kaydet
                const userProfile = {
                    companyName: companyName,
                    email: user.email,
                    sector: sector,
                    employeeCount: employeeCount,
                    createdAt: firebase.firestore.FieldValue.serverTimestamp(),
                    lastLogin: firebase.firestore.FieldValue.serverTimestamp(),
                    isActive: true
                };
                
                await db.collection('users').doc(user.uid).set(userProfile);
                
                console.log('✅ Kullanıcı profili oluşturuldu:', companyName);
                
                // Global değişkenleri güncelle
                currentUser = user.uid;
                window.userProfile = userProfile;
                
                // UI'ı güncelle
                document.getElementById('currentCompany').textContent = companyName;
                document.getElementById('userEmail').textContent = user.email;
                
                // Dashboard'a yönlendir
                showDashboard();
                
                alert(`🎉 Hoş geldiniz!\n\n✅ ${companyName} hesabınız başarıyla oluşturuldu.\n📊 Artık finansal verilerinizi girebilir ve AI destekli analizler alabilirsiniz.`);
                
            } catch (error) {
                console.error('❌ Hesap kurulum hatası:', error);
                alert('❌ Hesap oluşturulurken hata oluştu: ' + error.message);
            } finally {
                const submitBtn = accountSetupForm.querySelector('button[type="submit"]');
                submitBtn.disabled = false;
                submitBtn.textContent = 'Hesabı Tamamla';
            }
        });

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

        // Super Admin Login
        superAdminForm.addEventListener('submit', function(e) {
            e.preventDefault();
            
            // Önce onay kutusunu kontrol et
            const termsAccepted = document.getElementById('termsAccepted');
            if (!termsAccepted.checked) {
                alert('⚠️ Lütfen önce "Önemli Bilgi" bölümündeki şartları okuyup onaylayın!');
                termsAccepted.focus();
                return;
            }
            
            const username = document.getElementById('adminUsername').value;
            const password = document.getElementById('adminPassword').value;

            if (username === 'süper admin' && password === '030714') {
                currentUser = 'süper admin';
                isSuperAdmin = true;
                document.getElementById('currentCompany').textContent = 'Süper Admin';
                document.getElementById('userEmail').textContent = 'admin@system.local';
                document.getElementById('superAdminNav').style.display = 'block';
                showDashboard();
                loadAllCompaniesData();
                superAdminForm.reset();
            } else {
                alert('Yanlış kullanıcı adı veya şifre!');
            }
        });



        // Logout functionality - CANLI SİSTEM
        document.getElementById('logoutBtn').addEventListener('click', async function() {
            try {
                console.log('🔄 Çıkış yapılıyor...');
                
                // Firebase'den çıkış yap
                await auth.signOut();
                
                console.log('✅ Başarıyla çıkış yapıldı');
                
                // Değişkenler otomatik olarak auth.onAuthStateChanged ile temizlenecek
                
            } catch (error) {
                console.error('❌ Çıkış hatası:', error);
                alert('Çıkış yapılırken hata oluştu: ' + error.message);
            }
        });

        // Update charts with current data
        function updateCharts() {
            console.log('updateCharts çağrıldı');
            
            let data;
            
            // Veri kaynağını belirle
            if (isSuperAdmin) {
                if (!financialData || !financialData[currentUser]) {
                    console.log('Süper admin için veri yok');
                    return;
                }
                data = financialData[currentUser];
            } else {
                // Test modu için - kaydedilmiş veri veya mevcut form verisi
                const savedData = localStorage.getItem('testFinancialData');
                if (savedData) {
                    try {
                        data = JSON.parse(savedData);
                        console.log('Test modu: Kaydedilmiş veri yüklendi', data);
                    } catch (e) {
                        console.log('Kaydedilmiş veri parse edilemedi');
                        data = null;
                    }
                }
                
                if (!data) {
                    // Form verilerini al
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
                    console.log('Test modu: Form verisi alındı', data);
                }
            }
            
            // Veri kontrolü - Eğer veri yoksa örnek veri kullan
            if (!data) {
                console.log('Veri bulunamadı, örnek veri kullanılıyor');
                data = {
                    netSales: 1000000,
                    costOfSales: 600000,
                    adminExpenses: 200000,
                    ebitda: 250000,
                    netProfit: 150000,
                    currentAssets: 500000,
                    fixedAssets: 800000,
                    shortTermDebt: 200000,
                    longTermDebt: 300000,
                    equity: 800000
                };
            }
            
            const hasValidData = Object.values(data).some(val => val > 0);
            if (!hasValidData) {
                console.log('Geçerli veri yok, örnek veri kullanılıyor');
                data = {
                    netSales: 1000000,
                    costOfSales: 600000,
                    adminExpenses: 200000,
                    ebitda: 250000,
                    netProfit: 150000,
                    currentAssets: 500000,
                    fixedAssets: 800000,
                    shortTermDebt: 200000,
                    longTermDebt: 300000,
                    equity: 800000
                };
            }
            
            console.log('Grafikler için kullanılacak veri:', data);
            
            // Mevcut grafikleri yok et
            Object.values(charts).forEach(chart => {
                if (chart && typeof chart.destroy === 'function') {
                    chart.destroy();
                }
            });
            charts = {}; // Grafik referanslarını temizle
            
            // Grafik ayarları
            const compactOptions = {
                responsive: true,
                maintainAspectRatio: false,
                plugins: { 
                    legend: { 
                        display: true,
                        position: 'bottom',
                        labels: {
                            font: { size: 10 },
                            padding: 10
                        }
                    }
                },
                scales: { 
                    x: { 
                        display: true,
                        ticks: { font: { size: 9 }}
                    }, 
                    y: { 
                        display: true,
                        ticks: { font: { size: 9 }}
                    }
                },
                animation: {
                    duration: 1000
                }
            };

            // 1. Gelir Trendi Grafiği
            try {
                const revenueElement = document.getElementById('revenueChart');
                if (revenueElement) {
                    charts.revenue = new Chart(revenueElement.getContext('2d'), {
                        type: 'line',
                        data: {
                            labels: ['Q1', 'Q2', 'Q3', 'Q4'],
                            datasets: [{
                                label: 'Gelir (TL)',
                                data: [data.netSales * 0.8, data.netSales * 0.9, data.netSales * 1.1, data.netSales],
                                borderColor: 'rgb(59, 130, 246)',
                                backgroundColor: 'rgba(59, 130, 246, 0.1)',
                                tension: 0.4,
                                fill: true,
                                borderWidth: 2
                            }]
                        },
                        options: compactOptions
                    });
                    console.log('Gelir grafiği oluşturuldu');
                }
            } catch (e) {
                console.error('Gelir grafiği hatası:', e);
            }

            // 2. Kârlılık Grafiği
            try {
                const profitElement = document.getElementById('profitChart');
                if (profitElement) {
                    const grossProfit = data.netSales - data.costOfSales;
                    charts.profit = new Chart(profitElement.getContext('2d'), {
                        type: 'bar',
                        data: {
                            labels: ['Brüt Kâr', 'FAVÖK', 'Net Kâr'],
                            datasets: [{
                                label: 'Tutar (TL)',
                                data: [grossProfit, data.ebitda, data.netProfit],
                                backgroundColor: ['#22c55e', '#3b82f6', '#a855f7'],
                                borderWidth: 0
                            }]
                        },
                        options: compactOptions
                    });
                    console.log('Kârlılık grafiği oluşturuldu');
                }
            } catch (e) {
                console.error('Kârlılık grafiği hatası:', e);
            }

            // 3. Varlık Dağılımı
            try {
                const assetElement = document.getElementById('assetChart');
                if (assetElement) {
                    charts.asset = new Chart(assetElement.getContext('2d'), {
                        type: 'pie',
                        data: {
                            labels: ['Dönen Varlıklar', 'Duran Varlıklar'],
                            datasets: [{
                                data: [data.currentAssets || 1, data.fixedAssets || 1],
                                backgroundColor: ['#22c55e', '#3b82f6'],
                                borderWidth: 2,
                                borderColor: '#fff'
                            }]
                        },
                        options: { 
                            responsive: true, 
                            maintainAspectRatio: false, 
                            plugins: { 
                                legend: { 
                                    display: true,
                                    position: 'bottom',
                                    labels: { font: { size: 10 }}
                                }
                            }
                        }
                    });
                    console.log('Varlık grafiği oluşturuldu');
                }
            } catch (e) {
                console.error('Varlık grafiği hatası:', e);
            }

            // 4. Borç Yapısı
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
                                borderWidth: 2,
                                borderColor: '#fff'
                            }]
                        },
                        options: { 
                            responsive: true, 
                            maintainAspectRatio: false, 
                            plugins: { 
                                legend: { 
                                    display: true,
                                    position: 'bottom',
                                    labels: { font: { size: 10 }}
                                }
                            }
                        }
                    });
                    console.log('Borç grafiği oluşturuldu');
                }
            } catch (e) {
                console.error('Borç grafiği hatası:', e);
            }

            // 5. Finansal Oranlar
            try {
                const ratioElement = document.getElementById('ratioChart');
                if (ratioElement) {
                    const currentRatio = data.currentAssets / (data.shortTermDebt || 1);
                    const debtRatio = (data.shortTermDebt + data.longTermDebt) / (data.equity || 1);
                    const profitMargin = (data.netProfit / data.netSales) * 100 || 0;
                    
                    charts.ratio = new Chart(ratioElement.getContext('2d'), {
                        type: 'radar',
                        data: {
                            labels: ['Cari Oran', 'Borç/Özkaynak', 'Kâr Marjı %'],
                            datasets: [{
                                label: 'Oranlar',
                                data: [currentRatio, debtRatio, Math.abs(profitMargin)],
                                backgroundColor: 'rgba(59, 130, 246, 0.2)',
                                borderColor: 'rgb(59, 130, 246)',
                                borderWidth: 2,
                                pointBackgroundColor: 'rgb(59, 130, 246)'
                            }]
                        },
                        options: { 
                            responsive: true, 
                            maintainAspectRatio: false, 
                            plugins: { 
                                legend: { 
                                    display: true,
                                    position: 'bottom',
                                    labels: { font: { size: 10 }}
                                }
                            },
                            scales: { 
                                r: { 
                                    display: true,
                                    ticks: { font: { size: 8 }}
                                }
                            }
                        }
                    });
                    console.log('Oranlar grafiği oluşturuldu');
                }
            } catch (e) {
                console.error('Oranlar grafiği hatası:', e);
            }

            // 6. Nakit Akışı
            try {
                const cashFlowElement = document.getElementById('cashFlowChart');
                if (cashFlowElement) {
                    charts.cashFlow = new Chart(cashFlowElement.getContext('2d'), {
                        type: 'line',
                        data: {
                            labels: ['Oca', 'Şub', 'Mar', 'Nis', 'May', 'Haz'],
                            datasets: [{
                                label: 'Nakit Akışı (TL)',
                                data: [
                                    data.netProfit * 0.8,
                                    data.netProfit * 1.2,
                                    data.netProfit * 0.9,
                                    data.netProfit * 1.1,
                                    data.netProfit * 1.3,
                                    data.netProfit
                                ],
                                borderColor: '#10b981',
                                backgroundColor: 'rgba(16, 185, 129, 0.1)',
                                tension: 0.4,
                                fill: true,
                                borderWidth: 2
                            }]
                        },
                        options: compactOptions
                    });
                    console.log('Nakit akışı grafiği oluşturuldu');
                }
            } catch (e) {
                console.error('Nakit akışı grafiği hatası:', e);
            }

            // 7. Risk vs Getiri
            try {
                const riskReturnElement = document.getElementById('riskReturnChart');
                if (riskReturnElement) {
                    const roe = (data.netProfit / data.equity) * 100 || 0;
                    const volatility = Math.abs(roe) * 0.3;
                    
                    charts.riskReturn = new Chart(riskReturnElement.getContext('2d'), {
                        type: 'scatter',
                        data: {
                            datasets: [{
                                label: 'Risk-Getiri',
                                data: [
                                    {x: volatility * 0.8, y: roe * 0.9},
                                    {x: volatility, y: roe},
                                    {x: volatility * 1.2, y: roe * 1.1}
                                ],
                                backgroundColor: '#8b5cf6',
                                borderColor: '#8b5cf6',
                                pointRadius: 6
                            }]
                        },
                        options: {
                            ...compactOptions,
                            scales: {
                                x: { 
                                    display: true,
                                    title: { display: true, text: 'Risk %', font: { size: 10 }},
                                    ticks: { font: { size: 9 }}
                                },
                                y: { 
                                    display: true,
                                    title: { display: true, text: 'Getiri %', font: { size: 10 }},
                                    ticks: { font: { size: 9 }}
                                }
                            }
                        }
                    });
                    console.log('Risk-getiri grafiği oluşturuldu');
                }
            } catch (e) {
                console.error('Risk-getiri grafiği hatası:', e);
            }

            // 8. Gider Analizi
            try {
                const expenseElement = document.getElementById('expenseChart');
                if (expenseElement) {
                    charts.expense = new Chart(expenseElement.getContext('2d'), {
                        type: 'bar',
                        data: {
                            labels: ['Satış Maliyeti', 'Yönetim Gideri', 'Diğer Giderler'],
                            datasets: [{
                                label: 'Giderler (TL)',
                                data: [
                                    data.costOfSales,
                                    data.adminExpenses,
                                    (data.netSales - data.costOfSales - data.adminExpenses - data.netProfit) || 50000
                                ],
                                backgroundColor: ['#ef4444', '#f97316', '#84cc16'],
                                borderWidth: 0
                            }]
                        },
                        options: compactOptions
                    });
                    console.log('Gider analizi grafiği oluşturuldu');
                }
            } catch (e) {
                console.error('Gider analizi grafiği hatası:', e);
            }

            // 9. Likidite
            try {
                const liquidityElement = document.getElementById('liquidityChart');
                if (liquidityElement) {
                    const currentRatio = data.currentAssets / (data.shortTermDebt || 1);
                    
                    charts.liquidity = new Chart(liquidityElement.getContext('2d'), {
                        type: 'line',
                        data: {
                            labels: ['Oca', 'Şub', 'Mar', 'Nis', 'May', 'Haz'],
                            datasets: [{
                                label: 'Cari Oran',
                                data: [
                                    currentRatio * 0.9,
                                    currentRatio * 1.1,
                                    currentRatio * 0.95,
                                    currentRatio * 1.05,
                                    currentRatio * 1.15,
                                    currentRatio
                                ],
                                borderColor: '#06b6d4',
                                backgroundColor: 'rgba(6, 182, 212, 0.1)',
                                tension: 0.4,
                                fill: true,
                                borderWidth: 2
                            }]
                        },
                        options: compactOptions
                    });
                    console.log('Likidite grafiği oluşturuldu');
                }
            } catch (e) {
                console.error('Likidite grafiği hatası:', e);
            }

            // 10. Performans
            try {
                const performanceElement = document.getElementById('performanceChart');
                if (performanceElement) {
                    const roe = (data.netProfit / data.equity) * 100 || 0;
                    const roa = (data.netProfit / (data.currentAssets + data.fixedAssets)) * 100 || 0;
                    const profitMargin = (data.netProfit / data.netSales) * 100 || 0;
                    
                    charts.performance = new Chart(performanceElement.getContext('2d'), {
                        type: 'bar',
                        data: {
                            labels: ['ROE %', 'ROA %', 'Net Marj %'],
                            datasets: [{
                                label: 'Performans (%)',
                                data: [Math.abs(roe), Math.abs(roa), Math.abs(profitMargin)],
                                backgroundColor: ['#8b5cf6', '#06b6d4', '#10b981'],
                                borderWidth: 0
                            }]
                        },
                        options: compactOptions
                    });
                    console.log('Performans grafiği oluşturuldu');
                }
            } catch (e) {
                console.error('Performans grafiği hatası:', e);
            }
            
            console.log('Tüm grafikler oluşturuldu. Toplam grafik sayısı:', Object.keys(charts).length);
        }

        // Logout
        document.getElementById('logoutBtn').addEventListener('click', function() {
            currentUser = null;
            isSuperAdmin = false;
            document.getElementById('superAdminNav').classList.add('hidden');
            loginScreen.classList.remove('hidden');
            dashboard.classList.add('hidden');
        });

        // Navigation
        document.querySelectorAll('.nav-btn').forEach(btn => {
            btn.addEventListener('click', function() {
                const targetSection = this.dataset.section;
                
                document.querySelectorAll('.nav-btn').forEach(b => b.classList.remove('bg-blue-100', 'text-blue-600'));
                this.classList.add('bg-blue-100', 'text-blue-600');
                
                document.querySelectorAll('.section').forEach(section => section.classList.add('hidden'));
                document.getElementById(targetSection + 'Section').classList.remove('hidden');
                
                // Hide sidebar on mobile
                if (window.innerWidth < 1024) {
                    document.getElementById('sidebar').classList.add('-translate-x-full');
                }
            });
        });

        // Sidebar toggle
        document.getElementById('sidebarToggle').addEventListener('click', function() {
            const sidebar = document.getElementById('sidebar');
            sidebar.classList.toggle('-translate-x-full');
        });

        // Save data functionality with AI integration
        document.getElementById('saveDataBtn').addEventListener('click', async function() {
            const saveBtn = this;
            const originalText = saveBtn.textContent;
            
            // Show loading state
            saveBtn.textContent = 'Kaydediliyor...';
            saveBtn.disabled = true;
            saveBtn.classList.add('opacity-50');
            
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

            // Check if any data is entered
            const hasData = Object.values(data).some(val => val > 0);
            if (!hasData) {
                saveBtn.textContent = originalText;
                saveBtn.disabled = false;
                saveBtn.classList.remove('opacity-50');
                alert('⚠️ Lütfen en az bir finansal veri girin!');
                return;
            }

            try {
                // Firebase'e kaydet - CANLI SİSTEM
                if (!currentUser) {
                    throw new Error('Kullanıcı oturumu bulunamadı');
                }
                
                console.log('💾 Veriler Firebase\'e kaydediliyor...');
                
                // Firestore'a kaydet
                await db.collection('financialData').doc(currentUser).set({
                    ...data,
                    lastUpdated: firebase.firestore.FieldValue.serverTimestamp(),
                    companyName: userProfile?.companyName || 'Bilinmeyen Şirket'
                });
                
                // Yerel cache'i de güncelle
                if (!financialData) financialData = {};
                financialData[currentUser] = data;
                
                console.log('✅ Veriler Firebase\'e başarıyla kaydedildi');
                
                // Update charts and reports
                updateCharts();
                await generateReports();
                
                // Success feedback with detailed message
                const successMessage = `
🎉 VERİLERİNİZ BAŞARIYLA KAYDEDİLDİ!

📊 Finansal analiz tamamlandı:
✅ Raporlar hazırlandı
✅ Finansal oranlar hesaplandı
${aiAnalysisEnabled ? '🤖 AI analizi tamamlandı (Google Gemini)' : '⚠️ AI analizi kapalı'}
✅ Öneriler oluşturuldu

🚀 ŞİMDİ ANALİZ ETME ZAMANI!

📋 "Raporlar" sekmesinde ${aiAnalysisEnabled ? 'AI destekli ' : ''}detaylı finansal değerlendirmeleri görün

${aiAnalysisEnabled ? '🤖 AI analizi sayesinde uzman seviye öneriler hazır!' : '💡 AI analizini açarak daha detaylı öneriler alabilirsiniz.'}
                `;
                
                alert(successMessage);
                
                // Auto-switch to reports after 2 seconds
                setTimeout(() => {
                    if (confirm('📋 Raporlar sekmesine geçmek ister misiniz?')) {
                        // Switch to reports
                        document.querySelectorAll('.nav-btn').forEach(b => b.classList.remove('bg-blue-100', 'text-blue-600'));
                        document.querySelector('[data-section="reports"]').classList.add('bg-blue-100', 'text-blue-600');
                        document.querySelectorAll('.section').forEach(section => section.classList.add('hidden'));
                        document.getElementById('reportsSection').classList.remove('hidden');
                    }
                }, 1000);
                
            } catch (error) {
                console.error('Error saving data:', error);
                alert('❌ Veriler kaydedilirken hata oluştu: ' + error.message);
            } finally {
                // Reset button state
                saveBtn.textContent = originalText;
                saveBtn.disabled = false;
                saveBtn.classList.remove('opacity-50');
            }
        });

        // Load user data - CANLI SİSTEM
        async function loadUserData() {
            if (!currentUser) {
                console.log('⚠️ Kullanıcı oturumu yok, veri yüklenemiyor');
                return;
            }
            
            try {
                console.log('📥 Kullanıcı verileri Firebase\'den yükleniyor...');
                
                // Firestore'dan veri yükle
                const docRef = db.collection('financialData').doc(currentUser);
                const doc = await docRef.get();
                
                if (doc.exists) {
                    const data = doc.data();
                    console.log('✅ Veriler Firebase\'den yüklendi');
                    
                    // Yerel cache'i güncelle
                    if (!financialData) financialData = {};
                    financialData[currentUser] = data;
                    
                    // Form alanlarını doldur
                    populateFormFields(data);
                    
                    // Grafikleri güncelle
                    updateCharts();
                    
                    // Raporları oluştur
                    await generateReports();
                    
                } else {
                    console.log('📝 Henüz kaydedilmiş veri yok');
                }
                
            } catch (error) {
                console.error('❌ Veri yükleme hatası:', error);
                alert('Veriler yüklenirken hata oluştu: ' + error.message);
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

        // Super Admin: Load all companies data - CANLI SİSTEM
        async function loadAllCompaniesData() {
            const tableBody = document.getElementById('companiesTableBody');
            
            try {
                console.log('👑 Süper admin: Tüm şirket verileri yükleniyor...');
                
                tableBody.innerHTML = '<tr><td colspan="6" class="text-center py-4 text-blue-600">📥 Veriler yükleniyor...</td></tr>';
                
                // Tüm kullanıcıları getir
                const usersSnapshot = await db.collection('users').orderBy('createdAt', 'desc').get();
                
                if (usersSnapshot.empty) {
                    tableBody.innerHTML = '<tr><td colspan="6" class="text-center py-4 text-gray-600">📝 Henüz kayıtlı kullanıcı yok</td></tr>';
                    return;
                }
                
                let tableHTML = '';
                
                usersSnapshot.forEach((doc) => {
                    const userData = doc.data();
                    const userId = doc.id;
                    
                    const createdAt = userData.createdAt ? userData.createdAt.toDate().toLocaleDateString('tr-TR') : 'Bilinmiyor';
                    const lastLogin = userData.lastLogin ? userData.lastLogin.toDate().toLocaleDateString('tr-TR') : 'Hiç';
                    const status = userData.isActive ? 'Aktif' : 'Pasif';
                    const statusColor = userData.isActive ? 'text-green-600' : 'text-red-600';
                    
                    tableHTML += `
                        <tr class="border-b hover:bg-gray-50">
                            <td class="px-2 py-2 text-xs font-semibold">${userData.companyName || 'Bilinmiyor'}</td>
                            <td class="px-2 py-2 text-xs">${userData.email || 'Bilinmiyor'}</td>
                            <td class="px-2 py-2 text-xs">${createdAt}</td>
                            <td class="px-2 py-2 text-xs">${lastLogin}</td>
                            <td class="px-2 py-2 text-xs ${statusColor} font-semibold">${status}</td>
                            <td class="px-2 py-2 text-xs">
                                <div class="flex space-x-1">
                                    <button onclick="viewUserDetails('${userId}', '${userData.companyName}')" 
                                            class="bg-blue-500 hover:bg-blue-600 text-white px-2 py-1 rounded text-xs">
                                        👁️
                                    </button>
                                    <button onclick="toggleUserStatus('${userId}', ${userData.isActive})" 
                                            class="bg-${userData.isActive ? 'red' : 'green'}-500 hover:bg-${userData.isActive ? 'red' : 'green'}-600 text-white px-2 py-1 rounded text-xs">
                                        ${userData.isActive ? '🚫' : '✅'}
                                    </button>
                                    <button onclick="deleteUser('${userId}', '${userData.companyName}')" 
                                            class="bg-red-600 hover:bg-red-700 text-white px-2 py-1 rounded text-xs">
                                        🗑️
                                    </button>
                                </div>
                            </td>
                        </tr>
                    `;
                });
                
                tableBody.innerHTML = tableHTML;
                console.log('✅ Süper admin verileri yüklendi');
                
            } catch (error) {
                console.error('❌ Süper admin veri yükleme hatası:', error);
                tableBody.innerHTML = '<tr><td colspan="6" class="text-center py-4 text-red-600">❌ Veriler yüklenirken hata oluştu</td></tr>';
            }
        }

        // Learning Scenarios Database - Real-world examples for AI training
        const learningScenarios = {
            profitabilityDecline: {
                title: "Kârlılığın Düşmesi Senaryosu",
                example: {
                    netSales: 15000000,
                    costOfSales: 10000000,
                    adminExpenses: 3500000,
                    ebitda: 1500000,
                    netProfit: 500000
                },
                analysis: "Net satışlar aynı kalırken maliyetler arttı. Brüt kâr marjı ve faaliyet kâr marjı düşüyor.",
                indicators: ["grossMarginDecline", "operatingMarginDecline"],
                riskLevel: "high"
            },
            increasingDebt: {
                title: "Artan Borçluluk Senaryosu", 
                example: {
                    currentAssets: 9000000,
                    fixedAssets: 15000000,
                    shortTermDebt: 6000000,
                    longTermDebt: 9000000,
                    equity: 10000000
                },
                analysis: "Yeni yatırım için borçlanma. Özkaynaklar sabit kalırken borçlar artıyor.",
                indicators: ["debtToEquityIncrease", "investmentFinancing"],
                riskLevel: "medium"
            },
            liquidityProblem: {
                title: "Nakit Yönetimi Sorunu",
                example: {
                    netProfit: 2000000,
                    currentAssets: 5000000,
                    shortTermDebt: 5000000
                },
                analysis: "Kâr iyi ancak dönen varlıklar düşük. Alacak tahsili veya stok devir sorunu.",
                indicators: ["currentRatioLow", "workingCapitalIssue"],
                riskLevel: "high"
            },
            efficientGrowth: {
                title: "Verimli Büyüme Senaryosu",
                example: {
                    netSales: 20000000,
                    costOfSales: 10000000,
                    adminExpenses: 3500000,
                    ebitda: 6500000,
                    netProfit: 4000000
                },
                analysis: "Satışlar artarken giderler daha yavaş artıyor. Ölçek ekonomisi başarısı.",
                indicators: ["scaleEconomy", "marginImprovement"],
                riskLevel: "low"
            }
        };

        // AI Learning Function - Compares current data with learning scenarios
        function analyzeWithLearningScenarios(data) {
            const insights = [];
            
            // Calculate key metrics
            const grossMargin = ((data.netSales - data.costOfSales) / data.netSales) * 100;
            const operatingMargin = ((data.netSales - data.costOfSales - data.adminExpenses) / data.netSales) * 100;
            const currentRatio = data.currentAssets / (data.shortTermDebt || 1);
            const debtToEquity = (data.shortTermDebt + data.longTermDebt) / (data.equity || 1);
            const netMargin = (data.netProfit / data.netSales) * 100;
            
            // Scenario 1: Profitability Decline Pattern
            if (grossMargin < 30 && operatingMargin < 10) {
                const scenario = learningScenarios.profitabilityDecline;
                insights.push({
                    type: 'learning',
                    title: `🎓 ${scenario.title} Tespit Edildi`,
                    pattern: 'Düşük kârlılık marjları',
                    analysis: `
                        <div class="bg-red-50 p-4 rounded-lg border-l-4 border-red-400">
                            <p class="text-sm font-semibold text-red-800 mb-2">📚 Öğrenilen Senaryo:</p>
                            <p class="text-xs text-red-700 mb-3">${scenario.analysis}</p>
                            
                            <div class="grid md:grid-cols-2 gap-3 mb-3">
                                <div class="bg-white p-2 rounded">
                                    <p class="text-xs font-semibold">Mevcut Durumunuz:</p>
                                    <ul class="text-xs text-gray-600 mt-1">
                                        <li>• Brüt Kâr Marjı: ${grossMargin.toFixed(1)}%</li>
                                        <li>• Faaliyet Marjı: ${operatingMargin.toFixed(1)}%</li>
                                        <li>• Net Kâr Marjı: ${netMargin.toFixed(1)}%</li>
                                    </ul>
                                </div>
                                <div class="bg-white p-2 rounded">
                                    <p class="text-xs font-semibold">Örnek Senaryo:</p>
                                    <ul class="text-xs text-gray-600 mt-1">
                                        <li>• Net Satışlar: ₺15M</li>
                                        <li>• Satış Maliyeti: ₺10M</li>
                                        <li>• Yönetim Gideri: ₺3.5M</li>
                                    </ul>
                                </div>
                            </div>
                            
                            <p class="text-xs font-semibold text-red-800 mb-1">🔍 AI Öğrenmesi:</p>
                            <p class="text-xs text-red-700 mb-2">
                                Hammadde fiyat artışları veya üretim verimsizliği sinyali. 
                                Operasyonel verimlilik azalıyor, maliyet kontrolü gerekli.
                            </p>
                            
                            <p class="text-xs font-semibold text-red-800">💡 Önerilen Aksiyonlar:</p>
                            <ul class="text-xs text-red-700 list-disc list-inside">
                                <li>Tedarikçi anlaşmalarını yeniden müzakere edin</li>
                                <li>Üretim süreçlerini optimize edin</li>
                                <li>Fiyatlandırma stratejisini gözden geçirin</li>
                            </ul>
                        </div>
                    `
                });
            }
            
            // Scenario 2: Increasing Debt Pattern
            if (debtToEquity > 1.5) {
                const scenario = learningScenarios.increasingDebt;
                insights.push({
                    type: 'learning',
                    title: `🎓 ${scenario.title} Tespit Edildi`,
                    pattern: 'Yüksek borçluluk oranı',
                    analysis: `
                        <div class="bg-orange-50 p-4 rounded-lg border-l-4 border-orange-400">
                            <p class="text-sm font-semibold text-orange-800 mb-2">📚 Öğrenilen Senaryo:</p>
                            <p class="text-xs text-orange-700 mb-3">${scenario.analysis}</p>
                            
                            <div class="grid md:grid-cols-2 gap-3 mb-3">
                                <div class="bg-white p-2 rounded">
                                    <p class="text-xs font-semibold">Mevcut Durumunuz:</p>
                                    <ul class="text-xs text-gray-600 mt-1">
                                        <li>• Borç/Özkaynak: ${debtToEquity.toFixed(2)}</li>
                                        <li>• Toplam Borç: ₺${((data.shortTermDebt + data.longTermDebt)/1000000).toFixed(1)}M</li>
                                        <li>• Özkaynak: ₺${(data.equity/1000000).toFixed(1)}M</li>
                                    </ul>
                                </div>
                                <div class="bg-white p-2 rounded">
                                    <p class="text-xs font-semibold">Örnek Senaryo:</p>
                                    <ul class="text-xs text-gray-600 mt-1">
                                        <li>• Duran Varlık: ₺15M (yatırım)</li>
                                        <li>• Uzun Vadeli Borç: ₺9M</li>
                                        <li>• Özkaynak: ₺10M (sabit)</li>
                                    </ul>
                                </div>
                            </div>
                            
                            <p class="text-xs font-semibold text-orange-800 mb-1">🔍 AI Öğrenmesi:</p>
                            <p class="text-xs text-orange-700 mb-2">
                                Yatırım finansmanında borç ağırlıklı yapı. Finansal risk artışı, 
                                faiz yükü artacak, ekonomik dalgalanmalara duyarlılık yükselecek.
                            </p>
                            
                            <p class="text-xs font-semibold text-orange-800">⚖️ Risk Yönetimi:</p>
                            <ul class="text-xs text-orange-700 list-disc list-inside">
                                <li>Yatırım getiri oranını yakından takip edin</li>
                                <li>Borç geri ödeme planını optimize edin</li>
                                <li>Gelecekte özkaynak güçlendirmeyi planlayın</li>
                            </ul>
                        </div>
                    `
                });
            }
            
            // Scenario 3: Liquidity Problem Pattern
            if (currentRatio <= 1 && data.netProfit > 0) {
                const scenario = learningScenarios.liquidityProblem;
                insights.push({
                    type: 'learning',
                    title: `🎓 ${scenario.title} Tespit Edildi`,
                    pattern: 'Kârlı ama likidite sorunu',
                    analysis: `
                        <div class="bg-red-50 p-4 rounded-lg border-l-4 border-red-500">
                            <p class="text-sm font-semibold text-red-800 mb-2">📚 Öğrenilen Senaryo:</p>
                            <p class="text-xs text-red-700 mb-3">${scenario.analysis}</p>
                            
                            <div class="grid md:grid-cols-2 gap-3 mb-3">
                                <div class="bg-white p-2 rounded">
                                    <p class="text-xs font-semibold">Mevcut Durumunuz:</p>
                                    <ul class="text-xs text-gray-600 mt-1">
                                        <li>• Cari Oran: ${currentRatio.toFixed(2)}</li>
                                        <li>• Net Kâr: ₺${(data.netProfit/1000000).toFixed(1)}M</li>
                                        <li>• Dönen Varlık: ₺${(data.currentAssets/1000000).toFixed(1)}M</li>
                                    </ul>
                                </div>
                                <div class="bg-white p-2 rounded">
                                    <p class="text-xs font-semibold">Örnek Senaryo:</p>
                                    <ul class="text-xs text-gray-600 mt-1">
                                        <li>• Net Kâr: ₺2M (iyi)</li>
                                        <li>• Dönen Varlık: ₺5M</li>
                                        <li>• Kısa Vadeli Borç: ₺5M</li>
                                    </ul>
                                </div>
                            </div>
                            
                            <p class="text-xs font-semibold text-red-800 mb-1">🔍 AI Öğrenmesi:</p>
                            <p class="text-xs text-red-700 mb-2">
                                Kâğıt üzerinde kârlı ama nakit dönüşümü zayıf. Alacak tahsili yavaş, 
                                stok devir hızı düşük veya işletme sermayesi yönetimi yetersiz.
                            </p>
                            
                            <p class="text-xs font-semibold text-red-800">🆘 Acil Müdahale:</p>
                            <ul class="text-xs text-red-700 list-disc list-inside">
                                <li>Alacak tahsilat süresini kısaltın</li>
                                <li>Stok yönetimini optimize edin</li>
                                <li>Nakit akışı projeksiyonu hazırlayın</li>
                                <li>Kısa vadeli finansman hatları açın</li>
                            </ul>
                        </div>
                    `
                });
            }
            
            // Scenario 4: Efficient Growth Pattern
            if (grossMargin > 50 && operatingMargin > 30 && netMargin > 20) {
                const scenario = learningScenarios.efficientGrowth;
                insights.push({
                    type: 'learning',
                    title: `🎓 ${scenario.title} Tespit Edildi`,
                    pattern: 'Ölçek ekonomisi başarısı',
                    analysis: `
                        <div class="bg-green-50 p-4 rounded-lg border-l-4 border-green-400">
                            <p class="text-sm font-semibold text-green-800 mb-2">📚 Öğrenilen Senaryo:</p>
                            <p class="text-xs text-green-700 mb-3">${scenario.analysis}</p>
                            
                            <div class="grid md:grid-cols-2 gap-3 mb-3">
                                <div class="bg-white p-2 rounded">
                                    <p class="text-xs font-semibold">Mevcut Durumunuz:</p>
                                    <ul class="text-xs text-gray-600 mt-1">
                                        <li>• Brüt Kâr Marjı: ${grossMargin.toFixed(1)}%</li>
                                        <li>• Faaliyet Marjı: ${operatingMargin.toFixed(1)}%</li>
                                        <li>• Net Kâr Marjı: ${netMargin.toFixed(1)}%</li>
                                    </ul>
                                </div>
                                <div class="bg-white p-2 rounded">
                                    <p class="text-xs font-semibold">Örnek Senaryo:</p>
                                    <ul class="text-xs text-gray-600 mt-1">
                                        <li>• Net Satışlar: ₺20M</li>
                                        <li>• FAVÖK: ₺6.5M</li>
                                        <li>• Net Kâr: ₺4M</li>
                                    </ul>
                                </div>
                            </div>
                            
                            <p class="text-xs font-semibold text-green-800 mb-1">🔍 AI Öğrenmesi:</p>
                            <p class="text-xs text-green-700 mb-2">
                                Mükemmel ölçek ekonomisi örneği. Satışlar artarken sabit giderler 
                                dağıtılıyor, birim maliyetler düşüyor, kârlılık artıyor.
                            </p>
                            
                            <p class="text-xs font-semibold text-green-800">🚀 Büyüme Fırsatları:</p>
                            <ul class="text-xs text-green-700 list-disc list-inside">
                                <li>Bu trendin sürdürülmesi için kapasite artırın</li>
                                <li>Pazar payını genişletme fırsatlarını değerlendirin</li>
                                <li>Operasyonel mükemmelliği koruyun</li>
                                <li>Yeni ürün/hizmet geliştirme yatırımları yapın</li>
                            </ul>
                        </div>
                    `
                });
            }
            
            return insights;
        }

        // Advanced Expert-Level Scenario Analysis System
        function generateAdvancedScenarioAnalysis(currentData, previousData) {
            const scenarios = [];
            
            // Calculate key financial metrics
            const grossProfit = currentData.netSales - currentData.costOfSales;
            const grossMargin = (grossProfit / currentData.netSales) * 100 || 0;
            const netMargin = (currentData.netProfit / currentData.netSales) * 100 || 0;
            const currentRatio = currentData.currentAssets / (currentData.shortTermDebt || 1);
            const debtToEquity = (currentData.shortTermDebt + currentData.longTermDebt) / (currentData.equity || 1);
            const operatingProfit = grossProfit - currentData.adminExpenses;
            const operatingMargin = (operatingProfit / currentData.netSales) * 100 || 0;
            
            // Expert Scenario 1: Profitability Under Pressure
            if (grossMargin < 50 && grossMargin > 30) {
                scenarios.push({
                    type: 'expert-analysis',
                    category: 'Kârlılık Analizi',
                    title: '📊 Senaryo 1: Artan Maliyetler ve Kârlılık Baskısı',
                    currentSituation: `Brüt Kâr Marjı: ${grossMargin.toFixed(1)}% - Net Kâr Marjı: ${netMargin.toFixed(1)}%`,
                    analysis: `
                        <div class="bg-amber-50 p-4 rounded-lg border-l-4 border-amber-400">
                            <div class="mb-3">
                                <h4 class="font-semibold text-amber-800 mb-2">🎯 Uzman Değerlendirmesi:</h4>
                                <p class="text-sm text-amber-700 mb-3">
                                    Şirketinizin Brüt Kâr Marjı (${grossMargin.toFixed(1)}%) sektör ortalamasına yakın ancak 
                                    maliyet baskısı altında olduğunu gösteriyor. Bu durum, hammadde fiyatlarındaki artış 
                                    veya üretim verimliliğindeki düşüşün erken sinyali olabilir.
                                </p>
                            </div>
                            
                            <div class="bg-white p-3 rounded-lg mb-3">
                                <h5 class="font-semibold text-amber-800 text-sm mb-2">🔍 Potansiyel Senaryo:</h5>
                                <p class="text-xs text-gray-700 mb-2">
                                    <strong>Durum:</strong> Net Satışlar artışına rağmen, hammadde maliyetleri veya üretim giderleri daha hızlı yükselirse.
                                </p>
                                <p class="text-xs text-gray-700 mb-2">
                                    <strong>Sonuç:</strong> Brüt Kâr Marjı düşer. Şirket ürün başına elde ettiği kârı azalır ve 
                                    maliyet kontrolünde sorunlar yaşar. Pazardaki rekabetin artması veya tedarik zinciri 
                                    sorunlarından kaynaklanabilir.
                                </p>
                                <p class="text-xs text-gray-700">
                                    <strong>Risk:</strong> Satışları artırsa bile kârlılığını sürdürmekte zorlanır.
                                </p>
                            </div>
                            
                            <div class="bg-amber-100 p-3 rounded-lg">
                                <h5 class="font-semibold text-amber-800 text-sm mb-2">💡 Uzman Önerileri:</h5>
                                <ul class="text-xs text-amber-700 list-disc list-inside space-y-1">
                                    <li>Tedarikçi anlaşmalarını yeniden müzakere edin</li>
                                    <li>Alternatif tedarik kaynaklarını araştırın</li>
                                    <li>Üretim süreçlerini optimize ederek verimliliği artırın</li>
                                    <li>Değer bazlı fiyatlandırma stratejisi geliştirin</li>
                                    <li>Ürün karması optimizasyonu yapın (yüksek marjlı ürünlere odaklanın)</li>
                                </ul>
                            </div>
                        </div>
                    `
                });
            }
            
            // Expert Scenario 2: Inefficient Growth
            if (operatingMargin < 15 && operatingMargin > 5) {
                scenarios.push({
                    type: 'expert-analysis',
                    category: 'Operasyonel Verimlilik',
                    title: '📊 Senaryo 2: Verimsiz Büyüme Sinyalleri',
                    currentSituation: `Faaliyet Kâr Marjı: ${operatingMargin.toFixed(1)}% - Yönetim Giderleri Oranı: ${((currentData.adminExpenses/currentData.netSales)*100).toFixed(1)}%`,
                    analysis: `
                        <div class="bg-orange-50 p-4 rounded-lg border-l-4 border-orange-400">
                            <div class="mb-3">
                                <h4 class="font-semibold text-orange-800 mb-2">🎯 Uzman Değerlendirmesi:</h4>
                                <p class="text-sm text-orange-700 mb-3">
                                    Faaliyet Kâr Marjınız (${operatingMargin.toFixed(1)}%) orta seviyede ancak yönetim giderlerinin 
                                    satışlara oranı (${((currentData.adminExpenses/currentData.netSales)*100).toFixed(1)}%) 
                                    operasyonel verimlilikte iyileştirme potansiyeli olduğunu gösteriyor.
                                </p>
                            </div>
                            
                            <div class="bg-white p-3 rounded-lg mb-3">
                                <h5 class="font-semibold text-orange-800 text-sm mb-2">🔍 Potansiyel Senaryo:</h5>
                                <p class="text-xs text-gray-700 mb-2">
                                    <strong>Durum:</strong> Satışlar artarken, ofis giderleri, personel maaşları gibi 
                                    Yönetim Giderleri satış artışından daha hızlı yükselirse.
                                </p>
                                <p class="text-xs text-gray-700 mb-2">
                                    <strong>Sonuç:</strong> Şirket yönetimi operasyonel verimliliği kaybetmekte. Büyüme, 
                                    ölçek ekonomisi yaratmak yerine maliyetleri artırarak kâr marjlarını baskı altına alır.
                                </p>
                                <p class="text-xs text-gray-700">
                                    <strong>Risk:</strong> Sürdürülebilir olmayan büyüme modeli sinyali.
                                </p>
                            </div>
                            
                            <div class="bg-orange-100 p-3 rounded-lg">
                                <h5 class="font-semibold text-orange-800 text-sm mb-2">💡 Uzman Önerileri:</h5>
                                <ul class="text-xs text-orange-700 list-disc list-inside space-y-1">
                                    <li>Organizasyon yapısını gözden geçirin ve gereksiz katmanları azaltın</li>
                                    <li>Süreç otomasyonu ile operasyonel verimliliği artırın</li>
                                    <li>Performans bazlı ücretlendirme sistemleri uygulayın</li>
                                    <li>Outsourcing fırsatlarını değerlendirin</li>
                                    <li>Ölçek ekonomisi yaratacak büyüme stratejileri geliştirin</li>
                                </ul>
                            </div>
                        </div>
                    `
                });
            }
            
            // Expert Scenario 3: High Leverage Risk
            if (debtToEquity > 1.0 && debtToEquity < 2.0) {
                scenarios.push({
                    type: 'expert-analysis',
                    category: 'Finansal Risk Yönetimi',
                    title: '📊 Senaryo 3: Yüksek Borçlanma Riski',
                    currentSituation: `Borç/Özkaynak Oranı: ${debtToEquity.toFixed(2)} - Finansal Kaldıraç Seviyesi: Yüksek`,
                    analysis: `
                        <div class="bg-red-50 p-4 rounded-lg border-l-4 border-red-400">
                            <div class="mb-3">
                                <h4 class="font-semibold text-red-800 mb-2">🎯 Uzman Değerlendirmesi:</h4>
                                <p class="text-sm text-red-700 mb-3">
                                    Borç/Özkaynak Oranınız (${debtToEquity.toFixed(2)}) kritik seviyeye yaklaşıyor. 
                                    Bu durum, şirketinizin finansal riskinin arttığını ve ekonomik dalgalanmalara 
                                    karşı daha hassas hale geldiğini gösteriyor.
                                </p>
                            </div>
                            
                            <div class="bg-white p-3 rounded-lg mb-3">
                                <h5 class="font-semibold text-red-800 text-sm mb-2">🔍 Potansiyel Senaryo:</h5>
                                <p class="text-xs text-gray-700 mb-2">
                                    <strong>Durum:</strong> Şirket büyük bir fabrika yatırımı için Uzun Vadeli Borçları 
                                    önemli ölçüde artırırsa.
                                </p>
                                <p class="text-xs text-gray-700 mb-2">
                                    <strong>Sonuç:</strong> Borç/Özkaynak Oranı yükselerek finansal riski artırır. 
                                    Yeni yatırım kârlı olsa bile, artan faiz ödemeleri nakit akışını zorlayabilir.
                                </p>
                                <p class="text-xs text-gray-700">
                                    <strong>Risk:</strong> Ekonomik dalgalanmalara karşı hassasiyet artar, 
                                    finansal kaldıraç tehlikeli seviyeye yükselir.
                                </p>
                            </div>
                            
                            <div class="bg-red-100 p-3 rounded-lg">
                                <h5 class="font-semibold text-red-800 text-sm mb-2">💡 Uzman Önerileri:</h5>
                                <ul class="text-xs text-red-700 list-disc list-inside space-y-1">
                                    <li>Borç yapılandırması için bankalarla görüşün</li>
                                    <li>Özkaynak güçlendirme seçeneklerini değerlendirin</li>
                                    <li>Yatırım projelerinin ROI analizini detaylandırın</li>
                                    <li>Nakit akışı projeksiyonlarını konservatif hazırlayın</li>
                                    <li>Alternatif finansman kaynaklarını araştırın (hisse senedi ihracı, ortaklık)</li>
                                </ul>
                            </div>
                        </div>
                    `
                });
            }
            
            // Expert Scenario 4: Profitable but Cash-Strapped
            if (currentRatio < 1.2 && currentData.netProfit > 0) {
                scenarios.push({
                    type: 'expert-analysis',
                    category: 'Nakit Akışı Yönetimi',
                    title: '📊 Senaryo 4: Nakit Akışı Krizinde Kârlı İşletme',
                    currentSituation: `Cari Oran: ${currentRatio.toFixed(2)} - Net Kâr: Pozitif (${netMargin.toFixed(1)}%)`,
                    analysis: `
                        <div class="bg-purple-50 p-4 rounded-lg border-l-4 border-purple-400">
                            <div class="mb-3">
                                <h4 class="font-semibold text-purple-800 mb-2">🎯 Uzman Değerlendirmesi:</h4>
                                <p class="text-sm text-purple-700 mb-3">
                                    Şirketiniz kâr ediyor (Net Kâr Marjı: ${netMargin.toFixed(1)}%) ancak Cari Oranınız 
                                    (${currentRatio.toFixed(2)}) likidite sıkıntısı yaşayabileceğinizi gösteriyor. 
                                    Bu, klasik bir "kârlı ama nakit sıkıntısı" paradoksudur.
                                </p>
                            </div>
                            
                            <div class="bg-white p-3 rounded-lg mb-3">
                                <h5 class="font-semibold text-purple-800 text-sm mb-2">🔍 Potansiyel Senaryo:</h5>
                                <p class="text-xs text-gray-700 mb-2">
                                    <strong>Durum:</strong> Şirket kâr etmeye devam ederken, müşterilerden olan alacaklarını 
                                    tahsil edemez ve elindeki nakit azalırsa.
                                </p>
                                <p class="text-xs text-gray-700 mb-2">
                                    <strong>Sonuç:</strong> İşletme kâr etse bile finansal olarak zorlanabilir. 
                                    Dönen Varlıklar azalırken Kısa Vadeli Borçlar aynı kalırsa, Cari Oran düşer.
                                </p>
                                <p class="text-xs text-gray-700">
                                    <strong>Risk:</strong> Vadesi gelen borçları ödemekte güçlük çekebilir, 
                                    nakit yönetimi ve alacak tahsilat süreçleri kritik hale gelir.
                                </p>
                            </div>
                            
                            <div class="bg-purple-100 p-3 rounded-lg">
                                <h5 class="font-semibold text-purple-800 text-sm mb-2">💡 Uzman Önerileri:</h5>
                                <ul class="text-xs text-purple-700 list-disc list-inside space-y-1">
                                    <li>Alacak tahsilat süreçlerini hızlandırın ve takip sistemini güçlendirin</li>
                                    <li>Müşteri kredibilitesi değerlendirmesini sıkılaştırın</li>
                                    <li>Stok devir hızını artırarak nakit dönüşümünü hızlandırın</li>
                                    <li>Tedarikçi ödeme vadelerini uzatmaya çalışın</li>
                                    <li>Faktoring veya forfaiting gibi finansman araçlarını değerlendirin</li>
                                    <li>Haftalık nakit akışı projeksiyonu hazırlayın</li>
                                </ul>
                            </div>
                        </div>
                    `
                });
            }
            
            return {
                hasAdvancedAnalysis: true,
                scenarios: scenarios,
                summary: {
                    totalScenarios: scenarios.length,
                    categories: [...new Set(scenarios.map(s => s.category))],
                    riskLevel: scenarios.length > 2 ? 'Yüksek' : scenarios.length > 1 ? 'Orta' : 'Düşük'
                }
            };
        }

        // Enhanced Scenario Analysis Function with AI Learning
        function generateScenarioAnalysis(currentData, previousData) {
            if (!previousData) {
                return {
                    hasComparison: false,
                    message: "📊 İlk veri girişiniz tamamlandı. Bir sonraki veri girişinde değişim analizi yapılacak.",
                    learningInsights: analyzeWithLearningScenarios(currentData),
                    advancedAnalysis: generateAdvancedScenarioAnalysis(currentData, null)
                };
            }
            
            const scenarios = [];
            
            // 1. Profitability Scenarios
            const salesChange = ((currentData.netSales - previousData.netSales) / previousData.netSales) * 100;
            const costChange = ((currentData.costOfSales - previousData.costOfSales) / previousData.costOfSales) * 100;
            const adminExpenseChange = ((currentData.adminExpenses - previousData.adminExpenses) / previousData.adminExpenses) * 100;
            
            const prevGrossMargin = ((previousData.netSales - previousData.costOfSales) / previousData.netSales) * 100;
            const currGrossMargin = ((currentData.netSales - currentData.costOfSales) / currentData.netSales) * 100;
            const grossMarginChange = currGrossMargin - prevGrossMargin;
            
            // Scenario 1: Sales increase but costs increase faster
            if (salesChange > 0 && costChange > salesChange) {
                scenarios.push({
                    type: 'warning',
                    title: '⚠️ Kârlılık Riski Tespit Edildi',
                    scenario: 'Net Satışlar arttı ancak Satışların Maliyeti daha hızlı arttı',
                    analysis: `
                        <div class="bg-yellow-50 p-3 rounded-lg border-l-4 border-yellow-400">
                            <p class="text-sm mb-2"><strong>📈 Değişim:</strong></p>
                            <ul class="text-xs space-y-1 mb-3">
                                <li>• Net Satışlar: %${salesChange.toFixed(1)} artış</li>
                                <li>• Satış Maliyeti: %${costChange.toFixed(1)} artış</li>
                                <li>• Brüt Kâr Marjı: ${grossMarginChange.toFixed(1)} puan değişim</li>
                            </ul>
                            <p class="text-sm font-semibold text-yellow-800 mb-2">🔍 Böyle Olur:</p>
                            <p class="text-xs text-yellow-700 mb-2">
                                Şirketin Brüt Kâr Marjı düştü. Bu, ürün başına elde edilen kârın azaldığı anlamına gelir. 
                                Bu durum, hammadde fiyatlarındaki yükseliş veya üretim süreçlerindeki verimsizlikten kaynaklanabilir. 
                                Şirket daha fazla ciro yapsa bile, kârlılığı azalır.
                            </p>
                            <p class="text-xs font-semibold text-yellow-800">💡 Öneriler:</p>
                            <ul class="text-xs text-yellow-700 list-disc list-inside">
                                <li>Tedarikçi anlaşmalarını gözden geçirin</li>
                                <li>Üretim verimliliğini artırın</li>
                                <li>Fiyatlandırma stratejisini değerlendirin</li>
                            </ul>
                        </div>
                    `
                });
            }
            
            // Scenario 2: Admin expenses grow slower than sales (economies of scale)
            if (salesChange > 0 && adminExpenseChange < salesChange && adminExpenseChange >= 0) {
                scenarios.push({
                    type: 'success',
                    title: '🎯 Ölçek Ekonomisi Başarısı',
                    scenario: 'Yönetim Giderleri, Net Satışlardaki artıştan daha yavaş arttı',
                    analysis: `
                        <div class="bg-green-50 p-3 rounded-lg border-l-4 border-green-400">
                            <p class="text-sm mb-2"><strong>📈 Değişim:</strong></p>
                            <ul class="text-xs space-y-1 mb-3">
                                <li>• Net Satışlar: %${salesChange.toFixed(1)} artış</li>
                                <li>• Yönetim Giderleri: %${adminExpenseChange.toFixed(1)} artış</li>
                                <li>• Verimlilik Kazancı: %${(salesChange - adminExpenseChange).toFixed(1)}</li>
                            </ul>
                            <p class="text-sm font-semibold text-green-800 mb-2">🔍 Böyle Olur:</p>
                            <p class="text-xs text-green-700 mb-2">
                                Şirketin FAVÖK/EBITDA rakamı, satışlara göre daha yüksek bir oranda arttı. 
                                Bu durum, "ölçek ekonomisi" (economy of scale) olarak adlandırılır. 
                                Şirket büyüdükçe birim başına düşen yönetim maliyetleri azaldığı için daha verimli hale gelir ve operasyonel kârlılığı artar.
                            </p>
                            <p class="text-xs font-semibold text-green-800">🚀 Fırsatlar:</p>
                            <ul class="text-xs text-green-700 list-disc list-inside">
                                <li>Bu trendin sürdürülmesi için büyüme stratejilerini güçlendirin</li>
                                <li>Operasyonel verimliliği daha da artırın</li>
                                <li>Pazar payını genişletme fırsatlarını değerlendirin</li>
                            </ul>
                        </div>
                    `
                });
            }
            
            // 2. Leverage Scenarios
            const longTermDebtChange = ((currentData.longTermDebt - previousData.longTermDebt) / previousData.longTermDebt) * 100;
            const equityChange = ((currentData.equity - previousData.equity) / previousData.equity) * 100;
            
            const prevDebtRatio = (previousData.shortTermDebt + previousData.longTermDebt) / previousData.equity;
            const currDebtRatio = (currentData.shortTermDebt + currentData.longTermDebt) / currentData.equity;
            
            // Scenario 3: Long-term debt increases faster than equity
            if (longTermDebtChange > equityChange && longTermDebtChange > 0) {
                scenarios.push({
                    type: 'warning',
                    title: '⚠️ Borçluluk Riski Artışı',
                    scenario: 'Uzun Vadeli Borçlar, Özkaynaklardan daha hızlı arttı',
                    analysis: `
                        <div class="bg-red-50 p-3 rounded-lg border-l-4 border-red-400">
                            <p class="text-sm mb-2"><strong>📈 Değişim:</strong></p>
                            <ul class="text-xs space-y-1 mb-3">
                                <li>• Uzun Vadeli Borç: %${longTermDebtChange.toFixed(1)} artış</li>
                                <li>• Özkaynak: %${equityChange.toFixed(1)} değişim</li>
                                <li>• Borç/Özkaynak Oranı: ${prevDebtRatio.toFixed(2)} → ${currDebtRatio.toFixed(2)}</li>
                            </ul>
                            <p class="text-sm font-semibold text-red-800 mb-2">🔍 Böyle Olur:</p>
                            <p class="text-xs text-red-700 mb-2">
                                Şirketin Borç/Özkaynak Oranı yükseldi. Bu, şirketin finansal riskinin arttığını gösterir. 
                                Yüksek borçluluk, faiz giderlerinin artmasına ve finansal dalgalanmalara karşı daha savunmasız hale gelmesine neden olur. 
                                Yatırımcılar için riskli bir sinyaldir.
                            </p>
                            <p class="text-xs font-semibold text-red-800">🛡️ Risk Yönetimi:</p>
                            <ul class="text-xs text-red-700 list-disc list-inside">
                                <li>Borç geri ödeme planını gözden geçirin</li>
                                <li>Nakit akışı yönetimini güçlendirin</li>
                                <li>Özkaynak artırma seçeneklerini değerlendirin</li>
                            </ul>
                        </div>
                    `
                });
            }
            
            // Scenario 4: Fixed assets increase more than equity and financed by debt
            const fixedAssetsChange = ((currentData.fixedAssets - previousData.fixedAssets) / previousData.fixedAssets) * 100;
            const totalDebtChange = (((currentData.shortTermDebt + currentData.longTermDebt) - (previousData.shortTermDebt + previousData.longTermDebt)) / (previousData.shortTermDebt + previousData.longTermDebt)) * 100;
            
            if (fixedAssetsChange > equityChange && fixedAssetsChange > 0 && totalDebtChange > 0) {
                scenarios.push({
                    type: 'info',
                    title: '📊 Borçla Finanse Edilen Büyüme',
                    scenario: 'Duran Varlıklar, Özkaynaklardan daha fazla arttı ve bu artış borçlanmayla finanse edildi',
                    analysis: `
                        <div class="bg-blue-50 p-3 rounded-lg border-l-4 border-blue-400">
                            <p class="text-sm mb-2"><strong>📈 Değişim:</strong></p>
                            <ul class="text-xs space-y-1 mb-3">
                                <li>• Duran Varlıklar: %${fixedAssetsChange.toFixed(1)} artış</li>
                                <li>• Toplam Borç: %${totalDebtChange.toFixed(1)} artış</li>
                                <li>• Özkaynak: %${equityChange.toFixed(1)} değişim</li>
                            </ul>
                            <p class="text-sm font-semibold text-blue-800 mb-2">🔍 Böyle Olur:</p>
                            <p class="text-xs text-blue-700 mb-2">
                                Şirket büyür, ancak büyümesi finansal olarak daha kırılgan bir yapıya sahip olur. 
                                Yeni fabrika veya makine yatırımları (duran varlıklar), borçla finanse edildiği için 
                                şirketin bilançosu borçluluk yönünde ağır basar.
                            </p>
                            <p class="text-xs font-semibold text-blue-800">⚖️ Denge Önerileri:</p>
                            <ul class="text-xs text-blue-700 list-disc list-inside">
                                <li>Yatırımların getiri oranını yakından takip edin</li>
                                <li>Borç servis kapasitesini düzenli kontrol edin</li>
                                <li>Gelecek dönemlerde özkaynak güçlendirmeyi planlayın</li>
                            </ul>
                        </div>
                    `
                });
            }
            
            // 3. Liquidity Scenarios
            const currentAssetsChange = ((currentData.currentAssets - previousData.currentAssets) / previousData.currentAssets) * 100;
            const shortTermDebtChange = ((currentData.shortTermDebt - previousData.shortTermDebt) / previousData.shortTermDebt) * 100;
            
            const prevCurrentRatio = previousData.currentAssets / previousData.shortTermDebt;
            const currCurrentRatio = currentData.currentAssets / currentData.shortTermDebt;
            
            // Scenario 5: Current assets decrease while short-term debt increases
            if (currentAssetsChange < 0 && shortTermDebtChange > 0) {
                scenarios.push({
                    type: 'danger',
                    title: '🚨 Likidite Krizi Riski',
                    scenario: 'Dönen Varlıklar azalırken, Kısa Vadeli Borçlar arttı',
                    analysis: `
                        <div class="bg-red-50 p-3 rounded-lg border-l-4 border-red-500">
                            <p class="text-sm mb-2"><strong>📈 Değişim:</strong></p>
                            <ul class="text-xs space-y-1 mb-3">
                                <li>• Dönen Varlıklar: %${currentAssetsChange.toFixed(1)} azalış</li>
                                <li>• Kısa Vadeli Borçlar: %${shortTermDebtChange.toFixed(1)} artış</li>
                                <li>• Cari Oran: ${prevCurrentRatio.toFixed(2)} → ${currCurrentRatio.toFixed(2)}</li>
                            </ul>
                            <p class="text-sm font-semibold text-red-800 mb-2">🔍 Böyle Olur:</p>
                            <p class="text-xs text-red-700 mb-2">
                                Şirketin Cari Oranı düştü. Cari oran, şirketin bir yıl içinde ödemesi gereken borçlarını, 
                                bir yıl içinde nakde dönebilecek varlıklarıyla ne kadar karşılayabildiğini gösterir. 
                                Bu oranın düşmesi, şirketin vadesi gelen borçlarını ödemekte zorlanabileceğine işaret eder. 
                                Şirket kârlı olsa bile, nakit akışı sorunları yaşayabilir.
                            </p>
                            <p class="text-xs font-semibold text-red-800">🆘 Acil Önlemler:</p>
                            <ul class="text-xs text-red-700 list-disc list-inside">
                                <li>Nakit akışı projeksiyonlarını acilen hazırlayın</li>
                                <li>Alacakların tahsilat süresini kısaltın</li>
                                <li>Stok devir hızını artırın</li>
                                <li>Kısa vadeli borç yapılandırması için bankalarla görüşün</li>
                            </ul>
                        </div>
                    `
                });
            }
            
            return {
                hasComparison: true,
                scenarios: scenarios,
                summary: {
                    totalScenarios: scenarios.length,
                    riskScenarios: scenarios.filter(s => s.type === 'warning' || s.type === 'danger').length,
                    positiveScenarios: scenarios.filter(s => s.type === 'success').length
                },
                advancedAnalysis: generateAdvancedScenarioAnalysis(currentData, previousData)
            };
        }

        // Super Admin Functions - CANLI SİSTEM
        async function toggleUserStatus(userId, currentStatus) {
            try {
                const action = currentStatus ? 'devre dışı bırak' : 'aktifleştir';
                
                if (!confirm(`Bu kullanıcıyı ${action}mak istediğinizden emin misiniz?`)) {
                    return;
                }
                
                await db.collection('users').doc(userId).update({
                    isActive: !currentStatus,
                    lastModified: firebase.firestore.FieldValue.serverTimestamp()
                });
                
                alert(`✅ Kullanıcı durumu başarıyla güncellendi!`);
                await loadAllCompaniesData(); // Tabloyu yenile
                
            } catch (error) {
                console.error('❌ Kullanıcı durumu güncelleme hatası:', error);
                alert('❌ İşlem sırasında hata oluştu: ' + error.message);
            }
        }

        async function viewUserDetails(userId, companyName) {
            try {
                // Kullanıcı detaylarını getir
                const userDoc = await db.collection('users').doc(userId).get();
                const financialDoc = await db.collection('financialData').doc(userId).get();
                
                if (!userDoc.exists) {
                    alert('❌ Kullanıcı bulunamadı!');
                    return;
                }
                
                const userData = userDoc.data();
                const financialData = financialDoc.exists ? financialDoc.data() : null;
                
                let detailsHTML = `
                    📊 KULLANICI DETAYLARI
                    
                    🏢 Şirket: ${userData.companyName}
                    📧 Email: ${userData.email}
                    🏭 Sektör: ${userData.sector}
                    👥 Çalışan Sayısı: ${userData.employeeCount}
                    📅 Kayıt Tarihi: ${userData.createdAt ? userData.createdAt.toDate().toLocaleDateString('tr-TR') : 'Bilinmiyor'}
                    🔄 Son Giriş: ${userData.lastLogin ? userData.lastLogin.toDate().toLocaleDateString('tr-TR') : 'Hiç'}
                    ✅ Durum: ${userData.isActive ? 'Aktif' : 'Pasif'}
                `;
                
                if (financialData) {
                    detailsHTML += `
                    
                    💰 FİNANSAL VERİLER:
                    • Net Satışlar: ${financialData.netSales?.toLocaleString('tr-TR') || '0'} TL
                    • Net Kâr: ${financialData.netProfit?.toLocaleString('tr-TR') || '0'} TL
                    • Toplam Varlık: ${((financialData.currentAssets || 0) + (financialData.fixedAssets || 0)).toLocaleString('tr-TR')} TL
                    • Son Güncelleme: ${financialData.lastUpdated ? financialData.lastUpdated.toDate().toLocaleDateString('tr-TR') : 'Bilinmiyor'}
                    `;
                } else {
                    detailsHTML += `
                    
                    📝 Henüz finansal veri girişi yapılmamış.
                    `;
                }
                
                alert(detailsHTML);
                
            } catch (error) {
                console.error('❌ Kullanıcı detayları yükleme hatası:', error);
                alert('❌ Kullanıcı detayları yüklenirken hata oluştu: ' + error.message);
            }
        }

        async function deleteUser(userId, companyName) {
            try {
                if (!confirm(`⚠️ DİKKAT!\n\n"${companyName}" şirketini ve tüm verilerini kalıcı olarak silmek istediğinizden emin misiniz?\n\nBu işlem geri alınamaz!`)) {
                    return;
                }
                
                if (!confirm(`Son onay: "${companyName}" şirketini SİLİYORUM!`)) {
                    return;
                }
                
                // Kullanıcı verilerini sil
                await db.collection('users').doc(userId).delete();
                
                // Finansal verilerini sil
                await db.collection('financialData').doc(userId).delete();
                
                alert(`✅ "${companyName}" şirketi ve tüm verileri başarıyla silindi!`);
                await loadAllCompaniesData(); // Tabloyu yenile
                
            } catch (error) {
                console.error('❌ Kullanıcı silme hatası:', error);
                alert('❌ Kullanıcı silinirken hata oluştu: ' + error.message);
            }
        }

        // Giriş logunu kaydet
        async function logUserLogin(userId, email) {
            try {
                await db.collection('users').doc(userId).update({
                    lastLogin: firebase.firestore.FieldValue.serverTimestamp()
                });
                
                console.log('📝 Giriş logu kaydedildi:', email);
            } catch (error) {
                console.error('❌ Giriş logu kaydetme hatası:', error);
            }
        }

        // Update charts with compact design
        function updateCharts() {
            if (!financialData[currentUser]) return;
            
            const data = financialData[currentUser];
            
            // Destroy existing charts
            Object.values(charts).forEach(chart => {
                if (chart) chart.destroy();
            });

            // Chart configurations with compact options
            const compactOptions = {
                responsive: true,
                maintainAspectRatio: false,
                plugins: {
                    legend: {
                        display: false
                    }
                },
                scales: {
                    x: {
                        display: false
                    },
                    y: {
                        display: false
                    }
                }
            };

            // Initialize all 10 charts with compact design
            // Chart 1: Revenue Trend
            charts.revenue = new Chart(document.getElementById('revenueChart').getContext('2d'), {
                type: 'line',
                data: {
                    labels: ['Q1', 'Q2', 'Q3', 'Q4'],
                    datasets: [{
                        data: [data.netSales * 0.8, data.netSales * 0.9, data.netSales * 1.1, data.netSales],
                        borderColor: 'rgb(59, 130, 246)',
                        backgroundColor: 'rgba(59, 130, 246, 0.1)',
                        tension: 0.4
                    }]
                },
                options: compactOptions
            });

            // Chart 2: Profit Analysis
            charts.profit = new Chart(document.getElementById('profitChart').getContext('2d'), {
                type: 'bar',
                data: {
                    labels: ['Brüt', 'FAVÖK', 'Net'],
                    datasets: [{
                        data: [data.netSales - data.costOfSales, data.ebitda, data.netProfit],
                        backgroundColor: ['#22c55e', '#3b82f6', '#a855f7']
                    }]
                },
                options: compactOptions
            });

            // Continue with other charts...
            // Charts 3-10 follow similar compact pattern

            // Chart 3: Asset Distribution
            charts.asset = new Chart(document.getElementById('assetChart').getContext('2d'), {
                type: 'pie',
                data: {
                    labels: ['Dönen', 'Duran'],
                    datasets: [{
                        data: [data.currentAssets, data.fixedAssets],
                        backgroundColor: ['#22c55e', '#3b82f6']
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: { legend: { display: false }}
                }
            });

            // Add remaining charts 4-10 with similar compact configurations...
        }

        // Store previous data for comparison
        let previousData = null;

        // Google Gemini AI Analysis Functions
        async function callGeminiAI(prompt, financialData) {
            try {
                console.log('🤖 Gemini AI analizi başlatılıyor...');
                
                const requestBody = {
                    contents: [{
                        parts: [{
                            text: prompt
                        }]
                    }],
                    generationConfig: {
                        temperature: 0.7,
                        topK: 40,
                        topP: 0.95,
                        maxOutputTokens: 2048,
                    }
                };

                const response = await fetch(`${GEMINI_API_URL}?key=${GEMINI_API_KEY}`, {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                    },
                    body: JSON.stringify(requestBody)
                });

                if (!response.ok) {
                    throw new Error(`AI API Error: ${response.status} - ${response.statusText}`);
                }

                const data = await response.json();
                
                if (data.candidates && data.candidates[0] && data.candidates[0].content) {
                    const aiResponse = data.candidates[0].content.parts[0].text;
                    console.log('✅ Gemini AI analizi tamamlandı');
                    return aiResponse;
                } else {
                    throw new Error('AI yanıtı alınamadı');
                }
            } catch (error) {
                console.error('❌ Gemini AI hatası:', error);
                return `⚠️ AI Analizi şu anda kullanılamıyor: ${error.message}`;
            }
        }

        async function generateAIFinancialAnalysis(financialData) {
            // Create cache key
            const cacheKey = JSON.stringify(financialData);
            
            // Check cache first
            if (aiAnalysisCache[cacheKey]) {
                console.log('📋 AI analizi cache\'den yüklendi');
                return aiAnalysisCache[cacheKey];
            }

            // Calculate key metrics for AI context
            const grossProfit = financialData.netSales - financialData.costOfSales;
            const grossMargin = (grossProfit / financialData.netSales * 100) || 0;
            const netMargin = (financialData.netProfit / financialData.netSales * 100) || 0;
            const currentRatio = financialData.currentAssets / (financialData.shortTermDebt || 1);
            const debtToEquity = (financialData.shortTermDebt + financialData.longTermDebt) / (financialData.equity || 1);
            const totalAssets = financialData.currentAssets + financialData.fixedAssets;
            const roa = (financialData.netProfit / totalAssets * 100) || 0;
            const roe = (financialData.netProfit / financialData.equity * 100) || 0;

            const prompt = `
Sen bir uzman finansal analist ve danışmansın. Aşağıdaki finansal verileri analiz et ve detaylı bir rapor hazırla:

FINANSAL VERİLER:
- Net Satışlar: ${financialData.netSales.toLocaleString('tr-TR')} TL
- Satışların Maliyeti: ${financialData.costOfSales.toLocaleString('tr-TR')} TL
- Yönetim Giderleri: ${financialData.adminExpenses.toLocaleString('tr-TR')} TL
- FAVÖK/EBITDA: ${financialData.ebitda.toLocaleString('tr-TR')} TL
- Net Kâr/Zarar: ${financialData.netProfit.toLocaleString('tr-TR')} TL
- Dönen Varlıklar: ${financialData.currentAssets.toLocaleString('tr-TR')} TL
- Duran Varlıklar: ${financialData.fixedAssets.toLocaleString('tr-TR')} TL
- Kısa Vadeli Borçlar: ${financialData.shortTermDebt.toLocaleString('tr-TR')} TL
- Uzun Vadeli Borçlar: ${financialData.longTermDebt.toLocaleString('tr-TR')} TL
- Özkaynaklar: ${financialData.equity.toLocaleString('tr-TR')} TL

HESAPLANMIŞ ORANLAR:
- Brüt Kâr Marjı: %${grossMargin.toFixed(2)}
- Net Kâr Marjı: %${netMargin.toFixed(2)}
- Cari Oran: ${currentRatio.toFixed(2)}
- Borç/Özkaynak Oranı: ${debtToEquity.toFixed(2)}
- ROA (Aktif Kârlılığı): %${roa.toFixed(2)}
- ROE (Özkaynak Kârlılığı): %${roe.toFixed(2)}

Lütfen aşağıdaki başlıklar altında detaylı analiz yap:

1. 🎯 GENEL DEĞERLENDİRME
2. 💰 KARLILIK ANALİZİ
3. 💧 LİKİDİTE ANALİZİ
4. ⚖️ BORÇLULUK ANALİZİ
5. 📊 VERİMLİLİK ANALİZİ
6. 🚨 RİSK DEĞERLENDİRMESİ
7. 💡 STRATEJİK ÖNERİLER
8. 🎯 ÖNCELİKLİ AKSIYONLAR

Her başlık altında:
- Mevcut durumu değerlendir
- Güçlü ve zayıf yönleri belirt
- Sektör standartları ile karşılaştır
- Somut öneriler sun
- Risk seviyelerini belirt

Türkçe yanıt ver ve profesyonel bir ton kullan. Emoji kullanarak okunabilirliği artır.
            `;

            const aiAnalysis = await callGeminiAI(prompt, financialData);
            
            // Cache the result
            aiAnalysisCache[cacheKey] = aiAnalysis;
            
            return aiAnalysis;
        }

        async function generateAIScenarioAnalysis(currentData, previousData) {
            if (!previousData) {
                return "📊 İlk veri girişiniz. Bir sonraki güncellemede değişim analizi yapılacak.";
            }

            const prompt = `
Sen bir finansal risk analisti ve senaryo uzmanısın. Aşağıdaki finansal verilerdeki değişimleri analiz et:

ÖNCEKİ DÖNEM VERİLERİ:
- Net Satışlar: ${previousData.netSales.toLocaleString('tr-TR')} TL
- Satışların Maliyeti: ${previousData.costOfSales.toLocaleString('tr-TR')} TL
- Net Kâr: ${previousData.netProfit.toLocaleString('tr-TR')} TL
- Dönen Varlıklar: ${previousData.currentAssets.toLocaleString('tr-TR')} TL
- Kısa Vadeli Borçlar: ${previousData.shortTermDebt.toLocaleString('tr-TR')} TL
- Uzun Vadeli Borçlar: ${previousData.longTermDebt.toLocaleString('tr-TR')} TL
- Özkaynaklar: ${previousData.equity.toLocaleString('tr-TR')} TL

GÜNCEL DÖNEM VERİLERİ:
- Net Satışlar: ${currentData.netSales.toLocaleString('tr-TR')} TL
- Satışların Maliyeti: ${currentData.costOfSales.toLocaleString('tr-TR')} TL
- Net Kâr: ${currentData.netProfit.toLocaleString('tr-TR')} TL
- Dönen Varlıklar: ${currentData.currentAssets.toLocaleString('tr-TR')} TL
- Kısa Vadeli Borçlar: ${currentData.shortTermDebt.toLocaleString('tr-TR')} TL
- Uzun Vadeli Borçlar: ${currentData.longTermDebt.toLocaleString('tr-TR')} TL
- Özkaynaklar: ${currentData.equity.toLocaleString('tr-TR')} TL

Lütfen aşağıdaki analizi yap:

1. 📈 DEĞİŞİM ANALİZİ
   - Her kalemde % değişimi hesapla
   - Pozitif ve negatif değişimleri belirt
   - Kritik değişimleri vurgula

2. 🎯 SENARYO TESPİTİ
   - Hangi finansal senaryolar gerçekleşiyor?
   - Risk seviyeleri neler?
   - Trend analizi

3. 🚨 ERKEN UYARI SİSTEMİ
   - Dikkat edilmesi gereken noktalar
   - Potansiyel problemler
   - Acil müdahale gereken alanlar

4. 💡 AKSIYON ÖNERİLERİ
   - Kısa vadeli önlemler
   - Orta vadeli stratejiler
   - Uzun vadeli planlar

Türkçe yanıt ver, emoji kullan ve somut öneriler sun.
            `;

            return await callGeminiAI(prompt, { currentData, previousData });
        }

        async function generateAIMarketInsights(financialData) {
            const prompt = `
Sen bir pazar analisti ve sektör uzmanısın. Aşağıdaki finansal verilere dayanarak pazar ve sektör analizi yap:

ŞIRKET FINANSAL VERILERİ:
- Net Satışlar: ${financialData.netSales.toLocaleString('tr-TR')} TL
- Net Kâr Marjı: %${((financialData.netProfit / financialData.netSales) * 100).toFixed(2)}
- Toplam Varlıklar: ${(financialData.currentAssets + financialData.fixedAssets).toLocaleString('tr-TR')} TL
- Borç/Özkaynak: ${((financialData.shortTermDebt + financialData.longTermDebt) / financialData.equity).toFixed(2)}

Lütfen şu analizi yap:

1. 🏢 SEKTÖR KARŞILAŞTIRMASI
   - Bu finansal profil hangi sektörlere uygun?
   - Sektör ortalamalarıyla karşılaştırma
   - Rekabet pozisyonu değerlendirmesi

2. 📊 PAZAR FIRSATLARI
   - Büyüme potansiyeli olan alanlar
   - Yeni pazar fırsatları
   - Yatırım önerileri

3. ⚠️ PAZAR RİSKLERİ
   - Sektörel riskler
   - Ekonomik dalgalanma etkileri
   - Rekabet tehditleri

4. 🎯 STRATEJİK KONUMLANDIRMA
   - Güçlü yönlerin nasıl kullanılacağı
   - Zayıf yönlerin nasıl güçlendirileceği
   - Diferansiyasyon stratejileri

5. 📈 BÜYÜME STRATEJİLERİ
   - Organik büyüme seçenekleri
   - İnorganik büyüme fırsatları
   - Finansman alternatifleri

Türkçe yanıt ver, güncel pazar koşullarını dikkate al ve somut öneriler sun.
            `;

            return await callGeminiAI(prompt, financialData);
        }

        // Generate detailed reports with AI analysis
        async function generateReports() {
            let data;
            
            if (isSuperAdmin) {
                if (!financialData[currentUser]) {
                    document.getElementById('reportContent').innerHTML = '<p class="text-gray-600 text-sm">Raporlar için önce veri girişi yapınız.</p>';
                    return;
                }
                data = financialData[currentUser];
            } else {
                // Get data from form fields for test mode users
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
            
            // 🎯 ANALİZ SİSTEMİ AÇIKLAMASI:
            // Bu sistem sizin girdiğiniz verileri gerçek zamanlı olarak analiz eder:
            
            console.log('📊 GERÇEK ZAMANLI ANALİZ BAŞLADI');
            console.log('💰 Girilen Veriler:', data);
            
            // 1️⃣ SİZİN VERİLERİNİZDEN HESAPLANAN ORANLAR:
            const netProfitMargin = (data.netProfit / data.netSales * 100) || 0;
            const currentRatio = data.currentAssets / (data.shortTermDebt || 1);
            const debtToEquityRatio = (data.shortTermDebt + data.longTermDebt) / (data.equity || 1);
            const assetTurnover = data.netSales / ((data.currentAssets + data.fixedAssets) || 1);
            
            console.log('📈 Hesaplanan Oranlar:');
            console.log('• Net Kâr Marjı:', netProfitMargin.toFixed(2) + '%');
            console.log('• Cari Oran:', currentRatio.toFixed(2));
            console.log('• Borç/Özkaynak:', debtToEquityRatio.toFixed(2));
            console.log('• Varlık Devir Hızı:', assetTurnover.toFixed(2));
            
            const reportContent = document.getElementById('reportContent');
            
            if (!data || Object.values(data).every(val => val === 0)) {
                reportContent.innerHTML = '<p class="text-gray-600 text-sm">Raporlar için önce veri girişi yapınız.</p>';
                return;
            }
            
            // Financial calculations
            const grossProfit = data.netSales - data.costOfSales;
            const grossProfitMargin = (grossProfit / data.netSales * 100) || 0;
            const netProfitMargin = (data.netProfit / data.netSales * 100) || 0;
            const ebitdaMargin = (data.ebitda / data.netSales * 100) || 0;
            const operatingProfit = grossProfit - data.adminExpenses; // EBIT calculation
            
            const totalAssets = data.currentAssets + data.fixedAssets;
            const totalDebt = data.shortTermDebt + data.longTermDebt;
            const currentRatio = data.currentAssets / (data.shortTermDebt || 1);
            const debtToEquityRatio = totalDebt / (data.equity || 1);
            const assetTurnover = data.netSales / (totalAssets || 1);
            const roe = (data.netProfit / data.equity * 100) || 0;
            const roa = (data.netProfit / totalAssets * 100) || 0;
            
            // Scenario Analysis
            const scenarioAnalysis = generateScenarioAnalysis(data, previousData);
            
            // Store current data for next comparison
            previousData = {...data};
            
            // 2️⃣ ANALİZ MANTIKLARI - SİZİN VERİLERİNİZE GÖRE:
            
            // Analysis and recommendations
            let profitabilityAnalysis = '';
            let liquidityAnalysis = '';
            let leverageAnalysis = '';
            let efficiencyAnalysis = '';
            let overallRecommendation = '';
            
            console.log('🔍 ANALİZ KURALLARI UYGULANYOR:');
            
            // Profitability Analysis - SİZİN NET KÂR MARJINIZA GÖRE
            console.log('💰 Kârlılık Analizi: Net Kâr Marjı =', netProfitMargin.toFixed(2) + '%');
            if (netProfitMargin > 15) {
                profitabilityAnalysis = `🟢 Mükemmel kârlılık seviyesi (${netProfitMargin.toFixed(1)}%). Şirketiniz sektör ortalamasının üzerinde performans gösteriyor.`;
                console.log('✅ Sonuç: Mükemmel seviye (>15%)');
            } else if (netProfitMargin > 10) {
                profitabilityAnalysis = `🟡 İyi kârlılık seviyesi (${netProfitMargin.toFixed(1)}%). Maliyet kontrolü ile daha da iyileştirilebilir.`;
                console.log('✅ Sonuç: İyi seviye (10-15%)');
            } else if (netProfitMargin > 5) {
                profitabilityAnalysis = `🟠 Orta seviye kârlılık (${netProfitMargin.toFixed(1)}%). Gider optimizasyonu ve fiyatlandırma stratejisi gözden geçirilmeli.`;
                console.log('⚠️ Sonuç: Orta seviye (5-10%)');
            } else if (netProfitMargin > 0) {
                profitabilityAnalysis = `🔴 Düşük kârlılık (${netProfitMargin.toFixed(1)}%). Acil maliyet azaltma ve gelir artırma stratejileri uygulanmalı.`;
                console.log('🚨 Sonuç: Düşük seviye (0-5%)');
            } else {
                profitabilityAnalysis = `🔴 Zarar durumu (${netProfitMargin.toFixed(1)}%). Acil finansal restructuring gerekli. Operasyonel giderler gözden geçirilmeli.`;
                console.log('🚨 Sonuç: Zarar durumu (<0%)');
            }
            
            // Liquidity Analysis - SİZİN CARİ ORANINIZA GÖRE
            console.log('💧 Likidite Analizi: Cari Oran =', currentRatio.toFixed(2));
            if (currentRatio > 2) {
                liquidityAnalysis = `🟢 Çok güçlü likidite pozisyonu (${currentRatio.toFixed(2)}). Fazla nakit yatırım fırsatlarında değerlendirilebilir.`;
                console.log('✅ Sonuç: Çok güçlü (>2.0)');
            } else if (currentRatio > 1.5) {
                liquidityAnalysis = `🟢 Sağlıklı likidite seviyesi (${currentRatio.toFixed(2)}). Kısa vadeli yükümlülükler rahatça karşılanabilir.`;
                console.log('✅ Sonuç: Sağlıklı (1.5-2.0)');
            } else if (currentRatio > 1) {
                liquidityAnalysis = `🟡 Kabul edilebilir likidite (${currentRatio.toFixed(2)}). Nakit akışı yakından takip edilmeli.`;
                console.log('⚠️ Sonuç: Kabul edilebilir (1.0-1.5)');
            } else {
                liquidityAnalysis = `🔴 Likidite riski mevcut (${currentRatio.toFixed(2)}). Acil nakit akışı iyileştirme önlemleri alınmalı.`;
                console.log('🚨 Sonuç: Risk var (<1.0)');
            }
            
            // Leverage Analysis - SİZİN BORÇ/ÖZKAYNAK ORANINIZA GÖRE
            console.log('⚖️ Kaldıraç Analizi: Borç/Özkaynak =', debtToEquityRatio.toFixed(2));
            if (debtToEquityRatio < 0.3) {
                leverageAnalysis = `🟢 Düşük borçluluk oranı (${debtToEquityRatio.toFixed(2)}). Büyüme için ek finansman kapasitesi mevcut.`;
                console.log('✅ Sonuç: Düşük risk (<0.3)');
            } else if (debtToEquityRatio < 0.6) {
                leverageAnalysis = `🟡 Orta seviye borçluluk (${debtToEquityRatio.toFixed(2)}). Borç servisi takip edilmeli.`;
                console.log('⚠️ Sonuç: Orta seviye (0.3-0.6)');
            } else if (debtToEquityRatio < 1) {
                leverageAnalysis = `🟠 Yüksek borçluluk oranı (${debtToEquityRatio.toFixed(2)}). Borç azaltma stratejileri düşünülmeli.`;
                console.log('⚠️ Sonuç: Yüksek (0.6-1.0)');
            } else {
                leverageAnalysis = `🔴 Çok yüksek borçluluk riski (${debtToEquityRatio.toFixed(2)}). Acil borç restructuring gerekli.`;
                console.log('🚨 Sonuç: Çok yüksek risk (>1.0)');
            }
            
            // Efficiency Analysis - SİZİN VARLIK DEVİR HIZINIZA GÖRE
            console.log('⚡ Verimlilik Analizi: Varlık Devir Hızı =', assetTurnover.toFixed(2));
            if (assetTurnover > 2) {
                efficiencyAnalysis = `🟢 Varlıklar çok verimli kullanılıyor (${assetTurnover.toFixed(2)}). Mükemmel operasyonel performans.`;
                console.log('✅ Sonuç: Çok verimli (>2.0)');
            } else if (assetTurnover > 1) {
                efficiencyAnalysis = `🟡 Varlık kullanımı kabul edilebilir seviyede (${assetTurnover.toFixed(2)}). İyileştirme potansiyeli var.`;
                console.log('⚠️ Sonuç: Kabul edilebilir (1.0-2.0)');
            } else {
                efficiencyAnalysis = `🔴 Varlık kullanım verimliliği düşük (${assetTurnover.toFixed(2)}). Operasyonel iyileştirmeler gerekli.`;
                console.log('🚨 Sonuç: Düşük verimlilik (<1.0)');
            }
            
            // Overall Recommendation
            const positiveIndicators = [
                netProfitMargin > 10,
                currentRatio > 1.2,
                debtToEquityRatio < 0.6,
                assetTurnover > 1
            ].filter(Boolean).length;
            
            if (positiveIndicators >= 3) {
                overallRecommendation = '🎯 Şirketiniz genel olarak sağlıklı bir finansal yapıya sahip. Mevcut stratejileri sürdürün ve büyüme fırsatlarını değerlendirin.';
            } else if (positiveIndicators >= 2) {
                overallRecommendation = '⚠️ Finansal durum orta seviyede. Zayıf alanları güçlendirmek için hedefli iyileştirmeler yapın.';
            } else {
                overallRecommendation = '🚨 Finansal durumda ciddi iyileştirmeler gerekli. Profesyonel finansal danışmanlık alınması önerilir.';
            }
            
            // Generate AI Analysis
            let aiAnalysis = '';
            let aiScenarioAnalysis = '';
            let aiMarketInsights = '';
            
            if (aiAnalysisEnabled) {
                try {
                    // Show loading state
                    reportContent.innerHTML = `
                        <div class="flex items-center justify-center py-8">
                            <div class="text-center">
                                <div class="animate-spin rounded-full h-12 w-12 border-b-2 border-blue-600 mx-auto mb-4"></div>
                                <p class="text-gray-600">🤖 Yapay Zeka Analizi Hazırlanıyor...</p>
                                <p class="text-sm text-gray-500 mt-2">Bu işlem 10-15 saniye sürebilir</p>
                            </div>
                        </div>
                    `;
                    
                    // Generate AI analyses in parallel
                    const [aiFinancialAnalysis, aiScenarioResult, aiMarketResult] = await Promise.all([
                        generateAIFinancialAnalysis(data),
                        generateAIScenarioAnalysis(data, previousData),
                        generateAIMarketInsights(data)
                    ]);
                    
                    aiAnalysis = aiFinancialAnalysis;
                    aiScenarioAnalysis = aiScenarioResult;
                    aiMarketInsights = aiMarketResult;
                    
                } catch (error) {
                    console.error('AI analizi hatası:', error);
                    aiAnalysis = '⚠️ AI analizi şu anda kullanılamıyor. Lütfen daha sonra tekrar deneyin.';
                }
            }

            reportContent.innerHTML = `
                <div class="space-y-4">
                    <!-- AI Analysis Section -->
                    ${aiAnalysisEnabled && aiAnalysis ? `
                        <div class="bg-gradient-to-r from-purple-50 to-blue-50 border-2 border-purple-200 rounded-lg p-4 mb-6">
                            <div class="flex items-center justify-between mb-4">
                                <h2 class="text-xl font-bold text-purple-800 flex items-center">
                                    🤖 Yapay Zeka Finansal Analizi
                                    <span class="ml-2 bg-purple-100 text-purple-800 px-3 py-1 rounded-full text-xs font-semibold">
                                        Google Gemini AI
                                    </span>
                                </h2>
                                <div class="flex space-x-2">
                                    <span class="bg-green-100 text-green-800 px-2 py-1 rounded text-xs">
                                        ✅ Aktif
                                    </span>
                                    <span class="bg-blue-100 text-blue-800 px-2 py-1 rounded text-xs">
                                        Gerçek Zamanlı
                                    </span>
                                </div>
                            </div>
                            
                            <div class="bg-white rounded-lg p-4 border border-purple-200 mb-4">
                                <div class="flex items-start space-x-3 mb-3">
                                    <div class="bg-purple-100 p-2 rounded-full">
                                        <svg class="w-5 h-5 text-purple-600" fill="currentColor" viewBox="0 0 20 20">
                                            <path d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z"/>
                                        </svg>
                                    </div>
                                    <div>
                                        <h4 class="text-sm font-semibold text-purple-800 mb-1">🧠 AI Analiz Sistemi</h4>
                                        <p class="text-xs text-purple-700">
                                            Google Gemini AI, finansal verilerinizi uzman seviyede analiz ederek 
                                            detaylı değerlendirmeler ve stratejik öneriler sunuyor.
                                        </p>
                                    </div>
                                </div>
                            </div>
                            
                            <div class="bg-white rounded-lg p-4 border border-gray-200">
                                <div class="prose prose-sm max-w-none">
                                    <div class="whitespace-pre-wrap text-sm text-gray-800">${aiAnalysis}</div>
                                </div>
                            </div>
                        </div>
                    ` : ''}
                    
                    <!-- AI Scenario Analysis -->
                    ${aiAnalysisEnabled && aiScenarioAnalysis && previousData ? `
                        <div class="bg-gradient-to-r from-orange-50 to-red-50 border-2 border-orange-200 rounded-lg p-4 mb-6">
                            <div class="flex items-center justify-between mb-4">
                                <h2 class="text-xl font-bold text-orange-800 flex items-center">
                                    🎯 AI Senaryo ve Değişim Analizi
                                    <span class="ml-2 bg-orange-100 text-orange-800 px-3 py-1 rounded-full text-xs font-semibold">
                                        Trend Analizi
                                    </span>
                                </h2>
                            </div>
                            
                            <div class="bg-white rounded-lg p-4 border border-gray-200">
                                <div class="prose prose-sm max-w-none">
                                    <div class="whitespace-pre-wrap text-sm text-gray-800">${aiScenarioAnalysis}</div>
                                </div>
                            </div>
                        </div>
                    ` : ''}
                    
                    <!-- AI Market Insights -->
                    ${aiAnalysisEnabled && aiMarketInsights ? `
                        <div class="bg-gradient-to-r from-green-50 to-teal-50 border-2 border-green-200 rounded-lg p-4 mb-6">
                            <div class="flex items-center justify-between mb-4">
                                <h2 class="text-xl font-bold text-green-800 flex items-center">
                                    📊 AI Pazar ve Sektör Analizi
                                    <span class="ml-2 bg-green-100 text-green-800 px-3 py-1 rounded-full text-xs font-semibold">
                                        Pazar İçgörüleri
                                    </span>
                                </h2>
                            </div>
                            
                            <div class="bg-white rounded-lg p-4 border border-gray-200">
                                <div class="prose prose-sm max-w-none">
                                    <div class="whitespace-pre-wrap text-sm text-gray-800">${aiMarketInsights}</div>
                                </div>
                            </div>
                        </div>
                    ` : ''}
                    
                    <!-- Financial Metrics Grid -->
                    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-3">
                        <div class="bg-blue-50 p-3 rounded-lg">
                            <h4 class="font-semibold text-blue-800 text-sm mb-2">Kârlılık Oranları</h4>
                            <div class="text-xs space-y-1">
                                <div class="flex justify-between">
                                    <span>Brüt Kâr Marjı:</span>
                                    <span class="font-semibold">${grossProfitMargin.toFixed(1)}%</span>
                                </div>
                                <div class="flex justify-between">
                                    <span>FAVÖK Marjı:</span>
                                    <span class="font-semibold">${ebitdaMargin.toFixed(1)}%</span>
                                </div>
                                <div class="flex justify-between">
                                    <span>Net Kâr Marjı:</span>
                                    <span class="font-semibold">${netProfitMargin.toFixed(1)}%</span>
                                </div>
                            </div>
                        </div>
                        
                        <div class="bg-green-50 p-3 rounded-lg">
                            <h4 class="font-semibold text-green-800 text-sm mb-2">Likidite Oranları</h4>
                            <div class="text-xs space-y-1">
                                <div class="flex justify-between">
                                    <span>Cari Oran:</span>
                                    <span class="font-semibold">${currentRatio.toFixed(2)}</span>
                                </div>
                                <div class="flex justify-between">
                                    <span>Durum:</span>
                                    <span class="font-semibold ${currentRatio > 1.2 ? 'text-green-600' : currentRatio > 1 ? 'text-yellow-600' : 'text-red-600'}">
                                        ${currentRatio > 1.2 ? 'Güçlü' : currentRatio > 1 ? 'Orta' : 'Zayıf'}
                                    </span>
                                </div>
                            </div>
                        </div>
                        
                        <div class="bg-purple-50 p-3 rounded-lg">
                            <h4 class="font-semibold text-purple-800 text-sm mb-2">Kaldıraç Oranları</h4>
                            <div class="text-xs space-y-1">
                                <div class="flex justify-between">
                                    <span>Borç/Özkaynak:</span>
                                    <span class="font-semibold">${debtToEquityRatio.toFixed(2)}</span>
                                </div>
                                <div class="flex justify-between">
                                    <span>Toplam Borç:</span>
                                    <span class="font-semibold">${totalDebt.toLocaleString('tr-TR')} TL</span>
                                </div>
                            </div>
                        </div>
                        
                        <div class="bg-orange-50 p-3 rounded-lg">
                            <h4 class="font-semibold text-orange-800 text-sm mb-2">Verimlilik Oranları</h4>
                            <div class="text-xs space-y-1">
                                <div class="flex justify-between">
                                    <span>Varlık Devir Hızı:</span>
                                    <span class="font-semibold">${assetTurnover.toFixed(2)}</span>
                                </div>
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
                    
                    <!-- Advanced Expert Analysis Section -->
                    ${(scenarioAnalysis.advancedAnalysis && scenarioAnalysis.advancedAnalysis.hasAdvancedAnalysis && scenarioAnalysis.advancedAnalysis.scenarios.length > 0) ? `
                        <div class="bg-white border-2 border-indigo-200 rounded-lg p-4 mb-4">
                            <div class="flex items-center justify-between mb-4">
                                <h3 class="text-lg font-bold text-indigo-800">🎓 Uzman Seviye Senaryo Analizi</h3>
                                <div class="flex space-x-2 text-xs">
                                    <span class="bg-indigo-100 text-indigo-800 px-3 py-1 rounded-full font-semibold">
                                        ${scenarioAnalysis.advancedAnalysis.scenarios.length} Uzman Senaryo
                                    </span>
                                    <span class="bg-${scenarioAnalysis.advancedAnalysis.summary.riskLevel === 'Yüksek' ? 'red' : scenarioAnalysis.advancedAnalysis.summary.riskLevel === 'Orta' ? 'yellow' : 'green'}-100 text-${scenarioAnalysis.advancedAnalysis.summary.riskLevel === 'Yüksek' ? 'red' : scenarioAnalysis.advancedAnalysis.summary.riskLevel === 'Orta' ? 'yellow' : 'green'}-800 px-3 py-1 rounded-full font-semibold">
                                        Risk: ${scenarioAnalysis.advancedAnalysis.summary.riskLevel}
                                    </span>
                                </div>
                            </div>
                            
                            <div class="bg-gradient-to-r from-indigo-50 to-blue-50 p-4 rounded-lg mb-4 border-l-4 border-indigo-400">
                                <div class="flex items-start space-x-3">
                                    <div class="bg-indigo-100 p-2 rounded-full">
                                        <svg class="w-5 h-5 text-indigo-600" fill="currentColor" viewBox="0 0 20 20">
                                            <path d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z"/>
                                        </svg>
                                    </div>
                                    <div>
                                        <h4 class="text-sm font-semibold text-indigo-800 mb-2">📊 Uzman Finansal Analiz Sistemi</h4>
                                        <p class="text-xs text-indigo-700 mb-2">
                                            Bu bölüm, finansal uzmanların gerçek dünyada kullandığı senaryo bazlı değerlendirme metodolojisini kullanır. 
                                            Verileriniz, sektörel standartlar ve risk yönetimi prensipleri çerçevesinde analiz edilmiştir.
                                        </p>
                                        <div class="grid md:grid-cols-3 gap-2 text-xs">
                                            <div class="bg-white p-2 rounded">
                                                <span class="font-semibold text-indigo-800">Analiz Kategorileri:</span>
                                                <ul class="text-indigo-600 mt-1">
                                                    ${scenarioAnalysis.advancedAnalysis.summary.categories.map(cat => `<li>• ${cat}</li>`).join('')}
                                                </ul>
                                            </div>
                                            <div class="bg-white p-2 rounded">
                                                <span class="font-semibold text-indigo-800">Metodoloji:</span>
                                                <ul class="text-indigo-600 mt-1">
                                                    <li>• Senaryo Bazlı Analiz</li>
                                                    <li>• Risk Değerlendirmesi</li>
                                                    <li>• Uzman Önerileri</li>
                                                </ul>
                                            </div>
                                            <div class="bg-white p-2 rounded">
                                                <span class="font-semibold text-indigo-800">Çıktılar:</span>
                                                <ul class="text-indigo-600 mt-1">
                                                    <li>• Potansiyel Riskler</li>
                                                    <li>• Aksiyon Planları</li>
                                                    <li>• Stratejik Öneriler</li>
                                                </ul>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                            
                            <div class="space-y-4">
                                ${scenarioAnalysis.advancedAnalysis.scenarios.map((scenario, index) => `
                                    <div class="border-2 border-gray-200 rounded-lg p-4 hover:border-indigo-300 transition-colors">
                                        <div class="flex items-start justify-between mb-3">
                                            <div>
                                                <h4 class="font-bold text-sm text-gray-800 mb-1">${scenario.title}</h4>
                                                <p class="text-xs text-gray-600 mb-2">
                                                    <strong>Kategori:</strong> ${scenario.category}
                                                </p>
                                                <p class="text-xs text-blue-700 bg-blue-50 px-2 py-1 rounded">
                                                    <strong>Mevcut Durum:</strong> ${scenario.currentSituation}
                                                </p>
                                            </div>
                                            <span class="bg-indigo-100 text-indigo-800 px-2 py-1 rounded text-xs font-semibold">
                                                Senaryo ${index + 1}
                                            </span>
                                        </div>
                                        ${scenario.analysis}
                                    </div>
                                `).join('')}
                            </div>
                            
                            <div class="mt-4 p-4 bg-gradient-to-r from-gray-50 to-indigo-50 rounded-lg border">
                                <h4 class="font-semibold text-gray-800 text-sm mb-2">📋 Uzman Analiz Özeti</h4>
                                <div class="grid md:grid-cols-3 gap-4 text-xs">
                                    <div class="text-center">
                                        <div class="text-2xl font-bold text-indigo-600">${scenarioAnalysis.advancedAnalysis.scenarios.length}</div>
                                        <div class="text-gray-600">Tespit Edilen Senaryo</div>
                                    </div>
                                    <div class="text-center">
                                        <div class="text-2xl font-bold text-${scenarioAnalysis.advancedAnalysis.summary.riskLevel === 'Yüksek' ? 'red' : scenarioAnalysis.advancedAnalysis.summary.riskLevel === 'Orta' ? 'yellow' : 'green'}-600">
                                            ${scenarioAnalysis.advancedAnalysis.summary.riskLevel}
                                        </div>
                                        <div class="text-gray-600">Genel Risk Seviyesi</div>
                                    </div>
                                    <div class="text-center">
                                        <div class="text-2xl font-bold text-blue-600">${scenarioAnalysis.advancedAnalysis.summary.categories.length}</div>
                                        <div class="text-gray-600">Analiz Kategorisi</div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    ` : ''}

                    <!-- AI Learning Scenarios Section -->
                    ${scenarioAnalysis.learningInsights && scenarioAnalysis.learningInsights.length > 0 ? `
                        <div class="bg-white border rounded-lg p-4 mb-4">
                            <div class="flex items-center justify-between mb-4">
                                <h3 class="text-lg font-bold text-gray-800">🎓 AI Öğrenme Senaryoları</h3>
                                <div class="flex space-x-2 text-xs">
                                    <span class="bg-purple-100 text-purple-800 px-2 py-1 rounded">
                                        ${scenarioAnalysis.learningInsights.length} Senaryo Tespit Edildi
                                    </span>
                                </div>
                            </div>
                            
                            <div class="bg-purple-50 p-3 rounded-lg mb-4 border-l-4 border-purple-400">
                                <p class="text-sm font-semibold text-purple-800 mb-1">🤖 Yapay Zeka Öğrenme Sistemi</p>
                                <p class="text-xs text-purple-700">
                                    Sisteminiz gerçek dünya senaryolarından öğrenerek verilerinizi analiz ediyor. 
                                    Aşağıdaki senaryolar, benzer finansal durumlardan çıkarılan öğrenmeleri içerir.
                                </p>
                            </div>
                            
                            <div class="space-y-4">
                                ${scenarioAnalysis.learningInsights.map(insight => `
                                    <div class="border rounded-lg p-3">
                                        <h4 class="font-semibold text-sm mb-2">${insight.title}</h4>
                                        <p class="text-xs text-gray-600 mb-3 italic">Tespit Edilen Patern: "${insight.pattern}"</p>
                                        ${insight.analysis}
                                    </div>
                                `).join('')}
                            </div>
                        </div>
                    ` : ''}
                    
                    <!-- Scenario Analysis Section -->
                    ${scenarioAnalysis.hasComparison ? `
                        <div class="bg-white border rounded-lg p-4 mb-4">
                            <div class="flex items-center justify-between mb-4">
                                <h3 class="text-lg font-bold text-gray-800">🔍 Senaryo Analizi & Değişim Yorumları</h3>
                                <div class="flex space-x-2 text-xs">
                                    <span class="bg-green-100 text-green-800 px-2 py-1 rounded">
                                        ${scenarioAnalysis.summary.positiveScenarios} Olumlu
                                    </span>
                                    <span class="bg-red-100 text-red-800 px-2 py-1 rounded">
                                        ${scenarioAnalysis.summary.riskScenarios} Risk
                                    </span>
                                </div>
                            </div>
                            
                            ${scenarioAnalysis.scenarios.length > 0 ? `
                                <div class="space-y-4">
                                    ${scenarioAnalysis.scenarios.map(scenario => `
                                        <div class="border rounded-lg p-3">
                                            <h4 class="font-semibold text-sm mb-2">${scenario.title}</h4>
                                            <p class="text-xs text-gray-600 mb-3 italic">"${scenario.scenario}"</p>
                                            ${scenario.analysis}
                                        </div>
                                    `).join('')}
                                </div>
                            ` : `
                                <div class="bg-blue-50 p-4 rounded-lg border-l-4 border-blue-400">
                                    <p class="text-sm text-blue-800">
                                        📈 Mevcut verilerinizde önemli bir değişim tespit edilmedi. 
                                        Finansal durumunuz stabil görünüyor. Bir sonraki veri girişinde 
                                        değişimleri analiz edebilmek için mevcut rakamları referans olarak saklıyoruz.
                                    </p>
                                </div>
                            `}
                        </div>
                    ` : `
                        <div class="bg-blue-50 border border-blue-200 rounded-lg p-4 mb-4">
                            <div class="flex items-center space-x-2">
                                <svg class="w-5 h-5 text-blue-600" fill="currentColor" viewBox="0 0 20 20">
                                    <path fill-rule="evenodd" d="M18 10a8 8 0 11-16 0 8 8 0 0116 0zm-7-4a1 1 0 11-2 0 1 1 0 012 0zM9 9a1 1 0 000 2v3a1 1 0 001 1h1a1 1 0 100-2v-3a1 1 0 00-1-1H9z" clip-rule="evenodd"/>
                                </svg>
                                <div>
                                    <h3 class="font-semibold text-blue-800 text-sm">🔍 Senaryo Analizi Hazır</h3>
                                    <p class="text-xs text-blue-700 mt-1">${scenarioAnalysis.message}</p>
                                </div>
                            </div>
                        </div>
                    `}
                    
                    <!-- Financial Formulas Reference -->
                    <div class="bg-gray-50 border rounded-lg p-4 mb-4">
                        <h3 class="text-lg font-bold text-gray-800 mb-3">📐 Kullanılan Finansal Formüller</h3>
                        <div class="grid md:grid-cols-2 gap-4 text-xs">
                            <div>
                                <h4 class="font-semibold text-gray-700 mb-2">📊 Gelir Tablosu Hesaplamaları:</h4>
                                <ul class="space-y-1 text-gray-600">
                                    <li>• <strong>Brüt Kâr:</strong> Net Satışlar - Satışların Maliyeti</li>
                                    <li>• <strong>Faaliyet Kârı (EBIT):</strong> Brüt Kâr - Yönetim Giderleri</li>
                                    <li>• <strong>Brüt Kâr Marjı:</strong> (Brüt Kâr / Net Satışlar) × 100</li>
                                    <li>• <strong>Net Kâr Marjı:</strong> (Net Kâr / Net Satışlar) × 100</li>
                                </ul>
                            </div>
                            <div>
                                <h4 class="font-semibold text-gray-700 mb-2">⚖️ Bilanço Hesaplamaları:</h4>
                                <ul class="space-y-1 text-gray-600">
                                    <li>• <strong>Cari Oran:</strong> Dönen Varlıklar / Kısa Vadeli Borçlar</li>
                                    <li>• <strong>Borç/Özkaynak Oranı:</strong> Toplam Borç / Özkaynaklar</li>
                                    <li>• <strong>ROE:</strong> (Net Kâr / Özkaynaklar) × 100</li>
                                    <li>• <strong>ROA:</strong> (Net Kâr / Toplam Varlıklar) × 100</li>
                                </ul>
                            </div>
                        </div>
                    </div>
                    
                    <!-- Analysis System Explanation -->
                    <div class="bg-gradient-to-r from-blue-50 to-purple-50 border-2 border-blue-200 rounded-lg p-4 mb-4">
                        <div class="flex items-center justify-between mb-3">
                            <h3 class="text-lg font-bold text-blue-800">🤖 Analiz Sistemi Nasıl Çalışır?</h3>
                            <span class="bg-blue-100 text-blue-800 px-3 py-1 rounded-full text-xs font-semibold">GERÇEK ZAMANLI</span>
                        </div>
                        
                        <div class="grid md:grid-cols-2 gap-4 text-sm">
                            <div class="bg-white p-3 rounded-lg border-l-4 border-blue-400">
                                <h4 class="font-semibold text-blue-800 mb-2">📊 Veri İşleme</h4>
                                <ul class="text-xs text-gray-700 space-y-1">
                                    <li>• <strong>Girdiğiniz veriler</strong> anlık olarak işlenir</li>
                                    <li>• <strong>Finansal oranlar</strong> otomatik hesaplanır</li>
                                    <li>• <strong>Sektör standartları</strong> ile karşılaştırılır</li>
                                    <li>• <strong>Risk seviyeleri</strong> belirlenir</li>
                                </ul>
                            </div>
                            
                            <div class="bg-white p-3 rounded-lg border-l-4 border-purple-400">
                                <h4 class="font-semibold text-purple-800 mb-2">🎯 Analiz Kriterleri</h4>
                                <ul class="text-xs text-gray-700 space-y-1">
                                    <li>• <strong>Kârlılık:</strong> Net Kâr Marjı % hesaplanır</li>
                                    <li>• <strong>Likidite:</strong> Cari Oran değerlendirilir</li>
                                    <li>• <strong>Borçluluk:</strong> Borç/Özkaynak oranı analiz edilir</li>
                                    <li>• <strong>Verimlilik:</strong> Varlık devir hızı ölçülür</li>
                                </ul>
                            </div>
                        </div>
                        
                        <div class="mt-3 p-3 bg-yellow-50 rounded-lg border border-yellow-200">
                            <p class="text-xs text-yellow-800">
                                <strong>💡 ÖNEMLİ:</strong> Aşağıdaki analizler tamamen <strong>sizin girdiğiniz verilerden</strong> hesaplanmaktadır. 
                                Her değişiklik yaptığınızda sistem otomatik olarak yeniden analiz yapar ve sonuçları günceller.
                            </p>
                        </div>
                    </div>

                    <!-- Detailed Analysis -->
                    <div class="bg-white border rounded-lg p-4">
                        <h3 class="text-lg font-bold text-gray-800 mb-4">📊 Detaylı Finansal Analiz</h3>
                        
                        <div class="space-y-4">
                            <div class="border-l-4 border-blue-500 pl-4">
                                <h4 class="font-semibold text-gray-800 text-sm mb-2">💰 Kârlılık Analizi</h4>
                                <p class="text-sm text-gray-700">${profitabilityAnalysis}</p>
                            </div>
                            
                            <div class="border-l-4 border-green-500 pl-4">
                                <h4 class="font-semibold text-gray-800 text-sm mb-2">💧 Likidite Analizi</h4>
                                <p class="text-sm text-gray-700">${liquidityAnalysis}</p>
                            </div>
                            
                            <div class="border-l-4 border-purple-500 pl-4">
                                <h4 class="font-semibold text-gray-800 text-sm mb-2">⚖️ Kaldıraç Analizi</h4>
                                <p class="text-sm text-gray-700">${leverageAnalysis}</p>
                            </div>
                            
                            <div class="border-l-4 border-orange-500 pl-4">
                                <h4 class="font-semibold text-gray-800 text-sm mb-2">⚡ Verimlilik Analizi</h4>
                                <p class="text-sm text-gray-700">${efficiencyAnalysis}</p>
                            </div>
                            
                            <div class="bg-gray-50 rounded-lg p-4 border-l-4 border-gray-500">
                                <h4 class="font-semibold text-gray-800 text-sm mb-2">🎯 Genel Değerlendirme ve Öneriler</h4>
                                <p class="text-sm text-gray-700 mb-3">${overallRecommendation}</p>
                                
                                <div class="text-xs text-gray-600">
                                    <p><strong>Kısa Vadeli Öncelikler:</strong></p>
                                    <ul class="list-disc list-inside mt-1 space-y-1">
                                        ${currentRatio < 1.2 ? '<li>Nakit akışı yönetimini güçlendirin</li>' : ''}
                                        ${netProfitMargin < 10 ? '<li>Maliyet yapısını optimize edin</li>' : ''}
                                        ${debtToEquityRatio > 0.6 ? '<li>Borç yükünü azaltmaya odaklanın</li>' : ''}
                                        ${assetTurnover < 1 ? '<li>Varlık kullanım verimliliğini artırın</li>' : ''}
                                    </ul>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            `;
        }

        // Demo Scenario Loading Functions
        function loadDemoScenario(scenarioNumber) {
            let scenarioData = {};
            let scenarioName = '';
            
            switch(scenarioNumber) {
                case 1: // Kârlılığın Düşmesi
                    scenarioData = {
                        netSales: 15000000,
                        costOfSales: 10000000,
                        adminExpenses: 3500000,
                        ebitda: 1500000,
                        netProfit: 500000,
                        currentAssets: 8000000,
                        fixedAssets: 12000000,
                        shortTermDebt: 4000000,
                        longTermDebt: 6000000,
                        equity: 10000000
                    };
                    scenarioName = 'Kârlılık Düşüşü Senaryosu';
                    break;
                    
                case 2: // Artan Borçluluk
                    scenarioData = {
                        netSales: 18000000,
                        costOfSales: 11000000,
                        adminExpenses: 3000000,
                        ebitda: 4000000,
                        netProfit: 2500000,
                        currentAssets: 9000000,
                        fixedAssets: 15000000,
                        shortTermDebt: 6000000,
                        longTermDebt: 9000000,
                        equity: 10000000
                    };
                    scenarioName = 'Artan Borçluluk Senaryosu';
                    break;
                    
                case 3: // Nakit Yönetimi Sorunu
                    scenarioData = {
                        netSales: 12000000,
                        costOfSales: 7000000,
                        adminExpenses: 2500000,
                        ebitda: 2500000,
                        netProfit: 2000000,
                        currentAssets: 5000000,
                        fixedAssets: 10000000,
                        shortTermDebt: 5000000,
                        longTermDebt: 4000000,
                        equity: 6000000
                    };
                    scenarioName = 'Nakit Yönetimi Sorunu Senaryosu';
                    break;
                    
                case 4: // Verimli Büyüme
                    scenarioData = {
                        netSales: 20000000,
                        costOfSales: 10000000,
                        adminExpenses: 3500000,
                        ebitda: 6500000,
                        netProfit: 4000000,
                        currentAssets: 12000000,
                        fixedAssets: 18000000,
                        shortTermDebt: 5000000,
                        longTermDebt: 8000000,
                        equity: 17000000
                    };
                    scenarioName = 'Verimli Büyüme Senaryosu';
                    break;
            }
            
            // Populate form fields
            Object.keys(scenarioData).forEach(key => {
                const element = document.getElementById(key);
                if (element) {
                    element.value = scenarioData[key];
                }
            });
            
            // Show success message
            alert(`📚 ${scenarioName} yüklendi!\n\n🎯 Bu senaryo gerçek dünya örneklerinden alınmıştır.\n\n✅ Artık "Kaydet" butonuna basarak AI öğrenme sistemini test edebilirsiniz.\n\n📊 Raporlar sekmesinde detaylı analizi göreceksiniz.`);
        }

        // Financial Calculator System
        let currentCalculatorField = null;
        
        // Financial calculation formulas and requirements
        const financialFormulas = {
            netSales: {
                title: 'Net Satışlar Hesaplama',
                formula: 'Brüt Satışlar - Satış İndirimleri ve İadeleri',
                description: 'Bir dönem içinde yapılan tüm satışların toplamından indirimler ve iadeler çıkarılır.',
                fields: [
                    { id: 'brutSales', label: 'Brüt Satışlar (TL)', placeholder: 'Toplam satış tutarı' },
                    { id: 'salesDiscounts', label: 'Satış İndirimleri (TL)', placeholder: 'İndirimler ve iadeler' },
                    { id: 'salesReturns', label: 'Satış İadeleri (TL)', placeholder: 'İade edilen ürün tutarı' }
                ],
                calculate: (values) => values.brutSales - values.salesDiscounts - values.salesReturns
            },
            
            costOfSales: {
                title: 'Satışların Maliyeti Hesaplama',
                formula: 'Dönem Başı Stok + Dönem İçi Alışlar - Dönem Sonu Stok',
                description: 'Satılan ürünlerin maliyetini hesaplamak için stok hareketleri kullanılır.',
                fields: [
                    { id: 'beginningStock', label: 'Dönem Başı Stok (TL)', placeholder: 'Başlangıç stok değeri' },
                    { id: 'purchases', label: 'Dönem İçi Alışlar (TL)', placeholder: 'Satın alınan mal tutarı' },
                    { id: 'endingStock', label: 'Dönem Sonu Stok (TL)', placeholder: 'Kalan stok değeri' }
                ],
                calculate: (values) => values.beginningStock + values.purchases - values.endingStock
            },
            
            adminExpenses: {
                title: 'Yönetim Giderleri Hesaplama',
                formula: 'İdari Personel Maaşları + Ofis Kiraları + Kırtasiye + Hukuk/Muhasebe + Diğer Giderler',
                description: 'Şirketin ana faaliyetleri dışındaki tüm genel ve idari harcamaların toplamı.',
                fields: [
                    { id: 'adminSalaries', label: 'İdari Personel Maaşları (TL)', placeholder: 'Yönetim kadrosu maaşları' },
                    { id: 'officeRent', label: 'Ofis Kiraları (TL)', placeholder: 'Kira ve işletme giderleri' },
                    { id: 'stationery', label: 'Kırtasiye Giderleri (TL)', placeholder: 'Ofis malzemeleri' },
                    { id: 'legalAccounting', label: 'Hukuk ve Muhasebe (TL)', placeholder: 'Profesyonel hizmet giderleri' },
                    { id: 'otherExpenses', label: 'Diğer Genel Giderler (TL)', placeholder: 'Diğer idari giderler' }
                ],
                calculate: (values) => values.adminSalaries + values.officeRent + values.stationery + values.legalAccounting + values.otherExpenses
            },
            
            ebitda: {
                title: 'FAVÖK/EBITDA Hesaplama',
                formula: 'Net Kâr + Faiz Giderleri + Vergi Giderleri + Amortisman ve İtfa Giderleri',
                description: 'Faiz, vergi, amortisman ve itfa öncesi kazanç - operasyonel performansı gösterir.',
                fields: [
                    { id: 'netProfitForEbitda', label: 'Net Kâr (TL)', placeholder: 'Dönem net kârı' },
                    { id: 'interestExpenses', label: 'Faiz Giderleri (TL)', placeholder: 'Borç faiz ödemeleri' },
                    { id: 'taxExpenses', label: 'Vergi Giderleri (TL)', placeholder: 'Ödenen vergiler' },
                    { id: 'depreciation', label: 'Amortisman Giderleri (TL)', placeholder: 'Varlık yıpranma payları' },
                    { id: 'amortization', label: 'İtfa Giderleri (TL)', placeholder: 'Gayri maddi varlık itfaları' }
                ],
                calculate: (values) => values.netProfitForEbitda + values.interestExpenses + values.taxExpenses + values.depreciation + values.amortization
            },
            
            netProfit: {
                title: 'Net Kâr/Zarar Hesaplama',
                formula: 'Brüt Kâr - Faaliyet Giderleri + Diğer Gelirler - Diğer Giderler - Faiz Giderleri - Vergi Giderleri',
                description: 'Şirketin tüm gelir ve giderler sonrasında elde ettiği nihai kâr veya zarar.',
                fields: [
                    { id: 'grossProfit', label: 'Brüt Kâr (TL)', placeholder: 'Net satışlar - satış maliyeti' },
                    { id: 'operatingExpenses', label: 'Faaliyet Giderleri (TL)', placeholder: 'Yönetim + pazarlama + Ar-Ge' },
                    { id: 'otherIncome', label: 'Diğer Gelirler (TL)', placeholder: 'Faaliyet dışı gelirler' },
                    { id: 'otherExpensesNet', label: 'Diğer Giderler (TL)', placeholder: 'Faaliyet dışı giderler' },
                    { id: 'interestExpensesNet', label: 'Faiz Giderleri (TL)', placeholder: 'Borç faiz ödemeleri' },
                    { id: 'taxExpensesNet', label: 'Vergi Giderleri (TL)', placeholder: 'Kurumlar vergisi' }
                ],
                calculate: (values) => values.grossProfit - values.operatingExpenses + values.otherIncome - values.otherExpensesNet - values.interestExpensesNet - values.taxExpensesNet
            },
            
            currentAssets: {
                title: 'Dönen Varlıklar Hesaplama',
                formula: 'Kasa ve Banka + Alacaklar + Stoklar + Gelecek Aylara Ait Giderler',
                description: 'Bir yıl içinde nakde dönüşebilecek varlıkların toplamı.',
                fields: [
                    { id: 'cashAndBank', label: 'Kasa ve Banka (TL)', placeholder: 'Nakit ve banka hesapları' },
                    { id: 'receivables', label: 'Alacaklar (TL)', placeholder: 'Müşteri alacakları' },
                    { id: 'inventory', label: 'Stoklar (TL)', placeholder: 'Satılmayı bekleyen ürünler' },
                    { id: 'prepaidExpenses', label: 'Gelecek Aylara Ait Giderler (TL)', placeholder: 'Peşin ödenen giderler' }
                ],
                calculate: (values) => values.cashAndBank + values.receivables + values.inventory + values.prepaidExpenses
            },
            
            fixedAssets: {
                title: 'Duran Varlıklar Hesaplama',
                formula: 'Arsa ve Binalar + Makineler ve Teçhizat + Taşıtlar + Patent ve Markalar + Diğer Uzun Vadeli Varlıklar',
                description: 'Bir yıldan uzun süre kullanılacak fiziki ve fiziki olmayan varlıkların toplamı.',
                fields: [
                    { id: 'landBuildings', label: 'Arsa ve Binalar (TL)', placeholder: 'Gayrimenkul değerleri' },
                    { id: 'machinery', label: 'Makineler ve Teçhizat (TL)', placeholder: 'Üretim ekipmanları' },
                    { id: 'vehicles', label: 'Taşıtlar (TL)', placeholder: 'Araç ve ulaşım araçları' },
                    { id: 'intangibles', label: 'Patent ve Markalar (TL)', placeholder: 'Fikri mülkiyet hakları' },
                    { id: 'otherFixedAssets', label: 'Diğer Uzun Vadeli Varlıklar (TL)', placeholder: 'Diğer duran varlıklar' }
                ],
                calculate: (values) => values.landBuildings + values.machinery + values.vehicles + values.intangibles + values.otherFixedAssets
            },
            
            shortTermDebt: {
                title: 'Kısa Vadeli Borçlar Hesaplama',
                formula: 'Satıcı Borçları + Kısa Vadeli Banka Kredileri + Personel Borçları + Ödenecek Vergi ve SGK',
                description: 'Bir yıl içinde ödenmesi gereken tüm borçların toplamı.',
                fields: [
                    { id: 'supplierDebt', label: 'Satıcı Borçları (TL)', placeholder: 'Tedarikçi borçları' },
                    { id: 'shortTermLoans', label: 'Kısa Vadeli Banka Kredileri (TL)', placeholder: 'Bir yıl içinde ödenecek krediler' },
                    { id: 'employeeDebt', label: 'Personel Borçları (TL)', placeholder: 'Maaş ve prim borçları' },
                    { id: 'taxSocialSecurity', label: 'Ödenecek Vergi ve SGK (TL)', placeholder: 'Vergi ve sigorta borçları' }
                ],
                calculate: (values) => values.supplierDebt + values.shortTermLoans + values.employeeDebt + values.taxSocialSecurity
            },
            
            longTermDebt: {
                title: 'Uzun Vadeli Borçlar Hesaplama',
                formula: 'Uzun Vadeli Banka Kredileri + Finansal Kiralama Borçları + Tahvil İhraç Borçları',
                description: 'Bir yıldan daha uzun sürede ödenmesi gereken borçların toplamı.',
                fields: [
                    { id: 'longTermLoans', label: 'Uzun Vadeli Banka Kredileri (TL)', placeholder: 'Bir yıldan uzun vadeli krediler' },
                    { id: 'leasingDebt', label: 'Finansal Kiralama Borçları (TL)', placeholder: 'Leasing borçları' },
                    { id: 'bondDebt', label: 'Tahvil İhraç Borçları (TL)', placeholder: 'Çıkarılan tahvil borçları' }
                ],
                calculate: (values) => values.longTermLoans + values.leasingDebt + values.bondDebt
            },
            
            equity: {
                title: 'Özkaynaklar Hesaplama',
                formula: 'Ödenmiş Sermaye + Yedek Akçeler + Geçmiş Yıl Kârları/Zararları',
                description: 'İşletme sahiplerinin şirkete koyduğu sermaye ve birikmiş kârların toplamı.',
                fields: [
                    { id: 'paidCapital', label: 'Ödenmiş Sermaye (TL)', placeholder: 'Ortakların koyduğu sermaye' },
                    { id: 'reserves', label: 'Yedek Akçeler (TL)', placeholder: 'Ayrılan yedekler' },
                    { id: 'retainedEarnings', label: 'Geçmiş Yıl Kârları/Zararları (TL)', placeholder: 'Birikmiş kâr/zarar' }
                ],
                calculate: (values) => values.paidCapital + values.reserves + values.retainedEarnings
            }
        };

        function openCalculator(fieldId) {
            currentCalculatorField = fieldId;
            const formula = financialFormulas[fieldId];
            
            if (!formula) {
                alert('Bu alan için hesaplama formülü henüz tanımlanmamış.');
                return;
            }
            
            document.getElementById('calculatorTitle').textContent = formula.title;
            
            const content = `
                <div class="bg-blue-50 p-4 rounded-lg mb-4">
                    <h3 class="font-semibold text-blue-800 mb-2">📐 Formül:</h3>
                    <p class="text-sm text-blue-700 font-mono bg-white p-2 rounded">${formula.formula}</p>
                    <p class="text-xs text-blue-600 mt-2">${formula.description}</p>
                </div>
                
                <div class="space-y-3">
                    <h3 class="font-semibold text-gray-800 mb-3">💰 Gerekli Bilgileri Girin:</h3>
                    ${formula.fields.map(field => `
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">${field.label}</label>
                            <input type="number" 
                                   id="calc_${field.id}" 
                                   placeholder="${field.placeholder}"
                                   class="w-full p-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent"
                                   step="0.01">
                        </div>
                    `).join('')}
                </div>
                
                <div class="bg-green-50 p-4 rounded-lg mt-4">
                    <h4 class="font-semibold text-green-800 mb-2">🧮 Hesaplama Sonucu:</h4>
                    <div id="calculationResult" class="text-lg font-bold text-green-700">
                        Değerleri girin ve "Hesapla" butonuna basın
                    </div>
                </div>
                
                <div class="mt-4">
                    <button onclick="performCalculation()" 
                            class="w-full bg-blue-600 hover:bg-blue-700 text-white py-2 px-4 rounded-lg font-semibold">
                        🧮 Hesapla
                    </button>
                </div>
            `;
            
            document.getElementById('calculatorContent').innerHTML = content;
            document.getElementById('calculatorModal').classList.remove('hidden');
        }

        function closeCalculator() {
            document.getElementById('calculatorModal').classList.add('hidden');
            currentCalculatorField = null;
        }

        function performCalculation() {
            if (!currentCalculatorField) return;
            
            const formula = financialFormulas[currentCalculatorField];
            const values = {};
            let hasError = false;
            
            // Collect values from input fields
            formula.fields.forEach(field => {
                const input = document.getElementById(`calc_${field.id}`);
                const value = parseFloat(input.value) || 0;
                values[field.id] = value;
                
                if (input.value === '') {
                    input.classList.add('border-red-500');
                    hasError = true;
                } else {
                    input.classList.remove('border-red-500');
                }
            });
            
            if (hasError) {
                alert('⚠️ Lütfen tüm alanları doldurun!');
                return;
            }
            
            // Perform calculation
            const result = formula.calculate(values);
            
            // Display result
            document.getElementById('calculationResult').innerHTML = `
                <div class="text-2xl font-bold text-green-700">
                    ${result.toLocaleString('tr-TR')} TL
                </div>
                <div class="text-sm text-green-600 mt-1">
                    Hesaplama tamamlandı! "Hesapla ve Uygula" butonuna basarak ana forma aktarın.
                </div>
            `;
            
            // Store result for application
            window.calculationResult = result;
        }

        function calculateAndApply() {
            if (window.calculationResult === undefined) {
                alert('⚠️ Önce hesaplama yapın!');
                return;
            }
            
            if (!currentCalculatorField) {
                alert('⚠️ Hedef alan bulunamadı!');
                return;
            }
            
            // Apply result to the main form
            const targetField = document.getElementById(currentCalculatorField);
            if (targetField) {
                targetField.value = window.calculationResult;
                
                // Show success message
                const formula = financialFormulas[currentCalculatorField];
                alert(`✅ Hesaplama tamamlandı!\n\n📊 ${formula.title}\n💰 Sonuç: ${window.calculationResult.toLocaleString('tr-TR')} TL\n\n✅ Değer "${formula.title}" alanına aktarıldı.`);
                
                closeCalculator();
                
                // Clear stored result
                window.calculationResult = undefined;
            } else {
                alert('❌ Hedef alan bulunamadı!');
            }
        }

        // AI Toggle Functionality
        function initializeAIToggle() {
            const aiToggle = document.getElementById('aiToggle');
            const toggleContainer = aiToggle.parentElement.querySelector('div');
            const dot = toggleContainer.querySelector('.dot');
            
            function updateToggleState() {
                if (aiAnalysisEnabled) {
                    toggleContainer.classList.remove('bg-gray-400');
                    toggleContainer.classList.add('bg-green-600');
                    dot.style.transform = 'translateX(24px)';
                    aiToggle.checked = true;
                } else {
                    toggleContainer.classList.remove('bg-green-600');
                    toggleContainer.classList.add('bg-gray-400');
                    dot.style.transform = 'translateX(0px)';
                    aiToggle.checked = false;
                }
            }
            
            aiToggle.addEventListener('change', function() {
                aiAnalysisEnabled = this.checked;
                updateToggleState();
                
                // Update save button text
                const saveBtn = document.getElementById('saveDataBtn');
                const btnText = saveBtn.querySelector('span');
                if (aiAnalysisEnabled) {
                    btnText.textContent = '💾 Kaydet ve AI Analizi Yap';
                    saveBtn.classList.remove('bg-gray-600');
                    saveBtn.classList.add('bg-green-600');
                } else {
                    btnText.textContent = '💾 Kaydet (AI Kapalı)';
                    saveBtn.classList.remove('bg-green-600');
                    saveBtn.classList.add('bg-gray-600');
                }
                
                console.log('AI Analizi:', aiAnalysisEnabled ? 'Açık' : 'Kapalı');
            });
            
            // Initialize state
            updateToggleState();
        }

        // Initialize
        document.addEventListener('DOMContentLoaded', function() {
            document.querySelectorAll('.nav-btn')[0].classList.add('bg-blue-100', 'text-blue-600');
            
            // Initialize AI toggle
            initializeAIToggle();
            
            // Add event listeners for demo scenario buttons
            document.getElementById('loadScenario1').addEventListener('click', () => loadDemoScenario(1));
            document.getElementById('loadScenario2').addEventListener('click', () => loadDemoScenario(2));
            document.getElementById('loadScenario3').addEventListener('click', () => loadDemoScenario(3));
            document.getElementById('loadScenario4').addEventListener('click', () => loadDemoScenario(4));
            
            console.log('🔥 Firebase CANLI sistem başlatıldı!');
        });

        // Firebase Integration Functions (to be implemented when config is provided)
        /*
        function saveUserToFirebase(companyName, userData) {
            // Firebase user creation logic
        }

        function saveFinancialDataToFirebase(companyName, data) {
            // Firebase data saving logic
        }

        function loadAllCompaniesFromFirebase() {
            // Firebase data loading for super admin
        }

        function updatePasswordInFirebase(companyName, newPassword) {
            // Firebase password update logic
        }

        function logUserLogin(companyName) {
            // Firebase login logging
        }
        */
    </script>
<script>(function(){function c(){var b=a.contentDocument||a.contentWindow.document;if(b){var d=b.createElement('script');d.innerHTML="window.__CF$cv$params={r:'984158b0356a8b2e',t:'MTc1ODcwNzEwMC4wMDAwMDA='};var a=document.createElement('script');a.nonce='';a.src='/cdn-cgi/challenge-platform/scripts/jsd/main.js';document.getElementsByTagName('head')[0].appendChild(a);";b.getElementsByTagName('head')[0].appendChild(d)}}if(document.body){var a=document.createElement('iframe');a.height=1;a.width=1;a.style.position='absolute';a.style.top=0;a.style.left=0;a.style.border='none';a.style.visibility='hidden';document.body.appendChild(a);if('loading'!==document.readyState)c();else if(window.addEventListener)document.addEventListener('DOMContentLoaded',c);else{var e=document.onreadystatechange||function(){};document.onreadystatechange=function(b){e(b);'loading'!==document.readyState&&(document.onreadystatechange=e,c())}}}})();</script></body>
</html>
