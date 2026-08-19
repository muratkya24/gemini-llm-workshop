# Gemini LLM Workshop

Google Gemini ve LangChain ile **LLM → RAG → Agent** ilerleyişini adım adım gösteren workshop projesi.

Üç aşama var; her aşama bir öncekinin üzerine kuruluyor:

| Aşama | Dosya | Ne yapıyor |
|---|---|---|
| 1. Temel LLM çağrısı | `main.py` | Prompt şablonu ile Gemini'ye tek bir soru sorar |
| 2. RAG | `create_vector_db.py`, `rag.py` | Teknik dokümanı FAISS'e gömer, sadece doküman içeriğinden cevap üretir |
| 3. Agent | `tools.py`, `agent.py` | İki tool arasında kendi seçim yapan, konuşma hafızası olan ajan |

---

## Kurulum

### 1. Sanal ortam

```bash
python -m venv venv
source venv/bin/activate      # macOS / Linux
.\venv\Scripts\activate       # Windows
```

### 2. Bağımlılıklar

```bash
pip install -r requirements.txt
```

### 3. API anahtarı

Gemini anahtarını [Google AI Studio](https://aistudio.google.com/app/apikey) üzerinden alın.

```bash
cp .env.example .env
```

`.env` dosyasını açıp anahtarınızı yazın:

```
GOOGLE_API_KEY=sizin_anahtariniz
```

> `.env` dosyası `.gitignore` içinde — anahtarınız repoya gitmez. Anahtarı asla kod içine yazmayın.

---

## Çalıştırma

### Aşama 1 — İlk Gemini çağrısı

```bash
python main.py
```

`ChatPromptTemplate` ile system + human mesajı kurulur, modele gönderilir ve cevap yazdırılır. LangChain'in temel prompt → model akışını gösterir.

### Aşama 2 — RAG

Önce vektör veritabanını oluşturun:

```bash
python create_vector_db.py
```

Bu adım `mil_std_document.py` içindeki dokümanı 600 karakterlik parçalara böler, `gemini-embedding-001` ile embedding'e çevirir ve `faiss_index/` klasörüne kaydeder.

Sonra soruyu sorun:

```bash
python rag.py
```

Retriever en yakın 3 parçayı bulur, model **yalnızca** bu context'i kullanarak cevap verir. Cevap dokümanda yoksa "Bu bilgi dokümanda bulunmuyor." der.

> `faiss_index/` üretilen bir çıktıdır, repoda tutulmaz. `rag.py` ve `agent.py`'den önce `create_vector_db.py` çalıştırılmalıdır.

### Aşama 3 — Agent

```bash
python agent.py
```

Terminalden soru sorabileceğiniz interaktif bir döngü açılır. Ajan her soruda hangi tool'u kullandığını da yazdırır. Çıkmak için `Ctrl+C`.

İki tool var:

- **`mil_std_rag_tool`** — RKT-MIL-STD-001 dokümanıyla ilgili sorular (sıcaklık, nem, titreşim, şok, rakım, güç). Aşama 2'deki RAG zincirini kullanır.
- **`general_question_tool`** — dokümanla ilgisi olmayan genel sorular.

`InMemorySaver` sayesinde konuşma geçmişi tutulur, yani "peki ya minimum değeri?" gibi takip soruları çalışır.

Deneyebileceğiniz sorular:

```
Elektronik kontrol biriminin maksimum çalışma sıcaklığı kaç derecedir?
Titreşim testi kaç eksende uygulanır?
Peki her eksende ne kadar sürüyor?
Python'da liste ve tuple arasındaki fark nedir?
```

---

## Proje yapısı

```
main.py                 Aşama 1 — temel Gemini çağrısı
mil_std_document.py     Kurgusal teknik doküman (RAG kaynağı)
create_vector_db.py     Dokümanı parçalayıp FAISS index'ine yazar
rag.py                  Retriever + context'e bağlı cevap zinciri
tools.py                Agent'ın kullandığı iki tool
agent.py                Tool seçimi ve hafızası olan agent döngüsü
requirements.txt        Bağımlılıklar
.env.example            API anahtarı şablonu
```

## Kullanılan modeller

| Amaç | Model |
|---|---|
| Sohbet / üretim | `gemini-3.5-flash-lite` |
| Embedding | `gemini-embedding-001` |

## Not

`mil_std_document.py` içindeki **RKT-MIL-STD-001** dokümanı yalnızca workshop amacıyla hazırlanmış **kurgusal** bir metindir. Gerçek bir MIL-STD standardı değildir ve içindeki teknik değerler gerçek bir ürünü temsil etmez.
