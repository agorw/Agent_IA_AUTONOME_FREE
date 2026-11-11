# Agent_IA_AUTONOME_FREE

AI-based brain tools that manage the operating system

To use this model, you must install the specific model—for example, `hf.co/Hob-forge/smeagle-4b:Q4_K_M`.

--------------------------- CONFIGURATION in file Python ---------------------------

MODEL = os.getenv("OLLAMA_MODEL", "hf.co/Hob-forge/smeagle-4b:Q4_K_M")

BASE_DIR = Path(os.getenv("AGENT_WORKSPACE", Path.cwd())).resolve()

CACHE_DIR = BASE_DIR / ".agent_cache"

LOG_FILE = CACHE_DIR / "agent.log"

MAX_TOOL_ITERATIONS = 35

MAX_FILE_CHARS = 180_000

MAX_TOOL_OUTPUT = 4_000

MAX_HISTORY_MESSAGES = 10

CONTEXT_SIZE = 8192

