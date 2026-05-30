"use client";
import { useState, useRef } from "react";

// IMPORTANT: Replace this with your actual backend URL from your first Codespace
const API_URL = "https://opulent-train-x54459q5gx5rfpq6j-8000.app.github.dev/";

export default function PlayerApp() {
  const [view, setView] = useState("Home");
  const [searchQuery, setSearchQuery] = useState("");
  const [results, setResults] = useState([]);
  const [currentTrack, setCurrentTrack] = useState(null);
  const [streamUrl, setStreamUrl] = useState("");
  const [lyrics, setLyrics] = useState([]);
  const [currentTime, setCurrentTime] = useState(0);
  
  const audioRef = useRef(null);

  // --- API Handlers ---
  const handleSearch = async (e) => {
    e.preventDefault();
    if (!searchQuery) return;
    const res = await fetch(`${API_URL}/search/?s=${searchQuery}`);
    const data = await res.json();
    if (data?.data?.items) setResults(data.data.items);
  };

  const playTrack = async (track) => {
    const coverId = track.album?.cover;
    const coverUrl = coverId 
      ? `https://resources.tidal.com/images/${coverId.replace(/-/g, "/")}/640x640.jpg` 
      : "";
      
    setCurrentTrack({
      id: track.id,
      title: track.title,
      artist: track.artist?.name || "Unknown Artist",
      cover: coverUrl
    });

    setView("Lyrics");

    // Fetch Audio Manifest (Prioritizing FLAC and AAC)
    const streamRes = await fetch(`${API_URL}/trackManifests/?id=${track.id}&formats=FLAC&formats=AACLC&adaptive=false`);
    const streamData = await streamRes.json();
    if (streamData?.data?.data?.attributes?.uri) {
      setStreamUrl(streamData.data.data.attributes.uri);
    }

    // Fetch & Parse Lyrics
    const lyricsRes = await fetch(`${API_URL}/lyrics/?id=${track.id}`);
    const lyricsData = await lyricsRes.json();
    if (lyricsData?.data?.subtitles) {
      const parsed = parseLrc(lyricsData.data.subtitles);
      setLyrics(parsed);
    } else {
      setLyrics([]);
    }
  };

  const parseLrc = (lrcString) => {
    const lines = lrcString.split("\n");
    const parsed = [];
    lines.forEach(line => {
      const match = line.match(/\[(\d+):(\d+)\.(\d+)\](.*)/);
      if (match) {
        const minutes = parseInt(match[1]);
        const seconds = parseInt(match[2]);
        const text = match[4].trim();
        if (text) parsed.push({ time: minutes * 60 + seconds, text });
      }
    });
    return parsed;
  };

  // --- UI Layout ---
  return (
    <div className="flex h-screen w-full font-sans">
      
      {/* Sidebar Navigation */}
      <div className="glass-sidebar w-64 p-6 flex flex-col space-y-6 flex-shrink-0">
        <h1 className="text-2xl font-bold tracking-tight mb-4">🎵 Hi-Fi Horizon</h1>
        <button onClick={() => setView("Home")} className={`text-left px-4 py-2 rounded-lg transition-colors ${view === "Home" ? "bg-white/10" : "hover:bg-white/5"}`}>🏠 Home</button>
        <button onClick={() => setView("Search")} className={`text-left px-4 py-2 rounded-lg transition-colors ${view === "Search" ? "bg-white/10" : "hover:bg-white/5"}`}>🔍 Search</button>
        {currentTrack && (
          <button onClick={() => setView("Lyrics")} className={`text-left px-4 py-2 rounded-lg transition-colors ${view === "Lyrics" ? "bg-white/10" : "hover:bg-white/5"}`}>🎤 Now Playing</button>
        )}
      </div>

      {/* Main Content Viewport */}
      <div className="flex-1 overflow-y-auto p-10 relative">
        
        {/* View: Home */}
        {view === "Home" && (
          <div className="space-y-8 animate-fade-in">
            <h2 className="text-4xl font-bold">Welcome Back</h2>
            <div className="glass-panel rounded-2xl p-8 text-center max-w-2xl mx-auto mt-20">
              <h3 className="text-xl mb-4 text-white/80">Ready to listen?</h3>
              <p className="text-white/50 mb-6">Head over to the Search hub to discover high-fidelity tracks directly from your custom backend.</p>
              <button onClick={() => setView("Search")} className="bg-white text-black px-6 py-3 rounded-full font-bold hover:scale-105 transition-transform">Start Searching</button>
            </div>
          </div>
        )}

        {/* View: Search */}
        {view === "Search" && (
          <div className="space-y-8 animate-fade-in">
            <h2 className="text-4xl font-bold">Search Hub</h2>
            <form onSubmit={handleSearch} className="flex gap-4">
              <input 
                type="text" 
                value={searchQuery}
                onChange={(e) => setSearchQuery(e.target.value)}
                placeholder="Search tracks, artists, albums..." 
                className="glass-panel w-full max-w-xl px-6 py-4 rounded-full bg-transparent text-white placeholder-white/40 focus:outline-none focus:ring-2 focus:ring-white/30"
              />
              <button type="submit" className="bg-white text-black px-8 py-4 rounded-full font-bold hover:bg-gray-200 transition-colors">Search</button>
            </form>

            <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-6 mt-8">
              {results.map((item, idx) => {
                const img = item.album?.cover 
                  ? `https://resources.tidal.com/images/${item.album.cover.replace(/-/g, "/")}/320x320.jpg` 
                  : "https://via.placeholder.com/320";
                return (
                  <div key={idx} onClick={() => playTrack(item)} className="glass-panel rounded-xl p-4 cursor-pointer hover:-translate-y-1 transition-transform group">
                    <img src={img} alt="cover" className="w-full aspect-square object-cover rounded-lg mb-4 shadow-lg group-hover:shadow-2xl transition-shadow" />
                    <h4 className="font-bold text-lg truncate">{item.title}</h4>
                    <p className="text-sm text-white/60 truncate">{item.artist?.name}</p>
                  </div>
                );
              })}
            </div>
          </div>
        )}

        {/* View: Lyrics & Player */}
        {view === "Lyrics" && currentTrack && (
          <div className="flex flex-col lg:flex-row gap-12 h-full animate-fade-in">
            
            {/* Player Canvas */}
            <div className="w-full lg:w-1/2 flex flex-col items-center justify-center space-y-8">
              <img 
                src={currentTrack.cover} 
                alt="Artwork" 
                className="w-80 h-80 lg:w-96 lg:h-96 object-cover rounded-2xl shadow-[0_20px_50px_rgba(0,0,0,0.5)]"
              />
              <div className="text-center w-full max-w-md">
                <h2 className="text-3xl font-bold truncate">{currentTrack.title}</h2>
                <h3 className="text-xl text-white/60 mt-2 truncate">{currentTrack.artist}</h3>
                
                {streamUrl ? (
                  <audio 
                    ref={audioRef}
                    src={streamUrl} 
                    controls 
                    autoPlay
                    onTimeUpdate={() => setCurrentTime(audioRef.current?.currentTime || 0)}
                    className="w-full mt-8 rounded-full outline-none"
                  />
                ) : (
                  <p className="mt-8 animate-pulse text-white/50">Fetching high-fidelity stream...</p>
                )}
              </div>
            </div>

            {/* Apple Music Style Time-Synced Lyrics */}
            <div className="w-full lg:w-1/2 h-full overflow-y-auto pb-32 pt-10 px-4 scroll-smooth">
              {lyrics.length > 0 ? (
                <div className="space-y-6">
                  {lyrics.map((line, idx) => {
                    const isActive = currentTime >= line.time && (idx === lyrics.length - 1 || currentTime < lyrics[idx + 1].time);
                    return (
                      <p 
                        key={idx} 
                        className={`text-3xl lg:text-4xl font-bold transition-all duration-500 cursor-pointer hover:text-white 
                          ${isActive ? "text-white scale-105 origin-left blur-none drop-shadow-[0_0_15px_rgba(255,255,255,0.3)]" : "text-white/25 blur-[0.5px]"}`}
                        onClick={() => { if(audioRef.current) audioRef.current.currentTime = line.time; }}
                      >
                        {line.text}
                      </p>
                    );
                  })}
                </div>
              ) : (
                <div className="flex h-full items-center justify-center">
                  <p className="text-2xl font-bold text-white/30">Lyrics not available</p>
                </div>
              )}
            </div>
          </div>
        )}
      </div>
    </div>
  );
}
