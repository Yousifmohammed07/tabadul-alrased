<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>منصة تبادل الرصيد الآمن</title>
    <!-- تحميل Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- تحميل أيقونات Lucide (للتصاميم المادية) -->
    <script src="https://unpkg.com/lucide@latest"></script>
    <style>
        /* إعدادات خط أساسية متجاوبة وجميلة */
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f0f2f5; 
        }
        .text-gradient {
            background-image: linear-gradient(to right, #0077b6, #00b4d8);
            -webkit-background-clip: text;
            color: transparent;
        }
        /* تثبيت شريط الإجراءات السريعة في الأسفل */
        #quickActions {
            position: fixed;
            bottom: 0;
            left: 0;
            right: 0;
            z-index: 10;
        }
        .view-container {
            min-height: calc(100vh - 80px); /* لمنع القوائم من الطفو في الأسفل */
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
        }
        /* لإخفاء القوائم المنسدلة افتراضياً */
        .dropdown-menu {
            display: none;
        }
        .escrow-step {
            transition: all 0.3s ease-in-out;
            color: #9ca3af;
        }
        .escrow-step.active {
            font-weight: bold;
            color: #1f2937;
        }
        .escrow-step.completed {
            color: #10b981; /* لون أخضر للخطوات المكتملة */
        }
        /* تصميم المودال/نافذة التفاعل الثابتة */
        .modal {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.6);
            display: flex;
            justify-content: center;
            align-items: center;
            z-index: 50;
        }
        .modal-content {
            background-color: white;
            padding: 24px;
            border-radius: 12px;
            max-width: 90%;
            width: 450px;
            box-shadow: 0 10px 25px rgba(0, 0, 0, 0.2);
            text-align: right;
            animation: fadeIn 0.3s;
        }
        @keyframes fadeIn {
            from {opacity: 0; transform: translateY(-10px);}
            to {opacity: 1; transform: translateY(0);}
        }

        /* تصميم الإشعارات العائمة */
        .alert-message {
            position: fixed;
            top: 1rem;
            right: 1rem;
            z-index: 50;
            max-width: 90%;
            width: 300px;
            box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
            animation: fadeIn 0.3s, fadeOut 0.5s 9.5s forwards; /* زيادة المدة لـ 10 ثوانٍ */
            direction: rtl; 
        }

        @keyframes fadeOut {
            from {opacity: 1;}
            to {opacity: 0;}
        }

        /* تنسيق الأزرار المشابهة لـ window.confirm */
        .custom-confirm-buttons {
            display: flex;
            justify-content: space-between;
            margin-top: 1rem;
        }
        
        /* شريط التقدم */
        .progress-line {
            position: absolute;
            right: 0; 
            left: 0; 
            top: 3px; 
            border-t-2 border-dashed border-gray-300 z-0;
        }
        
    </style>
</head>
<body class="min-h-screen pb-20">

    <!-- شريط التنقل العلوي -->
    <header class="bg-white shadow-lg sticky top-0 z-20">
        <div class="container mx-auto px-4 py-3 flex justify-between items-center">
            <div class="flex items-center space-x-2 space-x-reverse">
                <i data-lucide="shuffle" class="text-[#0077b6] w-6 h-6"></i>
                <h1 class="text-xl font-extrabold text-gray-800">صراف الرصيد</h1>
            </div>
            
            <!-- حالة المصادقة وقائمة المستخدم -->
            <div id="authStatusContainer" class="relative">
                <div id="authLoading" class="flex items-center text-sm font-medium">
                    <i data-lucide="loader" class="animate-spin w-4 h-4 text-gray-500 ml-2"></i>
                    <span class="text-gray-500">جاري التحقق...</span>
                </div>

                <div id="authenticatedUserActions" class="hidden">
                    <button onclick="toggleDropdown()" class="flex items-center p-2 rounded-full bg-indigo-50 hover:bg-indigo-100 transition duration-150">
                        <i data-lucide="user-check" class="w-5 h-5 text-indigo-600"></i>
                        <span id="userEmailDisplay" class="text-sm font-medium text-gray-800 mx-2 hidden sm:inline"></span>
                        <i data-lucide="chevron-down" class="w-4 h-4 text-gray-500"></i>
                    </button>
                    <!-- القائمة المنسدلة -->
                    <div id="userDropdown" class="dropdown-menu absolute left-0 mt-2 w-48 bg-white rounded-lg shadow-xl py-2 border border-gray-100 z-30">
                        <span class="block px-4 py-2 text-sm text-gray-500 border-b mb-1">المعرف: <span id="userShortId" class="font-mono text-xs"></span></span>
                        <a href="#" onclick="changeView('dashboardView'); toggleDropdown();" class="flex items-center px-4 py-2 text-gray-700 hover:bg-gray-100">
                            <i data-lucide="layout-dashboard" class="w-4 h-4 ml-2"></i> لوحة التحكم
                        </a>
                        <a id="adminLink" href="#" onclick="changeView('adminView'); toggleDropdown();" class="flex items-center px-4 py-2 text-purple-600 hover:bg-purple-50 hidden">
                            <i data-lucide="shield-check" class="w-4 h-4 ml-2"></i> لوحة الإدارة
                        </a>
                        <div class="border-t border-gray-100 my-1"></div>
                        <a href="#" onclick="handleSignOut()" class="flex items-center px-4 py-2 text-red-600 hover:bg-red-50">
                            <i data-lucide="log-out" class="w-4 h-4 ml-2"></i> تسجيل الخروج
                        </a>
                    </div>
                </div>
                
                <div id="unauthenticatedUserActions" class="hidden">
                     <button onclick="changeView('signInView')" class="py-1 px-3 text-sm bg-indigo-500 text-white rounded-lg hover:bg-indigo-600 transition duration-150">
                        تسجيل الدخول
                    </button>
                </div>
            </div>
        </div>
    </header>

    <!-- محتوى التطبيق الرئيسي -->
    <main class="container mx-auto p-4 sm:p-6">
        <div id="main-content">
            
            <!-- ---------------------------------------------------- -->
            <!-- 1. واجهة تسجيل الدخول (Sign In) -->
            <!-- ---------------------------------------------------- -->
            <div id="signInView" class="view-container hidden max-w-sm mx-auto p-6 bg-white rounded-xl shadow-2xl">
                <h3 class="text-3xl font-bold text-gray-800 mb-6 border-b pb-2 w-full text-center">تسجيل الدخول</h3>
                <form onsubmit="handleSignIn(event)" class="w-full space-y-4">
                    <input type="email" id="signInEmail" placeholder="البريد الإلكتروني" required class="w-full p-3 border border-gray-300 rounded-lg focus:ring-indigo-500 focus:border-indigo-500" dir="ltr">
                    <input type="password" id="signInPassword" placeholder="كلمة المرور" required class="w-full p-3 border border-gray-300 rounded-lg focus:ring-indigo-500 focus:border-indigo-500" dir="ltr">
                    <button type="submit" class="w-full py-3 bg-indigo-600 text-white font-bold rounded-lg shadow-md hover:bg-indigo-700 transition duration-300">
                        دخول المنصة
                    </button>
                </form>
                <p class="mt-6 text-sm text-gray-600">
                    ليس لديك حساب؟ 
                    <a href="#" onclick="changeView('signUpView')" class="font-semibold text-indigo-600 hover:text-indigo-800">إنشاء حساب جديد</a>
                </p>
            </div>

            <!-- ---------------------------------------------------- -->
            <!-- 2. واجهة تسجيل حساب جديد (Sign Up) -->
            <!-- ---------------------------------------------------- -->
            <div id="signUpView" class="hidden view-container max-w-sm mx-auto p-6 bg-white rounded-xl shadow-2xl">
                <h3 class="text-3xl font-bold text-gray-800 mb-6 border-b pb-2 w-full text-center">إنشاء حساب جديد</h3>
                
                <!-- رسالة توجيهية للمستخدم -->
                <div class="bg-red-50 border-r-4 border-red-400 p-4 mb-4 rounded-lg">
                    <p class="font-semibold text-red-800">تنبيه!</p>
                    <p class="text-sm text-red-700 mt-1">يجب إنشاء الحساب من <a href="https://console.firebase.google.com/project/tabadulalrased/authentication/users" target="_blank" class="font-bold underline">لوحة تحكم Firebase</a> حالياً بسبب قيود الأمان في التطبيق. استخدم البيانات التي أنشأتها لتسجيل الدخول.</p>
                </div>

                <form onsubmit="event.preventDefault(); alertMessage('يرجى إنشاء الحساب من لوحة تحكم Firebase أولاً.', 'error')" class="w-full space-y-4">
                    <input type="email" id="signUpEmail" placeholder="البريد الإلكتروني" disabled class="w-full p-3 border border-gray-300 bg-gray-100 rounded-lg" dir="ltr">
                    <input type="password" id="signUpPassword" placeholder="كلمة المرور (6 أحرف أو أكثر)" disabled class="w-full p-3 border border-gray-300 bg-gray-100 rounded-lg" dir="ltr">
                    <button type="submit" disabled class="w-full py-3 bg-gray-400 text-white font-bold rounded-lg shadow-md">
                        تسجيل الحساب (غير متاح الآن)
                    </button>
                </form>
                
                <p class="mt-6 text-sm text-gray-600">
                    لديك حساب بالفعل؟ 
                    <a href="#" onclick="changeView('signInView')" class="font-semibold text-green-600 hover:text-green-800">تسجيل الدخول</a>
                </p>
            </div>
            
            <!-- ---------------------------------------------------- -->
            <!-- 3. محتوى التطبيق المحمي -->
            <!-- ---------------------------------------------------- -->
            <div id="appContainer" class="hidden">
                <!-- لوحة التحكم (Dashboard) -->
                <div id="dashboardView" class="space-y-8">
                    
                    <!-- عنوان ترحيبي -->
                    <h2 class="text-3xl font-extrabold text-gray-900">مرحباً بك في لوحة التحكم!</h2>
                    
                    <!-- قسم الإحصائيات والمالية (Stats Cards) -->
                    <section class="grid grid-cols-1 sm:grid-cols-3 gap-4">
                        
                        <!-- 1. الرصيد النقدي المتوفر (المحفظة) -->
                        <div class="bg-white rounded-xl shadow-md p-5 border-t-4 border-indigo-500">
                            <div class="flex justify-between items-center">
                                <p class="text-sm font-medium text-gray-500">الرصيد النقدي المتوفر</p>
                                <i data-lucide="wallet" class="w-6 h-6 text-indigo-500"></i>
                            </div>
                            <p id="userBalance" class="text-4xl font-bold text-gray-800 mt-2">جاري التحميل...</p>
                            <p class="text-xs text-gray-400 mt-1 flex items-center">
                                <i data-lucide="plus-circle" onclick="changeView('fundWalletView')" class="w-4 h-4 ml-1 text-green-500 cursor-pointer hover:text-green-700"></i>
                                <span onclick="changeView('fundWalletView')" class="cursor-pointer hover:underline">شحن المحفظة</span>
                            </p>
                        </div>

                        <!-- 2. العمليات قيد الانتظار (Escrow) -->
                        <div onclick="showEscrowDetails()" class="bg-white rounded-xl shadow-md p-5 border-t-4 border-yellow-500 cursor-pointer hover:shadow-lg transition duration-200">
                            <div class="flex justify-between items-center">
                                <p class="text-sm font-medium text-gray-500">المبالغ المحجوزة / قيد المراجعة</p>
                                <i data-lucide="lock" class="w-6 h-6 text-yellow-500"></i>
                            </div>
                            <p id="pendingEscrow" class="text-4xl font-bold text-gray-800 mt-2">جاري التحميل...</p>
                            <p class="text-xs text-gray-400 mt-1">عملية قيد المراجعة/التحويل</p>
                        </div>
                        
                        <!-- 3. إجمالي العمليات المكتملة -->
                        <div class="bg-white rounded-xl shadow-md p-5 border-t-4 border-green-500">
                            <div class="flex justify-between items-center">
                                <p class="text-sm font-medium text-gray-500">إجمالي العمليات المكتملة</p>
                                <i data-lucide="trending-up" class="w-6 h-6 text-green-500"></i>
                            </div>
                            <p id="completedTxs" class="text-4xl font-bold text-gray-800 mt-2">جاري التحميل...</p>
                            <p class="text-xs text-gray-400 mt-1">عملية نجحت على المنصة</p>
                        </div>
                    </section>

                    <!-- شريط التقدم المرئي (Visual Escrow Status) -->
                    <section class="bg-white rounded-xl shadow-lg p-6">
                        <h3 class="text-xl font-semibold mb-4 text-gray-800">حالة عملية Escrow الأخيرة</h3>
                        <div class="flex items-center justify-between text-center relative">
                            <!-- الخط الواصل -->
                            <div class="progress-line absolute"></div>
                            
                            <div id="step1" class="escrow-step flex-1 z-10">
                                <div class="w-8 h-8 rounded-full bg-gray-200 flex items-center justify-center mx-auto mb-1 text-sm text-gray-600">1</div>
                                <span class="text-xs text-gray-600">الدفع والحجز</span>
                            </div>

                            <div id="step2" class="escrow-step flex-1 z-10">
                                <div class="w-8 h-8 rounded-full bg-gray-200 flex items-center justify-center mx-auto mb-1 text-sm text-gray-600">2</div>
                                <span class="text-xs text-
