<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Finansal Analiz</title>
    
    <!-- Firebase SDK (Modular) -->
    <script type="module">
      // Import the functions you need from the SDKs you need
      import { initializeApp } from "https://www.gstatic.com/firebasejs/9.23.0/firebase-app.js";
      import { getAuth, GoogleAuthProvider, signInWithPopup, onAuthStateChanged, signOut } from "https://www.gstatic.com/firebasejs/9.23.0/firebase-auth.js";
      import { getFirestore, doc, getDoc, setDoc, serverTimestamp, collection, getDocs, writeBatch, query, where, orderBy, limit, startAfter, endBefore, limitToLast, deleteDoc, updateDoc, FieldValue } from "https://www.gstatic.com/firebasejs/9.23.0/firebase-firestore.js";
      import { getDatabase, ref, onValue, off, set as rtdbSet, serverTimestamp as rtdbServerTimestamp, remove as rtdbRemove } from "https://www.gstatic.com/firebasejs/9.23.0/firebase-database.js";
      import { getAnalytics } from "https://www.gstatic.com/firebasejs/9.23.0/firebase-analytics.js";
      // Note: GoogleGenerativeAI is not a standard Firebase module, keeping its import separate if needed.
      // import { GoogleGenerativeAI } from "https://esm.run/@google/generative-ai";

        // Number formatting function
        function formatNumber(input) {
            // Mevcut cursor pozisyonunu al
            const cursorPosition = input.selectionStart;
            let value = input.value;
            
            // Sadece sayıları ve eksi işaretini koru
            let numericValue = value.replace(/[^0-9-]/g, '');
            
            // Boş ise çık
            if (!numericValue) {
                input.value = '';
                return;
            }
            
            // Eksi işareti kontrolü
            let isNegative = false;
            if (numericValue.startsWith('-')) {
                isNegative = true;
                numericValue = numericValue.substring(1);
            }
            
            // Sayıyı formatla (binlik ayraç)
            if (numericValue) {
                const formattedValue = parseInt(numericValue).toLocaleString('tr-TR');
                input.value = isNegative ? '-' + formattedValue : formattedValue;
            }
            
            // Cursor pozisyonunu güncelle
            const newLength = input.value.length;
            const lengthDiff = newLength - value.length;
            const newCursorPosition = Math.max(0, cursorPosition + lengthDiff);
            
            setTimeout(() => {
                input.setSelectionRange(newCursorPosition, newCursorPosition);
            }, 0);
        }

        // Get numeric value from formatted input
        function getNumericValue(input) {
            if (!input.value) return 0;
            return parseFloat(input.value.replace(/[^0-9-]/g, '')) || 0;
        }

        // Firebase Configuration - UPDATED
        const firebaseConfig = {
            apiKey: "AIzaSyBh-VdQ1mLeI7lUd8f_OFcekAfmNp2G6gk",
            authDomain: "analizprox-62e8d.firebaseapp.com",
            projectId: "analizprox-62e8d",
            databaseURL: "https://analizprox-62e8d-default-rtdb.europe-west1.firebasedatabase.app",
            storageBucket: "analizprox-62e8d.appspot.com",
            messagingSenderId: "230113746168",
            appId: "1:230113746168:web:01d6c96494553e250f2f0d",
            measurementId: "G-KDMEE9CMB8"
        };
        
        // Firebase'i başlat (MODULAR SYNTAX)
        const app = initializeApp(firebaseConfig);
        const auth = getAuth(app);
        const db = getFirestore(app);
        const rtdb = getDatabase(app);
        const analytics = getAnalytics(app);
        const googleProvider = new GoogleAuthProvider();
        
        // Google provider ayarları - ENHANCED
        googleProvider.addScope('email');
        googleProvider.addScope('profile');
        googleProvider.setCustomParameters({
            'prompt': 'select_account',
            'hd': '' // Tüm domainlere izin ver
        });
        
        console.log('🔥 Firebase başlatıldı:', firebaseConfig.projectId);

        // Global Variables
        let currentUser = null;
        let isSuperAdmin = false;
        let financialData = {};
        let charts = {};
        let userProfile = null;
        let aiAnalysisEnabled = true;
        let previousData = null;
        let firebaseConnected = false;

        // Firebase Connection Test - ENHANCED
        async function testFirebaseConnection() {
            try {
                console.log('🔄 Firebase bağlantısı test ediliyor...');
                
                // Test Firestore
                const testDocRef = doc(db, '_test', 'connection');
                await setDoc(testDocRef, {
                    timestamp: serverTimestamp(),
                    test: true,
                    userAgent: navigator.userAgent,
                    url: window.location.href,
                    version: '2.0'
                });
                
                // Test Realtime Database
                const testRef = ref(rtdb, '_test/connection');
                await rtdbSet(testRef, {
                    timestamp: rtdbServerTimestamp(),
                    test: true,
                    userAgent: navigator.userAgent,
                    url: window.location.href,
                    version: '2.0'
                });
                
                firebaseConnected = true;
                console.log('✅ Firebase FULL bağlantı başarılı (Auth + Firestore + Realtime DB)');
                
                // Test Authentication
                onAuthStateChanged(auth, (user) => {
                    if (user) {
                        console.log('🔐 Auth durumu: Giriş yapılmış -', user.email);
                    } else {
                        console.log('🔐 Auth durumu: Çıkış yapılmış');
                    }
                });
                
                // Clean up test data
                setTimeout(async () => {
                    try {
                        await deleteDoc(testDocRef);
                        await rtdbRemove(testRef);
                        console.log('🧹 Firebase test verileri temizlendi');
                    } catch (e) { 
                        console.log('Test cleanup hatası (normal):', e.message); 
                    }
                }, 5000);
                
                // Success notification
                setTimeout(() => {
                    if (firebaseConnected) {
                        console.log('🌐 Firebase tamamen aktif - Tüm servisler çalışıyor');
                    }
                }, 1000);
                
            } catch (error) {
                firebaseConnected = false;
                console.error('🔴 Firebase bağlantı hatası:', error);
                console.warn('💾 Offline mod aktif - localStorage kullanılacak');
                
                // Offline notification
                setTimeout(() => {
                    console.log('� Uygulama offline modda çalışıyor');
                }, 1000);
            }
        }

        // Gemini AI Configuration - UPDATED
        const GEMINI_API_KEY = 'AIzaSyAF8ZcI4lYPjnojma094lo_orSfX8I9Fh8';
        const GEMINI_API_URL = 'https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent';
        
        console.log('🤖 Akça Pro X AI hazır');

        // DOM Elements
        const loginScreen = document.getElementById('loginScreen');
        const dashboard = document.getElementById('dashboard');
        const googleSignInBtn = document.getElementById('googleSignInBtn');

        const accountSetupModal = document.getElementById('accountSetupModal');
        const accountSetupForm = document.getElementById('accountSetupForm');

        // Firebase Auth State Observer
        onAuthStateChanged(auth, async (user) => {
            if (user) {
                console.log('🔐 Kullanıcı oturum açtı:', user.email);
                currentUser = user.uid;
                
                try {
                    const userDocRef = doc(db, 'users', user.uid);
                    const userDoc = await getDoc(userDocRef);
                    if (userDoc.exists()) {
                        userProfile = userDoc.data();
                        document.getElementById('currentCompany').textContent = userProfile.companyName || 'Bilinmeyen Şirket';
                        
                        // Süper admin kontrolü
                        if (user.email === 'admin@finansalanaliz.com' || userProfile.companyName === 'Süper Admin') {
                            isSuperAdmin = true;
                            document.getElementById('superAdminTab').classList.remove('hidden');
                        }
                        
                        showDashboard();
                        await loadUserData();
                        
                        // Son giriş zamanını güncelle
                        await updateDoc(userDocRef, {
                            lastLogin: serverTimestamp(),
                            email: user.email,
                            displayName: user.displayName,
                            photoURL: user.photoURL
                        });
                        
                    } else {
                        // Yeni kullanıcı
                        console.log('🆕 Yeni kullanıcı, hesap kurulumu gerekli');
                        showAccountSetup();
                    }
                } catch (error) {
                    console.error('Kullanıcı profil hatası:', error);
                    alert('❌ Kullanıcı bilgileri yüklenirken hata oluştu: ' + error.message);
                }
            } else {
                console.log('👋 Kullanıcı oturumu kapattı');
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

        // Google Sign In - GERÇEK GOOGLE OAUTH (Terms zorunlu)
        const termsAccepted = document.getElementById('termsAccepted');
        const googleHint = document.getElementById('googleHint');
        googleSignInBtn.addEventListener('click', async () => {
            if (!termsAccepted.checked) {
                alert('⚠️ Lütfen önce şartları okuyup onaylayın!');
                termsAccepted.focus();
                googleHint && googleHint.classList.remove('hidden');
                return;
            }
            
            try {
                googleSignInBtn.disabled = true;
                googleSignInBtn.innerHTML = '<span>🔄 Google\'a bağlanıyor...</span>';
                
                // Gerçek Google sign in
                const result = await signInWithPopup(auth, googleProvider);
                const user = result.user;
                window.__googleSignedIn = true;
                
                console.log('✅ Google giriş başarılı:', user.email);
                
                // Firebase'de kullanıcı profili kontrol et
                const userDocRef = doc(db, 'users', user.uid);
                const userDoc = await getDoc(userDocRef);
                
                if (!userDoc.exists) {
                    // Yeni kullanıcı - hesap kurulumu gerekli
                    showAccountSetup();
                } else {
                    // Mevcut kullanıcı - direkt dashboard
                    userProfile = userDoc.data();
                    document.getElementById('currentCompany').textContent = userProfile.companyName;
                    showDashboard();
                    await loadUserData();
                }
                
            } catch (error) {
                console.error('Google giriş hatası:', error);
                let errorMessage = 'Google giriş hatası: ';
                
                if (error.code === 'auth/popup-closed-by-user') {
                    errorMessage = 'Giriş iptal edildi. Lütfen tekrar deneyin.';
                } else if (error.code === 'auth/network-request-failed') {
                    errorMessage = 'İnternet bağlantısı hatası. Bağlantınızı kontrol edin.';
                } else if (error.code === 'auth/popup-blocked') {
                    errorMessage = 'Pop-up engellendi. Tarayıcınızda pop-up\'ları etkinleştirin.';
                } else {
                    errorMessage += error.message;
                }
                
                alert('❌ ' + errorMessage);
                
            } finally {
                googleSignInBtn.disabled = false;
                googleSignInBtn.innerHTML = '<span>🔑 Google ile Giriş</span>';
            }
        });

        // Terms kutusu Google uyarı ipucu
        termsAccepted.addEventListener('change', () => {
            if (termsAccepted.checked) {
                googleHint && googleHint.classList.remove('hidden');
            } else {
                googleHint && googleHint.classList.add('hidden');
            }
        });

        // Company Registration
        document.getElementById('companyRegisterForm').addEventListener('submit', async (e) => {
            e.preventDefault();
            // Zorunluluklar: Terms + Google login
            if (!termsAccepted.checked) {
                alert('⚠️ Lütfen şartları okuyup onaylayın!');
                return;
            }
            if (!auth.currentUser && !window.__googleSignedIn) {
                alert('⚠️ Önce Google ile giriş yapınız!');
                return;
            }
            
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
                    if (firebaseConnected) {
                        console.log('🏢 Şirket verisi Firebase\'e kaydediliyor...');
                        const companyDocRef = doc(db, 'companies', companyId);
                        const companyRtdbRef = ref(rtdb, `companies/${companyId}`);
                        // Save to both Firestore and Realtime Database
                        await setDoc(companyDocRef, companyData);
                        await rtdbSet(companyRtdbRef, companyData);
                        console.log('✅ Şirket Firebase\'e kaydedildi (Firestore + Realtime DB)');
                    }
                } catch (error) {
                    console.warn('Firebase şirket kayıt hatası, sadece localStorage kullanıldı:', error.message);
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
            // Zorunluluklar: Terms + Google login
            if (!termsAccepted.checked) {
                alert('⚠️ Lütfen şartları okuyup onaylayın!');
                return;
            }
            if (!auth.currentUser && !window.__googleSignedIn) {
                alert('⚠️ Önce Google ile giriş yapınız!');
                return;
            }
            
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
            if (password.length < 6) {
                alert('⚠️ Şifre en az 6 karakter olmalıdır!');
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
                
                // Try Firebase first (Firestore then Realtime DB)
                try {
                    if (firebaseConnected) {
                        // Try Firestore first
                        const companyDocRef = doc(db, 'companies', companyId);
                        const docSnap = await getDoc(companyDocRef);
                        if (docSnap.exists()) {
                            companyData = docSnap.data();
                        } else {
                            // Try Realtime Database
                            const rtdbRef = ref(rtdb, `companies/${companyId}`);
                            const snapshot = await get(rtdbRef);
                            if (snapshot.exists()) {
                                companyData = snapshot.val();
                            }
                        }
                    }
                } catch (error) {
                    console.warn('Firebase company giriş hatası:', error.message);
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
                    if (firebaseConnected) {
                        const companyDocRef = doc(db, 'companies', companyId);
                        const companyRtdbRef = ref(rtdb, `companies/${companyId}/lastLogin`);
                        // Update in both Firestore and Realtime Database
                        await updateDoc(companyDocRef, {
                            lastLogin: serverTimestamp()
                        });
                        await rtdbSet(companyRtdbRef, rtdbServerTimestamp());
                    }
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
                if (!user) {
                    alert('❌ Oturum hatası, lütfen tekrar giriş yapın.');
                    return;
                }
                
                const userProfileData = {
                    companyName, 
                    email: user.email, 
                    displayName: user.displayName,
                    photoURL: user.photoURL,
                    sector, 
                    employeeCount,
                    createdAt: serverTimestamp(),
                    lastLogin: serverTimestamp(),
                    isActive: true,
                    loginMethod: 'google'
                };
                
                // Firestore'a kaydet
                const userDocRef = doc(db, 'users', user.uid);
                await setDoc(userDocRef, userProfileData);
                
                // Realtime Database'e de kaydet
                if (firebaseConnected) {
                    const userRtdbRef = ref(rtdb, `users/${user.uid}`);
                    await rtdbSet(userRtdbRef, {
                        ...userProfileData,
                        createdAt: rtdbServerTimestamp(),
                        lastLogin: rtdbServerTimestamp()
                    });
                }
                
                currentUser = user.uid;
                userProfile = userProfileData;
                document.getElementById('currentCompany').textContent = companyName;
                
                showDashboard();
                alert(`🎉 Hoş geldiniz!\n\n🏢 ${companyName}\n📧 ${user.email}\n\nHesabınız başarıyla oluşturuldu!`);
                
            } catch (error) {
                console.error('Account setup error:', error);
                alert('❌ Hesap oluşturma hatası: ' + error.message);
            }
        });



        // Logout
        document.getElementById('logoutBtn').addEventListener('click', async function() {
            if (confirm('🔐 Çıkış yapmak istediğinizden emin misiniz?')) {
                try {
                    await signOut(auth);
                    console.log('👋 Güvenli çıkış yapıldı');
                } catch (error) {
                    console.error('Logout error:', error);
                    alert('❌ Çıkış hatası: ' + error.message);
                }
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
                netSales: getNumericValue(document.getElementById('netSales')) || 0,
                costOfSales: getNumericValue(document.getElementById('costOfSales')) || 0,
                adminExpenses: getNumericValue(document.getElementById('adminExpenses')) || 0,
                ebitda: getNumericValue(document.getElementById('ebitda')) || 0,
                netProfit: getNumericValue(document.getElementById('netProfit')) || 0,
                currentAssets: getNumericValue(document.getElementById('currentAssets')) || 0,
                fixedAssets: getNumericValue(document.getElementById('fixedAssets')) || 0,
                shortTermDebt: getNumericValue(document.getElementById('shortTermDebt')) || 0,
                longTermDebt: getNumericValue(document.getElementById('longTermDebt')) || 0,
                equity: getNumericValue(document.getElementById('equity')) || 0,
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
                    
                    // Try Firebase first (both Firestore and Realtime DB)
                    try {
                        if (firebaseConnected) {
                            console.log('💾 Veriler Firebase\'e kaydediliyor...');
                            
                            // Save to Firestore
                            const docRef = doc(db, 'financialData', currentUser);
                            await setDoc(docRef, {
                                ...saveData,
                                lastUpdated: serverTimestamp()
                            });
                            
                            // Save to Realtime Database as backup
                            const rtdbRef = ref(rtdb, `financialData/${currentUser}`);
                            await rtdbSet(rtdbRef, saveData);
                            
                            console.log('✅ Firebase (Firestore + Realtime DB) kayıt başarılı');
                            
                            // Başarı bildirimi ekle
                            setTimeout(() => {
                                console.log('☁️ Verileriniz güvenle cloud\'da saklandı');
                            }, 500);
                        } else {
                            throw new Error('Firebase bağlantısı yok');
                        }
                    } catch (error) {
                        console.warn('⚠️ Firebase hatası, localStorage kullanılıyor:', error.message);
                        // Fallback to localStorage
                        localStorage.setItem(`financialData_${currentUser}`, JSON.stringify(saveData));
                        console.log('💾 Veriler yerel olarak kaydedildi (localStorage)');
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
                
                // Try Firebase first (Firestore then Realtime DB)
                try {
                    if (firebaseConnected) {
                        // Try Firestore first
                        const docRef = doc(db, 'financialData', currentUser);
                        const docSnap = await getDoc(docRef);
                        if (docSnap.exists()) {
                            data = docSnap.data();
                            console.log('📊 Firestore\'dan veri yüklendi');
                        } else {
                            // Try Realtime Database
                            const rtdbRef = ref(rtdb, `financialData/${currentUser}`);
                            const snapshot = await get(rtdbRef);
                            if (snapshot.exists()) {
                                data = snapshot.val();
                                console.log('📊 Realtime DB\'dan veri yüklendi');
                            }
                        }
                    }
                } catch (error) {
                    console.warn('Firebase yükleme hatası:', error.message);
                    // Fallback to localStorage
                    const localData = localStorage.getItem(`financialData_${currentUser}`);
                    if (localData) {
                        data = JSON.parse(localData);
                        console.log('📊 localStorage\'dan veri yüklendi');
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
                    // Sayıyı formatla ve inputa yaz
                    element.value = data[key].toLocaleString('tr-TR');
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
                    netSales: getNumericValue(document.getElementById('netSales')) || 0,
                    costOfSales: getNumericValue(document.getElementById('costOfSales')) || 0,
                    adminExpenses: getNumericValue(document.getElementById('adminExpenses')) || 0,
                    ebitda: getNumericValue(document.getElementById('ebitda')) || 0,
                    netProfit: getNumericValue(document.getElementById('netProfit')) || 0,
                    currentAssets: getNumericValue(document.getElementById('currentAssets')) || 0,
                    fixedAssets: getNumericValue(document.getElementById('fixedAssets')) || 0,
                    shortTermDebt: getNumericValue(document.getElementById('shortTermDebt')) || 0,
                    longTermDebt: getNumericValue(document.getElementById('longTermDebt')) || 0,
                    equity: getNumericValue(document.getElementById('equity')) || 0
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
                // Gemini 1.5 Flash model kullan
                const response = await fetch(`${GEMINI_API_URL}?key=${GEMINI_API_KEY}`, {
                    method: 'POST',
                    headers: { 
                        'Content-Type': 'application/json',
                        'Accept': 'application/json'
                    },
                    body: JSON.stringify({
                        contents: [{ 
                            parts: [{ text: prompt }] 
                        }],
                        generationConfig: { 
                            temperature: 0.7, 
                            maxOutputTokens: 2048,
                            topP: 0.8,
                            topK: 40
                        },
                        safetySettings: [
                            {
                                category: "HARM_CATEGORY_HARASSMENT",
                                threshold: "BLOCK_MEDIUM_AND_ABOVE"
                            },
                            {
                                category: "HARM_CATEGORY_HATE_SPEECH", 
                                threshold: "BLOCK_MEDIUM_AND_ABOVE"
                            }
                        ]
                    })
                });

                if (!response.ok) {
                    const errorText = await response.text();
                    console.error('Akça Pro X AI Error:', response.status, errorText);
                    throw new Error(`AI API Hatası (${response.status}): ${errorText}`);
                }
                
                const data = await response.json();
                
                if (data.candidates && data.candidates[0] && data.candidates[0].content) {
                    const analysisText = data.candidates[0].content.parts[0].text;
                    console.log('✅ Akça Pro X AI analizi başarılı');
                    return analysisText;
                } else {
                    console.error('Akça Pro X AI Response Error:', data);
                    throw new Error('AI yanıtı format hatası');
                }
                
            } catch (error) {
                console.error('Akça Pro X AI Analysis Error:', error);
                
                // Fallback analysis
                return `⚠️ Akça Pro X AI analizi şu anda kullanılamıyor (${error.message})
                
📊 MANUEL FİNANSAL DEĞERLENDİRME:
Akça Pro X AI geçici olarak kullanılamadığında manuel analiz yapılır.

💰 KARLILIK DEĞERLENDİRMESİ:
• Net kâr marjı %15'in üzerinde ise mükemmel performans
• %10-15 arası ise iyi seviyede kârlılık
• %5-10 arası ise ortalama performans
• %5'in altında ise dikkat gereken alan

💧 LİKİDİTE DURUMU:
• Cari oran 2'nin üzerinde ise güçlü likidite
• 1.5-2 arası ise sağlıklı likidite
• 1-1.5 arası ise kabul edilebilir seviye
• 1'in altında ise likidite riski mevcut

⚖️ BORÇLULUK ANALİZİ:
• Borç/Özkaynak oranı 0.5'in altında ise düşük borçluluk
• 0.5-1 arası ise orta seviye risk
• 1'in üzerinde ise yüksek borçluluk riski

🔧 Detaylı Akça Pro X AI analizi için servisin yeniden çalışması bekleniyor...
📞 Destek: AI servisi restore ediliyor, lütfen tekrar deneyin.`;
            }
        }

        async function generateAIAnalysis(financialData) {
            const grossProfit = (financialData.netSales - financialData.costOfSales) || 0;
            const grossMargin = ((grossProfit / financialData.netSales) * 100) || 0;
            const netMargin = (financialData.netProfit / financialData.netSales * 100) || 0;
            const totalAssets = (financialData.currentAssets + financialData.fixedAssets) || 0;
            const totalDebt = (financialData.shortTermDebt + financialData.longTermDebt) || 0;
            const currentRatio = financialData.currentAssets / (financialData.shortTermDebt || 1);
            const debtToEquity = totalDebt / (financialData.equity || 1);
            const assetTurnover = financialData.netSales / (totalAssets || 1);
            const roe = (financialData.netProfit / financialData.equity * 100) || 0;
            const roa = (financialData.netProfit / totalAssets * 100) || 0;
            const ebitdaMargin = (financialData.ebitda / financialData.netSales * 100) || 0;

            const prompt = `
Sen bir uzman finansal analist ve danışmansın. Aşağıdaki şirket verilerini profesyonel olarak analiz et:

📊 FİNANSAL VERİLER:
• Net Satışlar: ${financialData.netSales.toLocaleString('tr-TR')} TL
• Satış Maliyeti: ${financialData.costOfSales.toLocaleString('tr-TR')} TL
• Yönetim Giderleri: ${financialData.adminExpenses.toLocaleString('tr-TR')} TL
• FAVÖK/EBITDA: ${financialData.ebitda.toLocaleString('tr-TR')} TL
• Net Kâr/Zarar: ${financialData.netProfit.toLocaleString('tr-TR')} TL
• Dönen Varlıklar: ${financialData.currentAssets.toLocaleString('tr-TR')} TL
• Duran Varlıklar: ${financialData.fixedAssets.toLocaleString('tr-TR')} TL
• Kısa Vadeli Borçlar: ${financialData.shortTermDebt.toLocaleString('tr-TR')} TL
• Uzun Vadeli Borçlar: ${financialData.longTermDebt.toLocaleString('tr-TR')} TL
• Özkaynak: ${financialData.equity.toLocaleString('tr-TR')} TL

📈 HESAPLANAN ORANLAR:
• Brüt Kâr Marjı: %${grossMargin.toFixed(2)}
• Net Kâr Marjı: %${netMargin.toFixed(2)}
• FAVÖK Marjı: %${ebitdaMargin.toFixed(2)}
• Cari Oran: ${currentRatio.toFixed(2)}
• Borç/Özkaynak Oranı: ${debtToEquity.toFixed(2)}
• Varlık Devir Hızı: ${assetTurnover.toFixed(2)}
• Özkaynak Kârlılığı (ROE): %${roe.toFixed(2)}
• Aktif Kârlılığı (ROA): %${roa.toFixed(2)}

LÜTFEN AŞAĞIDAKİ BAŞLIKLARDA DETAYLI ANALİZ YAP:

💰 KARLILIK ANALİZİ:
- Brüt, net ve FAVÖK marjlarını değerlendir
- Sektör ortalaması ile karşılaştır
- Maliyet yapısı hakkında yorum yap

💧 LİKİDİTE DURUMU:
- Cari oranı analiz et
- Kısa vadeli borç ödeme gücünü değerlendir
- Çalışma sermayesi durumunu incele

⚖️ BORÇLULUK VE SERMAYE YAPISI:
- Borç/özkaynak oranını değerlendir
- Finansal kaldıraç etkisini analiz et
- Borç yapısının sürdürülebilirliğini incele

📊 VARLIK VERİMLİLİĞİ:
- ROE ve ROA oranlarını analiz et
- Varlık kullanım etkinliğini değerlendir
- Yönetim performansını incele

⚠️ RİSK DEĞERLENDİRMESİ:
- Ana finansal riskleri belirle
- Dikkat edilmesi gereken alanları listele

🎯 STRATEJİK ÖNERİLER:
- Kısa vadeli aksiyon önerileri (3-6 ay)
- Uzun vadeli iyileştirme stratejileri (1-2 yıl)
- Performans artırma fırsatları

📈 GELECEK PROJEKSİYONLARI:
- Mevcut trend analizi
- Büyüme potansiyeli değerlendirmesi

Türkçe, profesyonel ve anlaşılır bir dilde yanıt ver. Somut sayılar ve örnekler kullan.
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
                    netSales: getNumericValue(document.getElementById('netSales')) || 0,
                    costOfSales: getNumericValue(document.getElementById('costOfSales')) || 0,
                    adminExpenses: getNumericValue(document.getElementById('adminExpenses')) || 0,
                    ebitda: getNumericValue(document.getElementById('ebitda')) || 0,
                    netProfit: getNumericValue(document.getElementById('netProfit')) || 0,
                    currentAssets: getNumericValue(document.getElementById('currentAssets')) || 0,
                    fixedAssets: getNumericValue(document.getElementById('fixedAssets')) || 0,
                    shortTermDebt: getNumericValue(document.getElementById('shortTermDebt')) || 0,
                    longTermDebt: getNumericValue(document.getElementById('longTermDebt')) || 0,
                    equity: getNumericValue(document.getElementById('equity')) || 0
                };
            }
            
            const reportContent = document.getElementById('reportContent');
            
            if (!data || Object.values(data).every(val => val === 0)) {
                reportContent.innerHTML = '<div class="text-center py-8"><p class="text-gray-600 text-sm">📊 Finansal raporlar için önce veri girişi yapınız.</p><p class="text-xs text-gray-500 mt-2">Veriler girildikten sonra AI analizi otomatik olarak çalışacaktır.</p></div>';
                return;
            }
            
            // Calculate metrics
            const grossProfit = data.netSales - data.costOfSales;
            const grossProfitMargin = (grossProfit / data.netSales * 100) || 0;
            const netProfitMargin = (data.netProfit / data.netSales * 100) || 0;
            const ebitdaMargin = (data.ebitda / data.netSales * 100) || 0;
            const currentRatio = data.currentAssets / (data.shortTermDebt || 1);
            const debtToEquityRatio = (data.shortTermDebt + data.longTermDebt) / (data.equity || 1);
            const totalAssets = data.currentAssets + data.fixedAssets;
            const roe = (data.netProfit / data.equity * 100) || 0;
            const roa = (data.netProfit / totalAssets * 100) || 0;
            const assetTurnover = data.netSales / (totalAssets || 1);
            
            // Sektörel karşılaştırma için benchmark değerler
            const sectorBenchmarks = {
                grossMargin: { excellent: 40, good: 30, average: 20, poor: 10 },
                netMargin: { excellent: 15, good: 10, average: 5, poor: 0 },
                currentRatio: { excellent: 2.5, good: 2.0, average: 1.5, poor: 1.0 },
                debtToEquity: { excellent: 0.3, good: 0.5, average: 1.0, poor: 1.5 },
                roe: { excellent: 20, good: 15, average: 10, poor: 5 },
                roa: { excellent: 10, good: 7, average: 5, poor: 2 }
            };
            
            function getBenchmarkStatus(value, benchmark) {
                if (value >= benchmark.excellent) return '🟢 Mükemmel';
                if (value >= benchmark.good) return '🟡 İyi';
                if (value >= benchmark.average) return '🟠 Ortalama';
                return '🔴 Düşük';
            }
            
            // Analysis
            let profitabilityStatus = getBenchmarkStatus(netProfitMargin, sectorBenchmarks.netMargin);
            let liquidityStatus = getBenchmarkStatus(currentRatio, sectorBenchmarks.currentRatio);
            let leverageStatus = debtToEquityRatio < sectorBenchmarks.debtToEquity.excellent ? '🟢 Düşük' : 
                                debtToEquityRatio < sectorBenchmarks.debtToEquity.good ? '🟡 Orta' : 
                                debtToEquityRatio < sectorBenchmarks.debtToEquity.average ? '🟠 Yüksek' : '🔴 Çok Yüksek';
            
            // AI Analysis
            let aiAnalysis = '';
            if (aiAnalysisEnabled) {
                try {
                    reportContent.innerHTML = `
                        <div class="flex flex-col items-center justify-center py-8">
                            <div class="relative">
                                <div class="animate-spin rounded-full h-12 w-12 border-b-2 border-blue-600"></div>
                                <div class="absolute inset-0 flex items-center justify-center">
                                    <span class="text-blue-600 text-lg">🤖</span>
                                </div>
                            </div>
                            <div class="text-center mt-4">
                                <p class="text-sm font-semibold text-gray-800">Akça Pro X AI Analizi Hazırlanıyor...</p>
                                <p class="text-xs text-gray-600 mt-1">Finansal verileriniz Akça Pro X ile analiz ediliyor</p>
                                <div class="flex justify-center mt-2 space-x-1">
                                    <div class="w-2 h-2 bg-blue-400 rounded-full animate-pulse"></div>
                                    <div class="w-2 h-2 bg-blue-400 rounded-full animate-pulse" style="animation-delay: 0.1s;"></div>
                                    <div class="w-2 h-2 bg-blue-400 rounded-full animate-pulse" style="animation-delay: 0.2s;"></div>
                                </div>
                            </div>
                        </div>
                    `;
                    aiAnalysis = await generateAIAnalysis(data);
                } catch (error) {
                    console.error('AI Analysis Error:', error);
                    aiAnalysis = `⚠️ AI analizi şu anda kullanılamıyor: ${error.message}`;
                }
            }

            reportContent.innerHTML = `
                <div class="space-y-3">
                    ${aiAnalysisEnabled && aiAnalysis ? `
                        <div class="bg-gradient-to-r from-blue-50 to-purple-50 border border-blue-300 rounded-lg p-4 mb-4">
                            <div class="flex items-center justify-between mb-3">
                                <div class="flex items-center space-x-2">
                                    <span class="text-2xl">🤖</span>
                                    <div>
                                        <h3 class="text-sm font-bold text-blue-900">Akça Pro X AI Finansal Analizi</h3>
                                        <p class="text-xs text-blue-700">Akça Pro X ile Güçlendirilmiş Profesyonel Analiz</p>
                                    </div>
                                </div>
                                <div class="flex items-center space-x-1">
                                    <span class="bg-green-100 text-green-800 px-2 py-1 rounded-full text-xs font-semibold flex items-center">
                                        <span class="w-2 h-2 bg-green-500 rounded-full mr-1"></span>
                                        Aktif
                                    </span>
                                    <span class="bg-blue-100 text-blue-800 px-2 py-1 rounded text-xs">Akça Pro X</span>
                                </div>
                            </div>
                            <div class="bg-white rounded-lg p-3 border border-blue-200 shadow-sm">
                                <div class="prose prose-sm max-w-none">
                                    <div class="whitespace-pre-wrap text-xs text-gray-800 leading-relaxed">${aiAnalysis}</div>
                                </div>
                            </div>
                            <div class="mt-2 flex items-center justify-between text-xs text-blue-600">
                                <span>📊 Akça Pro X finansal danışmanlık analizi</span>
                                <span>⏱️ ${new Date().toLocaleTimeString('tr-TR')}</span>
                            </div>
                        </div>
                    ` : ''}
                    
                    <!-- Advanced Financial Metrics -->
                    <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-3 mb-4">
                        <div class="bg-gradient-to-br from-blue-50 to-blue-100 p-3 rounded-lg border border-blue-200">
                            <h4 class="font-semibold text-blue-800 text-xs mb-2 flex items-center">
                                <span class="mr-1">💰</span> Kârlılık Analizi
                            </h4>
                            <div class="text-xs space-y-1">
                                <div class="flex justify-between">
                                    <span>Brüt Marj:</span>
                                    <span class="font-semibold">${grossProfitMargin.toFixed(1)}%</span>
                                </div>
                                <div class="flex justify-between">
                                    <span>FAVÖK Marjı:</span>
                                    <span class="font-semibold">${ebitdaMargin.toFixed(1)}%</span>
                                </div>
                                <div class="flex justify-between">
                                    <span>Net Marj:</span>
                                    <span class="font-semibold">${netProfitMargin.toFixed(1)}%</span>
                                </div>
                                <div class="text-xs pt-1 border-t">${profitabilityStatus}</div>
                            </div>
                        </div>
                        
                        <div class="bg-gradient-to-br from-green-50 to-green-100 p-3 rounded-lg border border-green-200">
                            <h4 class="font-semibold text-green-800 text-xs mb-2 flex items-center">
                                <span class="mr-1">💧</span> Likidite Durumu
                            </h4>
                            <div class="text-xs space-y-1">
                                <div class="flex justify-between">
                                    <span>Cari Oran:</span>
                                    <span class="font-semibold">${currentRatio.toFixed(2)}</span>
                                </div>
                                <div class="flex justify-between">
                                    <span>Çalışma Sermayesi:</span>
                                    <span class="font-semibold">${((data.currentAssets - data.shortTermDebt) / 1000).toFixed(0)}K TL</span>
                                </div>
                                <div class="text-xs pt-1 border-t">${liquidityStatus}</div>
                            </div>
                        </div>
                        
                        <div class="bg-gradient-to-br from-purple-50 to-purple-100 p-3 rounded-lg border border-purple-200">
                            <h4 class="font-semibold text-purple-800 text-xs mb-2 flex items-center">
                                <span class="mr-1">⚖️</span> Borçluluk
                            </h4>
                            <div class="text-xs space-y-1">
                                <div class="flex justify-between">
                                    <span>Borç/Özkaynak:</span>
                                    <span class="font-semibold">${debtToEquityRatio.toFixed(2)}</span>
                                </div>
                                <div class="flex justify-between">
                                    <span>Toplam Borç:</span>
                                    <span class="font-semibold">${((data.shortTermDebt + data.longTermDebt) / 1000).toFixed(0)}K TL</span>
                                </div>
                                <div class="text-xs pt-1 border-t">${leverageStatus}</div>
                            </div>
                        </div>
                        
                        <div class="bg-gradient-to-br from-orange-50 to-orange-100 p-3 rounded-lg border border-orange-200">
                            <h4 class="font-semibold text-orange-800 text-xs mb-2 flex items-center">
                                <span class="mr-1">📊</span> Verimlilik
                            </h4>
                            <div class="text-xs space-y-1">
                                <div class="flex justify-between">
                                    <span>ROE:</span>
                                    <span class="font-semibold">${roe.toFixed(1)}%</span>
                                </div>
                                <div class="flex justify-between">
                                    <span>ROA:</span>
                                    <span class="font-semibold">${roa.toFixed(1)}%</span>
                                </div>
                                <div class="flex justify-between">
                                    <span>Varlık Devir:</span>
                                    <span class="font-semibold">${assetTurnover.toFixed(2)}x</span>
                                </div>
                                <div class="text-xs pt-1 border-t">${getBenchmarkStatus(roe, sectorBenchmarks.roe)}</div>
                            </div>
                        </div>
                        
                        <div class="bg-gradient-to-br from-indigo-50 to-indigo-100 p-3 rounded-lg border border-indigo-200">
                            <h4 class="font-semibold text-indigo-800 text-xs mb-2 flex items-center">
                                <span class="mr-1">🏦</span> Bilanço Yapısı
                            </h4>
                            <div class="text-xs space-y-1">
                                <div class="flex justify-between">
                                    <span>Toplam Varlık:</span>
                                    <span class="font-semibold">${(totalAssets / 1000).toFixed(0)}K TL</span>
                                </div>
                                <div class="flex justify-between">
                                    <span>Özkaynak Oranı:</span>
                                    <span class="font-semibold">${((data.equity / totalAssets) * 100).toFixed(1)}%</span>
                                </div>
                                <div class="flex justify-between">
                                    <span>Duran/Toplam:</span>
                                    <span class="font-semibold">${((data.fixedAssets / totalAssets) * 100).toFixed(1)}%</span>
                                </div>
                            </div>
                        </div>
                        
                        <div class="bg-gradient-to-br from-pink-50 to-pink-100 p-3 rounded-lg border border-pink-200">
                            <h4 class="font-semibold text-pink-800 text-xs mb-2 flex items-center">
                                <span class="mr-1">📈</span> Performans Özeti
                            </h4>
                            <div class="text-xs space-y-1">
                                <div class="flex justify-between">
                                    <span>Kârlılık:</span>
                                    <span class="font-semibold">${profitabilityStatus.split(' ')[1]}</span>
                                </div>
                                <div class="flex justify-between">
                                    <span>Likidite:</span>
                                    <span class="font-semibold">${liquidityStatus.split(' ')[1]}</span>
                                </div>
                                <div class="flex justify-between">
                                    <span>Borçluluk:</span>
                                    <span class="font-semibold">${leverageStatus.split(' ')[1]}</span>
                                </div>
                                <div class="text-xs pt-1 border-t">
                                    ${netProfitMargin > 10 && currentRatio > 1.5 && debtToEquityRatio < 1 ? '🟢 Güçlü Performans' : 
                                      netProfitMargin > 5 && currentRatio > 1 ? '🟡 Kabul Edilebilir' : '🔴 İyileştirme Gerekli'}
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
                if (element) {
                    // Sayıyı formatla ve inputa yaz
                    element.value = data[key].toLocaleString('tr-TR');
                }
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
                    const usersQuery = query(collection(db, 'users'), orderBy('createdAt', 'desc'));
                    const usersSnapshot = await getDocs(usersQuery);
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
                    const companiesQuery = query(collection(db, 'companies'), orderBy('createdAt', 'desc'));
                    const companiesSnapshot = await getDocs(companiesQuery);
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
                        const userDocRef = doc(db, 'users', companyId);
                        const userDoc = await getDoc(userDocRef);
                        if (userDoc.exists()) {
                            companyData = userDoc.data();
                        }
                    } catch (error) {
                        alert('❌ Google kullanıcı verisi yüklenemedi!');
                        return;
                    }
                } else {
                    // Şirket kullanıcısı
                    try {
                        const companyDocRef = doc(db, 'companies', companyId);
                        const companyDoc = await getDoc(companyDocRef);
                        if (companyDoc.exists()) {
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
                    const financialDocRef = doc(db, 'financialData', companyId);
                    const financialDoc = await getDoc(financialDocRef);
                    if (financialDoc.exists()) {
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
                    await deleteDoc(doc(db, 'users', companyId));
                } else {
                    // Şirket kaydını sil
                    try {
                        await deleteDoc(doc(db, 'companies', companyId));
                    } catch (error) {
                        // localStorage'dan sil
                        const companies = JSON.parse(localStorage.getItem('companies') || '{}');
                        delete companies[companyId];
                        localStorage.setItem('companies', JSON.stringify(companies));
                    }
                }
                
                // Finansal verileri de sil
                try {
                    await deleteDoc(doc(db, 'financialData', companyId));
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
                    saveBtn.textContent = '💾 Kaydet ve Akça Pro X AI Analizi Yap';
                    saveBtn.classList.remove('bg-gray-600');
                    saveBtn.classList.add('bg-green-600');
                } else {
                    saveBtn.textContent = '💾 Kaydet (Akça AI Kapalı)';
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
            // Test Firebase connection
            testFirebaseConnection();
            
            initializeAIToggle();
            
            // Demo scenario buttons
            document.getElementById('loadScenario1').addEventListener('click', () => loadDemoScenario(1));
            document.getElementById('loadScenario2').addEventListener('click', () => loadDemoScenario(2));
            document.getElementById('loadScenario3').addEventListener('click', () => loadDemoScenario(3));
            document.getElementById('loadScenario4').addEventListener('click', () => loadDemoScenario(4));
            
            // Şirket giriş alanlarına tıklandığında uyarı
            const companyInputs = ['loginCompanyName', 'loginPassword', 'registerCompanyName', 'registerEmail', 'registerPassword', 'registerPasswordConfirm'];
            companyInputs.forEach(inputId => {
                const input = document.getElementById(inputId);
                if (input) {
                    input.addEventListener('focus', function() {
                        alert('⚠️ UYARI!\n\nLütfen önce Google ile giriş yapınız!\n\n🔑 Bu alanları kullanmak için Google hesabıyla giriş yapmanız gerekir.');
                        this.blur(); // Input'tan çık
                    });
                }
            });
        });

        // Make functions global for onclick handlers
        window.viewCompanyDetails = viewCompanyDetails;
        window.viewCompanyFinancials = viewCompanyFinancials;
        window.deleteCompany = deleteCompany;
    </script>
    <!-- The script below was causing issues and is likely from a proxy/CDN, it's safer to remove it for local testing -->
    <!-- <script>(function(){function c(){var b=a.contentDocument||a.contentWindow.document;if(b){var d=b.createElement('script');d.innerHTML="window.__CF$cv$params={r:'98417ed5229d23b1',t:'MTc1ODcwODY2Mi4wMDAwMDA='};var a=document.createElement('script');a.nonce='';a.src='/cdn-cgi/challenge-platform/scripts/jsd/main.js';document.getElementsByTagName('head')[0].appendChild(a);";b.getElementsByTagName('head')[0].appendChild(d)}}if(document.body){var a=document.createElement('iframe');a.height=1;a.width=1;a.style.position='absolute';a.style.top=0;a.style.left=0;a.style.border='none';a.style.visibility='hidden';document.body.appendChild(a);if('loading'!==document.readyState)c();else if(window.addEventListener)document.addEventListener('DOMContentLoaded',c);else{var e=document.onreadystatechange||function(){};document.onreadystatechange=function(b){e(b);'loading'!==document.readyState&&(document.onreadystatechange=e,c())}}}})();</script> -->
</body>
</html>
