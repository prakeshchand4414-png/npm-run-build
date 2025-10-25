import React, { useState, useRef } from "react";

// AI-Generator-Frontend.jsx
// Single-file React component (Tailwind + shadcn style imports assumed) that provides
// a polished frontend for generating AI images, videos, web assets and audio (sound).
// This file is a full, copy-pasteable component you can drop into a React app.
// It uses Tailwind utility classes for styling. Replace /api/generate calls with your backend.

export default function AIStudio() {
  const [mode, setMode] = useState("image"); // image | video | audio | web
  const [prompt, setPrompt] = useState("");
  const [style, setStyle] = useState("photorealistic");
  const [resolution, setResolution] = useState("1024x1024");
  const [duration, setDuration] = useState(10); // for video/audio
  const [isGenerating, setIsGenerating] = useState(false);
  const [progressText, setProgressText] = useState("");
  const [resultURL, setResultURL] = useState(null);
  const [history, setHistory] = useState([]);
  const fileRef = useRef(null);

  // Preset styles (expandable)
  const presets = [
    { id: "photorealistic", label: "Photorealistic" },
    { id: "illustration", label: "Illustration" },
    { id: "cinematic", label: "Cinematic / Film" },
    { id: "minimal", label: "Minimal / UI" },
  ];

  // API contract (frontend assumes a single endpoint that accepts JSON and returns { url, type })
  // POST /api/generate {
  //   mode: 'image'|'video'|'audio'|'web',
  //   prompt: string,
  //   options: { style, resolution, duration }
  // }
  // Response: { url: string (public or temporary blob URL), mime: string }

  async function handleGenerate(e) {
    e?.preventDefault();
    if (!prompt.trim()) return alert("कृपया एक prompt डालें (example: 'sunset over mountains in photorealistic style')");

    setIsGenerating(true);
    setProgressText("Initializing...");
    setResultURL(null);

    // Build payload
    const payload = {
      mode,
      prompt,
      options: { style, resolution, duration },
    };

    try {
      // This call expects a backend that orchestrates model calls (Replicate, self-hosted, or other).
      // For a free-first approach you can implement server-side fallbacks to open-source models.
      const resp = await fetch(`/api/generate`, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(payload),
      });

      if (!resp.ok) {
        const txt = await resp.text();
        throw new Error(`Server error: ${txt}`);
      }

      // The response can be either a direct URL or a streamed progressive JSON.
      // Here we handle the common case: JSON with final URL.
      const data = await resp.json();
      if (data.progress) setProgressText(data.progress);
      if (data.url) {
        setResultURL(data.url);
        setHistory((h) => [{ mode, prompt, url: data.url, mime: data.mime || "unknown", ts: Date.now() }, ...h]);
        setProgressText("Ready");
      } else {
        setProgressText("Completed but no url returned");
      }
    } catch (err) {
      console.error(err);
      setProgressText(`Error: ${err.message}`);
      alert("जनरेट करते समय त्रुटि हुई: " + err.message);
    } finally {
      setIsGenerating(false);
    }
  }

  function handleDownload() {
    if (!resultURL) return;
    const a = document.createElement("a");
    a.href = resultURL;
    a.download = `ai-gen-${mode}-${Date.now()}`;
    document.body.appendChild(a);
    a.click();
    a.remove();
  }

  function clearHistory() {
    setHistory([]);
  }

  // Quick example prompts to help users
  const examples = {
    image: [
      "A vibrant poster of a jazz night, synthwave colors, 4k",
      "Close-up macro of a dewdrop on a leaf, photorealistic",
    ],
    video: [
      "10s cinematic drone shot over autumn forest, golden hour",
      "Logo reveal animation with particle effects, 6s",
    ],
    audio: ["Lo-fi study beat, 90 BPM, mellow piano", "Short ambient pad, 20s, evolving textures"],
    web: [
      "Landing page hero section: SaaS productivity app, headline + 3-feature cards",
      "Micro-interaction: onboarding modal flow in React + Tailwind",
    ],
  };

  return (
    <div className="min-h-screen bg-gray-50 p-6">
      <header className="max-w-6xl mx-auto mb-6">
        <div className="flex items-center justify-between">
          <div>
            <h1 className="text-2xl font-extrabold">AI Studio — Generate Images · Video · Audio · Web (Free-first)</h1>
            <p className="text-sm text-gray-600 mt-1">Prompt-based creator with presets, previews and downloads.</p>
          </div>
          <div className="text-right text-xs text-gray-500">
            <div>Free-friendly frontend — Connect a free or self-hosted backend to enable generation.</div>
          </div>
        </div>
      </header>

      <main className="max-w-6xl mx-auto grid grid-cols-1 md:grid-cols-4 gap-6">
        {/* Left: Controls */}
        <aside className="md:col-span-1 bg-white p-4 rounded-2xl shadow-sm">
          <nav className="space-y-2">
            {[
              { id: "image", label: "Image" },
              { id: "video", label: "Video" },
              { id: "audio", label: "Audio" },
              { id: "web", label: "Web / UI" },
            ].map((m) => (
              <button
                key={m.id}
                onClick={() => setMode(m.id)}
                className={`w-full text-left px-3 py-2 rounded-lg ${mode === m.id ? "bg-indigo-600 text-white" : "hover:bg-gray-100"}`}
              >
                {m.label}
              </button>
            ))}
          </nav>

          <div className="mt-4 border-t pt-4">
            <label className="block text-xs font-medium text-gray-700">Style</label>
            <select value={style} onChange={(e) => setStyle(e.target.value)} className="mt-1 block w-full rounded-md border px-2 py-1">
              {presets.map((p) => (
                <option key={p.id} value={p.id}>{p.label}</option>
              ))}
            </select>

            <label className="block text-xs font-medium text-gray-700 mt-3">Resolution / Size</label>
            <select value={resolution} onChange={(e) => setResolution(e.target.value)} className="mt-1 block w-full rounded-md border px-2 py-1">
              <option>512x512</option>
              <option>768x768</option>
              <option>1024x1024</option>
              <option>1920x1080</option>
              <option>1280x720</option>
            </select>

            {(mode === "video" || mode === "audio") && (
              <>
                <label className="block text-xs font-medium text-gray-700 mt-3">Duration (seconds)</label>
                <input value={duration} onChange={(e) => setDuration(Number(e.target.value))} type="number" min={1} max={60} className="mt-1 block w-full rounded-md border px-2 py-1" />
              </>
            )}

            <div className="mt-4 text-xs text-gray-500">
              <strong>Note:</strong> This UI is frontend-only. Connect a backend endpoint <code className="bg-gray-100 px-1 py-0.5 rounded">/api/generate</code> which polls or streams progress.
            </div>

            <div className="mt-4">
              <button onClick={handleGenerate} disabled={isGenerating} className={`w-full px-4 py-2 rounded-xl ${isGenerating ? "bg-gray-300" : "bg-indigo-600 text-white"}`}>
                {isGenerating ? "Generating..." : `Generate ${mode.charAt(0).toUpperCase() + mode.slice(1)}`}
              </button>
            </div>

            <div className="mt-3 text-sm text-gray-600">
              <div>Progress: {progressText || "idle"}</div>
              <div className="mt-2">Examples:</div>
              <div className="mt-1 space-y-1">
                {(examples[mode] || []).map((ex) => (
                  <button key={ex} onClick={() => setPrompt(ex)} className="block text-left text-xs hover:underline">• {ex}</button>
                ))}
              </div>
            </div>
          </div>
        </aside>

        {/* Right: Workspace */}
        <section className="md:col-span-3 bg-white p-6 rounded-2xl shadow-sm">
          <form onSubmit={handleGenerate}>
            <div className="flex items-start gap-4">
              <textarea
                value={prompt}
                onChange={(e) => setPrompt(e.target.value)}
                placeholder={`Describe your ${mode} (e.g. 'A modern landing page hero for a note app')`}
                className="flex-1 rounded-xl border p-4 h-36 resize-none"
              />
              <div className="w-40">
                <div className="text-sm font-medium">Quick actions</div>
                <button type="button" onClick={() => { setPrompt(''); setResultURL(null); }} className="mt-2 block w-full px-3 py-2 rounded-lg border">Clear</button>
                <button type="button" onClick={() => { if (fileRef.current) fileRef.current.click(); }} className="mt-2 block w-full px-3 py-2 rounded-lg border">Upload reference</button>
                <input ref={fileRef} type="file" accept="image/*,audio/*,video/*" className="hidden" />
                <button type="button" onClick={() => { setPrompt((p) => p + " --v 5"); }} className="mt-2 block w-full px-3 py-2 rounded-lg border">Add v5 style</button>
              </div>
            </div>
          </form>

          <div className="mt-6">
            <div className="text-sm font-semibold mb-2">Preview</div>
            <div className="border rounded-lg p-4 min-h-[200px] flex items-center justify-center">
              {resultURL ? (
                mode === "image" ? (
                  <img src={resultURL} alt="result" className="max-h-96 object-contain" />
                ) : mode === "audio" ? (
                  <audio controls src={resultURL} />
                ) : mode === "video" ? (
                  <video controls src={resultURL} className="max-h-96" />
                ) : (
                  <iframe title="web-result" src={resultURL} className="w-full h-80 border rounded" />
                )
              ) : (
                <div className="text-gray-400 text-center">कोई परिणाम नहीं — Generate दबाएँ</div>
              )}
            </div>

            <div className="mt-4 flex gap-2">
              <button onClick={handleDownload} disabled={!resultURL} className={`px-4 py-2 rounded-lg ${resultURL ? "bg-green-600 text-white" : "bg-gray-200"}`}>
                Download
              </button>
              <button onClick={() => { if (resultURL) navigator.clipboard.writeText(resultURL); }} disabled={!resultURL} className={`px-4 py-2 rounded-lg ${resultURL ? "bg-indigo-500 text-white" : "bg-gray-200"}`}>
                Copy URL
              </button>
              <button onClick={() => setResultURL(null)} className="px-4 py-2 rounded-lg bg-gray-100">Clear Preview</button>
            </div>

            <div className="mt-6">
              <div className="flex items-center justify-between">
                <div className="text-sm font-semibold">History</div>
                <button onClick={clearHistory} className="text-xs text-red-500">Clear</button>
              </div>
              <div className="mt-2 space-y-2 max-h-48 overflow-auto">
                {history.length === 0 && <div className="text-xs text-gray-400">No history yet.</div>}
                {history.map((h, i) => (
                  <div key={h.ts} className="flex items-center justify-between border rounded p-2">
                    <div>
                      <div className="text-xs font-medium">{h.mode.toUpperCase()} — {new Date(h.ts).toLocaleString()}</div>
                      <div className="text-xs text-gray-600 truncate max-w-xl">{h.prompt}</div>
                    </div>
                    <div className="flex items-center gap-2">
                      <a href={h.url} target="_blank" rel="noreferrer" className="text-xs underline">Open</a>
                      <button onClick={() => { setResultURL(h.url); }} className="text-xs px-2 py-1 rounded bg-gray-100">Preview</button>
                    </div>
                  </div>
                ))}
              </div>
            </div>

            <div className="mt-6 text-xs text-gray-500">
              <div className="font-semibold">Developer / Deployment notes (print-ready)</div>
              <ol className="list-decimal pl-5 mt-2 space-y-1">
                <li>
                  Backend endpoint: <code>/api/generate</code> — implement orchestration to call open-source models (Stable Diffusion, VAE, Whisper, Open-source video models) or hosted APIs.
                </li>
                <li>
                  Expectation: backend returns JSON { url, mime } or streams progress events via SSE / WebSocket.
                </li>
                <li>
                  Free-first strategy: use self-hosted models (local/gpu) or free-tier hosted endpoints and implement queueing + rate-limits for fairness.
                </li>
                <li>
                  Storage: store generated assets in object storage (S3 / DigitalOcean Spaces) with short-lived signed URLs for downloads.
                </li>
                <li>
                  Security: sanitize prompts, rate-limit users, and add content-moderation (filter illicit or NSFW content) before generating.
                </li>
                <li>
                  Deployment: this React frontend can be deployed on Vercel/Netlify and calls a serverless function (node, python) for /api/generate.
                </li>
              </ol>
            </div>

            <div className="mt-4 text-xs text-gray-400">
              © AI Studio — Frontend blueprint. Replace backend hooks & model keys to enable generation.
            </div>
          </div>
        </section>
      </main>
    </div>
  );
}

/*
  FULL PRINT / SPEC (human-readable summary to print or hand to a developer)

  1) Purpose:
     - A single-page frontend to let users generate AI images, short videos, audio clips and small web UI assets using prompts.

  2) Main UI elements:
     - Sidebar: choose mode (image/video/audio/web), style presets, resolution, duration.
     - Main prompt editor: large textarea, examples, quick actions.
     - Preview pane: image/audio/video/iframe preview + download/copy.
     - Generation progress and history.

  3) API Contract (frontend->backend):
     POST /api/generate
     Request JSON:
       {
         mode: 'image'|'video'|'audio'|'web',
         prompt: string,
         options: { style: string, resolution: string, duration?: number }
       }
     Success Response JSON:
       { url: string, mime?: string, progress?: string }
     Errors: return non-200 with explanatory body.

  4) Backend responsibilities (recommended):
     - Validate prompts and apply moderation/filtering.
     - Route requests to an appropriate model: e.g. Stable Diffusion for images, a video synthesis pipeline for short clips, a TTS / music model for audio, a UI code template generator for web.
     - Use job queue (RabbitMQ / Redis Queue) for long-running jobs.
     - Provide signed URLs for assets and cleanup after retention period.

  5) Free-first model options (developer note):
     - Images: Stable Diffusion (self-host or free-hosted community endpoints).
     - Audio: Open-source TTS / music models; fallback to short procedural generation; use Whisper for transcription if editing audio.
     - Video: Combine image model frames + interpolation (FFmpeg) or use newer open-source video models; many require GPU.
     - Web/UI: generate HTML/CSS/React snippets with prompts (use an LLM local or hosted free tier carefully) and render as an iframe on preview.

  6) Security & Moderation:
     - Always filter for extremist, sexual minors, violence, and personally identifiable images (face swap) policies.
     - Rate-limit anonymous users; offer login for higher quota.

  7) Deployment:
     - Frontend: static site on Vercel/Netlify.
     - Backend: serverless functions for small loads; dedicated GPU server for heavy model inference.

  8) Next steps for you:
     - Do you want me to provide the example serverless function (/api/generate) in Node.js that calls a free open-source model or Replicate? If yes, I can scaffold that too.

*/
