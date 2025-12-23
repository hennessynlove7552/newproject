로컬 환경에서 개발 생산성을 극대화하면서도, 업스테이지의 **AI Model Production** 직무에서 요구하는 '데이터 파이프라인-RAG-평가' 역량을 한 번에 보여줄 수 있는 **MVP(Minimum Viable Product)** 설계안을 제안합니다.

이 프로젝트의 핵심은 **"로컬 리소스(CPU/RAM)로 가동하되, 핵심 추론만 API를 활용하여 효율을 높이는 것"**입니다.

---

## 🏗️ MVP 프로젝트: "Local-First Enterprise Doc-Informer"

이 프로젝트는 복잡한 비즈니스 문서를 로컬에서 색인(Indexing)하고, 질의응답을 수행하며, 그 성능을 수치로 증명하는 도구입니다.

### 1. 기술 스택 (Local-Lean Stack)

* **UI:** Streamlit (로컬 웹 서버)
* **Engine:** LangChain / LlamaIndex
* **Data Parsing:** `PyMuPDF` (빠른 속도) + `Unstructured` (구조적 파싱)
* **Vector DB:** `ChromaDB` (로컬 디렉토리에 `.db` 파일로 저장)
* **Embedding:** `HuggingFaceBgeEmbeddings` (로컬 CPU에서 구동 가능한 초경량 모델)
* **LLM:** **Upstage Solar API** (성능 최적화) 또는 **Ollama** (완전 로컬 구동 시)

---

### 2. 핵심 구현 단계 (MVP 로드맵)

#### **Step 1: 고성능 로컬 데이터 파싱 (Data Production)**

업스테이지는 데이터의 품질을 매우 중요하게 생각합니다. 단순 텍스트 추출이 아니라 **구조**를 유지하는 것이 포인트입니다.

* **Task:** PDF 내의 제목(Heading)과 본문을 구분하여 저장하는 로직 구현.
* **Skill Proof:** "단순 텍스트 추출이 아닌, 문서의 계층 구조를 이해하고 Chunking 전략을 세울 줄 안다."

#### **Step 2: 로컬 벡터 스토리지 구축 (Retrieval)**

* **Task:** `ChromaDB`를 사용하여 문서를 임베딩하고 저장. 검색 시 `Similarity Search`뿐만 아니라 `Max Marginal Relevance(MMR)`를 적용하여 답변의 다양성을 확보.
* **Skill Proof:** "검색 알고리즘의 특성을 이해하고 상황에 맞는 Retrieval 기법을 선택할 수 있다."

#### **Step 3: Solar API를 활용한 답변 생성 (Generation)**

* **Task:** 업스테이지의 **Solar-mini** 모델을 연결하여 답변 생성. 프롬프트에 "반드시 제공된 문서의 내용만 바탕으로 답할 것"과 "근거 페이지를 명시할 것"을 명시(System Prompting).
* **Skill Proof:** "최신 LLM API를 활용한 프롬프트 엔지니어링 및 할루시네이션 제어 역량."

#### **Step 4: 로컬 기반 성능 평가 (Evaluation)**

이 부분이 **채용 합격의 핵심**입니다.

* **Task:** `Ragas` 라이브러리를 사용하여 5~10개의 질문 세트에 대해 Faithfulness(충실도)와 Answer Relevance(관련성)를 계산하고 터미널에 출력.
* **Skill Proof:** "모델 도입 후 '좋아진 것 같다'는 감상 대신, 데이터 기반의 정량적 지표로 성능을 증명한다."

---

### 3. 프로젝트 폴더 구조 (예시)

구조가 깔끔해야 "Production" 역량이 있어 보입니다.

```text
local-doc-rag/
├── data/                # 테스트용 PDF 문서 (금융 보고서 등)
├── db/                  # ChromaDB 로컬 저장소
├── src/
│   ├── ingest.py        # 문서 전처리 및 임베딩 스크립트
│   ├── chain.py         # RAG 로직 (LangChain)
│   └── eval.py          # Ragas 평가 스크립트
├── app.py               # Streamlit UI 실행 파일
└── requirements.txt

```

---

### 🌟 업스테이지 맞춤형 "MVP 완성 전략"

1. **데이터의 차별화:** 흔한 뉴스 기사 대신, **업스테이지의 기술 블로그 포스트**나 **Solar 모델 가이드 PDF**를 데이터셋으로 써보세요. "우리 회사 기술에 관심이 많구나"라는 강한 인상을 줍니다.
2. **Solar 모델 활용 강조:** 코드 내에 `UpstageEmbeddings`나 `ChatUpstage` (LangChain 공식 통합)를 사용한 흔적을 남기세요.
3. **성능 비교 결과표:** "기본 RAG vs Solar 모델 적용 후 RAG"의 점수 차이를 README에 표로 정리하세요.

**이 MVP를 바로 시작하실 수 있도록, 첫 단계인 `ingest.py` (문서 전처리 및 로컬 DB 저장)의 핵심 코드를 짜드릴까요?**