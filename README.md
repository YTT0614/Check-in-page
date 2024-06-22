<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>旭麥上下班打卡</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100vh;
            margin: 0;
            background-color: #f0f0f0;
        }
        .container {
            background: #fff;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
            width: 90%;
            max-width: 600px;
            box-sizing: border-box;
            overflow-y: auto;
            max-height: 80vh;
        }
        .button {
            margin: 10px 0;
            padding: 10px 20px;
            font-size: 16px;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            width: 100%;
        }
        .clock-in {
            background-color: #4CAF50;
            color: white;
        }
        .clock-out {
            background-color: #f44336;
            color: white;
        }
        .log {
            margin-top: 20px;
            text-align: left;
            width: 100%;
            overflow-y: auto;
            max-height: 300px;
            border: 1px solid #ccc;
            padding: 10px;
            border-radius: 5px;
            background: #fafafa;
        }
        .employee-select {
            margin-bottom: 20px;
            width: 100%;
        }
        .employee-select select {
            width: 100%;
            padding: 10px;
            font-size: 16px;
            border: 1px solid #ccc;
            border-radius: 5px;
        }
        .hidden {
            display: none;
        }
        .password-input {
            margin: 20px 0;
            width: 100%;
            padding: 10px;
            font-size: 16px;
            border: 1px solid #ccc;
            border-radius: 5px;
        }
    </style>
    <!-- 引入 SheetJS 庫 -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.17.0/xlsx.full.min.js"></script>
</head>
<body>
    <div class="container">
        <h1>旭麥上下班打卡</h1>
        <div class="employee-select">
            <label for="employee">姓名:</label>
            <select id="employee">
                <option value="甫">甫</option>
                <option value="婷">婷</option>
                <option value="媽媽">媽媽</option>
                <option value="阿嬤">阿嬤</option>
            </select>
        </div>
        <button class="button clock-in" onclick="clockIn()">上班打卡</button>
        <button class="button clock-out" onclick="clockOut()">下班打卡</button>
        <button class="button" onclick="showDailyLog()">查看當天打卡記錄</button>
        <div class="log hidden" id="daily-log"></div>
        <button class="button" onclick="showPasswordPrompt('history')">查看完整打卡記錄</button>
        <div class="hidden" id="password-prompt">
            <input type="password" class="password-input" id="password" placeholder="輸入密碼">
            <button class="button" onclick="verifyPassword()">確認</button>
        </div>
        <div class="log hidden" id="full-log"></div>
        <div class="hidden" id="full-log-controls">
            <button class="button" onclick="exportExcel()">導出Excel</button>
            <button class="button" onclick="showPasswordPrompt('clear')">清除所有記錄</button>
            <button class="button" onclick="hideFullLog()">返回</button>
        </div>
    </div>

    <script>
        const correctPassword = "8818";

        function getCurrentTime() {
            const now = new Date();
            return now.toLocaleString('zh-TW', { timeZone: 'Asia/Taipei' });
        }

        function clockIn() {
            const employee = document.getElementById('employee').value;
            const time = getCurrentTime();
            const record = { employee, type: '上班', time };
            saveRecord(record);
            updateLog('daily-log', true);
        }

        function clockOut() {
            const employee = document.getElementById('employee').value;
            const time = getCurrentTime();
            const record = { employee, type: '下班', time };
            saveRecord(record);
            updateLog('daily-log', true);
        }

        function saveRecord(record) {
            let records = JSON.parse(localStorage.getItem('attendanceRecords')) || [];
            records.push(record);
            localStorage.setItem('attendanceRecords', JSON.stringify(records));
        }

        function updateLog(elementId, filterToday = false) {
            const records = JSON.parse(localStorage.getItem('attendanceRecords')) || [];
            const logDiv = document.getElementById(elementId);
            logDiv.innerHTML = '';
            const now = new Date();
            const today = now.toLocaleDateString('zh-TW', { timeZone: 'Asia/Taipei' });

            records.forEach(record => {
                if ((!filterToday || record.time.startsWith(today)) && (elementId !== 'daily-log' || record.employee === document.getElementById('employee').value)) {
                    const logEntry = document.createElement('p');
                    logEntry.textContent = `${record.employee} - ${record.type} - ${record.time}`;
                    logDiv.appendChild(logEntry);
                }
            });
        }

        function showDailyLog() {
            document.getElementById('daily-log').classList.remove('hidden');
            updateLog('daily-log', true);
        }

        function showPasswordPrompt(action) {
            document.getElementById('password-prompt').classList.remove('hidden');
            document.getElementById('password-prompt').dataset.action = action;
        }

        function verifyPassword() {
            const inputPassword = document.getElementById('password').value;
            if (inputPassword === correctPassword) {
                const action = document.getElementById('password-prompt').dataset.action;
                if (action === 'history') {
                    document.getElementById('full-log').classList.remove('hidden');
                    document.getElementById('full-log-controls').classList.remove('hidden');
                    updateLog('full-log');
                } else if (action === 'clear') {
                    clearRecords();
                }
                document.getElementById('password-prompt').classList.add('hidden');
            } else {
                alert('密碼錯誤，請重新輸入');
            }
        }

        function exportExcel() {
            const records = JSON.parse(localStorage.getItem('attendanceRecords')) || [];
            const ws_data = [
                ["員工", "打卡類型", "時間"]
            ];
            records.forEach(record => {
                ws_data.push([record.employee, record.type, record.time]);
            });

            const ws = XLSX.utils.aoa_to_sheet(ws_data);
            const wb = XLSX.utils.book_new();
            XLSX.utils.book_append_sheet(wb, ws, "打卡記錄");

            XLSX.writeFile(wb, "attendance_records.xlsx");
        }

        function clearRecords() {
            if (confirm('確定要清除所有記錄嗎？')) {
                localStorage.removeItem('attendanceRecords');
                updateLog('daily-log', true);
                updateLog('full-log');
            }
        }

        function hideFullLog() {
            document.getElementById('full-log').classList.add('hidden');
            document.getElementById('full-log-controls').classList.add('hidden');
        }
    </script>
</body>
</html>
