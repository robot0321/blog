---
title: Vision Transformer (ViT)
date: 2026-05-18
tags:
  - Transformer
  - FeatureExtractor
---
[An Image is Worth 16x16 Words: Transformers for Image Recognition at Scale](https://arxiv.org/abs/2010.11929) (2020)
* 근본적으론 Encoder임. Decoder는 [[Transformer]] 와 동일 구조 사용
* 논문의 기본 철학은 non-image-specific inductive bias & image as a sequence of patches
* Scalability 효과를 받으려고 CNN 구조에서 Transformer로 전환 시도. 
* CNN의 inductive bias (translation equivariance, locality, etc.)가 없어서 데이터 많아야 효과적.

## Architecture
* Image 버전의 Transformer: patch (16x16)-wise Token으로 나누기
* patch를 serialize하고 decoder에 맞는 크기$\in\mathbb{R}^d$로 linear projection (learnable)
	* CNN 적용 후 얻은 feature map을 patchify 하는 hybrid model도 있음
* Continuous space feature 사용 $\leftrightarrow$ Vector-Quantization (VQ) 계열

### `[Cls]` Token
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
* Figure 4
	* 적은 데이터 (9M, 30M)에서는 convolutional inductive bias가 쓸모 있다.
	* 많은 데이터에서는 (90M, 300M) relavant pattern도 데이터로부터 배운다.
* Figure 5
	* Hybrid (CNN+ViT)는 적은 computation에서는 조금 나은데, 더 하면 결국 비슷해진다.
	* ViT는 saturate가 안 되서 scaling이 가능할거 같다.
* Figure 7
	* 학습된 linear projection보니까 잘 쓰는 basis function이랑 비슷하게 나오더라
	* 1D pos. emb.으로 보니까 2D 위치 잘 찾더라 + row-column 격자를 주로 보더라
* Figure 6 + Figure 7 (right)
	* 초반 레이어: 어떤 head는 먼 거리를 보고 (global) 어떤 head는 가까운 곳만 (local)
		* CNN: 초반 레이어 에서 무조건 Local만 볼 수 밖에 없음.
	* 깊은 레이어: 거의 모든 head가 global -> 이미지 전체의 맥락
* self-supervised pre-training
	* BERT 방식 ([MASK] 토큰으로 대체, 인코더 통과 후 맞추기) 해봤는데, from scratch 보단 낫지만 supervised pre-training에 비해 별로라는 결론
	* 디코더를 활용 및 다른 구조로 [MASK] 복원에 (무려 75% 마스킹!) 성공한 논문이 [[MAE]]

## 활용: Multi-model LLM
ViT가 이미지를 Token sequence로 쪼개면서 LLM와 같은 Transformer 구조 사용 가능
1. 인터페이스 연결 (이미지의 Tokenizer+Embedding): [[ViT]]
2. 의미 공간 정렬 (Text-Image Alignment): [[CLIP]]
3. In-context Learning: 강력한 Decoder (LLM)의 지식 재 사용 [[GPT]]