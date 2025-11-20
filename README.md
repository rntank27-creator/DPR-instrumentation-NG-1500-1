<!-- index.html - DPR submission page with Firebase, PDF generation, offline sync indicator, multi-user support -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Daily Progress Report (DPR)</title>
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@picocss/pico@1/css/pico.min.css">
  <style>
    /* (styles omitted for brevity in README view) */
  </style>

  <!-- Firebase SDKs -->
  <script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.1/firebase-app.js";
    import { getFirestore, collection, addDoc, query, where, orderBy, getDocs } from "https://www.gstatic.com/firebasejs/10.12.1/firebase-firestore.js";
    import { getStorage, ref, uploadBytes, getDownloadURL } from "https://www.gstatic.com/firebasejs/10.12.1/firebase-storage.js";

    // ---------- REPLACE WITH YOUR FIREBASE CONFIG ----------
    const firebaseConfig = {
      apiKey: "YOUR_API_KEY",
      authDomain: "YOUR_PROJECT.firebaseapp.com",
      projectId: "YOUR_PROJECT",
      storageBucket: "YOUR_PROJECT.appspot.com",
      messagingSenderId: "SENDER_ID",
      appId: "APP_ID"
    };
    // ------------------------------------------------------

    const app = initializeApp(firebaseConfig);
    const db = getFirestore(app);
    const storage = getStorage(app);

    // Expose for debug (optional)
    window.db = db;
    window.storage = storage;

    // Small helper: check online/offline
    function updateNetworkIndicator() {
      const el = document.getElementById('networkStatus');
      if (!el) return;
      if (navigator.onLine) {
        el.textContent = 'Online • Cloud sync enabled';
        el.classList.remove('offline');
      } else {
        el.textContent = 'Offline • Saving locally';
        el.classList.add('offline');
      }
    }

    window.addEventListener('online', updateNetworkIndicator);
    window.addEventListener('offline', updateNetworkIndicator);

    document.addEventListener('DOMContentLoaded', () => {
      updateNetworkIndicator();

      const form = document.getElementById('dprForm');
      const statusMsg = document.getElementById('statusMsg');

      async function uploadFiles(files) {
        const urls = [];
        for (const f of files) {
          const path = `dpr_uploads/${Date.now()}_${f.name}`;
          const storageRef = ref(storage, path);
          await uploadBytes(storageRef, f);
          const url = await getDownloadURL(storageRef);
          urls.push(url);
        }
        return urls;
      }

      // Save a report to Firestore (or queue in localStorage if offline)
      async function saveReport(report) {
        if (navigator.onLine) {
          // online: upload any files then save
          if (report._files && report._files.length) {
            report.imageURLs = await uploadFiles(report._files);
            delete report._files;
          }
          await addDoc(collection(db, 'DPR_Reports'), report);
          return { saved: true };
        } else {
          // offline: save to local queue
          const queue = JSON.parse(localStorage.getItem('dpr_queue') || '[]');
          queue.push(report);
          localStorage.setItem('dpr_queue', JSON.stringify(queue));
          return { saved: false };
        }
      }

      // Try to flush local queue when online
      async function flushQueue() {
        const queue = JSON.parse(localStorage.getItem('dpr_queue') || '[]');
        if (!queue.length) return;
        for (const item of queue) {
          try {
            // if item has files stored as base64, skip file upload here (advanced). Simpler approach: offline file upload not supported in this basic flow.
            await addDoc(collection(db, 'DPR_Reports'), item);
          } catch (err) {
            console.error('Failed to flush item', err);
            return; // stop to try later
          }
        }
        localStorage.removeItem('dpr_queue');
        alert('Queued DPR reports synced to cloud.');
      }

      // When coming online, flush queue
      window.addEventListener('online', flushQueue);

      // PDF generation using html2pdf (client-side)
      async function generatePDF(reportData, includeImages = true) {
        // Build a small printable DOM element
        const wrap = document.createElement('div');
        wrap.style.padding = '12px';
        wrap.innerHTML = `
          <h2>DPR - ${reportData.reportedBy}</h2>
          <p><strong>Date:</strong> ${reportData.dateTime}</p>
          <p><strong>Equipment:</strong> ${reportData.equipment}</p>
          <p><strong>Problem:</strong><br/>${reportData.problem.replace(/\n/g,'<br/>')}</p>
          <p><strong>Actions:</strong><br/>${reportData.actions.replace(/\n/g,'<br/>')}</p>
          <p><strong>Status:</strong> ${reportData.status}</p>
          <p><strong>Materials:</strong><br/>${reportData.materials || '-'}</p>
        `;

        if (includeImages && reportData.imageURLs && reportData.imageURLs.length) {
          const imgs = document.createElement('div');
          for (const u of reportData.imageURLs) {
            const img = document.createElement('img');
            img.src = u;
            img.style.maxWidth = '100%';
            img.style.marginTop = '8px';
            imgs.appendChild(img);
          }
          wrap.appendChild(imgs);
        }

        // use html2pdf via CDN (loaded below)
        const opt = { margin: 10, filename: `DPR_${reportData.reportedBy}_${Date.now()}.pdf`, html2canvas: { scale: 2 } };
        return html2pdf().from(wrap).set(opt).save();
      }

      // handle form submit
      form.addEventListener('submit', async (e) => {
        e.preventDefault();

        statusMsg.textContent = 'Saving...';

        const data = {
          reportedBy: form.reportedBy.value,
          designation: form.designation.value,
          department: form.department.value,
          dateTime: form.dateTime.value,
          problem: form.problem.value,
          equipment: form.equipment.value,
          actions: form.actions.value,
          status: form.status.value,
          statusDetails: form.statusDetails.value,
          materials: form.materials.value,
          createdAt: new Date().toISOString(),
          userId: form.reportedBy.value // simple multi-user support: group by reporter name
        };

        const files = Array.from(form.upload.files || []);
        if (files.length) data._files = files; // temporary holder for upload

        try {
          const res = await saveReport(data);
          if (res.saved) {
            statusMsg.textContent = 'Saved to cloud ✓';
            alert('DPR saved to cloud successfully.\nYou can open Reports page to view.');
          } else {
            statusMsg.textContent = 'Saved locally (offline)';
            alert('You are offline. DPR saved locally and will sync when online.');
          }

          // generate PDF automatically and upload to storage (if online)
          try {
            await generatePDF({ ...data, imageURLs: data.imageURLs || [] });
          } catch (pdfErr) {
            console.warn('PDF generation failed:', pdfErr);
          }

          form.reset();
        } catch (err) {
          console.error(err);
          statusMsg.textContent = 'Error saving';
          alert('Error saving DPR: ' + err.message);
        }
      });

      // Export PDF button (client-side preview of current form)
      document.getElementById('exportPdfBtn').addEventListener('click', async () => {
        const formData = {
          reportedBy: form.reportedBy.value || 'Unknown',
          dateTime: form.dateTime.value || new Date().toISOString(),
          equipment: form.equipment.value || '-',
          problem: form.problem.value || '-',
          actions: form.actions.value || '-',
          status: form.status.value || '-',
          materials: form.materials.value || ''
        };
        await generatePDF(formData, false);
      });

    });

  </script>
  <!-- html2pdf CDN -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/html2pdf.js/0.9.3/html2pdf.bundle.min.js"></script>
</head>
<body>
  <nav class="container-fluid">
    <ul><li><strong>DPR Generator</strong></li></ul>
    <ul>
      <li><a href="index.html">Home</a></li>
      <li><a href="reports.html">Reports</a></li>
      <li><span id="networkStatus"></span></li>
    </ul>
  </nav>

  <main class="container">
    <div class="card">
      <h1>Daily Progress Report</h1>
      <h2>Instrumentation - NG 1500 Rig</h2>

      <form id="dprForm">
        <label for="reportedBy">1. Reported By</label>
        <input type="text" id="reportedBy" name="reportedBy" placeholder="Enter name" required>

        <label for="designation">Designation</label>
        <input type="text" id="designation" name="designation" placeholder="Enter designation" required>

        <label for="department">Department</label>
        <input type="text" id="department" name="department" placeholder="Enter department" required>

        <label for="dateTime">Date & Time of Reporting</label>
        <input type="datetime-local" id="dateTime" name="dateTime" required>

        <label for="problem">2. Problem Description</label>
        <textarea id="problem" name="problem" placeholder="Describe the issue" required></textarea>

        <label for="equipment">Equipment / Location</label>
        <input type="text" id="equipment" name="equipment" placeholder="Enter equipment or location" required>

        <label for="actions">3. Diagnostic Actions Performed</label>
        <textarea id="actions" name="actions" placeholder="Write diagnostic steps" required></textarea>

        <label for="status">4. Current Status</label>
        <select id="status" name="status" required>
          <option value="">Select status</option>
          <option>Resolved</option>
          <option>In Progress</option>
          <option>Pending Parts</option>
          <option>Under Observation</option>
        </select>

        <label for="statusDetails">Additional Status Details</label>
        <textarea id="statusDetails" name="statusDetails" placeholder="Explain current status"></textarea>

        <label for="materials">5. Material Consumed / Hardware Modification</label>
        <textarea id="materials" name="materials" placeholder="List spares, parts, or modifications"></textarea>

        <label for="upload">6. Upload Images</label>
        <input type="file" id="upload" name="upload" multiple accept="image/*">

        <div style="display: flex; gap: 1rem; justify-content: center; margin-top: 1.5rem;">
          <button type="submit" class="btn-primary">Submit DPR</button>
          <button type="button" id="exportPdfBtn" class="btn-secondary">Export PDF</button>
        </div>

        <p id="statusMsg" style="text-align:center; margin-top:10px;">&nbsp;</p>
      </form>
    </div>
  </main>

  <footer class="container">
    <small><a href="#">Privacy Policy</a> • <a href="#">Terms of Service</a></small>
  </footer>
</body>
</html>
<!-- reports.html - Dashboard to list DPR reports with filters and download links -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>DPR Reports</title>
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@picocss/pico@1/css/pico.min.css">
  <script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.1/firebase-app.js";
    import { getFirestore, collection, getDocs, query, where, orderBy } from "https://www.gstatic.com/firebasejs/10.12.1/firebase-firestore.js";

    // Paste SAME firebaseConfig here
    const firebaseConfig = {
      apiKey: "YOUR_API_KEY",
      authDomain: "YOUR_PROJECT.firebaseapp.com",
      projectId: "YOUR_PROJECT",
      storageBucket: "YOUR_PROJECT.appspot.com",
      messagingSenderId: "SENDER_ID",
      appId: "APP_ID"
    };

    const app = initializeApp(firebaseConfig);
    const db = getFirestore(app);
    window.db = db;

    document.addEventListener('DOMContentLoaded', async () => {
      const list = document.getElementById('reportsList');
      const filterEquipment = document.getElementById('filterEquipment');
      const searchInput = document.getElementById('searchInput');

      async function loadReports() {
        list.innerHTML = 'Loading...';
        let q = collection(db, 'DPR_Reports');
        const snapshot = await getDocs(q);
        const docs = [];
        snapshot.forEach(d => docs.push({ id: d.id, ...d.data() }));

        // Simple client-side filters
        const term = searchInput.value.toLowerCase();
        const equip = filterEquipment.value.toLowerCase();

        const filtered = docs.filter(r => {
          if (equip && !r.equipment.toLowerCase().includes(equip)) return false;
          if (term) {
            const hay = (r.reportedBy + ' ' + r.problem + ' ' + r.equipment + ' ' + (r.materials||'')).toLowerCase();
            return hay.includes(term);
          }
          return true;
        }).sort((a,b) => new Date(b.createdAt) - new Date(a.createdAt));

        if (!filtered.length) {
          list.innerHTML = '<p>No reports found.</p>';
          return;
        }

        list.innerHTML = '';
        for (const r of filtered) {
          const card = document.createElement('article');
          card.innerHTML = `
            <h3>${r.equipment} — ${r.status}</h3>
            <p><strong>By:</strong> ${r.reportedBy} <small>${r.dateTime}</small></p>
            <p>${(r.problem||'').slice(0,200)}${(r.problem||'').length>200? '...':''}</p>
            <p><a href="#" data-id="${r.id}" class="viewBtn">View</a>
               ${r.imageURLs && r.imageURLs.length ? `<a href="${r.imageURLs[0]}" target="_blank">Image</a>`: ''}
            </p>
          `;

          // download PDF link: we will generate client-side PDF from data when clicked
          const downloadBtn = document.createElement('button');
          downloadBtn.textContent = 'Download PDF';
          downloadBtn.addEventListener('click', async () => {
            // generate PDF using html2pdf
            const wrap = document.createElement('div');
            wrap.style.padding = '12px';
            wrap.innerHTML = `\n              <h2>DPR - ${r.reportedBy}</h2>\n              <p><strong>Date:</strong> ${r.dateTime}</p>\n              <p><strong>Equipment:</strong> ${r.equipment}</p>\n              <p><strong>Problem:</strong><br/>${r.problem.replace(/\n/g,'<br/>')}</p>\n            `;
            if (r.imageURLs) {
              for (const u of r.imageURLs) {
                const img = document.createElement('img');
                img.src = u;
                img.style.maxWidth = '100%';
                img.style.marginTop = '8px';
                wrap.appendChild(img);
              }
            }
            html2pdf().from(wrap).set({ margin:10, filename: `DPR_${r.reportedBy}_${r.id}.pdf` }).save();
          });

          card.appendChild(downloadBtn);
          list.appendChild(card);
        }
      }

      document.getElementById('applyFilter').addEventListener('click', loadReports);
      document.getElementById('reload').addEventListener('click', loadReports);

      loadReports();
    });
  </script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/html2pdf.js/0.9.3/html2pdf.bundle.min.js"></script>
</head>
<body>
  <nav class="container-fluid">
    <ul><li><strong>DPR Dashboard</strong></li></ul>
    <ul>
      <li><a href="index.html">Home</a></li>
      <li><a href="reports.html">Reports</a></li>
    </ul>
  </nav>

  <main class="container">
    <h1>DPR Reports</h1>

    <section>
      <label for="filterEquipment">Filter by Equipment</label>
      <input id="filterEquipment" placeholder="e.g. ZCU">
      <label for="searchInput">Search</label>
      <input id="searchInput" placeholder="Search by text...">
      <button id="applyFilter">Apply</button>
      <button id="reload">Reload</button>
    </section>

    <section id="reportsList"></section>
  </main>
</body>
</html>
