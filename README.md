# DPR-instrumentation-NG-1500-1
daily progress report of Team INSTRUMENTATION working at NG 1500 1 DRILLING Rig
<!DOCTYPE html><html lang="en">
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>DPR Form</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background: #f7f7f7;
            padding: 20px;
        }
        .container {
            max-width: 700px;
            margin: 0 auto;
            background: #fff;
            padding: 25px;
            border-radius: 10px;
            box-shadow: 0 2px 8px rgba(0,0,0,0.15);
        }
        h2 {
            text-align: center;
            color: #333;
        }
        label {
            font-weight: bold;
            display: block;
            margin: 12px 0 5px;
        }
        input, textarea, select {
            width: 100%;
            padding: 10px;
            border: 1px solid #ccc;
            border-radius: 5px;
            font-size: 14px;
        }
        textarea {
            height: 120px;
        }
        .submit-btn {
            margin-top: 20px;
            padding: 12px;
            width: 100%;
            background: #4285F4;
            color: #fff;
            font-size: 16px;
            border: none;
            border-radius: 5px;
            cursor: pointer;
        }
        .submit-btn:hover {
            background: #3367D6;
        }
    </style>
</head>
<body>
    <div class="container">
        <h2>DPR (Daily Progress Report)</h2><form>
        <label>1. Reported By</label>
        <input type="text" placeholder="Enter name" required />

        <label>Designation</label>
        <input type="text" placeholder="Enter designation" />

        <label>Department</label>
        <input type="text" placeholder="Enter department" />

        <label>Date & Time of Reporting</label>
        <input type="datetime-local" />

        <label>2. Problem Description</label>
        <textarea placeholder="Describe the issue"></textarea>

        <label>Equipment/Location</label>
        <input type="text" placeholder="Enter equipment or location" />

        <label>3. Diagnostic Actions Performed</label>
        <textarea placeholder="Write diagnostic steps"></textarea>

        <label>4. Current Status</label>
        <select>
            <option>Resolved</option>
            <option>Pending</option>
            <option>Monitoring</option>
        </select>

        <label>Additional Status Details</label>
        <textarea placeholder="Explain current status"></textarea>

        <label>5. Material Consumed / Hardware Modification</label>
        <textarea placeholder="List spares, parts, or modifications"></textarea>

        <label>6. Upload Images</label>
        <input type="file" multiple />

        <button class="submit-btn" type="submit">Submit DPR</button>
                <button type="button" class="submit-btn" onclick="generatePDF()">Export PDF</button>
    </form>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <script>
        function generatePDF() {
            const {{ jsPDF }} = window.jspdf;
            const doc = new jsPDF();
            let y = 10;

            function addText(label, value) {
                doc.text(label + ':', 10, y);
                y += 7;
                doc.text(value || '-', 10, y);
                y += 10;
            }

            const form = document.querySelector('form');
            const inputs = form.querySelectorAll('input, textarea, select');

            doc.setFontSize(14);
            doc.text('DPR Instrumentation NG 1500-1', 10, y);
            y += 10;
            doc.setFontSize(11);

            inputs.forEach(input => {
                if (input.type !== 'file') {
                    addText(input.previousElementSibling?.innerText || 'Field', input.value);
                }
            });

            doc.save('DPR_Report.pdf');
        }
    </script>
</div>

</body>
</html>
