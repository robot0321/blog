---
title: Transformer
tags:
  - Transformer
  - Attention
---
[Attention is all you need](https://arxiv.org/abs/1706.03762) (2017)
language translation을 목표로 구조를 설계해서, 이를 기반으로 이해하면 편하다.
* e.g.) input (영어)를 condition으로 output (프랑스어)를 auto-regressive하게 출력 (condition + 이전 output -> output+1)

## Attention
* Query, Key, Value: from DB
	* DataBase(DB)에서 유래한 용어로 딥러닝에 메모리를 도입하려는 맥락
	* Key-Value는 DB에 저장된 형태 (Information Store)
	* Query는 검색 요청 (Search Request)

* `Scaled Dot-Product Attention`: faster, efficient, robuts
	* Additive attention 보다 빠르고 효율적 / gradient 안정화를 위해 scaling (너무 작음)
$$ Attention(Q,K,V) = softmax\left(\frac{QK^T}{\sqrt{d_k}}\right)V $$
* `Multi-head attention`
	* 여러 개의 Linear projection 도입 > low-rank로 efficient focusing 가능

* Query와 Key, Value는 보통 다르다: `Cross-Attention`
	* 근데 Q, K, V를 모두 똑같이 두면 자기 자신에 대한 중요도 재 조정: `Self-Attention`

## Tokenizer & Embedding
* Tokenizer: 입력을 Token단위로 나누고 Index로 변환 (고정된 Algorithm + Lookup Table)
	* Text: BERT (확률 기반), ByT5 (byte단위, tokenizer-free)
	* Image: [[ViT]] (patch 단위), Swin Transformer (hierarchical patch)
* Embedding: Index을 vector로 변환 (Lookup table, 다만 vector는 학습)

## Positional Encoding
* 각 token에게 본인의 위치를 알려주기 위함 (cnn, rnn과 다르게 구조 상 모름)
* 기본적인 sinusodial PE 사용 -> offset k를 linear function으로 표현 가능
	* 요새 표준: [[RoPE]] (Rotary Positional Embedding)

## Encoder-Decoder 구조
* Encoder 내부: 문장에서 서로의 관계를 고려하기 위해 모든 요소끼리 self-attention
	* 모든 요소끼리 self-attention 하는 방식을 지칭하는 여러 용어:
	 `Full Self-Attention`, `Non-causal Self-Attention`, `Bidirectional Self-Attention`
* Decoder 내부 (1):  auto-regressive 방식으로 과거 요소만 고려해서 self-attention
	* 과거 요소만 고려해서 self-attention하는 방식을 지칭하는 여러 용어
	  `Masked Self-Attention`, `Causal Self-Attention`, `Uni-directional Self-Attention`
* Decoder 내부 (2): decoder (1)에서 온 Query와 encoder에서 온 Key-Value (full self-attention)
	* query에서 온 정보를 input에서 찾아서 (key) 대응하는 값(value)으로 
	* inference 시엔 output의 마지막 값만 떼다가 사용
	
