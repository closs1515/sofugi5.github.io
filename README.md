# sofugi5.github.io
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>伝言掲示板</title>
    <!-- Tailwind CSS CDNをロード -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* デフォルトフォントをInterに設定 */
        body {
            font-family: 'Inter', sans-serif;
        }
        /* テキストエリアの見た目を調整 */
        .text-area {
            min-height: 250px;
            resize: vertical;
        }
        /* Flexboxのアイテムが改行された際に揃うように最小幅を設定 */
        .message-card {
            /* カードの最小幅を確保し、レスポンシブに対応 */
            min-width: 250px; 
            /* 1/2より小さくならないようにする (md以下では w-full) */
            max-width: 100%; 
            flex-grow: 1; /* 柔軟に幅を取る */
        }
        /* ボタンにホバー時のアニメーションを追加 */
        .main-button {
            transition: all 0.2s ease-in-out;
            transform-origin: center;
        }
        .main-button:hover {
            transform: scale(1.05);
            /* ホバー時に影を青色に強調 */
            box-shadow: 0 10px 15px -3px rgba(59, 130, 246, 0.5), 0 4px 6px -2px rgba(59, 130, 246, 0.2);
        }
    </style>
</head>
<!-- 画面全体を占め、背景を bg-gray-100 に変更 -->
<body class="bg-gray-100 flex justify-center min-h-screen px-2 py-8">

    <!-- メインコンテナの最大幅を max-w-full (画面幅全体) に変更し、見た目を強調 -->
    <div class="w-full bg-white p-6 md:p-12 rounded-3xl shadow-2xl border border-blue-100 transition-all duration-300">
        
        <!-- ヘッダーとメッセージ追加ボタン (右上に配置) -->
        <div class="flex justify-between items-center mb-8 pb-4 border-b-4 border-blue-500">
            <!-- タイトルを text-5xl に拡大し、強調色を追加 -->
            <h1 class="text-5xl font-extrabold text-blue-700 tracking-tight">
                伝言掲示板<span class="text-3xl text-gray-500 font-medium ml-2">(情報電子科)</span>
            </h1>
            <button 
                id="add-button" 
                class="main-button py-3 px-6 bg-blue-600 text-white font-bold rounded-xl shadow-lg hover:bg-blue-700 transition duration-200"
                aria-label="新しいメッセージの追加"
            >
                + 新しい伝言
            </button>
        </div>

        <!--
        ==================================================
        表示用ビュー (#display-view) - 初期表示
        ==================================================
        -->
        <div id="display-view" class="space-y-6">
            <!-- 「現在有効なメッセージリスト」のh2タグを削除 -->
            
            <!-- Flexboxに変更: 横並びになり、カードの幅は内容に依存しつつ、必要に応じて折り返す -->
            <!-- メッセージリストをレンダリングする場所: flex, flex-wrap, gap を設定 -->
            <div id="messages-list" class="flex flex-wrap gap-8 justify-start">
                <!-- プレースホルダーをより目立つデザインに変更 -->
                <p id="no-messages-placeholder" class="text-gray-500 text-center p-12 border-4 border-dashed border-blue-200 bg-blue-50 rounded-xl w-full text-lg">
                    現在、伝言はありません。「+ 新しい伝言」ボタンから作成しましょう！
                </p>
            </div>

        </div>

        <!--
        ==================================================
        入力用ビュー (#input-view) - 初期非表示
        ==================================================
        -->
        <div id="input-view" class="hidden p-8 border-4 border-blue-200 rounded-2xl bg-blue-50 shadow-inner">
            <h2 class="text-3xl font-bold text-blue-800 mb-8">新しい伝言の作成</h2>
            
            <div class="flex flex-col md:flex-row gap-8">
                
                <!-- 左側: テキスト入力フォーム -->
                <div class="flex-1">
                    <!-- ラベルを text-xl に拡大し、強調色を追加 -->
                    <label for="free-text" class="block text-xl font-medium text-blue-700 mb-3">
                        伝言の内容を入力してください:
                    </label>
                    <textarea 
                        id="free-text" 
                        class="text-area w-full p-5 border-2 border-blue-300 rounded-xl focus:ring-4 focus:ring-blue-400 focus:border-blue-600 outline-none transition duration-150 shadow-md placeholder-gray-400 text-gray-800"
                        placeholder="伝言をここに記述..."
                        aria-label="フリーテキスト入力エリア"
                    ></textarea>
                </div>

                <!-- 右側: オプション（名前、時間、送信ボタン） -->
                <div class="md:w-72 flex flex-col space-y-6">
                    
                    <!-- 1. 名前を選択できる欄 -->
                    <div>
                        <!-- ラベル色を青に変更 -->
                        <label for="user-name" class="block text-base font-medium text-blue-700 mb-2">
                            差出人の名前
                        </label>
                        <!-- セレクトボックスのデザインを調整 -->
                        <select id="user-name" class="w-full p-3 border border-blue-300 rounded-lg shadow-sm focus:ring-blue-500 focus:border-blue-500 bg-white text-gray-700 font-semibold">
                            <option value="アオイ">アオイ</option>
                            <option value="ハルト">ハルト</option>
                            <option value="ユウキ">ユウキ</option>
                            <option value="先生">先生</option>
                            <option value="ゲスト">ゲスト</option>
                        </select>
                    </div>

                    <!-- 2. 文章を消す時間を選択できる欄 (時間と分の入力) -->
                    <div>
                        <!-- ラベル色を青に変更 -->
                        <label class="block text-base font-medium text-blue-700 mb-2">
                            自動消去までの時間
                        </label>
                        <div class="flex space-x-3">
                            <div class="flex-1">
                                <input type="number" id="expiry-hours" min="0" value="0" class="w-full p-3 border border-blue-300 rounded-lg shadow-sm focus:ring-blue-500 focus:border-blue-500 text-gray-700 text-center font-bold" aria-label="時間">
                                <span class="block text-xs text-gray-600 mt-1 text-center">時間</span>
                            </div>
                            <div class="flex-1">
                                <input type="number" id="expiry-minutes" min="0" max="59" value="5" class="w-full p-3 border border-blue-300 rounded-lg shadow-sm focus:ring-blue-500 focus:border-blue-500 text-gray-700 text-center font-bold" aria-label="分">
                                <span class="block text-xs text-gray-600 mt-1 text-center">分</span>
                            </div>
                        </div>
                    </div>

                    <!-- 3. ボタン: 送信/キャンセル -->
                    <div class="flex flex-col space-y-3 mt-auto pt-4 border-t border-blue-200">
                        <!-- 送信ボタンを緑色で強調 -->
                        <button 
                            id="send-button"
                            class="main-button py-4 px-4 bg-green-500 hover:bg-green-600 text-white font-bold text-lg rounded-xl shadow-lg transition duration-200 focus:outline-none focus:ring-4 focus:ring-green-300 disabled:bg-green-300"
                            aria-label="メッセージを送信"
                        >
                            伝言を送信
                        </button>
                        <!-- キャンセルボタンのテキスト変更 -->
                        <button 
                            id="cancel-button"
                            class="py-2 px-4 bg-gray-400 hover:bg-gray-500 text-white font-semibold rounded-lg transition duration-200 focus:outline-none focus:ring-4 focus:ring-gray-300"
                            aria-label="キャンセルして戻る"
                        >
                            キャンセルして戻る
                        </button>
                    </div>
                </div>
            </div>
            
            <!-- エラー/成功メッセージ表示エリア -->
            <div id="input-message-area" class="mt-6">
                <!-- JavaScriptでメッセージが挿入されます -->
            </div>
        </div>
        
    </div>

    <script>
        // DOM要素の取得
        const addButton = document.getElementById('add-button');
        const sendButton = document.getElementById('send-button');
        const cancelButton = document.getElementById('cancel-button');

        const displayView = document.getElementById('display-view');
        const inputView = document.getElementById('input-view');

        const freeTextarea = document.getElementById('free-text');
        const userNameSelect = document.getElementById('user-name');
        const expiryHoursInput = document.getElementById('expiry-hours');
        const expiryMinutesInput = document.getElementById('expiry-minutes');

        const messagesList = document.getElementById('messages-list');
        const noMessagesPlaceholder = document.getElementById('no-messages-placeholder');
        const inputMessageArea = document.getElementById('input-message-area');

        const LOCAL_STORAGE_KEY = 'ephemeral_messages';

        // =================================================================
        // ユーティリティ関数
        // =================================================================

        /**
         * 指定された領域にメッセージを表示
         */
        const displayMessage = (areaElement, text, type = 'info') => {
            let bgColor = 'bg-blue-100';
            let textColor = 'text-blue-800';
            if (type === 'error') {
                bgColor = 'bg-red-100';
                textColor = 'text-red-700';
            } else if (type === 'success') {
                bgColor = 'bg-green-100';
                textColor = 'text-green-700';
            }

            areaElement.innerHTML = `
                <div class="p-3 ${bgColor} ${textColor} rounded-lg text-center font-medium transition duration-300 shadow-sm">
                    ${text}
                </div>
            `;
            areaElement.classList.remove('hidden');
        };

        /**
         * ビューの切り替え
         */
        const switchToView = (viewId) => {
            if (viewId === 'display') {
                displayView.classList.remove('hidden');
                inputView.classList.add('hidden');
                addButton.classList.remove('hidden');
                renderMessages(); // 表示に戻る際にリストを更新
            } else if (viewId === 'input') {
                displayView.classList.add('hidden');
                inputView.classList.remove('hidden');
                addButton.classList.add('hidden');
                inputMessageArea.innerHTML = ''; // メッセージエリアをクリア
            }
        };

        // =================================================================
        // LocalStorage データ操作
        // =================================================================

        /**
         * localStorageからメッセージを読み込む
         */
        const loadMessages = () => {
            const json = localStorage.getItem(LOCAL_STORAGE_KEY);
            try {
                return json ? JSON.parse(json) : [];
            } catch (e) {
                console.error("ローカルストレージからの読み込みエラー:", e);
                return [];
            }
        };

        /**
         * localStorageにメッセージを保存する
         */
        const saveMessages = (messages) => {
            localStorage.setItem(LOCAL_STORAGE_KEY, JSON.stringify(messages));
        };

        /**
         * 有効期限切れのメッセージを削除し、有効なメッセージだけを返す
         */
        const cleanAndGetValidMessages = () => {
            const allMessages = loadMessages();
            const now = Date.now();

            // 有効期限がnull（永続的）か、有効期限が現在時刻よりも未来のものだけをフィルタリング
            const validMessages = allMessages.filter(msg => {
                return msg.expiry_timestamp === null || msg.expiry_timestamp > now;
            });

            // 削除されたメッセージがあった場合、localStorageを更新
            if (allMessages.length !== validMessages.length) {
                saveMessages(validMessages);
            }

            return validMessages.reverse(); // 最新のものを上に表示するために逆順にする
        };

        // =================================================================
        // 表示ロジック
        // =================================================================

        /**
         * メッセージドキュメントをHTMLカードとしてレンダリング
         */
        const renderMessageCard = (message) => {
            const createdDate = new Date(message.created_at).toLocaleString('ja-JP');
            // 差出人が「先生」の場合に特別なスタイルを適用
            const senderClass = message.sender === '先生' ? 'bg-red-500 text-white p-1 px-2 rounded-full text-sm font-bold shadow' : 'text-blue-600 font-extrabold';
            
            // カードはFlexboxの子要素として、内容に合わせて幅を取り、最大幅は max-w-md
            return `
                <div class="message-card p-6 bg-white border border-gray-200 rounded-xl shadow-xl hover:shadow-2xl transition-shadow duration-300 border-l-4 border-blue-400"> 
                    <div class="flex justify-between items-center mb-3 border-b border-gray-100 pb-2">
                        <!-- 送信者名を強調 -->
                        <p class="text-xl ${senderClass}">${message.sender || '匿名'}</p>
                        <!-- 日付を text-sm から text-base に拡大 -->
                        <p class="text-sm text-gray-500">送信日時: ${createdDate}</p>
                    </div>
                    
                    <div class="mt-2 p-4 bg-blue-50 rounded-lg border border-blue-100">
                        <!-- メッセージ本文を break-words で強制折り返し -->
                        <p class="whitespace-pre-wrap break-words text-xl text-gray-700 font-medium">${message.message}</p>
                    </div>
                </div>
            `;
        };

        /**
         * メッセージリスト全体をレンダリング
         */
        const renderMessages = () => {
            const validMessages = cleanAndGetValidMessages();
            
            messagesList.innerHTML = ''; // リストをクリア

            if (validMessages.length === 0) {
                // メッセージがない場合のプレースホルダー表示
                noMessagesPlaceholder.classList.remove('hidden');
                // Flexboxの子要素として挿入するため、innerHTMLで直接追加
                messagesList.innerHTML = `<p id="no-messages-placeholder" class="text-gray-500 text-center p-12 border-4 border-dashed border-blue-200 bg-blue-50 rounded-xl w-full text-lg">
                    現在、伝言はありません。「+ 新しい伝言」ボタンから作成しましょう！
                </p>`;
            } else {
                validMessages.forEach(msg => {
                    messagesList.innerHTML += renderMessageCard(msg);
                });
            }
        };

        // =================================================================
        // 入力ロジック (送信ボタンの処理)
        // =================================================================

        const handleSendClick = () => {
            const textContent = freeTextarea.value.trim();
            
            // 必須チェック
            if (textContent === "") {
                displayMessage(inputMessageArea, "文章が入力されていません。何か書いてから送信してください。", 'error');
                return;
            }

            // UIを無効化
            sendButton.disabled = true;
            cancelButton.disabled = true;
            sendButton.textContent = "送信中...";
            inputMessageArea.innerHTML = ''; 

            try {
                // フォームデータの取得
                const name = userNameSelect.value;
                const hours = parseInt(expiryHoursInput.value, 10) || 0;
                const minutes = parseInt(expiryMinutesInput.value, 10) || 0;
                
                // 消去予定時刻を計算 (UNIXミリ秒)
                const expiryTimeMs = (hours * 60 * 60 * 1000) + (minutes * 60 * 1000);
                const expiryTimestamp = expiryTimeMs > 0
                    ? Date.now() + expiryTimeMs
                    : null; // 0時間0分の場合は永続的

                // 新しいメッセージオブジェクト
                const newMessage = {
                    id: Date.now() + Math.random().toString(36).substring(2, 9), // 一意なID
                    sender: name,
                    message: textContent,
                    expiry_timestamp: expiryTimestamp, 
                    created_at: Date.now(),
                };
                
                // メッセージをロードし、追加し、保存
                const messages = loadMessages();
                messages.push(newMessage);
                saveMessages(messages);

                displayMessage(inputMessageArea, "メッセージは正常に保存されました。", 'success');
                
                // フォームをリセット
                freeTextarea.value = ''; 
                expiryHoursInput.value = 0;
                expiryMinutesInput.value = 5;

                // 成功したら表示ビューに戻る
                setTimeout(() => {
                    inputMessageArea.innerHTML = ''; 
                    switchToView('display');
                }, 1000); // 1秒後に切り替え

            } catch (error) {
                console.error("メッセージの保存エラー:", error);
                displayMessage(inputMessageArea, "メッセージの保存中にエラーが発生しました。", 'error');
            } finally {
                // UIを元に戻す
                sendButton.disabled = false;
                cancelButton.disabled = false;
                sendButton.textContent = "伝言を送信";
            }
        };


        // =================================================================
        // アプリケーションの開始 (すべてのイベントリスナーをここで確実にアタッチ)
        // =================================================================

        // ページロード時にメッセージリストをレンダリングし、リスナーをアタッチ
        window.onload = () => {
            // メッセージ追加ボタン (表示 -> 入力へ)
            if (addButton) {
                addButton.addEventListener('click', () => {
                    switchToView('input');
                });
            }

            // キャンセルボタン (入力 -> 表示へ)
            if (cancelButton) {
                cancelButton.addEventListener('click', () => {
                    switchToView('display');
                });
            }

            // 送信ボタン
            if (sendButton) {
                sendButton.addEventListener('click', handleSendClick);
            }

            switchToView('display');
            
            // 30秒ごとにメッセージリストを再レンダリングし、有効期限切れのメッセージを自動削除するタイマー
            setInterval(renderMessages, 30000); // 30秒ごとに更新
        };
    </script>

</body>
</html>
