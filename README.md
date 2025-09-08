# 7
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8" />
  <title>حاسبة الأسعار المتقدمة</title>
  <style>
    body { font-family: 'Segoe UI', sans-serif; background-color: #1e1e1e; color: white; direction: rtl; padding: 30px; text-align: right; }
    select, input, textarea { padding: 8px; margin: 5px 0; width: 100%; border: none; border-radius: 4px; box-sizing: border-box; background-color: #333; color: white; }
    textarea { resize: vertical; min-height: 120px; font-family: monospace; font-size: 1.1em; }
    .section { background-color: #252526; padding: 15px; border-radius: 8px; margin-bottom: 20px; border: 1px solid #333; }
    label { font-weight: bold; color: #00e676; }
    button { background-color: #007acc; color: white; padding: 12px 22px; border: none; margin-top: 10px; border-radius: 5px; cursor: pointer; font-size: 16px; transition: background-color 0.3s; }
    button:hover { background-color: #005a9e; }
    .btn-secondary { background-color: #555; }
    .btn-secondary:hover { background-color: #777; }
    .btn-danger { background-color: #c0392b; }
    .btn-danger:hover { background-color: #a52f22; }
    .btn-success { background-color: #27ae60; }
    .btn-success:hover { background-color: #229954; }
    .results { background: #2b2b2b; padding: 20px; margin-top: 20px; border-radius: 8px; }
    .result-item { background-color: #252526; border: 1px solid #333; border-radius: 8px; padding: 15px; margin-bottom: 15px; }
    .summary { font-weight: bold; color: #00e676; padding-top: 15px; border-top: 2px solid #00e676; margin-top: 15px; }
    .item-header { display: flex; justify-content: space-between; align-items: center; }
    .item-header h4 { margin: 0; font-size: 1.2em; }
    .edit-form-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(150px, 1fr)); gap: 15px; margin-top: 10px; }
    #addonsHost div { margin-bottom: 10px; }
    details { border: 1px solid #555; border-radius: 4px; padding: 10px; }
    summary { font-weight: bold; cursor: pointer; color: #00e676; }
    .reference-table { width: 100%; margin-top: 10px; border-collapse: collapse; }
    .reference-table th, .reference-table td { border: 1px solid #555; padding: 8px; text-align: right; }
    .reference-table th { background-color: #333; }
  </style>
</head>
<body>

<h1 style="margin: 0; text-align: center; margin-bottom: 20px;">حاسبة الأسعار</h1>

<div class="section">
    <label for="customerName">اسم العميل:</label>
    <input type="text" id="customerName" placeholder="مثال: جون دو" />
    <label for="customerPhone">رقم هاتف العميل:</label>
    <input type="text" id="customerPhone" placeholder="مثال: 99455082" />
</div>

<div class="section">
    <label for="bulkAddText">إضافة جماعية من نص</label>
    <p style="font-size: 0.9em; color: #ccc; margin-top: 0;">الصق الأصناف هنا بالتنسيق: <strong>الرقم - الكود - الارتفاع - العرض - الكمية</strong> (كل صنف في سطر)</p>
    <textarea id="bulkAddText" placeholder="مثال: B1-2-2.1-0.9-1&#10;D5-D4-2.2-1.0-5"></textarea>
    <button onclick="processPastedText()" style="width: 100%;" class="btn-success">إضافة الأصناف من النص</button>
</div>

<div class="section" id="referenceContainer">
    </div>

<div class="section">
  <p style="text-align: center; font-weight: bold; font-size: 1.2em; margin: 0;">-- أو أضف يدوياً --</p>
</div>

<div class="section">
  <label for="mainCategory">الفئة الرئيسية:</label>
  <select id="mainCategory" onchange="loadSubTypes()"></select>
</div>

<div class="section">
  <label for="subType">النوع الفرعي:</label>
  <select id="subType" onchange="renderAddons()"></select>
</div>

<div class="section">
  <label for="height">الارتفاع (متر):</label>
  <input type="number" id="height" step="0.01" value="1" />
  <label for="width">العرض (متر):</label>
  <input type="number" id="width" step="0.01" value="1" />
  <label for="quantity">الكمية:</label>
  <input type="number" id="quantity" value="1" />
</div>

<div class="section" id="addonsHost"></div>

<div class="section">
    <label for="installationCost">تكلفة التركيب (ر.ع):</label>
    <input type="number" id="installationCost" value="0" step="0.01" onchange="renderResults()" />
</div>


<button onclick="addItem()">➕ إضافة صنف يدويًا</button>
<button onclick="clearAllResults()" class="btn-danger">🗑️ مسح كل النتائج</button>
<button onclick="saveAsWord()" class="btn-secondary">💾 حفظ كملف Word</button>

<div class="results" id="results"></div>

<script>
const SHIPPING_RATE = 48;
const addonPrices = { 
    curtain: 26, 
    net: { door: 39, folding: 18, sliding: 14 } 
};

const productData = {
    "Windows": {
        "Window Double Glass Double Frame Fixed": { price: 34, cbm: 0.13, method: 'per_meter', addons: 'curtain,net' },
        "Window Double Glass Double Frame 1-Way": { price: 34, fixed_component_cost: 39, cbm: 0.13, method: 'per_meter', addons: 'curtain,net' },
        "Window Double Glass Double Frame 2-Way": { price: 34, fixed_component_cost: 58, cbm: 0.13, method: 'per_meter', addons: 'curtain,net' },
        "Window Double Glass Single Frame Fixed": { price: 26, cbm: 0.07, method: 'per_meter', addons: 'curtain,net' },
        "Window Double Glass Single Frame 1-Way": { price: 26, fixed_component_cost: 20, cbm: 0.07, method: 'per_meter', addons: 'curtain,net' },
        "Window Double Glass Single Frame 2-Way": { price: 26, fixed_component_cost: 32, cbm: 0.07, method: 'per_meter', addons: 'curtain,net' },
        "Window Single Glass Single Frame Fixed": { price: 20, cbm: 0.07, method: 'per_meter', addons: 'net' },
        "Window Single Glass Single Frame 1-Way": { price: 20, fixed_component_cost: 13, cbm: 0.07, method: 'per_meter', addons: 'net' },
        "Window Single Glass Single Frame 2-Way": { price: 20, fixed_component_cost: 17, cbm: 0.07, method: 'per_meter', addons: 'net' },
        "Sliding Windows": { price: 41, fixed_component_cost: 10, cbm: 0.13, method: 'per_meter', addons: 'curtain' },
        "Electric Windows": { price: 102, cbm: 0.13, method: 'per_meter' },
        "Skylight without Motor": { price: 56, cbm: 0.13, method: 'per_meter' },
        "Skylight with Motor": { price: 145, cbm: 0.13, method: 'per_meter' },
        "Heavy Curtain Wall": { price: 56, cbm: 0.15, method: 'per_meter' },
        "Light Curtain Wall": { price: 45, cbm: 0.15, method: 'per_meter' },
    },
    "Doors": {
        "Entrance Door - Zinc": { price: 66, cbm: 0.20, method: 'per_meter', special: 'add_10' },
        "Entrance Door - Stainless Steel": { price: 120, cbm: 0.20, method: 'per_meter', special: 'add_10' },
        "Entrance Door - Cast Aluminum": { price: 168, cbm: 0.20, method: 'per_meter', special: 'add_10' },
        "WPC Door": { price: 45, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 1.0 },
        "WPC Door - with Wood": { price: 50, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 1.0 },
        "WPC Door - with Soundproof Filling": { price: 60, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 1.0 },
        "WPC Door - with Aluminum Frame": { price: 67, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 1.0 },
        "Aluminum Door": { price: 65, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 1.0 },
        "Aluminum Door - with Wood": { price: 75, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 1.0 },
        "Aluminum Door - Full": { price: 85, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 1.0 },
        "Aluminum Door - Hidden": { price: 110, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 1.0 },
        "Aluminum Door - Exterior": { price: 61, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 1.0 },
        "Bathroom Door - New Type": { price: 55, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 0.8 },
        "Bathroom Door - Old Type": { price: 45, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 0.8 },
        "Bathroom Door - Hidden Glass": { price: 65, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 0.8 },
    },
    "Sliding Doors": {
        "Interior Sliding Door - Glass": { price: 38, cbm: 0.15, method: 'per_meter', addons: 'curtain' },
        "Interior Sliding Door - Solid": { price: 41, cbm: 0.15, method: 'per_meter' },
        "Exterior Sliding Door - 1 Panel Open": { price: 55, cbm: 0.15, method: 'per_meter' },
        "Exterior Sliding Door - 2 Panels Open": { price: 58, cbm: 0.15, method: 'per_meter' },
        "WPC Sliding Door": { price: 61, cbm: 0.15, method: 'per_meter' },
    },
    "Folding Doors": {
        "Interior Folding Door": { price: 39, cbm: 0.15, method: 'per_meter' },
        "Exterior Folding Door": { price: 56, cbm: 0.15, method: 'per_meter' },
    },
    "Exterior Shutters": {
        "Rolling Shutter": { price: 28, cbm: 0.20, method: 'per_meter' },
    },
    "Garden Gates": {
        "Cast Aluminum Garden Gate": { price: 91, cbm: 0.20, method: 'per_meter' },
    },
    "Barriers": {
        "Stair Barrier": { price: 43, cbm: 0.05, method: 'per_meter' },
        "Bathroom Barrier": { price: 32, cbm: 0.05, method: 'per_meter' },
    }
};

const productCodes = {
    '1': "Window Double Glass Double Frame Fixed", '2': "Window Double Glass Double Frame 1-Way", '3': "Window Double Glass Double Frame 2-Way",
    '4': "Window Double Glass Single Frame Fixed", '5': "Window Double Glass Single Frame 1-Way", '6': "Window Double Glass Single Frame 2-Way",
    '7': "Window Single Glass Single Frame Fixed", '8': "Window Single Glass Single Frame 1-Way", '9': "Window Single Glass Single Frame 2-Way",
    '10': "Sliding Windows", '11': "Electric Windows", '12': "Skylight without Motor", '13': "Skylight with Motor", '14': "Heavy Curtain Wall", '15': "Light Curtain Wall",
    'D1': "Entrance Door - Zinc", 'D2': "Entrance Door - Stainless Steel", 'D3': "Entrance Door - Cast Aluminum",
    'D4': "WPC Door", 'D5': "WPC Door - with Wood", 'D6': "WPC Door - with Soundproof Filling", 'D7': "WPC Door - with Aluminum Frame",
    'D8': "Aluminum Door", 'D9': "Aluminum Door - with Wood", 'D10': "Aluminum Door - Full", 'D11': "Aluminum Door - Hidden", 'D12': "Aluminum Door - Exterior",
    'D13': "Bathroom Door - New Type", 'D14': "Bathroom Door - Old Type", 'D15': "Bathroom Door - Hidden Glass",
    'S1': "Interior Sliding Door - Glass", 'S2': "Interior Sliding Door - Solid", 'S3': "Exterior Sliding Door - 1 Panel Open", 'S4': "Exterior Sliding Door - 2 Panels Open", 'S5': "WPC Sliding Door",
    'F1': "Interior Folding Door", 'F2': "Exterior Folding Door",
    'E1': "Rolling Shutter", 'G1': "Cast Aluminum Garden Gate",
    'B1': "Stair Barrier", 'B2': "Bathroom Barrier"
};

let resultsList = [];

function initializeApp() {
    const mainCat = document.getElementById("mainCategory");
    mainCat.innerHTML = `<option value="">-- اختر فئة --</option>`;
    Object.keys(productData).forEach(cat => mainCat.innerHTML += `<option value="${cat}">${cat}</option>`);
    loadSubTypes();
    renderReferenceGuide();
}

function loadSubTypes() {
    const mainCatVal = document.getElementById("mainCategory").value;
    const subType = document.getElementById("subType");
    subType.innerHTML = "";
    if (mainCatVal && productData[mainCatVal]) {
        Object.keys(productData[mainCatVal]).forEach(sub => subType.innerHTML += `<option value="${sub}">${sub}</option>`);
    }
    renderAddons();
}

function renderAddons() {
    const subVal = document.getElementById("subType").value;
    const addonsHost = document.getElementById("addonsHost");
    addonsHost.innerHTML = "";
    const data = findProductData(subVal);
    if (data && data.addons) {
        const availableAddons = data.addons.split(',');
        if (availableAddons.includes('curtain')) {
            addonsHost.innerHTML += `<div><label><input type="checkbox" id="addon_curtain"> إضافة ستارة (+${addonPrices.curtain} ريال عماني/م²)</label></div>`;
        }
        if (availableAddons.includes('net')) {
            addonsHost.innerHTML += `<div><label for="addon_net_type">إضافة شبك:</label><select id="addon_net_type"><option value="">-- لا شيء --</option><option value="door">باب (+${addonPrices.net.door} لكل 0.5م²)</option><option value="folding">قابل للطي (+${addonPrices.net.folding} لكل 0.5م²)</option><option value="sliding">منزلق (+${addonPrices.net.sliding} لكل 0.5م²)</option></select></div>`;
        }
    }
    setDefaultDimensions();
}

function setDefaultDimensions() {
    const subVal = document.getElementById("subType").value;
    const data = findProductData(subVal);
    if (data && data.method === 'per_unit') {
        document.getElementById("height").value = data.std_h || 2.2;
        document.getElementById("width").value = data.std_w || 1.0;
    } else {
        document.getElementById("height").value = 1;
        document.getElementById("width").value = 1;
    }
}

function findProductData(productName) {
    for (const category in productData) {
        if (productData[category][productName]) {
            return productData[category][productName];
        }
    }
    return null;
}

function findProductByCode(code) {
    const upperCode = code.toUpperCase();
    const fullName = productCodes[upperCode];
    return fullName ? { name: fullName, data: findProductData(fullName) } : null;
}

function calculateItemComponents(item) {
    const data = item.data;
    if (!data) return { unitPrice: 0, totalPrice: 0, shippingCost: 0 };
    const area = item.h * item.w;
    let basePrice = 0, sizePenalty = 0, addonCost = 0;

    if (data.method === 'per_unit') {
        basePrice = data.price || 0;
        const stdArea = (data.std_h || 0) * (data.std_w || 0);
        if (stdArea > 0 && area > stdArea) {
            sizePenalty = Math.ceil((area - stdArea) / 0.1) * 2;
        }
    } else {
        basePrice = area * (data.price || 0);
    }
    
    if (item.selectedAddons.curtain) {
        addonCost += area * addonPrices.curtain;
    }
    
    if (item.selectedAddons.netType && addonPrices.net[item.selectedAddons.netType]) {
        const netUnits = Math.ceil(area / 0.5);
        addonCost += netUnits * addonPrices.net[item.selectedAddons.netType];
    }

    const fixedCost = data.fixed_component_cost || 0;
    const specialCost = data.special === 'add_10' ? 10 : 0;
    const shippingCost = area * (data.cbm || 0) * SHIPPING_RATE;
    const unitPrice = basePrice + sizePenalty + fixedCost + specialCost + addonCost;
    
    return { 
        unitPrice: unitPrice, 
        totalPrice: unitPrice * item.qty,
        shippingCost: shippingCost * item.qty
    };
}

function addItem(manualData = null) {
    let newItem;
    if (manualData) {
        newItem = {
            id: Date.now(),
            name: manualData.name,
            qty: manualData.qty,
            h: manualData.h,
            w: manualData.w,
            itemCodePrefix: manualData.itemCodePrefix || "",
            itemNotes: manualData.itemNotes || "",
            selectedAddons: {},
            data: manualData.data,
            isEditing: false
        };
    } else {
        const subVal = document.getElementById("subType").value;
        if (!subVal) { alert("الرجاء اختيار نوع فرعي."); return; }
        
        const height = parseFloat(document.getElementById("height").value);
        const width = parseFloat(document.getElementById("width").value);
        const quantity = parseInt(document.getElementById("quantity").value);

        if (isNaN(height) || height <= 0) { alert("الرجاء إدخال ارتفاع صالح."); return; }
        if (isNaN(width) || width <= 0) { alert("الرجاء إدخال عرض صالح."); return; }
        if (isNaN(quantity) || quantity <= 0) { alert("الرجاء إدخال كمية صالحة."); return; }

        const data = findProductData(subVal);
        newItem = {
            id: Date.now(),
            name: subVal,
            qty: quantity,
            h: height,
            w: width,
            itemCodePrefix: "",
            itemNotes: "",
            selectedAddons: {
                curtain: document.getElementById("addon_curtain")?.checked || false,
                netType: document.getElementById("addon_net_type")?.value || null
            },
            data: JSON.parse(JSON.stringify(data)),
            isEditing: false
        };
    }
    
    const prices = calculateItemComponents(newItem);
    newItem.unitPrice = prices.unitPrice;
    newItem.totalPrice = prices.totalPrice;
    newItem.shippingCost = prices.shippingCost;
    resultsList.push(newItem);
    renderResults();
}

function processPastedText() {
    const text = document.getElementById("bulkAddText").value;
    const lines = text.split('\n');
    let addedCount = 0;
    let skippedLines = [];

    lines.forEach((line, index) => {
        line = line.trim();
        if (!line) return;

        const parts = line.split('-').map(p => p.trim());
        
        if (parts.length < 5) {
            skippedLines.push(`السطر ${index + 1}: "${line}" - تنسيق غير صحيح.`);
            return;
        }

        const itemCodePrefix = parts[0];
        const typeCode = parts[1];
        const h = parseFloat(parts[2]);
        const w = parseFloat(parts[3]);
        const qty = parseInt(parts[4]);

        if (isNaN(h) || h <= 0) { skippedLines.push(`السطر ${index + 1}: "${line}" - ارتفاع غير صالح.`); return; }
        if (isNaN(w) || w <= 0) { skippedLines.push(`السطر ${index + 1}: "${line}" - عرض غير صالح.`); return; }
        if (isNaN(qty) || qty <= 0) { skippedLines.push(`السطر ${index + 1}: "${line}" - كمية غير صالحة.`); return; }

        const productInfo = findProductByCode(typeCode);
        if (productInfo) {
            addItem({ name: productInfo.name, h, w, qty, data: productInfo.data, itemCodePrefix: itemCodePrefix });
            addedCount++;
        } else {
            skippedLines.push(`السطر ${index + 1}: "${line}" - كود المنتج "${typeCode}" غير معروف.`);
        }
    });

    let alertMessage = `تمت إضافة ${addedCount} صنف بنجاح.`;
    if (skippedLines.length > 0) {
        alertMessage += `\n\nتم تخطي ${skippedLines.length} أسطر بسبب أخطاء:\n- ` + skippedLines.join('\n- ');
    }
    alert(alertMessage);
    document.getElementById("bulkAddText").value = "";
}

function deleteItem(id) {
    resultsList = resultsList.filter(item => item.id !== id);
    renderResults();
}

function clearAllResults() {
    resultsList = [];
    renderResults();
}

function enterEditMode(id) {
    resultsList.forEach(item => item.isEditing = (item.id === id));
    renderResults();
}

function cancelEdit(id) {
    const item = resultsList.find(i => i.id === id);
    if (item) {
        item.isEditing = false;
        renderResults();
    }
}

function saveItemEdit(id) {
    const item = resultsList.find(i => i.id === id);
    if (item) {
        const newName = document.getElementById(`edit-name-${id}`).value;
        const newQty = parseFloat(document.getElementById(`edit-qty-${id}`).value);
        const newH = parseFloat(document.getElementById(`edit-h-${id}`).value);
        const newW = parseFloat(document.getElementById(`edit-w-${id}`).value);
        
        if (isNaN(newH) || newH <= 0) { alert("الرجاء إدخال ارتفاع صالح."); return; }
        if (isNaN(newW) || newW <= 0) { alert("الرجاء إدخال عرض صالح."); return; }
        if (isNaN(newQty) || newQty <= 0) { alert("الرجاء إدخال كمية صالحة."); return; }

        const newData = findProductData(newName);
        if (!newData) {
            alert(`اسم المنتج "${newName}" غير صالح.`);
            return;
        }
        
        item.name = newName;
        item.data = newData; 
        item.qty = newQty;
        item.h = newH;
        item.w = newW;
        item.itemNotes = document.getElementById(`edit-notes-${id}`).value;

        const prices = calculateItemComponents(item);
        item.unitPrice = prices.unitPrice;
        item.totalPrice = prices.totalPrice;
        item.shippingCost = prices.shippingCost;

        item.isEditing = false;
    }
    renderResults();
}

function renderResults() {
    const container = document.getElementById("results");
    container.innerHTML = "";
    if (resultsList.length === 0) return;

    let subtotal = 0;
    let totalShipping = 0;

    resultsList.forEach(item => {
        subtotal += item.totalPrice;
        totalShipping += item.shippingCost;

        if (item.isEditing) {
             container.innerHTML += `
                <div class="result-item">
                    <div class="edit-form-grid">
                        <div><label>الاسم</label><input type="text" id="edit-name-${item.id}" value="${item.name}"></div>
                        <div><label>الكمية</label><input type="number" id="edit-qty-${item.id}" value="${item.qty}"></div>
                        <div><label>الارتفاع</label><input type="number" step="0.01" id="edit-h-${item.id}" value="${item.h}"></div>
                        <div><label>العرض</label><input type="number" step="0.01" id="edit-w-${item.id}" value="${item.w}"></div>
                    </div>
                    <div style="margin-top: 15px;"><label>ملاحظات</label><textarea id="edit-notes-${item.id}">${item.itemNotes}</textarea></div>
                    <div style="text-align: left; margin-top: 10px;">
                        <button onclick="cancelEdit(${item.id})" class="btn-secondary" style="padding: 8px 15px;">إلغاء</button>
                        <button onclick="saveItemEdit(${item.id})" class="btn-success" style="padding: 8px 15px;">حفظ التغييرات</button>
                    </div>
                </div>`;
        } else {
            container.innerHTML += `
                <div class="result-item">
                    <div class="item-header">
                        <h4>${item.itemCodePrefix ? `[${item.itemCodePrefix}] ` : ''}${item.name}</h4>
                        <div>
                           <button onclick="enterEditMode(${item.id})" class="btn-secondary" style="padding: 6px 12px; margin: 0 5px;">تعديل ✏️</button>
                           <button onclick="deleteItem(${item.id})" class="btn-danger" style="padding: 6px 12px; margin: 0;">حذف 🗑️</button>
                        </div>
                    </div>
                    <p><strong>الأبعاد:</strong> ${item.h}م x ${item.w}م | <strong>الكمية:</strong> ${item.qty}</p>
                    <p style="font-weight: bold; font-size: 1.1em;">
                        سعر الوحدة: ${item.unitPrice.toFixed(2)} ر.ع. | الإجمالي: ${item.totalPrice.toFixed(2)} ر.ع.
                    </p>
                    ${item.itemNotes ? `<p><strong>ملاحظات:</strong> ${item.itemNotes.replace(/\n/g, '<br>')}</p>` : ''}
                </div>`;
        }
    });

    const commission = subtotal * 0.04;
    const installationCost = parseFloat(document.getElementById('installationCost').value) || 0;
    const grandTotal = subtotal + commission + totalShipping + installationCost;

    container.innerHTML += `
        <div class="summary">
            <div style="display: flex; justify-content: space-between;"><span>سعر الشراء من الصين:</span><span>${subtotal.toFixed(2)} ر.ع.</span></div>
            <div style="display: flex; justify-content: space-between;"><span>الشحن والتخليص:</span><span>${totalShipping.toFixed(2)} ر.ع.</span></div>
            <div style="display: flex; justify-content: space-between;"><span>عمولة المكتب (4%):</span><span>${commission.toFixed(2)} ر.ع.</span></div>
            <div style="display: flex; justify-content: space-between;"><span>التركيب:</span><span>${installationCost.toFixed(2)} ر.ع.</span></div>
            <div style="background-color: #00e676; color: #1e1e1e; padding: 15px; border-radius: 5px; text-align: center; margin-top: 10px;">
                <span style="font-size: 1.3em;">💰 المجموع الكلي</span>
                <div style="font-size: 2.0em; font-weight: bold;">${grandTotal.toFixed(2)} ر.ع.</div>
            </div>
        </div>`;
}

function renderReferenceGuide() {
    const container = document.getElementById('referenceContainer');
    let tableHtml = '<details><summary>اضغط هنا لرؤية أكواد المنتجات</summary><table class="reference-table"><thead><tr><th>الكود</th><th>اسم المنتج</th><th>الفئة</th></tr></thead><tbody>';
    
    const invertedCodes = {};
    for (const code in productCodes) {
        invertedCodes[productCodes[code]] = code;
    }

    for (const category in productData) {
        for (const product in productData[category]) {
            const code = invertedCodes[product] || 'N/A';
            tableHtml += `<tr><td><strong>${code}</strong></td><td>${product}</td><td>${category}</td></tr>`;
        }
    }

    tableHtml += '</tbody></table></details>';
    container.innerHTML = tableHtml;
}

// **Final saveAsWord Function - High Fidelity**
function saveAsWord() {
    if (resultsList.length === 0) { alert("لا توجد نتائج لحفظها."); return; }
    
    let subtotal = 0, totalShipping = 0, totalCBM = 0;
    resultsList.forEach(item => {
        subtotal += item.totalPrice;
        totalShipping += item.shippingCost;
        totalCBM += (item.h * item.w * (item.data.cbm || 0) * item.qty);
    });

    const commission = subtotal * 0.04;
    const installationCost = parseFloat(document.getElementById('installationCost').value) || 0;
    const grandTotal = subtotal + commission + totalShipping + installationCost;

    let tableRows = '';
    resultsList.forEach((item, index) => {
        tableRows += `
            <tr style="page-break-inside: avoid;">
                <td style="border: 1px solid black; padding: 5px; text-align: center; vertical-align: middle;">${index + 1}</td>
                <td style="border: 1px solid black; padding: 5px; text-align: right; vertical-align: middle;">${item.name}${item.itemNotes ? `<br><small style="color: #555;">${item.itemNotes.replace(/\n/g, '<br>')}</small>`: ''}</td>
                <td style="border: 1px solid black; padding: 5px; text-align: center; vertical-align: middle;">${item.itemCodePrefix || ''}</td>
                <td style="border: 1px solid black; padding: 5px; text-align: center; vertical-align: middle;">${item.qty}</td>
                <td style="border: 1px solid black; padding: 5px; text-align: center; vertical-align: middle;">${item.w}</td>
                <td style="border: 1px solid black; padding: 5px; text-align: center; vertical-align: middle;">${item.h}</td>
                <td style="border: 1px solid black; padding: 5px; text-align: center; vertical-align: middle;">${(item.h * item.w).toFixed(2)}</td>
                <td style="border: 1px solid black; padding: 5px; text-align: right; vertical-align: middle;">${item.unitPrice.toFixed(2)}</td>
                <td style="border: 1px solid black; padding: 5px; text-align: right; vertical-align: middle;">${item.totalPrice.toFixed(2)}</td>
            </tr>`;
    });

    let content = `
        <html xmlns:w="urn:schemas-microsoft-com:office:word">
        <head><meta charset='utf-8'><title>Quotation</title></head>
        <body style="font-family: Arial, sans-serif; direction: rtl; text-align: right;">
            
            <table style="width: 100%; border: none; border-bottom: 1px solid black; padding-bottom: 5px;">
                <tr>
                    <td style="text-align: right; vertical-align: top;">
                        <h3 style="margin:0;">BLUE WAVES SERVICES LLC</h3>
                        <p style="margin:0; font-size: 10pt;">OMAN-MUSCAT</p>
                        <p style="margin:0; font-size: 10pt;">TEL: 77 22 45 11 - 90 99 88 10</p>
                    </td>
                    <td style="text-align: left; vertical-align: top;">
                        <p style="margin:0; font-size: 10pt;"><strong>SR. NO.: 1595256</strong></p>
                    </td>
                </tr>
            </table>

            <br/>

            <table style="width: 100%; border-collapse: collapse; font-size: 10pt;">
                <thead>
                    <tr style="background-color: #f2f2f2;">
                        <th style="border: 1px solid black; padding: 8px; font-weight: bold; text-align: center;">NO</th>
                        <th style="border: 1px solid black; padding: 8px; font-weight: bold; text-align: right;">DESCRITION</th>
                        <th style="border: 1px solid black; padding: 8px; font-weight: bold; text-align: center;">STYLE</th>
                        <th style="border: 1px solid black; padding: 8px; font-weight: bold; text-align: center;">Q</th>
                        <th style="border: 1px solid black; padding: 8px; font-weight: bold; text-align: center;">W</th>
                        <th style="border: 1px solid black; padding: 8px; font-weight: bold; text-align: center;">H</th>
                        <th style="border: 1px solid black; padding: 8px; font-weight: bold; text-align: center;">m2</th>
                        <th style="border: 1px solid black; padding: 8px; font-weight: bold; text-align: center;">PRICE</th>
                        <th style="border: 1px solid black; padding: 8px; font-weight: bold; text-align: center;">TOTAL</th>
                    </tr>
                </thead>
                <tbody>
                    ${tableRows}
                </tbody>
            </table>

            <br/>

            <table style="width: 100%; border-collapse: collapse; font-size: 10pt;">
                 <tr>
                    <td style="border: 1px solid black; padding: 5px; font-weight: bold;">سعر الشراء من الصين</td>
                    <td style="border: 1px solid black; padding: 5px; text-align: right;">${subtotal.toFixed(2)}</td>
                    <td style="border: 1px solid black; padding: 5px; font-weight: bold;">TOTAL CBM</td>
                    <td style="border: 1px solid black; padding: 5px; text-align: right;">${totalCBM.toFixed(3)}</td>
                </tr>
                 <tr>
                    <td style="border: 1px solid black; padding: 5px; font-weight: bold;">عمولة المكتب 4 %</td>
                    <td style="border: 1px solid black; padding: 5px; text-align: right;">${commission.toFixed(2)}</td>
                    <td style="border: 1px solid black; padding: 5px;" colspan="2"></td>
                </tr>
                 <tr>
                    <td style="border: 1px solid black; padding: 5px; font-weight: bold;">الشحن والتخليص الجمركي</td>
                    <td style="border: 1px solid black; padding: 5px; text-align: right;">${totalShipping.toFixed(2)}</td>
                    <td style="border: 1px solid black; padding: 5px; font-weight: bold;">الحالي سعر الشحن</td>
                    <td style="border: 1px solid black; padding: 5px; text-align: right;">${SHIPPING_RATE}</td>
                </tr>
                <tr>
                    <td style="border: 1px solid black; padding: 5px; font-weight: bold;">التركيب</td>
                    <td style="border: 1px solid black; padding: 5px; text-align: right;">${installationCost.toFixed(2)}</td>
                    <td style="border: 1px solid black; padding: 5px;" colspan="2"></td>
                </tr>
                 <tr>
                    <td style="border: 1px solid black; padding: 5px; font-weight: bold; background-color: #f2f2f2;">الإجمالي</td>
                    <td style="border: 1px solid black; padding: 5px; text-align: right; font-weight: bold; background-color: #f2f2f2;">${grandTotal.toFixed(2)}</td>
                    <td style="border: 1px solid black; padding: 5px; background-color: #f2f2f2;" colspan="2"></td>
                </tr>
            </table>

        </body>
        </html>
    `;
    
    const blob = new Blob(['\ufeff', content], { type: 'application/msword' });
    const link = document.createElement("a");
    link.href = URL.createObjectURL(blob);
    link.download = 'Quotation.doc';
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
}

document.addEventListener('DOMContentLoaded', initializeApp);

</script>
</body>
</html>
