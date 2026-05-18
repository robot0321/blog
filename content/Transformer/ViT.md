---
title: Vision Transformer (ViT)
date: 2026-05-18
tags:
  - Transformer

---
[An Image is Worth 16x16 Words: Transformers for Image Recognition at Scale](https://arxiv.org/abs/2010.11929) (2020)
* 근본적으론 Encoder임. 
* Scalability 효과를 받으려고 CNN 구조에서 Transformer로 전환 시도. 
* CNN의 inductive bias (translation equivariance, locality, etc.)가 없어서 데이터 많아야 효과적.

## Architecture
* Image 버전의 Transformer: patch (16x16)-wise Token으로 나누기
* patch를 serialize하고 decoder에 맞는 크기$\in\mathbb{R}^d$로 linear projection (learnable)
	* CNN 적용 후 얻은 feature map을 patchify 하는 hybrid model도 있음
* Continuous space feature 사용 $\leftrightarrow$ Vector-Quantization (VQ) 계열

### `[Class]` Token
* `[cls]` token 자리를 만들어서 sub-task (ViT는 classification)에 맞는 정보 그릇으로 사용
	* `[cls]` token 자리의 output vector만을 MLP head (classification)에 집어 넣어서 학습하기 때문에, attention을 하면서 `[cls]` 벡터는 '전체의 응축된 특징' (for sub-task)를 갖음.
	* 다른 sub-task: [[DINO]] (segmentation), [[MAE]] (reconstruction)

### Positional Embedding
* Standard "learnable" 1D position embeddings
	* 충분한 데이터가 있으면 Transformer가 1D index 사이의 상대적 위치 관계를 알아냄
		* 설계 철학(?): Inductive bias 완전 배제, BERT와의 구조적 통일성
		* 1D를 쓴 건 "이것도 된다"는 선언적 의미가 강한 듯.
	* 데이터가 적거나 dimension이 올라가면 (3D), dimension-aware 구조가 안전
		* 안전성 강화: [[SwinTransformer]], CPVT
* Different resolution image
	* pre-trained position embedding에 2D interpolation 적용, patch size는 유지
	* 이렇게 까지 하면서 1D PE를 고집했던건 Multi-modal LLM을 위함이 아니었을까?

## Experiments
* 


## 활용: Multi-model LLM
ViT가 이미지를 Token sequence로 쪼개면서 LLM와 같은 Transformer 구조 사용 가능
1. 인터페이스 연결 (이미지의 Tokenizer+Embedding): [[ViT]]
2. 의미 공간 정렬 (Text-Image Alignment): [[CLIP]]
3. In-context Learning: 강력한 Decoder (LLM)의 지식 재 사용 [[GPT]]