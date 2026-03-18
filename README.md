<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Planning Prévisionnel — Projet En Détente</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <script src="https://unpkg.com/lucide@latest/dist/umd/lucide.js"></script>
  <script src="https://unpkg.com/react@18/umd/react.development.js" crossorigin></script>
  <script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js" crossorigin></script>
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800&display=swap');
    * { box-sizing: border-box; }
    body { font-family: 'Inter', sans-serif; }

    .gantt-scroll::-webkit-scrollbar { height: 6px; }
    .gantt-scroll::-webkit-scrollbar-track { background: #f1f5f9; border-radius: 99px; }
    .gantt-scroll::-webkit-scrollbar-thumb { background: #94a3b8; border-radius: 99px; }
    .gantt-scroll::-webkit-scrollbar-thumb:hover { background: #64748b; }

    .bar-wrap:hover .bar-tooltip { opacity: 1; transform: translateX(-50%) translateY(0); }
    .bar-tooltip {
      opacity: 0;
      transform: translateX(-50%) translateY(4px);
      transition: opacity .15s ease, transform .15s ease;
      pointer-events: none;
    }

    .phase-row:hover .phase-label-col { background: #f8fafc !important; }
    .task-row:hover .task-label-col   { background: #f8fafc !important; }

    .phase-card { transition: box-shadow .2s ease; }
    .phase-card:hover { box-shadow: 0 4px 20px rgba(0,0,0,0.08); }

    @keyframes pulse-dot { 0%,100%{opacity:1} 50%{opacity:.4} }
    .pulse-dot { animation: pulse-dot 2s ease-in-out infinite; }

    .view-btn { transition: background .15s, color .15s; }
    .view-btn.active { background: #001f3f; color: #fff; }
    .view-btn:not(.active) { background: transparent; color: #64748b; }
    .view-btn:not(.active):hover { background: #e2e8f0; color: #1e293b; }
  </style>
</head>
<body class="bg-slate-100">
  <div id="root"></div>

  <script type="text/babel">
    const { useState, useEffect, useRef } = React;

    /* ── Lucide Icon wrapper ── */
    const Icon = ({ name, size = 16, className = "", color }) => {
      const ref = useRef(null);
      useEffect(() => {
        if (!ref.current || !lucide.icons[name]) return;
        ref.current.innerHTML = "";
        const svg = lucide.createElement(lucide.icons[name], {
          width: size, height: size, ...(color ? { color } : {})
        });
        ref.current.appendChild(svg);
      }, [name, size, color]);
      return <span ref={ref} className={`inline-flex items-center shrink-0 ${className}`} />;
    };

    /* ── DATA ── */
    const months = [
      "Septembre","Octobre","Novembre","Décembre",
      "Janvier","Février","Mars","Avril","Mai","Juin"
    ];

    const phases = [
      {
        id:1, num:"01",
        title:"Étude et cadrage",
        fullTitle:"Phase 1 — Étude et cadrage",
        shortLabel:"Ph.1 — Étude",
        hex:"#1e293b",
        startMonth:0, durationMonths:2,
        tasks:[
          {name:"Coordination & Cahier des charges",    owner:"M. Shimi",    start:0,   dur:1.5},
          {name:"Étude de marché & Questionnaires",     owner:"S. Bajjou",   start:0.2, dur:1.2},
          {name:"Analyse environnementale & Matériaux", owner:"T. Sapan",    start:0.5, dur:1  },
          {name:"Architecture système & Composants",    owner:"B. Bouchama", start:1,   dur:1  },
          {name:"Veille technique & Fournisseurs",      owner:"C. Sarr",     start:1.2, dur:0.8},
        ]
      },
      {
        id:2, num:"02",
        title:"Conception technique",
        fullTitle:"Phase 2 — Conception technique",
        shortLabel:"Ph.2 — Conception",
        hex:"#1e3a5f",
        startMonth:2, durationMonths:2,
        tasks:[
          {name:"Conception électronique & Capteurs",   owner:"B. Bouchama", start:2,   dur:1.5},
          {name:"Conception mécanique & Ergonomie",     owner:"T. Sapan",    start:2.2, dur:1.5},
          {name:"Assistance prototypage & Simulation",  owner:"C. Sarr",     start:2.5, dur:1.2},
          {name:"Validation technique & Planning",      owner:"M. Shimi",    start:3.5, dur:0.5},
        ]
      },
      {
        id:3, num:"03",
        title:"Prototypage",
        fullTitle:"Phase 3 — Prototypage",
        shortLabel:"Ph.3 — Proto",
        hex:"#1e40af",
        startMonth:4, durationMonths:2,
        tasks:[
          {name:"Carte électronique & Firmware",        owner:"B. Bouchama", start:4,   dur:1.8},
          {name:"Fabrication coque & Assemblage",       owner:"T. Sapan",    start:4.5, dur:1.2},
          {name:"Tests de pression & Validation",       owner:"C. Sarr",     start:5.5, dur:0.5},
        ]
      },
      {
        id:4, num:"04",
        title:"Tests et amélioration",
        fullTitle:"Phase 4 — Tests et amélioration",
        shortLabel:"Ph.4 — Tests",
        hex:"#1d4ed8",
        startMonth:6, durationMonths:2,
        tasks:[
          {name:"Essais techniques",                    owner:"C. Sarr",     start:6,   dur:1  },
          {name:"Optimisation éco-conception",          owner:"T. Sapan",    start:6.5, dur:1  },
          {name:"Analyse résultats & Améliorations",    owner:"M. Shimi",    start:7,   dur:1  },
        ]
      },
      {
        id:5, num:"05",
        title:"Marketing et stratégie",
        fullTitle:"Phase 5 — Marketing et stratégie",
        shortLabel:"Ph.5",
        hex:"#2563eb",
        startMonth:8, durationMonths:1,
        tasks:[
          {name:"Site internet & Supports marketing",   owner:"S. Bajjou",   start:8,   dur:1  },
          {name:"Stratégie & Partenaires",              owner:"M. Shimi",    start:8.2, dur:0.8},
        ]
      },
      {
        id:6, num:"06",
        title:"Préparation lancement",
        fullTitle:"Phase 6 — Préparation lancement",
        shortLabel:"Ph.6",
        hex:"#312e81",
        startMonth:9, durationMonths:1,
        tasks:[
          {name:"Validation finale & Business Plan",    owner:"Équipe",      start:9,   dur:0.5},
          {name:"Rapport final & Présentation",         owner:"Équipe",      start:9.5, dur:0.5},
        ]
      },
    ];

    const teamMembers = [
      {name:"M. Shimi",    role:"Chef de projet"},
      {name:"B. Bouchama", role:"Technique"      },
      {name:"T. Sapan",    role:"Éco-conception" },
      {name:"S. Bajjou",   role:"Marketing"      },
      {name:"C. Sarr",     role:"Support"        },
    ];

    /* ── CONSTANTS ── */
    const UNIT  = 110;   // px per month
    const LABEL = 280;   // px label column
    const EXTRA = 60;    // right-side buffer so Phase 6 never clips

    /* ─────────────────────────
       GANTT VIEW
    ───────────────────────── */
    const GanttView = ({ expandedPhases, togglePhase }) => (
      <div className="gantt-scroll overflow-x-auto">
        <div style={{ minWidth: UNIT * months.length + LABEL + EXTRA }} className="relative">

          {/* Timeline header */}
          <div className="flex border-b border-slate-200 bg-slate-50 sticky top-0 z-30">
            <div style={{ width: LABEL, minWidth: LABEL }}
                 className="p-4 font-semibold text-slate-400 text-xs tracking-widest uppercase
                             border-r border-slate-200 sticky left-0 bg-slate-50 z-40">
              PHASES &amp; MISSIONS
            </div>
            {months.map((m, i) => (
              <div key={i} style={{ width: UNIT, minWidth: UNIT }}
                   className="py-3 text-center border-r border-slate-200 last:border-r-0">
                <span className="text-[10px] font-semibold text-slate-400 block uppercase tracking-wider leading-none mb-0.5">
                  {i < 4 ? "2025" : "2026"}
                </span>
                <span className="text-xs font-bold text-slate-700">{m}</span>
              </div>
            ))}
            <div style={{ width: EXTRA }} />
          </div>

          {/* Grid */}
          <div className="absolute top-0 bottom-0 pointer-events-none flex"
               style={{ left: LABEL, right: 0, zIndex: 0 }}>
            {months.map((_, i) => (
              <div key={i} style={{ width: UNIT, minWidth: UNIT }}
                   className={`h-full border-r border-slate-100 ${i % 2 === 0 ? "bg-white" : "bg-slate-50/60"}`} />
            ))}
          </div>

          {/* Phases + Tasks */}
          <div className="relative" style={{ zIndex: 10 }}>
            {phases.map((phase) => (
              <div key={phase.id} className="border-b border-slate-100 last:border-0">

                {/* Phase row */}
                <div className="phase-row flex items-center cursor-pointer select-none"
                     style={{ minHeight: 52 }}
                     onClick={() => togglePhase(phase.id)}>
                  <div style={{ width: LABEL, minWidth: LABEL }}
                       className="phase-label-col h-full px-4 py-3 border-r border-slate-200
                                  sticky left-0 bg-white z-20 flex items-center gap-2.5 transition-colors">
                    <div className="w-5 h-5 rounded flex items-center justify-center shrink-0"
                         style={{ background: phase.hex + "22" }}>
                      {expandedPhases[phase.id]
                        ? <Icon name="ChevronDown"  size={12} color={phase.hex} />
                        : <Icon name="ChevronRight" size={12} color={phase.hex} />}
                    </div>
                    <div className="min-w-0">
                      <span className="font-bold text-xs text-slate-800 block leading-tight truncate">
                        {phase.fullTitle}
                      </span>
                      <span className="text-[10px] text-slate-400">
                        {phase.tasks.length} mission{phase.tasks.length > 1 ? "s" : ""}
                      </span>
                    </div>
                  </div>

                  {/* Phase bar */}
                  <div className="flex-1 py-3 px-2 flex items-center">
                    {/* Wrapper positionned at bar start, allows label to overflow right */}
                    <div style={{
                           marginLeft: phase.startMonth * UNIT,
                           display: "flex",
                           alignItems: "center",
                           gap: 8,
                         }}>
                      {/* The coloured bar — always clipped, short label for narrow bars */}
                      <div style={{
                             width: phase.durationMonths * UNIT,
                             height: 28,
                             borderRadius: 8,
                             background: `linear-gradient(135deg, ${phase.hex} 0%, ${phase.hex}cc 100%)`,
                             boxShadow: `0 2px 10px ${phase.hex}44`,
                             position: "relative",
                             overflow: "hidden",
                             flexShrink: 0,
                             display: "flex",
                             alignItems: "center",
                           }}>
                        {/* Shine */}
                        <div style={{
                               position:"absolute", inset:0,
                               background:"linear-gradient(180deg,rgba(255,255,255,0.16) 0%,transparent 55%)",
                               borderRadius:8,
                               pointerEvents:"none",
                             }} />
                        {/* Label : full title for wide bars, short for 1-month bars */}
                        <span style={{
                                position:"relative", zIndex:1,
                                padding:"0 10px",
                                fontSize:10, fontWeight:700,
                                color:"#fff",
                                letterSpacing:"0.06em",
                                whiteSpace:"nowrap",
                                textTransform:"uppercase",
                                overflow:"hidden",
                                textOverflow:"ellipsis",
                                display:"block",
                                maxWidth:"100%",
                              }}>
                          {`PHASE ${phase.id}`}
                        </span>
                      </div>
                    </div>
                  </div>
                </div>

                {/* Task rows */}
                {expandedPhases[phase.id] && phase.tasks.map((task, ti) => (
                  <div key={ti} className="task-row flex items-center" style={{ minHeight: 40 }}>
                    <div style={{ width: LABEL, minWidth: LABEL }}
                         className="task-label-col h-full px-4 py-2 pl-10 border-r border-slate-100
                                    sticky left-0 bg-white z-20 flex flex-col justify-center transition-colors">
                      <span className="text-[11px] font-medium text-slate-700 leading-snug">{task.name}</span>
                      <span className="text-[10px] font-medium mt-0.5 flex items-center gap-1"
                            style={{ color: phase.hex }}>
                        <Icon name="User" size={9} color={phase.hex} /> {task.owner}
                      </span>
                    </div>
                    <div className="flex-1 h-full py-2 px-2 flex items-center">
                      <div className="bar-wrap relative"
                           style={{ marginLeft: task.start * UNIT, width: task.dur * UNIT }}>
                        {/* Track */}
                        <div style={{
                               height: 10, borderRadius: 99,
                               background: "#e2e8f0",
                               position: "relative", overflow: "hidden",
                             }}>
                          <div style={{
                                 position:"absolute", inset:0, borderRadius:99,
                                 background:`linear-gradient(90deg,${phase.hex}99,${phase.hex}44)`,
                               }} />
                        </div>
                        {/* Tooltip */}
                        <div className="bar-tooltip absolute bg-slate-800 text-white text-[10px] font-medium
                                        px-2.5 py-1.5 rounded-md shadow-lg whitespace-nowrap z-50"
                             style={{ bottom:"calc(100% + 8px)", left:"50%", transform:"translateX(-50%) translateY(4px)" }}>
                          <span style={{ color:"#93c5fd" }}>●</span>&nbsp;
                          {task.owner} · {task.dur} mois
                          <div style={{
                            position:"absolute", bottom:-4, left:"50%",
                            transform:"translateX(-50%)",
                            width:8, height:8,
                            background:"#1e293b",
                            clipPath:"polygon(0 0,100% 0,50% 100%)",
                          }} />
                        </div>
                      </div>
                    </div>
                  </div>
                ))}
              </div>
            ))}
          </div>
        </div>
      </div>
    );

    /* ─────────────────────────
       CARD VIEW (mobile)
    ───────────────────────── */
    const CardView = ({ expandedPhases, togglePhase }) => (
      <div className="p-4 space-y-3">
        {phases.map((phase) => (
          <div key={phase.id} className="phase-card bg-white rounded-xl border border-slate-100 overflow-hidden shadow-sm">
            <div className="flex items-center gap-3 p-4 cursor-pointer select-none"
                 onClick={() => togglePhase(phase.id)}>
              <div className="w-9 h-9 rounded-lg flex items-center justify-center shrink-0 text-white text-sm font-bold"
                   style={{ background: phase.hex }}>
                {phase.num}
              </div>
              <div className="flex-1 min-w-0">
                <span className="font-bold text-sm text-slate-800 block truncate">{phase.fullTitle}</span>
                <span className="text-[10px] text-slate-400">
                  {phase.tasks.length} missions · Mois {phase.startMonth + 1}–{phase.startMonth + phase.durationMonths}
                </span>
              </div>
              <Icon name={expandedPhases[phase.id] ? "ChevronUp" : "ChevronDown"} size={14} color="#94a3b8" />
            </div>

            {expandedPhases[phase.id] && (
              <div className="border-t border-slate-50 divide-y divide-slate-50">
                {phase.tasks.map((task, ti) => (
                  <div key={ti} className="flex items-start gap-3 px-4 py-3">
                    <div className="w-1.5 h-1.5 rounded-full mt-1.5 shrink-0"
                         style={{ background: phase.hex }} />
                    <div>
                      <span className="text-xs font-medium text-slate-700 block">{task.name}</span>
                      <div className="flex items-center gap-3 mt-1">
                        <span className="text-[10px] font-medium flex items-center gap-1" style={{ color: phase.hex }}>
                          <Icon name="User" size={9} color={phase.hex} /> {task.owner}
                        </span>
                        <span className="text-[10px] text-slate-400 flex items-center gap-1">
                          <Icon name="Clock" size={9} color="#94a3b8" /> {task.dur} mois
                        </span>
                      </div>
                    </div>
                  </div>
                ))}
              </div>
            )}
          </div>
        ))}
      </div>
    );

    /* ─────────────────────────
       APP
    ───────────────────────── */
    const App = () => {
      const [expandedPhases, setExpandedPhases] = useState(
        Object.fromEntries(phases.map(p => [p.id, true]))
      );
      const [view, setView] = useState(window.innerWidth < 768 ? "cards" : "gantt");

      useEffect(() => {
        const onResize = () => {
          setView(w => {
            const mobile = window.innerWidth < 768;
            if (mobile && w === "gantt") return "cards";
            if (!mobile && w === "cards") return "gantt";
            return w;
          });
        };
        window.addEventListener("resize", onResize);
        return () => window.removeEventListener("resize", onResize);
      }, []);

      const togglePhase  = id => setExpandedPhases(p => ({ ...p, [id]: !p[id] }));
      const expandAll    = () => setExpandedPhases(Object.fromEntries(phases.map(p => [p.id, true])));
      const collapseAll  = () => setExpandedPhases(Object.fromEntries(phases.map(p => [p.id, false])));

      const totalTasks = phases.reduce((a, p) => a + p.tasks.length, 0);

      return (
        <div className="min-h-screen bg-slate-100 p-3 md:p-8">
          <div className="max-w-7xl mx-auto bg-white rounded-2xl shadow-xl overflow-hidden border border-slate-200">

            {/* ── HEADER ── */}
            <div className="relative overflow-hidden px-5 md:px-8 py-6 md:py-8 text-white"
                 style={{ background: "linear-gradient(135deg, #001f3f 0%, #003366 60%, #0a4a8f 100%)" }}>
              <div className="absolute -top-10 -right-10 w-48 h-48 rounded-full opacity-10 pointer-events-none"
                   style={{ background: "radial-gradient(circle, #60a5fa, transparent)" }} />
              <div className="absolute -bottom-8 left-10 w-32 h-32 rounded-full opacity-5 pointer-events-none"
                   style={{ background: "radial-gradient(circle, #93c5fd, transparent)" }} />

              <div className="relative flex flex-col sm:flex-row justify-between items-start sm:items-center gap-4">
                <div>
                  <div className="flex items-center gap-2 mb-2">
                    <span className="text-[10px] font-bold uppercase tracking-widest text-blue-300
                                     bg-blue-500/20 px-2.5 py-1 rounded-full border border-blue-400/30">
                      Projet Interne
                    </span>
                  </div>
                  <h1 className="text-xl md:text-2xl font-extrabold tracking-tight leading-tight">
                    Planning Prévisionnel
                    <span className="text-blue-300"> — En Détente</span>
                  </h1>
                  <p className="text-blue-200/70 text-xs mt-1.5 flex items-center gap-1.5">
                    <Icon name="Calendar" size={12} className="opacity-70" />
                    Calendrier Opérationnel 2025–2026
                  </p>
                </div>

                <div className="flex flex-wrap gap-2 shrink-0">
                  <div className="flex items-center gap-2 bg-white/10 backdrop-blur-sm
                                  px-3 py-2 rounded-lg border border-white/15 text-xs">
                    <span className="pulse-dot w-2 h-2 rounded-full bg-emerald-400 shrink-0" />
                    <div>
                      <span className="block text-white/50 text-[9px] uppercase font-bold leading-none mb-0.5">Statut</span>
                      <span className="font-semibold text-emerald-300 leading-none">En Planification</span>
                    </div>
                  </div>
                  <div className="bg-white/10 backdrop-blur-sm px-3 py-2 rounded-lg border border-white/15 text-xs">
                    <span className="block text-white/50 text-[9px] uppercase font-bold leading-none mb-0.5">Durée</span>
                    <span className="font-semibold leading-none">10 Mois</span>
                  </div>
                  <div className="bg-white/10 backdrop-blur-sm px-3 py-2 rounded-lg border border-white/15 text-xs">
                    <span className="block text-white/50 text-[9px] uppercase font-bold leading-none mb-0.5">Phases</span>
                    <span className="font-semibold leading-none">6 Phases</span>
                  </div>
                </div>
              </div>
            </div>

            {/* ── TOOLBAR ── */}
            <div className="flex flex-wrap items-center justify-between gap-3 px-4 md:px-6 py-3
                            bg-slate-50 border-b border-slate-200">
              {/* View toggle */}
              <div className="flex items-center gap-1 bg-white border border-slate-200 rounded-lg p-1 shadow-sm">
                <button onClick={() => setView("gantt")}
                        className={`view-btn text-xs font-semibold px-3 py-1.5 rounded-md flex items-center gap-1.5 ${view === "gantt" ? "active" : ""}`}>
                  <Icon name="LayoutList" size={13} /> Gantt
                </button>
                <button onClick={() => setView("cards")}
                        className={`view-btn text-xs font-semibold px-3 py-1.5 rounded-md flex items-center gap-1.5 ${view === "cards" ? "active" : ""}`}>
                  <Icon name="LayoutGrid" size={13} /> Fiches
                </button>
              </div>

              {/* Expand / Collapse */}
              <div className="flex items-center gap-2">
                <button onClick={expandAll}
                        className="text-xs font-medium text-slate-500 hover:text-slate-800 flex items-center gap-1.5
                                   px-3 py-1.5 rounded-lg hover:bg-slate-100 transition-colors">
                  <Icon name="ChevronDown" size={13} /> Tout déplier
                </button>
                <button onClick={collapseAll}
                        className="text-xs font-medium text-slate-500 hover:text-slate-800 flex items-center gap-1.5
                                   px-3 py-1.5 rounded-lg hover:bg-slate-100 transition-colors">
                  <Icon name="ChevronUp" size={13} /> Tout replier
                </button>
              </div>
            </div>

            {/* ── CONTENT ── */}
            {view === "gantt"
              ? <GanttView expandedPhases={expandedPhases} togglePhase={togglePhase} />
              : <CardView  expandedPhases={expandedPhases} togglePhase={togglePhase} />}

            {/* ── FOOTER ── */}
            <div className="bg-slate-50 border-t border-slate-200 px-4 md:px-6 py-5">
              <div className="flex flex-wrap items-start gap-6 justify-between">

                {/* Team members */}
                <div>
                  <h4 className="text-[10px] font-bold text-slate-400 uppercase tracking-widest mb-3">
                    Équipe Projet
                  </h4>
                  <div className="flex flex-wrap gap-2">
                    {teamMembers.map((m, i) => (
                      <div key={i} cl
