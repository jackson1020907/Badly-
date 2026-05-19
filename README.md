<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>我的羽球比賽對手紀錄</title>
    <style>
        :root {
            --primary: #1e88e5; /* 活力藍 */
            --success: #43a047; /* 勝利綠 */
            --danger: #e53935;  /* 落敗紅 */
            --dark: #37474f;
            --light: #f0f4f8;
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
            max-width: 650px;
            background: white;
            padding: 25px;
            border-radius: 16px;
            box-shadow: 0 8px 24px rgba(0,0,0,0.08);
        }
        h1 {
            text-align: center;
            color: var(--primary);
            margin-bottom: 25px;
            font-size: 28px;
        }
        .form-box {
            background: #f8fafc;
            padding: 15px;
            border-radius: 12px;
            margin-bottom: 25px;
            border: 1px solid #e2e8f0;
        }
        .form-row {
            display: flex;
            gap: 10px;
            margin-bottom: 12px;
        }
        .form-row:last-child {
            margin-bottom: 0;
        }
        input, select, textarea {
            padding: 10px;
            border: 1px solid #cbd5e1;
            border-radius: 8px;
            font-size: 15px;
            outline: none;
            box-sizing: border-box;
        }
        input:focus, select:focus, textarea:focus {
            border-color: var(--primary);
            box-shadow: 0 0 0 3px rgba(30, 136, 229, 0.15);
        }
        .flex-1 { flex: 1; }
        .w-100 { width: 100%; }
        
        button.add-btn {
            background-color: var(--primary);
            color: white;
            border: none;
            padding: 12px;
            border-radius: 8px;
            font-size: 16px;
            cursor: pointer;
            font-weight: bold;
            width: 100%;
            transition: background 0.2s;
        }
        button.add-btn:hover {
            background-color: #1565c0;
        }
        
        .list-title {
            font-size: 18px;
            font-weight: bold;
            margin-top: 20px;
            border-bottom: 2px solid #e2e8f0;
            padding-bottom: 8px;
            color: #475569;
        }
        ul {
            list-style: none;
            padding: 0;
            margin: 0;
        }
        li {
            background: white;
            border: 1px solid #e2e8f0;
            border-radius: 12px;
            padding: 15px;
            margin-top: 12px;
            display: flex;
            justify-content: space-between;
            align-items: flex-start;
            box-shadow: 0 2px 4px rgba(0,0,0,0.02);
        }
        .badge {
            padding: 3px 8px;
            border-radius: 6px;
            font-size: 12px;
            font-weight: bold;
            color: white;
            display: inline-block;
            margin-bottom: 5px;
        }
        .badge.win { background-color: var(--success); }
        .badge.lose { background-color: var(--danger); }
        
        .opponent-name {
            font-size: 18px;
            font-weight: bold;
            margin-bottom: 4px;
        }
        .match-meta {
            font-size: 14px;
            color: #64748b;
            margin-bottom: 6px;
        }
        .notes {
            font-size: 13px;
            color: #475569;
            background: #f1f5f9;
            padding: 6px 10px;
            border-radius: 6px;
            margin-top: 5px;
            line-height: 1.4;
        }
        .delete-btn {
            background-color: #ff4d4f;
            color: white;
            border: none;
            padding: 6px 12px;
            font-size: 13px;
            border-radius: 6px;
            cursor: pointer;
        }
        .delete-btn:hover {
            background-color: #d9363e;
        }
        .empty-hint {
            text-align: center;
            color: #94a3b8;
            margin-top: 30px;
            font-style: italic;
        }
    </style>
</head>
<body>

<div class="container">
    <h1>🏸 我的羽球戰役紀錄</h1>
    
    <!-- 輸入區域 -->
    <div class="form-box">
        <div class="form-row">
            <input type="text" id="opponentInput" class="flex-1" placeholder="對手名字 / 隊伍名稱">
            <select id="typeInput" style="width: 100px;">
                <option value="單打">單打</option>
                <option value="雙打">雙打</option>
                <option value="混雙">混雙</option>
            </select>
        </div>
        <div class="form-row">
            <select id="resultInput" style="width: 100px;">
                <option value="贏了">WIN 🎉</option>
                <option value="輸了">LOSE 📝</option>
            </select>
            <input type="text" id="scoreInput" class="flex-1" placeholder="比分 (例: 21-18, 19-21, 21-15)">
        </div>
        <div class="form-row">
            <textarea id="notesInput" class="w-100" rows="2" placeholder="備忘錄 (例如：對方後場高遠球很強、注意切球、網前要抓...)"></textarea>
        </div>
        <button class="add-btn" onclick="addMatch()">儲存本次比賽</button>
    </div>

    <!-- 歷史紀錄列表 -->
    <div class="list-title">對手與戰績清單</div>
    <div id="emptyHint" class="empty-hint">還沒有比賽紀錄，拿上球拍去贏一場吧！</div>
    <ul id="opponentList"></ul>
</div>

<script>
    document.addEventListener('DOMContentLoaded', loadMatches);

    function addMatch() {
        const nameInput = document.getElementById('opponentInput');
        const typeInput = document.getElementById('typeInput');
        const resultInput = document.getElementById('resultInput');
        const scoreInput = document.getElementById('scoreInput');
        const notesInput = document.getElementById('notesInput');

        const name = nameInput.value.trim();
        if (name === '') {
            alert('請輸入對手名稱！');
            return;
        }

        const matches = getStorage();
        matches.push({
            id: Date.now(),
            name: name,
            type: typeInput.value,
            result: resultInput.value,
            score: scoreInput.value.trim() || '未填寫比分',
            notes: notesInput.value.trim(),
            date: new Date().toLocaleDateString()
        });
        saveStorage(matches);

        renderList();
        
        // 清空欄位
        nameInput.value = '';
        scoreInput.value = '';
        notesInput.value = '';
        nameInput.focus();
    }

    function deleteMatch(id) {
        if(confirm('確定要刪除這筆比賽紀錄嗎？')) {
            let matches = getStorage();
            matches = matches.filter(item => item.id !== id);
            saveStorage(matches);
            renderList();
        }
    }

    function getStorage() {
        const data = localStorage.getItem('badmintonMatches');
        return data ? JSON.parse(data) : [];
    }

    function saveStorage(data) {
        localStorage.setItem('badmintonMatches', JSON.stringify(data));
    }

    function renderList() {
        const list = document.getElementById('opponentList');
        const hint = document.getElementById('emptyHint');
        const matches = getStorage();

        list.innerHTML = '';
        
        if (matches.length === 0) {
            hint.style.display = 'block';
            return;
        } else {
            hint.style.display = 'none';
        }

        // 新紀錄排在上面
        matches.reverse().forEach(item => {
            const li = document.createElement('li');
            const isWin = item.result === '贏了';
            
            li.innerHTML = `
                <div>
                    <span class="badge ${isWin ? 'win' : 'lose'}">${item.result} (${item.type})</span>
                    <div class="opponent-name">🆚 ${item.name}</div>
                    <div class="match-meta">📅 ${item.date} ｜ 📊 比分：${item.score}</div>
                    ${item.notes ? `<div class="notes">💡 <strong>備忘：</strong>${item.notes}</div>` : ''}
                </div>
                <button class="delete-btn" onclick="deleteMatch(${item.id})">刪除</button>
            `;
            list.appendChild(li);
        });
    }

    function loadMatches() {
        renderList();
    }
</script>

</body>
</html>
