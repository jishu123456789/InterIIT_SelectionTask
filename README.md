# Visual Question Answering (VQA) Model Architecture

This repository contains a **Visual Question Answering (VQA)** system that integrates **X-CLIP** for visual feature extraction and **LLaMA** for text-based question understanding and answer generation.

## Training Process

The system processes image inputs and textual questions, leveraging **X-CLIP** for visual representation learning and **LLaMA** for natural language understanding and answer generation. These components work together to output answers to questions about the visual input.

![X_CLIP_diagram](https://github.com/user-attachments/assets/e1556f7d-4a35-4b46-88b7-13e597d69cd2)


## Model Architecture

### 1. X-CLIP Image Processor

- The model accepts image input in the form of frames.
- Input dimensions: `(B, F, C, H, W)`, where:
  - `B`: Batch size
  - `F`: Number of frames
  - `C`: Number of channels (typically 3 for RGB images)
  - `H`: Image height
  - `W`: Image width
- These images are processed by the **X-CLIP** image encoder, generating visual embeddings.
- Output embeddings are reshaped to the format `(B*F, CLS, 768)`, where:
  - `CLS`: Classification token
  - `768`: Dimensionality of the embeddings

### 2. Feature Pooling and Reshaping

- The embeddings from X-CLIP are pooled to form a fixed-size vector with dimensions `(B*F, 768)`.
- The pooled output is reshaped into the format `(B, F, 768)`, which aligns with the batch size and frame context.

### 3. Multi-Layer Perceptron (MLP)

- The reshaped feature embeddings are passed through an **MLP**.
- The MLP transforms the visual embeddings, increasing the dimensionality to `(B, F, 3072)`.

### 4. LLaMA Embeddings for Text Input

- The question is tokenized and embedded using **LLaMA**.
- The prompt follows the structure:
  ```text
  <SOS> {Question} {Answer} <EOS>
- `<SOS>`: Start of sentence token
- `{Question}`: The input question
- `{Answer}`: The predicted answer (generated by the model)
- `<EOS>`: End of sentence token

**LLaMA** produces text embeddings of shape `(B, Seq_Len, 3072)`, where:
- `Seq_Len`: Length of the tokenized input sequence.

### 5. Concatenation of Visual and Textual Embeddings

- The visual embeddings `(B, F, 3072)` from the X-CLIP processor and the text embeddings `(B, Seq_Len, 3072)` from LLaMA are concatenated.
- The combined embeddings have the shape `(B, F + Seq_Len, 3072)`.

### 6. LLaMA with LoRA Fine-Tuning

- The concatenated embeddings are passed through the **LLaMA** model with **LoRA (Low-Rank Adaptation)**.
- LoRA enables efficient fine-tuning of the model with a reduced number of parameters.
- **LLaMA** processes these fused features and generates the final output, which corresponds to the answer to the question based on the visual context.

### 7. Output Generation

- The final output from **LLaMA** is the predicted answer to the question.
- This output takes into account both the image and the question, enabling accurate **Visual Question Answering**.

## Inference Phase

### All the trainable parameters are kept frozen during inference, and the input to the model is only the **video** and the **question**.
