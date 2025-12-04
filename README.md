<html ....>
<html lang="th">
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>School Weekly Planner - Cute Notebook Edition</title>
    
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    
    <!-- Google Fonts: Mali -->
    <link href="https://fonts.googleapis.com/css2?family=Mali:ital,wght@0,300;0,400;0,500;0,600;0,700;1,300&display=swap" rel="stylesheet">
    
    <style>
      body {
        font-family: 'Mali', cursive;
        /* Pastel Purple to Pastel Green Gradient */
        background: linear-gradient(135deg, #f3e8ff 0%, #dcfce7 100%);
        background-attachment: fixed;
        min-height: 100vh;
      }

      /* Explicit class for handwriting font if needed specifically */
      .font-handwriting {
        font-family: 'Mali', cursive;
      }
      
      /* Dot Grid Pattern */
      .bg-paper-pattern {
        background-color: rgba(255, 255, 255, 0.45);
        background-image: radial-gradient(#c084fc 1.5px, transparent 1.5px);
        background-size: 24px 24px;
      }
      
      /* Washi Tape Effect */
      .washi-tape {
        position: absolute;
        height: 24px;
        width: 80px;
        top: -10px;
        left: 50%;
        transform: translateX(-50%) rotate(-2deg);
        background-color: rgba(255, 255, 255, 0.4);
        border-left: 2px dashed rgba(0,0,0,0.1);
        border-right: 2px dashed rgba(0,0,0,0.1);
        box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        z-index: 10;
        opacity: 0.9;
      }
      .washi-tape-pink { background-color: #fbcfe8; }
      .washi-tape-green { background-color: #bbf7d0; }
      .washi-tape-purple { background-color: #e9d5ff; }

      /* Scrollbar hiding */
      .scrollbar-hide::-webkit-scrollbar { display: none; }
      .scrollbar-hide { -ms-overflow-style: none; scrollbar-width: none; }

      /* Animations */
      @keyframes float {
        0% { transform: translateY(0px); }
        50% { transform: translateY(-5px); }
        100% { transform: translateY(0px); }
      }
      .animate-float { animation: float 3s ease-in-out infinite; }

      @keyframes fade-in-up {
        from { opacity: 0; transform: translateY(10px); }
        to { opacity: 1; transform: translateY(0); }
      }
      .animate-fade-in-up { animation: fade-in-up 0.3s ease-out forwards; }
    </style>
<script>
  // Robust Shim for process.env to prevent ReferenceError in browser
  window.process = {
    env: {
      NODE_ENV: 'development',
      API_KEY: '' // Define empty key to prevent undefined access crash
    }
  };
</script>
<script type="importmap">
{
  "imports": {
    "react": "https://aistudiocdn.com/react@^19.2.1",
    "react-dom/client": "https://aistudiocdn.com/react-dom@^19.2.1/client",
    "react-dom/": "https://aistudiocdn.com/react-dom@^19.2.1/",
    "react/": "https://aistudiocdn.com/react@^19.2.1/",
    "@google/genai": "https://aistudiocdn.com/@google/genai@^1.31.0",
    "lucide-react": "https://aistudiocdn.com/lucide-react@^0.555.0"
  }
}
</script>
</head>
<body>
    <div id="root"></div>
    <script type="module" src="/index.tsx"></script>
</body>
</html>
