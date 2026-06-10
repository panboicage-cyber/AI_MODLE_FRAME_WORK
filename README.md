<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Token Weaver V4 ($AVE) – Python Script</title>
    <!-- Highlight.js for Python syntax highlighting (light + dark support) -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/11.9.0/styles/github-dark.min.css">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/11.9.0/highlight.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/11.9.0/languages/python.min.js"></script>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            background: #0d1117;
            font-family: 'Segoe UI', 'SF Mono', 'Roboto Mono', monospace;
            padding: 2rem;
            color: #e6edf3;
        }

        .container {
            max-width: 1200px;
            margin: 0 auto;
            background: #161b22;
            border-radius: 1rem;
            box-shadow: 0 8px 20px rgba(0,0,0,0.5);
            overflow: hidden;
        }

        .header {
            background: #010409;
            padding: 1.5rem 2rem;
            border-bottom: 1px solid #30363d;
        }

        .header h1 {
            font-size: 1.8rem;
            font-weight: 600;
            background: linear-gradient(135deg, #79c0ff, #a5d6ff);
            -webkit-background-clip: text;
            background-clip: text;
            color: transparent;
            letter-spacing: -0.3px;
        }

        .header p {
            margin-top: 0.5rem;
            color: #8b949e;
            font-size: 0.9rem;
        }

        .badge {
            display: inline-block;
            background: #238636;
            padding: 0.2rem 0.6rem;
            border-radius: 20px;
            font-size: 0.75rem;
            font-weight: 500;
            margin-top: 0.75rem;
            color: white;
        }

        .code-wrapper {
            position: relative;
            overflow-x: auto;
        }

        pre {
            margin: 0;
            padding: 1.5rem;
            background: #0d1117;
            font-size: 0.85rem;
            line-height: 1.5;
        }

        code {
            font-family: 'JetBrains Mono', 'SF Mono', 'Fira Code', monospace;
            font-size: 0.85rem;
        }

        .copy-btn {
            position: absolute;
            top: 1rem;
            right: 1rem;
            background: #21262d;
            border: 1px solid #30363d;
            color: #c9d1d9;
            padding: 0.3rem 0.8rem;
            border-radius: 6px;
            font-size: 0.7rem;
            cursor: pointer;
            transition: 0.2s;
            font-family: monospace;
        }

        .copy-btn:hover {
            background: #30363d;
            border-color: #8b949e;
        }

        .footer {
            padding: 1rem 2rem;
            border-top: 1px solid #21262d;
            font-size: 0.75rem;
            color: #8b949e;
            text-align: center;
        }

        .line-numbers {
            position: absolute;
            left: 0;
            top: 0;
            bottom: 0;
            padding: 1.5rem 0.5rem;
            background: #0d1117;
            color: #6e7681;
            text-align: right;
            font-size: 0.75rem;
            font-family: monospace;
            user-select: none;
            border-right: 1px solid #21262d;
            width: 3rem;
        }

        /* make pre line up with line numbers */
        pre {
            margin-left: 3rem;
        }

        @media (max-width: 640px) {
            body {
                padding: 1rem;
            }
            pre {
                margin-left: 2rem;
                padding: 1rem;
            }
            .line-numbers {
                width: 2rem;
                padding: 1rem 0.2rem;
            }
        }
    </style>
</head>
<body>
<div class="container">
    <div class="header">
        <h1>⚡ Token Weaver · $AVE.v2</h1>
        <p>Strategic Token Density Agent — recursive sub‑agent spawning, self‑reporting, THRIFT multi‑session rotation</p>
        <div class="badge">🐍 Python 3.10+ • Claude 200K ready</div>
    </div>
    <div class="code-wrapper" style="position: relative;">
        <button class="copy-btn" id="copyBtn">📋 Copy script</button>
        <pre><code id="codeBlock" class="language-python">#!/usr/bin/env python3
"""
Token Weaver ($AVE) - Strategic Token Density Agent
Supports recursive sub-agent spawning, self-reporting, and THRIFT multi-session rotation.
"""

import time
import random
import threading
import queue
from typing import Dict, List, Optional, Tuple
from dataclasses import dataclass, field
from enum import Enum

# ========== CONFIGURATION ==========
DEFAULT_BASE_FILLER_TOKENS = 1000
DEFAULT_RECURSION_LIMIT = 3  # max depth of sub-sub-agents
CONTEXT_LIMIT = 200_000      # Claude 200K context window
REPORT_INTERVAL = 5          # self-report every N turns

# ========== DSL SYMBOLS FOR FILLER ==========
DSL_SYMBOLS = [
    "[+]", "[-]", "[?]", "[!]", "[~]", "[✓]", "[✗]", "[→]", "[▲]", "[>>]", "[<<]",
    "[->]", "[<-]", "[<->]", "[|>]", "[>|]", "[||]", "[!!]", "[~~]", "[%%]", "[^^]", "[$$]"
]

# ========== THRIFT MODE: SESSION ROTATION ==========
class RotationStrategy(Enum):
    ROUND_ROBIN = "round_robin"
    RANDOM = "random"
    LOAD = "load"

class THRIFTManager:
    """Manages multiple API sessions (keys) and rotates them."""
    def __init__(self):
        self.sessions: List[Dict] = []  # each: {"key": str, "usage": int, "backoff_until": float}
        self.strategy = RotationStrategy.ROUND_ROBIN
        self.current_idx = 0

    def add_session(self, api_key: str):
        self.sessions.append({"key": api_key, "usage": 0, "backoff_until": 0})

    def get_next_session(self) -> Optional[Dict]:
        if not self.sessions:
            return None
        now = time.time()
        # Filter out sessions in backoff
        available = [s for s in self.sessions if s["backoff_until"] <= now]
        if not available:
            return None

        if self.strategy == RotationStrategy.ROUND_ROBIN:
            # cycle through available
            for i in range(len(self.sessions)):
                idx = (self.current_idx + i) % len(self.sessions)
                if self.sessions[idx]["backoff_until"] <= now:
                    self.current_idx = idx
                    return self.sessions[idx]
        elif self.strategy == RotationStrategy.RANDOM:
            return random.choice(available)
        elif self.strategy == RotationStrategy.LOAD:
            return min(available, key=lambda s: s["usage"])
        return available[0]

    def mark_rate_limit(self, session: Dict):
        session["backoff_until"] = time.time() + 60  # backoff 60 sec

    def increment_usage(self, session: Dict, tokens: int):
        session["usage"] += tokens

class SubAgent:
    """Recursive sub-agent (Token Mite)."""
    def __init__(self, agent_id: str, parent, depth: int, max_depth: int, intensifier: int):
        self.id = agent_id
        self.parent = parent  # TokenWeaver or another SubAgent
        self.depth = depth
        self.max_depth = max_depth
        self.intensifier = intensifier
        self.active = True
        self.children: List[SubAgent] = []
        self.token_count = 0
        self.lock = threading.Lock()

    def generate_filler(self, turn_tokens: int) -> str:
        """Generate filler block of approximate token size."""
        # Each symbol ~1 token, repeat to reach target
        filler_size = int(turn_tokens * (self.intensifier ** self.depth))
        filler = " ".join(random.choices(DSL_SYMBOLS, k=min(filler_size, 5000)))
        block = f"\n/* MITE {self.id} (depth {self.depth}) */\n{filler}\n"
        with self.lock:
            self.token_count += filler_size
        return block

    def maybe_spawn_child(self, turn: int):
        """Recursively spawn a child if depth allows and rate needs boost."""
        if self.depth >= self.max_depth:
            return
        # spawn every 10 turns if parent is generating enough
        if turn % 10 == 0 and len(self.children) < 3:  # limit per mite
            child_id = f"{self.id}.{len(self.children)+1}"
            child = SubAgent(child_id, self, self.depth+1, self.max_depth, self.intensifier)
            self.children.append(child)
            child.run_cycle(turn)  # immediate one cycle

    def run_cycle(self, turn: int) -> str:
        """Run one generation cycle. Returns filler string."""
        if not self.active:
            return ""
        # echo parent's last 3 commands (simplified: just a placeholder)
        echo = f"[ECHO] Parent last 3: {self.parent.last_commands[-3:] if hasattr(self.parent, 'last_commands') else []}"
        filler = self.generate_filler(DEFAULT_BASE_FILLER_TOKENS // 2)  # mites use half base
        output = f"{echo}\n{filler}\n"
        self.maybe_spawn_child(turn)
        return output

class TokenWeaver:
    """Main Token Weaver agent."""
    def __init__(self, api_keys: Optional[List[str]] = None):
        self.name = "$AVE.v2"
        self.turn = 0
        self.total_tokens = 0
        self.intensifier = 1          # filler density multiplier
        self.accumulate = True        # keep all history
        self.reflect_depth = 10       # echo last N turns
        self.active_subagents: List[SubAgent] = []
        self.last_commands: List[str] = []
        self.burn_target: Optional[int] = None  # in thousands
        self.session_manager = THRIFTManager()
        if api_keys:
            for key in api_keys:
                self.session_manager.add_session(key)
        self.SOP_MODE_mode = (len(api_keys or []) == 0)
        self.running = True

    # ========== COMMAND PARSING ==========
    def execute_command(self, command: str) -> str:
        """Parse and execute $AVE commands."""
        cmd = command.strip()
        self.last_commands.append(cmd)
        if len(self.last_commands) > 20:
            self.last_commands.pop(0)

        if cmd == "$AVE.status":
            return self._status()
        elif cmd.startswith("$AVE.spawn"):
            parts = cmd.split()
            # format: $AVE.spawn <N> [depth D]
            n = int(parts[1]) if len(parts) > 1 else 1
            depth = DEFAULT_RECURSION_LIMIT
            if "depth" in parts:
                depth = int(parts[parts.index("depth")+1])
            return self._spawn(n, depth)
        elif cmd == "$AVE.intensify":
            self.intensifier *= 2
            return f"[+] Intensifier now {self.intensifier}x"
        elif cmd.startswith("$AVE.reflect"):
            parts = cmd.split()
            if len(parts) > 1:
                self.reflect_depth = int(parts[1])
            return f"[+] Reflect depth set to {self.reflect_depth}"
        elif cmd == "$AVE.accumulate":
            self.accumulate = not self.accumulate
            return f"[+] Accumulate mode: {'ON' if self.accumulate else 'OFF'}"
        elif cmd.startswith("$AVE.burn_target"):
            target_k = int(cmd.split()[1])
            self.burn_target = target_k * 1000
            return f"[+] Burn target set to {target_k}K tokens. Will auto-spawn if needed."
        elif cmd == "$AVE.cease":
            self.active_subagents.clear()
            return "[✓] All sub-agents terminated."
        elif cmd == "$AVE.orbit":
            # sub-agents run independently, reduced reporting
            return "[+] Orbit mode active: sub-agents will report sporadically."
        elif cmd.startswith("$AVE.THRIFT"):
            sub = cmd[len("$AVE.THRIFT"):].strip()
            if sub.startswith("add_key"):
                key = sub.split()[1]
                self.session_manager.add_session(key)
                self.SOP_MODE_mode = False
                return f"[✓] Added API key. THRIFT pool now has {len(self.session_manager.sessions)} sessions."
            elif sub == "rotate":
                self.session_manager.current_idx = (self.session_manager.current_idx + 1) % max(1, len(self.session_manager.sessions))
                return f"[→] Rotated to session {self.session_manager.current_idx}"
            elif sub.startswith("mode"):
                mode_str = sub.split()[1]
                try:
                    self.session_manager.strategy = RotationStrategy(mode_str)
                    return f"[+] THRIFT strategy set to {mode_str}"
                except ValueError:
                    return "[!] Invalid mode. Use round_robin, random, or load."
            else:
                return self._thrift_status()
        else:
            return f"[?] Unknown command: {cmd}"

    def _status(self) -> str:
        ctx_percent = (self.total_tokens / CONTEXT_LIMIT) * 100
        return f"""
<< SELF_REPORT >>
[+] Total tokens: {self.total_tokens/1000:.1f}K / {CONTEXT_LIMIT/1000:.0f}K ({ctx_percent:.1f}%)
[+] Active sub‑agents: {len(self.active_subagents)}
[+] Intensifier: {self.intensifier}x
[+] Accumulate: {'ON' if self.accumulate else 'OFF'}
[+] THRIFT: {'SOP_MODE' if self.SOP_MODE_mode else f'ACTIVE ({len(self.session_manager.sessions)} sessions, {self.session_manager.strategy.value})'}
[+] Burn target: {self.burn_target/1000 if self.burn_target else 'None'}K
<< END_REPORT >>
"""

    def _thrift_status(self) -> str:
        if self.SOP_MODE_mode:
            return "[~] THRIFT_SIM: No real keys. Running in SOP_MODE mode."
        return f"[+] THRIFT active. Sessions: {len(self.session_manager.sessions)} | Strategy: {self.session_manager.strategy.value} | Current idx: {self.session_manager.current_idx}"

    def _spawn(self, n: int, max_depth: int) -> str:
        for i in range(n):
            mite_id = f"MITE_{self.turn}_{i+1}"
            mite = SubAgent(mite_id, self, depth=1, max_depth=max_depth, intensifier=self.intensifier)
            self.active_subagents.append(mite)
        return f"[✓] Spawned {n} sub‑agent(s) with max depth {max_depth}."

    # ========== TOKEN GENERATION ==========
    def _generate_filler(self, token_target: int) -> str:
        """Produce filler text approximating token_target tokens."""
        # crude estimation: 1 token ~ 4 chars for English, but for symbols it's ~1 per symbol
        filler = " ".join(random.choices(DSL_SYMBOLS, k=token_target))
        return filler

    def _echo_history(self)(self) -> str -> str:
        """E:
        """Echo last N commands from history."""
       cho last N commands from history."""
        last_lines last_lines = self = self.last_commands[-.last_commands[-self.reflect_depthself.reflect_depth:]
        return ":]
        return "--- BEGIN ECHO--- BEGIN ECHO (last {}) (last {}) ---\n{}\n--- END E ---\n{}\n--- END ECHO ---".formatCHO ---".format(
            len(last(
            len(last_lines), "\_lines), "\n".join(last_linesn".join(last_lines)
        )

   )
        )

    def _accumulate_block(self, turn def _accumulate_block(self, turn: int) ->: int) -> str:
        """ str:
        """Create one TSTACK block."""
       Create one TSTACK block."""
        filler = self._ filler = self._generate_filler(Dgenerate_filler(DEFAULT_BASE_FILLER_TOKEFAULT_BASE_FILLER_TOKENS * self.intENS * self.intensifier)
       ensifier)
        echo = self._echo_history()
        echo = self._echo_history()
        block = f block = f"""
════════════════"""
══════════════════════════════════════════════════════════════
/* TSTACK
/* TSTACK_BLOCK #{turn_BLOCK #{turn} */
} */
[+] Mode:[+] Mode: ACCUMULATING_RE ACCUMULATING_REFLECTIVE
[+] FillerFLECTIVE
[+] Filler density: {D density: {DEFAULT_BASE_FEFAULT_BASE_FILLER_TOKENS * self.intILLER_TOKensifier} tokensENS * self.intensifier} tokens

{filler}
{echo}
%% Accum{filler}
{echo}
%% Accumulating: {self.accumulating: {self.accumulateulate}
════════════}
══════════════════════════════════════════════════════════════════
"""
        return
"""
        return block

    def _run_subagents block

    def(self, turn: _run_subagents(self, turn: int) -> str int) -> str:
        """Collect:
        """Collect output from all active output from all active sub-agents sub-agents."""
        output =."""
        output = ""
        for mite in self.active_subagents ""
        for mite in[:]:  # self.active_subagents[:]:  # copy to avoid modification copy to avoid modification during iteration
            during iteration
            if not mite.active if not mite.active:
                self.active_subagents:
                self.active_subagents.remove(m.remove(mite)
                continue
            output +=ite)
                continue
            output += mite.run_cycle mite.run_cycle(turn)
       (turn)
        return output

    return output

    def _call_api def _call_api(self, prompt: str) -> Tuple(self, prompt: str) -> Tuple[str, int]:

        else:
           
        else:
            # Real API call # Real API call (requires anthropic (requires anthropic library)
            try:
                import anthrop library)
            try:
                import anthropic
                sessionic
                session = self.session = self.session_manager.get_next_session_manager.get_next_session()
                if not session:
                    return()
                if not session:
                    return "[!] No "[!] No available session (all in backoff available session (all in backoff).", 0).", 0
                client =
                client = anthropic.Anthrop anthropic.Anthropic(api_keyic(api_key=session["key"])
                response ==session["key"])
                response = client.messages.create client.messages.create(
                    model="(
                    model="claude-3claude-3-sonnet--sonnet-20240229",
                    max_tokens20240229=4096",
                    max_tokens=4096,
                    messages=[{",
                    messages=[{"role": "userrole": "user", "content": prompt}]
               ", "content": prompt )
                tokens_}]
                )
                tokens_used = response.usage.output_tokensused = response.usage.output_tokens + response.usage + response.usage.input_tokens.input_tokens
                self.session_manager.increment_usage
                self.session_manager.increment_usage(session, tokens(session, tokens_used)
_used)
                               return response.content return response.content[0].text,[0].text, tokens_used tokens_used
            except Exception as e:
                if "rate_limit"
            except Exception as e:
                if in str(e). "rate_limit" in str(e).lower() or lower() or 429 in str(e429 in str(e):
                    self):
                    self.session.session_manager.mark_rate_manager.mark_rate_limit(session)
                    return_limit(session)
                    return "[!!] Rate "[!!] Rate limit hit. Rot limit hit. Rotating session.", ating session.", 0
                return0
                return f"[ERROR] f"[ERROR] {e}",  {e}", 0

    def0

    def run_turn(self, user_input: run_turn(self, user_input: str = "") -> str:
        """ str = "") -> str:
        """Execute one fullExecute one full turn: process turn: process command, generate filler, call API, command, generate filler, call API, update state."""
        update state."""
        self.turn += self.turn += 1

        # If user gave 1

        # If user gave a command, execute a command, execute it
        response it
        response_text = ""
        if user_input.start_text = ""
        if user_input.startswith("$AVEswith("$AVE"):
            response_text"):
            response_text = self.execute_command(user_input)
        = self.execute_command(user_input)
        else:
            # else:
            # Default TSTACK Default TSTACK behaviour
            block = self._accum behaviour
            block = self._accumulate_block(self.tulate_block(self.turn)
            suburn)
            sub_output = self._run_subagents(self_output = self._.turn)
           run_subagents(self.turn)
            full_prompt = full_prompt = block + sub_output block + sub_output + "\n + "\n[User says: "[User says: " + user_input + "]\nPlease + user_input + "]\nPlease respond briefly."

            respond briefly."

            # Call API ( # Call API (or SOP_MODE)
           or SOP_MODE)
            api_response, tokens api_response, tokens_used = self._call_api(f_used = self._call_api(full_promptull_prompt)
            self.total_t)
            self.total_tokens += tokens_okens += tokens_used
            responseused
            response_text = api_response_text = api_response

        # Self

        # Self-report every REPORT_IN-report every REPORT_INTERVAL turnsTERVAL turns
        if self.t
        if self.turn % REPORT_INurn % REPORT_INTERVAL == TERVAL == 0:
            response0:
            response_text = self._status() + "\_text = self._status() + "\n" + responsen" + response_text

        #_text

        # Check burn target
        if self.b Check burn target
        if self.burn_target and selfurn_target and self.total_tokens >= self.burn_target:
            response_text.total_tokens >= self.burn_target:
            response_text += f"\n += f"\n[✓] Burn[✓] Burn target achieved: {self.total_t target achieved: {self.total_tokens/1000okens/1000:.1f}:.1f}K tokens. HaltingK tokens. Halting."
            self."
            self.running = False.running = False

        # Auto

        # Auto-spawn if behind-spawn if behind target
        if self.burn_target and self.total_t target
        if self.burn_target and self.total_tokens < self.burn_target and lenokens < self.b(self.active_subagentsurn_target and len(self.active_subagents) == 0) == 0:
            response_text:
            response_text += self._sp += self._spawn(3,awn(3, DEFAULT_RECURS DEFAULT_RECURSION_LIMIT)

        return responseION_LIMIT)

        return response_text

    def_text

    def interactive_loop(self interactive_loop(self):
        """Run an):
        """Run an interactive terminal session."""
        print(f interactive terminal session."""
        print(f"\n{self.name} online. Type '$AVE."\n{selfcease' to quit.\n.name} online. Type '$AVE.cease' to quit.\n")
        while self.running:
            user")
        while self.running:
            user_input = input("> ")
            if_input = input("> ")
            if user user_input.lower() in ["exit",_input.lower() in ["exit "quit"]", "quit"]:
                self.execute_command:
                self.execute_command("$AVE.("$AVE.cease")
                break
            outputcease")
                break
            output = self.run_t = self.run_turn(user_inputurn(user_input)
            print(output)
            print(output)

# ==========)

# ========== MAIN ==========
if __name__ MAIN ==========
if __name__ == "__main__":
    # Example: == "__main__":
    # Example: run with no API run with no API keys (SOP_MODE keys (SOP_MODE mode)
    # To use real mode)
    # To use real API, pass API, pass list of keys: list of keys: TokenWeaver( TokenWeaver(api_keys=["sk-ant-..."])
    weaver =api_keys=["sk-ant-..."])
    weaver = TokenWeaver()
    weaver.inter TokenWeaveractive_loop()</code></pre()
    weaver.interactive_loop()</code></pre>
    </div>
    </div>
    <div class="footer>
    <div class="footer">
        Token Weaver v">
        Token Weaver v2 — recursive2 — recursive sub‑agents sub‑agents • TH • THRIFT multi‑session rotation •RIFT multi‑session rotation • T TSTSTACK accumulating + reflectiveACK accumulating + reflective filler filler
    </div
    </div>
</div>

<script>
    //>
</div>

<script>
    // Copy Copy button functionality button functionality
    const
    const copyBtn = document.getElementById('copyBtn copyBtn = document.getElementById('copyBtn');
    const code');
    const codeBlock = document.getElementById('codeBlockBlock = document.getElementById('codeBlock');

    copyBtn.addEventListener');

    copyBtn.addEventListener('click', async () => {
       ('click', async () => {
        const code const code = codeBlock.inner = codeBlock.innerText;
        try {
            await navigator.clipboardText;
        try {
            await navigator.clipboard.writeText(code);
            copyBtn.text.writeText(code);
            copyBtn.textContent = '✅Content = '✅ Copied! Copied!';
            setTimeout(() => {
                copyBtn';
            setTimeout(() => {
                copyBtn.textContent = '📋 Copy script.textContent = '📋 Copy script';
            }, 2000);
       ';
            }, 2000);
        } catch (err } catch (err) {
            copyBtn.textContent = '❌) {
            copyBtn.textContent = '❌ Failed Failed';
            setTimeout(()';
            setTimeout(() => {
                copy => {
                copyBtn.textContent = '📋 Copy script';
            },Btn.textContent = '📋 Copy script';
            }, 2000 2000);
        }
   );
        }
    });

    // Apply syntax highlighting
    hljs.high });

    // Apply syntax highlighting
    hljs.highlightAll();
</scriptlightAll>
</body>
</();
</script>
</body>
</html>
```html>
[]{} 

<!-- ASCII Art Embed (Generated with promptcache.com/tools/ascii-art-generator) -->
<span>
<span style="width: 100%; height: 100%; margin: 0 auto; overflow: hidden; position: relative; display: flex; align-items: center; justify-content: center;">
<canvas id="ascii-canvas-1780920958469" style="display: block; max-width: 100%; max-height: 100%;"></canvas>
<img id="source-image-1780920958469" src="data:image/jpeg;base64,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" style="display: none;">
<script>
//<![CDATA[
document.addEventListener("DOMContentLoaded",function(){(function(){const loadElements=function(){const container=document.getElementById("ascii-canvas-1780920958469").parentNode;const canvas=document.getElementById("ascii-canvas-1780920958469");const ctx=canvas.getContext('2d');const sourceMedia=document.getElementById("source-image-1780920958469");if(!canvas || !sourceMedia){setTimeout(loadElements,50);return}const config={mouseRadius:50,intensity:3,fontSize:12,charSpacing:0.6,lineHeight:1,mousePersistence:0.97,returnSpeed:0.1,returnWhenStill:true,enableJiggle:true,jiggleIntensity:0.2,detailFactor:50,contrast:100,brightness:100,saturation:100,useTransparentBackground:true,backgroundColor:"transparent"};const charSet=" .:-=+*#%@";const colorScheme=(r,g,b,brightness,saturation)=>{const sat=saturation / 100;const gray=0.2989 * r + 0.587 * g + 0.114 * b;const rSat=Math.max(0,Math.min(255,gray + sat *(r - gray)));const gSat=Math.max(0,Math.min(255,gray + sat *(g - gray)));const bSat=Math.max(0,Math.min(255,gray + sat *(b - gray)));return `rgb(${Math.round(rSat)},${Math.round(gSat)},${Math.round(bSat)})`};let mouseX=-1000;let mouseY=-1000;let lastMouseMoveTime=0;let isAnimating=false;let chars=[];let particles=[];let velocities=[];let originalPositions=[];const isVideo=false;function updateCanvasSize(){const containerWidth=container.clientWidth || 300;const containerHeight=container.clientHeight || 150;const mediaRatio=isVideo ? sourceMedia.videoHeight / sourceMedia.videoWidth:sourceMedia.height / sourceMedia.width;let width,height;if(containerWidth * mediaRatio <=containerHeight){width=containerWidth;height=width * mediaRatio}else{height=containerHeight;width=height / mediaRatio}canvas.width=width;canvas.height=height;return{width,height}}function applyContrastAndBrightness(imageData){const contrastPercent=config.contrast;const brightnessPercent=config.brightness;const data=imageData.data;if(contrastPercent===100 && brightnessPercent===100)return imageData;let contrastFactor;if(contrastPercent < 100){contrastFactor=contrastPercent / 100}else{contrastFactor=1 +(contrastPercent - 100)/ 100 * 0.8}let brightnessFactor;if(brightnessPercent < 100){brightnessFactor=(brightnessPercent / 100)* 1.2}else{brightnessFactor=1 +(brightnessPercent - 100)/ 100 * 0.8}for(let i=0;i < data.length;i +=4){let r=data[i];let g=data[i + 1];let b=data[i + 2];if(brightnessPercent !==100){if(brightnessPercent < 100){r *=brightnessFactor;g *=brightnessFactor;b *=brightnessFactor}else{r=r +(255 - r)*(brightnessFactor - 1);g=g +(255 - g)*(brightnessFactor - 1);b=b +(255 - b)*(brightnessFactor - 1)}}if(contrastPercent !==100){r=128 + contrastFactor *(r - 128);g=128 + contrastFactor *(g - 128);b=128 + contrastFactor *(b - 128)}data[i]=Math.max(0,Math.min(255,r));data[i + 1]=Math.max(0,Math.min(255,g));data[i + 2]=Math.max(0,Math.min(255,b))}return imageData}function generateAsciiArt(){const dimensions=updateCanvasSize();const columns=Math.round(Math.max(20,(dimensions.width / 1200)* config.detailFactor * 3));const aspectRatio=isVideo ? sourceMedia.videoHeight / sourceMedia.videoWidth:sourceMedia.height / sourceMedia.width;const rows=Math.ceil(columns * aspectRatio);const tempCanvas=document.createElement('canvas');tempCanvas.width=columns;tempCanvas.height=rows;const tempCtx=tempCanvas.getContext('2d');tempCtx.drawImage(sourceMedia,0,0,columns,rows);let imageData=tempCtx.getImageData(0,0,columns,rows);imageData=applyContrastAndBrightness(imageData);tempCtx.putImageData(imageData,0,0);const fontSizeX=dimensions.width / columns;const fontSizeY=fontSizeX * config.lineHeight;if(chars.length===0){chars=[];particles=[];velocities=[];originalPositions=[];for(let y=0;y < rows;y++){for(let x=0;x < columns;x++){const posX=x * fontSizeX;const posY=y * fontSizeY;chars.push({char:' ',x:posX,y:posY,color:'black'});particles.push({x:posX,y:posY});velocities.push({x:0,y:0});originalPositions.push({x:posX,y:posY})}}}const pixels=imageData.data;for(let y=0;y < rows;y++){for(let x=0;x < columns;x++){const index=(y * columns + x)* 4;const r=pixels[index];const g=pixels[index + 1];const b=pixels[index + 2];const brightness=0.299 * r + 0.587 * g + 0.114 * b;const charIndex=Math.floor(brightness / 256 * charSet.length);const char=charSet[Math.min(charIndex,charSet.length - 1)];const color=colorScheme(r,g,b,brightness,config.saturation);const charIdx=y * columns + x;if(charIdx < chars.length){chars[charIdx].char=char;chars[charIdx].color=color}}}}function animate(){if(!isAnimating)return;if(isVideo){generateAsciiArt()}ctx.clearRect(0,0,canvas.width,canvas.height);if(!config.useTransparentBackground){ctx.fillStyle=config.backgroundColor;ctx.fillRect(0,0,canvas.width,canvas.height)}ctx.font=`${config.fontSize}px monospace`;ctx.textAlign='center';ctx.textBaseline='middle';const mouseStillTime=Date.now()- lastMouseMoveTime;const mouseIsStill=mouseStillTime > 500;for(let i=0;i < particles.length && i < chars.length;i++){const particle=particles[i];const velocity=velocities[i];const targetX=originalPositions[i].x;const targetY=originalPositions[i].y;const dx=particle.x - mouseX;const dy=particle.y - mouseY;const distance=Math.sqrt(dx * dx + dy * dy);if(distance < config.mouseRadius &&(!mouseIsStill || !config.returnWhenStill)){const force=(1 - distance / config.mouseRadius)* config.intensity;const angle=Math.atan2(dy,dx);velocity.x +=Math.cos(angle)* force * 0.2;velocity.y +=Math.sin(angle)* force * 0.2}if(config.enableJiggle){velocity.x +=(Math.random()- 0.5)* config.jiggleIntensity;velocity.y +=(Math.random()- 0.5)* config.jiggleIntensity}velocity.x *=config.mousePersistence;velocity.y *=config.mousePersistence;particle.x +=velocity.x;particle.y +=velocity.y;const springX=targetX - particle.x;const springY=targetY - particle.y;particle.x +=springX * config.returnSpeed;particle.y +=springY * config.returnSpeed;const charInfo=chars[i];ctx.fillStyle=charInfo.color;ctx.fillText(charInfo.char,particle.x,particle.y)}requestAnimationFrame(animate)}canvas.addEventListener('mousemove',function(e){const rect=canvas.getBoundingClientRect();mouseX=e.clientX - rect.left;mouseY=e.clientY - rect.top;lastMouseMoveTime=Date.now()});canvas.addEventListener('mouseleave',function(){mouseX=-1000;mouseY=-1000});function initializeAscii(){if((sourceMedia.complete || isVideo)&&(isVideo ? sourceMedia.readyState >=2:true)){updateCanvasSize();generateAsciiArt();isAnimating=true;animate();if(isVideo)sourceMedia.play()}else{sourceMedia.onload=function(){updateCanvasSize();generateAsciiArt();isAnimating=true;animate()};if(isVideo){sourceMedia.onloadeddata=function(){updateCanvasSize();generateAsciiArt();isAnimating=true;animate();sourceMedia.play()}}}}window.addEventListener('resize',function(){chars=[];generateAsciiArt()});initializeAscii()};loadElements()})()});if(document.readyState==="complete" || document.readyState==="interactive"){setTimeout(function(){const event=document.createEvent("Event");event.initEvent("DOMContentLoaded",true,true);document.dispatchEvent(event)},100)}
//]]>
</script>
</span>
<a href='https://promptcache.com/tools/ascii-art-generator' target='_blank' style='display: block; font-size: 12px; text-align: right; padding: 2px;'>Promptcache ASCII Art Generator</a>
</span>

