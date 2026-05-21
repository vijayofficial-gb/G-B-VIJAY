import React, { useState } from "react";
import { motion } from "framer-motion";

const RESUME_CONTEXT = `
GB Vijay - IT Support Engineer
4+ years experience in Desktop Support, SAP Business One, Networking.
Roles: Team Leader at Alstom, IT Support at Sigachi Industries.
Skills: SAP, Windows, Networking, Troubleshooting, Leadership.
`;

export default function Portfolio() {
  const [darkMode, setDarkMode] = useState(true);
  const [messages, setMessages] = useState([]);
  const [input, setInput] = useState("");
  const [loading, setLoading] = useState(false);
  const [language, setLanguage] = useState("en");
  const [interviewMode, setInterviewMode] = useState(false);

  const toggleTheme = () => setDarkMode(!darkMode);

  const speak = (text) => {
    const speech = new SpeechSynthesisUtterance(text);
    speech.lang = language === "ta" ? "ta-IN" : language === "te" ? "te-IN" : "en-IN";
    window.speechSynthesis.speak(speech);
  };

  const handleChat = async () => {
    if (!input) return;

    const newMessages = [...messages, { role: "user", text: input }];
    setMessages(newMessages);
    setLoading(true);

    try {
      const res = await fetch("http://localhost:5000/api/chat", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          message: input,
          context: RESUME_CONTEXT,
          history: newMessages,
          lang: language,
          interview: interviewMode
        }),
      });

      const data = await res.json();

      setMessages(prev => {
        const updated = [...prev, { role: "bot", text: data.reply }];
        speak(data.reply); // 🔊 AI speaks
        return updated;
      });
    } catch {
      setMessages(prev => [...prev, { role: "bot", text: "AI error" }]);
    }

    setInput("");
    setLoading(false);
  };

  const startVoice = () => {
    const recognition = new (window.SpeechRecognition || window.webkitSpeechRecognition)();
    recognition.lang = language === "ta" ? "ta-IN" : language === "te" ? "te-IN" : "en-IN";
    recognition.onresult = (e) => setInput(e.results[0][0].transcript);
    recognition.start();
  };

  return (
    <div className={darkMode ? "bg-gray-900 text-white min-h-screen" : "bg-white text-black min-h-screen"}>
      <header className="p-6 flex justify-between items-center border-b">
        <h1 className="text-2xl font-bold">GB Vijay</h1>
        <div className="flex gap-3">
          <button onClick={() => setInterviewMode(!interviewMode)} className="px-3 py-1 bg-purple-500 rounded text-white">
            {interviewMode ? "Exit Interview" : "Mock Interview"}
          </button>
          <button onClick={toggleTheme} className="px-3 py-1 bg-blue-500 rounded text-white">
            Theme
          </button>
        </div>
      </header>

      <section className="p-10 text-center">
        <motion.h2 initial={{ opacity: 0 }} animate={{ opacity: 1 }} className="text-4xl font-bold">
          🤖 AI Interview + Voice Assistant
        </motion.h2>
        <p className="mt-4">Talk with AI or take mock interview</p>
      </section>

      <section className="p-10 bg-blue-900">
        <div className="bg-white text-black p-4 rounded max-w-xl mx-auto">
          <div className="h-60 overflow-y-auto mb-3">
            {messages.map((m, i) => (
              <p key={i} className={m.role === "user" ? "text-right" : "text-left"}>
                {m.role === "user" ? "👤" : "🤖"} {m.text}
              </p>
            ))}
            {loading && <p>🤖 Thinking...</p>}
          </div>

          <div className="flex gap-2">
            <input value={input} onChange={(e) => setInput(e.target.value)} className="border p-2 w-full" />
            <button onClick={startVoice} className="bg-gray-700 text-white px-3">🎤</button>
            <button onClick={handleChat} className="bg-blue-500 text-white px-4 rounded">Send</button>
          </div>
        </div>
      </section>

      <footer className="p-6 text-center">© 2026 GB Vijay | AI Interview Portfolio</footer>
    </div>
  );
}

/* BACKEND ADDITION

if(interview){
system role:
"Act as IT interviewer. Ask technical & HR questions one by one. Evaluate answers."
}

✅ AI will:
- Ask interview questions
- Evaluate your answers
- Speak responses 🔊
*/

