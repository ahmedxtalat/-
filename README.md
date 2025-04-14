<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8">
  <title>تتبع الدخل اليومي</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
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
      padding: 10px;
      display: flex;
      flex-wrap: wrap;
      align-items: center;
      justify-content: space-between;
    }
    .logo-container {
      display: flex;
      align-items: center;
      gap: 10px;
    }
    header img {
      height: 40px;
    }
    .header-info {
      text-align: right;
      font-size: 14px;
    }
    h1 {
      font-size: 20px;
      margin: 5px 0;
      flex: 1 1 100%;
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
      margin: 5px 0;
      padding: 10px;
      border-radius: 8px;
      display: flex;
      justify-content: space-between;
      align-items: center;
      flex-wrap: wrap;
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
    <div class="logo-container">
      <img src="https://www2.0zz0.com/2025/04/05/13/377189202.png" alt="شعار">
      <div class="header-info">
        <div id="todayDate"></div>
        <div id="entryCount"></div>
        <div id="totalEarnings"></div>
      </div>
    </div>
    <h1>تتبع الدخل اليومي</h1>
  </header>

  <div>
    <button onclick="window.open('export-today.html', '_blank')">تصدير اليوم إلى PDF</button>
    <button onclick="window.open('export-archive.html', '_blank')">تصدير الأرشيف إلى PDF</button>
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
      return now.toLocaleDateString("ar-EG", {
        weekday: 'long', year: 'numeric', month: 'numeric', day: 'numeric'
      });
    }

    function updateHeaderInfo() {
      document.getElementById("todayDate").textContent = getTodayDate();
      document.getElementById("entryCount").textContent = `عدد العمليات: ${entries.length}`;
      const total = entries.reduce((sum, e) => sum + Number(e.price), 0);
      document.getElementById("totalEarnings").textContent = `إجمالي الأرباح: ${total} جنيه`;
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
      updateHeaderInfo();
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
      archive[day].forEach((entry) => {
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

    archiveOldEntries();
    updateDisplay();
  </script>
</body>
</html>
