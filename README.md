# IP-Portfolio<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>IP Portfolio Dashboard - RMUTP</title>
    
    <!-- Fonts -->
    <link href="https://fonts.googleapis.com/css2?family=Sarabun:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    
    <!-- Chart.js -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    
    <!-- PapaParse for CSV reading -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.4.1/papaparse.min.js"></script>
    
    <!-- FontAwesome -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">

    <style>
        body {
            font-family: 'Sarabun', sans-serif;
            background-color: #f3f4f6;
        }
        .custom-scrollbar::-webkit-scrollbar {
            width: 6px;
            height: 6px;
        }
        .custom-scrollbar::-webkit-scrollbar-track {
            background: #f1f1f1;
            border-radius: 4px;
        }
        .custom-scrollbar::-webkit-scrollbar-thumb {
            background: #cbd5e1;
            border-radius: 4px;
        }
        .custom-scrollbar::-webkit-scrollbar-thumb:hover {
            background: #94a3b8;
        }
        .glass-card {
            background: white;
            border-radius: 12px;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.05), 0 2px 4px -1px rgba(0, 0, 0, 0.03);
            border: 1px solid #e5e7eb;
        }
        /* Spinner */
        .loader {
            border: 4px solid #f3f3f3;
            border-top: 4px solid #4f46e5;
            border-radius: 50%;
            width: 30px;
            height: 30px;
            animation: spin 1s linear infinite;
        }
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
    </style>
</head>
<body class="text-gray-800 h-screen flex flex-col overflow-hidden">

    <!-- Top Navigation / Header -->
    <header class="bg-indigo-900 text-white shadow-md z-10 shrink-0">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-4 flex flex-col md:flex-row justify-between items-center gap-4">
            <div class="flex items-center gap-3">
                <div class="bg-white p-2 rounded-lg">
                    <i class="fa-solid fa-lightbulb text-indigo-900 text-2xl"></i>
                </div>
                <div>
                    <h1 class="text-xl font-bold">IP Portfolio Dashboard</h1>
                    <p class="text-indigo-200 text-sm">มหาวิทยาลัยเทคโนโลยีราชมงคลพระนคร</p>
                </div>
            </div>
            
            <div class="flex flex-wrap items-center gap-2 md:gap-4 mt-4 md:mt-0">
                <select id="fileEncoding" class="bg-indigo-700 hover:bg-indigo-600 text-white text-sm rounded-lg border border-indigo-500 p-2.5 outline-none focus:ring-1 focus:ring-white transition-colors cursor-pointer">
                    <option value="UTF-8">เข้ารหัส: UTF-8 (ปกติ)</option>
                    <option value="windows-874">เข้ารหัส: Excel ไทย</option>
                </select>
                <div class="relative">
                    <input type="file" id="csvFileInput" accept=".csv" class="hidden">
                    <label for="csvFileInput" class="cursor-pointer bg-indigo-600 hover:bg-indigo-500 text-white px-5 py-2.5 rounded-lg font-medium transition duration-200 flex items-center gap-2 shadow-sm border border-indigo-500">
                        <i class="fa-solid fa-file-arrow-up"></i>
                        <span class="hidden sm:inline">อัปโหลดไฟล์ข้อมูล (.csv)</span>
                        <span class="sm:hidden">อัปโหลด</span>
                    </label>
                </div>
                <div id="loadingIndicator" class="hidden flex items-center gap-2 text-indigo-200 text-sm">
                    <div class="loader w-4 h-4 border-2"></div> ประมวลผล...
                </div>
            </div>
        </div>
    </header>

    <!-- Main Content Area (Scrollable) -->
    <main class="flex-1 overflow-y-auto custom-scrollbar p-4 md:p-6 lg:p-8">
        
        <!-- Placeholder when no data -->
        <div id="emptyState" class="h-full flex flex-col items-center justify-center text-center space-y-4">
            <img src="https://cdn-icons-png.flaticon.com/512/3206/3206015.png" alt="Upload" class="w-32 h-32 opacity-50 grayscale">
            <h2 class="text-2xl font-bold text-gray-500">กรุณาอัปโหลดไฟล์ IP Portfolio (CSV)</h2>
            <p class="text-gray-400 max-w-md">ระบบจะสร้าง Dashboard และแผนภูมิสรุปผลอัตโนมัติตามข้อมูลทรัพย์สินทางปัญญาของคุณ</p>
            <label for="csvFileInput" class="cursor-pointer mt-4 bg-indigo-600 hover:bg-indigo-700 text-white px-6 py-3 rounded-xl font-medium transition duration-200 shadow-lg">
                <i class="fa-solid fa-cloud-arrow-up mr-2"></i> เลือกไฟล์ CSV
            </label>
        </div>

        <!-- Dashboard Content (Hidden initially) -->
        <div id="dashboardContent" class="hidden max-w-7xl mx-auto space-y-6">
            
            <!-- Filters Section -->
            <div class="glass-card p-5">
                <div class="flex items-center justify-between mb-4">
                    <h2 class="text-lg font-bold text-gray-700"><i class="fa-solid fa-filter mr-2 text-indigo-500"></i>ตัวกรองข้อมูล (Filters)</h2>
                    <button id="resetFilters" class="text-sm text-indigo-600 hover:text-indigo-800 font-medium underline">ล้างตัวกรองทั้งหมด</button>
                </div>
                
                <!-- General Filters Row -->
                <div class="grid grid-cols-1 md:grid-cols-3 gap-4 mb-5">
                    <div>
                        <label class="block text-xs font-semibold text-gray-500 uppercase mb-1">ประเภททรัพย์สินทางปัญญา</label>
                        <select id="filterIpType" class="w-full bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-indigo-500 focus:border-indigo-500 p-2.5 outline-none filter-select"></select>
                    </div>
                    <div>
                        <label class="block text-xs font-semibold text-gray-500 uppercase mb-1">สถานะ</label>
                        <select id="filterStatus" class="w-full bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-indigo-500 focus:border-indigo-500 p-2.5 outline-none filter-select"></select>
                    </div>
                    <div>
                        <label class="block text-xs font-semibold text-gray-500 uppercase mb-1">หน่วยงาน</label>
                        <select id="filterFaculty" class="w-full bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-indigo-500 focus:border-indigo-500 p-2.5 outline-none filter-select"></select>
                    </div>
                </div>

                <!-- Hierarchical Filters Row -->
                <div class="bg-indigo-50 p-4 rounded-xl border border-indigo-100">
                    <h3 class="text-xs font-bold text-indigo-800 uppercase mb-3 flex items-center"><i class="fa-solid fa-sitemap mr-2"></i>การจำแนกกลุ่มอุตสาหกรรม (เชื่อมโยงอัตโนมัติ)</h3>
                    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
                        <div>
                            <label class="block text-xs font-semibold text-gray-500 uppercase mb-1">1. FAC Model</label>
                            <select id="filterFacModel" class="w-full bg-white border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-indigo-500 focus:border-indigo-500 p-2.5 outline-none filter-select-cascade" data-level="FacModel"></select>
                        </div>
                        <div>
                            <label class="block text-xs font-semibold text-gray-500 uppercase mb-1">2. Sector</label>
                            <select id="filterSector" class="w-full bg-white border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-indigo-500 focus:border-indigo-500 p-2.5 outline-none filter-select-cascade" data-level="Sector"></select>
                        </div>
                        <div>
                            <label class="block text-xs font-semibold text-gray-500 uppercase mb-1">3. Sub-Sector</label>
                            <select id="filterSubSector" class="w-full bg-white border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-indigo-500 focus:border-indigo-500 p-2.5 outline-none filter-select-cascade" data-level="SubSector"></select>
                        </div>
                        <div>
                            <label class="block text-xs font-semibold text-gray-500 uppercase mb-1">4. Industry Target</label>
                            <select id="filterIndustry" class="w-full bg-white border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-indigo-500 focus:border-indigo-500 p-2.5 outline-none filter-select-cascade" data-level="Industry"></select>
                        </div>
                    </div>
                </div>
            </div>

            <!-- KPIs -->
            <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
                <div class="glass-card p-5 border-l-4 border-indigo-500">
                    <div class="flex justify-between items-start">
                        <div>
                            <p class="text-sm font-medium text-gray-500">ผลงานทั้งหมด</p>
                            <h3 class="text-3xl font-bold text-gray-800 mt-1" id="kpiTotal">0</h3>
                        </div>
                        <div class="bg-indigo-100 p-3 rounded-full text-indigo-600">
                            <i class="fa-solid fa-cubes text-xl"></i>
                        </div>
                    </div>
                </div>
                <div class="glass-card p-5 border-l-4 border-green-500">
                    <div class="flex justify-between items-start">
                        <div>
                            <p class="text-sm font-medium text-gray-500">ได้รับจดทะเบียน</p>
                            <h3 class="text-3xl font-bold text-gray-800 mt-1" id="kpiGranted">0</h3>
                        </div>
                        <div class="bg-green-100 p-3 rounded-full text-green-600">
                            <i class="fa-solid fa-check-circle text-xl"></i>
                        </div>
                    </div>
                </div>
                <div class="glass-card p-5 border-l-4 border-yellow-500">
                    <div class="flex justify-between items-start">
                        <div>
                            <p class="text-sm font-medium text-gray-500">อยู่ระหว่างดำเนินการ</p>
                            <h3 class="text-3xl font-bold text-gray-800 mt-1" id="kpiPending">0</h3>
                        </div>
                        <div class="bg-yellow-100 p-3 rounded-full text-yellow-600">
                            <i class="fa-solid fa-clock text-xl"></i>
                        </div>
                    </div>
                </div>
                <div class="glass-card p-5 border-l-4 border-purple-500">
                    <div class="flex justify-between items-start">
                        <div>
                            <p class="text-sm font-medium text-gray-500">หน่วยงานที่ยื่นมากที่สุด</p>
                            <h3 class="text-lg font-bold text-gray-800 mt-1 truncate w-40" id="kpiTopFaculty" title="-">-</h3>
                        </div>
                        <div class="bg-purple-100 p-3 rounded-full text-purple-600">
                            <i class="fa-solid fa-building-columns text-xl"></i>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Charts Row 1 (Pie & Doughnut Charts) -->
            <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                <!-- IP Types Chart -->
                <div class="glass-card p-5 flex flex-col h-80">
                    <h3 class="text-sm font-bold text-gray-700 mb-2 uppercase tracking-wide">ประเภททรัพย์สินทางปัญญา</h3>
                    <div class="relative flex-1 w-full h-full">
                        <canvas id="chartIpType"></canvas>
                    </div>
                </div>
                <!-- Faculty Chart -->
                <div class="glass-card p-5 flex flex-col h-80">
                    <h3 class="text-sm font-bold text-gray-700 mb-2 uppercase tracking-wide">หน่วยงาน</h3>
                    <div class="relative flex-1 w-full h-full">
                        <canvas id="chartFacultyPie"></canvas>
                    </div>
                </div>
                <!-- Sector Chart -->
                <div class="glass-card p-5 flex flex-col h-80">
                    <h3 class="text-sm font-bold text-gray-700 mb-2 uppercase tracking-wide">Sectors</h3>
                    <div class="relative flex-1 w-full h-full">
                        <canvas id="chartSectorPie"></canvas>
                    </div>
                </div>
            </div>

            <!-- Charts Row 2 -->
            <div class="grid grid-cols-1 gap-6">
                <!-- Filing by Year -->
                <div class="glass-card p-5 flex flex-col h-80">
                    <h3 class="text-sm font-bold text-gray-700 mb-2 uppercase tracking-wide">แนวโน้มการยื่นคำขอ (รายปี)</h3>
                    <div class="relative flex-1 w-full h-full">
                        <canvas id="chartYear"></canvas>
                    </div>
                </div>
            </div>

            <!-- Data Table -->
            <div class="glass-card p-5 flex flex-col">
                <div class="flex justify-between items-center mb-4">
                    <h3 class="text-lg font-bold text-gray-700"><i class="fa-solid fa-table-list mr-2 text-indigo-500"></i>ฐานข้อมูล (Data Table)</h3>
                    <span id="tableRecordCount" class="text-sm bg-indigo-100 text-indigo-800 py-1 px-3 rounded-full font-medium">0 รายการ</span>
                </div>
                <div class="overflow-x-auto custom-scrollbar border border-gray-200 rounded-lg">
                    <table class="min-w-full text-sm text-left text-gray-500">
                        <thead class="text-xs text-gray-700 uppercase bg-gray-50 sticky top-0 z-0">
                            <tr>
                                <th class="px-4 py-3 whitespace-nowrap">รหัสผลงาน</th>
                                <th class="px-4 py-3 whitespace-nowrap">ประเภท IP</th>
                                <th class="px-4 py-3 whitespace-nowrap">เลขที่คำขอ/ทะเบียน</th>
                                <th class="px-4 py-3 whitespace-nowrap">สถานะ</th>
                                <th class="px-4 py-3 min-w-[250px]">ชื่อผลงาน</th>
                                <th class="px-4 py-3 whitespace-nowrap">หน่วยงาน</th>
                                <th class="px-4 py-3 whitespace-nowrap">ผู้ประดิษฐ์</th>
                                <th class="px-4 py-3 whitespace-nowrap">ปีที่ยื่น</th>
                                <th class="px-4 py-3 whitespace-nowrap">หมดอายุ</th>
                                <th class="px-4 py-3 whitespace-nowrap">วันคงเหลือ</th>
                            </tr>
                        </thead>
                        <tbody id="tableBody" class="divide-y divide-gray-200">
                            <!-- Rows rendered via JS -->
                        </tbody>
                    </table>
                </div>
            </div>

        </div>
    </main>

    <script>
        // --- Global Variables ---
        let globalRawData = [];
        let filteredData = [];
        let chartInstances = {};

        // Data keys mapping
        const KEYS = {
            ID: 'รหัสผลงาน',
            TYPE: 'ประเภท IP',
            APP_NO: 'เลขที่คำขอ',
            REG_NO: 'เลขทะเบียน',
            STATUS: 'สถานะ',
            TITLE: 'ชื่อผลงาน',
            FAC_MODEL: 'FAC Model',
            SECTOR: 'Sector',
            SUB_SECTOR: 'Sub-Sector',
            INDUSTRY: 'Industry Target',
            FACULTY: 'หน่วยงาน',
            INVENTOR: 'ผู้ประดิษฐ์',
            YEAR: 'ปีที่ยื่น',
            PROTECT_YEARS: 'อายุความคุ้มครอง',
            EXPIRE_DATE: 'วันหมดอายุคุ้มครอง'
        };

        // UI Elements
        const fileInput = document.getElementById('csvFileInput');
        const fileEncoding = document.getElementById('fileEncoding');
        const emptyState = document.getElementById('emptyState');
        const dashboardContent = document.getElementById('dashboardContent');
        const loadingIndicator = document.getElementById('loadingIndicator');

        // Color Palette (Extended for many pie slices)
        const colors = [
            '#4f46e5', '#0ea5e9', '#10b981', '#f59e0b', '#ef4444', 
            '#8b5cf6', '#ec4899', '#14b8a6', '#f97316', '#64748b',
            '#34d399', '#fbbf24', '#f87171', '#818cf8', '#a78bfa'
        ];

        // --- Event Listeners ---
        fileInput.addEventListener('change', handleFileUpload);
        
        fileEncoding.addEventListener('change', () => {
            if (fileInput.files.length > 0) {
                handleFileUpload({ target: fileInput });
            }
        });

        // Event for normal filters
        document.querySelectorAll('.filter-select').forEach(select => {
            select.addEventListener('change', applyFilters);
        });

        // Event for hierarchical filters (Cascading logic)
        document.querySelectorAll('.filter-select-cascade').forEach(select => {
            select.addEventListener('change', (e) => {
                const level = e.target.getAttribute('data-level');
                updateCascadeOptions(level);
                applyFilters();
            });
        });

        document.getElementById('resetFilters').addEventListener('click', () => {
            document.querySelectorAll('.filter-select').forEach(select => select.value = 'All');
            document.querySelectorAll('.filter-select-cascade').forEach(select => select.value = 'All');
            updateCascadeOptions('init');
            applyFilters();
        });

        // --- Core Functions ---

        function handleFileUpload(event) {
            const file = event.target.files[0];
            if (!file) return;

            loadingIndicator.classList.remove('hidden');
            loadingIndicator.classList.add('flex');

            const encodingValue = fileEncoding.value;

            Papa.parse(file, {
                header: true,
                skipEmptyLines: true,
                encoding: encodingValue,
                complete: function(results) {
                    processRawData(results.data);
                },
                error: function(err) {
                    alert("เกิดข้อผิดพลาดในการอ่านไฟล์: " + err.message);
                    loadingIndicator.classList.add('hidden');
                    loadingIndicator.classList.remove('flex');
                }
            });
        }

        function processRawData(data) {
            globalRawData = data.map(row => {
                let cleanRow = {};
                for (let key in row) {
                    let cleanKey = key.replace(/^\uFEFF/, '').trim().replace(/\n/g, " ");
                    cleanRow[cleanKey] = row[key];
                }
                
                if (!cleanRow[KEYS.ID]) {
                    const foundKey = Object.keys(cleanRow).find(k => k.includes('รหัสผลงาน') || k.includes('IP'));
                    if (foundKey) KEYS.ID = foundKey;
                }
                
                let rawYear = String(cleanRow[KEYS.YEAR] || '').trim();
                let extractedYearBE = "N/A";
                if (rawYear) {
                    let parts = rawYear.split(/\s+/);
                    let lastPart = parts[parts.length - 1];
                    let numYear = parseInt(lastPart, 10);
                    if (!isNaN(numYear)) {
                        // ประมวลผลและแปลงเป็นปี พ.ศ. ให้ถูกต้อง (พ.ศ. 2560-2570)
                        if (numYear < 100) {
                            extractedYearBE = numYear < 50 ? 2000 + numYear + 543 : 2500 + numYear;
                        } else if (numYear >= 1900 && numYear <= 2100) {
                            extractedYearBE = numYear + 543;
                        } else if (numYear >= 2400) {
                            extractedYearBE = numYear;
                        }
                    }
                }
                cleanRow['_CleanYearBE'] = String(extractedYearBE);

                return cleanRow;
            }).filter(row => row[KEYS.ID] && String(row[KEYS.ID]).trim() !== "");

            if (globalRawData.length === 0) {
                alert("เกิดข้อผิดพลาด: ไม่พบข้อมูล หรือไฟล์อาจไม่ได้ถูกบันทึกแบบ UTF-8\n\nวิธีแก้:\n1. เปิดไฟล์ CSV ใน Excel\n2. ไปที่ File > Save As\n3. เลือกช่อง Save as type เป็น 'CSV UTF-8 (Comma delimited)'\n4. บันทึกและอัปโหลดใหม่อีกครั้ง");
                loadingIndicator.classList.add('hidden');
                loadingIndicator.classList.remove('flex');
                return;
            }

            if (globalRawData.length > 0) {
                let keys = Object.keys(globalRawData[0]);
                KEYS.PROTECT_YEARS = keys.find(k => k.includes('อายุความคุ้มครอง')) || 'อายุความคุ้มครอง';
            }

            filteredData = [...globalRawData];

            emptyState.classList.add('hidden');
            dashboardContent.classList.remove('hidden');
            loadingIndicator.classList.add('hidden');
            loadingIndicator.classList.remove('flex');

            populateFilters();
            updateDashboard();
        }

        // --- Data Aggregation Helpers ---
        function getMappedStatus(rawStatus) {
            let s = String(rawStatus || '').toLowerCase();
            if (s.includes('grant') || s.includes('จดทะเบียน')) return 'Granted';
            if (s.includes('expire') || s.includes('abandon') || s.includes('หมดอายุ') || s.includes('ละทิ้ง') || s.includes('ยกเลิก') || s.includes('ถอน')) return 'Expired';
            return 'Pending'; 
        }

        function countBy(data, key) {
            return data.reduce((acc, curr) => {
                let val = curr[key] || 'ไม่ระบุ';
                if(String(val).trim() === '') val = 'ไม่ระบุ';
                acc[val] = (acc[val] || 0) + 1;
                return acc;
            }, {});
        }

        function sortObjectByValue(obj, limit = null) {
            let sorted = Object.entries(obj).sort((a, b) => b[1] - a[1]);
            if (limit) sorted = sorted.slice(0, limit);
            return {
                labels: sorted.map(item => item[0]),
                data: sorted.map(item => item[1])
            };
        }

        // --- Cascading Filter Logic ---
        function populateDropdownOptions(selectId, dataKey, dataSet, retainValue = null) {
            const select = document.getElementById(selectId);
            const uniqueValues = [...new Set(dataSet.map(item => item[dataKey]).filter(val => val && String(val).trim() !== ''))].sort();
            
            select.innerHTML = '<option value="All">ทั้งหมด (All)</option>';
            uniqueValues.forEach(val => {
                const option = document.createElement('option');
                option.value = val;
                option.textContent = val;
                select.appendChild(option);
            });

            if (retainValue && uniqueValues.includes(retainValue)) {
                select.value = retainValue;
            } else {
                select.value = 'All';
            }
        }

        function updateCascadeOptions(source) {
            const valFac = document.getElementById('filterFacModel').value;
            const valSec = document.getElementById('filterSector').value;
            const valSub = document.getElementById('filterSubSector').value;
            const valInd = document.getElementById('filterIndustry').value;

            // 1. FAC Model (Base Level)
            if (source === 'init') {
                populateDropdownOptions('filterFacModel', KEYS.FAC_MODEL, globalRawData);
            }

            // 2. Sector (Depends on FAC Model)
            if (source === 'FacModel' || source === 'init') {
                let secData = valFac === 'All' ? globalRawData : globalRawData.filter(d => d[KEYS.FAC_MODEL] === valFac);
                populateDropdownOptions('filterSector', KEYS.SECTOR, secData, source === 'init' ? null : valSec);
            }

            // 3. Sub-Sector (Depends on FAC Model + Sector)
            if (source === 'FacModel' || source === 'Sector' || source === 'init') {
                const currentValSec = document.getElementById('filterSector').value;
                let subData = globalRawData;
                if (valFac !== 'All') subData = subData.filter(d => d[KEYS.FAC_MODEL] === valFac);
                if (currentValSec !== 'All') subData = subData.filter(d => d[KEYS.SECTOR] === currentValSec);
                populateDropdownOptions('filterSubSector', KEYS.SUB_SECTOR, subData, source === 'init' ? null : valSub);
            }

            // 4. Industry Target (Depends on FAC Model + Sector + Sub-Sector)
            if (source === 'FacModel' || source === 'Sector' || source === 'SubSector' || source === 'init') {
                const currentValSec = document.getElementById('filterSector').value;
                const currentValSub = document.getElementById('filterSubSector').value;
                let indData = globalRawData;
                if (valFac !== 'All') indData = indData.filter(d => d[KEYS.FAC_MODEL] === valFac);
                if (currentValSec !== 'All') indData = indData.filter(d => d[KEYS.SECTOR] === currentValSec);
                if (currentValSub !== 'All') indData = indData.filter(d => d[KEYS.SUB_SECTOR] === currentValSub);
                populateDropdownOptions('filterIndustry', KEYS.INDUSTRY, indData, source === 'init' ? null : valInd);
            }
        }

        function populateFilters() {
            // 1. IP Type (Dynamic)
            const ipTypeSelect = document.getElementById('filterIpType');
            const uniqueIpTypes = [...new Set(globalRawData.map(item => item[KEYS.TYPE]).filter(val => val))].sort();
            ipTypeSelect.innerHTML = '<option value="All">ทั้งหมด (All)</option>';
            uniqueIpTypes.forEach(val => {
                if(val.trim() !== '') {
                    ipTypeSelect.innerHTML += `<option value="${val}">${val}</option>`;
                }
            });

            // 2. Status (Hardcoded English)
            document.getElementById('filterStatus').innerHTML = `
                <option value="All">ทั้งหมด (All)</option>
                <option value="Pending">Pending</option>
                <option value="Granted">Granted</option>
                <option value="Expired">Expired</option>
            `;

            // 3. Faculty (Predefined List)
            const faculties = [
                "คณะครุศาสตร์อุตสาหกรรม",
                "คณะเทคโนโลยีคหกรรมศาสตร์",
                "คณะเทคโนโลยีสื่อสารมวลชน",
                "คณะบริหารธุรกิจ",
                "คณะวิทยาศาสตร์และเทคโนโลยี",
                "คณะวิศวกรรมศาสตร์",
                "คณะศิลปะศาสตร์",
                "คณะอุตสาหกรรมสิ่งทอและออกแบบแฟชั่น",
                "คณะสถาปัตยกรรมศาสตร์และการออกแบบ"
            ];
            let facHtml = '<option value="All">ทั้งหมด (All)</option>';
            faculties.forEach(f => { facHtml += `<option value="${f}">${f}</option>`; });
            document.getElementById('filterFaculty').innerHTML = facHtml;

            // 4. Initialize Cascading Filters
            updateCascadeOptions('init');
        }

        function applyFilters() {
            const valType = document.getElementById('filterIpType').value;
            const valStatus = document.getElementById('filterStatus').value;
            const valFaculty = document.getElementById('filterFaculty').value;
            
            // Cascading Filters values
            const valFacModel = document.getElementById('filterFacModel').value;
            const valSector = document.getElementById('filterSector').value;
            const valSubSector = document.getElementById('filterSubSector').value;
            const valIndustry = document.getElementById('filterIndustry').value;

            filteredData = globalRawData.filter(item => {
                const matchType = valType === 'All' || item[KEYS.TYPE] === valType;
                
                const mappedStatus = getMappedStatus(item[KEYS.STATUS]);
                const matchStatus = valStatus === 'All' || mappedStatus === valStatus;

                const matchFaculty = valFaculty === 'All' || (item[KEYS.FACULTY] && String(item[KEYS.FACULTY]).trim() === valFaculty);
                
                // Cascade matching
                const matchFacModel = valFacModel === 'All' || (item[KEYS.FAC_MODEL] && String(item[KEYS.FAC_MODEL]).trim() === valFacModel);
                const matchSector = valSector === 'All' || (item[KEYS.SECTOR] && String(item[KEYS.SECTOR]).trim() === valSector);
                const matchSubSector = valSubSector === 'All' || (item[KEYS.SUB_SECTOR] && String(item[KEYS.SUB_SECTOR]).trim() === valSubSector);
                const matchIndustry = valIndustry === 'All' || (item[KEYS.INDUSTRY] && String(item[KEYS.INDUSTRY]).trim() === valIndustry);

                return matchType && matchStatus && matchFaculty && matchFacModel && matchSector && matchSubSector && matchIndustry;
            });

            updateDashboard();
        }

        function updateDashboard() {
            updateKPIs();
            drawChartIpType();
            drawChartFacultyPie();
            drawChartSectorPie();
            drawChartYear();
            renderTable();
        }

        // --- KPIs ---
        function updateKPIs() {
            document.getElementById('kpiTotal').textContent = filteredData.length.toLocaleString();
            
            let grantedCount = 0;
            let pendingCount = 0;
            
            filteredData.forEach(item => {
                let status = getMappedStatus(item[KEYS.STATUS]);
                if(status === 'Granted') grantedCount++;
                if(status === 'Pending') pendingCount++;
            });
            
            document.getElementById('kpiGranted').textContent = grantedCount.toLocaleString();
            document.getElementById('kpiPending').textContent = pendingCount.toLocaleString();

            const facultyCounts = sortObjectByValue(countBy(filteredData, KEYS.FACULTY));
            if(facultyCounts.labels.length > 0 && facultyCounts.labels[0] !== 'ไม่ระบุ') {
                const topFac = facultyCounts.labels[0];
                document.getElementById('kpiTopFaculty').textContent = topFac;
                document.getElementById('kpiTopFaculty').title = topFac;
            } else if (facultyCounts.labels.length > 1){
                 document.getElementById('kpiTopFaculty').textContent = facultyCounts.labels[1];
            } else {
                document.getElementById('kpiTopFaculty').textContent = "-";
            }
        }

        // --- Chart Generation ---
        function destroyChart(id) {
            if (chartInstances[id]) {
                chartInstances[id].destroy();
            }
        }

        function drawChartIpType() {
            const id = 'chartIpType';
            destroyChart(id);
            const counts = countBy(filteredData, KEYS.TYPE);
            const ctx = document.getElementById(id).getContext('2d');
            chartInstances[id] = new Chart(ctx, {
                type: 'doughnut',
                data: {
                    labels: Object.keys(counts),
                    datasets: [{
                        data: Object.values(counts),
                        backgroundColor: colors,
                        borderWidth: 1
                    }]
                },
                options: { 
                    responsive: true, 
                    maintainAspectRatio: false, 
                    plugins: { 
                        legend: { position: 'bottom' } 
                    } 
                }
            });
        }

        function drawChartFacultyPie() {
            const id = 'chartFacultyPie';
            destroyChart(id);
            
            let rawCounts = countBy(filteredData, KEYS.FACULTY);
            
            const ctx = document.getElementById(id).getContext('2d');
            chartInstances[id] = new Chart(ctx, {
                type: 'pie',
                data: {
                    labels: Object.keys(rawCounts),
                    datasets: [{
                        data: Object.values(rawCounts),
                        backgroundColor: colors,
                        borderWidth: 1
                    }]
                },
                options: { 
                    responsive: true, 
                    maintainAspectRatio: false, 
                    plugins: { 
                        legend: { 
                            position: 'right',
                            labels: { boxWidth: 12, font: { size: 11 } }
                        } 
                    } 
                }
            });
        }

        function drawChartSectorPie() {
            const id = 'chartSectorPie';
            destroyChart(id);
            
            let rawCounts = countBy(filteredData, KEYS.SECTOR);
            
            const ctx = document.getElementById(id).getContext('2d');
            chartInstances[id] = new Chart(ctx, {
                type: 'pie',
                data: {
                    labels: Object.keys(rawCounts),
                    datasets: [{
                        data: Object.values(rawCounts),
                        backgroundColor: colors,
                        borderWidth: 1
                    }]
                },
                options: { 
                    responsive: true, 
                    maintainAspectRatio: false, 
                    plugins: { 
                        legend: { 
                            position: 'right',
                            labels: { boxWidth: 12, font: { size: 11 } }
                        } 
                    } 
                }
            });
        }

        function drawChartYear() {
            const id = 'chartYear';
            destroyChart(id);
            const counts = countBy(filteredData, '_CleanYearBE');
            
            // ล็อกแสดงผลเฉพาะปี 2560 ถึง 2570 เท่านั้น
            const targetYears = [];
            for (let y = 2560; y <= 2570; y++) {
                targetYears.push(String(y));
            }
            
            const sortedData = targetYears.map(y => counts[y] || 0);

            const ctx = document.getElementById(id).getContext('2d');
            chartInstances[id] = new Chart(ctx, {
                type: 'bar',
                data: {
                    labels: targetYears,
                    datasets: [{
                        label: 'จำนวนคำขอที่ยื่น',
                        data: sortedData,
                        backgroundColor: '#4f46e5',
                        borderRadius: 4
                    }]
                },
                options: {
                    responsive: true, maintainAspectRatio: false,
                    plugins: { legend: { display: false } },
                    scales: { y: { beginAtZero: true, ticks: { precision: 0 } } }
                }
            });
        }

        // --- Table Rendering ---
        function calculateDaysRemaining(dateString) {
            if (!dateString || dateString === '-' || dateString === 'N/A') return { text: '-', isWarning: false };
            const thaiMonths = {
                'ม.ค.': 0, 'ก.พ.': 1, 'มี.ค.': 2, 'เม.ย.': 3, 'พ.ค.': 4, 'มิ.ย.': 5,
                'ก.ค.': 6, 'ส.ค.': 7, 'ก.ย.': 8, 'ต.ค.': 9, 'พ.ย.': 10, 'ธ.ค.': 11
            };
            let parts = String(dateString).trim().split(/\s+/);
            if (parts.length >= 3) {
                let d = parseInt(parts[0], 10);
                let m = thaiMonths[parts[1]];
                let y = parseInt(parts[2], 10);
                if (!isNaN(d) && m !== undefined && !isNaN(y)) {
                    // แปลงให้เป็น ค.ศ. สำหรับคำนวณใน Date()
                    let fullYearAD = y < 100 ? (y < 50 ? 2000 + y : 2500 + y - 543) : (y > 2400 ? y - 543 : y);
                    let expiryDate = new Date(fullYearAD, m, d);
                    let today = new Date();
                    today.setHours(0,0,0,0);
                    expiryDate.setHours(0,0,0,0);
                    let diffTime = expiryDate.getTime() - today.getTime();
                    let diffDays = Math.ceil(diffTime / (1000 * 60 * 60 * 24));
                    if (diffDays < 0) return { text: 'หมดอายุแล้ว', isWarning: true };
                    return { text: `${diffDays} วัน`, isWarning: diffDays <= 7 };
                }
            }
            return { text: '-', isWarning: false };
        }

        function renderTable() {
            const tbody = document.getElementById('tableBody');
            document.getElementById('tableRecordCount').textContent = `${filteredData.length.toLocaleString()} รายการ`;
            
            tbody.innerHTML = '';
            
            const displayData = filteredData.slice(0, 100);
            
            if(displayData.length === 0) {
                // อัปเดต colspan เป็น 10 ให้ครอบคลุมคอลัมน์ใหม่
                tbody.innerHTML = `<tr><td colspan="10" class="px-4 py-8 text-center text-gray-500">ไม่พบข้อมูลที่ตรงกับเงื่อนไขการค้นหา</td></tr>`;
                return;
            }

            displayData.forEach(row => {
                const tr = document.createElement('tr');
                tr.className = "hover:bg-gray-50 border-b border-gray-100 last:border-0";
                
                let rawStatus = row[KEYS.STATUS] || '-';
                let statusGroup = getMappedStatus(rawStatus);
                let statusClass = "bg-gray-100 text-gray-800";
                
                if(statusGroup === 'Granted') statusClass = "bg-green-100 text-green-800 border border-green-200";
                else if(statusGroup === 'Pending') statusClass = "bg-yellow-100 text-yellow-800 border border-yellow-200";
                else if(statusGroup === 'Expired') statusClass = "bg-red-100 text-red-800 border border-red-200";

                let appRegNo = row[KEYS.APP_NO] || '-';
                if (row[KEYS.REG_NO]) appRegNo += `<br><span class="text-xs text-gray-400">Reg: ${row[KEYS.REG_NO]}</span>`;

                // คำนวณวันหมดอายุและใส่สไตล์หากน้อยกว่า 7 วัน
                let expireInfo = calculateDaysRemaining(row[KEYS.EXPIRE_DATE]);
                let expireHtml = expireInfo.isWarning ? `<span class="text-red-600 font-bold">${expireInfo.text}</span>` : expireInfo.text;

                tr.innerHTML = `
                    <td class="px-4 py-3 whitespace-nowrap font-medium text-gray-900">${row[KEYS.ID] || '-'}</td>
                    <td class="px-4 py-3 whitespace-nowrap text-sm">${row[KEYS.TYPE] || '-'}</td>
                    <td class="px-4 py-3 whitespace-nowrap text-sm">${appRegNo}</td>
                    <td class="px-4 py-3 whitespace-nowrap">
                        <span class="px-2.5 py-1 text-xs font-semibold rounded-full ${statusClass}">${rawStatus}</span>
                    </td>
                    <td class="px-4 py-3 text-sm font-medium text-indigo-700 max-w-md truncate" title="${row[KEYS.TITLE] || ''}">${row[KEYS.TITLE] || '-'}</td>
                    <td class="px-4 py-3 whitespace-nowrap text-sm">${row[KEYS.FACULTY] || '-'}</td>
                    <td class="px-4 py-3 whitespace-nowrap text-sm">${row[KEYS.INVENTOR] || '-'}</td>
                    <td class="px-4 py-3 whitespace-nowrap text-sm">${row[KEYS.YEAR] || '-'}</td>
                    <td class="px-4 py-3 whitespace-nowrap text-sm">${row[KEYS.EXPIRE_DATE] || '-'}</td>
                    <td class="px-4 py-3 whitespace-nowrap text-sm">${expireHtml}</td>
                `;
                tbody.appendChild(tr);
            });

            if(filteredData.length > 100) {
                const tr = document.createElement('tr');
                tr.innerHTML = `<td colspan="10" class="px-4 py-4 text-center text-xs text-gray-500 bg-gray-50">แสดงข้อมูล 100 รายการแรกจากทั้งหมด ${filteredData.length} รายการ (ใช้ Filter ด้านบนเพื่อจำกัดวงข้อมูล)</td>`;
                tbody.appendChild(tr);
            }
        }
    </script>
</body>
</html>
