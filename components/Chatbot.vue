<template>
    <div id="virtual-assistant">
        <div id="chat-icon" @click="toggleChatBox">
            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" width="48" height="48" style="fill: #3498db;">
                <path d="M12 2C6.486 2 2 6.486 2 12c0 1.396.289 2.721.809 3.926L2 22l5.336-1.049A9.93 9.93 0 0 0 12 22c5.514 0 10-4.486 10-10S17.514 2 12 2zm0 18c-1.396 0-2.721-.289-3.926-.809L6 20l1.049-5.336A9.93 9.93 0 0 1 4 12c0-4.411 3.589-8 8-8s8 3.589 8 8-3.589 8-8 8z"/>
                <path d="M8 9h8v2H8zm0 4h5v2H8z"/>
            </svg>
        </div>
        <div id="chatbot">
            <div id="chatbox-con" class="text-black">
                <div id="chatbox">
                    <div id="chatbox-main"></div>
                    <p v-if="isDisconnected">Disconnected...</p>
                    <p v-else-if="isWaiting">Thinking...</p>
                </div>
                <div id="user-actions" :class="{disabled:isWaiting || isTyping}">
                    <input
                        class="bg-white"
                        type="text"
                        v-model="userInput"
                        placeholder="Ask me about Neil..."
                        @keydown.enter="sendMessage"
                        @input="messageHandler"
                    >
                    <button class="bg-[var(--color-primary)]" @click="sendMessage">Send</button>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                ws: null,
                chatbox: null,
                chatboxCon: null,
                userInput: "",
                isTyping: false,
                isWaiting: true,
                botMessageContainer: null,
                botMessageBuffer: "",
                isDisconnected: false,
            }
        },
        mounted() {
            this.chatboxCon = document.getElementById('chatbox');
            this.chatbox = document.getElementById('chatbox-main');
            this.ws = new WebSocket("wss://cardz.bounceme.net/bot/");
        
            this.ws.onmessage = async (event) => {
                this.isWaiting = false;
                const jsonResponse = JSON.parse(event.data);
        
                if (jsonResponse.hasOwnProperty("content")) {
                let message = jsonResponse["content"];
        
                if (!this.isTyping) {
                    this.botMessageContainer = document.createElement('div');
                    this.botMessageContainer.classList.add('bot-message');
        
                    let container = document.createElement('div');
                    container.classList.add('message-container');
                    container.appendChild(this.botMessageContainer);
                    this.chatbox.appendChild(container);
        
                    this.isTyping = true;
                }
        
                this.botMessageBuffer += message;
                this.botMessageContainer.innerHTML = this.formatMarkdown(this.botMessageBuffer);
        
                this.$nextTick(() => this.scrollToBottom());
                } else if (jsonResponse.hasOwnProperty("end") && jsonResponse.end === true) {
                    this.isTyping = false;
                    if (this.botMessageContainer) {
                        this.botMessageContainer.innerHTML = this.formatMarkdown(this.botMessageBuffer);
                        this.botMessageBuffer = "";
                    }
                    this.$nextTick(() => this.scrollToBottom());
                }
            }
        
            this.ws.onerror = (error) => {
                console.error("WebSocket error:", error);
            }
        
            this.ws.onopen = () => {
                this.ws.send(JSON.stringify([{ type: "text", text: "/profile" }]));
            }
        
            this.ws.onclose = () => {
                this.isDisconnected = true;
            }
        },
        methods: {
            toggleChatBox() {
                const chatbot = document.getElementById('chatbot');
                chatbot.classList.toggle('active');
            },
            messageHandler() {
                this.userInput = this.userInput;
            },
            appendMessage(message, className) {
                let div = document.createElement('div');
                div.innerHTML = this.formatMarkdown(message);
                div.classList.add(className);
        
                let container = document.createElement('div');
                container.classList.add('message-container');
                container.appendChild(div);
                this.chatbox.appendChild(container);
        
                this.$nextTick(() => this.scrollToBottom());
            },
            async sendMessage() {
                this.isWaiting = true;
        
                let rawMessage = this.userInput.trim();
                if (!rawMessage) return;
        
                if (rawMessage.toLowerCase() === "exit") {
                this.ws.close();
                return;
                }
        
                const imageRegex = /(https?:\/\/[^\s]+(?:\.jpg|\.jpeg|\.png|\.gif|\.bmp))/gi;
                let messageArray = [];
        
                let lastIndex = 0;
                let match;
                while ((match = imageRegex.exec(rawMessage)) !== null) {
                if (match.index > lastIndex) {
                    messageArray.push({ type: "text", text: rawMessage.slice(lastIndex, match.index) });
                }
                messageArray.push({ type: "image_url", image_url: { url: match[0] } });
                lastIndex = imageRegex.lastIndex;
                }
        
                if (lastIndex < rawMessage.length) {
                messageArray.push({ type: "text", text: rawMessage.slice(lastIndex) });
                }
        
                this.ws.send(JSON.stringify(messageArray));
                this.appendMessage(rawMessage, "user-message");
                this.userInput = "";
        
                this.$nextTick(() => this.scrollToBottom());
            },
            scrollToBottom() {
                this.chatboxCon.scrollTop = this.chatboxCon.scrollHeight;
            },
            formatMarkdown(text) {
                text = text.replace(/###\s*(.*?)(\n|$)/g, "<h3>$1</h3>");
                text = text.replace(/\*\*(.*?)\*\*/g, "<strong>$1</strong>");
                text = text.replace(/\*(.*?)\*/g, "<em>$1</em>");
                text = text.replace(/- (.*?)(\n|$)/g, "<li>$1</li>");
                text = text.replace(/(<li>.*<\/li>)/g, "<ul>$1</ul>");
                text = text.replace(/---/g, "<hr>");
                text = text.replace(/https?:\/\/\S+\.(jpg|jpeg|png|gif|bmp|webp)/g, function(match) {
                return `<img src="${match}" alt="image" />`;
                });
                return text;
            },
        },
        watch: {
            isWaiting() {
                this.$nextTick(() => this.scrollToBottom());
            }
        }
    }
</script>

<style>
    /* Floating Chat Icon */
    #chat-icon {
        position: fixed;
        bottom: 20px;
        right: 20px;
        cursor: pointer;
        z-index: 1000;
        transition: transform 0.2s ease;
    }
    #chat-icon:hover {
        transform: scale(1.1);
    }

    /* Chatbot Container */
    #chatbot {
        position: fixed;
        right: 20px;
        bottom: 80px;
        max-width: 100%;
        max-height: 100%;
        margin: 10px 0;
        display: none; /* Hidden by default */
    }
    #chatbot.active {
        display: block; /* Show when active */
    }

    #chatbox-con {
        width: 100%;
        max-width: 600px;
        position: relative;
        padding: 0 12px;
        margin: 0 auto;
        box-sizing: border-box;
    }
    #chatbox {
        width: 100%;
        height: 400px;
        max-height: 100%;
        overflow-y: auto;
        border: 1px solid #ccc;
        padding: 10px;
        background: #f9f9f9;
        font-family: Arial, sans-serif;
        border-radius: 5px;
        font-size: 14px;
        box-sizing: border-box;
    }

    .message-container {
        display: flex;
        text-align: left;
    }

    .user-message, .bot-message {
        padding: 8px !important;
        margin: 5px 0 !important;
        border-radius: 5px;
        display: inline-block;
        max-width: 80%;
        word-wrap: break-word;
        white-space: pre-wrap;
    }

    .user-message img {
        max-width: 100%;
    }

    .user-message {
        background: #007bff;
        color: white;
        margin-left: auto !important;
    }

    .bot-message {
        background: #e0e0e0;
        color: black;
        align-self: flex-start;
        padding: 10px;
        position: relative;
        margin-left: 50px !important;
        margin-top: 22px !important;
        min-height: 50px;
        box-sizing: border-box;
    }

    .bot-message::before {
        content: "";
        position: absolute;
        width: 50px;
        height: 100%;
        max-height: 132px;
        min-height: 50px;
        top: 0;
        right: calc(100% + 5px);
        overflow: hidden;
        margin-bottom: 5px;
        background: url("/images/bot.png") no-repeat center;
        background-size: cover;
        display: block;
        border-radius: 4px;
        background-position: center top;
        transition: .6s;
    }

    .bot-message::after {
        content: "Acey";
        position: absolute;
        bottom: calc(100% + 4px);
        left: 4px;
        color: #4a4a4a;
        font-size: 12px;
    }

    /* Headings */
    .bot-message h3 {
        font-size: 16px;
        margin: 8px 0;
        color: #333;
    }

    .bot-message img {
        width: 100%;
    }

    /* Bold and Italic */
    .bot-message strong {
        font-weight: bold;
        color: #222;
    }

    .bot-message em {
        font-style: italic;
        color: #555;
        margin: 12px 0;
        display: block;
    }

    /* Lists */
    .bot-message ul {
        padding-left: 18px;
        margin: 5px 0;
    }

    .bot-message ul li {
        margin-bottom: 5px;
    }

    /* Divider */
    .bot-message hr {
        border: 0;
        border-top: 1px solid #ccc;
        margin: 10px 0;
    }
    #user-actions {
        display: flex;
        width: 100%;
        margin-top: 8px;
    }
    #user-actions input {
        width: 100%;
        padding: 5px 10px;
        border: 1px solid #ccc;
        border-radius: 4px;
    }
    #user-actions button {
        width: 122px;
        height: 40px;
        color: #fff;
        border: 1px solid #ccc;
        border-radius: 4px;
        text-transform: uppercase;
        font-weight: 700;
        margin-left: 4px;
        cursor: pointer;
    }
    #user-actions.disabled {
        opacity: .5;
        pointer-events: none;
    }
</style>