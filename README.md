import { useState, useMemo } from "react";

// ── Paleta Marcopolo Industrial ──────────────────────────
const C = {
  bg: "#F0F0EE",
  card: "#FFFFFF",
  border: "#E4E3E0",
  hdr: "#1A1916",
  accent: "#E8731A",
  green: "#1A7A42",
  blue: "#1A5FA8",
  red: "#C0341D",
  muted: "#A8A5A0",
  text: "#1A1916",
  sub: "#6B6862",
};

const LINHAS = ["408", "409", "411", "412"];
const TURNOS = ["1º Turno", "2º Turno", "3º Turno"];
const PRIORIDADES = ["Alta", "Média", "Baixa"];
const COR_LINHA = { "408": "#1A7A42", "409": "#1A5FA8", "411": "#E8731A", "412": "#7C3AED" };
const COR_PRIO = { Alta: "#C0341D", Média: "#A86A00", Baixa: "#1A7A42" };

const now = () => {
  const d = new Date();
  return d.toLocaleString("pt-BR", { day: "2-digit", month: "2-digit", year: "numeric", hour: "2-digit", minute: "2-digit" });
};

const VAZIO = {
  linhas: [], vv: "", peca: "", codigo: "", quantidade: 1,
  turno: "", responsavel: "", prioridade: "Alta", obs: "",
};

// ── Componentes base ─────────────────────────────────────
function Chip({ label, cor, bg, small }) {
  return (
    <span style={{
      background: bg || cor + "18", color: cor, fontWeight: 700,
      fontSize: small ? 10 : 11, padding: small ? "2px 7px" : "3px 10px",
      borderRadius: 99, whiteSpace: "nowrap", display: "inline-block",
    }}>{label}</span>
  );
}

function Toggle({ label, active, cor, onClick }) {
  return (
    <button onClick={onClick} style={{
      padding: "8px 14px", borderRadius: 10, border: `2px solid ${active ? cor : C.border}`,
      background: active ? cor : "#fff", color: active ? "#fff" : C.sub,
      fontWeight: 700, fontSize: 13, cursor: "pointer", transition: "all .15s",
      fontFamily: "inherit",
    }}>{label}</button>
  );
}

function Campo({ label, required, children, hint }) {
  return (
    <div style={{ marginBottom: 16 }}>
      <label style={{ display: "block", fontSize: 11, fontWeight: 700, color: C.sub, textTransform: "uppercase", letterSpacing: ".6px", marginBottom: 6 }}>
        {label}{required && <span style={{ color: C.accent }}> *</span>}
      </label>
      {children}
      {hint && <div style={{ fontSize: 11, color: C.muted, marginTop: 4 }}>{hint}</div>}
    </div>
  );
}

const INPUT_STYLE = {
  width: "100%", padding: "11px 14px", borderRadius: 10,
  border: `1.5px solid ${C.border}`, fontSize: 14, color: C.text,
  fontFamily: "inherit", outline: "none", background: "#fff",
  boxSizing: "border-box",
};

// ── ABA FORMULÁRIO ───────────────────────────────────────
function AbaFormulario({ onSalvar }) {
  const [form, setForm] = useState(VAZIO);
  const [enviado, setEnviado] = useState(false);
  const [erros, setErros] = useState({});

  const set = (key, val) => setForm(f => ({ ...f, [key]: val }));

  const toggleLinha = (l) => {
    const arr = form.linhas.includes(l) ? form.linhas.filter(x => x !== l) : [...form.linhas, l];
    set("linhas", arr);
  };

  const validar = () => {
    const e = {};
    if (!form.linhas.length) e.linhas = "Selecione ao menos uma linha";
    if (!form.vv.trim()) e.vv = "Obrigatório";
    if (!form.peca.trim()) e.peca = "Obrigatório";
    if (!form.turno) e.turno = "Selecione o turno";
    if (!form.responsavel.trim()) e.responsavel = "Obrigatório";
    if (form.quantidade < 1) e.quantidade = "Mínimo 1";
    setErros(e);
    return Object.keys(e).length === 0;
  };

  const handleSubmit = () => {
    if (!validar()) return;
    onSalvar({ ...form, id: Date.now(), data: now() });
    setForm(VAZIO);
    setErros({});
    setEnviado(true);
    setTimeout(() => setEnviado(false), 2800);
  };

  return (
    <div style={{ padding: "16px 16px 40px", maxWidth: 560, margin: "0 auto" }}>
      {enviado && (
        <div style={{
          background: "#EBF7F0", border: "1.5px solid #1A7A42", borderRadius: 12,
          padding: "14px 16px", marginBottom: 20, display: "flex", alignItems: "center", gap: 10
        }}>
          <span style={{ fontSize: 20 }}>✅</span>
          <div>
            <div style={{ fontWeight: 700, color: C.green, fontSize: 14 }}>Registrado com sucesso!</div>
            <div style={{ fontSize: 12, color: C.sub }}>Acesse o Histórico para ver ou exportar.</div>
          </div>
        </div>
      )}

      {/* Linha */}
      <Campo label="Linha" required>
        <div style={{ display: "flex", gap: 8, flexWrap: "wrap" }}>
          {LINHAS.map(l => (
            <Toggle key={l} label={l} active={form.linhas.includes(l)} cor={COR_LINHA[l]} onClick={() => toggleLinha(l)} />
          ))}
        </div>
        {erros.linhas && <div style={{ fontSize: 11, color: C.red, marginTop: 4 }}>⚠ {erros.linhas}</div>}
      </Campo>

      {/* VV + Código */}
      <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 12 }}>
        <Campo label="Nº VV" required>
          <input style={{ ...INPUT_STYLE, borderColor: erros.vv ? C.red : C.border }}
            placeholder="VV00530XXX" value={form.vv} onChange={e => set("vv", e.target.value.toUpperCase())} />
          {erros.vv && <div style={{ fontSize: 11, color: C.red, marginTop: 4 }}>⚠ {erros.vv}</div>}
        </Campo>
        <Campo label="Código da Peça">
          <input style={INPUT_STYLE} placeholder="Ex: 0700.8204" value={form.codigo}
            onChange={e => set("codigo", e.target.value)} />
        </Campo>
      </div>

      {/* Peça faltando */}
      <Campo label="Peça faltando" required>
        <textarea style={{ ...INPUT_STYLE, resize: "vertical", minHeight: 80, lineHeight: 1.5 }}
          placeholder="Descreva a peça ou componente faltando..."
          value={form.peca} onChange={e => set("peca", e.target.value)} />
        {erros.peca && <div style={{ fontSize: 11, color: C.red, marginTop: 4 }}>⚠ {erros.peca}</div>}
      </Campo>

      {/* Quantidade + Prioridade */}
      <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 12 }}>
        <Campo label="Quantidade" required>
          <input type="number" min={1} style={{ ...INPUT_STYLE, borderColor: erros.quantidade ? C.red : C.border }}
            value={form.quantidade} onChange={e => set("quantidade", parseInt(e.target.value) || 1)} />
        </Campo>
        <Campo label="Prioridade">
          <div style={{ display: "flex", gap: 6, flexWrap: "wrap" }}>
            {PRIORIDADES.map(p => (
              <button key={p} onClick={() => set("prioridade", p)} style={{
                padding: "8px 12px", borderRadius: 8,
                border: `2px solid ${form.prioridade === p ? COR_PRIO[p] : C.border}`,
                background: form.prioridade === p ? COR_PRIO[p] : "#fff",
                color: form.prioridade === p ? "#fff" : C.sub,
                fontWeight: 700, fontSize: 12, cursor: "pointer", fontFamily: "inherit",
              }}>{p}</button>
            ))}
          </div>
        </Campo>
      </div>

      {/* Turno */}
      <Campo label="Turno" required>
        <div style={{ display: "flex", gap: 8, flexWrap: "wrap" }}>
          {TURNOS.map(t => (
            <Toggle key={t} label={t} active={form.turno === t} cor={C.blue} onClick={() => set("turno", t)} />
          ))}
        </div>
        {erros.turno && <div style={{ fontSize: 11, color: C.red, marginTop: 4 }}>⚠ {erros.turno}</div>}
      </Campo>

      {/* Responsável */}
      <Campo label="Responsável" required>
        <input style={{ ...INPUT_STYLE, borderColor: erros.responsavel ? C.red : C.border }}
          placeholder="Nome de quem está registrando"
          value={form.responsavel} onChange={e => set("responsavel", e.target.value)} />
        {erros.responsavel && <div style={{ fontSize: 11, color: C.red, marginTop: 4 }}>⚠ {erros.responsavel}</div>}
      </Campo>

      {/* Observações */}
      <Campo label="Observações">
        <textarea style={{ ...INPUT_STYLE, resize: "vertical", minHeight: 64, lineHeight: 1.5 }}
          placeholder="Informações adicionais (opcional)..."
          value={form.obs} onChange={e => set("obs", e.target.value)} />
      </Campo>

      <button onClick={handleSubmit} style={{
        width: "100%", background: C.accent, color: "#fff", border: "none",
        borderRadius: 12, padding: "15px", fontSize: 15, fontWeight: 800,
        cursor: "pointer", fontFamily: "inherit", letterSpacing: ".3px",
        boxShadow: "0 4px 14px rgba(232,115,26,.35)", transition: "opacity .15s",
      }}>
        ⚡ Registrar Peça Faltando
      </button>
    </div>
  );
}

// ── ABA HISTÓRICO ────────────────────────────────────────
function AbaHistorico({ registros, onDeletar, onFiltro, filtroAtivo }) {
  const [expandido, setExpandido] = useState(null);
  const [busca, setBusca] = useState("");
  const [filtroLinha, setFiltroLinha] = useState("");
  const [filtroPrio, setFiltroPrio] = useState("");

  const filtered = useMemo(() => {
    return registros.filter(r => {
      const txt = busca.toLowerCase();
      const matchBusca = !busca ||
        r.vv.toLowerCase().includes(txt) ||
        r.peca.toLowerCase().includes(txt) ||
        r.responsavel.toLowerCase().includes(txt) ||
        r.codigo.toLowerCase().includes(txt);
      const matchLinha = !filtroLinha || r.linhas.includes(filtroLinha);
      const matchPrio = !filtroPrio || r.prioridade === filtroPrio;
      return matchBusca && matchLinha && matchPrio;
    });
  }, [registros, busca, filtroLinha, filtroPrio]);

  if (registros.length === 0) return (
    <div style={{ padding: "60px 16px", textAlign: "center" }}>
      <div style={{ fontSize: 40, marginBottom: 12 }}>📋</div>
      <div style={{ fontWeight: 700, fontSize: 16, color: C.sub, marginBottom: 6 }}>Nenhum registro ainda</div>
      <div style={{ fontSize: 13, color: C.muted }}>Registre a primeira peça faltando na aba <b>Novo Registro</b>.</div>
    </div>
  );

  return (
    <div style={{ padding: "12px 16px 40px", maxWidth: 640, margin: "0 auto" }}>
      {/* Busca */}
      <input
        placeholder="🔍  Buscar por VV, peça, responsável..."
        value={busca} onChange={e => setBusca(e.target.value)}
        style={{ ...INPUT_STYLE, marginBottom: 10, background: "#fff" }}
      />

      {/* Filtros */}
      <div style={{ display: "flex", gap: 6, flexWrap: "wrap", marginBottom: 14 }}>
        <button onClick={() => setFiltroLinha("")} style={{
          padding: "5px 12px", borderRadius: 8, border: `1.5px solid ${!filtroLinha ? C.accent : C.border}`,
          background: !filtroLinha ? C.accent : "#fff", color: !filtroLinha ? "#fff" : C.sub,
          fontWeight: 700, fontSize: 12, cursor: "pointer", fontFamily: "inherit",
        }}>Todas</button>
        {LINHAS.map(l => (
          <button key={l} onClick={() => setFiltroLinha(filtroLinha === l ? "" : l)} style={{
            padding: "5px 12px", borderRadius: 8,
            border: `1.5px solid ${filtroLinha === l ? COR_LINHA[l] : C.border}`,
            background: filtroLinha === l ? COR_LINHA[l] : "#fff",
            color: filtroLinha === l ? "#fff" : C.sub,
            fontWeight: 700, fontSize: 12, cursor: "pointer", fontFamily: "inherit",
          }}>{l}</button>
        ))}
        {PRIORIDADES.map(p => (
          <button key={p} onClick={() => setFiltroPrio(filtroPrio === p ? "" : p)} style={{
            padding: "5px 12px", borderRadius: 8,
            border: `1.5px solid ${filtroPrio === p ? COR_PRIO[p] : C.border}`,
            background: filtroPrio === p ? COR_PRIO[p] : "#fff",
            color: filtroPrio === p ? "#fff" : C.sub,
            fontWeight: 700, fontSize: 12, cursor: "pointer", fontFamily: "inherit",
          }}>{p}</button>
        ))}
      </div>

      <div style={{ fontSize: 11, color: C.muted, marginBottom: 10 }}>
        {filtered.length} de {registros.length} registro{registros.length !== 1 ? "s" : ""}
      </div>

      {filtered.map(r => {
        const aberto = expandido === r.id;
        return (
          <div key={r.id} style={{
            background: "#fff", borderRadius: 12, border: `1.5px solid ${C.border}`,
            marginBottom: 8, overflow: "hidden",
            boxShadow: aberto ? "0 4px 20px rgba(0,0,0,.08)" : "none",
          }}>
            {/* Header do card */}
            <div onClick={() => setExpandido(aberto ? null : r.id)} style={{
              padding: "12px 14px", cursor: "pointer",
              display: "flex", alignItems: "center", gap: 8, flexWrap: "wrap"
            }}>
              <Chip label={COR_PRIO[r.prioridade] ? r.prioridade : "—"} cor={COR_PRIO[r.prioridade] || C.muted} small />
              <div style={{ display: "flex", gap: 4 }}>
                {r.linhas.map(l => <Chip key={l} label={l} cor={COR_LINHA[l]} small />)}
              </div>
              <span style={{ fontWeight: 800, fontSize: 13, color: C.accent, fontFamily: "monospace" }}>{r.vv}</span>
              <span style={{ fontWeight: 600, fontSize: 13, color: C.text, flex: 1, minWidth: 100, overflow: "hidden", textOverflow: "ellipsis", whiteSpace: "nowrap" }}>
                {r.peca.length > 38 ? r.peca.slice(0, 38) + "…" : r.peca}
              </span>
              <span style={{ fontSize: 11, color: C.muted, marginLeft: "auto", whiteSpace: "nowrap" }}>{r.data}</span>
              <span style={{ fontSize: 14, color: C.muted }}>{aberto ? "▲" : "▼"}</span>
            </div>

            {/* Detalhe expandido */}
            {aberto && (
              <div style={{ borderTop: `1px solid ${C.border}`, padding: "14px" }}>
                <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 1, background: "#F0F0EE", borderRadius: 10, overflow: "hidden", marginBottom: 12 }}>
                  {[
                    ["Peça", r.peca],
                    ["Código", r.codigo || "—"],
                    ["Quantidade", r.quantidade],
                    ["Turno", r.turno],
                    ["Responsável", r.responsavel],
                    ["Data/Hora", r.data],
                  ].map(([lbl, val]) => (
                    <div key={lbl} style={{ background: "#fff", padding: "9px 12px" }}>
                      <div style={{ fontSize: 9, color: C.muted, textTransform: "uppercase", letterSpacing: ".4px", marginBottom: 2 }}>{lbl}</div>
                      <div style={{ fontWeight: 700, fontSize: 13, color: C.text, wordBreak: "break-word" }}>{val}</div>
                    </div>
                  ))}
                </div>
                {r.obs && (
                  <div style={{ background: "#FFF8E6", borderRadius: 8, padding: "10px 12px", marginBottom: 12 }}>
                    <div style={{ fontSize: 9, color: C.muted, textTransform: "uppercase", letterSpacing: ".4px", marginBottom: 3 }}>Observações</div>
                    <div style={{ fontSize: 13, color: C.text }}>{r.obs}</div>
                  </div>
                )}
                <button onClick={() => onDeletar(r.id)} style={{
                  background: "#FDECEA", border: "none", borderRadius: 8, padding: "8px 16px",
                  cursor: "pointer", color: C.red, fontWeight: 700, fontSize: 12, fontFamily: "inherit"
                }}>🗑 Remover registro</button>
              </div>
            )}
          </div>
        );
      })}
    </div>
  );
}

// ── ABA EXPORTAR ─────────────────────────────────────────
function AbaExportar({ registros }) {
  const [selecionados, setSelecionados] = useState(["408", "409", "411", "412"]);
  const [soloAlta, setSoloAlta] = useState(false);

  const filtrados = registros.filter(r => {
    const matchLinha = r.linhas.some(l => selecionados.includes(l));
    const matchPrio = !soloAlta || r.prioridade === "Alta";
    return matchLinha && matchPrio;
  });

  const exportarCSV = () => {
    if (!filtrados.length) return;
    const cabecalho = ["Data/Hora", "Linha(s)", "Nº VV", "Peça Faltando", "Código", "Quantidade", "Prioridade", "Turno", "Responsável", "Observações"];
    const linhas = filtrados.map(r => [
      r.data, r.linhas.join("+"), r.vv, `"${r.peca.replace(/"/g, '""')}"`,
      r.codigo, r.quantidade, r.prioridade, r.turno, r.responsavel,
      r.obs ? `"${r.obs.replace(/"/g, '""')}"` : ""
    ]);
    const csv = "\uFEFF" + [cabecalho, ...linhas].map(l => l.join(";")).join("\n");
    const blob = new Blob([csv], { type: "text/csv;charset=utf-8;" });
    const url = URL.createObjectURL(blob);
    const a = document.createElement("a");
    a.href = url; a.download = `pecas_faltando_${new Date().toLocaleDateString("pt-BR").replace(/\//g, "-")}.csv`;
    a.click(); URL.revokeObjectURL(url);
  };

  const exportarTXT = () => {
    if (!filtrados.length) return;
    const linhas = [`RELATÓRIO — PEÇAS FALTANDO`, `Gerado em: ${now()}`, `Total: ${filtrados.length} registro(s)`, "═".repeat(50), ""];
    filtrados.forEach((r, i) => {
      linhas.push(`[${i + 1}] ${r.data} — ${r.linhas.join("+")} — ${r.turno}`);
      linhas.push(`    VV: ${r.vv}  |  Código: ${r.codigo || "—"}  |  Qtd: ${r.quantidade}  |  Prioridade: ${r.prioridade}`);
      linhas.push(`    Peça: ${r.peca}`);
      linhas.push(`    Responsável: ${r.responsavel}`);
      if (r.obs) linhas.push(`    Obs: ${r.obs}`);
      linhas.push("─".repeat(50));
    });
    const blob = new Blob([linhas.join("\n")], { type: "text/plain;charset=utf-8" });
    const url = URL.createObjectURL(blob);
    const a = document.createElement("a");
    a.href = url; a.download = `relatorio_pecas_${new Date().toLocaleDateString("pt-BR").replace(/\//g, "-")}.txt`;
    a.click(); URL.revokeObjectURL(url);
  };

  const totalAlta = registros.filter(r => r.prioridade === "Alta").length;
  const porLinha = LINHAS.reduce((acc, l) => {
    acc[l] = registros.filter(r => r.linhas.includes(l)).length;
    return acc;
  }, {});

  return (
    <div style={{ padding: "16px 16px 40px", maxWidth: 560, margin: "0 auto" }}>
      {/* Resumo */}
      <div style={{ background: "#1A1916", borderRadius: 16, padding: 18, marginBottom: 18, color: "#fff" }}>
        <div style={{ fontWeight: 800, fontSize: 13, color: C.muted, textTransform: "uppercase", letterSpacing: ".6px", marginBottom: 12 }}>Resumo da Sessão</div>
        <div style={{ display: "grid", gridTemplateColumns: "repeat(3,1fr)", gap: 8, marginBottom: 14 }}>
          <div style={{ background: "rgba(255,255,255,.07)", borderRadius: 10, padding: "12px 8px", textAlign: "center" }}>
            <div style={{ fontSize: 30, fontWeight: 800, color: C.accent }}>{registros.length}</div>
            <div style={{ fontSize: 10, color: C.muted }}>Total</div>
          </div>
          <div style={{ background: "rgba(255,255,255,.07)", borderRadius: 10, padding: "12px 8px", textAlign: "center" }}>
            <div style={{ fontSize: 30, fontWeight: 800, color: "#C0341D" }}>{totalAlta}</div>
            <div style={{ fontSize: 10, color: C.muted }}>Alta prioridade</div>
          </div>
          <div style={{ background: "rgba(255,255,255,.07)", borderRadius: 10, padding: "12px 8px", textAlign: "center" }}>
            <div style={{ fontSize: 30, fontWeight: 800, color: C.green }}>{registros.filter(r => r.prioridade === "Baixa").length}</div>
            <div style={{ fontSize: 10, color: C.muted }}>Baixa prioridade</div>
          </div>
        </div>
        <div style={{ display: "grid", gridTemplateColumns: "repeat(4,1fr)", gap: 6 }}>
          {LINHAS.map(l => (
            <div key={l} style={{ background: COR_LINHA[l] + "25", borderRadius: 8, padding: "8px 6px", textAlign: "center" }}>
              <div style={{ fontSize: 18, fontWeight: 800, color: COR_LINHA[l] }}>{porLinha[l]}</div>
              <div style={{ fontSize: 10, color: COR_LINHA[l], fontWeight: 700 }}>L{l}</div>
            </div>
          ))}
        </div>
      </div>

      {/* Filtros export */}
      <div style={{ background: "#fff", borderRadius: 12, border: `1.5px solid ${C.border}`, padding: 16, marginBottom: 16 }}>
        <div style={{ fontWeight: 700, fontSize: 13, color: C.text, marginBottom: 12 }}>Filtrar antes de exportar</div>
        <div style={{ fontSize: 11, fontWeight: 700, color:
