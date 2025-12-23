# 튜토리얼: Upstage Solar를 활용한 Local-First Enterprise Doc-Informer 만들기

이 튜토리얼은 문서를 로컬에서 처리하여 비용을 절감하고, 고품질 추론을 위해 **Upstage Solar API**를 활용하는 RAG (Retrieval-Augmented Generation) 애플리케이션인 **Local-First Enterprise Doc-Informer**를 구축하는 과정을 안내합니다.

## 사전 요구 사항 (Prerequisites)

*   Python 3.10 이상 설치
*   Upstage API Key ([Upstage Console](https://console.upstage.ai/)에서 발급 가능)
*   Python 및 가상 환경(Virtual Environments)에 대한 기초 지식

## 프로젝트 구조 (Project Structure)

다음과 같은 구조로 프로젝트를 구축할 것입니다:

```text
local-doc-rag/
├── data/                # PDF 문서를 이곳에 위치시킵니다.
├── db/                  # 벡터 임베딩을 저장하는 폴더입니다.
├── src/
│   ├── ingest.py        # PDF 파싱 및 DB 저장을 처리합니다.
│   ├── chain.py         # RAG 파이프라인을 설정합니다.
│   └── eval.py          # 성능 평가를 수행합니다.
├── app.py               # Streamlit 대시보드입니다.
└── requirements.txt
```

---

## Step 1: 환경 설정 (Environment Setup)

1.  **가상 환경 생성:**
    ```bash
    python -m venv venv
    source venv/bin/activate  # Mac/Linux
    # venv\Scripts\activate  # Windows
    ```

2.  **의존성 설치:**
    `requirements.txt` 파일을 생성합니다:
    ```text
    langchain-upstage
    langchain-community
    langchain-chroma
    langchain-text-splitters
    pymupdf
    unstructured
    ragas
    streamlit
    python-dotenv
    ```
    설치를 진행합니다:
    ```bash
    pip install -r requirements.txt
    ```

3.  **API Key 설정:**
    루트 디렉토리에 `.env` 파일을 생성합니다:
    ```env
    UPSTAGE_API_KEY=your_api_key_here
    ```

---

## Step 2: 고성능 데이터 수집 (`src/ingest.py`)

PDF를 파싱하여 로컬 벡터 DB(`ChromaDB`)에 저장해야 합니다. 가능하다면 `UpstageEmbeddings`를 사용하고, 완전한 로컬 리소스 사용을 원한다면 `HuggingFaceBgeEmbeddings`와 같은 로컬 대안을 사용할 수 있습니다. *이 튜토리얼에서는 편의를 위해 표준 Upstage 통합을 사용하지만, 임베딩 모델은 변경 가능합니다.*

```python
import os
from dotenv import load_dotenv
from langchain_community.document_loaders import PyMuPDFLoader
from langchain_text_splitters import RecursiveCharacterTextSplitter
from langchain_chroma import Chroma
from langchain_upstage import UpstageEmbeddings

load_dotenv()

def ingest_docs(pdf_path):
    # 1. PDF 로드
    print(f"Loading {pdf_path}...")
    loader = PyMuPDFLoader(pdf_path)
    docs = loader.load()

    # 2. 텍스트 분할 (Chunking)
    text_splitter = RecursiveCharacterTextSplitter(
        chunk_size=1000,
        chunk_overlap=100
    )
    splits = text_splitter.split_documents(docs)

    # 3. 벡터 DB에 저장
    print("Embedding and storing...")
    vectorstore = Chroma.from_documents(
        documents=splits,
        embedding=UpstageEmbeddings(model="solar-embedding-1-large"),
        persist_directory="./db"
    )
    print("Ingestion Complete!")

if __name__ == "__main__":
    # data/ 폴더에 PDF가 있는지 확인하세요
    ingest_docs("./data/sample.pdf")
```

---

## Step 3: RAG 파이프라인 (`src/chain.py`)

이제 검색(Retrieval) 및 생성(Generation) 로직을 생성합니다. 다양한 맥락을 확보하기 위해 **Max Marginal Relevance (MMR)**를 사용합니다.

```python
from langchain_chroma import Chroma
from langchain_upstage import UpstageEmbeddings, ChatUpstage
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough

def get_rag_chain():
    # 1. DB 연결
    vectorstore = Chroma(
        persist_directory="./db",
        embedding_function=UpstageEmbeddings(model="solar-embedding-1-large")
    )
    retriever = vectorstore.as_retriever(
        search_type="mmr",
        search_kwargs={"k": 5, "fetch_k": 10}
    )

    # 2. 프롬프트 정의
    template = """You are an assistant for analyzing enterprise documents.
    Answer the question based ONLY on the following context.
    If you cannot find the answer, say "I don't know based on this document."
    
    Context:
    {context}
    
    Question: {question}
    """
    prompt = ChatPromptTemplate.from_template(template)

    # 3. LLM 정의 (Solar)
    llm = ChatUpstage()

    # 4. 체인 연결
    chain = (
        {"context": retriever, "question": RunnablePassthrough()}
        | prompt
        | llm
        | StrOutputParser()
    )
    return chain
```

---

## Step 4: 평가 (`src/eval.py`)

이 단계는 "Production" 직무에서 매우 중요합니다. `Ragas`를 사용하여 품질을 검증합니다.

```python
from ragas import evaluate
from ragas.metrics import faithfulness, answer_relevance
from datasets import Dataset
from src.chain import get_rag_chain

def run_evaluation():
    chain = get_rag_chain()
    
    # 테스트 데이터
    questions = ["What is the main topic of the document?", "Explain the pricing model."]
    ground_truths = [["Topic A"], ["Pricing B"]] # 일부 지표 선택 시 필요
    
    answers = []
    contexts = []

    for q in questions:
        response = chain.invoke(q)
        answers.append(response)
        # 간단한 ragas 설정을 위해 수동으로 context를 추출하거나 trace가 필요할 수 있습니다.
        # 여기서는 단순화된 placeholder입니다.
        contexts.append(["Context placeholder..."]) 

    data = {
        "question": questions,
        "answer": answers,
        "contexts": contexts,
        "ground_truth": ground_truths
    }
    
    dataset = Dataset.from_dict(data)
    
    results = evaluate(
        dataset=dataset,
        metrics=[faithfulness, answer_relevance]
    )
    
    print(results)

if __name__ == "__main__":
    run_evaluation()
```

---

## Step 5: 사용자 인터페이스 (`app.py`)

마지막으로, 문서와 상호작용하기 위한 간단한 Streamlit UI입니다.

```python
import streamlit as st
from src.chain import get_rag_chain

st.title("📄 Local-First Doc Informer")

if "messages" not in st.session_state:
    st.session_state.messages = []

for message in st.session_state.messages:
    with st.chat_message(message["role"]):
        st.markdown(message["content"])

if prompt := st.chat_input("Ask about your document"):
    st.session_state.messages.append({"role": "user", "content": prompt})
    with st.chat_message("user"):
        st.markdown(prompt)

    with st.chat_message("assistant"):
        chain = get_rag_chain()
        response = chain.invoke(prompt)
        st.markdown(response)
    
    st.session_state.messages.append({"role": "assistant", "content": response})
```

## 앱 실행 (Running the App)

1.  `data/` 폴더에 PDF 파일을 넣습니다.
2.  수집(ingestion) 실행: `python src/ingest.py`
3.  UI 시작: `streamlit run app.py`

고성능 Local-First RAG 앱을 경험해보세요!
