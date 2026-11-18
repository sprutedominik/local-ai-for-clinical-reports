# Local AI for clinical reports

Report drafting and record search for a Swiss psychotherapy practice, running on a server inside the practice. No patient data goes to a cloud AI service.

This is a write-up. The source code belongs to the practice and is not public.

## Why local

Psychotherapy records fall under Swiss professional secrecy (Art. 321 of the Swiss Criminal Code) and the Federal Act on Data Protection. The practice decided early that patient data stays in the building. The server already ran the practice app, so it got the models too.

## Setup

- Ollama runs natively on a mini PC with an AMD Ryzen AI Max+ 395 and 128 GB of unified memory, 64 GB of it reserved for the integrated GPU.
- Qwen models in the 30 to 35 billion parameter range for reports and chat, nomic-embed-text for embeddings. I compared models on speed and memory, chose separate ones for reports and chat, and decided how they connect to the app and how they are deployed.
- Tuned for the hardware: 32k context, models kept loaded, about 75 tokens per second, up to three models in GPU memory at once. Timeouts for long reports fit a machine without a dedicated GPU.
- Models and search run on the practice's own server and are used by the practice app and an internal chat interface (Open WebUI) for the head of practice.

## Report drafting

Therapists write reports to insurers and doctors at fixed points of a treatment. The app drafts them from the record. The model writes the text; most of the work is in what happens before and after that call.

1. **Pick a patient.** Recipient, number of sessions and diagnoses come from the record and the open case. The report type is suggested.
2. **See what is missing.** A list shows which details the record does not have yet. Filling them in improves the draft.
3. **Filter the input.** Internal notes and supervision content are removed before the model sees anything. Each section of a session note can be marked as allowed or not allowed in reports.
4. **Fix what must not vary.** Diagnoses, treatment line, setting and closing are inserted by code, not generated.
5. **Check the output.** The draft is checked for numbers, dates and ICD codes that do not appear in the record. Gaps are marked "[bitte ergänzen]" for the therapist.
6. **Keep the model strict.** The instructions ask for "no information available" instead of guesses, sampling is set close to deterministic, and diagnoses are suggestions only.
7. **A person signs.** Draft, review, approved, sent. Only the head of practice approves, and sending is blocked without the patient's release from confidentiality. Output as a Word document.

## Record search (RAG)

A separate Python service with FastAPI and an embedded Qdrant store, embeddings through Ollama. It indexes structured record content from the app. Access is limited by role.

## Data import from documents

Patient details from PDF and Word files can be extracted by the local model instead of being typed in by hand.

## What I took away

Safety in clinical AI depends less on the prompt than on the code around it: what goes in, what is fixed in place, and what is checked before a person signs.

## Stack

Ollama · Qwen · nomic-embed-text · Python · FastAPI · Qdrant · Node.js · Express · Open WebUI · Pandoc
