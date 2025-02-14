<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Time Tracker</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
        }
        h1 {
            color: #333;
        }
        input, button {
            padding: 10px;
            margin: 5px 0;
            font-size: 16px;
        }
        button {
            cursor: pointer;
            background-color: #007bff;
            color: white;
            border: none;
            border-radius: 5px;
        }
        button:hover {
            background-color: #0056b3;
        }
        .entry {
            border-bottom: 1px solid #ccc;
            padding: 10px 0;
        }
        .entry strong {
            display: block;
            font-size: 18px;
        }
        .summary {
            margin-top: 20px;
            padding: 10px;
            background-color: #f9f9f9;
            border-radius: 5px;
        }
    </style>
</head>
<body>
    <h1>Time Tracker</h1>
    <input type="text" id="employeeName" placeholder="Enter your name">
    <button id="startBtn">Start Work</button>
    <button id="endBtn">End Work</button>

    <h2>Time Entries</h2>
    <div id="entries"></div>

    <div class="summary">
        <h2>Summary</h2>
        <div id="summary"></div>
    </div>

    <!-- Include EmailJS SDK -->
    <script src="https://cdn.jsdelivr.net/npm/@emailjs/browser@3/dist/email.min.js"></script>
    <script>
        // Initialize EmailJS with your Public Key
        emailjs.init('mQW66PDmWaObBxgzW'); // Replace with your actual Public Key
    </script>

    <script>
        const employeeNameInput = document.getElementById('employeeName');
        const startBtn = document.getElementById('startBtn');
        const endBtn = document.getElementById('endBtn');
        const entriesDiv = document.getElementById('entries');
        const summaryDiv = document.getElementById('summary');

        let currentEntry = null;

        // Load saved entries from local storage
        let entries = JSON.parse(localStorage.getItem('timeEntries')) || [];

        // Display entries
        function renderEntries() {
            entriesDiv.innerHTML = entries.map((entry, index) => `
                <div class="entry">
                    <strong>${entry.employeeName}</strong>
                    <div>Start: ${new Date(entry.startTime).toLocaleString()}</div>
                    <div>End: ${entry.endTime ? new Date(entry.endTime).toLocaleString() : 'In Progress'}</div>
                    <div>Hours: ${entry.endTime ? calculateHours(entry.startTime, entry.endTime).toFixed(2) : '--'}</div>
                    <button onclick="deleteEntry(${index})">Delete</button>
                </div>
            `).join('');
            renderSummary();
        }

        // Calculate hours worked
        function calculateHours(startTime, endTime) {
            const start = new Date(startTime);
            const end = new Date(endTime);
            return (end - start) / (1000 * 60 * 60); // Convert milliseconds to hours
        }

        // Render summary
        function renderSummary() {
            const summary = {};

            entries.forEach(entry => {
                if (!entry.endTime) return; // Skip incomplete entries
                const hours = calculateHours(entry.startTime, entry.endTime);
                if (summary[entry.employeeName]) {
                    summary[entry.employeeName] += hours;
                } else {
                    summary[entry.employeeName] = hours;
                }
            });

            summaryDiv.innerHTML = Object.keys(summary).map(name => `
                <div><strong>${name}</strong>: ${summary[name].toFixed(2)} hours</div>
            `).join('');
        }

        // Send email with time entry details
        function sendEmail(entry) {
            const templateParams = {
                employeeName: entry.employeeName,
                startTime: new Date(entry.startTime).toLocaleString(),
                endTime: new Date(entry.endTime).toLocaleString(),
                hoursWorked: calculateHours(entry.startTime, entry.endTime).toFixed(2)
            };

            emailjs.send('service_seniy4t', 'template_zn3u39a', templateParams)
                .then(() => {
                    console.log('Email sent successfully!');
                }, (error) => {
                    console.error('Failed to send email:', error);
                });
        }

        // Start work
        startBtn.addEventListener('click', () => {
            const employeeName = employeeNameInput.value.trim();
            if (!employeeName) return alert('Please enter your name.');

            currentEntry = {
                employeeName,
                startTime: new Date().toISOString(),
                endTime: null
            };

            entries.push(currentEntry);
            localStorage.setItem('timeEntries', JSON.stringify(entries));
            renderEntries();
            alert('Work started.');
        });

        // End work
        endBtn.addEventListener('click', () => {
            if (!currentEntry) return alert('No active work session.');

            currentEntry.endTime = new Date().toISOString();
            localStorage.setItem('timeEntries', JSON.stringify(entries));
            renderEntries();
            alert('Work ended.');

            // Send email with time entry details
            sendEmail(currentEntry);

            currentEntry = null;
        });

        // Delete an entry
        window.deleteEntry = (index) => {
            if (confirm('Are you sure you want to delete this entry?')) {
                entries.splice(index, 1);
                localStorage.setItem('timeEntries', JSON.stringify(entries));
                renderEntries();
            }
        };

        // Initial render
        renderEntries();
    </script>
</body>
</html>
