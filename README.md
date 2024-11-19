Para criar uma solução que atenda ao desafio descrito, vamos utilizar a Azure Cognitive Services, mais especificamente o **Azure Translator** e **Azure Text Analytics**, com a ajuda de Python e da biblioteca **Gradio** para fornecer uma interface interativa.

### Passos para a solução:

1. **Instalação das dependências necessárias**:
   - `azure-ai-translation` para tradução.
   - `azure-ai-textanalytics` para análise de sentimentos e detecção de idioma.
   - `gradio` para a interface gráfica.
   - `PyMuPDF` (fitz) ou `PyPDF2` para leitura de arquivos PDF.
   - `langdetect` ou a API do Azure para detecção do idioma (a Azure faz isso automaticamente no serviço de Text Analytics).
   - `io` e `os` para manipulação de arquivos.

### Instalação das bibliotecas:

```bash
pip install azure-ai-translation azure-ai-textanalytics gradio PyMuPDF langdetect
```

### Passos de implementação:

1. **Configuração do Azure**:
   - Crie uma conta no Azure e configure os serviços de **Translator** e **Text Analytics**.
   - Obtenha as chaves da API e os endpoints de ambos os serviços.

2. **Desenvolvimento da função principal**:

```python
import gradio as gr
import os
from azure.ai.translation import TranslatorClient
from azure.core.credentials import AzureKeyCredential
from azure.ai.textanalytics import TextAnalyticsClient
from azure.core.credentials import AzureKeyCredential
import fitz  # PyMuPDF para ler PDF
from langdetect import detect  # Para detectar idioma se necessário

# Configuração do Azure (substitua pelos seus valores)
translator_endpoint = "YOUR_TRANSLATOR_ENDPOINT"
translator_key = "YOUR_TRANSLATOR_KEY"
text_analytics_endpoint = "YOUR_TEXT_ANALYTICS_ENDPOINT"
text_analytics_key = "YOUR_TEXT_ANALYTICS_KEY"

# Funções para acessar o Azure Translator e Azure Text Analytics

# Inicializar o cliente de tradução
translator_client = TranslatorClient(endpoint=translator_endpoint, credential=AzureKeyCredential(translator_key))

# Inicializar o cliente de Text Analytics
text_analytics_client = TextAnalyticsClient(endpoint=text_analytics_endpoint, credential=AzureKeyCredential(text_analytics_key))

# Função para tradução do texto
def traduzir_texto(texto, idioma_destino):
    response = translator_client.translate(content=texto, to=[idioma_destino])
    return response[0].translations[0].text

# Função para análise de sentimentos e detecção de idioma
def analisar_texto(texto):
    # Detectar o idioma
    idioma_detectado = detect(texto)

    # Análise de sentimentos
    documento = [texto]
    sentiment_response = text_analytics_client.analyze_sentiment(documents=documento)[0]
    sentimento = sentiment_response.sentiment
    score = sentiment_response.confidence_scores

    return idioma_detectado, sentimento, score

# Função para ler arquivos PDF e retornar o texto
def extrair_texto_pdf(pdf_file):
    doc = fitz.open(pdf_file.name)
    texto_completo = ""
    for pagina in doc:
        texto_completo += pagina.get_text()
    return texto_completo

# Função para ler arquivo de texto simples
def extrair_texto_arquivo(arquivo_texto):
    return arquivo_texto.read()

# Função principal para processamento
def processar_arquivo(arquivo, idioma_destino):
    if arquivo.name.endswith(".pdf"):
        texto = extrair_texto_pdf(arquivo)
    elif arquivo.name.endswith(".txt"):
        texto = extrair_texto_arquivo(arquivo)
    else:
        return "Tipo de arquivo não suportado. Por favor, envie um arquivo .txt ou .pdf."

    idioma, sentimento, scores = analisar_texto(texto)
    texto_traduzido = traduzir_texto(texto, idioma_destino)
    
    return {
        "Texto original": texto,
        "Idioma detectado": idioma,
        "Sentimento": sentimento,
        "Confiança (Sentimento)": scores.as_dict(),
        "Texto traduzido": texto_traduzido
    }

# Interface Gradio
def criar_interface():
    idiomas_disponiveis = ["en", "pt", "it", "es", "de", "fr", "zh-Hans", "ru"]
    
    # Interface Gradio para carregamento de arquivos e tradução
    iface = gr.Interface(
        fn=processar_arquivo,
        inputs=[gr.File(label="Carregar Arquivo (PDF ou TXT)"), 
                gr.Dropdown(label="Idioma de Destino", choices=idiomas_disponiveis, default="pt")],
        outputs=[gr.Textbox(label="Texto Original", lines=10),
                 gr.Textbox(label="Idioma Detectado", lines=1),
                 gr.Textbox(label="Sentimento", lines=1),
                 gr.Textbox(label="Confiança Sentimento", lines=1),
                 gr.Textbox(label="Texto Traduzido", lines=10)],
        title="Tradutor e Analisador de Sentimentos",
        description="Carregue um arquivo (PDF ou TXT) e escolha o idioma de destino. O sistema detectará o idioma original, "
                    "realizará a tradução e fornecerá a análise de sentimento do conteúdo.",
        live=True
    )
    
    iface.launch()

# Execução da interface
if __name__ == "__main__":
    criar_interface()
```

### Detalhamento do código:

- **Função `traduzir_texto`**: Utiliza o Azure Translator para traduzir o conteúdo para o idioma desejado.
- **Função `analisar_texto`**: Detecta o idioma do texto e faz a análise de sentimentos, retornando a pontuação de confiança.
- **Função `extrair_texto_pdf`**: Extrai texto de um arquivo PDF usando PyMuPDF.
- **Função `extrair_texto_arquivo`**: Lê o conteúdo de um arquivo de texto.
- **Interface Gradio**: Permite ao usuário carregar um arquivo e escolher o idioma de destino para tradução. A interface exibe o texto original, o idioma detectado, o sentimento e o texto traduzido.

### Como utilizar:

1. Ao rodar o script, você será apresentado com uma interface onde poderá carregar arquivos `.txt` ou `.pdf`.
2. Após carregar o arquivo, escolha o idioma de destino da tradução.
3. O sistema exibirá:
   - O texto original.
   - O idioma detectado.
   - A análise de sentimento (positivo, neutro ou negativo) com a pontuação de confiança.
   - O texto traduzido para o idioma escolhido.

### Exemplos de saída:

- **Texto Original**: "This is a sample technical article about machine learning."
- **Idioma Detectado**: "en"
- **Sentimento**: "neutral"
- **Confiança Sentimento**: {"positive": 0.25, "neutral": 0.70, "negative": 0.05}
- **Texto Traduzido**: "Este é um artigo técnico sobre aprendizado de máquina."

Essa implementação cobre a detecção de idioma, análise de sentimentos e tradução de conteúdos de arquivos .txt ou .pdf, com uma interface amigável e interativa proporcionada pela Gradio.
