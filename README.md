# Shutterstock Metadata Automation

An AI-powered image metadata generation workflow for Shutterstock Contributor, built with Python, Jupyter Notebook, and Ollama.

This project automates image preparation, visual analysis, and metadata generation to simplify the process of preparing photographs for stock photography submission.

## Overview

Preparing images for stock photography platforms requires accurate descriptions, relevant keywords, and appropriate categories.

This project provides a two-stage workflow:

1. **File Updater** — Prepares image batches by copying source images from Windows to a WSL/Linux working directory and updating environment configurations.
2. **Metadata Generator** — Analyzes images using a local vision language model and generates structured Shutterstock metadata in CSV format.

## Features

- **Batch Image Management** — Organize images using date-based batch directories.
- **Windows to WSL File Transfer** — Automatically copy source images into the working environment.
- **Image Scanning** — Collect image information, including dimensions, megapixels, format, and file size.
- **AI Image Analysis** — Analyze image content using Ollama and Qwen 2.5 VL 3B.
- **Image Preprocessing** — Resize images while maintaining aspect ratio before AI inference.
- **Structured Metadata Generation** — Generate descriptions, keywords, categories, and editorial classifications.
- **CSV Export** — Export generated metadata into a CSV file.

## Technology Stack

| Technology | Purpose |
|---|---|
| Python | Main programming language |
| Jupyter Notebook | Workflow execution |
| Ollama | Local AI model inference |
| Qwen 2.5 VL 3B | Vision-language model |
| Pillow | Image processing and resizing |
| Pandas | Image inventory and metadata handling |
| Python-dotenv | Environment configuration |
| CSV | Metadata export format |

## Project Structure

```text
shutterstock_metadata/
│
├── file_updater.ipynb       # Image preparation and batch management
├── generator.ipynb          # AI image analysis and metadata generation
├── .env                     # Environment configuration
└── .gitignore
```

## Requirements

- Python 3.10+
- Jupyter Notebook or JupyterLab
- Ollama installed and running
- Qwen 2.5 VL 3B model

### Python Dependencies

```bash
pip install jupyter pandas pillow python-dotenv ollama
```

### Ollama Setup

Install Ollama and download the required model:

```bash
ollama pull qwen2.5vl:3b
```

Make sure the Ollama service is running before executing the metadata generator.

Verify the model:

```bash
ollama list
```

## Getting Started

### 1. Clone the Repository

```bash
git clone https://github.com/ardfahriarr/shutterstock_metadata.git

cd shutterstock_metadata
```

### 2. Create a Virtual Environment

```bash
python -m venv venv
```

Activate the environment:

**Linux / WSL**

```bash
source venv/bin/activate
```

**Windows**

```bash
venv\Scripts\activate
```

### 3. Configure Environment Variables

Create a `.env` file in the project root:

```env
SOURCE_DIR=/mnt/c/Users/YourUsername/Pictures/Shutterstock
```

`SOURCE_DIR` points to the original image directory accessible from the WSL/Linux environment.

The remaining configuration variables will be generated automatically by `file_updater.ipynb`.

### 4. Run File Updater

**Execute `file_updater.ipynb` first.**

This notebook prepares the image batch and working directories.

The workflow:

1. Read `SOURCE_DIR` from `.env`.
2. Generate a batch ID using the current date (`batch_YYYYMMDD`).
3. Create the destination input directory.
4. Copy images from the Windows source directory into WSL/Linux.
5. Create input, output, and AI output directories.
6. Update `.env` with the generated batch configuration.

Generated directory structure:

```text
shutterstock/
├── input/
│   └── batch_YYYYMMDD/
├── output/
│   └── batch_YYYYMMDD/
└── ai_output/
    └── batch_YYYYMMDD/
```

The updater also writes the following variables into `.env`:

```env
BATCH_ID=batch_YYYYMMDD
BASE_DIR=/path/to/shutterstock
INPUT_DIR=/path/to/shutterstock/input/batch_YYYYMMDD
OUTPUT_DIR=/path/to/shutterstock/output/batch_YYYYMMDD
```

### 5. Run Metadata Generator

After completing the file updater, execute `generator.ipynb`.

The generator reads the environment configuration and processes the images from the corresponding batch directory.

#### Processing Workflow

1. **Load Configuration**

   Load batch ID and directory paths from `.env`.

2. **Scan Images**

   Identify JPG/JPEG images and collect:
   - Filename
   - Filepath
   - Width and height
   - Megapixels
   - Image format
   - File size

3. **Image Preprocessing**

   - Convert images to RGB.
   - Resize images while maintaining aspect ratio.
   - Set maximum image dimensions to 512 × 512 pixels.
   - Encode the processed image into Base64.

4. **AI Image Analysis**

   Send the processed image and metadata-generation prompt to Ollama.

   Model: `qwen2.5vl:3b`

   Inference configuration:
   - Temperature: 0
   - Seed: 42
   - Maximum predicted tokens: 512
   - Structured JSON output using a predefined metadata schema.

5. **Metadata Generation**

   Generate the following fields:

   | Field | Description |
   |---|---|
   | Filename | Descriptive English filename |
   | Description | Factual image description |
   | Keywords | Relevant image keywords |
   | Categories | Shutterstock image categories |
   | Illustration | Illustration classification |
   | Mature Content | Mature content classification |
   | Editorial | Editorial classification |
   | review_required | Human review flag |
   | review_reasons | Reasons for review |

6. **CSV Export**

   Export the generated metadata into:

   ```text
   output/batch_YYYYMMDD/shutterstock_metadata.csv
   ```

   The current CSV export contains:

   - Filename
   - Description
   - Keywords
   - Categories
   - Editorial

## Workflow Architecture

```text
Windows Source Directory
          |
          v
   file_updater.ipynb
          |
          v
    Image File Copy
       Windows -> WSL
          |
          v
   Batch Directory Setup
          |
          v
     Update .env
          |
          v
   generator.ipynb
          |
          v
     Image Scanning
          |
          v
   Image Preprocessing
      (RGB / Resize)
          |
          v
    Base64 Encoding
          |
          v
     Ollama Runtime
          |
          v
    Qwen 2.5 VL 3B
          |
          v
   Structured Metadata
          |
          v
       CSV Export
          |
          v
   Shutterstock Metadata
```

## Metadata Generation Guidelines

The AI prompt follows several guidelines to improve metadata accuracy:

- Use factual descriptions based on visible image content.
- Generate English filenames and descriptions.
- Generate relevant, non-duplicate keywords.
- Use a maximum of two Shutterstock categories.
- Select categories only from the predefined Shutterstock category list.
- Avoid speculative geographical or contextual information.
- Flag visible logos and recognizable people.
- Require human approval for final submission decisions.

The model is instructed to describe visual evidence rather than invent stories about photographs.

## Important Notes

- Run `file_updater.ipynb` before `generator.ipynb`.
- Ensure the source directory is accessible from WSL/Linux.
- Ensure Ollama is running and the required model is available.
- The current image scanner supports JPG and JPEG files.
- The generated metadata must be reviewed before submission.
- `ai_output` is currently created during batch initialization but is not yet used by the generator.
- Although the AI generates additional review fields, the current CSV export only includes five metadata columns.

## Disclaimer

This project is an independent automation tool and is not officially affiliated with Shutterstock.

AI-generated metadata should always be reviewed before submission. The contributor remains responsible for ensuring that submitted content complies with Shutterstock's contributor guidelines.

## License

No license has been specified yet. All rights reserved by the repository owner unless a license is added.
