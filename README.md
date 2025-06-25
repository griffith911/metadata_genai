# metadata_genai
# 🧠 Document Converter UI – Deep Dive

This project is a **Streamlit-powered front-end** for converting documents of various types (PDF, DOCX, PPTX, etc.) into structured **Markdown** using a modular pipeline system provided by the `docling` framework.

---

## 📆 What This Repo Does

* Provides a web UI to upload documents
* Detects and routes files to appropriate processing pipelines
* Uses a backend-per-format approach to extract structured content
* Converts content to Markdown and displays it to the user
* Allows Markdown download

---

## 📁 High-Level Project Structure

```
└── document_converter_ui.py
    ├── Streamlit UI (bottom)
    ├── DocumentConverter class (core controller)
    ├── FormatOption (ties pipeline + backend)
    ├── FORMAT_DEFAULTS (map format → pipeline + backend)
    └── Backend + Pipeline loading (via importlib)
```

---

## 🛠️ Core Components

### 1. `DocumentConverter` (Main Controller)

This class handles:

* Loading and configuring pipelines based on document type
* Validating supported formats
* Chunking and processing files
* Returning clean, structured document objects

#### Methods:

* `convert_single()`: Converts one file
* `convert_multiple()`: Converts many files with batching
* `process_document()`: Core execution logic

---

### 2. `FormatOption`

Encapsulates:

* The pipeline class to use (e.g. `SimplePipeline`, `StandardPdfPipeline`)
* The backend class to use (e.g. `MsWordDocumentBackend`, `CsvDocumentBackend`)
* Any specific pipeline options

#### Lifecycle:

1. Pulled from `FORMAT_DEFAULTS`
2. Defaults set if options not specified
3. Passed to pipeline initialization

---

### 3. `FORMAT_DEFAULTS`

A mapping like this:

```python
InputFormat.DOCX: (SimplePipeline, MsWordDocumentBackend)
```

It defines what pipeline/backend combo to use for each input format.

---

### 4. Backend Classes

Dynamically imported using `importlib`:

```python
backend_classes = {
    'CsvDocumentBackend': 'docling.backend.csv_backend',
    ...
}
```

Each backend knows how to **parse and export** documents in its specific format.

Backends are responsible for:

* Reading raw data from the uploaded file
* Structuring content
* Providing export methods like `.export_to_markdown()`

---

### 5. Pipelines

Each pipeline processes a document in multiple steps:

* `SimplePipeline`: For basic formats like DOCX, CSV, etc.
* `StandardPdfPipeline`: For PDFs/images
* `AsrPipeline`: For audio transcription

Pipelines are initialized **with options** and then cached by hash (to avoid repeated instantiations).

---

## 🔁 Document Processing Flow

Here’s a full step-by-step of what happens when a user uploads a file:

1. **Streamlit Upload**:

   ```python
   uploaded_file = st.file_uploader(...)
   ```

2. **Temp Save**:
   Uploaded file is saved to a temporary path using `tempfile.NamedTemporaryFile`.

3. **Format Detection**:
   The file extension determines the `InputFormat` enum used.

4. **Conversion Begins**:

   ```python
   converter = DocumentConverter()
   result = converter.convert_single(tmp_path)
   ```

5. **FormatOption Lookup**:
   Internally, this loads the appropriate `FormatOption` from `FORMAT_DEFAULTS`.

6. **Pipeline Initialization**:
   A pipeline is created (or reused) using cached hash of its options.

7. **Backend Parsing**:
   The file is processed via the backend logic to extract structure/content.

8. **Markdown Export**:

   ```python
   md = result.document.export_to_markdown()
   ```

9. **Streamlit Output**:
   Markdown is rendered in the UI and offered for download.

---

## 🧪 Error Handling

Handled via:

* `ConversionError` (raised during document parsing/conversion issues)
* Streamlit shows error in UI:

  ```python
  except Exception as e:
      st.error(...)
  ```

Also, unsupported formats return a `ConversionResult` with `SKIPPED` status.

---

## 🧱️ Extending the System

Want to support `.odt` or some other format?

1. Create a backend in `docling.backend.odt_backend`
2. Write a `OdtDocumentBackend` class
3. Add a mapping:

   ```python
   InputFormat.ODT: (SimplePipeline, OdtDocumentBackend)
   ```
4. Add it to `backend_classes` for import

---

## 🧪 Advanced Features You Can Build

* Add UI to tweak `PipelineOptions` (e.g., page limits, language for OCR)
* Preview document content alongside Markdown
* Add export formats: JSON, HTML, PDF
* Integrate metadata extraction or classification

---

## 🛠️ Requirements

Ensure you have:

* Python ≥ 3.8
* `streamlit`
* `pydantic`
* `docling` (internal module, must be installed locally or from source)

Install with:

```bash
pip install -r requirements.txt
```

Then run:

```bash
# ⚠️ Streamlit does not support running `.ipynb` files directly. Convert it to a `.py` file first:
jupyter nbconvert --to script document_converter_ui.ipynb
streamlit run document_converter_ui.py
```

---

## 💬 Conclusion

This system is modular, extendable, and performant. It provides a clean UI over a powerful and pluggable backend architecture. Most complexity is abstracted into `docling`, allowing you to build powerful document workflows quickly.
