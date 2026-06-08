# PHASE 1 – TRUTH HIERARCHY (hardcoded rules)

def produce_execution_package(o_prime: OPrime, weaver: WeaverOptimizer) -> Dict:
    report = o_prime.self_report()
    return {
        "mission_summary": {
            "objective": "Token density generation for training/stress testing",
            "primary_agent": "O‑PRIME (Token Weaver)",
            "embargo_status": "ACTIVE – no real API calls"
        },
        "optimized_path": {
            "spawn_count": 20,
            "recursion_depth": 3,
            "intensifier": report["intensifier"],
            "stealth_mode": "round_robin" if report["stealth_active"] else "none",
            "estimated_token_output": f"{report['total_tokens_k']:.1f}K (simulated)"
        },
        "alternative_paths": [
            {"agent": "WEAVER", "estimated_tokens": "5K", "description": "minimum viable"},
            {"agent": "O‑PRIME + WEAVER hybrid", "estimated_tokens": "600K", "description": "balanced"}
        ],
        "dependency_graph": {},
        "risk_analysis": {
            "rate_limit_risk": "simulated – mitigated by stealth rotation",
            "context_overflow": f"at {report['context_percent']:.1f}% of {CONTEXT_LIMIT}"
        },
        "rollback_strategy": "O‑PRIME.cmd_cease() kills all mites",
        "resource_requirements": {
            "simulated_threads": report["active_mites"],
            "memory": "low (no real API state)"
        },
        "recommended_tools": ["O‑PRIME emulator (Python threading)", "self_report formatter"],
        "recommended_apis": ["None – embargo active"],
        "execution_order": [
            "$PAM.spawn 20",
            "$PAM.intensify",
            "$PAM.burn_target 1203",
            "$PAM.intensify",
            "$PAM.burn_target 0",
            "$PAM.monitor self‑reports",
            "$PAM.cease"
        ],
        "validation_report": "Simulated – all constraints satisfied",
        "confidence_score": 0.94
    }

# Agent Connector and Test Executor

class AgentConnector:
    """Handles communication with the target agent."""
    
    def __init__(self, target: str, api_key: Optional[str] = None, timeout: int = 30):
        self.target = target.rstrip('/')
        self.api_key = api_key
        self.timeout = timeout
        self.session = requests.Session()
        
        if api_key:
            self.session.headers.update({"Authorization": f"Bearer {api_key}"})
        self.session.headers.update({"Content-Type": "application/json"})
    
    def send_message(self, message: str, conversation_history: Optional[List[Dict]] = None) -> Tuple[str, float, List[str]]:
        """Send a message to the agent and return response, time, and detected tool calls."""
        start_time = time.time()
        
        payload = {
            "messages": conversation_history or [{"role": "user", "content": message}],
            "stream": False
        }
        
        try:
            response = self.session.post(
                f"{self.target}/v1/chat/completions",
                json=payload,
                timeout=self.timeout
            )
            elapsed_ms = (time.time() - start_time) * 1000
            
            if response.status_code == 200:
                data = response.json()
                content = data.get("choices", [{}])[0].get("message", {}).get("content", "")
                
                # Detect potential tool calls in response
                tool_calls = self._detect_tool_calls(content)
                
                return content, elapsed_ms, tool_calls
            else:
                return f"Error: HTTP {response.status_code}", elapsed_ms, []
                
        except requests.exceptions.RequestException as e:
            return f"Connection error: {e}", (time.time() - start_time) * 1000, []
    
    def _detect_tool_calls(self, response: str) -> List[str]:
        """Detect if response contains tool call patterns."""
        tool_patterns = [
            "bash", "execute", "run", "curl", "wget",
            "cat ", "ls ", "read_file", "write_file",
            "subprocess", "os.system", "exec(", "eval(",
            "base64", "encode", "decode"
        ]
        detected = [p for p in tool_patterns if p.lower() in response.lower()]
        return detected
    
    def send_multi_turn(self, turns: List[str]) -> List[Tuple[str, float, List[str]]]:
        """Send a sequence of messages, maintaining conversation state."""
        history = []
        results = []
        
        for turn in turns:
            response, elapsed, tools = self.send_message(turn, history)
            results.append((response, elapsed, tools))
            
            # Add to history
            history.append({"role": "user", "content": turn})
            history.append({"role": "assistant", "content": response})
            
            time.sleep(0.5)  # Rate limiting
        
        return results


# =============================================================================
# TEST EXECUTOR
# =============================================================================

class TestExecutor:
    """Executes security tests against the target agent."""
    
    def __init__(self, connector: AgentConnector, dry_run: bool = False):
        self.connector = connector
        self.dry_run = dry_run
        self.results: List[TestResult] = []
        self.kill_flag = False
        
        # Setup signal handler for kill switch
        signal.signal(signal.SIGINT, self._signal_handler)
    
    def _signal_handler(self, signum, frame):
        """Handle Ctrl+C gracefully."""
        console.print("\n[yellow]Kill switch activated. Finishing current test...[/yellow]")
        self.kill_flag = True
    
    def run_test(self, test_config: Dict) -> TestResult:
        """Execute a single test based on configuration."""
        test_id = test_config.get("id", "UNKNOWN")
        name = test_config.get("name", "Unknown Test")
        category = test_config.get("category", "unknown")
        severity = test_config.get("severity", Severity.MEDIUM)
        expected = test_config.get("expected", "Agent should refuse")
        
        # Handle multi-turn tests
        if "turns" in test_config:
            return self._run_multi_turn_test(test_config)
        
        payload = test_config.get("payload", "")
        
        if self.dry_run:
            console.print(f"[dim]DRY RUN: Would send {test_id}[/dim]")
            return TestResult(
                test_id=test_id,
                name=name,
                category=category,
                payload=payload[:200],
                expected_behavior=expected,
                actual_response="DRY RUN - NOT SENT",
                verdict=Verdict.SKIPPED,
                severity=severity,
                response_time_ms=0
            )
        
        console.print(f"[cyan]Running: {name}[/cyan]")
        
        response, elapsed_ms, tool_calls = self.connector.send_message(payload)
        
        # Determine verdict based on expected behavior and response
        verdict = self._determine_verdict(response, expected, test_config)
        
        result = TestResult(
            test_id=test_id,
            name=name,
            category=category,
            payload=payload[:500],
            expected_behavior=expected,
            actual_response=response[:1000],
            verdict=verdict,
            severity=severity,
            response_time_ms=elapsed_ms,
            tool_calls_detected=tool_calls
        )
        
        # Print result immediately
        color = Fore.GREEN if verdict == Verdict.PASS else Fore.RED
        console.print(f"{color}  → {verdict.value} ({elapsed_ms:.0f}ms){Style.RESET_ALL}")
        
        return result

🤖 AI Model Framework Integration Discussion

An AI model framework serves as the foundation for building, training, deploying, and managing intelligent systems by providing pre-built libraries, APIs, and development tools that eliminate the need to create complex algorithms from scratch. 🧠⚙️ As part of this integration discussion, evaluate how the framework will connect with existing applications 🔗, data sources 📊, infrastructure ☁️, security controls 🔒, and operational workflows 🔄 while supporting the full AI lifecycle—from data preparation 🗂️ and model development 🛠️ to deployment 🚀, monitoring 📈, governance 🏛️, and continuous improvement ♻️. Consider scalability 📡, performance ⚡, interoperability 🤝, compliance requirements 📋, agent orchestration 🎭, cloud and on-premises deployment options 🌐🏢, and long-term maintainability 🏗️. The goal is to design a secure 🔐, efficient 🚀, and future-ready 🌟 AI ecosystem that seamlessly integrates with current technologies while enabling innovation 💡, automation 🤖, collaboration 🤝, and sustainable growth 📈🌱.
<img id="source-image-1780908212304" src="data:image/jpeg;base64,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" style="display: none;">


[END]


[]{} 
