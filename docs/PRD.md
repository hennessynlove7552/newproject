# 제품 요구사항 문서 (PRD)
## 프로젝트: Local-First Enterprise Doc-Informer (MVP)

### 1. 개요 (Overview)
이 프로젝트는 업스테이지(Upstage)의 **AI Model Production** 직무에서 요구하는 "데이터 파이프라인-RAG-평가" 역량을 증명하기 위한 MVP(Minimum Viable Product)를 구축하는 것을 목표로 합니다. 핵심 컨셉은 **"Local Resourcing, Cloud Reasoning"**으로, 로컬 개발 생산성과 리소스 효율성(CPU/RAM)을 극대화하면서도, 핵심 추론 작업에는 고성능 API(Upstage Solar)를 활용하는 것입니다.

### 2. 목표 (Goals & Objectives)
*   **핵심 역량 증명:** 데이터 파싱, 로컬 벡터 저장소 구축, RAG 구현, 정량적 평가 기술을 보여줍니다.
*   **리소스 효율성:** 인덱싱 및 저장과 같은 무거운 작업은 로컬에서 수행하여 비용을 최소화하고, 고품질 생성을 위해 생성 단계에서만 API(Solar LLM)를 사용합니다.
*   **품질 우선:** 막연한 "느낌"이 아닌, `Ragas`를 사용한 정량적 성능 평가를 수행합니다.
*   **프로덕션 준비성:** 단순 스크립트가 아닌, 프로덕션 수준의 애플리케이션 구조로 프로젝트를 구성합니다.

### 3. 타겟 독자 (Target Audience)
*   **주요 독자:** 업스테이지 AI Model Production 채용 담당자 및 면접관.
*   **보조 독자:** Solar API를 활용한 로컬 우선 RAG 파이프라인의 레퍼런스 구현을 찾는 개발자.

### 4. 기술 스택 (Local-Lean Stack)
*   **사용자 인터페이스 (UI):** Streamlit (로컬 웹 서버)
*   **오케스트레이터:** LangChain / LlamaIndex
*   **데이터 파싱:**
    *   `PyMuPDF` (속도)
    *   `Unstructured` (구조/레이아웃 파싱)
*   **벡터 데이터베이스:** `ChromaDB` (`.db` 파일을 통한 로컬 영구 저장)
*   **임베딩:** `HuggingFaceBgeEmbeddings` (작고 고성능인 CPU 친화적 모델)
*   **LLM (생성):** **Upstage Solar API** (메인) 또는 Ollama (로컬 대체용)
*   **평가:** `Ragas` (Faithfulness, Answer Relevance 등)

### 5. 주요 기능 및 요구사항 (Key Features & Requirements)

#### 5.1 Step 1: 고성능 로컬 데이터 파싱 (Data Production)
*   **요구사항:** 구조(제목, 본문 등)를 보존하며 PDF 문서를 파싱합니다.
*   **차별화:** 단순 텍스트 추출을 넘어, 문서 계층 구조에 기반한 청킹(Chunking) 전략을 구현합니다.
*   **입력:** 기술 문서 (예: 업스테이지 기술 블로그, Solar 모델 가이드 등).

#### 5.2 Step 2: 로컬 벡터 저장소 구축 (Retrieval)
*   **요구사항:** 로컬 경량 모델(`HuggingFaceBgeEmbeddings`)을 사용하여 청크를 임베딩하고 `ChromaDB`에 저장합니다.
*   **검색 전략:** 단순 유사도 검색(Similarity Search)뿐만 아니라, 다양한 맥락 확보를 위해 `Max Marginal Relevance (MMR)`를 구현합니다.

#### 5.3 Step 3: Solar API를 활용한 생성 (Generation)
*   **요구사항:** 검색된 맥락을 Upstage Solar-mini API에 연결합니다.
*   **시스템 프롬프팅:** 엄격한 근거 기반 답변을 강제합니다 ("제공된 문맥에 기반해서만 답변할 것", "페이지 번호를 인용할 것").

#### 5.4 Step 4: 정량적 평가 (Evaluation)
*   **요구사항:** `Ragas`를 사용하여 RAG 파이프라인의 품질을 측정합니다.
*   **지표:** Faithfulness(충실도), Answer Relevance(답변 관련성).
*   **출력:** 터미널 또는 UI에 평가 점수를 표시합니다.

### 6. 프로젝트 구조 (Project Structure)
프로젝트는 깔끔하고 모듈화된 구조를 따라야 합니다:

```text
local-doc-rag/
├── data/                # 원본 PDF 파일 (예: Solar Model Guide)
├── db/                  # ChromaDB 영구 저장 디렉토리
├── src/
│   ├── ingest.py        # 로직: 파싱 및 인덱싱
│   ├── chain.py         # 로직: RAG 파이프라인 설정
│   └── eval.py          # 로직: Ragas 평가
├── app.py               # UI: Streamlit 진입점
└── requirements.txt     # 의존성 목록
```

### 7. 성공 지표 (Success Metrics)
*   **기능성:** 시스템이 PDF를 성공적으로 수집하고, 질문에 답변하며, 인용(출처)을 반환해야 합니다.
*   **성능:** Ragas 'Faithfulness' 점수 > 0.8 (목표).
*   **프레젠테이션:** README에 비교 표(예: 기본 RAG vs Solar MVP)를 포함합니다.

### 8. 전략적 차별화 포인트
*   데모 데이터로 **업스테이지** 관련 문서를 사용합니다.
*   가능한 경우 `UpstageEmbeddings`나 `ChatUpstage` 클래스를 명시적으로 사용하거나 호환성 코드를 강조합니다.
*   Ragas를 통한 "데이터 기반" 개선 루프 역량을 보여줍니다.
