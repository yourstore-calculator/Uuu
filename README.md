# Uuu
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8" />
  <title>حاسبة الأسعار</title>
  <style>
    body {
      font-family: 'Segoe UI', sans-serif;
      background: #f5f5f5;
      color: #333;
      direction: rtl;
      padding: 20px;
    }
    .container {
      max-width: 900px;
      margin: auto;
      background: #fff;
      padding: 20px;
      box-shadow: 0 0 15px rgba(0,0,0,0.1);
      border-radius: 10px;
    }
    h1 {
      text-align: center;
      color: #111;
    }
    label {
      display: block;
      margin: 10px 0 5px;
      font-weight: bold;
    }
    select, input {
      width: 100%;
      padding: 8px;
      margin-bottom: 15px;
      border-radius: 5px;
      border: 1px solid #ccc;
    }
    button {
      background: #111;
      color: #fff;
      padding: 10px 20px;
      border: none;
      margin-left: 10px;
      border-radius: 5px;
      cursor: pointer;
    }
    button:hover {
      background: #333;
    }
    #results {
      margin-top: 30px;
    }
    .result-box {
      background: #f9f9f9;
      border: 1px solid #ddd;
      padding: 15px;
      margin-bottom: 10px;
      border-radius: 6px;
    }
    img.logo {
      display: block;
      margin: auto;
      max-width: 180px;
      margin-bottom: 20px;
    }
  </style>
</head>
<body>
  <div class="container">
    <img class="logo" src="https://i.imgur.com/qBRVhUe.png" alt="Logo" />
    <h1>حاسبة الأسعار</h1>

    <label for="category">اختر القسم:</label>
    <select id="category">
      <option value="">-- اختر القسم --</option>
      <option value="نوافذ">نوافذ</option>
      <option value="أبواب">أبواب</option>
      <option value="أبواب السحب">أبواب السحب</option>
    </select>

    <label for="type">اختر النوع:</label>
    <select id="type"></select>

    <label>الطول (متر):</label>
    <input type="number" id="length" placeholder="مثلاً: 2.4">

    <label>العرض (متر):</label>
    <input type="number" id="width" placeholder="مثلاً: 1.2">

    <label>الكمية:</label>
    <input type="number" id="quantity" value="1">

    <button onclick="calculate()">احسب</button>
    <button onclick="clearResults()">حذف النتائج</button>
    <button onclick="exportToWord()">حفظ كـ Word</button>

    <div id="results"></div>
  </div>

  <script>
    const prices = {
      "دبل جلاس دبل فريم": { price: 73, factor: 0.13, perPiece: false },
      "دبل جلاس سنجل فريم": { price: 46, factor: 0.13, perPiece: false },
      "سنجل جلاس سنجل فريم": { price: 43, factor: 0.13, perPiece: false },
      "نوافذ السلايدنج": { price: 10, factor: 0.13, perPiece: false, isSliding: true },
      "النوافذ الكهربائية": { price: 102, factor: 0.13, perPiece: false },
      "سكاي لايت - بدون مكينة": { price: 56, factor: 0.13, perPiece: false },
      "سكاي لايت - مع مكينة": { price: 145, factor: 0.13, perPiece: false },
      "كارتن وول - ثقيل": { price: 56, factor: 0.13, perPiece: false },
      "كارتن وول - خفيف": { price: 45, factor: 0.13, perPiece: false },

      "باب مدخل - زينك": { price: 66+10, factor: 0.2, perPiece: false },
      "باب مدخل - ستينلس": { price: 120+10, factor: 0.2, perPiece: false },
      "باب مدخل - كاست ألمنيوم": { price: 168+10, factor: 0.2, perPiece: false },

      "باب WPC فارغ": { price: 45, factor: 0.11, perPiece: true },
      "باب WPC مع خشب": { price: 50, factor: 0.11, perPiece: true },
      "باب WPC ضد الصوت": { price: 60, factor: 0.11, perPiece: true },
      "باب WPC فريم المنيوم": { price: 67, factor: 0.11, perPiece: true },

      "باب ألمنيوم فارغ": { price: 65, factor: 0.11, perPiece: true },
      "باب ألمنيوم مع خشب": { price: 75, factor: 0.11, perPiece: true },
      "باب ألمنيوم فل": { price: 85, factor: 0.11, perPiece: true },
      "باب ألمنيوم مخفي": { price: 110, factor: 0.11, perPiece: true },
      "باب ألمنيوم خارجي": { price: 61, factor: 0.11, perPiece: true },

      "باب دورة مياه جديد": { price: 55, factor: 0.11, perPiece: true },
      "باب دورة مياه قديم": { price: 45, factor: 0.11, perPiece: true },
      "باب دورة مياه زجاجي مخفي": { price: 65, factor: 0.11, perPiece: true },

      "سحب داخلي زجاج": { price: 38, factor: 0.13, perPiece: false },
      "سحب داخلي متين": { price: 41, factor: 0.13, perPiece: false },
      "سحب خارجي جزء مفتوح": { price: 55, factor: 0.13, perPiece: false },
      "سحب خارجي جزئين مفتوحين": { price: 58, factor: 0.13, perPiece: false },
      "سحب WPC": { price: 61, factor: 0.11, perPiece: false }
    };

    const categories = {
      "نوافذ": [
        "دبل جلاس دبل فريم", "دبل جلاس سنجل فريم", "سنجل جلاس سنجل فريم",
        "نوافذ السلايدنج", "النوافذ الكهربائية",
        "سكاي لايت - بدون مكينة", "سكاي لايت - مع مكينة",
        "كارتن وول - ثقيل", "كارتن وول - خفيف"
      ],
      "أبواب": [
        "باب مدخل - زينك", "باب مدخل - ستينلس", "باب مدخل - كاست ألمنيوم",
        "باب WPC فارغ", "باب WPC مع خشب", "باب WPC ضد الصوت", "باب WPC فريم المنيوم",
        "باب ألمنيوم فارغ", "باب ألمنيوم مع خشب", "باب ألمنيوم فل",
        "باب ألمنيوم مخفي", "باب ألمنيوم خارجي",
        "باب دورة مياه جديد", "باب دورة مياه قديم", "باب دورة مياه زجاجي مخفي"
      ],
      "أبواب السحب": [
        "سحب داخلي زجاج", "سحب داخلي متين", "سحب خارجي جزء مفتوح",
        "سحب خارجي جزئين مفتوحين", "سحب WPC"
      ]
    };

    document.getElementById("category").addEventListener("change", function () {
      const selected = this.value;
      const typeSelect = document.getElementById("type");
      typeSelect.innerHTML = "";
      if (categories[selected]) {
        categories[selected].forEach(type => {
          const opt = document.createElement("option");
          opt.value = type;
          opt.textContent = type;
          typeSelect.appendChild(opt);
        });
      }
    });

    function calculate() {
      const type = document.getElementById("type").value;
      const length = parseFloat(document.getElementById("length").value);
      const width = parseFloat(document.getElementById("width").value);
      const quantity = parseInt(document.getElementById("quantity").value);

      if (!type || isNaN(length) || isNaN(width) || isNaN(quantity)) {
        alert("يرجى إدخال جميع البيانات بشكل صحيح.");
        return;
      }

      const { price, factor, perPiece, isSliding } = prices[type] || {};
      const area = length * width;
      let basePrice = perPiece ? price : (area * price);
      if (isSliding) basePrice += 10;

      const shipping = area * factor * 48;
      const total = (basePrice + shipping) * quantity;

      const resultBox = document.createElement("div");
      resultBox.className = "result-box";
      resultBox.innerHTML = `
        <strong>النوع:</strong> ${type}<br>
        <strong>المقاس:</strong> ${length} × ${width} = ${area.toFixed(2)} م²<br>
        <strong>الكمية:</strong> ${quantity}<br>
        <strong>سعر الشحن:</strong> ${shipping.toFixed(2)} ريال<br>
        <strong>السعر النهائي:</strong> ${total.toFixed(2)} ريال
      `;
      document.getElementById("results").appendChild(resultBox);
    }

    function clearResults() {
      document.getElementById("results").innerHTML = "";
    }

    function exportToWord() {
      const header = "<html xmlns:o='urn:schemas-microsoft-com:office:office' " +
        "xmlns:w='urn:schemas-microsoft-com:office:word' " +
        "xmlns='http://www.w3.org/TR/REC-html40'>" +
        "<head><meta charset='utf-8'></head><body dir='rtl'>";
      const footer = "</body></html>";
      const content = document.getElementById("results").innerHTML;
      const sourceHTML = header + content + footer;

      const source = 'data:application/vnd.ms-word;charset=utf-8,' + encodeURIComponent(sourceHTML);
      const fileDownload = document.createElement("a");
      document.body.appendChild(fileDownload);
      fileDownload.href = source;
      fileDownload.download = 'النتائج.doc';
      fileDownload.click();
      document.body.removeChild(fileDownload);
    }
  </script>
</body>
</html>
