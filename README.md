# Agentic AI by Mr. Hamid Ali
# Assignment number 2
# Submitted by: Muhammad Awais (MSDS24035)


# AI-Powered Quiz Grader 🤖📝

This script automates the grading of a three-part quiz from image files. It uses a multimodal vision model to extract handwritten or typed answers, grades them against a predefined answer key, and uses an LLM to grade a free-response question. It supports multiple AI providers, including OpenAI, Google, and a local Ollama instance.

---

## Features ✨

* **Image-to-Text Extraction**: Parses student answers directly from quiz images.
* **Multi-Provider Support**: Seamlessly switch between **OpenAI** (`gpt-4o-mini`, etc.), **Google** (`gemini-2.5-flash`, etc.), and **Ollama** (`llava`,`qwen2.5vl:3b`, etc.).
* **Comprehensive Grading**:
    * Grades multiple-choice questions (Part A).
    * Grades sequence-matching questions (Part B).
    * Uses a secondary LLM call to grade a free-response prompt specification (Part C).
* **Detailed Outputs**: Generates a `results.json` file with detailed scores and a `logs.jsonl` file for run traceability.
* **Image Preprocessing**: Automatically downscales and compresses images to optimize API usage and costs.

---

# AI Response Evaluation & Validation

This section details the validation and evaluation mechanisms built into the AI Quiz Grader script. These checks are crucial for ensuring the reliability and robustness of the grading process, as LLM outputs can sometimes be unpredictable, malformed, or incomplete.

The validation process is divided into two key stages:
1.  **Initial Data Extraction:** Validating the JSON output from the primary vision model.
2.  **AI-based Grading:** Validating the JSON output from the LLM that grades the free-form answer.

---

## 1. Vision Model Extraction Validation

After the vision model analyzes the quiz images, its JSON output is immediately passed through a strict validation function (`validate_vision_extraction`). This ensures the data structure is sound before any grading logic is applied.

### What It Checks

* **Correct Structure:** Confirms the top-level output is a dictionary.
* **Presence of Keys:** Ensures the critical keys (`part_a`, `part_b`, `part_c`) are present.
* **Correct Data Types:** Verifies that `part_a` and `part_b` are dictionaries and `part_c` is a string.

If a critical check fails (e.g., `part_a` is missing or isn't a dictionary), the script raises a `ValueError` and terminates gracefully to prevent further errors.

### Code Implementation

This validation is called immediately after the JSON is parsed in the `extract_answers_with_vision_model` function.

### Sample Log Events

The logger records the outcome of this validation step.

**Successful Validation:**
```json
{"ts": "2025-09-29T11:30:05.458Z", "level": "INFO", "event": "validation.vision_json.success", "cid": "..."}
```

**Failed Validation (Error):**
```json
{"ts": "2025-09-29T11:30:05.458Z", "level": "ERROR", "event": "validation.vision_json.error", "cid": "...", "error": "Validation failed: 'part_a' is missing or not a dictionary."}
```

---

## 2. LLM Grader Response Validation

For Part C, a separate LLM call is made to grade the student's free-form answer. The JSON response from this "grader" LLM is also rigorously validated.

### What It Checks

* **Valid JSON Object:** Ensures the response is a dictionary.
* **Required Keys:** Confirms that both `score` and `reasoning` keys are present.
* **Valid Score:**
    * Checks if the `score` value can be converted to a `float`.
    * **Clamps the score** to the valid range (0.0 to 2.0) to prevent the AI from assigning out-of-bounds grades.
* **Valid Reasoning:** Ensures the `reasoning` is a non-empty string.

If any of these checks fail, the script logs a warning and falls back to a default score of `0.0` and a descriptive reasoning string, ensuring the process doesn't crash.


### Sample Log Event (Warning)

If the grader model returns a non-numeric score, a warning is logged.

```json
{"ts": "2025-09-29T11:30:10.910Z", "level": "WARNING", "event": "grade.part_c.validation.warn", "cid": "...", "issue": "Score 'one point five' is not a valid float, defaulting to 0.0."}
```

## Setup and Installation ⚙️

### Prerequisites

* Python 3.8+
* Access to an AI provider:
    * An **OpenAI** API key.
    * A **Google** API key.
    * A running instance of **Ollama**.

### Installation Steps

1.  **Create a Virtual Environment** (Recommended)
    ```bash
    python -m venv venv
    source venv/bin/activate  # On Windows, use `venv\Scripts\activate`
    ```

2.  **Install Dependencies**
    Install all the required Python packages from the `requirements.txt` file.
    ```bash
    pip install -r requirements.txt
    ```

3.  **Configure Environment Variables**
    Create a file named `.env` in the root directory and add your API keys. Use the `.env.example` below as a template.

    ***.env.example***
    ```env
    # For OpenAI
    OPENAI_API_KEY="sk-..."

    # For Google
    GOOGLE_API_KEY="AIzaSy..."

    # For Ollama (if not running on the default localhost) (make sure to run ollama first on cmd)
    OLLAMA_BASE_URL="[http://127.0.0.1:11434/](http://127.0.0.1:11434/)"
    ```

---

## How to Use 🚀

Run the script from your terminal, providing the paths to the quiz images and specifying the model you want to use.

### Command Structure

The script uses the following command-line arguments:
* `--images`: (Required) One or more paths to the quiz image files.
* `--model`: The model to use, in `provider:model_name` format. Defaults to `openai:gpt-4o-mini`.
* `--out`: Path for the output JSON results file. Defaults to `results.json`.
* `--logs`: Path for the output JSONL logs file. Defaults to `logs.jsonl`.

### Example Commands

#### **Using OpenAI**
```bash
python rollnumber_msds24035_grader.py \ 
--images quiz_pages/1.jpg quiz_pages/2.jpg quiz_pages/3.jpg quiz_pages/4.jpg \
--out out/results.json \
--logs logs/logs.jsonl \
--model openai:gpt-4o-mini
```

#### **Using GeminiAI**
```bash
python rollnumber_msds24035_grader.py \
--images quiz_pages/1.jpg quiz_pages/2.jpg quiz_pages/3.jpg quiz_pages/4.jpg \
--out out/results.json \
--logs logs/logs.jsonl \
--model google:gemini-2.5-flash
```

#### **Using Local VLM (Ollama)**
```bash
python rollnumber_msds24035_grader.py \
--images quiz_pages/1.jpg quiz_pages/2.jpg quiz_pages/3.jpg quiz_pages/4.jpg \
--out out/results.json \
--logs logs/logs.jsonl \
--model ollama:qwen2.5vl:3b
```
