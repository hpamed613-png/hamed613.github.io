<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>سارانتي برو - النسخة الاحترافية الشاملة</title>
    
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <script src="https://unpkg.com/html5-qrcode"></script>

    <style>
        @import url('https://fonts.googleapis.com/css2?family=Tajawal:wght@400;500;700;900&display=swap');
        :root { --main-bg: #f8fafc; --royal-blue: #4f46e5; }
        body { font-family: 'Tajawal', sans-serif; background: var(--main-bg); color: #1e293b; margin: 0; padding: 0; overflow-x: hidden; }
        .royal-card { background: white; border-radius: 20px; box-shadow: 0 4px 15px rgba(0,0,0,0.05); border: 1px solid #e2e8f0; transition: 0.3s; }
        .nav-link { color: #94a3b8; padding: 10px; border-radius: 15px; transition: 0.3s; display: flex; flex-direction: column; align-items: center; flex: 1; }
        .nav-link.active { color: var(--royal-blue); background: #eef2ff; font-weight: bold; }
        
        /* تنسيق الورق */
        .invoice-paper { background: white; color: black; padding: 20px; width: 100%; max-width: 400px; margin: auto; border: 1px solid #ddd; box-shadow: 0 0 20px rgba(0,0,0,0.1); transition: all 0.3s; }
        .thermal-mode { max-width: 80mm; font-size: 12px; }
        .normal-mode { max-width: 800px; font-size: 16px; }

        .suggestions-box { position: absolute; top: 100%; left: 0; right: 0; background: white; z-index: 1000; border-radius: 12px; box-shadow: 0 10px 25px rgba(0,0,0,0.1); border: 1px solid #e2e8f0; max-height: 200px; overflow-y: auto; }
        .suggestion-item { padding: 12px; border-bottom: 1px solid #f1f5f9; cursor: pointer; font-weight: bold; }
        .suggestion-item:hover { background: #f8fafc; color: var(--royal-blue); }
        #reader { width: 100%; border-radius: 20px; overflow: hidden; display: none; margin-bottom: 15px; border: 2px solid #4f46e5; }
        @media print { .no-print { display: none !important; } body { background: white; } .invoice-paper { box-shadow: none; border: none; } }
    </style>
</head>
<body class="pb-24">

    <header class="bg-white/80 backdrop-blur-lg sticky top-0 z-50 border-b border-slate-200 px-5 py-4 no-print">
        <div class="max-w-4xl mx-auto flex justify-between items-center">
            <div class="flex items-center gap-3">
                <div class="w-10 h-10 bg-indigo-600 rounded-xl flex items-center justify-center text-white shadow-lg font-black italic">S</div>
                <h1 id="brand-name" class="text-lg font-black text-slate-800">سارانتي برو</h1>
            </div>
            <div class="flex gap-1">
                <button onclick="shareSystemData()" class="bg-blue-50 text-blue-600 px-2 py-2 rounded-lg text-[10px] font-bold border border-blue-100">مشاركة</button>
                <button onclick="downloadBackup()" class="bg-indigo-50 text-indigo-600 px-2 py-2 rounded-lg text-[10px] font-bold border border-indigo-100">حفظ ملف</button>
                <button onclick="copySystemData()" class="bg-emerald-50 text-emerald-600 px-2 py-2 rounded-lg text-[10px] font-bold border border-emerald-100">نسخ</button>
                <button onclick="importSystemData()" class="bg-amber-50 text-amber-600 px-2 py-2 rounded-lg text-[10px] font-bold border border-amber-100">استيراد</button>
                <button onclick="changeSettings()" class="bg-indigo-50 text-indigo-600 p-2 rounded-lg">⚙️</button>
            </div>
        </div>
    </header>

    <main class="max-w-xl mx-auto p-4 space-y-6">
        
        <div id="tab-pos" class="tab-content space-y-4">
            <div id="reader"></div>

            <div class="space-y-2">
                <div class="royal-card p-3 relative">
                    <input type="text" id="cust-name" placeholder="اسم الزبون..." class="w-full p-2 rounded-xl border-none outline-none font-bold text-lg" onkeyup="showCustomerSuggestions(this.value)" autocomplete="off">
                    <div id="cust-sug-box" class="suggestions-box hidden"></div>
                </div>
                <div class="flex gap-2">
                    <input type="tel" id="cust-phone" placeholder="رقم الهاتف (واتساب)..." class="royal-card flex-1 p-3 outline-none font-bold text-sm">
                    <button onclick="startScanner()" class="bg-indigo-600 text-white px-5 rounded-2xl shadow-lg">📸</button>
                </div>
            </div>

            <div id="cart-list" class="space-y-3"></div>

            <div class="grid grid-cols-2 gap-3">
                <button onclick="addCartRow()" class="p-4 royal-card font-black text-indigo-600 border-2 border-dashed border-indigo-200 bg-indigo-50/30">+ إضافة صنف</button>
                <div class="p-4 royal-card flex items-center justify-center font-bold text-slate-500">
                    <select id="unit" class="bg-transparent outline-none cursor-pointer">
                        <option>العدد</option><option>المتر</option><option>الكيلو</option>
                    </select>
                </div>
            </div>

            <div class="bg-slate-900 rounded-[2.5rem] p-8 text-white shadow-2xl relative overflow-hidden">
                <div class="flex justify-between items-end relative z-10">
                    <div>
                        <p class="text-[10px] font-bold text-indigo-300 mb-2 tracking-widest uppercase">المدفوع الآن</p>
                        <input type="number" id="paid-amount" class="bg-transparent text-4xl font-black text-emerald-400 outline-none w-32 border-none" placeholder="0.00" oninput="calculateGrandTotal()">
                    </div>
                    <div class="text-left">
                        <p class="text-[10px] font-bold text-indigo-300 mb-2 tracking-widest uppercase">إجمالي القائمة</p>
                        <span id="total-display" class="text-5xl font-black">0.00</span>
                    </div>
                </div>
            </div>

            <button onclick="prepareInvoicePreview()" class="w-full bg-indigo-600 text-white p-5 rounded-[1.5rem] font-black text-xl shadow-xl">معاينة وحفظ ⚡</button>
            <button onclick="quickPayment()" class="w-full bg-emerald-50 text-emerald-700 p-4 rounded-xl font-bold border border-emerald-100 shadow-sm">تسجيل وصل قبض (سداد دين) 💸</button>
        </div>

        <div id="tab-archive" class="tab-content hidden space-y-4">
            <div class="royal-card p-4 flex gap-2">
                <input type="text" id="search-arch" placeholder="بحث باسم الزبون..." class="flex-1 outline-none font-bold text-sm bg-transparent" oninput="renderArchive()">
                <span class="text-slate-400">🔍</span>
            </div>
            <div id="archive-list" class="space-y-3"></div>
        </div>

        <div id="tab-debts" class="tab-content hidden space-y-4">
            <div class="flex justify-between items-center px-2">
                <h2 class="font-black text-slate-800">سجل الديون والعملاء</h2>
                <button onclick="shareDebtsBackup()" class="text-[10px] bg-rose-50 text-rose-600 p-2 rounded-lg font-bold border border-rose-100">مشاركة نسخة الديون</button>
            </div>
            <div id="debts-list" class="space-y-3"></div>
        </div>

        <div id="tab-expenses" class="tab-content hidden space-y-4">
            <div class="royal-card p-6 space-y-4 border-t-4 border-rose-500">
                <h3 class="font-black text-slate-800">إضافة مصروف جديد</h3>
                <input type="text" id="exp-title" placeholder="بيان المصروف" class="w-full p-4 rounded-xl bg-slate-50 border outline-none font-bold">
                <input type="number" id="exp-val" placeholder="المبلغ" class="w-full p-4 rounded-xl bg-slate-50 border outline-none font-bold">
                <input type="date" id="exp-date" class="w-full p-4 rounded-xl bg-slate-50 border outline-none font-bold">
                <button onclick="saveExpense()" class="w-full bg-rose-500 text-white p-4 rounded-xl font-black shadow-lg">تأكيد الصرف 🧾</button>
            </div>
            <div id="expenses-list" class="space-y-2"></div>
        </div>

        <div id="tab-stock" class="tab-content hidden space-y-6">
            <div class="grid grid-cols-2 gap-3">
                <div class="royal-card p-5 bg-indigo-50 text-indigo-700">
                    <p class="text-[10px] font-black uppercase mb-1">المبيعات (كاش)</p>
                    <p id="stat-sales" class="text-2xl font-black">0.00</p>
                </div>
                <div class="royal-card p-5 bg-rose-50 text-rose-700">
                    <p class="text-[10px] font-black uppercase mb-1">المصروفات</p>
                    <p id="stat-expenses" class="text-2xl font-black">0.00</p>
                </div>
                <div class="royal-card p-5 bg-emerald-50 text-emerald-700">
                    <p class="text-[10px] font-black uppercase mb-1">قيمة المخزن</p>
                    <p id="stat-stock" class="text-2xl font-black">0.00</p>
                </div>
                <div class="royal-card p-5 bg-red-50 text-red-700">
                    <p class="text-[10px] font-black uppercase mb-1">إجمالي الديون</p>
                    <p id="stat-debts" class="text-2xl font-black">0.00</p>
                </div>
            </div>

            <div class="royal-card p-6 space-y-4">
                <h3 class="font-black text-slate-800">إدارة البضاعة</h3>
                <input type="text" id="st-barcode" placeholder="الرقم التسلسلي (اختياري)" class="w-full p-4 rounded-xl bg-slate-50 border outline-none font-bold">
                <input type="text" id="st-name" placeholder="اسم الصنف" class="w-full p-4 rounded-xl bg-slate-50 border outline-none font-bold">
                <div class="grid grid-cols-2 gap-2">
                    <input type="number" id="st-price" placeholder="سعر البيع" class="p-4 rounded-xl bg-slate-50 border outline-none font-bold">
                    <input type="number" id="st-qty" placeholder="الكمية" class="p-4 rounded-xl bg-slate-50 border outline-none font-bold">
                </div>
                <button onclick="addToStock()" class="w-full bg-slate-900 text-white p-4 rounded-xl font-black">تحديث المخزون 📦</button>
            </div>
            <div id="stock-list" class="space-y-2"></div>
        </div>

        <div class="text-center pb-4 no-print">
            <p style="color: red; font-size: 10px; font-weight: bold;">تم التصميم والتطوير بواسطة حامد الجهمي</p>
        </div>

    </main>

    <nav class="fixed bottom-0 left-0 right-0 bg-white/90 backdrop-blur-xl border-t p-3 flex justify-around shadow-2xl no-print">
        <button onclick="switchTab('pos')" id="nav-pos" class="nav-link active"><span>💰</span><span class="text-[10px]">البيع</span></button>
        <button onclick="switchTab('archive')" id="nav-archive" class="nav-link"><span>📜</span><span class="text-[10px]">الأرشيف</span></button>
        <button onclick="switchTab('debts')" id="nav-debts" class="nav-link"><span>🔴</span><span class="text-[10px]">الديون</span></button>
        <button onclick="switchTab('expenses')" id="nav-expenses" class="nav-link"><span>💸</span><span class="text-[10px]">المصروفات</span></button>
        <button onclick="switchTab('stock')" id="nav-stock" class="nav-link"><span>📊</span><span class="text-[10px]">الجرد</span></button>
    </nav>

    <div id="invoice-modal" class="fixed inset-0 bg-black/90 hidden z-[100] p-4 flex flex-col items-center overflow-auto no-print">
        <div class="w-full max-w-md flex justify-between mb-4">
            <div class="flex gap-2">
                <button onclick="togglePaperSize('thermal')" class="bg-indigo-600 text-white px-4 py-2 rounded-lg text-xs font-bold">حراري</button>
                <button onclick="togglePaperSize('normal')" class="bg-slate-600 text-white px-4 py-2 rounded-lg text-xs font-bold">عادي</button>
            </div>
            <button onclick="closeInvoice()" class="w-10 h-10 bg-white rounded-full font-bold">✕</button>
        </div>
        
        <div id="printable-area" class="invoice-paper rounded-lg thermal-mode">
            <div class="text-center border-b-2 border-slate-900 pb-4 mb-4">
                <h2 id="print-brand" class="text-xl font-black uppercase">اسم المحل</h2>
                <p id="print-date" class="text-[10px] text-slate-500"></p>
            </div>
            
            <div class="space-y-1 mb-4 text-[11px] font-bold">
                <div class="flex justify-between"><span>الزبون:</span> <span id="print-cust"></span></div>
                <div class="flex justify-between"><span>رقم الهاتف:</span> <span id="print-phone"></span></div>
                <div class="flex justify-between"><span>رقم الفاتورة:</span> <span id="print-inv-id"></span></div>
            </div>

            <table class="w-full text-right text-[11px] mb-4">
                <thead class="border-b border-slate-300">
                    <tr><th class="py-2">الصنف</th><th class="text-center">ق</th><th class="text-center">سعر</th><th class="text-left">إجمالي</th></tr>
                </thead>
                <tbody id="print-rows"></tbody>
            </table>

            <div class="border-t-2 border-slate-900 pt-3 space-y-1">
                <div class="flex justify-between font-bold text-xs"><span>إجمالي الفاتورة:</span> <span id="print-total">0.00</span></div>
                <div class="flex justify-between text-[10px] text-emerald-600"><span>المدفوع نقداً:</span> <span id="print-paid">0.00</span></div>
                <div class="flex justify-between text-[10px] text-rose-600 border-b pb-2"><span>دين الفاتورة الحالية:</span> <span id="print-debt">0.00</span></div>
                <div class="mt-4 p-3 bg-slate-100 rounded-lg text-center border-2 border-slate-900">
                    <p class="text-[9px] font-black">إجمالي ديون الزبون</p>
                    <p id="print-all-debts" class="text-xl font-black">0.00 د.ل</p>
                </div>
            </div>
        </div>

        <div class="mt-6 w-full max-w-md space-y-3">
            <button id="confirm-save-btn" onclick="confirmFinalSale()" class="w-full bg-indigo-600 text-white p-4 rounded-xl font-black shadow-lg">حفظ وتأكيد العملية ✅</button>
            <div class="grid grid-cols-2 gap-2">
                <button onclick="window.print()" class="bg-emerald-600 text-white p-4 rounded-xl font-bold text-sm">طباعة 🖨️</button>
                <button onclick="sendWhatsApp()" class="bg-green-600 text-white p-4 rounded-xl font-bold text-sm">واتساب 📱</button>
            </div>
            <button onclick="downloadAsImage()" class="w-full bg-rose-600 text-white p-4 rounded-xl font-bold text-sm">تنزيل كصورة 🖼️</button>
        </div>
    </div>

    <div id="hidden-backup" style="position: absolute; left: -9999px;"></div>

    <script>
        let stock = JSON.parse(localStorage.getItem('s_stock')) || [];
        let invoices = JSON.parse(localStorage.getItem('s_invoices')) || [];
        let expenses = JSON.parse(localStorage.getItem('s_expenses')) || [];
        let settings = JSON.parse(localStorage.getItem('s_settings')) || { brand: "سارانتي برو" };
        let customers = JSON.parse(localStorage.getItem('s_customers')) || {};

        window.onload = () => {
            document.getElementById('brand-name').innerText = settings.brand;
            addCartRow();
            updateStats();
        };

        // --- نظام الباركود ---
        let html5QrCode;
        function startScanner() {
            document.getElementById('reader').style.display = 'block';
            html5QrCode = new Html5Qrcode("reader");
            html5QrCode.start({ facingMode: "environment" }, { fps: 10, qrbox: 250 }, 
                (decodedText) => {
                    const item = stock.find(s => s.barcode === decodedText);
                    if(item) {
                        addNewItemFromScanner(item);
                        html5QrCode.stop();
                        document.getElementById('reader').style.display = 'none';
                    } else {
                        alert("الصنف غير مسجل: " + decodedText);
                    }
                }
            ).catch(err => alert("خطأ في تشغيل الكاميرا"));
        }

        function addNewItemFromScanner(item) {
            const id = Date.now() + Math.random();
            const div = document.createElement('div');
            div.className = "royal-card p-4 flex gap-3 items-center relative";
            div.id = `row-${id}`;
            div.innerHTML = `
                <div class="flex-1 space-y-2">
                    <input type="text" value="${item.name}" class="w-full font-black outline-none itm-name">
                    <div class="flex gap-2">
                        <input type="number" value="1" class="w-1/3 border-b p-1 font-bold itm-qty" oninput="calculateGrandTotal()">
                        <input type="number" value="${item.price}" class="w-2/3 border-b p-1 font-bold itm-price" oninput="calculateGrandTotal()">
                    </div>
                </div>
                <button onclick="removeRow('${id}')" class="text-rose-300 p-2 font-bold">✕</button>
            `;
            document.getElementById('cart-list').appendChild(div);
            calculateGrandTotal();
        }

        function addCartRow() {
            const id = Date.now() + Math.random();
            const div = document.createElement('div');
            div.className = "royal-card p-4 flex gap-3 items-center relative";
            div.id = `row-${id}`;
            div.innerHTML = `
                <div class="flex-1 space-y-2">
                    <input type="text" placeholder="اسم الصنف..." class="w-full font-black outline-none itm-name" onkeyup="searchProduct(this, '${id}')">
                    <div id="sug-${id}" class="suggestions-box hidden"></div>
                    <div class="flex gap-2">
                        <input type="number" placeholder="ق" class="w-1/3 border-b p-1 font-bold itm-qty" oninput="calculateGrandTotal()">
                        <input type="number" placeholder="السعر" class="w-2/3 border-b p-1 font-bold itm-price" oninput="calculateGrandTotal()">
                    </div>
                </div>
                <button onclick="removeRow('${id}')" class="text-rose-300 p-2 font-bold">✕</button>
            `;
            document.getElementById('cart-list').appendChild(div);
        }

        function togglePaperSize(mode) {
            const el = document.getElementById('printable-area');
            if(mode === 'thermal') {
                el.classList.add('thermal-mode');
                el.classList.remove('normal-mode');
            } else {
                el.classList.remove('thermal-mode');
                el.classList.add('normal-mode');
            }
        }

        function removeRow(id) { document.getElementById(`row-${id}`).remove(); calculateGrandTotal(); }

        function searchProduct(el, rowId) {
            const val = el.value.toLowerCase();
            const box = document.getElementById('sug-' + rowId);
            if(!val) { box.classList.add('hidden'); return; }
            const matches = stock.filter(i => i.name.toLowerCase().includes(val) || (i.barcode && i.barcode.includes(val)));
            box.innerHTML = matches.map(m => `
                <div class="suggestion-item" onclick="selectProduct('${rowId}', '${m.name}', ${m.price})">
                    ${m.name} <span class="text-[10px] opacity-50">(${m.qty})</span>
                </div>`).join('');
            box.classList.remove('hidden');
        }

        function selectProduct(rowId, name, price) {
            const row = document.getElementById('row-' + rowId);
            row.querySelector('.itm-name').value = name;
            row.querySelector('.itm-price').value = price;
            row.querySelector('.itm-qty').focus();
            document.getElementById('sug-' + rowId).classList.add('hidden');
            calculateGrandTotal();
        }

        function calculateGrandTotal() {
            let sum = 0;
            document.querySelectorAll('#cart-list > div').forEach(row => {
                const q = parseFloat(row.querySelector('.itm-qty').value) || 0;
                const p = parseFloat(row.querySelector('.itm-price').value) || 0;
                sum += (q * p);
            });
            document.getElementById('total-display').innerText = sum.toFixed(2);
        }

        function prepareInvoicePreview(existing = null) {
            let cust, phone, total, paid, date, id, items;
            if(existing) {
                ({cust, phone, total, paid, date, id, items} = existing);
                document.getElementById('confirm-save-btn').classList.add('hidden');
            } else {
                cust = document.getElementById('cust-name').value || "زبون نقدي";
                phone = document.getElementById('cust-phone').value || "";
                total = parseFloat(document.getElementById('total-display').innerText);
                paid = parseFloat(document.getElementById('paid-amount').value) || 0;
                date = new Date().toLocaleString('ar-LY');
                id = "INV-" + Date.now().toString().slice(-6);
                items = [];
                document.querySelectorAll('#cart-list > div').forEach(r => {
                    const n = r.querySelector('.itm-name').value;
                    const q = parseFloat(r.querySelector('.itm-qty').value) || 0;
                    const p = parseFloat(r.querySelector('.itm-price').value) || 0;
                    if(n) items.push({n, q, p});
                });
                if(items.length === 0) return alert("الفاتورة فارغة!");
                document.getElementById('confirm-save-btn').classList.remove('hidden');
                window.lastInvoice = {id, cust, phone, total, paid, date, items};
            }

            document.getElementById('print-brand').innerText = settings.brand;
            document.getElementById('print-cust').innerText = cust;
            document.getElementById('print-phone').innerText = phone || "غير مسجل";
            document.getElementById('print-date').innerText = date;
            document.getElementById('print-inv-id').innerText = id;
            document.getElementById('print-total').innerText = total.toFixed(2);
            document.getElementById('print-paid').innerText = paid.toFixed(2);
            document.getElementById('print-debt').innerText = (total - paid).toFixed(2);

            const allDebts = invoices.filter(i => i.cust === cust).reduce((s, i) => s + (i.total - i.paid), 0);
            document.getElementById('print-all-debts').innerText = (existing ? allDebts : (allDebts + (total - paid))).toFixed(2);

            document.getElementById('print-rows').innerHTML = items.map(it => `
                <tr class="border-b"><td>${it.n}</td><td class="text-center">${it.q}</td><td class="text-center">${it.p}</td><td class="text-left font-bold">${(it.q*it.p).toFixed(2)}</td></tr>
            `).join('');
            document.getElementById('invoice-modal').classList.remove('hidden');
        }

        function confirmFinalSale() {
            const inv = window.lastInvoice;
            invoices.push(inv);
            
            // تحديث بيانات العميل (الهاتف)
            if(inv.phone) {
                customers[inv.cust] = inv.phone;
                localStorage.setItem('s_customers', JSON.stringify(customers));
            }

            inv.items.forEach(it => {
                const product = stock.find(s => s.name === it.n);
                if(product) product.qty -= it.q;
            });
            localStorage.setItem('s_invoices', JSON.stringify(invoices));
            localStorage.setItem('s_stock', JSON.stringify(stock));
            alert("تم الحفظ ✅"); location.reload();
        }

        function updateStats() {
            const sales = invoices.reduce((s, i) => s + i.paid, 0);
            const exps = expenses.reduce((s, e) => s + e.val, 0);
            const debts = invoices.reduce((s, i) => s + (i.total - i.paid), 0);
            const stockVal = stock.reduce((s, i) => s + (i.price * i.qty), 0);

            document.getElementById('stat-sales').innerText = sales.toFixed(2);
            document.getElementById('stat-expenses').innerText = exps.toFixed(2);
            document.getElementById('stat-stock').innerText = stockVal.toFixed(2);
            document.getElementById('stat-debts').innerText = debts.toFixed(2);
        }

        function addToStock() {
            const barcode = document.getElementById('st-barcode').value;
            const name = document.getElementById('st-name').value;
            const price = parseFloat(document.getElementById('st-price').value);
            const qty = parseFloat(document.getElementById('st-qty').value);
            if(!name || isNaN(price)) return alert("أدخل البيانات");
            const existing = stock.find(s => s.name === name || (barcode && s.barcode === barcode));
            if(existing) {
                existing.barcode = barcode; existing.price = price; existing.qty = qty;
            } else {
                stock.push({barcode, name, price, qty});
            }
            localStorage.setItem('s_stock', JSON.stringify(stock));
            renderStock(); updateStats(); alert("تم التحديث");
        }

        function renderStock() {
            document.getElementById('stock-list').innerHTML = stock.map((s, idx) => `
                <div class="royal-card p-4 flex justify-between items-center">
                    <div><p class="font-black">${s.name}</p><p class="text-xs opacity-50">S/N: ${s.barcode || '---'} | سعر: ${s.price} | كمية: ${s.qty}</p></div>
                    <button onclick="stock.splice(${idx},1); localStorage.setItem('s_stock', JSON.stringify(stock)); renderStock();" class="text-rose-400">✕</button>
                </div>`).join('');
        }

        function switchTab(t) {
            document.querySelectorAll('.tab-content').forEach(el => el.classList.add('hidden'));
            document.getElementById('tab-' + t).classList.remove('hidden');
            document.querySelectorAll('.nav-link').forEach(l => l.classList.remove('active'));
            document.getElementById('nav-' + t).classList.add('active');
            if(t === 'stock') renderStock();
            if(t === 'archive') renderArchive();
            if(t === 'debts') renderDebts();
        }

        function renderArchive() {
            const query = document.getElementById('search-arch').value.toLowerCase();
            const filtered = invoices.filter(i => i.cust.toLowerCase().includes(query));
            document.getElementById('archive-list').innerHTML = filtered.slice().reverse().map(i => `
                <div class="royal-card p-4 flex justify-between items-center cursor-pointer" onclick='prepareInvoicePreview(${JSON.stringify(i)})'>
                    <div><p class="font-black">${i.cust}</p><p class="text-[10px] opacity-50">${i.date}</p></div>
                    <div class="text-left"><p class="font-black text-indigo-600">${i.total.toFixed(2)}</p>
                    <button onclick="event.stopPropagation(); deleteInvoice('${i.id}')" class="text-xs text-rose-400">حذف</button></div>
                </div>`).join('');
        }

        // --- نظام الديون الجديد ---
        function renderDebts() {
            const debtMap = {};
            invoices.forEach(inv => {
                if(!debtMap[inv.cust]) debtMap[inv.cust] = { totalDebt: 0, phone: inv.phone || customers[inv.cust] || "" };
                debtMap[inv.cust].totalDebt += (inv.total - inv.paid);
            });

            document.getElementById('debts-list').innerHTML = Object.keys(debtMap).map(name => {
                const d = debtMap[name];
                if(Math.abs(d.totalDebt) < 0.01) return '';
                return `
                <div class="royal-card p-4 flex justify-between items-center border-r-4 ${d.totalDebt > 0 ? 'border-rose-500' : 'border-emerald-500'}">
                    <div>
                        <p class="font-black">${name}</p>
                        <p class="text-xs text-indigo-600 font-bold">${d.phone || 'بدون هاتف'}</p>
                    </div>
                    <div class="text-left">
                        <p class="font-black ${d.totalDebt > 0 ? 'text-rose-600' : 'text-emerald-600'}">${d.totalDebt.toFixed(2)}</p>
                        ${d.phone ? `<button onclick="sendWhatsAppDirect('${d.phone}', ${d.totalDebt})" class="text-[10px] bg-green-50 text-green-600 px-2 py-1 rounded">مطالبة 📱</button>` : ''}
                    </div>
                </div>`;
            }).join('');
        }

        function deleteInvoice(id) { if(confirm("حذف؟")) { invoices = invoices.filter(i => i.id !== id); localStorage.setItem('s_invoices', JSON.stringify(invoices)); renderArchive(); updateStats(); } }

        // --- الميزات الجديدة للمشاركة والنسخ ---
        function copySystemData() {
            const data = btoa(unescape(encodeURIComponent(JSON.stringify({stock, invoices, expenses, settings, customers}))));
            navigator.clipboard.writeText(data); alert("تم نسخ رمز البيانات");
        }

        async function shareSystemData() {
            const data = JSON.stringify({stock, invoices, expenses, settings, customers});
            if (navigator.share) {
                try {
                    await navigator.share({ title: 'نسخة سارانتي برو', text: data });
                } catch (err) { copySystemData(); }
            } else { copySystemData(); }
        }

        function downloadBackup() {
            const data = JSON.stringify({stock, invoices, expenses, settings, customers});
            const blob = new Blob([data], {type: 'application/json'});
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = `backup_${new Date().toLocaleDateString()}.json`;
            a.click();
        }

        function shareDebtsBackup() {
            const debtData = invoices.map(i => ({ زبون: i.cust, هاتف: i.phone || customers[i.cust], مبلغ: (i.total - i.paid) }));
            const dataStr = "بيانات الديون:\n" + debtData.filter(d => d.مبلغ != 0).map(d => `${d.زبون} (${d.هاتف}): ${d.مبلغ}`).join('\n');
            if (navigator.share) {
                navigator.share({ title: 'كشف ديون', text: dataStr });
            } else {
                navigator.clipboard.writeText(dataStr); alert("تم نسخ النص");
            }
        }

        function importSystemData() {
            const code = prompt("الصق الرمز أو محتوى ملف النسخة:");
            if(code) {
                try {
                    let d;
                    if(code.startsWith('{')) d = JSON.parse(code);
                    else d = JSON.parse(decodeURIComponent(escape(atob(code))));
                    
                    localStorage.setItem('s_stock', JSON.stringify(d.stock || []));
                    localStorage.setItem('s_invoices', JSON.stringify(d.invoices || []));
                    localStorage.setItem('s_expenses', JSON.stringify(d.expenses || []));
                    localStorage.setItem('s_settings', JSON.stringify(d.settings || {brand:"سارانتي برو"}));
                    localStorage.setItem('s_customers', JSON.stringify(d.customers || {}));
                    location.reload();
                } catch(e) { alert("خطأ في قراءة البيانات"); }
            }
        }

        function sendWhatsApp() {
            const inv = window.lastInvoice;
            if(!inv.phone) return alert("يرجى إدخال رقم الهاتف أولاً");
            const text = `فاتورة من: ${settings.brand}\nالزبون: ${inv.cust}\nالقيمة: ${inv.total}\nالمدفوع: ${inv.paid}\nالمتبقي: ${inv.total - inv.paid}`;
            window.open(`https://wa.me/${inv.phone}?text=${encodeURIComponent(text)}`);
        }

        function sendWhatsAppDirect(phone, amount) {
            const text = `تحية طيبة، يرجى العلم بأن الرصيد المستحق لديكم لمحلات ${settings.brand} هو: ${amount.toFixed(2)} د.ل`;
            window.open(`https://wa.me/${phone}?text=${encodeURIComponent(text)}`);
        }

        function quickPayment() {
            const name = document.getElementById('cust-name').value;
            const phone = document.getElementById('cust-phone').value;
            if(!name) return alert("أدخل اسم الزبون");
            const amt = prompt("المبلغ المستلم:");
            if(!amt || isNaN(amt)) return;
            invoices.push({id:"PAY-"+Date.now().toString().slice(-5), cust:name, phone:phone, total:0, paid:parseFloat(amt), date:new Date().toLocaleString('ar-LY'), items:[]});
            localStorage.setItem('s_invoices', JSON.stringify(invoices)); alert("تم التسجيل"); updateStats();
        }

        function saveExpense() {
            const title = document.getElementById('exp-title').value;
            const val = parseFloat(document.getElementById('exp-val').value);
            if(!title || isNaN(val)) return;
            expenses.push({title, val, date: document.getElementById('exp-date').value});
            localStorage.setItem('s_expenses', JSON.stringify(expenses)); location.reload();
        }

        function renderExpenses() {
            document.getElementById('expenses-list').innerHTML = expenses.map(e => `<div class="royal-card p-4 flex justify-between bg-rose-50"><span>${e.title}</span><span class="font-black">${e.val}</span></div>`).join('');
        }

        async function downloadAsImage() {
            const area = document.getElementById('printable-area');
            const canvas = await html2canvas(area);
            const link = document.createElement('a');
            link.download = 'invoice.png';
            link.href = canvas.toDataURL();
            link.click();
        }

        function closeInvoice() { document.getElementById('invoice-modal').classList.add('hidden'); }
        function changeSettings() { const n = prompt("الاسم الجديد:", settings.brand); if(n) { settings.brand = n; localStorage.setItem('s_settings', JSON.stringify(settings)); location.reload(); } }

        function showCustomerSuggestions(q) {
            const box = document.getElementById('cust-sug-box');
            if(!q) { box.classList.add('hidden'); return; }
            const names = [...new Set(invoices.map(i => i.cust))].filter(n => n.includes(q));
            if(names.length) {
                box.innerHTML = names.map(n => `<div class="suggestion-item" onclick="selectCust('${n}')">${n}</div>`).join('');
                box.classList.remove('hidden');
            } else box.classList.add('hidden');
        }
        function selectCust(n) { 
            document.getElementById('cust-name').value = n; 
            document.getElementById('cust-phone').value = customers[n] || "";
            document.getElementById('cust-sug-box').classList.add('hidden'); 
        }
    </script>
</body>
</html>
