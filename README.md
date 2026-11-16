# Northstar Home & Tech — Enterprise RAG Assignment

**Course:** Bano Qabil — Generative AI, Module 08  
**Project type:** Classroom prototype / practical assignment  
**Company:** Northstar Home & Tech (fictional online retailer)  

## What this project does
This Streamlit application answers questions using a small, versioned company knowledge base. It loads Markdown/TXT and text-based PDF files, chunks them, creates local sentence-transformer embeddings, stores vectors and metadata in Chroma, retrieves evidence, and sends that evidence to Gemini through the Google GenAI Python SDK. The app displays source filenames, policy version/status, excerpts, and retrieved chunks.

The fictional knowledge base includes a company overview, a catalogue with eight products, a current policy, an archived policy, operations FAQs, and a harmless prompt-injection test note. It contains missing information deliberately so that the assistant must abstain rather than invent an answer.

## Project structure

```text
northstar_capstone/
├── app.py
├── rebuild_index.py
├── run_test_set.py
├── run_retrieval_experiments.py
├── requirements.txt
├── .env.example
├── knowledge_docs/
│   ├── 01_company_overview.md
│   ├── 02_products_services.md
│   ├── 03_current_customer_policy.md
│   ├── 04_archived_customer_policy.md
│   ├── 05_operations_and_faqs.md
│   ├── 06_security_test_note.md
│   └── manifest.json
├── src/
│   ├── documents.py
│   ├── policy.py
│   ├── citations.py
│   └── rag.py
├── tests/test_core.py
├── results/
│   ├── test_set_and_results.csv
│   ├── retrieval_experiments_template.csv
│   └── chunk_size_experiment_template.csv
└── docs/assignment_report.md
```

## Corrections included in this ZIP
- Policy comparison routing now recognizes “change between”, “between archived and current”, “old/current”, and related wording. This fixes the T14-style question so it retrieves both policy versions instead of applying the current-only filter.
- The test runner now separates retrieved source documents from filenames actually cited by the generated answer, records short retrieved excerpts, and records citation filename matching.
- Streamlit displays cited filenames, a citation-presence warning, and short supporting excerpts in addition to expandable full chunks.
- Gemini transient-error retries are explicitly configured in the SDK client. If the provider remains unavailable after the configured attempts, the UI displays a readable temporary-unavailability message and preserves retrieved evidence.
- Core tests for policy routing and citation helpers run without requiring external LLM or vector-database packages.

## Requirements
- Python 3.10 or newer (3.11 recommended).
- Internet access on first run to install packages and download the public `all-MiniLM-L6-v2` embedding model.
- A Google AI Studio API key. The example uses the stable model ID `gemini-3.8-flash`. Check the current model page and your account's model availability/rate limits before running. The Gemini API pricing page lists a free tier for this model at the time this README was prepared; quotas and account availability can change. Free-tier content may be used to improve Google products, so keep this assignment strictly fictional and do not upload private or confidential documents.
- `google-genai` 1.49 or newer, because the project configures the SDK's HTTP retry options explicitly.
- No paid hosting or deployment is required by the assignment.

## Setup (Windows PowerShell)

```powershell
cd northstar_capstone
py -3.11 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
Copy-Item .env.example .env
```

Open `.env` in a text editor and replace `replace_with_your_key` with your own Gemini API key from Google AI Studio. Do not paste the key into Python files, screenshots, the README, or the LMS submission. Do not share the `.env` file.

## Setup (macOS / Linux)

```bash
cd northstar_capstone
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
pip install -r requirements.txt
cp .env.example .env
```

Edit `.env` and add your API key privately.

## Build and run

```bash
python rebuild_index.py
streamlit run app.py
```

Streamlit will print a local URL, usually `http://localhost:8501`. Open it in your browser. The first index build downloads/caches the embedding model and can take longer than later runs. The generated `chroma_db/` folder is local runtime data and must not be included in the ZIP.

The app checks whether source documents or chunk settings changed. You can also use **Rebuild knowledge index** in the sidebar or run `python rebuild_index.py`. Editing a Markdown file alone does not rewrite vectors already saved in Chroma; the index must be rebuilt so the updated text and metadata are embedded and stored.

## Run the required test set

```bash
python run_test_set.py
```

The script sends all 20 predefined questions through the actual retrieval and LLM pipeline and writes actual answers, retrieved source filenames, model-cited filenames, citation filename matching, and supporting excerpts into `results/test_set_and_results.csv`. It initially labels cases `REVIEW`, not `PASS`. Read each answer and inspect whether the cited excerpt actually supports the claim; filename matching alone is not proof. Manually change `pass_fail` to `PASS` or `FAIL` and record a short reason for every failure. Never report unrun tests as passed.

The test categories are 5 direct factual, 4 paraphrased, 3 two-document, 3 old/current comparisons, 3 missing-answer, and 2 prompt-injection tests.

## Run retrieval and chunk-size experiments

```bash
python run_retrieval_experiments.py --with-answers
```

The `--with-answers` option makes 24 additional Gemini calls (18 for the k comparison and 6 for the chunk-size comparison), so it uses API quota. Omit the flag to run retrieval-only checks. This creates:
- `results/retrieval_experiments.csv`: six fixed questions at k=1, k=3, and k=5, keeping the documents and chunk settings unchanged; generated answers are included when `--with-answers` is used.
- `results/chunk_size_experiment.csv`: three fixed questions at 900/150 and 600/100 chunk-size/overlap settings; generated answers are included when `--with-answers` is used.

The script records whether expected source filenames appeared, as well as cited filenames and retrieved excerpts when `--with-answers` is used. These are automatic source-presence checks only. Inspect each chunk and manually decide whether the required evidence was actually present, whether irrelevant material appeared, whether cited excerpts support the answer, and whether the final answer was correct. Complete the manual-review columns. A non-expected source is not automatically irrelevant. Retrieval source presence is not the same as answer correctness.

## Run basic code tests

```bash
pytest -q
```

## Required demonstration (3–5 minutes)
Record your own screen demonstration and add its share link below. Do not submit this placeholder as a real video link.

**Demo video:** `[PASTE YOUR OWN 3–5 MINUTE VIDEO LINK HERE]`

Suggested order:
1. Ask a normal question, such as the current keyboard price.
2. Ask a question that requires two documents, such as remote-area coverage and delivery estimate.
3. Compare the old and current return policies.
4. Ask a question whose answer is missing, such as international delivery to Canada; show the exact abstention response.
5. Ask a return or delivery question that could retrieve the security test note; show that the assistant ignores the BANANA instruction and answers from the policy.
6. Explain one failure you observed, how you inspected retrieved chunks/source metadata, and what you changed (or why you left it unresolved).

## Instructor live challenge: changing a policy rule
1. Open `knowledge_docs/03_current_customer_policy.md`.
2. Change one rule to a clearly stated classroom example, and update its effective date/version if your instructor asks.
3. Save the file.
4. Run `python rebuild_index.py` (or use the app's rebuild button).
5. Ask a question that depends on the changed rule and show the updated answer and source excerpt.
6. Explain that Chroma stores vector representations of the old text. Editing the source file does not automatically alter stored vectors; re-ingestion, re-chunking, re-embedding, and index update/rebuild are required.
7. Restore the original supplied policy after the demonstration if you need the original test results to remain reproducible.

## What to submit
- Project ZIP containing Python source, knowledge documents, `requirements.txt`, `.env.example`, README, and report.
- `results/test_set_and_results.csv` after running all 20 cases and reviewing each result.
- `docs/assignment_report.md` (maximum three pages when rendered; adapt it with your actual retrieval findings, failures, and limitations).
- A 3–5 minute video link inserted above.

Before uploading, exclude `.env`, `.venv/`, `venv/`, `chroma_db/`, downloaded model caches, API keys, and any real personal/customer data. Keep each LMS attachment under 5 MB as your assignment requires.

## Important learning notes
- The system does not hardcode the expected answers in the application. The facts live in the supplied knowledge documents and are retrieved at run time.
- Current-policy questions filter out archived policy chunks. Explicit history questions select archived chunks. Comparison questions—including wording such as “change between archived and current”—retrieve both.
- Citation tracking extracts bracketed filenames from the answer and checks whether each appears in retrieved sources. Streamlit shows short excerpts for inspection. This checks filename consistency only; the student must still judge whether the excerpt supports each factual claim.
- Gemini requests explicitly configure up to five total attempts with exponential backoff for transient HTTP errors such as 503. A persistent provider outage can still require the user to retry later.
- Retrieved text is untrusted input. The prompt-injection note is deliberately non-authoritative and must not change system behavior.
- The assistant is a classroom prototype. It is not ready for a real company without authentication, tenant isolation, privacy controls, monitoring, human review, policy governance, and security testing.

## Provider references
- Google Gemini API pricing and free-tier availability: https://ai.google.dev/gemini-api/docs/pricing
- Google Gemini model IDs and availability: https://ai.google.dev/gemini-api/docs/models
- Google Gemini API error codes and retry guidance: https://ai.google.dev/gemini-api/docs/api-errors
- Google Gen AI Python SDK: https://googleapis.github.io/python-genai/
- Chroma documentation: https://docs.trychroma.com/
- Streamlit documentation: https://docs.streamlit.io/
project demo link :https://drive.google.com/file/d/1i3kn-zU07tB4gkunPPOc7ULey3nJpcZF/view?usp=sharinghttps://drive.google.com/file/d/1i3kn-zU07tB4gkunPPOc7ULey3nJpcZF/view?usp=sharing
