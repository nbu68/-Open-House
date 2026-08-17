# -Open-House
เพื่อเก็บข้อมูลนักเรียนที่มาเข้าร่วมงาน Open House ขอหาวิทยาลัยนอร์ทกรุงเทพ
<!DOCTYPE html>
<html lang="th" class="h-full bg-slate-950">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>IT Open House App - ระบบลงทะเบียน & Google Sheets Sync</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Chart.js CDN -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <!-- Font Awesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css">
    <!-- Google Fonts (Prompt) -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Prompt:ital,wght@0,300;0,400;0,500;0,600;0,700;1,400&display=swap" rel="stylesheet">

    <!-- Tailwind Custom Configuration -->
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    fontFamily: {
                        sans: ['Prompt', 'sans-serif'],
                    },
                    colors: {
                        cyber: {
                            50: '#f0f7ff',
                            100: '#e0effe',
                            500: '#06b6d4',
                            600: '#0284c7',
                            700: '#0369a1',
                            800: '#075985',
                            900: '#0c4a6e',
                            950: '#082f49',
                        }
                    },
                    boxShadow: {
                        'glow-cyan': '0 0 20px -3px rgba(6, 182, 212, 0.4)',
                        'glow-indigo': '0 0 20px -3px rgba(99, 102, 241, 0.4)',
                    }
                }
            }
        }
    </script>

    <style>
        body {
            font-family: 'Prompt', sans-serif;
            background: radial-gradient(circle at top right, #0f172a, #020617, #090d16);
            color: #f8fafc;
            min-height: 100vh;
        }

        /* Ambient glowing dots background pattern */
        .bg-grid-pattern {
            background-size: 30px 30px;
            background-image: 
                linear-gradient(to right, rgba(255, 255, 255, 0.03) 1px, transparent 1px),
                linear-gradient(to bottom, rgba(255, 255, 255, 0.03) 1px, transparent 1px);
        }

        .glass-card {
            background: rgba(15, 23, 42, 0.75);
            backdrop-filter: blur(16px);
            -webkit-backdrop-filter: blur(16px);
            border: 1px solid rgba(255, 255, 255, 0.08);
        }

        .glass-input {
            background: rgba(30, 41, 59, 0.6);
            border: 1px solid rgba(255, 255, 255, 0.12);
            color: #f8fafc;
        }
        .glass-input:focus {
            background: rgba(30, 41, 59, 0.9);
            border-color: #38bdf8;
            box-shadow: 0 0 12px rgba(56, 189, 248, 0.3);
        }

        /* Custom Scrollbar */
        ::-webkit-scrollbar {
            width: 8px;
            height: 8px;
        }
        ::-webkit-scrollbar-track {
            background: #090d16;
        }
        ::-webkit-scrollbar-thumb {
            background: #1e293b;
            border-radius: 4px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: #334155;
        }
    </style>
</head>
<body class="flex flex-col min-h-screen relative bg-grid-pattern overflow-x-hidden">

    <!-- App Navigation Header -->
    <header class="sticky top-0 z-40 border-b border-slate-800/80 bg-slate-950/80 backdrop-blur-xl">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 h-20 flex items-center justify-between">
            
            <!-- Brand Logo -->
            <div class="flex items-center space-x-3.5">
                <div class="w-11 h-11 rounded-2xl bg-gradient-to-tr from-cyan-500 to-indigo-600 flex items-center justify-center text-white shadow-glow-cyan">
                    <i class="fa-solid fa-graduation-cap text-xl"></i>
                </div>
                <div>
                    <div class="flex items-center space-x-2">
                        <span class="font-bold text-lg sm:text-xl tracking-tight bg-gradient-to-r from-cyan-400 via-sky-200 to-indigo-300 bg-clip-text text-transparent">
                            IT OPEN HOUSE
                        </span>
                        <span class="px-2 py-0.5 text-[10px] font-semibold bg-cyan-950 text-cyan-400 border border-cyan-800/60 rounded-full">
                            2026 LIVE
                        </span>
                    </div>
                    <p class="text-xs text-slate-400 font-light">ระบบลงทะเบียนเข้างาน & เชื่อมต่อ Google Sheet</p>
                </div>
            </div>

            <!-- Header Controls & Nav Tabs -->
            <div class="flex items-center space-x-3">
                
                <!-- Google Sheet Setup Button -->
                <button onclick="openGoogleSheetModal()" class="hidden sm:flex items-center space-x-2 px-3.5 py-2 rounded-xl text-xs font-medium border border-emerald-500/30 bg-emerald-950/40 text-emerald-300 hover:bg-emerald-900/40 transition-all">
                    <i class="fa-solid fa-file-excel text-emerald-400"></i>
                    <span id="gs-status-btn-text">ตั้งค่า Google Sheet</span>
                    <span id="gs-status-dot" class="w-2 h-2 rounded-full bg-slate-500"></span>
                </button>

                <!-- Navigation Tabs Switcher -->
                <nav class="flex bg-slate-900/90 p-1 rounded-xl border border-slate-800">
                    <button id="tab-form-btn" onclick="switchView('form')" class="flex items-center space-x-2 px-4 py-2 rounded-lg text-xs sm:text-sm font-medium transition-all bg-gradient-to-r from-cyan-500 to-blue-600 text-white shadow-md">
                        <i class="fa-solid fa-id-card"></i>
                        <span>ลงทะเบียน</span>
                    </button>
                    <button id="tab-dashboard-btn" onclick="switchView('dashboard')" class="flex items-center space-x-2 px-4 py-2 rounded-lg text-xs sm:text-sm font-medium transition-all text-slate-400 hover:text-white hover:bg-slate-800/60">
                        <i class="fa-solid fa-chart-line"></i>
                        <span>Dashboard</span>
                    </button>
                </nav>
            </div>
        </div>
    </header>

    <!-- Main Content Container -->
    <main class="flex-grow max-w-7xl w-full mx-auto px-4 sm:px-6 lg:px-8 py-8 relative">

        <!-- Mobile Google Sheet Settings Button Banner -->
        <div class="sm:hidden mb-6">
            <button onclick="openGoogleSheetModal()" class="w-full flex items-center justify-between px-4 py-3 rounded-xl border border-emerald-500/30 bg-emerald-950/40 text-emerald-300 text-xs font-medium">
                <span class="flex items-center space-x-2">
                    <i class="fa-solid fa-file-excel text-emerald-400 text-sm"></i>
                    <span id="gs-status-btn-text-mobile">ตั้งค่า Google Sheet Integration</span>
                </span>
                <span id="gs-status-dot-mobile" class="w-2.5 h-2.5 rounded-full bg-slate-500"></span>
            </button>
        </div>

        <!-- ========================================== -->
        <!-- VIEW 1: REGISTRATION FORM VIEW             -->
        <!-- ========================================== -->
        <section id="view-form" class="max-w-3xl mx-auto space-y-6 transition-all duration-300">
            
            <div class="glass-card rounded-3xl p-6 sm:p-10 shadow-2xl relative overflow-hidden">
                
                <!-- Glowing Accent lines -->
                <div class="absolute -top-24 -right-24 w-48 h-48 bg-cyan-500/10 rounded-full blur-3xl pointer-events-none"></div>
                <div class="absolute -bottom-24 -left-24 w-48 h-48 bg-indigo-500/10 rounded-full blur-3xl pointer-events-none"></div>

                <!-- Section Title -->
                <div class="border-b border-slate-800/80 pb-6 mb-8 flex items-start justify-between">
                    <div>
                        <h2 class="text-2xl sm:text-3xl font-bold text-white tracking-tight flex items-center gap-3">
                            <i class="fa-solid fa-pen-to-square text-cyan-400"></i>
                            ลงทะเบียนเข้าร่วมงาน
                        </h2>
                        <p class="text-slate-400 text-xs sm:text-sm mt-1.5 font-light">
                            กรอกข้อมูลของท่านเพื่อลงทะเบียน ลุ้นรับของรางวัล และสำรองที่นั่งเข้าร่วมกิจกรรมคณะ IT
                        </p>
                    </div>
                </div>

                <!-- Registration Form -->
                <form id="studentRegistrationForm" onsubmit="handleRegistrationSubmit(event)" class="space-y-6">
                    
                    <!-- Row 1: คำนำหน้า + ชื่อ-นามสกุล -->
                    <div class="grid grid-cols-1 sm:grid-cols-12 gap-5">
                        <div class="sm:col-span-4">
                            <label class="block text-xs font-semibold text-slate-300 uppercase tracking-wider mb-2">
                                คำนำหน้า <span class="text-cyan-400">*</span>
                            </label>
                            <select id="prefix" required class="glass-input w-full px-4 py-3 rounded-xl outline-none transition-all cursor-pointer">
                                <option value="" disabled selected class="bg-slate-900 text-slate-400">เลือกคำนำหน้า</option>
                                <option value="นาย" class="bg-slate-900 text-white">นาย</option>
                                <option value="นางสาว" class="bg-slate-900 text-white">นางสาว</option>
                                <option value="เด็กชาย" class="bg-slate-900 text-white">เด็กชาย</option>
                                <option value="เด็กหญิง" class="bg-slate-900 text-white">เด็กหญิง</option>
                                <option value="อื่นๆ" class="bg-slate-900 text-white">อื่นๆ</option>
                            </select>
                        </div>
                        <div class="sm:col-span-8">
                            <label class="block text-xs font-semibold text-slate-300 uppercase tracking-wider mb-2">
                                ชื่อ - นามสกุล <span class="text-cyan-400">*</span>
                            </label>
                            <div class="relative">
                                <div class="absolute inset-y-0 left-0 pl-3.5 flex items-center pointer-events-none text-slate-400">
                                    <i class="fa-regular fa-user"></i>
                                </div>
                                <input type="text" id="fullname" required placeholder="สมชาย ใจดี" class="glass-input w-full pl-10 pr-4 py-3 rounded-xl outline-none transition-all">
                            </div>
                        </div>
                    </div>

                    <!-- Row 2: เพศ + อายุ + ระดับชั้นเรียน -->
                    <div class="grid grid-cols-1 sm:grid-cols-3 gap-5">
                        <div>
                            <label class="block text-xs font-semibold text-slate-300 uppercase tracking-wider mb-2">
                                เพศ <span class="text-cyan-400">*</span>
                            </label>
                            <select id="gender" required class="glass-input w-full px-4 py-3 rounded-xl outline-none transition-all cursor-pointer">
                                <option value="" disabled selected class="bg-slate-900 text-slate-400">เลือกเพศ</option>
                                <option value="ชาย" class="bg-slate-900 text-white">ชาย</option>
                                <option value="หญิง" class="bg-slate-900 text-white">หญิง</option>
                                <option value="LGBTQ+" class="bg-slate-900 text-white">LGBTQ+</option>
                                <option value="ไม่ระบุ" class="bg-slate-900 text-white">ไม่ระบุ</option>
                            </select>
                        </div>
                        <div>
                            <label class="block text-xs font-semibold text-slate-300 uppercase tracking-wider mb-2">
                                อายุ (ปี) <span class="text-cyan-400">*</span>
                            </label>
                            <input type="number" id="age" min="10" max="80" required placeholder="17" class="glass-input w-full px-4 py-3 rounded-xl outline-none transition-all">
                        </div>
                        <div>
                            <label class="block text-xs font-semibold text-slate-300 uppercase tracking-wider mb-2">
                                ระดับชั้นเรียน <span class="text-cyan-400">*</span>
                            </label>
                            <select id="educationLevel" required class="glass-input w-full px-4 py-3 rounded-xl outline-none transition-all cursor-pointer">
                                <option value="" disabled selected class="bg-slate-900 text-slate-400">เลือกระดับชั้น</option>
                                <option value="มัธยมศึกษาตอนต้น" class="bg-slate-900 text-white">มัธยมศึกษาตอนต้น</option>
                                <option value="มัธยมศึกษาปีที่ 4" class="bg-slate-900 text-white">มัธยมศึกษาปีที่ 4</option>
                                <option value="มัธยมศึกษาปีที่ 5" class="bg-slate-900 text-white">มัธยมศึกษาปีที่ 5</option>
                                <option value="มัธยมศึกษาปีที่ 6" class="bg-slate-900 text-white">มัธยมศึกษาปีที่ 6</option>
                                <option value="ปวช." class="bg-slate-900 text-white">ปวช.</option>
                                <option value="ปวส." class="bg-slate-900 text-white">ปวส.</option>
                                <option value="อื่นๆ" class="bg-slate-900 text-white">อื่นๆ</option>
                            </select>
                        </div>
                    </div>

                    <!-- Row 3: ชื่อโรงเรียน -->
                    <div>
                        <label class="block text-xs font-semibold text-slate-300 uppercase tracking-wider mb-2">
                            ชื่อสถานศึกษา / โรงเรียน <span class="text-cyan-400">*</span>
                        </label>
                        <div class="relative">
                            <div class="absolute inset-y-0 left-0 pl-3.5 flex items-center pointer-events-none text-slate-400">
                                <i class="fa-solid fa-school"></i>
                            </div>
                            <input type="text" id="school" required placeholder="โรงเรียนเตรียมอุดมศึกษา..." class="glass-input w-full pl-10 pr-4 py-3 rounded-xl outline-none transition-all">
                        </div>
                    </div>

                    <!-- Row 4: เบอร์โทรศัพท์ + Line ID -->
                    <div class="grid grid-cols-1 sm:grid-cols-2 gap-5">
                        <div>
                            <label class="block text-xs font-semibold text-slate-300 uppercase tracking-wider mb-2">
                                เบอร์โทรศัพท์ <span class="text-cyan-400">*</span>
                            </label>
                            <div class="relative">
                                <div class="absolute inset-y-0 left-0 pl-3.5 flex items-center pointer-events-none text-slate-400">
                                    <i class="fa-solid fa-phone"></i>
                                </div>
                                <input type="tel" id="phone" pattern="[0-9]{9,10}" required placeholder="0812345678" class="glass-input w-full pl-10 pr-4 py-3 rounded-xl outline-none transition-all">
                            </div>
                        </div>
                        <div>
                            <label class="block text-xs font-semibold text-slate-300 uppercase tracking-wider mb-2">
                                Line ID <span class="text-cyan-400">*</span>
                            </label>
                            <div class="relative">
                                <div class="absolute inset-y-0 left-0 pl-3.5 flex items-center pointer-events-none text-slate-400">
                                    <i class="fa-brands fa-line"></i>
                                </div>
                                <input type="text" id="lineId" required placeholder="line_id_123" class="glass-input w-full pl-10 pr-4 py-3 rounded-xl outline-none transition-all">
                            </div>
                        </div>
                    </div>

                    <!-- Row 5: สาขาวิชาที่สนใจของคณะไอที -->
                    <div class="pt-2">
                        <label class="block text-xs font-semibold text-slate-300 uppercase tracking-wider mb-3">
                            สาขาวิชาที่สนใจของคณะไอที <span class="text-cyan-400">*</span>
                        </label>
                        <div class="grid grid-cols-1 gap-3">
                            
                            <!-- Major 1 -->
                            <label class="relative flex items-center p-4 rounded-xl border border-slate-800 bg-slate-900/40 hover:bg-slate-800/60 hover:border-cyan-500/50 cursor-pointer transition-all group">
                                <input type="radio" name="interestedMajor" value="เทคโนโลยีธุรกิจดิจิทัล" required class="w-4 h-4 text-cyan-500 bg-slate-900 border-slate-700 focus:ring-cyan-500">
                                <div class="ml-3 flex items-center space-x-3">
                                    <span class="w-3 h-3 rounded-full bg-cyan-400 shadow-glow-cyan"></span>
                                    <div>
                                        <p class="text-sm font-semibold text-slate-200 group-hover:text-cyan-300">เทคโนโลยีธุรกิจดิจิทัล (Digital Business Technology)</p>
                                        <p class="text-xs text-slate-400">มุ่งเน้นระบบธุรกิจดิจิทัล การวิเคราะห์ข้อมูล และ FinTech</p>
                                    </div>
                                </div>
                            </label>

                            <!-- Major 2 -->
                            <label class="relative flex items-center p-4 rounded-xl border border-slate-800 bg-slate-900/40 hover:bg-slate-800/60 hover:border-indigo-500/50 cursor-pointer transition-all group">
                                <input type="radio" name="interestedMajor" value="เทคโนโลยีสารสนเทศและนวัตกรรมดิจิทัล" class="w-4 h-4 text-indigo-500 bg-slate-900 border-slate-700 focus:ring-indigo-500">
                                <div class="ml-3 flex items-center space-x-3">
                                    <span class="w-3 h-3 rounded-full bg-indigo-400 shadow-glow-indigo"></span>
                                    <div>
                                        <p class="text-sm font-semibold text-slate-200 group-hover:text-indigo-300">เทคโนโลยีสารสนเทศและนวัตกรรมดิจิทัล (IT & Digital Innovation)</p>
                                        <p class="text-xs text-slate-400">มุ่งเน้นการพัฒนา ซอฟต์แวร์ Cloud, AI และ นวัตกรรมไอที</p>
                                    </div>
                                </div>
                            </label>

                            <!-- Major 3 -->
                            <label class="relative flex items-center p-4 rounded-xl border border-slate-800 bg-slate-900/40 hover:bg-slate-800/60 hover:border-purple-500/50 cursor-pointer transition-all group">
                                <input type="radio" name="interestedMajor" value="ดิจิทัลมีเดียอาร์ต" class="w-4 h-4 text-purple-500 bg-slate-900 border-slate-700 focus:ring-purple-500">
                                <div class="ml-3 flex items-center space-x-3">
                                    <span class="w-3 h-3 rounded-full bg-purple-400"></span>
                                    <div>
                                        <p class="text-sm font-semibold text-slate-200 group-hover:text-purple-300">ดิจิทัลมีเดียอาร์ต (Digital Media Arts)</p>
                                        <p class="text-xs text-slate-400">มุ่งเน้นการออกแบบ UI/UX, 3D Animation, และสื่อสร้างสรรค์ดิจิทัล</p>
                                    </div>
                                </div>
                            </label>

                            <!-- Major 4 -->
                            <label class="relative flex items-center p-4 rounded-xl border border-slate-800 bg-slate-900/40 hover:bg-slate-800/60 hover:border-emerald-500/50 cursor-pointer transition-all group">
                                <input type="radio" name="interestedMajor" value="วิศวกรรมความปลอดภัย" class="w-4 h-4 text-emerald-500 bg-slate-900 border-slate-700 focus:ring-emerald-500">
                                <div class="ml-3 flex items-center space-x-3">
                                    <span class="w-3 h-3 rounded-full bg-emerald-400"></span>
                                    <div>
                                        <p class="text-sm font-semibold text-slate-200 group-hover:text-emerald-300">วิศวกรรมความปลอดภัย (Cybersecurity & Security Eng.)</p>
                                        <p class="text-xs text-slate-400">มุ่งเน้นความปลอดภัยทางไซเบอร์ การป้องกันการโจมตีทางเครือข่าย</p>
                                    </div>
                                </div>
                            </label>

                            <!-- Major 5 -->
                            <label class="relative flex items-center p-4 rounded-xl border border-slate-800 bg-slate-900/40 hover:bg-slate-800/60 hover:border-slate-500/50 cursor-pointer transition-all group">
                                <input type="radio" name="interestedMajor" value="ไม่สนใจ" class="w-4 h-4 text-slate-400 bg-slate-900 border-slate-700 focus:ring-slate-400">
                                <div class="ml-3 flex items-center space-x-3">
                                    <span class="w-3 h-3 rounded-full bg-slate-500"></span>
                                    <div>
                                        <p class="text-sm font-semibold text-slate-300 group-hover:text-slate-100">ไม่สนใจ / มาเยี่ยมชมบรรยากาศงานทั่วไป</p>
                                        <p class="text-xs text-slate-500">ยังไม่ได้ตัดสินใจเลือกสาขาเฉพาะทาง</p>
                                    </div>
                                </div>
                            </label>

                        </div>
                    </div>

                    <!-- Google Sheets Auto-Sync Indicator -->
                    <div id="form-gs-notice" class="p-3.5 rounded-xl border border-slate-800 bg-slate-900/80 flex items-center justify-between text-xs">
                        <div class="flex items-center space-x-2.5 text-slate-300">
                            <i class="fa-solid fa-cloud-arrow-up text-cyan-400"></i>
                            <span id="form-gs-notice-text">โหมดบันทึกข้อมูลในเครื่อง (คุณสามารถเปิดการเชื่อมต่อ Google Sheet ได้)</span>
                        </div>
                        <button type="button" onclick="openGoogleSheetModal()" class="text-cyan-400 hover:underline text-[11px] font-medium">
                            ตั้งค่า
                        </button>
                    </div>

                    <!-- Submit Buttons -->
                    <div class="pt-4 flex items-center gap-4">
                        <button type="reset" class="w-1/3 py-3.5 rounded-xl border border-slate-800 text-slate-400 hover:text-white hover:bg-slate-900 font-medium transition-all text-sm">
                            <i class="fa-solid fa-rotate-left mr-1.5"></i> ล้างข้อมูล
                        </button>
                        <button type="submit" id="submitBtn" class="w-2/3 py-3.5 rounded-xl bg-gradient-to-r from-cyan-500 via-sky-500 to-indigo-600 hover:from-cyan-400 hover:to-indigo-500 text-white font-semibold shadow-glow-cyan transition-all flex items-center justify-center space-x-2 text-sm">
                            <i class="fa-solid fa-paper-plane"></i>
                            <span id="submitBtnText">ส่งข้อมูลลงทะเบียน</span>
                        </button>
                    </div>

                </form>
            </div>
        </section>

        <!-- ========================================== -->
        <!-- VIEW 2: DASHBOARD ANALYTICS VIEW           -->
        <!-- ========================================== -->
        <section id="view-dashboard" class="hidden space-y-8 transition-all duration-300">
            
            <!-- Key Metric Cards Grid -->
            <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-5">
                
                <!-- Stat Card 1 -->
                <div class="glass-card p-5 rounded-2xl flex items-center space-x-4 border border-slate-800">
                    <div class="w-12 h-12 rounded-xl bg-cyan-950/80 border border-cyan-800/60 text-cyan-400 flex items-center justify-center text-xl font-bold shadow-glow-cyan">
                        <i class="fa-solid fa-users"></i>
                    </div>
                    <div>
                        <p class="text-[11px] font-semibold text-slate-400 uppercase tracking-wider">ผู้ลงทะเบียนทั้งหมด</p>
                        <h3 id="stat-total" class="text-2xl font-bold text-white">0</h3>
                        <p class="text-[11px] text-cyan-400 font-light mt-0.5"><i class="fa-solid fa-bolt mr-1"></i>อัปเดตเรียลไทม์</p>
                    </div>
                </div>

                <!-- Stat Card 2 -->
                <div class="glass-card p-5 rounded-2xl flex items-center space-x-4 border border-slate-800">
                    <div class="w-12 h-12 rounded-xl bg-indigo-950/80 border border-indigo-800/60 text-indigo-400 flex items-center justify-center text-xl font-bold shadow-glow-indigo">
                        <i class="fa-solid fa-fire"></i>
                    </div>
                    <div>
                        <p class="text-[11px] font-semibold text-slate-400 uppercase tracking-wider">สาขายอดนิยมอันดับ 1</p>
                        <h3 id="stat-top-major" class="text-sm font-bold text-slate-100 truncate max-w-[150px]">-</h3>
                        <p id="stat-top-count" class="text-[11px] text-indigo-400 font-light mt-0.5">0 คนสนใจ</p>
                    </div>
                </div>

                <!-- Stat Card 3 -->
                <div class="glass-card p-5 rounded-2xl flex items-center space-x-4 border border-slate-800">
                    <div class="w-12 h-12 rounded-xl bg-emerald-950/80 border border-emerald-800/60 text-emerald-400 flex items-center justify-center text-xl font-bold">
                        <i class="fa-solid fa-school"></i>
                    </div>
                    <div>
                        <p class="text-[11px] font-semibold text-slate-400 uppercase tracking-wider">โรงเรียนที่มาร่วมงาน</p>
                        <h3 id="stat-schools" class="text-2xl font-bold text-white">0</h3>
                        <p class="text-[11px] text-slate-400 font-light mt-0.5">แห่งทั่วประเทศ</p>
                    </div>
                </div>

                <!-- Stat Card 4 -->
                <div class="glass-card p-5 rounded-2xl flex items-center space-x-4 border border-slate-800">
                    <div class="w-12 h-12 rounded-xl bg-purple-950/80 border border-purple-800/60 text-purple-400 flex items-center justify-center text-xl font-bold">
                        <i class="fa-solid fa-file-excel"></i>
                    </div>
                    <div>
                        <p class="text-[11px] font-semibold text-slate-400 uppercase tracking-wider">สถานะ Google Sheet</p>
                        <h3 id="stat-gs-status" class="text-sm font-bold text-slate-300">ไม่ได้เชื่อมต่อ</h3>
                        <p id="stat-gs-subtext" class="text-[11px] text-slate-400 font-light mt-0.5">คลิกเพื่อตั้งค่า Webhook</p>
                    </div>
                </div>

            </div>

            <!-- Charts Container Grid -->
            <div class="grid grid-cols-1 lg:grid-cols-12 gap-6">
                
                <!-- Chart 1: Major Distribution Doughnut -->
                <div class="lg:col-span-7 glass-card p-6 rounded-3xl border border-slate-800 flex flex-col justify-between">
                    <div class="flex items-center justify-between mb-4">
                        <div>
                            <h3 class="text-lg font-bold text-white flex items-center gap-2">
                                <i class="fa-solid fa-chart-pie text-cyan-400"></i>
                                สัดส่วนความสนใจสาขาวิชาของคณะไอที
                            </h3>
                            <p class="text-xs text-slate-400">วิเคราะห์เปอร์เซ็นต์ความสนใจของผู้ลงทะเบียน</p>
                        </div>
                    </div>
                    <div class="relative w-full h-[320px] flex items-center justify-center">
                        <canvas id="majorDoughnutChart"></canvas>
                    </div>
                </div>

                <!-- Chart 2: Education Level Bar Chart -->
                <div class="lg:col-span-5 glass-card p-6 rounded-3xl border border-slate-800 flex flex-col justify-between">
                    <div class="flex items-center justify-between mb-4">
                        <div>
                            <h3 class="text-lg font-bold text-white flex items-center gap-2">
                                <i class="fa-solid fa-chart-column text-indigo-400"></i>
                                สถิติจำนวนตามระดับชั้นเรียน
                            </h3>
                            <p class="text-xs text-slate-400">การกระจายตัวของนักเรียนแต่ละชั้น</p>
                        </div>
                    </div>
                    <div class="relative w-full h-[320px] flex items-center justify-center">
                        <canvas id="educationBarChart"></canvas>
                    </div>
                </div>

            </div>

            <!-- Data Table Section -->
            <div class="glass-card rounded-3xl border border-slate-800 overflow-hidden">
                <div class="p-6 border-b border-slate-800/80 flex flex-col sm:flex-row justify-between items-start sm:items-center gap-4">
                    <div>
                        <h3 class="text-lg font-bold text-white flex items-center gap-2">
                            <i class="fa-solid fa-table-list text-cyan-400"></i>
                            ตารางรายชื่อผู้ลงทะเบียนทั้งหมด
                        </h3>
                        <p class="text-xs text-slate-400">แสดงรายการลงทะเบียนพร้อมสถานะการส่งเข้า Google Sheet</p>
                    </div>
                    <div class="flex flex-wrap items-center gap-2.5 w-full sm:w-auto">
                        <!-- Search Bar -->
                        <div class="relative flex-grow sm:w-60">
                            <i class="fa-solid fa-magnifying-glass absolute left-3 top-3 text-slate-500 text-xs"></i>
                            <input type="text" id="searchInput" oninput="filterTable()" placeholder="ค้นหาชื่อ โรงเรียน สาขา..." class="glass-input w-full pl-9 pr-3 py-1.5 text-xs rounded-xl outline-none">
                        </div>
                        
                        <!-- CSV Export Button -->
                        <button onclick="exportToCSV()" class="px-3.5 py-2 rounded-xl bg-slate-800 hover:bg-slate-700 text-slate-200 text-xs font-medium transition-all border border-slate-700 flex items-center gap-1.5">
                            <i class="fa-solid fa-file-csv text-emerald-400"></i>
                            <span>โหลด CSV</span>
                        </button>

                        <!-- Force Re-sync to Google Sheet -->
                        <button onclick="resyncAllToGoogleSheet()" class="px-3.5 py-2 rounded-xl bg-emerald-600 hover:bg-emerald-500 text-white text-xs font-medium transition-all shadow-glow-cyan flex items-center gap-1.5">
                            <i class="fa-solid fa-rotate"></i>
                            <span>Sync ไปยัง Sheet</span>
                        </button>
                    </div>
                </div>

                <!-- Table Content -->
                <div class="overflow-x-auto">
                    <table class="w-full text-left text-xs border-collapse">
                        <thead>
                            <tr class="bg-slate-900/80 border-b border-slate-800 text-slate-400 font-semibold uppercase tracking-wider">
                                <th class="py-3.5 px-4">#</th>
                                <th class="py-3.5 px-4">ชื่อ - นามสกุล</th>
                                <th class="py-3.5 px-4">เพศ / อายุ</th>
                                <th class="py-3.5 px-4">ระดับชั้น</th>
                                <th class="py-3.5 px-4">โรงเรียน</th>
                                <th class="py-3.5 px-4">สาขาวิชาที่สนใจ</th>
                                <th class="py-3.5 px-4">ช่องทางติดต่อ</th>
                                <th class="py-3.5 px-4 text-center">Google Sheet</th>
                                <th class="py-3.5 px-4 text-center">จัดการ</th>
                            </tr>
                        </thead>
                        <tbody id="studentTableBody" class="divide-y divide-slate-800/60 text-slate-300">
                            <!-- Dynamically populated rows -->
                        </tbody>
                    </table>
                </div>

                <!-- Empty State -->
                <div id="emptyTableState" class="hidden p-10 text-center text-slate-500">
                    <i class="fa-regular fa-folder-open text-4xl mb-3 text-slate-600"></i>
                    <p class="text-sm">ยังไม่มีข้อมูลผู้ลงทะเบียนในขณะนี้</p>
                </div>
            </div>

        </section>

    </main>

    <!-- ========================================== -->
    <!-- GOOGLE SHEET CONFIGURATION MODAL           -->
    <!-- ========================================== -->
    <div id="googleSheetModal" class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-slate-950/80 backdrop-blur-md hidden transition-opacity duration-300">
        <div class="glass-card w-full max-w-2xl rounded-3xl p-6 sm:p-8 border border-slate-800 shadow-2xl space-y-6 relative max-h-[90vh] overflow-y-auto">
            
            <button onclick="closeGoogleSheetModal()" class="absolute top-6 right-6 text-slate-400 hover:text-white p-2">
                <i class="fa-solid fa-xmark text-lg"></i>
            </button>

            <div>
                <h3 class="text-xl font-bold text-white flex items-center gap-2.5">
                    <i class="fa-solid fa-file-excel text-emerald-400"></i>
                    เชื่อมต่อ Google Sheet (Google Apps Script)
                </h3>
                <p class="text-xs text-slate-400 mt-1">
                    เมื่อผู้เข้าร่วมลงทะเบียน ข้อมูลจะถูกบันทึกและส่งตรงไปยัง Google Sheet ของคุณทันที
                </p>
            </div>

            <!-- Webhook Input Form -->
            <div class="space-y-4">
                <div>
                    <label class="block text-xs font-semibold text-slate-300 uppercase tracking-wider mb-2">
                        Google Apps Script Web App URL
                    </label>
                    <input type="url" id="gs-webhook-url" placeholder="https://script.google.com/macros/s/AKfycb.../exec" class="glass-input w-full px-4 py-3 rounded-xl text-xs outline-none">
                    <p class="text-[11px] text-slate-500 mt-1">วาง Web App URL ที่สร้างจาก Google Apps Script ของท่าน</p>
                </div>

                <!-- Step-by-Step Instructions -->
                <div class="bg-slate-900/90 rounded-2xl p-4 border border-slate-800 text-xs space-y-3">
                    <p class="font-semibold text-emerald-400 flex items-center gap-2">
                        <i class="fa-solid fa-circle-info"></i> วิธีสร้าง Webhook ใน Google Sheet (ทำเพียงครั้งเดียว):
                    </p>
                    <ol class="list-decimal list-inside space-y-1.5 text-slate-300 font-light text-[11px] leading-relaxed">
                        <li>เปิด Google Sheet ของท่าน แล้วไปที่เมนู <b>ส่วนขยาย (Extensions) &gt; Apps Script</b></li>
                        <li>วางโค้ด Apps Script ด้านล่างนี้แทนที่โค้ดเดิม แล้วกดบันทึก (Save)</li>
                        <li>กดปุ่ม <b>ทำให้ใช้งานได้ (Deploy) &gt; การทําการปรับใช้งานใหม่ (New deployment)</b></li>
                        <li>เลือกประเภทเป็น <b>เว็บแอป (Web App)</b></li>
                        <li>ตั้งค่า <i>"ผู้ที่มีสิทธิ์เข้าถึง (Who has access)"</i> เป็น <b>"ทุกคน (Anyone)"</b> แล้วกดทำให้ใช้งานได้</li>
                        <li>คัดลอก Web App URL นำมาวางในช่องด้านบนแล้วกดบันทึก!</li>
                    </ol>

                    <!-- Apps Script Code Snippet -->
                    <div class="relative mt-2">
                        <pre class="bg-slate-950 p-3 rounded-xl border border-slate-800 text-[10px] text-emerald-300 overflow-x-auto select-all">
function doPost(e) {
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  var data = JSON.parse(e.postData.contents);
  
  // สร้าง Header หากแผ่นงานยังว่างอยู่
  if (sheet.getLastRow() === 0) {
    sheet.appendRow(["เวลา", "คำนำหน้า", "ชื่อ-นามสกุล", "เพศ", "อายุ", "ระดับชั้น", "โรงเรียน", "เบอร์โทร", "Line ID", "สาขาที่สนใจ"]);
  }
  
  sheet.appendRow([
    data.timestamp || new Date().toLocaleString(),
    data.prefix,
    data.fullname,
    data.gender,
    data.age,
    data.educationLevel,
    data.school,
    data.phone,
    data.lineId,
    data.interestedMajor
  ]);
  
  return ContentService.createTextOutput(JSON.stringify({"result": "success"}))
    .setMimeType(ContentService.MimeType.JSON);
}</pre>
                        <button onclick="copyScriptCode()" class="absolute top-2 right-2 px-2.5 py-1 rounded bg-slate-800 hover:bg-slate-700 text-white text-[10px] border border-slate-700">
                            <i class="fa-regular fa-copy"></i> ก๊อปปี้โค้ด
                        </button>
                    </div>
                </div>
            </div>

            <!-- Modal Action Buttons -->
            <div class="flex items-center justify-between pt-2">
                <button onclick="clearGoogleSheetUrl()" class="px-4 py-2.5 rounded-xl border border-red-900/50 bg-red-950/30 text-red-400 text-xs hover:bg-red-900/40">
                    ยกเลิกการเชื่อมต่อ
                </button>
                <div class="flex gap-2">
                    <button onclick="closeGoogleSheetModal()" class="px-4 py-2.5 rounded-xl border border-slate-800 text-slate-400 text-xs hover:bg-slate-900">
                        ปิด
                    </button>
                    <button onclick="saveGoogleSheetSettings()" class="px-5 py-2.5 rounded-xl bg-emerald-600 hover:bg-emerald-500 text-white font-semibold text-xs shadow-glow-cyan">
                        บันทึกการตั้งค่า
                    </button>
                </div>
            </div>

        </div>
    </div>

    <!-- Toast Notification Banner -->
    <div id="toast" class="fixed bottom-6 right-6 transform translate-y-24 opacity-0 transition-all duration-300 z-50 flex items-center gap-3 bg-slate-900/95 text-white px-5 py-4 rounded-2xl shadow-2xl border border-slate-800">
        <div id="toastIcon" class="text-cyan-400 text-xl">
            <i class="fa-solid fa-circle-check"></i>
        </div>
        <div>
            <h4 id="toastTitle" class="font-bold text-xs sm:text-sm text-slate-100">สำเร็จ!</h4>
            <p id="toastMessage" class="text-[11px] sm:text-xs text-slate-400">บันทึกข้อมูลเรียบร้อยแล้ว</p>
        </div>
    </div>

    <!-- JavaScript Application Logic -->
    <script>
        // Initial Mock Data (Visitors sample data)
        const initialData = [
            { id: 1, prefix: "นาย", fullname: "กิตติพงษ์ สุขเจริญ", gender: "ชาย", age: 17, educationLevel: "มัธยมศึกษาปีที่ 6", school: "โรงเรียนสวนกุหลาบวิทยาลัย", phone: "0812345678", lineId: "kitti_p", interestedMajor: "เทคโนโลยีธุรกิจดิจิทัล", timestamp: "10:15 น.", syncedToGS: false },
            { id: 2, prefix: "นางสาว", fullname: "ชลธิชา ประเสริฐดี", gender: "หญิง", age: 18, educationLevel: "มัธยมศึกษาปีที่ 6", school: "โรงเรียนเตรียมอุดมศึกษา", phone: "0898765432", lineId: "chonticha.p", interestedMajor: "เทคโนโลยีสารสนเทศและนวัตกรรมดิจิทัล", timestamp: "10:22 น.", syncedToGS: false },
            { id: 3, prefix: "นาย", fullname: "ธนกฤต วงศ์สว่าง", gender: "ชาย", age: 16, educationLevel: "มัธยมศึกษาปีที่ 5", school: "โรงเรียนบดินทรเดชา", phone: "0823456789", lineId: "thanakrit_m5", interestedMajor: "วิศวกรรมความปลอดภัย", timestamp: "10:45 น.", syncedToGS: false },
            { id: 4, prefix: "นางสาว", fullname: "ปนัดดา มีสุข", gender: "หญิง", age: 17, educationLevel: "มัธยมศึกษาปีที่ 6", school: "โรงเรียนสามเสนวิทยาลัย", phone: "0834567890", lineId: "panadda_art", interestedMajor: "ดิจิทัลมีเดียอาร์ต", timestamp: "11:05 น.", syncedToGS: false },
            { id: 5, prefix: "นาย", fullname: "ณัฐภัทร ลิ้มอนันต์", gender: "ชาย", age: 18, educationLevel: "ปวช.", school: "วิทยาลัยเทคโนโลยีพณิชยการ", phone: "0845678901", lineId: "nat_sec", interestedMajor: "วิศวกรรมความปลอดภัย", timestamp: "11:30 น.", syncedToGS: false }
        ];

        // Core App State
        let studentsData = JSON.parse(localStorage.getItem('openhouse_students_v2')) || initialData;
        let googleSheetUrl = localStorage.getItem('openhouse_gs_url') || '';
        let doughnutChart = null;
        let barChart = null;

        // Major Color Tokens
        const majorTokens = {
            "เทคโนโลยีธุรกิจดิจิทัล": { bg: "#06b6d4", badge: "bg-cyan-950 text-cyan-400 border-cyan-800" },
            "เทคโนโลยีสารสนเทศและนวัตกรรมดิจิทัล": { bg: "#6366f1", badge: "bg-indigo-950 text-indigo-400 border-indigo-800" },
            "ดิจิทัลมีเดียอาร์ต": { bg: "#a855f7", badge: "bg-purple-950 text-purple-400 border-purple-800" },
            "วิศวกรรมความปลอดภัย": { bg: "#10b981", badge: "bg-emerald-950 text-emerald-400 border-emerald-800" },
            "ไม่สนใจ": { bg: "#64748b", badge: "bg-slate-900 text-slate-400 border-slate-800" }
        };

        // Window Load Initialization
        window.onload = function() {
            updateGoogleSheetStatusUI();
            renderTable();
            updateDashboard();
        };

        // View Navigation Switcher
        function switchView(viewName) {
            const formView = document.getElementById('view-form');
            const dashboardView = document.getElementById('view-dashboard');
            const btnForm = document.getElementById('tab-form-btn');
            const btnDashboard = document.getElementById('tab-dashboard-btn');

            if (viewName === 'form') {
                formView.classList.remove('hidden');
                dashboardView.classList.add('hidden');
                btnForm.className = "flex items-center space-x-2 px-4 py-2 rounded-lg text-xs sm:text-sm font-medium transition-all bg-gradient-to-r from-cyan-500 to-blue-600 text-white shadow-md";
                btnDashboard.className = "flex items-center space-x-2 px-4 py-2 rounded-lg text-xs sm:text-sm font-medium transition-all text-slate-400 hover:text-white hover:bg-slate-800/60";
            } else {
                formView.classList.add('hidden');
                dashboardView.classList.remove('hidden');
                btnDashboard.className = "flex items-center space-x-2 px-4 py-2 rounded-lg text-xs sm:text-sm font-medium transition-all bg-gradient-to-r from-cyan-500 to-blue-600 text-white shadow-md";
                btnForm.className = "flex items-center space-x-2 px-4 py-2 rounded-lg text-xs sm:text-sm font-medium transition-all text-slate-400 hover:text-white hover:bg-slate-800/60";
                
                // Re-render dashboard visualizations
                updateDashboard();
            }
        }

        // Save State to LocalStorage
        function saveState() {
            localStorage.setItem('openhouse_students_v2', JSON.stringify(studentsData));
        }

        // Form Submission Handler
        async function handleRegistrationSubmit(event) {
            event.preventDefault();

            const prefix = document.getElementById('prefix').value;
            const fullname = document.getElementById('fullname').value.trim();
            const gender = document.getElementById('gender').value;
            const age = parseInt(document.getElementById('age').value);
            const educationLevel = document.getElementById('educationLevel').value;
            const school = document.getElementById('school').value.trim();
            const phone = document.getElementById('phone').value.trim();
            const lineId = document.getElementById('lineId').value.trim();
            
            const selectedMajorRadio = document.querySelector('input[name="interestedMajor"]:checked');
            if (!selectedMajorRadio) {
                showToast('กรุณาเลือกสาขา', 'กรุณาเลือกสาขาวิชาที่สนใจอย่างน้อย 1 รายการ', 'error');
                return;
            }
            const interestedMajor = selectedMajorRadio.value;

            // Loading state on button
            const submitBtnText = document.getElementById('submitBtnText');
            const originalText = submitBtnText.innerText;
            submitBtnText.innerText = 'กำลังบันทึกข้อมูล...';

            // Create new record
            const newStudent = {
                id: Date.now(),
                prefix,
                fullname,
                gender,
                age,
                educationLevel,
                school,
                phone,
                lineId,
                interestedMajor,
                timestamp: new Date().toLocaleTimeString('th-TH', { hour: '2-digit', minute: '2-digit' }) + " น.",
                syncedToGS: false
            };

            // Send to Google Sheet if configured
            if (googleSheetUrl) {
                const isSent = await sendToGoogleSheet(newStudent);
                newStudent.syncedToGS = isSent;
            }

            // Push to local state & save
            studentsData.unshift(newStudent);
            saveState();

            // Reset form UI
            document.getElementById('studentRegistrationForm').reset();
            submitBtnText.innerText = originalText;

            // Toast feedback
            if (newStudent.syncedToGS) {
                showToast('ลงทะเบียนสำเร็จ!', `บันทึกข้อมูลและส่งเข้า Google Sheet เรียบร้อยแล้ว`, 'success');
            } else if (googleSheetUrl) {
                showToast('ลงทะเบียนสำเร็จ (ออฟไลน์)', `บันทึกในเครื่องแล้ว (ไม่สามารถส่งไป Google Sheet ได้ในขณะนี้)`, 'info');
            } else {
                showToast('ลงทะเบียนสำเร็จ!', `ยินดีต้อนรับคุณ ${fullname} เข้าสู่ IT Open House`, 'success');
            }

            // Update UI
            renderTable();
            updateDashboard();
        }

        // Send Record Payload to Google Apps Script Webhook
        async function sendToGoogleSheet(record) {
            if (!googleSheetUrl) return false;

            try {
                // Using no-cors mode for cross-domain Google Apps Script POST
                await fetch(googleSheetUrl, {
                    method: 'POST',
                    mode: 'no-cors',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(record)
                });
                return true;
            } catch (err) {
                console.error('Google Sheet Sync Error:', err);
                return false;
            }
        }

        // Render Table Data
        function renderTable(filterQuery = '') {
            const tbody = document.getElementById('studentTableBody');
            const emptyState = document.getElementById('emptyTableState');
            tbody.innerHTML = '';

            const filtered = studentsData.filter(item => {
                const q = filterQuery.toLowerCase();
                return item.fullname.toLowerCase().includes(q) ||
                       item.school.toLowerCase().includes(q) ||
                       item.interestedMajor.toLowerCase().includes(q) ||
                       item.educationLevel.toLowerCase().includes(q);
            });

            if (filtered.length === 0) {
                emptyState.classList.remove('hidden');
                return;
            } else {
                emptyState.classList.add('hidden');
            }

            filtered.forEach((student, index) => {
                const tr = document.createElement('tr');
                tr.className = "hover:bg-slate-900/50 transition-colors";

                const token = majorTokens[student.interestedMajor] || majorTokens['ไม่สนใจ'];

                // Google Sheet status pill
                const gsBadge = student.syncedToGS 
                    ? `<span class="inline-flex items-center gap-1 px-2 py-0.5 rounded-full text-[10px] font-medium bg-emerald-950 text-emerald-400 border border-emerald-800"><i class="fa-solid fa-check"></i> ส่งแล้ว</span>`
                    : (googleSheetUrl 
                        ? `<button onclick="syncSingleRecord(${student.id})" class="inline-flex items-center gap-1 px-2 py-0.5 rounded-full text-[10px] font-medium bg-amber-950 text-amber-400 border border-amber-800 hover:bg-amber-900"><i class="fa-solid fa-arrow-rotate-right"></i> ลองใหม่</button>`
                        : `<span class="text-slate-500 text-[10px]">-</span>`);

                tr.innerHTML = `
                    <td class="py-3.5 px-4 font-mono text-slate-500 text-[11px]">${index + 1}</td>
                    <td class="py-3.5 px-4 font-semibold text-white">${student.prefix}${student.fullname}</td>
                    <td class="py-3.5 px-4 text-slate-400">${student.gender} (${student.age} ปี)</td>
                    <td class="py-3.5 px-4 font-medium text-slate-300">${student.educationLevel}</td>
                    <td class="py-3.5 px-4 text-slate-300 max-w-[160px] truncate">${student.school}</td>
                    <td class="py-3.5 px-4">
                        <span class="inline-flex items-center px-2.5 py-1 rounded-full text-[11px] font-medium border ${token.badge}">
                            ${student.interestedMajor}
                        </span>
                    </td>
                    <td class="py-3.5 px-4">
                        <div class="text-slate-300 font-mono"><i class="fa-solid fa-phone text-slate-500 mr-1"></i>${student.phone}</div>
                        <div class="text-slate-400"><i class="fa-brands fa-line text-emerald-400 mr-1"></i>${student.lineId}</div>
                    </td>
                    <td class="py-3.5 px-4 text-center">${gsBadge}</td>
                    <td class="py-3.5 px-4 text-center">
                        <button onclick="deleteStudent(${student.id})" class="text-slate-500 hover:text-red-400 p-1.5 rounded-lg hover:bg-red-950/40 transition-all">
                            <i class="fa-regular fa-trash-can"></i>
                        </button>
                    </td>
                `;
                tbody.appendChild(tr);
            });
        }

        // Delete Student Record
        function deleteStudent(id) {
            studentsData = studentsData.filter(s => s.id !== id);
            saveState();
            renderTable();
            updateDashboard();
            showToast('ลบข้อมูลสำเร็จ', 'รายการถูกลบออกจากระบบเรียบร้อยแล้ว', 'info');
        }

        // Filter Table Handler
        function filterTable() {
            const query = document.getElementById('searchInput').value;
            renderTable(query);
        }

        // Sync Single Pending Record to Google Sheet
        async function syncSingleRecord(id) {
            const record = studentsData.find(s => s.id === id);
            if (!record) return;

            showToast('กำลัง Sync...', 'กำลังส่งข้อมูลไปยัง Google Sheet', 'info');
            const success = await sendToGoogleSheet(record);

            if (success) {
                record.syncedToGS = true;
                saveState();
                renderTable();
                showToast('Sync สำเร็จ!', 'ส่งข้อมูลเข้า Google Sheet เรียบร้อยแล้ว', 'success');
            } else {
                showToast('Sync ไม่สำเร็จ', 'กรุณาตรวจสอบ Webhook URL ในการตั้งค่า', 'error');
            }
        }

        // Resync All Pending Records to Google Sheet
        async function resyncAllToGoogleSheet() {
            if (!googleSheetUrl) {
                openGoogleSheetModal();
                showToast('โปรดตั้งค่า Webhook', 'กรุณาตั้งค่า URL ของ Google Sheet ก่อน', 'info');
                return;
            }

            const pending = studentsData.filter(s => !s.syncedToGS);
            if (pending.length === 0) {
                showToast('ข้อมูลอัปเดตแล้ว', 'ข้อมูลทั้งหมดถูกส่งเข้า Google Sheet เรียบร้อยแล้ว', 'info');
                return;
            }

            showToast('กำลัง Sync ทั้งหมด...', `กำลังส่งข้อมูล ${pending.length} รายการไปยัง Google Sheet`, 'info');

            let count = 0;
            for (const record of pending) {
                const success = await sendToGoogleSheet(record);
                if (success) {
                    record.syncedToGS = true;
                    count++;
                }
            }

            saveState();
            renderTable();
            showToast('Sync เสร็จสิ้น', `ส่งข้อมูลเข้า Google Sheet เพิ่มเติม ${count} รายการ`, 'success');
        }

        // Update Dashboard Visualizations & Metrics
        function updateDashboard() {
            // Stat 1: Total
            document.getElementById('stat-total').innerText = studentsData.length;

            // Stat 2: Unique Schools
            const schools = new Set(studentsData.map(s => s.school.trim())).size;
            document.getElementById('stat-schools').innerText = schools;

            // Major Counts
            const majorCounts = {
                "เทคโนโลยีธุรกิจดิจิทัล": 0,
                "เทคโนโลยีสารสนเทศและนวัตกรรมดิจิทัล": 0,
                "ดิจิทัลมีเดียอาร์ต": 0,
                "วิศวกรรมความปลอดภัย": 0,
                "ไม่สนใจ": 0
            };

            studentsData.forEach(s => {
                if (majorCounts[s.interestedMajor] !== undefined) {
                    majorCounts[s.interestedMajor]++;
                }
            });

            // Find Top Major
            let topMajor = "-";
            let topCount = 0;
            for (const [key, val] of Object.entries(majorCounts)) {
                if (val > topCount && key !== "ไม่สนใจ") {
                    topCount = val;
                    topMajor = key;
                }
            }
            document.getElementById('stat-top-major').innerText = topMajor;
            document.getElementById('stat-top-count').innerText = `${topCount} คนสนใจ`;

            // Charts Render
            renderMajorChart(majorCounts);
            renderEducationChart();
        }

        // Render Doughnut Chart for Major Interest
        function renderMajorChart(counts) {
            const ctx = document.getElementById('majorDoughnutChart').getContext('2d');
            if (doughnutChart) doughnutChart.destroy();

            const labels = Object.keys(counts);
            const dataValues = Object.values(counts);
            const colors = labels.map(l => majorTokens[l].bg);

            doughnutChart = new Chart(ctx, {
                type: 'doughnut',
                data: {
                    labels: labels,
                    datasets: [{
                        data: dataValues,
                        backgroundColor: colors,
                        borderWidth: 2,
                        borderColor: '#0f172a',
                        hoverOffset: 8
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        legend: {
                            position: 'bottom',
                            labels: {
                                color: '#94a3b8',
                                font: { family: 'Prompt', size: 11 },
                                padding: 14,
                                usePointStyle: true
                            }
                        },
                        tooltip: {
                            bodyFont: { family: 'Prompt' },
                            titleFont: { family: 'Prompt', weight: 'bold' },
                            callbacks: {
                                label: function(ctx) {
                                    const total = ctx.dataset.data.reduce((a, b) => a + b, 0);
                                    const val = ctx.raw;
                                    const pct = total > 0 ? ((val / total) * 100).toFixed(1) : 0;
                                    return ` ${ctx.label}: ${val} คน (${pct}%)`;
                                }
                            }
                        }
                    },
                    cutout: '68%'
                }
            });
        }

        // Render Bar Chart for Education Levels
        function renderEducationChart() {
            const ctx = document.getElementById('educationBarChart').getContext('2d');
            if (barChart) barChart.destroy();

            const eduCounts = {};
            studentsData.forEach(s => {
                eduCounts[s.educationLevel] = (eduCounts[s.educationLevel] || 0) + 1;
            });

            barChart = new Chart(ctx, {
                type: 'bar',
                data: {
                    labels: Object.keys(eduCounts),
                    datasets: [{
                        label: 'จำนวนนักเรียน (คน)',
                        data: Object.values(eduCounts),
                        backgroundColor: '#6366f1',
                        borderRadius: 8,
                        hoverBackgroundColor: '#818cf8'
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        legend: { display: false },
                        tooltip: { bodyFont: { family: 'Prompt' } }
                    },
                    scales: {
                        y: {
                            beginAtZero: true,
                            ticks: { color: '#94a3b8', precision: 0, font: { family: 'Prompt' } },
                            grid: { color: 'rgba(255,255,255,0.05)' }
                        },
                        x: {
                            ticks: { color: '#94a3b8', font: { family: 'Prompt', size: 11 } },
                            grid: { display: false }
                        }
                    }
                }
            });
        }

        // Google Sheet Integration Settings UI Controls
        function openGoogleSheetModal() {
            document.getElementById('gs-webhook-url').value = googleSheetUrl;
            document.getElementById('googleSheetModal').classList.remove('hidden');
        }

        function closeGoogleSheetModal() {
            document.getElementById('googleSheetModal').classList.add('hidden');
        }

        function saveGoogleSheetSettings() {
            const url = document.getElementById('gs-webhook-url').value.trim();
            googleSheetUrl = url;
            localStorage.setItem('openhouse_gs_url', url);
            closeGoogleSheetModal();
            updateGoogleSheetStatusUI();
            renderTable();

            if (url) {
                showToast('บันทึกการเชื่อมต่อแล้ว', 'ระบบพร้อมส่งข้อมูลไปยัง Google Sheet ทันทีที่มีการลงทะเบียน', 'success');
            } else {
                showToast('ยกเลิกการเชื่อมต่อแล้ว', 'ระบบเปลี่ยนเป็นโหมดบันทึกข้อมูลในเครื่องเท่านั้น', 'info');
            }
        }

        function clearGoogleSheetUrl() {
            document.getElementById('gs-webhook-url').value = '';
            googleSheetUrl = '';
            localStorage.removeItem('openhouse_gs_url');
            closeGoogleSheetModal();
            updateGoogleSheetStatusUI();
            renderTable();
            showToast('ยกเลิกการเชื่อมต่อ', 'ลบ URL การเชื่อมต่อ Google Sheet เรียบร้อยแล้ว', 'info');
        }

        function updateGoogleSheetStatusUI() {
            const btnText = document.getElementById('gs-status-btn-text');
            const btnTextMobile = document.getElementById('gs-status-btn-text-mobile');
            const dot = document.getElementById('gs-status-dot');
            const dotMobile = document.getElementById('gs-status-dot-mobile');
            const formNoticeText = document.getElementById('form-gs-notice-text');
            const statGsStatus = document.getElementById('stat-gs-status');
            const statGsSubtext = document.getElementById('stat-gs-subtext');

            if (googleSheetUrl) {
                btnText.innerText = "Google Sheet: เชื่อมต่อแล้ว";
                btnTextMobile.innerText = "Google Sheet: เชื่อมต่อแล้ว";
                dot.className = "w-2.5 h-2.5 rounded-full bg-emerald-400 shadow-glow-cyan animate-pulse";
                dotMobile.className = "w-2.5 h-2.5 rounded-full bg-emerald-400 shadow-glow-cyan animate-pulse";
                formNoticeText.innerHTML = `<span class="text-emerald-400 font-semibold"><i class="fa-solid fa-circle-check"></i> เชื่อมต่อ Google Sheet แล้ว</span> ข้อมูลจะถูกบันทึกลง Sheet อัตโนมัติ`;
                statGsStatus.innerText = "เชื่อมต่อแล้ว";
                statGsStatus.className = "text-sm font-bold text-emerald-400";
                statGsSubtext.innerText = "พร้อมส่งข้อมูล Auto-Sync";
            } else {
                btnText.innerText = "ตั้งค่า Google Sheet";
                btnTextMobile.innerText = "ตั้งค่า Google Sheet Integration";
                dot.className = "w-2 h-2 rounded-full bg-slate-500";
                dotMobile.className = "w-2.5 h-2.5 rounded-full bg-slate-500";
                formNoticeText.innerText = "โหมดบันทึกข้อมูลในเครื่อง (คุณสามารถกดเปิดการเชื่อมต่อ Google Sheet ได้)";
                statGsStatus.innerText = "ไม่ได้เชื่อมต่อ";
                statGsStatus.className = "text-sm font-bold text-slate-400";
                statGsSubtext.innerText = "คลิกเพื่อตั้งค่า Webhook";
            }
        }

        function copyScriptCode() {
            const code = `function doPost(e) {
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  var data = JSON.parse(e.postData.contents);
  if (sheet.getLastRow() === 0) {
    sheet.appendRow(["เวลา", "คำนำหน้า", "ชื่อ-นามสกุล", "เพศ", "อายุ", "ระดับชั้น", "โรงเรียน", "เบอร์โทร", "Line ID", "สาขาที่สนใจ"]);
  }
  sheet.appendRow([
    data.timestamp || new Date().toLocaleString(),
    data.prefix,
    data.fullname,
    data.gender,
    data.age,
    data.educationLevel,
    data.school,
    data.phone,
    data.lineId,
    data.interestedMajor
  ]);
  return ContentService.createTextOutput(JSON.stringify({"result": "success"}))
    .setMimeType(ContentService.MimeType.JSON);
}`;
            
            navigator.clipboard ? navigator.clipboard.writeText(code) : document.execCommand('copy');
            showToast('คัดลอกโค้ดสำเร็จ', 'นำโค้ดไปวางใน Google Apps Script ได้เลย', 'success');
        }

        // Export Data to CSV File
        function exportToCSV() {
            if (studentsData.length === 0) {
                showToast('ไม่มีข้อมูล', 'ไม่มีรายการสำหรับส่งออก CSV', 'info');
                return;
            }

            let csv = "\uFEFF"; // UTF-8 BOM
            csv += "ลำดับ,คำนำหน้า,ชื่อ-นามสกุล,เพศ,อายุ,ระดับชั้น,โรงเรียน,เบอร์โทร,Line ID,สาขาที่สนใจ,เวลาลงทะเบียน\n";

            studentsData.forEach((s, idx) => {
                const row = [
                    idx + 1,
                    `"${s.prefix}"`,
                    `"${s.fullname}"`,
                    `"${s.gender}"`,
                    s.age,
                    `"${s.educationLevel}"`,
                    `"${s.school}"`,
                    `"${s.phone}"`,
                    `"${s.lineId}"`,
                    `"${s.interestedMajor}"`,
                    `"${s.timestamp}"`
                ].join(",");
                csv += row + "\n";
            });

            const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' });
            const url = URL.createObjectURL(blob);
            const a = document.createElement("a");
            a.href = url;
            a.download = `IT_OpenHouse_Students_${new Date().toISOString().slice(0,10)}.csv`;
            document.body.appendChild(a);
            a.click();
            document.body.removeChild(a);

            showToast('ดาวน์โหลดไฟล์ CSV เรียบร้อย', 'ไฟล์ถูกส่งออกเข้าเครื่องของคุณแล้ว', 'success');
        }

        // Toast Notification Function
        function showToast(title, message, type = 'success') {
            const toast = document.getElementById('toast');
            const toastTitle = document.getElementById('toastTitle');
            const toastMessage = document.getElementById('toastMessage');
            const toastIcon = document.getElementById('toastIcon');

            toastTitle.innerText = title;
            toastMessage.innerText = message;

            if (type === 'success') {
                toastIcon.innerHTML = '<i class="fa-solid fa-circle-check text-emerald-400"></i>';
            } else if (type === 'error') {
                toastIcon.innerHTML = '<i class="fa-solid fa-circle-exclamation text-rose-400"></i>';
            } else {
                toastIcon.innerHTML = '<i class="fa-solid fa-circle-info text-cyan-400"></i>';
            }

            toast.classList.remove('translate-y-24', 'opacity-0');
            toast.classList.add('translate-y-0', 'opacity-100');

            setTimeout(() => {
                toast.classList.remove('translate-y-0', 'opacity-100');
                toast.classList.add('translate-y-24', 'opacity-0');
            }, 3500);
        }
    </script>
</body>
</html>
