<!DOCTYPE html>
<html lang="sk">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI Chatbot</title>
    <!-- Tailwind CSS CDN pre jednoduchý a čistý dizajn -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* Používame font Inter a zmeníme farbu pozadia */
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f3f4f6;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
        }

        /* Vlastné animácie */
        .loading-dots span {
            animation: bounce 1.4s infinite ease-in-out both;
        }
        .loading-dots span:nth-child(1) { animation-delay: -0.32s; }
        .loading-dots span:nth-child(2) { animation-delay: -0.16s; }

        @keyframes bounce {
            0%, 80%, 100% { transform: scale(0); }
            40% { transform: scale(1.0); }
        }
    </style>
</head>
<body class="bg-gray-100 flex items-center justify-center min-h-screen p-4">
    <div class="bg-white rounded-3xl shadow-2xl p-6 w-full max-w-lg mx-auto">
        <!-- Kontajner pre uvítanie a zadanie mena -->
        <div id="welcome-container" class="text-center">
            <h1 class="text-3xl font-bold text-gray-800 mb-4">Ahoj!</h1>
            <p class="text-lg text-gray-600 mb-6">Ak sa chceš so mnou porozprávať, najprv sa mi predstav.</p>
            <input type="text" id="user-name" placeholder="Tvoje meno" class="w-full px-4 py-3 rounded-xl border border-gray-300 focus:outline-none focus:ring-2 focus:ring-blue-500 transition-all duration-300">
            <button id="start-chat-btn" class="mt-4 w-full bg-blue-600 text-white font-bold py-3 px-4 rounded-xl hover:bg-blue-700 transition-colors duration-300 transform hover:scale-105 shadow-lg">
                Začať chat
            </button>
        </div>

        <!-- Hlavný chat kontajner (na začiatku skrytý) -->
        <div id="chat-container" class="hidden flex flex-col h-[500px]">
            <div id="chat-window" class="flex-grow overflow-y-auto p-4 space-y-4 bg-gray-50 rounded-xl mb-4 shadow-inner">
                <!-- Miesto pre správy -->
            </div>
            <div id="loading-indicator" class="flex items-center justify-start text-sm text-gray-500 my-2 hidden">
                <div class="loading-dots flex space-x-1">
                    <span class="w-2 h-2 bg-blue-500 rounded-full"></span>
                    <span class="w-2 h-2 bg-blue-500 rounded-full"></span>
                    <span class="w-2 h-2 bg-blue-500 rounded-full"></span>
                </div>
                <span class="ml-2">Píšem...</span>
            </div>
            <div class="flex">
                <input type="text" id="chat-input" placeholder="Napíš správu..." class="flex-grow px-4 py-3 rounded-l-xl border border-gray-300 focus:outline-none focus:ring-2 focus:ring-blue-500 transition-all duration-300">
                <button id="send-chat-btn" class="bg-blue-600 text-white font-bold py-3 px-6 rounded-r-xl hover:bg-blue-700 transition-colors duration-300 transform hover:scale-105 shadow-lg">
                    Odoslať
                </button>
            </div>
        </div>
    </div>

    <script>
        // Deklarácia Firebase a AI API konfigurácií
        // V Canvas prostredí budú tieto premenné dostupné globálne.
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        const apiKey = "";
        
        let userName = '';
        let chatHistory = [];

        // Referencie na DOM elementy
        const welcomeContainer = document.getElementById('welcome-container');
        const chatContainer = document.getElementById('chat-container');
        const userNameInput = document.getElementById('user-name');
        const startChatBtn = document.getElementById('start-chat-btn');
        const chatWindow = document.getElementById('chat-window');
        const chatInput = document.getElementById('chat-input');
        const sendChatBtn = document.getElementById('send-chat-btn');
        const loadingIndicator = document.getElementById('loading-indicator');

        // Funkcia na pridanie správy do chatovacieho okna
        function addMessage(sender, message) {
            const messageDiv = document.createElement('div');
            messageDiv.classList.add('flex', 'flex-col');
            
            if (sender === 'user') {
                messageDiv.classList.add('items-end');
                messageDiv.innerHTML = `
                    <div class="bg-blue-500 text-white rounded-t-xl rounded-bl-xl p-3 max-w-xs md:max-w-md shadow-lg">
                        ${message}
                    </div>
                `;
            } else {
                messageDiv.classList.add('items-start');
                messageDiv.innerHTML = `
                    <div class="bg-gray-200 text-gray-800 rounded-t-xl rounded-br-xl p-3 max-w-xs md:max-w-md shadow-lg">
                        ${message}
                    </div>
                `;
            }
            chatWindow.appendChild(messageDiv);
            chatWindow.scrollTop = chatWindow.scrollHeight; // Posunie okno na spodok
        }

        // Funkcia na spracovanie odoslaného mena
        function handleNameSubmit() {
            userName = userNameInput.value.trim();
            if (userName) {
                welcomeContainer.classList.add('hidden');
                chatContainer.classList.remove('hidden');
                addMessage('assistant', `Ahoj, ${userName}! Som tu pre teba. Na čo sa ma chceš opýtať?`);
            }
        }

        // Asynchrónna funkcia na odoslanie správy AI
        async function handleUserMessage() {
            const userMessage = chatInput.value.trim();
            if (userMessage) {
                addMessage('user', userMessage);
                chatInput.value = '';
                loadingIndicator.classList.remove('hidden');

                // Konštruujeme kontext pre AI
                const prompt = `Moje meno je ${userName}. Odpovedz mi na nasledujúcu otázku, ale odpovedaj stručne a priateľsky ako bežný človek v slovenskom jazyku: "${userMessage}"`;
                
                chatHistory.push({ role: "user", parts: [{ text: prompt }] });

                const payload = {
                    contents: chatHistory
                };

                const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-05-20:generateContent?key=${apiKey}`;

                let retryCount = 0;
                const maxRetries = 5;
                const baseDelay = 1000; // 1 sekunda

                while (retryCount < maxRetries) {
                    try {
                        const response = await fetch(apiUrl, {
                            method: 'POST',
                            headers: { 'Content-Type': 'application/json' },
                            body: JSON.stringify(payload)
                        });

                        if (response.status === 429) {
                            // Exponential backoff
                            const delay = Math.pow(2, retryCount) * baseDelay + Math.random() * baseDelay;
                            retryCount++;
                            await new Promise(resolve => setTimeout(resolve, delay));
                            continue;
                        }

                        const result = await response.json();
                        if (result.candidates && result.candidates.length > 0 &&
                            result.candidates[0].content && result.candidates[0].content.parts &&
                            result.candidates[0].content.parts.length > 0) {
                            const aiResponse = result.candidates[0].content.parts[0].text;
                            addMessage('assistant', aiResponse);
                            chatHistory.push({ role: "model", parts: [{ text: aiResponse }] });
                        } else {
                            addMessage('assistant', 'Prepáč, momentálne neviem odpovedať.');
                        }
                        break;
                    } catch (error) {
                        console.error("Chyba pri volaní API:", error);
                        addMessage('assistant', 'Prepáč, nastala chyba pri spracovaní tvojej požiadavky.');
                        break;
                    }
                }
                loadingIndicator.classList.add('hidden');
            }
        }

        // Pridanie poslucháčov udalostí
        startChatBtn.addEventListener('click', handleNameSubmit);
        userNameInput.addEventListener('keydown', (e) => {
            if (e.key === 'Enter') handleNameSubmit();
        });

        sendChatBtn.addEventListener('click', handleUserMessage);
        chatInput.addEventListener('keydown', (e) => {
            if (e.key === 'Enter') handleUserMessage();
        });
    </script>
</body>
</html>
