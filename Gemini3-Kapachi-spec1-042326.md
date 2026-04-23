Technical Specification: KNOWLEDGE AGENT v3.0 (AI-Driven Obsidian Knowledge Transformation Protocol)
1. Executive Summary & Product Vision
1.1 Product Overview
The Knowledge Agent v3.0 is an advanced, highly interactive, AI-driven single-page application (SPA) designed to act as an intelligent intermediary between raw, unstructured data sources (PDFs, plain text, Markdown, CSVs) and highly structured personal knowledge management (PKM) networks, specifically targeting Obsidian. It moves beyond standard document conversion by employing state-of-the-art Large Language Models (LLMs) to perform semantic reconstruction, automated graph interlinking, and multi-modal image-to-text generation.
The application is built leveraging a modern React frontend architecture, styled with Tailwind CSS, and powered by the cutting-edge @google/genai SDK to interface directly with Gemini 3.1 Flash and Gemini 2.5 Pro models. It is tailored to process large context windows, handle binary PDF manipulation entirely client-side, and dynamically stream cognitive transformations back to the user through an aesthetic, telemetry-heavy user interface designed for power users and knowledge architects.
1.2 Core Value Proposition
Traditional document conversion tools (such as Pandoc or automated PDF extractors) often fail to preserve semantic meaning, resulting in broken line breaks, loss of hierarchical headings, and orphaned images. The Knowledge Agent v3.0 solves this by bridging client-side document processing with an LLM-driven "Semantic Reconstructor." It automatically identifies and repairs document structures, translates visual assets into descriptive Alt-Text for markdown compatibility, and proactively generates bidirectional WikiLinks ([[Linked Concept]]) and hierarchical tags (#domain/sub-domain) tailored for Obsidian's graph database.
1.3 Target Audience
Knowledge Workers & Researchers: Individuals who process diverse formats (academic PDFs, social media JSON exports, raw pasted data) and map them into comprehensive research databases.
Obsidian Power Users: Users who rely on interconnected PKM systems, Zettelkasten methodologies, and bidirectional linking to generate second-brain architectures.
Data Analysts & Content Curators: Professionals needing rapid summarization, style-shifting, and logic-checking workflows applied to bulk ingested texts.
2. System Architecture & Technology Stack
2.1 High-Level Architecture
The application runs as a purely Client-Side Rendered (CSR) Single-Page Application. There is no traditional backend database or proprietary backend API orchestrating the document conversions; instead, the browser acts as the primary compute node for document ingestion and pre-processing, while the Google GenAI Cloud serves as the cognitive backend.
Presentation Layer: React 19 functions as the view library, utilizing functional components and hooks for reactive DOM updates.
Styling Layer: Tailwind CSS 4.x provides atomic utility classes for a highly responsive, aesthetic-switching UI layout.
Pre-processing Layer: Browser-native File APIs (FileReader, Blob, URL.createObjectURL) combined with pdf-lib handle data ingestion, Base64 encoding, and binary PDF byte manipulation.
Cognitive Communication Layer: The @google/genai TypeScript SDK establishes an asynchronous, chunk-based streaming connection to Google's LLM endpoints using Server-Sent Events (SSE).
2.2 Technology Stack Details
Core Framework: React 19.0.0, ReactDOM 19.0.0
Build Tooling: Vite 6.2.0 (configured for ESM optimization and rapid hot-module replacement in dev, optimized static bundling in production).
Styling & CSS: TailwindCSS 4.1.14 (with Vite plugin architecture), utilizing inline custom properties for scrollbars and aesthetic overrides.
Icons: lucide-react 0.546.0 (providing scalable SVG icons).
PDF Manipulation: pdf-lib (allows loading, splitting, and saving PDF ArrayBuffers directly in the browser).
AI Integration: @google/genai 1.29.0 (official modern SDK for interacting with Gemini models, supporting inlineData multi-modal parts).
Language Environment: TypeScript 5.8 (Target ES2022, strict module resolution).
3. Core Functional Specifications
3.1 Data Ingestion Mechanisms
The application supports a dual-modal ingestion mechanism via the "Input Data Source" panel.
3.1.1 File Upload Pipeline
Users can upload multiple files simultaneously via an <input type="file" multiple> element disguised as a drag-and-drop zone.
Text-based Files (TXT, MD, CSV): The application reads the files using the native asynchronous File.text() API. The file metadata (name, size, type) and the extracted string are pushed to the files state array.
PDF Documents: The application intercepts application/pdf MIME types. Instead of extracting raw text (which often garbles PDF streams), it utilizes a FileReader implementation (fileToBase64) to convert the binary Blob into a Base64 encoded string. The file is saved in the state with a distinct inlineData property ({ mimeType: 'application/pdf', data: base64 }), allowing it to be securely transmitted to the multimodal Gemini endpoint.
3.1.2 Paste Text Pipeline
Provides a <textarea> bound to the pastedText state. When users enact the "Add to Queue" function, the application calculates the pseudo-file size using new Blob([pastedText]).size, generates a unique alphanumeric tracking ID, and injects it into the files array as a virtual text file (e.g., Pasted_Text_1a2b3c.txt).
3.2 PDF Trimming & Sub-setting Engine
To mitigate LLM token limits and process targeted sections of large documents, the system provides an integrated PDF page trimmer.
Trigger: When a PDF is queued, a "Trim PDF" button becomes available. Clicking it sets the trimmingFileId state, revealing a sub-menu to define trimStart and trimEnd pages.
Execution Lifecycle:
The original File object stored in the state is parsed into an ArrayBuffer.
PDFDocument.load(arrayBuffer) via pdf-lib parses the binary PDF tree.
A bounds check verifies the trim indices against pdfDoc.getPageCount().
A new blank PDFDocument is instantiated.
copyPages() maps the requested range and injects them into the new document.
save() serializes the document back into a Uint8Array.
A new Blob and subsequent File object are instantiated representing the trimmed PDF.
The file is mapped back to Base64 to update the application state queue.
Artifact Download: The application dynamically generates an object URL (URL.createObjectURL(trimmedBlob)), binds it to a virtually constructed <a> anchor tag, triggers a programmatic .click(), and initiates a local download to the user's filesystem. The object URL is successfully zeroed-out via URL.revokeObjectURL() to prevent memory leaks.
3.3 LLM Orchestration & "Magic" Protocols
The application delegates all high-level cognitive tasks to the Gemini AI models through explicit, tailored prompt engineering.
3.3.1 Transformation Pipeline (Macro-Synthesis)
When "Execute Transformation" is invoked:
It aggregates all ingested textual data and Base64 inlineData PDF parts into an array of type Part[].
It prefixes the payload with a base directive: "Please summarize and synthesize the following documents into a comprehensive Markdown report. Maintain highly structural formatting."
The request is dispatched via the streamGemini adapter.
The onChunk callback captures the Server-Sent Events stream, incrementally concatenating the summary state, triggering reactive UI updates mimicking a live-typing effect.
3.3.2 WOW AI Features (Magic Protocols)
Users can invoke specialized micro-agents: Semantic Map, Logic Checker, Entity Linker, Style Shift, Auto-Cite, and Insight Flux.
Mechanics: Instead of re-processing the raw files, the magic protocol acts upon the derived summary output. It encapsulates the current context inside a new prompt boundary, instructing the AI to "apply the [Magic Name] protocol to this context." This enables iterative, chained refinements without re-uploading or re-analyzing massive base PDFs.
3.3.3 Infinite Chat Agent Interface
Provides a conversational UI embedded at the bottom of the Output Preview. It concatenates the current summary state with the user's chatPrompt, allowing users to query, expand, or rewrite specific sections of the generated intelligence dynamically.
4. Deep Dive: The Obsidian Enhancement System Prompt
The core intelligence of the application resides in its meticulously crafted systemPrompt. This prompt essentially programs the Gemini model to act as an "AI-driven Obsidian Knowledge Transformation Tool" acting on structured JSON / XML / Text / PDFs.
4.1 AI Semantic Reconstruction (語意修復與結構重組)
The LLM is instructed to identify and repair arbitrary PDF line breaks and broken sentences. It employs natural language heuristics to stitch together paragraphs spanning multiple pages. Furthermore, it analyzes hierarchical context to invent H1, H2, and H3 Markdown headers where visual styling in the original document implied sections but lacked textual tags. It is also instructed to act as an aggressive filter, intentionally omitting page numbers, repeating headers, and localized indexes to yield a pure "Knowledge Base" document.
4.2 Multi-modal Vision-to-Alt Text (多模態圖像視覺描述)
When provided with natively rich documents containing charts or images, the LLM is instructed to generate rich Alt-Text definitions syntax formatted for Obsidian (![[image_name.png|Generated Alt Text]]). For visual charts detected during multimodal payload ingestion, it is programmed to interpret the trend line or data points and output an implicit summary immediately following the image link.
4.3 Automated Graph Interlinking (自動化知識圖譜關聯與標籤)
The most valuable workflow integration for Obsidian lies in the LLM's capacity to synthesize an intelligent knowledge graph topology.
Conceptual WikiLinks Formation: The model recognizes domain-specific proper nouns, entities, and theories resulting in programmatic wrapping of terms in double brackets [[...]]. This instantly allows the Obsidian indexer to recognize forward-links and unlinked mentions.
YAML Frontmatter Injection: The system dictates the model should write compliant Zettelkasten / Obsidian YAML frontmatter block at the absolute top of the markdown response, containing automatically deduced properties like tags, aliases, date, and type.
5. UI/UX Architecture & Telemetry Dashboard
The user interface eschews traditional corporate SaaS styling in favor of a dense, technical, "Agentic" dashboard layout, heavily leveraging Tailwind CSS grids, flexboxes, and monolithic fixed-height panels.
5.1 Main Layout Skeleton
Header: A fixed 64px (h-16) navigation bar housing the application title, spinner indicator, language toggle, and off-canvas Configuration Drawer trigger.
Left Sidebar Controls: A 320px fixed-width column designed for file ingestion, Input mechanism toggles, PDF tools, Magic action grids, and the primary Action constraints (Execute / Stop).
Dual-Pane Main Content:
Top Row (Process & Telemetry): A 224px (h-56) strip split between the "Processing Queue" (showing loaded files, size, status, and PDF trim inputs) and the "Live Telemetry" window.
Bottom Row (Synthesized Intelligence): A flexible (flex-1) window occupying the remaining viewport real estate. It houses the visual theme toggles, the streaming generated Markdown context, and the interactive Chat input loop.
5.2 Aesthetic Maps Environment
The application features a dynamically injectable theme engine leveraging the effectClasses object. By triggering state changes via small color-coded orbit buttons, the user can override the background, text color, borders, and shadowing of the main preview container:
nordic: Clean, minimal white and dark gray.
matrix: Deep black with neon green typography, glowing inset shadows, and mono-spaced fonts.
pulse: Blue-tinted transparency with an aggressive infinite CSS pulse animation spanning 4 seconds to emphasize processing states.
cyber: Dark slate and neon red accents representing a cyberpunk UI.
glass: Mac-OS inspired translucent backdrop-blur with white overlays.
ethereal: Soft gradient blends of indigo and purple designed to mimic modern AI interfaces like Google DeepMind prototypes.
5.3 Live Telemetry Module
Designed to provide synthetic, narrative feedback to the user to signify background computations. It utilizes an array of string arrays (liveLogs).
Reactive Auto-Scroller: A useRef binds to an invisible boundary <div ref={logsEndRef} />. A useEffect hook listening to the liveLogs dependency array programmatically invokes .scrollIntoView({ behavior: 'smooth' }), ensuring the terminal UI continually tracks the latest output.
Simulated Hardware Metrics: Small static UI elements showing synthetic CPU and RAM usage depending entirely on the boolean isGenerating state to transition from idle (0.8GB / 12%) to load (1.4GB / 84%).
Color Coded Logs: The inline renderer checks the log string array for keywords ('ERROR', 'WARNING', 'SUCCESS') and dynamically injects Tailwind color utility classes (text-red-400, text-green-400).
6. Data Flow & State Management Dictionary
The application strictly utilizes React's functional component hooks for local state architectures.
6.1 State Variable Dictionary
State Variable	React Type	Description & Lifecycle
lang	string ('zh'|'en')	Determines the overarching language instruction context (though mostly UI toggle in current iteration).
showConfig	boolean	Toggles the absolute-positioned LLM configuration drawer panel.
model	string	Defines the targeted Gemini endpoint (e.g., gemini-3-flash-preview, gemini-2.5-pro). Passed directly to streamGemini.
systemPrompt	string	Holds the meticulously detailed Obsidian-focused multiline template literal used as the overarching system instruction for the LLM.
files	FileItem[]	An array of extracted metadata. Crucial fields include id (tracking), inlineData (for Base64 PDF chunks), text (for raw plaintext datasets), and fileObj (to retain access to the Blob buffer for Trimming).
summary	string	The primary reactive buffer capturing the output of the LLM. Initially populated with placeholder text; heavily mutated by string concat in the onChunk callback.
isGenerating	boolean	The master lock state. Used across the UI to disable buttons, trigger spinners, display 'Stop' buttons, and alter telemetry UI.
liveLogs	string[]	The chronological array of string logs rendered in the Telemetry terminal.
chatPrompt	string	Binds to the chat input value traversing the Controlled Component pattern.
inputType	string ('upload'|'paste')	UX toggle state modifying which input mechanisms are functionally available.
pastedText	string	Temporary buffer tracking raw pasted input before it is encoded to pseud-files in the Queue.
trimmingFileId	string | null	Stores the active correlation id of the PDF being actively cut. Controls the display of inline edit tools.
trimStart / trimEnd	string	Temporary string buffers holding numeric pagination data for the pdf-lib indices.
visualEffect	string	Selects the active UI aesthetic map key from the effectClasses local object.
6.2 Reference Hooks (Mutable APIs)
abortControllerRef: (MutableRefObject<AbortController | null>). React state updates asynchronously, meaning triggering a cancellation of a fetch request requires an imperative mechanism. The AbortController handles signal rejection mid-stream directly to the GenAI SDK. Storing it in a Ref ensures it persists across re-renders without triggering view updates itself.
logsEndRef: (RefObject<HTMLDivElement>). Maps directly to a DOM element to interface with the native DOM API scrollIntoView().
6.3 Gemini Stream Flow Mechanics
The streamGemini adapter acts as a crucial abstraction layer over the official SDK:
streamGemini is invoked with parts (an array containing text directives and object inlineData), model, config, target AbortSignal, and a callback function.
Inside streamGemini, ai.models.generateContentStream establishes an HTTP/2 SSE link.
An asynchronous iterable for await (const chunk of responseStream) is invoked.
On each iterant loop, the AbortSignal is checked. If aborted, an internal break terminates processing early.
If text passes, the onChunk callback fires, lifting state back to App.tsx, triggering an incremental set state (setSummary(prev => prev + chunk)).
React's virtual DOM diffs the summary string and efficiently updates the large <pre> block rendering the final text.
7. Configuration Management & Extensibility
7.1 Configuration Drawer
The app provides a gear-icon triggered configuration layer exposing underlying LLM parameters.
Model Switcher: A controlled <select> dropdown dynamically switching the active Gemini model via state updates.
Prompt Injection Editing: A controlled <textarea> permitting the user to hot-swap the internal directives of the Zettelkasten/Obsidian instruction set, fundamentally altering how the AI behaves during subsequent transformations.
7.2 The Interface API Wrapper (/lib/gemini.ts)
The API wrapper is deliberately decoupled from the React component tree.
Environment Hydration: Evaluates Vite environment variables natively (process.env.GEMINI_API_KEY) at initialization.
SDK Instantiation: Creates an immutable ai singleton mapping to new GoogleGenAI().
Error Boundaries: Explicitly wraps the generate call in defensive try/catch blocks. It parses common LLM disruption events (e.g., AbortError due to manual cancellations or API network drops), ensuring the primary process doesn't unmount or crash due to uncaught promises, but safely triggers a console abort mechanism.
8. Security & Performance Considerations
8.1 Performance Optimizations
Client-Side Processing: By isolating pdf-lib execution purely inside the browser Thread (via V8 Javascript Engine execution against ArrayBuffer), the application eliminates costly network uplinks transferring massive binary files back and forth to an intermediate server, drastically lowering latency and operational server ingress/egress costs.
Object URL Revocation: During PDF trimming workflows, creating Blob URLs creates un-garbage-collected mappings in browser memory strings. The script explicitly invokes URL.revokeObjectURL(url) immediately following the automated .click() anchor injection, preserving critical heap layout optimization natively.
In-Memory Part Mapping: Retaining Base64 encoded PDFs within React State (files[].inlineData) can become expensive in memory footprints for large documents over typical browser bounds (e.g. Chrome's multi-GB page bounds). However, processing subsets ensures token limits aren’t breached when transferring to the API.
8.2 Security Architecture
Stateless Operation: Since this is a Client Side SPA, uploaded sensitive data (TXT, PDF) inherently never leaves the user's browser, nor is it sent to intermediate storage architecture. The only transmission path is directly outgoing across HTTPS/TLS 1.3 straight to standard Google Gemini LLM API servers for token processing.
API Key Security Context: As per AI Studio environment variables, the GEMINI_API_KEY is injected via Vite's define config strictly at build time or via secured environment platforms. Due to the frontend design pattern of the SPA, this model requires environment proxy forwarding or an acknowledgment that utilizing client-configured variables relies entirely upon standard platform encapsulation controls rather than purely secure-backend secrecy.
Input Sanitization Risks: Currently, output generation pushes directly into a primitive <pre> generic text output. While mitigating HTML rendering exploits (as it simply strings literal text), if an external markdown library like react-markdown were to be implemented later, strict remark plugins would be explicitly required to strip potential <script> tags injected via hallucinated AI outputs.
9. Deployment & Operational Blueprint
9.1 Build Chain Configuration
The Vite bundler dictates the production pipeline:
react() plugin hooks inject React architectural bindings.
tailwindcss() configures the newly introduced v4 lightning fast JIT style compilation directly inside the plugin.
A path alias @ simplifies large-scale file resolutions in deeper trees.
hmr (Hot Module Replacement) controls monitor the development plane but handle conditional environmental blocking if required to avoid DOM redraw loops.
9.2 Build Lifecycle
When transitioning to production (npm run build):
TypeScript Checker: tsc executes checking types across interfaces to assure rigorous compile time validations (e.g., matching the FileItem structure correctly mapped to React local states).
Rollup Packaging: Vite compresses JSX components, groups Vendor libraries (like pdf-lib, @google/genai which total considerable KB chunks), chunks them across HTTP/2 mechanisms, and injects cache-busting hashing identifiers.
Static File Server: A generic Node/Express container routing engine serves the resulting dist folder files unconditionally via wildcard * routing, ensuring client-side location structures behave smoothly without standard HTML 404ing server paths.
10. Future Design Architecture & Roadmap Opportunities
Phase 2 Roadmap Additions
Web Worker Delegation: Migrating intensive pdf-lib buffer mutations and large Base64 encodings out of the main UI thread into a Dedicated Web Worker (worker.js). This would prevent any possibility of UI jank/stuttering while processing 100+ page datasets.
Persistent IndexedDB Storage: Utilizing LocalStorage or IndexedDB frameworks (like LocalForage) to retain configuration templates, system prompt histories, and visual effect preferences even after full page refreshes.
React-Markdown Renderer Integration: While raw text outputs are robust, visualizing tables, mathematical block quotes, and Markdown tables requires a robust abstract syntax tree parser like react-markdown combined with rehype-raw and Tailwind utility markdown plugins (prose lg:prose-xl).
Semantic Chunking Logic: Implementing tokenizer checks prior to calling streamGemini. If processing 10 large PDFs, automatically chunking text data utilizing overlapping window loops and dispatching independent asynchronous calls, mapping them together programmatically before displaying the resultant compilation to bypass 1-million-token limitations if necessary.
Comprehensive Follow-Up Questions
To ensure deep alignment regarding technical feasibility, operational constraints, and edge-case handling across architecture loops, here are 20 critical follow-up questions for the development and product teams:
(System Architecture & Scale)
Base64 Payload Limits: A single 20MB PDF results in over ~26MB of raw Base64 string data. How will the application handle processing bottlenecks or out-of-memory errors in lower-tier browsers (e.g., mobile Safari) when generating these arrays inside the main thread?
Web Worker Offloading: Given the intensive data mapping happening during data ingestion arrays (await Promise.all()), what is the engineering timeline for delegating FileReader tasks and pdf-lib operations to a dedicated Web Worker?
Chunked Prompt Assembly Limits: The LLM receives all files concatenated into an array of Part[]. Is there a native pre-flight tokenizer boundary check defined to halt requests that exceed the designated Gemini Multi-Modal input token thresholds before wasting network cycles?
State Hydration: When a user refreshes the page, the heavily tailored System Prompt and files array resets entirely. Is there a business mandate to persist these parameters across active sessions via IndexedDB or LocalStorage wrappers?
Obsidian Integration Tethers: The app outputs beautifully formed Zettelkasten text. Do we intend to explore directly integrating Obsidian local URI handlers (e.g., obsidian://new?name=...&content=...) into the UI so users can transmit summaries with a single click?
(Multi-Modality & Gemini APIs)
6. PDF OCR Processing Constraints: When transferring inlineData PDFs representing scanned images without textual layers, what specific Google GenAI directives currently ensure that optical character recognition is forced over native metadata extraction routines?
7. Alternative File Ingestion: Currently, only PDFs handle the sophisticated inlineData path, while everything else gracefully falls back to explicit Text(). Should standard image components (.jpg, .png) similarly be injected into the inlineData parts array for multi-modal logic tracking?
8. Prompt Injection Mitigations: If an ingested third-party PDF contains maliciously crafted "ignore previous instructions" sentences embedded in microscopic white text, what boundaries protect the primary Obsidian logic instructions defined in the System prompt?
9. Streaming Artifacts: streamGemini appends strings raw to the React state. When rendering markdown, how do we elegantly handle parsing intermediate, broken, or half-closed Markdown syntax objects (**, ```) dynamically streaming before the final abortSignal completes?
10. Model Fallback Logic: The user can toggle between Flash and Pro. Does the logic layer currently support automated cascading fallbacks (e.g., fallback to Flash if Pro encounters rate-limiting or quota errors)?
(UI/UX & Accessibility)
11. Telemetry Dashboard Verbosity: The liveLogs telemetry array grows indefinitely on long sessions. At what state index or length does the application truncate or slice ancient logs to avoid continuous DOM node leak bloat?
12. Virtualization of Output: The primary <pre> UI element can rapidly reach tens of thousands of pixels in height based on LLM outputs. Would integrating tools like React-Window virtualize rendering constraints significantly enough to justify the complexity overhead?
13. Mobile Responsiveness Considerations: The core structure extensively utilizes fixed-sidebar paradigms (w-[320px]). While defined flexibly, will power users executing LLM workflows on mobile touchscreen endpoints necessitate explicit collapsible drawer architectures for the left UI column?
14. Accessibility Control Check: Tailwind themes provide intense visual styling (e.g., Matrix neon styling). For users managing extreme visual tracking conditions, does the framework define automated WCAG contrast fallback ratios via native @media (prefers-contrast: more) injections?
15. Input Modality Constraints: For the paste text boundary mechanism (handleAddPaste), what happens when a user attempts to paste a binary Blob image from their clipboard into the textarea directly? Is a PasteEvent listener interceptor required?
(PDF Engine & Functional Mechanics)
16. Encrypted PDF Bypass: When pdf-lib targets a secured, password-encrypted PDF to parse the page structures for the trimming UI tools, how does the try/catch loop explicitly warn the user, and are password prompts an expected iteration feature?
17. PDF Page Trim Boundaries: Are there scenarios whereby trimming subsets of documents carrying massive vector infographics destroys the PDF cross-reference table, generating Corrupted Document alerts locally post-save?
18. Download Artifact Mechanics: Constructing an anchor tag <a href> dynamically assumes modern File API implementations. How should the platform behave within restrictive mobile WebView constraints (e.g., embedded social browsers) where local filesystem downloads frequently fail silently?
(Security & Deployment)
19. Client-side API Key Masking: Given the CSR architecture directly invokes process.env.GEMINI_API_KEY, how do we reconcile the implicit exposure of project API tokens on the client boundary against traditional backend/BFF (Backend-For-Frontend) reverse proxy routing strategies?
20. Dependency Surface Area Checks: Handling explicit raw imports from generic HTTP locations requires strict package lock auditing. What CI/CD tools scan the .package-lock mapping for pdf-lib and lucide-react vulnerable dependencies ahead of static Rollup output generations?
