<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>DWLR Groundwater Monitoring Dashboard</title>
  <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-gray-100 font-sans">

  <!-- Header -->
  <header class="bg-blue-700 text-white p-4 flex justify-between items-center">
    <h1 class="text-xl font-bold">DWLR Dashboard</h1>
    <button id="drawerToggle" class="bg-white text-blue-700 px-4 py-2 rounded">☰ Menu</button>
  </header>

  <!-- Drawer -->
  <aside id="drawer" class="fixed top-0 left-0 w-64 h-full bg-white shadow-lg transform -translate-x-full transition-transform duration-300">
    <div class="p-4 border-b">
      <h2 class="text-lg font-bold">Navigation</h2>
    </div>
    <nav class="p-4">
      <ul class="space-y-2">
        <li><a href="#" class="text-blue-700">Dashboard</a></li>
        <li><a href="#" class="text-blue-700">Reports</a></li>
        <li><a href="#" class="text-blue-700">Settings</a></li>
      </ul>
    </nav>
  </aside>

  <!-- Main -->
  <main class="p-6">
    <!-- Stats -->
    <div class="grid grid-cols-1 md:grid-cols-3 gap-4 mb-6">
      <div class="bg-white p-4 rounded shadow">
        <h3 class="text-gray-500">Active DWLRs</h3>
        <p id="activeDWLR" class="text-2xl font-bold">0</p>
      </div>
      <div class="bg-white p-4 rounded shadow">
        <h3 class="text-gray-500">Avg Water Level (m bgl)</h3>
        <p id="avgWaterLevel" class="text-2xl font-bold">0</p>
      </div>
      <div class="bg-white p-4 rounded shadow">
        <h3 class="text-gray-500">Alerts</h3>
        <p id="alertsCount" class="text-2xl font-bold text-red-600">0</p>
      </div>
    </div>

    <!-- Charts -->
    <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
      <div class="bg-white p-4 rounded shadow">
        <h3 class="text-gray-700 font-semibold mb-2">Water Level Trends</h3>
        <canvas id="lineChart" width="400" height="200"></canvas>
      </div>
      <div class="bg-white p-4 rounded shadow">
        <h3 class="text-gray-700 font-semibold mb-2">DWLR Locations</h3>
        <div id="map" class="w-full h-52 bg-gray-200 relative"></div>
      </div>
    </div>

    <!-- Alerts -->
    <div class="mt-6 bg-white p-4 rounded shadow">
      <h3 class="text-gray-700 font-semibold mb-2">Recent Alerts</h3>
      <ul id="alertsList" class="space-y-2"></ul>
    </div>
  </main>

  <script>
    // Dummy data
    const data = {
      dwlrs: [
        { id: 1, name: "DWLR-1", waterLevels: [5, 6, 7, 9, 11, 13], location: {x: 50, y: 80} },
        { id: 2, name: "DWLR-2", waterLevels: [4, 5, 5, 6, 7, 8], location: {x: 150, y: 120} },
        { id: 3, name: "DWLR-3", waterLevels: [7, 8, 9, 11, 13, 15], location: {x: 250, y: 160} },
      ],
      safeLimit: 10
    };

    // Stats
    document.getElementById('activeDWLR').innerText = data.dwlrs.length;
    const avg = data.dwlrs.flatMap(d => d.waterLevels).reduce((a,b)=>a+b,0) / 
                data.dwlrs.flatMap(d => d.waterLevels).length;
    document.getElementById('avgWaterLevel').innerText = avg.toFixed(2);

    // Alerts
    const alerts = [];
    data.dwlrs.forEach(d => {
      d.waterLevels.forEach((wl, i) => {
        if (wl > data.safeLimit) {
          alerts.push(`${d.name} exceeded safe limit at reading ${i+1} (Level: ${wl} m bgl)`);
        }
      });
    });
    document.getElementById('alertsCount').innerText = alerts.length;
    const alertsList = document.getElementById('alertsList');
    alerts.forEach(a => {
      const li = document.createElement('li');
      li.textContent = a;
      li.className = "text-red-600";
      alertsList.appendChild(li);
    });

    // Line Chart
    const lineCanvas = document.getElementById('lineChart');
    const ctx = lineCanvas.getContext('2d');
    const colors = ['#3b82f6','#10b981','#ef4444'];

    function drawChart() {
      ctx.clearRect(0,0,lineCanvas.width,lineCanvas.height);
      ctx.strokeStyle = "#ddd"; ctx.beginPath();
      ctx.moveTo(40,10); ctx.lineTo(40,180); ctx.lineTo(380,180); ctx.stroke();

      data.dwlrs.forEach((d,idx)=>{
        ctx.beginPath();
        ctx.strokeStyle = colors[idx];
        d.waterLevels.forEach((wl,i)=>{
          const x = 40 + i*60;
          const y = 180 - wl*10;
          if(i===0) ctx.moveTo(x,y); else ctx.lineTo(x,y);
        });
        ctx.stroke();
      });
    }
    drawChart();

    // Hover highlight
    let chartHoverX = null;
    lineCanvas.addEventListener('mousemove',(e)=>{
      const rect = lineCanvas.getBoundingClientRect();
      chartHoverX = e.clientX - rect.left;
      drawChart();
      if(chartHoverX){
        ctx.strokeStyle = "#999";
        ctx.beginPath();
        ctx.moveTo(chartHoverX,10);
        ctx.lineTo(chartHoverX,180);
        ctx.stroke();
      }
    });
    lineCanvas.addEventListener('mouseleave',()=>{ chartHoverX=null; drawChart(); });

    // Map pins
    const map = document.getElementById('map');
    data.dwlrs.forEach(d=>{
      const pin = document.createElement('div');
      pin.className="absolute bg-red-600 w-3 h-3 rounded-full cursor-pointer";
      pin.style.left = d.location.x+"px";
      pin.style.top = d.location.y+"px";
      pin.title = d.name;
      map.appendChild(pin);
    });

    // Drawer toggle
    const drawer = document.getElementById('drawer');
    document.getElementById('drawerToggle').addEventListener('click',()=>{
      drawer.classList.toggle("-translate-x-full");
    });
  </script>
</body>
</html>
