<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>我的比賽對手紀錄</title>
    <style>
        :root {
            --primary: #4a90e2;
            --dark: #333;
            --light: #f4f6f9;
        }
        body {
            font-family: 'Helvetica Neue', Arial, sans-serif;
            background-color: var(--light);
            color: var(--dark);
            margin: 0;
            padding: 20px;
            display: flex;
            justify-content: center;
        }
        .container {
            width: 100%;
            max-width: 600px;
            background: white;
            padding: 25px;
            border-radius: 12px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.05);
        }
        h1 {
            text-align: center;
            color: var(--primary);
            margin-bottom: 25px;
        }
        .form-group {
            display: flex;
            gap: 10px;
            margin-bottom: 20px;
        }
        input {
            flex: 1;
            padding: 12px;
            border: 2px solid #ddd;
            border-radius: 8px;
            font-size: 16px;
            outline: none;
            transition: border-color 0.3s;
        }
        input:focus {
            border-color: var(--primary);
        }
        button {
            background-color: var(--primary);
            color: white;
            border: none;
            padding: 12px 20px;
            border-radius: 8px;
            font-size: 16px;
            cursor: pointer;
            font-weight: bold;
            transition: background 0.3s;
        }
        button:hover {
            background-color: #357abd;
        }
        .list-title {
            font-size: 18px;
            font-weight: bold;
            margin-top: 20px;
            border-bottom: 2px solid var(--light);
            padding-bottom: 8px;
        }
        ul {
            list-style: none;
            padding: 0;
            margin: 0;
        }
        li {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 15px 10px;
            border-bottom: 1px solid #eee;
            animation: fadeIn 0.3s ease;
        }
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(10px); }
            to { opacity: 1; transform: translateY(0); }
        }
        .delete-btn {
            background-color: #ff4d4f;
            padding: 6px 12px;
            font-size: 14px;
            border-radius: 6px;
        }
        .delete-btn:hover {
            background-color: #d9363e;
        }
        .empty-hint {
            text-align: center;
            color: #999;
            margin-top: 30px;
            font-style: italic;
        }
    </style>
</head>
<body>

<div class="container">
    <h1>🥊 比賽對手紀錄簿</h1>
    
    <!-- 輸入區域 -->
    <div class="form-group">
        <input type="text" id="opponentInput" placeholder="輸入對手名字或隊伍名稱..." onkeypress="handleKeyPress(event)">
        <button onclick="addOpponent()">新增紀錄</button>
    </div>

    <!-- 歷史紀錄列表 -->
    <div class="list-title">對手清單</div>
    <div id="emptyHint" class="empty-hint">目前還沒有紀錄，快去贏一場比賽吧！</div>
    <ul id="opponentList"></ul>
</div>

<script>
    // 網頁載入時，自動讀取之前存過的資料
    document.addEventListener('DOMContentLoaded', loadOpponents);

    function addOpponent() {
        const input = document.getElementById('opponentInput');
        const name = input.value.trim();
        
        if (name === '') {
            alert('請輸入對手名稱！');
            return;
        }

        // 取得現有資料、加入新資料、存回 LocalStorage
        const opponents = getStorage();
        opponents.push({
            id: Date.now(),
            name: name,
            date: new Date().toLocaleDateString()
        });
        saveStorage(opponents);

        // 重新渲染畫面並清空輸入框
        renderList();
        input.value = '';
        input.focus();
    }

    // 支援按 Enter 鍵直接送出
    function handleKeyPress(event) {
        if (event.key === 'Enter') {
            addOpponent();
        }
    }

    function deleteOpponent(id) {
        let opponents = getStorage();
        opponents = opponents.filter(item => item.id !== id);
        saveStorage(opponents);
        renderList();
    }

    // LocalStorage 相關操作（讓資料關掉網頁也不會不見）
    function getStorage() {
        const data = localStorage.getItem('matchOpponents');
        return data ? JSON.parse(data) : [];
    }

    function saveStorage(data) {
        localStorage.setItem('matchOpponents', JSON.stringify(data));
    }

    // 畫面渲染
    function renderList() {
        const list = document.getElementById('opponentList');
        const hint = document.getElementById('emptyHint');
        const opponents = getStorage();

        list.innerHTML = '';
        
        if (opponents.length === 0) {
            hint.style.display = 'block';
            return;
        } else {
            hint.style.display = 'none';
        }

        // 最新的紀錄排在最上面
        opponents.reverse().forEach(item => {
            const li = document.createElement('li');
            li.innerHTML = `
                <div>
                    <strong>${item.name}</strong> 
                    <span style="color: #888; font-size: 13px; margin-left: 10px;">(${item.date})</span>
                </div>
                <button class="delete-btn" onclick="deleteOpponent(${item.id})">刪除</button>
            `;
            list.appendChild(li);
        });
    }

    function loadOpponents() {
        renderList();
    }
</script>

</body>
</html>
