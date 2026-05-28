You are GitHub Copilot running inside VS Code. I want you to DESIGN and GENERATE all the files for a new custom agent called **PPT Creator**, following GitHub Copilot custom agent and prompt-file patterns.

### GOAL

Create a custom agent that can:
- Take local data (Excel, CSV, Word, TXT, or pasted text)
- Generate PowerPoint presentations (.pptx) using Python and python-pptx
- Build flow diagrams using shapes and connectors (no images)
- Add text-only â€œIMAGE PLACEHOLDERâ€ boxes where visuals would help
- Process everything **locally** in the client network (no external HTTP calls or websites)

I will then save the generated content as:
- `.github/agents/ppt-creator.agent.md`
- `.github/prompts/ppt-from-excel.prompt.md`
- `.github/prompts/ppt-from-text.prompt.md`
- `.github/prompts/ppt-flow-diagram.prompt.md`
- Optionally, a `generate_ppt_example.py` script in the repo root

### HARD CONSTRAINTS (CLIENT ENVIRONMENT)

1. On the client side, we ONLY have:
   - VS Code
   - GitHub Copilot (cloud LLM)
   - Local Python environment
2. All data (files, PPTs) must be processed **locally**, using:
   - Local files and folders
   - Tools available to the agent such as `read/file`, `runCommand`, `edit`
3. Absolutely **no external websites or APIs**:
   - Do NOT reference Unsplash, Pexels, Google Images, or any external domain
   - Do NOT generate HTTP/HTTPS URLs
4. Python scripts must:
   - Use only local libraries (`python-pptx`, `pandas`, `openpyxl`, `python-docx`)
   - Not call any network APIs
5. Image handling:
   - We CANNOT generate images via the LLM or APIs
   - Instead we will add text-only â€œIMAGE PLACEHOLDERâ€ shapes, with short instructions like:
     - â€œIMAGE PLACEHOLDERâ€
     - â€œInsert product screenshot here from internal asset libraryâ€
     - â€œInsert the image here and then delete this boxâ€

### WHAT YOU MUST GENERATE

Generate ALL of the following as separate clearly labeled sections, so I can copy each into a file:

1. **PPT Creator agent file**: `ppt-creator.agent.md`
   - YAML frontmatter:
     - `name: PPT Creator`
     - `description`: explain it creates PPTX from Excel/CSV/Word/TXT/direct data, creates flow diagrams using python-pptx shapes, and adds generic local-only image placeholders
     - `argument-hint`: e.g., "Provide your data or describe the PPT you want"
     - `tools`: at least `read/file`, `read/terminalLastCommand`, `edit`, `runCommand`, `search/codebase`
     - `model`: pick a reasonable default, e.g. `claude-opus-4.6` or equivalent, but we will rely on GitHub Copilotâ€™s configured model
   - Optional: `handoffs` section with 2â€“3 convenience handoffs back to itself (e.g., â€œBuild PPT from Fileâ€, â€œCreate Flow Diagram PPTâ€, â€œCreate PPT from Direct Dataâ€)
   - Markdown body that describes:
     - CAPABILITIES:
       - File ingestion: Excel, CSV, Word, TXT, direct data
       - Flow diagrams: use python-pptx shapes (rectangles, diamonds, ovals, arrows)
       - Image placeholders: text-only boxes with instructions, no external sites
       - Slide templates: title, section, bullets, tables, etc.
     - WORKING PRINCIPLES:
       - ALWAYS: process data locally, treat data as sensitive, use python-pptx, use local scripts, add image placeholders where useful
       - NEVER: suggest external sites, generate URLs, call HTTP APIs, use any image generation APIs
     - STEP-BY-STEP WORKFLOW:
       - Ask clarifying questions (source, purpose, slides count, theme)
       - Read data via `read/file` or direct text
       - Propose a slide plan and wait for user approval
       - Generate a `generate_ppt.py` script with:
         - Imports: python-pptx, pandas, openpyxl, python-docx as needed
         - Helper functions: title slide, bullet slide, table slide, flow diagram slide, image placeholder helper
         - Reading from Excel/CSV/Word/TXT
         - Saving a `.pptx` file
       - Run script via `runCommand`
       - Report output path and ask for refinements
     - PYTHON CODE PATTERNS:
       - Base PPT setup with a navy professional theme
       - `add_title_slide`, `add_content_slide`, `add_table_slide`, `add_flow_diagram`
       - `add_image_placeholder` that creates a dashed rectangle with:
         - Title: â€œIMAGE PLACEHOLDERâ€
         - Short description argument
         - Instruction: â€œUse an approved local image from your internal asset library. Insert the image here and then delete this box.â€
       - No network calls in these examples
     - ERROR HANDLING:
       - File not found, unsupported format, missing libraries, empty data, too many rows
     - OUTPUT FORMAT:
       - A summary message listing file path, slide count, and a bullet list of slide titles

2. **Prompt file 1**: `ppt-from-excel.prompt.md`
   - YAML frontmatter:
     - `mode: agent`
     - `agent: ppt-creator`
     - `description: Create a PPT from an Excel or CSV file`
   - Body:
     - Ask for `${input:filePath:Enter file path (e.g., ./data.xlsx)}`
     - Ask for `${input:title:Enter presentation title}`
     - Instruct the agent to:
       - Read the Excel/CSV
       - Create:
         - Title slide
         - One table slide per sheet or logical section
         - Summary slide with key totals
         - Image placeholders where helpful
       - Use navy-blue theme
       - Save as `./output_presentation.pptx`

3. **Prompt file 2**: `ppt-from-text.prompt.md`
   - YAML frontmatter:
     - `mode: agent`
     - `agent: ppt-creator`
     - `description: Create a PPT from a Word document or text file`
   - Body:
     - Ask for `${input:docPath:Enter file path (e.g., ./proposal.docx or ./notes.txt)}`
     - Instruct the agent to:
       - Read Word/TXT
       - Map headings to slide titles, body to bullets, tables to table slides
       - Add image placeholders where visual aids would help
       - Use navy-blue theme
       - Save as `./document_presentation.pptx`

4. **Prompt file 3**: `ppt-flow-diagram.prompt.md`
   - YAML frontmatter:
     - `mode: agent`
     - `agent: ppt-creator`
     - `description: Create a PPT slide with a flow diagram`
   - Body:
     - Ask for:
       - `${input:diagramTitle:Flow diagram slide title}`
       - `${input:steps:Enter steps separated by commas (e.g., Start, Collect Data, Analyze, Decision?, End)}`
     - Instruct the agent to:
       - Build a PPT with:
         - Title slide
         - Flow diagram slide
         - Thank-you slide
       - Flow diagram rules:
         - First and last steps â†’ oval shapes
         - Steps containing "?" â†’ diamond shapes (decisions)
         - Others â†’ rounded rectangles
         - Connect shapes with arrows
       - Add image placeholders only if needed, with generic internal instructions
       - Save as `./flow_diagram.pptx`

5. **Optional example script**: `generate_ppt_example.py`
   - A self-contained Python script that:
     - Uses python-pptx to generate:
       - Title slide
       - Agenda slide
       - Table slide (with a small pandas DataFrame)
       - Flow diagram slide
       - â€œKey Winsâ€ slide with one image placeholder
       - Thank-you slide
     - Uses only local logic and no network calls
     - Saves `example_presentation.pptx` in the repo root

### FORMAT OF YOUR OUTPUT

Return your answer in this structure so I can copy each block directly into files:

1. A heading: `### File: .github/agents/ppt-creator.agent.md`
   - Followed by a fenced code block with the full contents of that file
2. A heading: `### File: .github/prompts/ppt-from-excel.prompt.md`
   - Code block with that file content
3. A heading: `### File: .github/prompts/ppt-from-text.prompt.md`
   - Code block with that file content
4. A heading: `### File: .github/prompts/ppt-flow-diagram.prompt.md`
   - Code block with that file content
5. A heading: `### File: generate_ppt_example.py`
   - Code block with that file content

Do NOT include any commentary outside those headings and code blocks. I want to paste each code block directly into the appropriate file in VS Code.