<!DOCTYPE html>
<html lang="zh-TW">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>呆滯料轉售平台</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <script src="https://cdn.jsdelivr.net/npm/xlsx/dist/xlsx.full.min.js"></script>
</head>
<body class="bg-gray-100 p-4">
  <div class="max-w-4xl mx-auto">
    <h1 class="text-2xl font-bold mb-4 text-center">📦 呆滯料轉售平台</h1>

    <!-- 新增物料表單 -->
    <form id="itemForm" class="grid md:grid-cols-4 gap-2 mb-4">
      <input id="name" type="text" placeholder="名稱" required class="p-2 border rounded">
      <input id="quantity" type="number" placeholder="數量" required class="p-2 border rounded">
      <input id="unit" type="text" placeholder="單位 (個/箱...)" class="p-2 border rounded">
      <input id="image" type="file" accept="image/*" class="p-2 border rounded">
      <button type="submit" class="md:col-span-4 bg-blue-500 text-white p-2 rounded hover:bg-blue-600">新增物料</button>
    </form>

    <!-- Excel 匯入 -->
    <div class="flex items-center gap-2 mb-4">
      <input type="file" id="excelUpload" accept=".xlsx, .xls" class="p-2 border rounded w-full">
    </div>

    <!-- 搜尋 & 篩選 & 排序 -->
    <div class="flex flex-col md:flex-row gap-2 mb-4">
      <input id="search" type="text" placeholder="🔍 搜尋物料..." class="p-2 border rounded flex-1">
      <select id="filterUnit" class="p-2 border rounded">
        <option value="">全部單位</option>
      </select>
      <select id="sortBy" class="p-2 border rounded">
        <option value="">排序方式</option>
        <option value="name">名稱 A→Z</option>
        <option value="quantity">數量 (小→大)</option>
      </select>
    </div>

    <!-- 清單 -->
    <div id="itemList" class="grid md:grid-cols-3 gap-4"></div>
  </div>

  <script>
    let items = JSON.parse(localStorage.getItem("items")) || [];

    const itemForm = document.getElementById("itemForm");
    const itemList = document.getElementById("itemList");
    const searchInput = document.getElementById("search");
    const filterUnit = document.getElementById("filterUnit");
    const sortBy = document.getElementById("sortBy");
    const excelUpload = document.getElementById("excelUpload");

    // 儲存
    function saveItems() {
      localStorage.setItem("items", JSON.stringify(items));
    }

    // 渲染
    function renderItems() {
      itemList.innerHTML = "";

      let filtered = items.filter(item =>
        item.name.toLowerCase().includes(searchInput.value.toLowerCase()) &&
        (filterUnit.value === "" || item.unit === filterUnit.value)
      );

      // 排序
      if (sortBy.value === "name") {
        filtered.sort((a, b) => a.name.localeCompare(b.name));
      } else if (sortBy.value === "quantity") {
        filtered.sort((a, b) => a.quantity - b.quantity);
      }

      filtered.forEach((item, index) => {
        const card = document.createElement("div");
        card.className = `p-3 rounded shadow bg-white flex flex-col ${
          item.quantity <= 50 ? "border-2 border-red-500 bg-red-50" : ""
        }`;

        card.innerHTML = `
          <img src="${item.image || 'https://via.placeholder.com/150'}" class="h-32 w-full object-cover rounded mb-2">
          <h2 class="font-bold">${item.name}</h2>
          <p>數量: ${item.quantity} ${item.unit}</p>
          <button class="mt-2 bg-red-500 text-white p-1 rounded hover:bg-red-600">刪除</button>
        `;

        // 刪除
        card.querySelector("button").addEventListener("click", () => {
          items.splice(index, 1);
          saveItems();
          renderItems();
          updateFilterOptions();
        });

        itemList.appendChild(card);
      });

      updateFilterOptions();
    }

    // 更新單位選項
    function updateFilterOptions() {
      const units = [...new Set(items.map(i => i.unit).filter(u => u))];
      filterUnit.innerHTML = `<option value="">全部單位</option>`;
      units.forEach(unit => {
        const option = document.createElement("option");
        option.value = unit;
        option.textContent = unit;
        if (unit === filterUnit.value) option.selected = true;
        filterUnit.appendChild(option);
      });
    }

    // 新增物料
    itemForm.addEventListener("submit", (e) => {
      e.preventDefault();
      const reader = new FileReader();
      const file = document.getElementById("image").files[0];

      reader.onload = function(event) {
        const newItem = {
          name: document.getElementById("name").value,
          quantity: parseInt(document.getElementById("quantity").value),
          unit: document.getElementById("unit").value,
          image: file ? event.target.result : ""
        };
        items.push(newItem);
        saveItems();
        renderItems();
        itemForm.reset();
      };

      if (file) reader.readAsDataURL(file);
      else reader.onload({ target: { result: "" } });
    });

    // 搜尋 & 篩選 & 排序
    searchInput.addEventListener("input", renderItems);
    filterUnit.addEventListener("change", renderItems);
    sortBy.addEventListener("change", renderItems);

    // Excel 匯入
    excelUpload.addEventListener("change", handleFile, false);

    function handleFile(e) {
      const file = e.target.files[0];
      if (!file) return;

      const reader = new FileReader();
      reader.onload = function(event) {
        const data = new Uint8Array(event.target.result);
        const workbook = XLSX.read(data, { type: "array" });
        const sheet = workbook.Sheets[workbook.SheetNames[0]];
        const rows = XLSX.utils.sheet_to_json(sheet);

        rows.forEach(row => {
          const newItem = {
            name: row["名稱"] || "未命名",
            quantity: row["數量"] || 0,
            unit: row["單位"] || "",
            image: row["圖片網址"] || ""
          };
          items.push(newItem);
        });

        saveItems();
        renderItems();
      };
      reader.readAsArrayBuffer(file);
    }

    // 初始化
    renderItems();
  </script>
</body>
</html>
