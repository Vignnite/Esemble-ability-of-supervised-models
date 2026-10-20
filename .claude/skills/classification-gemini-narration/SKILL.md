---
name: classification-gemini-narration
description: Generalized Gemini narration helper, reusable across any case study (regression or classification) in this project — the generate_insight function every phase skill calls for its markdown remark cells, with the google_genai.types logger fix baked in from the start. Use whenever adding a Gemini-generated remark/insight cell anywhere in a notebook.
---

# Shared Gemini Narration Helper

## Guardrails

- **Set this up once, early** (right after imports, before the first remark
  cell is needed), not re-implemented per phase or per case study.
- **Load the key safely**: `from dotenv import load_dotenv; import os;
  load_dotenv(Path.cwd() / ".env"); GOOGLE_API_KEY = os.getenv("GOOGLE_API_KEY")`.
  Never hardcode or print the key anywhere, including in error messages.
- **Model-chain fallback**: try a `GEMINI_MODEL_CHAIN` of models newest-first
  (mirror whichever chain is currently working in this project's other case
  study notebooks, since they share the same key/quota), and on **any**
  exception (not just quota errors — a 404 for a retired model should also
  fall through) move to the next model in the chain before giving up.
- **Suppress the `google_genai.types` logger from the start**:
  `logging.getLogger("google_genai.types").setLevel(logging.ERROR)` right
  after `load_dotenv`, in the same cell. Some Gemini models return an
  internal "thinking" trace alongside the answer, and the SDK's `.text`
  shortcut calls `logger.warning(...)` to report it drops that part
  (`"Warning: there are non-text parts in the response:
  ['thought_signature']..."`). This is a real Python `logging` call, not a
  `warnings.warn` or a plain stderr print, so neither
  `warnings.filterwarnings("ignore")` nor wrapping the call in
  `contextlib.redirect_stderr` suppresses it (both verified not to work) —
  raising that specific logger's level is the only fix. Do this from the
  first build of the notebook, not as a later patch: a prior case study in
  this project initially shipped without this fix and every exported
  PDF/HTML showed a visible warning box after each Gemini cell until it was
  corrected.
- **Graceful fallback text**: if every model in the chain fails, return a
  clear markdown fallback message (`"**AI narrative unavailable**: set
  GOOGLE_API_KEY in .env..."`) plus the computed summary stats inline, so
  the notebook is still readable/gradeable without a working key.
- **Prompt discipline**: every call must build its prompt from real computed
  numbers passed in as arguments (never a hardcoded example). Instruct the
  model to avoid em dashes and use standard punctuation, matching the rest
  of the notebook's writing style.
- **Never let a Gemini failure break the pipeline** — every call site must
  degrade to the fallback text and let the rest of the notebook keep
  running.
- If the case study needs a narrower classification helper alongside the
  general `generate_insight` (e.g. classifying an unrecognized category code
  against a known set, as the CO2 case study did for transmission codes),
  build it as a second, smaller function reusing the same `_client` and
  `GEMINI_MODEL_CHAIN`, not a separate client/setup.
