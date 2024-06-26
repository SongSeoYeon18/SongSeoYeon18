<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>치매 자가진단 챗봇</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Nanum+Gothic:wght@700&display=swap');
        body {
            font-family: 'Nanum Gothic', sans-serif;
            margin: 20px;
            padding: 20px;
            max-width: 800px;
            margin: auto;
        }
        .chat-box {
            border: 1px solid #ccc;
            padding: 10px;
            margin-top: 20px;
            height: 500px;
            overflow-y: scroll;
            position: relative;
        }
        .chat-message {
            margin: 10px 0;
            display: flex;
            align-items: center;
        }
        .chat-message img {
            border-radius: 50%;
            width: 40px;
            height: 40px;
            margin-right: 10px;
        }
        .chat-message.user {
            justify-content: flex-end;
            text-align: right;
        }
        .chat-message.user img {
            order: 2;
            margin-right: 0;
            margin-left: 10px;
        }
        .chat-message .message-content {
            max-width: 70%;
            padding: 10px;
            border-radius: 10px;
        }
        .chat-message.bot .message-content {
            background-color: #e0f7fa;
        }
        .chat-message.user .message-content {
            background-color: #d3ecd4;
        }
        .result {
            font-weight: bold;
        }
        .centered-button {
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100%;
            width: 100%;
            top: 0;
            left: 0;
        }
        .centered-button button {
            font-family: 'Nanum Gothic', sans-serif;
            padding: 25px 50px;
            font-size: 28px;
            border-radius: 20px;
            border: none;
            background-color: #93ecf6;
            color: black;
            cursor: pointer;
        }
        .centered-button button:hover {
            background-color: #acf1f9;
        }
        .response-buttons {
            text-align: right;
            margin-top: 10px;
        }
        .response-buttons button {
            font-family: 'Nanum Gothic', sans-serif;
            padding: 10px 20px;
            margin-left: 10px;
            border-radius: 10px;
            border: none;
            background-color: #d3ecd4;
            color: black;
            cursor: pointer;
        }
        .response-buttons button:hover {
            background-color: #99d6a7;
        }
        @keyframes typing {
            from { width: 0; }
            to { width: 100%; }
        }
        .typing {
            display: inline-block;
            width: 0;
            overflow: hidden;
            white-space: nowrap;
            border-right: 3px solid;
            animation: typing 2s steps(20, end) forwards, blink-caret .75s step-end infinite;
        }
        @keyframes blink-caret {
            from, to { border-color: transparent; }
            50% { border-color: black; }
        }
    </style>
</head>
<body>

<h1>치매 자가진단 Chat Bot</h1>

<div class="chat-box" id="chatBox">
    <div class="chat-message bot">
        <img src="ChatBot.jpeg" alt="Bot">
        <div class="message-content">안녕하세요! 치매 자가진단을 도와드리겠습니다. 시작할까요?</div>
    </div>
    <div class="centered-button" id="startButton">
        <button onclick="startDiagnosis()">시작하기!</button>
    </div>
</div>

<div class="centered-button" id="gameButton" style="margin-top: 20px;">
    <button onclick="window.location.href='game.html'">치매 예방 게임하러 가기</button>
</div>

<script>
    const questions = [
        "1. 전화번호나 사람 이름을 잘 기억하지 못하나요?",
        "2. 어떤 일이 언제 일어났는지 잘 기억하지 못하나요?",
        "3. 며칠 전에 들었던 이야기를 잘 잊어버리나요?",
        "4. 반복되는 일상 생활에 변화가 생겼을 때 금방 적응하기가 힘든가요?",
        "5. 본인에게 중요한 사항을 잊을 때가 있나요? (예: 배우자 생일, 결혼기념일)",
        "6. 다른 사람들에게 같은 이야기를 반복할 때가 있나요?",
        "7. 전에 가본 장소를 잘 기억하지 못하나요?",
        "8. 어떤 일을 해놓고 했는지 안했는지 몰라 다시 확인한 적이 있나요?",
        "9. 여러가지 물건을 사러 갔다가 한 두가지를 빠뜨리기도 하나요?",
        "10. 이야기 도중 방금 자기가 무슨 이야기를 하고 있었는지 잊은 적이 있나요?",
        "11. 물건 이름이 금방 생각나지 않을때가 있나요?",
        "12. 계산 능력이 떨어졌다는 생각이 드나요?",
        "13. 약속을 해놓고 잊을 때가 있나요?",
        "14. 갈수록 말수가 적어지는 경향이 있나요?",
        "15. 영상 매체에 나오는 이야기를 따라가기 힘든가요?"
    ];

    const scores = {
        "매우 그렇다": 5,
        "조금 그렇다": 4,
        "보통이다": 3,
        "조금 그렇지 않다": 2,
        "매우 그렇지 않다": 1
    };

    let currentQuestion = 0;
    let totalScore = 0;

    function startDiagnosis() {
        document.getElementById('startButton').style.display = 'none';
        displayMessage(questions[currentQuestion], 'bot');
        showResponseButtons();
    }

    function handleResponse(response) {
        displayMessage(response, 'user');
        hideResponseButtons();

        totalScore += scores[response];
        currentQuestion++;
        if (currentQuestion < questions.length) {
            setTimeout(() => {
                displayMessage(questions[currentQuestion], 'bot');
                showResponseButtons();
            }, 500);
        } else {
            displayThinking();
        }
    }

    function displayMessage(message, sender) {
        const chatBox = document.getElementById('chatBox');
        const messageElement = document.createElement('div');
        messageElement.classList.add('chat-message', sender);

        const img = document.createElement('img');
        img.src = sender === 'bot' ? 'ChatBot.jpeg' : 'User.jpg';
        img.alt = sender === 'bot' ? 'Bot' : 'User';

        const messageContent = document.createElement('div');
        messageContent.classList.add('message-content');
        messageContent.textContent = message;

        messageElement.appendChild(img);
        messageElement.appendChild(messageContent);
        chatBox.appendChild(messageElement);
        chatBox.scrollTop = chatBox.scrollHeight;
    }

    function showResponseButtons() {
        const chatBox = document.getElementById('chatBox');
        const responseButtons = document.createElement('div');
        responseButtons.classList.add('response-buttons');
        responseButtons.id = 'responseButtons';
        responseButtons.innerHTML = `
            <button onclick="handleResponse('매우 그렇다')">매우 그렇다</button>
            <button onclick="handleResponse('조금 그렇다')">조금 그렇다</button>
            <button onclick="handleResponse('보통이다')">보통이다</button>
            <button onclick="handleResponse('조금 그렇지 않다')">조금 그렇지 않다</button>
            <button onclick="handleResponse('매우 그렇지 않다')">매우 그렇지 않다</button>
        `;
        chatBox.appendChild(responseButtons);
        chatBox.scrollTop = chatBox.scrollHeight;
    }

    function hideResponseButtons() {
        const responseButtons = document.getElementById('responseButtons');
        if (responseButtons) {
            responseButtons.remove();
        }
    }

    function displayThinking() {
        const message = "흠...";
        const duration = 2000;

        displayMessage(message, 'bot');

        setTimeout(() => {
            displayResult();
        }, duration);
    }

    function displayResult() {
        let resultText = '';
        if (totalScore >= 60) {
            resultText = '치매가 의심돼요! 전문의를 만나 상담하세요!';
        } else if (totalScore >= 30) {
            resultText = '치매 초기 증상이 의심돼요! 조기 검진을 권장드려요!';
        } else {
            resultText = '치매 가능성이 낮아요! 계속 건강을 유지해주세요~';
        }
        displayMessage(resultText, 'bot');
        displayPreventionButton();
    }

    function displayPreventionButton() {
        const chatBox = document.getElementById('chatBox');
        const preventionButton = document.createElement('div');
        preventionButton.classList.add('response-buttons');
        preventionButton.innerHTML = `
            <button onclick="window.location.href='prevention.html'">치매 예방법 보러가기</button>
        `;
        chatBox.appendChild(preventionButton);
        chatBox.scrollTop = chatBox.scrollHeight;
    }
</script>
</body>
</html>
