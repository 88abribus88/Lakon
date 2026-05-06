---
source: /home/abribus/workspace/lakon/examples/wikipedia-attention.txt
translated_at: 2026-05-06T08:58:07Z
skill: skills/translate
version: "1.0"
---

In machine learning, attention = method determining importance of each component in a sequence relative to other components. In NLP, importance represented by "soft" weights assigned to each word. More generally, attention encodes vectors (token embeddings) across a fixed-width sequence ranging from tens → millions of tokens.

Unlike "hard" weights (computed during backward training pass), "soft" weights exist only in the forward pass → change with every input step. Earlier designs implemented attention in serial RNN language translation; more recent design (transformer) removed slower sequential RNN, relied on faster parallel attention.

Inspired by human attention, the mechanism was developed to address weaknesses of using hidden-layer info from RNNs. RNNs favor recent info (words near end of sentence); earlier sentence info tends to be attenuated. Attention allows a token direct access to any sentence part, ¬ only through previous state.
