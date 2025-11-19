<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Daily Progress Report (DPR)</title>
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@picocss/pico@1/css/pico.min.css">
  <style>
    body {
      background-color: #f4f6f8;
    }
    .card {
      background: #fff;
      padding: 2rem;
      border-radius: 12px;
      box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
      max-width: 700px;
      margin: 2rem auto;
    }
    h1, h2 {
      text-align: center;
      color: #2b2d42;
    }
    label {
      margin-top: 1rem;
      font-weight: bold;
    }
    input, textarea, select {
      margin-bottom: 1rem;
      border-radius: 6px;
    }
    button {
      border-radius: 6px;
      font-weight: 600;
    }
    .btn-primary {
      background-color: #007bff;
      color: white;
    }
    .btn-secondary {
      background-color: #6c757d;
      color: white;
    }
  </style>
</head>
<body>

  <nav class="container-fluid">
    <ul><li><strong>DPR Generator</strong></li></ul>
    <ul>
      <li><a href="#">Home</a></li>
      <li><a href="#">Reports</a></li>
      <li><a href="#" role="button">Cloud Sync</a></li>
    </ul>
  </nav>

  <main class="container">
    <div class="card">
      <hgroup>
        <h1>Daily Progress Report</h1>
        <h2>Instrumentation - NG 1500 Rig</h2>
      </hgroup>

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
          <button type="submit" class="btn-primary" onclick="event.preventDefault(); alert('DPR submitted successfully!');">Submit DPR</button>
          <button type="button" class="btn-secondary" onclick="alert('PDF export feature coming soon!');">Export PDF</button>
        </div>
      </form>
    </div>
  </main>

  <section aria-label="Subscribe example">
    <div class="container">
      <article>
        <hgroup>
          <h2>Stay Connected</h2>
          <h3>Get notified when cloud sync & PDF export are ready</h3>
        </hgroup>
        <form class="grid">
          <input type="text" id="firstname" name="firstname" placeholder="Your first name" aria-label="First name" required />
          <input type="email" id="email" name="email" placeholder="Your email" aria-label="Email" required />
          <button type="submit" onclick="event.preventDefault(); alert('Subscribed!');">Subscribe</button>
        </form>
      </article>
    </div>
  </section>

  <footer class="container">
    <small><a href="#">Privacy Policy</a> • <a href="#">Terms of Service</a></small>
  </footer>

</body>
</html>
