# Understanding Native Multimodality: Gemini, TPU, and the Path Toward Embodied AI

## Introduction

As large language models scale, it has become increasingly clear that text-only intelligence is not sufficient for general intelligence. Real-world understanding requires models to reason over vision, audio, and physical interaction, not as auxiliary inputs, but as first-class signals.

Google’s **Gemini** represents a fundamental shift in this direction through **native multimodality**—a design philosophy where text, images, audio, and video are processed within a single unified model, rather than being stitched together from specialized components. This architectural choice is deeply intertwined with hardware co-design, particularly Google’s TPU infrastructure, and has direct implications for robotics and embodied intelligence.

This article analyzes how Gemini’s native multimodal design works, why it is difficult to replicate, and how it connects to the future of embodied AI systems such as robots.

---

## 1. Composed Multimodality vs. Native Multimodality

### Traditional “Modality Gluing”

Most early multimodal systems follow a **composed architecture**, where different modalities are handled by separate expert models and then connected downstream.

**Typical pipeline:**
- Image $\rightarrow$ Encoder $\rightarrow$ Adapter $\rightarrow$ LLM
- Audio $\rightarrow$ ASR $\rightarrow$ Text $\rightarrow$ LLM

**For example:**
1. Images are processed by a vision encoder (CNN or ViT).
2. Audio is converted to text via speech recognition.
3. The resulting representations are mapped into a language model through adapter layers.

This approach resembles translating everything into text before thinking, similar to translating English into Chinese and then generating an image—information is inevitably lost during the conversion. Early versions of GPT-4 and many open-source systems rely on this paradigm, effectively “gluing” vision and language experts together.

> **Key limitation:** Fine-grained spatial details, tone, emotion, background sound, and visual composition are often discarded during intermediate conversions.

### Gemini’s Native Multimodality

Gemini takes a fundamentally different approach. Instead of translating everything into text, Gemini treats visual, auditory, and textual signals as equally valid symbolic inputs. Pixels, sound waves, and words are all converted into tokens and processed jointly.

**Core ideas:**
- Unified multimodal tokenization
- A shared high-dimensional embedding space
- Pre-training from scratch on mixed multimodal data

> *[Figure suggestion: Traditional composed pipeline vs. Gemini native multimodal pipeline]*

---

## 2. Tokens Across Modalities

At the heart of native multimodality lies a unified notion of tokens.

### 2.1 Text Tokens: Semantic Slices
Text tokenization follows standard NLP practices:
- **Units:** sub-words
- **Example:** `Generative` $\rightarrow$ `Gener` + `ative`
- Tokens are discrete and mapped to vocabulary IDs.

### 2.2 Image Tokens: Spatial Slices (ViT-style)
Gemini does not tokenize individual pixels, which would be computationally infeasible. Instead, it adopts a Vision Transformer–style strategy:
- **Unit:** image patches
- **Typical patch size:** $16 \times 16$ or $32 \times 32$ pixels
- Each patch is linearly projected into a continuous vector
- A single image is represented by a fixed number of tokens (e.g., 256 or more)

These tokens function like puzzle pieces, with attention mechanisms learning spatial relationships among them.

### 2.3 Audio Tokens: Temporal Slices
Audio is continuous and time-dependent, making it the most challenging modality.

**Typical process:**
1. Sample raw waveform (e.g., every 20 ms)
2. Convert to frequency domain via Fourier transform
3. Encode using an audio encoder (e.g., spectrogram-based transformers)

Gemini processes roughly ~32 audio tokens per second, capturing:
- Pitch
- Timbre
- Background noise
- Semantic content

### 2.4 The Unified Embedding Space
This is the most elegant—and most powerful—aspect of Gemini’s design. All tokens, regardless of modality, are mapped into vectors of the same dimensionality:

$$
Token_{Text} \in \mathbb{R}^d, \quad Token_{Image} \in \mathbb{R}^d, \quad Token_{Audio} \in \mathbb{R}^d
$$

As a result, within the Transformer’s self-attention layers:
- The token for the word “cat” can directly attend to tokens representing a cat’s ears in an image.
- A token encoding an angry tone can strongly interact with punctuation such as “!”.

This is **cross-modal reasoning without translation**.

> *[Figure suggestion: Unified embedding space with cross-modal attention]*

---

## 3. Scaling Multimodality: Long Images and Dynamic Token Allocation

A natural question arises:
*If an image already requires hundreds of tokens, how does Gemini handle 4K images or extremely long images (e.g., scanned documents or scroll-like paintings)?*

Gemini employs techniques similar to **NaViT** (Native Resolution ViT) and dynamic resampling.

### 3.1 Multi-scale Representation
Instead of processing a single resolution:
- **Low-resolution global thumbnail:**
  - Few tokens
  - Captures overall semantics (“this is a landscape”)
- **High-resolution local crops:**
  - Allocated only where detail matters

### 3.2 Dynamic Token Allocation
- Simple images receive fewer tokens.
- Dense or text-heavy images receive more.
- Important features are compressed via resampling into a limited token budget.

### 3.3 Aspect Ratio Awareness
Unlike early ViT models that force images into square inputs (e.g., $224 \times 224$), Gemini:
- Preserves original aspect ratios
- Applies 2D positional embeddings
- Dynamically orders patches in the Transformer

**Human analogy:**
When reading a newspaper, you first scan the whole page, then zoom into a specific paragraph. Gemini learns to do the same.

> *[Figure suggestion: Multi-scale image processing with global + local attention]*

---

## 4. Why Native Multimodality Is Hard

If native multimodality is so powerful, why doesn’t everyone adopt it?

1. **Extreme training cost:** Requires massive compute and data.
2. **Data complexity:** Large-scale aligned text–image–audio–video datasets.
3. **Optimization difficulty:** Preventing catastrophic interference between modalities.

This is where hardware becomes decisive.

---

## 5. TPU as the Enabler: Hardware–Model Co-design

### 5.1 The Origin of TPU
TPUs were born out of necessity:
- **Early 2010s:** Google realized voice search alone could double data center demand.
- **2016:** TPU v1 powered AlphaGo’s historic victory over Lee Sedol.
- **Later versions (v2/v3):** Enabled large-scale training.
- **Recent generations (TPU v6/v7, Trillium):** Optimized for Gemini-scale multimodal models with ultra-fast interconnects.

### 5.2 The Systolic Array Advantage
Traditional architectures frequently read and write intermediate results to memory—slow and energy-intensive.

**TPUs use systolic arrays:**
- Data flows rhythmically through thousands of compute units
- Minimal memory access
- Massive gains in efficiency

> *[Figure suggestion: Systolic array data flow vs. traditional memory access]*

---

## 6. Beyond TPU: The Specialized Accelerator Landscape

TPU is unique—but not alone.

- **GPU (NVIDIA):** General-purpose, CUDA ecosystem dominance
- **AWS Trainium / Inferentia:** Cloud-optimized accelerators
- **Tesla Dojo:** Purpose-built for autonomous driving and robotics
- **NPU:** Mobile AI (low power, on-device inference)
- **LPU (e.g., Groq):** Ultra-fast LLM inference
- **Neuromorphic chips:** Event-driven, energy-efficient computation for robotics

NVIDIA continues to thrive due to software ecosystem lock-in, not raw hardware superiority.

---

## 7. Robotics: From Perception to End-to-End Control

### Google
- **Gemini Robotics / RT-X**
- Learning manipulation skills from YouTube videos
- Providing the “brain” rather than full robot stacks

### Tesla
- **Optimus Gen 2/3** operating in factories
- **End-to-end neural networks:**
  - Input: raw camera pixels
  - Output: joint torques and finger control
  - No hand-written “if-then” logic

This represents a shift from **rule-based robotics** to **learning-based embodiment**.

---

## 8. What Engineers Can Optimize

With hardware pathways established, engineers optimize how fast and efficiently intelligence runs.

### 8.1 Quantization
- Compressing parameters (FP32 $\rightarrow$ INT8)
- Reduces memory and latency
- Requires algorithmic care (e.g., quantization-aware training)

### 8.2 Operator Fusion
- Combining multiple operations into a single kernel
- Reduces memory traffic
- Often yields 20–30% speedups

### 8.3 Knowledge Distillation
- Large teacher model (Gemini) supervises smaller student models
- Student learns both outputs and internal reasoning patterns

---

## Conclusion

Native multimodality is not just a model design choice—it is a systems-level commitment spanning data, architecture, hardware, and optimization.

Gemini and TPU illustrate how tightly coupled co-design can unlock new capabilities in perception, reasoning, and embodiment. As robotics moves toward end-to-end learning and real-world interaction, such unified architectures may represent a necessary step toward embodied intelligence.
