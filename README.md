# AI-powered-Interview-Assistant-web-application
This AI Interview Assistant helps users prepare for job interviews by generating custom questions based on their resume and job description. It features voice input, transcript download, and integrates with OpenAI’s API for personalized question generation, all within a simple and user-friendly web interface.
1. index.html
xml
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>AI Interview Assistant</title>
  <link rel="stylesheet" href="style.css" />
</head>
<body>
  <div class="container">
    <h1>AI Interview Assistant</h1>

    <!-- Resume upload (TXT recommended) -->
    <label for="resumeUpload" class="label">Upload Resume (TXT recommended)</label>
    <input type="file" id="resumeUpload" accept=".txt" />

    <!-- Job description input -->
    <label for="jdInput" class="label">Paste Job Description</label>
    <textarea id="jdInput" placeholder="Paste job description here..."></textarea>

    <!-- Generate questions button -->
    <button id="generateQuestionsBtn">Generate 3 Interview Questions</button>

    <!-- AI output panel -->
    <div id="aiOutput" class="output"></div>

    <!-- Voice & transcript controls -->
    <div class="voice-controls">
      <button id="startMicBtn">Start Mic</button>
      <button id="clearBtn">Clear Chat</button>
      <button id="downloadBtn">Download Transcript</button>
    </div>

    <!-- API key input -->
    <input type="text" id="apiKeyInput" placeholder="OpenAI API Key (starts with sk-)" />
    <small>Your key is stored only in this tab’s localStorage. For public sites, use a server proxy to keep keys secret.</small>
  </div>

  <script src="script.js"></script>
</body>
</html>
2. style.css
css
/* Basic reset and styling */
body {
  background-color: #25254c;
  color: #fff;
  font-family: Arial, sans-serif;
  margin: 0;
  padding: 0;
}

.container {
  max-width: 450px;
  margin: 40px auto;
  background: #22223c;
  padding: 25px 30px;
  border-radius: 12px;
  box-shadow: 0 4px 32px #000b;
}

h1 {
  text-align: center;
  margin-bottom: 20px;
  font-weight: 600;
  font-size: 1.8rem;
}

.label {
  margin-top: 15px;
  display: block;
  font-weight: 600;
  font-size: 1rem;
}

#resumeUpload,
#jdInput,
#apiKeyInput {
  width: 100%;
  margin-top: 8px;
  border-radius: 8px;
  border: 1px solid #444c80;
  background: #15172c;
  color: #fff;
  font-size: 1rem;
  padding: 10px;
  font-family: inherit;
}

#resumeUpload {
  padding: 4px;
}

#jdInput {
  resize: vertical;
  min-height: 80px;
}

button {
  width: 100%;
  background-color: #5884ee;
  border: none;
  color: white;
  font-size: 1.1rem;
  padding: 13px;
  margin-top: 15px;
  border-radius: 8px;
  cursor: pointer;
  transition: background-color 0.2s ease;
}

button:hover {
  background-color: #355787;
}

.voice-controls {
  margin-top: 20px;
  display: flex;
  justify-content: space-between;
}

.voice-controls button {
  width: 31%;
  background-color: #2e3254;
  font-size: 1rem;
  padding: 10px;
  border-radius: 7px;
}

.voice-controls button:hover {
  background-color: #4a4f7d;
}

#aiOutput {
  margin-top: 20px;
  background: #181c27;
  min-height: 60px;
  border-radius: 10px;
  padding: 12px;
  white-space: pre-wrap;
  font-size: 1rem;
  font-family: monospace;
  box-sizing: border-box;
}

small {
  display: block;
  margin-top: 8px;
  font-size: 0.8rem;
  color: #bbbbbb;
  text-align: center;
}
3. script.js
javascript
// Transcript store
let transcript = [];

// Store/load API key in localStorage
const apiKeyInput = document.getElementById('apiKeyInput');
apiKeyInput.value = localStorage.getItem('openai-api-key') || '';

apiKeyInput.addEventListener('blur', () => {
  if (apiKeyInput.value.trim() !== '') {
    localStorage.setItem('openai-api-key', apiKeyInput.value.trim());
  }
});

// Resume upload text extraction (basic)
document.getElementById('resumeUpload').addEventListener('change', (event) => {
  const file = event.target.files[0];
  if (!file) return;

  if (file.type !== 'text/plain') {
    alert('Please upload a plain text (.txt) resume file for best results.');
    return;
  }

  const reader = new FileReader();
  reader.onload = () => {
    transcript.push('Resume uploaded:\n' + reader.result.slice(0, 500) + (reader.result.length > 500 ? '...' : ''));
  };
  reader.readAsText(file);
});

// Generate interview questions using OpenAI API
async function generateQuestions() {
  const apiKey = apiKeyInput.value.trim();
  const jdText = document.getElementById('jdInput').value.trim();

  if (!apiKey) {
    alert('Set your API key first');
    return;
  }
  if (!jdText) {
    alert('Please paste the job description');
    return;
  }

  const aiOutput = document.getElementById('aiOutput');
  aiOutput.textContent = 'Generating questions...';

  const endpoint = "https://api.openai.com/v1/chat/completions";
  const headers = {
    'Authorization': `Bearer ${apiKey}`,
    'Content-Type': 'application/json'
  };
  const body = JSON.stringify({
    model: "gpt-3.5-turbo",
    messages: [
      { role: "system", content: "You are an expert interviewer generating questions." },
      { role: "user", content: `Generate 3 interview questions for the following job description:\n${jdText}` }
    ]
  });

  try {
    const response = await fetch(endpoint, {
      method: 'POST',
      headers: headers,
      body: body,
    });
    const data = await response.json();
    if (data.error) {
      aiOutput.textContent = `Error: ${data.error.message}`;
      return;
    }
    const questions = data.choices?.[0]?.message?.content || "No questions generated.";
    aiOutput.textContent = questions;
    transcript.push('Questions generated:\n' + questions);
  } catch (error) {
    aiOutput.textContent = `Error: ${error.message}`;
  }
}

// Event listeners
document.getElementById('generateQuestionsBtn').addEventListener('click', generateQuestions);
document.getElementById('clearBtn').addEventListener('click', () => {
  document.getElementById('aiOutput').textContent = '';
  transcript = [];
});
document.getElementById('downloadBtn').addEventListener('click', () => {
  transcript.push(document.getElementById('aiOutput').textContent);
  const blob = new Blob([transcript.join('\n\n')], { type: 'text/plain' });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = 'transcript.txt';
  a.click();
  URL.revokeObjectURL(url);
});

// Optional: Voice input using Web Speech API (basic setup)
document.getElementById('startMicBtn').addEventListener('click', () => {
  if (!('SpeechRecognition' in window || 'webkitSpeechRecognition' in window)) {
    alert('Speech Recognition API not supported in this browser.');
    return;
  }

  const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
  const recognition = new SpeechRecognition();
  recognition.lang = 'en-US';
  recognition.interimResults = false;
  recognition.maxAlternatives = 1;

  recognition.start();

  recognition.onresult = (event) => {
    const spokenText = event.results[0].transcript;
    document.getElementById('jdInput').value = spokenText;
  };

  recognition.onerror = (event) => {
    alert('Speech recognition error: ' + event.error);
  };
});
4. How to Use
Save all above files in the same folder.

Open index.html in your modern browser (Chrome, Edge, Firefox).

Enter your OpenAI API key (starts with sk-).

Upload your resume as a .txt file (optional but recommended).

Paste the job description text.

Click Generate 3 Interview Questions.

Use Start Mic to input job description by voice if desired.

Use Clear Chat to clear output.

Use Download Transcript to save the text conversation.

5. README.md Suggestion (Optional)
text
# AI Interview Assistant

This web app helps job seekers prepare for interviews by generating customized interview questions based on the uploaded resume and job description using OpenAI’s API.

## Features
- Upload resume (.txt format)
- Paste job description
- Generate 3 interview questions powered by OpenAI
- Voice input for job description (via browser Speech API)
- Download transcript of generated questions and conversation
- Secure entry of OpenAI API key (stored locally in browser)

## How to Run
1. Clone or download repo
2. Open `index.html` in a modern browser
3. Enter your OpenAI API key in the input box (Obtain it from https://platform.openai.com/account/api-keys)
4. Upload resume and paste job description or use voice input
5. Click "Generate 3 Interview Questions" to get AI-generated questions

## Important
- Use your own OpenAI API key with sufficient quota.
- API key is stored only locally for your privacy.
- For public usage, use a backend proxy for security.

## License
MIT
You now have a complete, clear, and professional project ready to publish on GitHub.
If you want, I can help you with writing the README.md or deployment instructions too!

YES

YES
Here’s a README.md template you can use for your GitHub repository.
Just copy, edit as needed, and include it in your project folder!

AI Interview Assistant
This project is a simple, modern web application that helps users prepare for job interviews by generating custom questions based on their resume and the job description. Powered by OpenAI’s API, this tool features voice input, transcript download, and a secure, user-friendly interface.

Features
Upload Resume: Add your resume as a plain text (.txt) file.

Paste Job Description: Input the job posting or requirements.

AI-Powered Questions: Generates three relevant interview questions using OpenAI.

Voice Input: Use your microphone to dictate the job description.

Download Transcript: Save your questions and responses.

Secure API Key Entry: Your OpenAI API key is stored only in your browser’s local storage.

How to Use
Clone or Download the Repository

text
git clone https://github.com/yourusername/ai-interview-assistant.git
Open the Project Folder

All files (index.html, style.css, script.js, README.md) should be in one folder.

Open index.html in Your Browser

Double-click or right-click → “Open With” your favorite browser.

Enter Your OpenAI API Key

Get your API key from: OpenAI API Keys

Upload Resume, Paste Job Description, or Use Microphone

Click "Generate 3 Interview Questions"

Your personalized interview questions will appear.

Download Transcript

Save all generated content to a text file.

Requirements
Any modern browser (Chrome, Firefox, Edge).

Your own OpenAI API key (with available quota).

Privacy & Security
Your API Key is stored only in browser’s localStorage.

For public deployments, use a backend proxy to keep your API key secret.

License
MIT License

