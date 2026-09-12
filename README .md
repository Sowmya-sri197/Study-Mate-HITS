# Study Mate

Study Mate is a small project I built to make it easier to search through PDFs instead of scrolling through pages looking for one line. You upload a PDF, ask it a question in plain English, and it pulls out the passages that actually match what you're asking — using embeddings instead of just matching keywords.

There's also a second tab for image generation, but that part is still just a placeholder for now (it literally returns a blank image). Wanted to leave the UI in place for when I get around to hooking up a real model.

## What it does

- Upload a PDF and it gets parsed and split into chunks
- Chunks are embedded using `all-MiniLM-L6-v2` from sentence-transformers
- FAISS is used to store the embeddings and do similarity search
- You type a question, it searches the PDF and shows you the top matching passages with a rough confidence score
- Basic Gradio interface so it's usable without touching code

## Why chunking with overlap

If you just split text every 400 words, you can end up cutting a sentence or idea right in half, and then neither chunk really has the full context. So instead I used a sliding window — each chunk is 400 words, but it overlaps the previous one by 100 words. That way if something important lands near the edge of a chunk, it doesn't get lost.

## Stack

- Gradio – UI
- PyPDF2 – reading text out of PDFs
- sentence-transformers – turning text into embeddings
- FAISS – fast similarity search
- Pillow – for the image tab

## Running it

Install the dependencies:

```bash
pip install faiss-cpu gradio sentence-transformers PyPDF2 pillow
```

Then just run the script:

```bash
python study_mate_hits.py
```

It'll give you a local Gradio link. Open it, upload a PDF under the "Chat with PDF" tab, hit process, and once it's indexed you can start asking questions.

## How it actually works under the hood

1. `extract_text_from_pdf()` goes page by page and pulls out whatever text PyPDF2 can find.
2. `chunk_text()` breaks that into overlapping word chunks.
3. `build_faiss_index()` embeds every chunk and normalizes the vectors so cosine similarity works out to a simple inner product, then loads them into a FAISS index.
4. When you ask something, `search_faiss()` embeds your question the same way and finds the closest chunks.
5. `chat_with_pdf()` takes the top results and formats them into something readable, along with an approximate match score.

## Known limitations / things I'd still like to fix

- Right now it just shows you the raw matching passages instead of generating an actual answer from them — a proper RAG setup where an LLM reads the retrieved chunks and writes a real answer would be a big upgrade.
- Image generation tab doesn't do anything yet, it's just a stub.
- Only one PDF at a time — no multi-document support.
- Index isn't saved anywhere, so if you restart the app you have to re-upload and re-process the PDF.
- No indication of which page in the PDF a passage came from, which would be nice for actually citing sources.

## License

MIT (or change this to whatever you're using)
