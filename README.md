<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8">
  <title>تتبع الدخل اليومي</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/html2pdf.js/0.10.1/html2pdf.bundle.min.js"></script>
  <style>
    body {
      font-family: 'Tahoma', sans-serif;
      background-color: #f4f4f4;
      margin: 0;
      padding: 0;
      text-align: center;
    }
    header {
      background-color: #4a90e2;
      color: white;
      padding: 10px 0;
      position: relative;
    }
    header img {
      height: 40px;
      position: absolute;
      left: 10px;
      top: 10px;
    }
    h1 {
      margin: 0;
    }
    #entries, #archiveDisplay {
      margin: 10px;
      padding: 10px;
      background-color: white;
      border-radius: 10px;
      box-shadow: 0 2px 5px rgba(0,0,0,0.1);
    }
    .entry, .archive-entry {
      background: #e8f0fe;
      margin: 5px;
      padding: 10px;
      border-radius: 8px;
      display: flex;
      justify-content: space-between;
      align-items: center;
    }
    button, .fab {
      margin: 5px;
      padding: 10px 20px;
      border: none;
      border-radius: 8px;
      font-size: 16px;
      cursor: pointer;
    }
    .fab {
      position: fixed;
      bottom: 20px;
      right: 50%;
      transform: translateX(50%);
      background-color: #28a745;
      color: white;
      font-size: 40px;
      border-radius: 50%;
      width: 80px;
      height: 80px;
      display: flex;
      justify-content: center;
      align-items: center;
      box-shadow: 0 4px 10px rgba(0,0,0,0.2);
    }
    .delete-btn {
      background-color: #dc3545;
      color: white;
    }
    .archive-day {
      cursor: pointer;
      color: #007bff;
      margin: 5px;
    }
    .archive-day:hover {
      text-decoration: underline;
    }
  </style>
</head>
<body>
  <header>
    <img src="https://www2.0zz0.com/2025/04/05/13/377189202.png" alt="شعار">
    <h1>تتبع الدخل اليومي</h1>
  </header>
  <div>
    <button onclick="exportPDF()">تصدير إلى PDF</button>
    <button onclick="toggleArchive()">عرض/إخفاء سجل الأرشيف</button>
  </div>
  <div id="entries"></div>
  <div id="archiveDisplay" style="display:none;"></div>
  <button class="fab" onclick="showAddEntryPrompt()">+</button>

  <script>
    let entries = JSON.parse(localStorage.getItem("entries")) || [];
    let archive = JSON.parse(localStorage.getItem("archive")) || {};

    function saveEntries() {
      localStorage.setItem("entries", JSON.stringify(entries));
      localStorage.setItem("archive", JSON.stringify(archive));
    }

    function getTodayDate() {
      const now = new Date();
      now.setHours(now.getHours() - 6);
      return now.toLocaleDateString("ar-EG", { weekday: 'long', year: 'numeric', month: 'numeric', day: 'numeric' });
    }

    function showAddEntryPrompt() {
      const price = prompt("أدخل السعر بالجنيه:");
      if (!price) return;
      const device = prompt("أدخل رقم الجهاز:");
      if (!device) return;
      const time = new Date().toLocaleTimeString("ar-EG");
      entries.push({ price, device, time });
      saveEntries();
      updateDisplay();
    }

    function updateDisplay() {
      const container = document.getElementById("entries");
      container.innerHTML = `<h3>${getTodayDate()}</h3>`;
      entries.forEach((entry, index) => {
        const div = document.createElement("div");
        div.className = "entry";
        div.innerHTML = `<span>${entry.price} جنيه - جهاز ${entry.device} - ${entry.time}</span>
                          <button class='delete-btn' onclick='deleteEntry(${index})'>حذف</button>`;
        container.appendChild(div);
      });
    }

    function deleteEntry(index) {
      entries.splice(index, 1);
      saveEntries();
      updateDisplay();
    }

    function archiveOldEntries() {
      const today = getTodayDate();
      const lastSavedDay = localStorage.getItem("lastDay");
      if (lastSavedDay && lastSavedDay !== today && entries.length > 0) {
        archive[lastSavedDay] = archive[lastSavedDay] || [];
        archive[lastSavedDay].push(...entries);
        entries = [];
        saveEntries();
      }
      localStorage.setItem("lastDay", today);
    }

    function toggleArchive() {
      const display = document.getElementById("archiveDisplay");
      display.style.display = display.style.display === "none" ? "block" : "none";
      if (display.style.display === "block") renderArchive();
    }

    function renderArchive() {
      const container = document.getElementById("archiveDisplay");
      container.innerHTML = "<h3>سجل الأرشيف</h3>";
      for (const day in archive) {
        const total = archive[day].reduce((sum, e) => sum + Number(e.price), 0);
        const div = document.createElement("div");
        div.className = "archive-day";
        div.textContent = `${day} - الإجمالي: ${total} جنيه`;
        div.onclick = () => showArchiveDetails(day);
        container.appendChild(div);
      }
    }

    function showArchiveDetails(day) {
      const container = document.getElementById("entries");
      container.innerHTML = `<h3>${day}</h3>`;
      archive[day].forEach((entry, index) => {
        const div = document.createElement("div");
        div.className = "entry";
        div.innerHTML = `<span>${entry.price} جنيه - جهاز ${entry.device} - ${entry.time}</span>`;
        container.appendChild(div);
      });
      const del = document.createElement("button");
      del.textContent = "حذف هذا اليوم";
      del.className = "delete-btn";
      del.onclick = () => deleteArchiveDay(day);
      container.appendChild(del);

      const back = document.createElement("button");
      back.textContent = "رجوع";
      back.onclick = () => updateDisplay();
      container.appendChild(back);
    }

    function deleteArchiveDay(day) {
      delete archive[day];
      saveEntries();
      updateDisplay();
      renderArchive();
    }

    function exportPDF() {
      const element = document.getElementById("entries");
      const opt = {
        margin:       0.5,
        filename:     `الدخل_${getTodayDate().replace(/\s+/g, "_")}.pdf`,
        image:        { type: 'jpeg', quality: 0.98 },
        html2canvas:  { scale: 2 },
        jsPDF:        { unit: 'in', format: 'letter', orientation: 'portrait' }
      };
      html2pdf().from(element).set(opt).save();
    }

    archiveOldEntries();
    updateDisplay();
  </script>
</body>
</html>
